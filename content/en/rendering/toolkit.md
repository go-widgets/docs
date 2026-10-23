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

At v0.329.0 it has **160 exported widget types** in about **69,000 lines** of
widget code. Those two numbers are measured at that tag, not carried over: the
toolkit's README gives the commands, and they should be run again rather than
the figures repeated.

## The families

| Family | Widgets |
|---|---|
| Action | `Button`, `ToggleButton`, `CheckButton`, `RadioButton` + `RadioGroup`, `Switch`, `SplitButton`, `IconButton`, `CycleButton`, `Chip`, `SegmentedBar` |
| Input | `Entry`, `TextView` (selection, IME preview, syntax spans, line numbers), `SpinButton`, `Scale`, `RangeSlider`, `SearchEntry`, `TagField`, `ComboBox`, `FormField` with `Validate`/`Rule` |
| Selection | `ListBox`, `TreeView`, `DropDown` |
| Containers | `Container` with a swappable `Layout` (`FitLayout`, `BoxLayout`, `BorderLayout`, `CardLayout`, `FlowLayout`); `HBox`, `VBox`, `Grid`, `Frame`, `Dock`, `Border`, `Stack`, `Overlay`, `Paned`, `Splitter`, `Expander`, `Accordion`; a declarative `Node` builder |
| Navigation | `Menu`, `MenuBar` (with `Alt`+letter), `ContextMenu`, `Popover`, `CommandPalette`, `Dialog`, `MessageDialog`, `Wizard`, `Notebook`, `ViewSwitcher`, `Carousel` |
| Feedback | `ProgressBar`, `ProgressCircle`, `LevelBar`, `Spinner`, `Tooltip`, `Notification`, `Toast`, `Banner`, `Alert`, `Badge`, `Skeleton` |
| Data | `Table` (cell editing, frozen columns, group rows, aggregates, row expanders), `TreeTable`, `PropertyGrid`, `Kanban`, `Gantt`, `Agenda` (week, month, quarter, year; several calendars) |
| Charts | `LineChart`, `BarChart`, `PieChart`, `AreaChart`, `ScatterChart`, `RadarChart`, `Gauge`, `Sparkline` |
| Composite | `FileChooser`, `ColorChooser`, `ColorPicker`, `FontChooser`, `Calendar`, `DatePicker`, `DateRangePicker`, `TimePicker`, `MarkdownView`, `MarkdownEditor`, `TerminalView` |
| Shell | `Window` with client-side decorations, `HeaderBar`, `Toolbar`, `Ribbon`, `Statusbar`, `StatusIcon`, `StatusArea`, `Wallpaper`, `Thumbnail` |

Every one of them, live: the [gallery](https://go-widgets.github.io/gallery/).

## Ribbon

`Ribbon` is an Office-style command ribbon: tabs of labelled groups holding
large buttons (icon above the text), small ones (icon beside it), toggles, split
buttons, drop-downs and a small gallery. It follows the Windows ribbon
guidelines: as the window narrows each group steps down by priority — full,
compact, icons only, then one pop-up button — so the ribbon shows the largest
combination that fits; it collapses to its tabs (Ctrl+F1, or the chevron) and a
tab then peeks at its body; Alt shows keytips; tooltips carry the shortcut and a
description. Tab and toolbar navigation follow the WAI-ARIA patterns, and its
boundaries and focus ring keep 3:1 contrast in light and dark themes.

Every control is a view on an `Action` — the one command a menu, a palette and
the ribbon share — and `NewCommandAction` makes one from a view-model
`mvvm.Command`, so its `CanExecute` greys the control. The active tab, the
collapsed state, a toggle and a drop-down's choice are `mvvm.Observable`s. Put
the ribbon under a `PopoverHost` and its menus draw over the page with no host
code.

## Splitter

`Splitter` lays N panes side by side (or one above the other, with
`vertical` true) and puts a draggable, focusable divider between each
neighbouring pair, after the WAI-ARIA window-splitter pattern. A pane is
either **fixed** — its `Size`, in logical pixels, is an `mvvm.Observable[int]`
the divider writes — or **flexible**, sharing what the fixed panes leave by its
`Weight`, also an `Observable`. Each pane has a `Min` and a `Max`. A
`Collapsible` fixed pane closes (size 0) when the divider is pulled below half
its `Min`, or on a double-click or Enter, and opens again the same way. A pane's
`Visible` observable switches it and its divider off. Because the sizes live in
observables, a view-model can bind, persist and restore them; `OnResized` is
called once per finished gesture with the panes the person changed, never for a
layout that merely followed a window resize.

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

A divider takes the focus: the arrow keys (Up and Down when vertical) move it
by `Step` logical pixels, four times that with Shift; Home and End take the pane
before it to its smallest and largest; Enter collapses or restores the fixed
pane it controls; Tab moves between dividers. Each divider is a
`RoleSeparator` with `aria-valuemin`, `aria-valuemax` and `aria-valuenow`, named
"Resize" and the pane's `Name`. `Cursor(x, y)` returns `"col-resize"` or
`"row-resize"` over a divider, and `SplitterColors(theme)` gives the divider
line and accent, which keep 3:1 contrast against the background and surface in
both default themes, as the focus ring does.

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
