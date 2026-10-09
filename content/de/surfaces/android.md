---
title: "Android"
linkTitle: "Android"
weight: 50
description: "go-widgets/android: eine installierbare APK, deren gesamte Oberfläche von einem CGO_ENABLED=0-Go-Prozess angeordnet und gezeichnet wird."
tags: [oberflächen, android]
---

Android ist die einzige Plattform, auf der ein CGO-freier Prozess kein Fenster besitzen kann: Jeder
Weg zu einer zeichenbaren Oberfläche führt über JNI, und JNI braucht cgo. Es gibt kein
Wire-Protokoll, das man stattdessen sprechen könnte, so wie X11 und Wayland eines haben.

Deshalb besteht [`go-widgets/android`](https://github.com/go-widgets/android) aus zwei
Prozessen:

| | |
|---|---|
| **Java-Host** (`host/`) | besitzt die `Activity`, die `SurfaceView`, Touch, Tasten und den Lebenszyklus. Überträgt Pixel per Blit; weiß nichts über Widgets. |
| **Go-Anwendung** (`cmd/gwapp`) | ein gewöhnliches `CGO_ENABLED=0 GOOS=android`-Executable. Besitzt Layout, Widgets, Theme, Hit-Testing und Fokus, unverändert gegenüber jedem anderen Back-End. |

Der Java-Host steht dort, wo unter Linux der X-Server steht. Pixel reisen durch
ein **memfd**, das die Anwendung anlegt und dem Host über den Socket übergibt;
die Go-Seite schreibt RGBA_8888, was Byte für Byte Androids ARGB_8888 entspricht, sodass
der Blit eine Kopie ohne Umwandlung ist, und zwar nur des geschädigten Rechtecks. Eingaben,
Insets, IME-Text und der Barrierefreiheitsbaum gehen über ein gerahmtes Protokoll über einen
abstrakten `LocalSocket`.

```go
c, err := android.Dial("my app", nil) // nil theme: toolkit.DefaultDark()
if errors.Is(err, android.ErrUnsupported) {
	return nil // not under a host
}
defer c.Close()
return c.Run(myWidgetTree())
```

`Client` erfüllt das `Backend` von `window`, sodass eine Anwendung zwischen
Android und den Desktop-Back-Ends wechselt, ohne eine Zeile oberhalb des Fensters zu ändern –
und [`window.Open`]({{< relref "/surfaces/native-window.md#how-open-chooses" >}})
wählt es von selbst, wenn der Socket des Hosts vorhanden ist.

## Nur arm64 {#arm64-only}

`android/arm64` ist das einzige Android-Ziel, das Go ohne cgo linkt; `arm`,
`amd64` und `386` erfordern externes Linken. Die CI prüft beide Hälften – das eine Ziel,
das baut, und die drei, die es nicht tun –, sodass der Build es meldet, sobald Go die Einschränkung aufhebt.

## Barrierefreiheit {#accessibility}

Ein Screenreader würde die `SurfaceView` als ein einziges undurchsichtiges Rechteck sehen. Der Host
gibt ihm stattdessen eine virtuelle View-Hierarchie: einen Knoten pro zugänglichem Widget,
mit der `android.widget.*`-Klasse, nach der Android seine Ansagen auswählt, dem
Text, den Grenzen und einer Aktivierungsaktion. Der Baum wird **abgeholt**, wenn
etwas ihn liest, nie gepusht, sodass eine App ohne angeschlossenen Barrierefreiheitsdienst
nie einen aufbaut. Eine Aktivierung kommt als gewöhnlicher Klick in der
Mitte des Elements zurück, über denselben Code, den eine Berührung nimmt.

## Touch und Animation {#touch-and-animation}

Jede Touch-Abtastung erreicht den Baum als Touch-Ereignis und dann als Maus-Ereignis: das
erste für gestenfähige Widgets, das zweite für die vielen, die nur auf
Klicks hören. Animierte Widgets laufen auf einer Frame-Schleife weiter, die startet, wenn etwas
zu animieren beginnt, und stoppt, wenn nichts mehr animiert.
