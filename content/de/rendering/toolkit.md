---
title: "Das Toolkit"
linkTitle: "Toolkit"
weight: 20
description: "160 Widget-Typen, gezeichnet über die Painter-Schnittstelle: Themes, Layouts, Text, Eingabe, Barrierefreiheit."
tags: [rendering, toolkit, widgets, theme, barrierefreiheit]
---

[`go-widgets/toolkit`](https://github.com/go-widgets/toolkit) ist der Widget-Satz.
Jedes Widget implementiert dieselbe Schnittstelle – `Bounds`, `SetBounds`,
`Draw(p painter.Painter, theme *Theme)`, `HitTest`, `OnEvent` –, sodass ein Host
einen Button, eine Tabelle und einen Terminkalender gleich behandelt. Nur Standardbibliothek,
`CGO_ENABLED=0`, 100 % Anweisungsabdeckung.

In v0.329.0 hat es **160 exportierte Widget-Typen** in etwa **69.000 Zeilen**
Widget-Code. Beide Zahlen sind an diesem Tag gemessen, nicht übernommen: Das
README des Toolkits nennt die Befehle, und man sollte sie erneut ausführen, statt
die Zahlen zu wiederholen.

## Die Familien {#the-families}

| Familie | Widgets |
|---|---|
| Aktion | `Button`, `ToggleButton`, `CheckButton`, `RadioButton` + `RadioGroup`, `Switch`, `SplitButton`, `IconButton`, `CycleButton`, `Chip`, `SegmentedBar` |
| Eingabe | `Entry`, `TextView` (Auswahl, IME-Vorschau, Syntax-Spans, Zeilennummern), `SpinButton`, `Scale`, `RangeSlider`, `SearchEntry`, `TagField`, `ComboBox`, `FormField` mit `Validate`/`Rule` |
| Auswahl | `ListBox`, `TreeView`, `DropDown` |
| Container | `Container` mit austauschbarem `Layout` (`FitLayout`, `BoxLayout`, `BorderLayout`, `CardLayout`, `FlowLayout`); `HBox`, `VBox`, `Grid`, `Frame`, `Dock`, `Border`, `Stack`, `Overlay`, `Paned`, `Splitter`, `Expander`, `Accordion`; ein deklarativer `Node`-Builder |
| Navigation | `Menu`, `MenuBar` (mit `Alt`+Buchstabe), `ContextMenu`, `Popover`, `CommandPalette`, `Dialog`, `MessageDialog`, `Wizard`, `Notebook`, `ViewSwitcher`, `Carousel` |
| Rückmeldung | `ProgressBar`, `ProgressCircle`, `LevelBar`, `Spinner`, `Tooltip`, `Notification`, `Toast`, `Banner`, `Alert`, `Badge`, `Skeleton` |
| Daten | `Table` (Zellbearbeitung, fixierte Spalten, Gruppenzeilen, Aggregate, aufklappbare Zeilen), `TreeTable`, `PropertyGrid`, `Kanban`, `Gantt`, `Agenda` (Woche, Monat, Quartal, Jahr; mehrere Kalender) |
| Diagramme | `LineChart`, `BarChart`, `PieChart`, `AreaChart`, `ScatterChart`, `RadarChart`, `Gauge`, `Sparkline` |
| Zusammengesetzt | `FileChooser`, `ColorChooser`, `ColorPicker`, `FontChooser`, `Calendar`, `DatePicker`, `DateRangePicker`, `TimePicker`, `MarkdownView`, `MarkdownEditor`, `TerminalView` |
| Shell | `Window` mit clientseitigen Dekorationen, `HeaderBar`, `Toolbar`, `Ribbon`, `Statusbar`, `StatusIcon`, `StatusArea`, `Wallpaper`, `Thumbnail` |

Jedes davon, live: die [Galerie](https://go-widgets.github.io/gallery/).

## Ribbon {#ribbon}

`Ribbon` ist ein Befehlsband im Office-Stil: Registerkarten mit beschrifteten
Gruppen aus großen Schaltflächen (Symbol über dem Text), kleinen (Symbol
daneben), Umschaltern, geteilten Schaltflächen, Auswahllisten und einer kleinen
Galerie. Es folgt den Windows-Richtlinien für Ribbons: Wird das Fenster
schmaler, verkleinert sich jede Gruppe nach ihrer Priorität — voll, kompakt,
nur Symbole, dann eine einzelne Menüschaltfläche —, sodass das Band die größte
Kombination zeigt, die passt; es klappt auf seine Registerkarten zusammen
(Strg+F1 oder der Pfeil), und eine Registerkarte zeigt dann kurz ihren Inhalt;
Alt blendet Tastenkürzel-Hinweise ein; QuickInfos nennen die Tastenkombination
und eine Beschreibung. Navigation per Registerkarten und Symbolleiste folgt den
WAI-ARIA-Mustern, und Umrisse und Fokusring behalten in hellen wie dunklen
Themes einen Kontrast von 3:1.

Jedes Steuerelement ist eine Sicht auf eine `Action` — den einen Befehl, den
sich Menü, Palette und Ribbon teilen —, und `NewCommandAction` erzeugt eine aus
einem `mvvm.Command` des Viewmodels, dessen `CanExecute` das Element ausgraut.
Aktive Registerkarte, Zusammenklapp-Zustand, ein Umschalter und die Auswahl
einer Liste sind `mvvm.Observable`s. Unter einem `PopoverHost` zeichnen sich
die Menüs ohne Host-Code über die Seite.

`RibbonGrid` ist eine Steuerelementart, deren Popup ein Raster aus `GridCols`
mal `GridRows` Zellen ist (8 mal 6 bei null, höchstens 16 mal 16), wie die
Schaltfläche Tabelle einer Textverarbeitung. Zeiger oder Pfeiltasten
beleuchten die Zellen von der Ecke bis zu der darunter, und eine Beschriftung
nennt „3 × 2“; Klick oder Eingabe wählt und ruft `OnGrid(cols, rows)` auf;
Pos1 und Ende springen, Esc schließt. Es wird hell und dunkel gezeichnet und
Hilfstechnologien als Schaltfläche gemeldet, die aufgeklappt ist, solange das
Raster offen ist.

```go
&toolkit.RibbonControl{Kind: toolkit.RibbonGrid, Label: "Table",
	GridCols: 8, GridRows: 8,
	OnGrid: insertTable}
```

## Splitter {#splitter}

`Splitter` ordnet N Bereiche nebeneinander (oder mit `vertical` gleich true
untereinander) an und setzt zwischen je zwei benachbarte einen ziehbaren,
fokussierbaren Teiler, nach dem WAI-ARIA-Muster „Window Splitter“. Ein Bereich
ist entweder **fest** — seine `Size` in logischen Pixeln ist ein
`mvvm.Observable[int]`, den der Teiler schreibt — oder **flexibel** und teilt
sich nach seinem `Weight`, ebenfalls ein `Observable`, was die festen Bereiche
übrig lassen. Jeder Bereich hat ein `Min` und ein `Max`. Ein `Collapsible`
fester Bereich schließt sich (Größe 0), wenn der Teiler unter die Hälfte seines
`Min` gezogen wird, oder per Doppelklick oder Enter, und öffnet sich auf
dieselbe Weise wieder. Das `Visible`-Observable eines Bereichs schaltet ihn samt
Teiler ab. Da die Größen in Observables liegen, kann ein Viewmodel sie binden,
speichern und wiederherstellen; `OnResized` wird einmal je beendeter Geste mit
den Bereichen aufgerufen, die die Person geändert hat, nie für ein Layout, das
nur einer Fenstergrößenänderung folgte.

```go
files := mvvm.NewObservable(240) // fixed pane, logical px
w := mvvm.NewObservable(1)       // flexible pane, weight

s := toolkit.NewSplitter(false,
	&toolkit.SplitPane{Widget: tree, Name: "Files", Size: files,
		Min: 120, Max: 480, Collapsible: true},
	&toolkit.SplitPane{Widget: editor, Name: "Editor", Weight: w, Min: 200},
)
s.OnResized = func(changed []int) { save(files.Get(), w.Get()) }
```

Ein Teiler erhält den Fokus: Die Pfeiltasten (hoch und runter bei vertikaler
Anordnung) bewegen ihn um `Step` logische Pixel, mit Umschalt um das Vierfache;
Pos1 und Ende bringen den davorliegenden Bereich auf seine kleinste und größte
Größe; Enter schließt oder stellt den festen Bereich wieder her, den er steuert;
Tab wechselt zwischen den Teilern. Jeder Teiler ist ein `RoleSeparator` mit
`aria-valuemin`, `aria-valuemax` und `aria-valuenow`, benannt „Resize“ plus
`Name` des Bereichs. `Cursor(x, y)` liefert über einem Teiler `"col-resize"`
oder `"row-resize"`, und `SplitterColors(theme)` gibt Linie und Akzent des
Teilers an, die in beiden Standard-Themes einen Kontrast von 3:1 zu Hintergrund
und Oberfläche behalten, ebenso wie der Fokusring.

## Themes {#themes}

Ein einziger `Theme`-Wert kaskadiert durch den ganzen Baum: Ändert man eine Farbe an der
Wurzel, zeichnet sich jedes Widget damit neu. `DefaultLight()` und `DefaultDark()`
sind eingebaut, und `LoadGTKTheme(css)` liest den `@define-color`-Block jedes
GTK-3- oder libadwaita-Themes – Adwaita, WhiteSur, Solarized – in ein `Theme` ein.

## Text {#text}

Der einkompilierte Standard ist eine 5×7-Bitmap-Schrift, damit Pixeltests ihre
Geometrie behalten. Ein Aufruf schaltet die ganze UI auf kantengeglätteten, geshapten Text um:

```go
toolkit.UseOpenTypeText() // Atkinson Hyperlegible, bundled; js/wasm-safe
```

Die Schrift stammt aus [go-opentype/fonts](https://github.com/go-opentype/fonts)
und wird von [go-opentype](https://github.com/go-opentype/opentype) gerastert,
in Go, ohne C-Schriftbibliothek und ohne Suche nach Systemschriften.
`NewTrueTypeFont(ttf, px)` lädt eine andere Schrift; `NewFallbackFont` verkettet
mehrere, etwa um eine CJK- oder arabische Schrift hinzuzufügen; das eigene `Font`-Feld eines Widgets
überschreibt die globale Schrift nur für dieses Widget.

## Eingabe {#input}

Maus, Tastatur, IME-Komposition, Touch (`EventTouchStart`/`Move`/`End`, mit
`GestureRecognizer` für Tippen, langes Drücken und Wischen), Drag and Drop
(`DragSource`, `DropTarget`). Mehrfachauswahl von Zeilen in `ListBox`, `Table` und
`TreeView` funktioniert mit der Maus und mit der Tastatur und beachtet ⌘ ebenso
wie Ctrl, weil unter macOS Ctrl-Klick der Sekundärklick ist.

## Binärgröße: ein Programm linkt, was es benutzt {#binary-size}

Das Toolkit ist ein großes Paket, und für den Linker ist das in Ordnung: Die
Dead-Code-Eliminierung von Go behält nur, was von `main.main` **und von den
Init-Aufgaben jedes importierten Pakets** aus erreichbar ist. Widgets, die ein
Programm nie berührt, fallen weg. Nicht weg fällt alles, was ein Initialisierer
auf Paketebene oder das `init` eines importierten Pakets erreicht; seit
v0.345.0 hält das Toolkit deshalb seine eigenen Initialisierer frei von Arbeit
und vermeidet Pakete, deren `init` Code nachzieht:

- `Material` und `Thumbnail` nutzen das codec-freie
  [`github.com/go-images/images/ops`](https://github.com/go-images/images)
  statt `github.com/go-images/images`, dessen Decoder sich in `init`
  registrieren (etwa 1,7 MB WebAssembly-Code).
- `CollabText`, `NewCollabText`, `MetaName` und `MetaColor` sind nach
  **`github.com/go-widgets/toolkit/collabui`** umgezogen: Sie brauchen eine
  laufende `go-crdt/collab`-Sitzung, deren Transporte `net/http` und
  `crypto/tls` hineinziehen. `CollabSheet` und `IsoCRDTDocument` brauchen nur
  `go-crdt/crdt` und bleiben in `toolkit`.
- Der reguläre Ausdruck der E-Mail-`Rule`, das eingebaute Iso-Icon-Register und
  die Schaltflächentabelle der `RichEditorToolbar` werden bei der ersten
  Benutzung gebaut, nicht von Initialisierern.

⛔ **Der Umzug ist eine inkompatible Änderung, ohne veraltete Weiterleitung.**
Ändern Sie den Import der vier Namen oben auf `collabui`; sonst ändert sich
nichts.

Größen für `GOOS=js GOARCH=wasm` mit `-ldflags="-s -w"`, roh und gzip -9:

| Programm | vorher (roh / gzip) | nachher (roh / gzip) |
|---|---|---|
| ein `Label` | 10,49 / 2,88 MB | 7,48 / 2,13 MB |
| go-wasmcontainers/dashboard | 20,42 / 5,17 MB | 17,72 / 4,58 MB |
| die Relay-Flusskarte | 22,62 / 5,44 MB | 19,95 / 4,85 MB |

Was übrig bleibt, ist vor allem die Laufzeit und der OpenType-Stack, den das
Textrendering braucht. Um zu sehen, was ein Symbol in Ihrem Programm hält,
bauen Sie mit `-ldflags=-dumpdep` und folgen der Kette der ersten Eltern vom
Symbol bis `main.main` oder `go:main.inittasks`.

v0.345.0 bringt außerdem `ColorHex` und `ParseColorHex`, um eine Farbe als Hex
zu schreiben und wieder zu lesen.

## Barrierefreiheit {#accessibility}

Ein Toolkit, das seine eigenen Pixel zeichnet, gibt einem Screenreader nichts zu lesen,
solange es nicht sagt, was es gezeichnet hat. Widgets implementieren `Accessible` – eine Rolle, einen Namen,
einen Wert –, und `CollectA11y` sammelt sie, damit der Host sie veröffentlicht: AT-SPI unter
Linux über [`window`]({{< relref "/surfaces/native-window.md" >}}) und eine
virtuelle View-Hierarchie unter [Android]({{< relref "/surfaces/android.md#accessibility" >}}).
