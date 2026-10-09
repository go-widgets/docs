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

In v0.328.0 hat es **160 exportierte Widget-Typen** in etwa **69.000 Zeilen**
Widget-Code. Beide Zahlen sind an diesem Tag gemessen, nicht übernommen: Das
README des Toolkits nennt die Befehle, und man sollte sie erneut ausführen, statt
die Zahlen zu wiederholen.

## Die Familien {#the-families}

| Familie | Widgets |
|---|---|
| Aktion | `Button`, `ToggleButton`, `CheckButton`, `RadioButton` + `RadioGroup`, `Switch`, `SplitButton`, `IconButton`, `CycleButton`, `Chip`, `SegmentedBar` |
| Eingabe | `Entry`, `TextView` (Auswahl, IME-Vorschau, Syntax-Spans, Zeilennummern), `SpinButton`, `Scale`, `RangeSlider`, `SearchEntry`, `TagField`, `ComboBox`, `FormField` mit `Validate`/`Rule` |
| Auswahl | `ListBox`, `TreeView`, `DropDown` |
| Container | `Container` mit austauschbarem `Layout` (`FitLayout`, `BoxLayout`, `BorderLayout`, `CardLayout`, `FlowLayout`); `HBox`, `VBox`, `Grid`, `Frame`, `Dock`, `Border`, `Stack`, `Overlay`, `Paned`, `Expander`, `Accordion`; ein deklarativer `Node`-Builder |
| Navigation | `Menu`, `MenuBar` (mit `Alt`+Buchstabe), `ContextMenu`, `Popover`, `CommandPalette`, `Dialog`, `MessageDialog`, `Wizard`, `Notebook`, `ViewSwitcher`, `Carousel` |
| Rückmeldung | `ProgressBar`, `ProgressCircle`, `LevelBar`, `Spinner`, `Tooltip`, `Notification`, `Toast`, `Banner`, `Alert`, `Badge`, `Skeleton` |
| Daten | `Table` (Zellbearbeitung, fixierte Spalten, Gruppenzeilen, Aggregate, aufklappbare Zeilen), `TreeTable`, `PropertyGrid`, `Kanban`, `Gantt`, `Agenda` (Woche, Monat, Quartal, Jahr; mehrere Kalender) |
| Diagramme | `LineChart`, `BarChart`, `PieChart`, `AreaChart`, `ScatterChart`, `RadarChart`, `Gauge`, `Sparkline` |
| Zusammengesetzt | `FileChooser`, `ColorChooser`, `ColorPicker`, `FontChooser`, `Calendar`, `DatePicker`, `DateRangePicker`, `TimePicker`, `MarkdownView`, `MarkdownEditor`, `TerminalView` |
| Shell | `Window` mit clientseitigen Dekorationen, `HeaderBar`, `Toolbar`, `Statusbar`, `StatusIcon`, `StatusArea`, `Wallpaper`, `Thumbnail` |

Jedes davon, live: die [Galerie](https://go-widgets.github.io/gallery/).

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

## Barrierefreiheit {#accessibility}

Ein Toolkit, das seine eigenen Pixel zeichnet, gibt einem Screenreader nichts zu lesen,
solange es nicht sagt, was es gezeichnet hat. Widgets implementieren `Accessible` – eine Rolle, einen Namen,
einen Wert –, und `CollectA11y` sammelt sie, damit der Host sie veröffentlicht: AT-SPI unter
Linux über [`window`]({{< relref "/surfaces/native-window.md" >}}) und eine
virtuelle View-Hierarchie unter [Android]({{< relref "/surfaces/android.md#accessibility" >}}).
