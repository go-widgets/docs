---
title: "The toolkit"
linkTitle: "Toolkit"
weight: 20
description: "160 widget types drawn through the painter seam: themes, layouts, text, input, accessibility."
tags: [rendering, toolkit, widgets, theme, accessibility]
---

[`go-widgets/toolkit`](https://github.com/go-widgets/toolkit) is the widget
set. Every widget implements the same interface — `Bounds`, `SetBounds`,
`Draw(p painter.Painter, theme *Theme)`, `HitTest`, `OnEvent` — so a host
treats a button, a table and an event calendar alike. Standard library only,
`CGO_ENABLED=0`, 100% statement coverage.

At v0.328.0 it has **160 exported widget types** in about **69,000 lines** of
widget code. Those two numbers are measured at that tag, not carried over: the
toolkit's README gives the commands, and they should be run again rather than
the figures repeated.

## The families

| Family | Widgets |
|---|---|
| Action | `Button`, `ToggleButton`, `CheckButton`, `RadioButton` + `RadioGroup`, `Switch`, `SplitButton`, `IconButton`, `CycleButton`, `Chip`, `SegmentedBar` |
| Input | `Entry`, `TextView` (selection, IME preview, syntax spans, line numbers), `SpinButton`, `Scale`, `RangeSlider`, `SearchEntry`, `TagField`, `ComboBox`, `FormField` with `Validate`/`Rule` |
| Selection | `ListBox`, `TreeView`, `DropDown` |
| Containers | `Container` with a swappable `Layout` (`FitLayout`, `BoxLayout`, `BorderLayout`, `CardLayout`, `FlowLayout`); `HBox`, `VBox`, `Grid`, `Frame`, `Dock`, `Border`, `Stack`, `Overlay`, `Paned`, `Expander`, `Accordion`; a declarative `Node` builder |
| Navigation | `Menu`, `MenuBar` (with `Alt`+letter), `ContextMenu`, `Popover`, `CommandPalette`, `Dialog`, `MessageDialog`, `Wizard`, `Notebook`, `ViewSwitcher`, `Carousel` |
| Feedback | `ProgressBar`, `ProgressCircle`, `LevelBar`, `Spinner`, `Tooltip`, `Notification`, `Toast`, `Banner`, `Alert`, `Badge`, `Skeleton` |
| Data | `Table` (cell editing, frozen columns, group rows, aggregates, row expanders), `TreeTable`, `PropertyGrid`, `Kanban`, `Gantt`, `Agenda` (week, month, quarter, year; several calendars) |
| Charts | `LineChart`, `BarChart`, `PieChart`, `AreaChart`, `ScatterChart`, `RadarChart`, `Gauge`, `Sparkline` |
| Composite | `FileChooser`, `ColorChooser`, `ColorPicker`, `FontChooser`, `Calendar`, `DatePicker`, `DateRangePicker`, `TimePicker`, `MarkdownView`, `MarkdownEditor`, `TerminalView` |
| Shell | `Window` with client-side decorations, `HeaderBar`, `Toolbar`, `Statusbar`, `StatusIcon`, `StatusArea`, `Wallpaper`, `Thumbnail` |

Every one of them, live: the [gallery](https://go-widgets.github.io/gallery/).

## Themes

One `Theme` value cascades through the whole tree: change a colour at the
root and every widget repaints with it. `DefaultLight()` and `DefaultDark()`
are built in, and `LoadGTKTheme(css)` reads the `@define-color` block of any
GTK 3 or libadwaita theme — Adwaita, WhiteSur, Solarized — into a `Theme`.

## Text

The compiled-in default is a 5×7 bitmap font, so pixel tests keep their
geometry. One call switches the whole UI to anti-aliased, shaped text:

```go
toolkit.UseOpenTypeText() // Atkinson Hyperlegible, bundled; js/wasm-safe
```

The face comes from [go-opentype/fonts](https://github.com/go-opentype/fonts)
and is rasterised by [go-opentype](https://github.com/go-opentype/opentype),
in Go, with no C font library and no system font lookup.
`NewTrueTypeFont(ttf, px)` loads another face; `NewFallbackFont` chains
several, for example to add a CJK or Arabic face; a widget's own `Font` field
overrides the global one for that widget alone.

## Input

Mouse, keyboard, IME composition, touch (`EventTouchStart`/`Move`/`End`, with
`GestureRecognizer` for taps, long presses and swipes), drag and drop
(`DragSource`, `DropTarget`). Multi-row selection in `ListBox`, `Table` and
`TreeView` works from the mouse and from the keyboard, and honours ⌘ as well
as Ctrl, because on macOS Ctrl-click is the secondary click.

## Accessibility

A toolkit that paints its own pixels gives a screen reader nothing to read
unless it says what it drew. Widgets implement `Accessible` — a role, a name,
a value — and `CollectA11y` gathers them for the host to publish: AT-SPI on
Linux through [`window`]({{< relref "/surfaces/native-window.md" >}}), and a
virtual view hierarchy on [Android]({{< relref "/surfaces/android.md#accessibility" >}}).
