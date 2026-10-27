---
title: "工具包"
linkTitle: "工具包"
weight: 20
description: "通过绘制器（painter）接缝绘制的 160 种控件类型：主题、布局、文本、输入、无障碍。"
tags: [rendering, toolkit, widgets, theme, accessibility]
---

[`go-widgets/toolkit`](https://github.com/go-widgets/toolkit) 是控件（widget）集。
每个控件都实现同一个接口——`Bounds`、`SetBounds`、
`Draw(p painter.Painter, theme *Theme)`、`HitTest`、`OnEvent`——因此宿主对待按钮、
表格和事件日历一视同仁。仅使用标准库，`CGO_ENABLED=0`，语句覆盖率 100%。

在 v0.329.0 中，它共有 **160 种导出的控件类型**，控件代码约 **69,000 行**。这两个数字
是在该标签处实测得到的，并非沿用旧值：工具包的 README 给出了相应命令，应当重新运行，
而不是照抄这些数字。

## 控件家族 {#the-families}

| 家族 | 控件 |
|---|---|
| 操作 | `Button`、`ToggleButton`、`CheckButton`、`RadioButton` + `RadioGroup`、`Switch`、`SplitButton`、`IconButton`、`CycleButton`、`Chip`、`SegmentedBar` |
| 输入 | `Entry`、`TextView`（选择、输入法预览、语法着色区间、行号）、`SpinButton`、`Scale`、`RangeSlider`、`SearchEntry`、`TagField`、`ComboBox`、带 `Validate`/`Rule` 的 `FormField` |
| 选择 | `ListBox`、`TreeView`、`DropDown` |
| 容器 | 带可替换 `Layout` 的 `Container`（`FitLayout`、`BoxLayout`、`BorderLayout`、`CardLayout`、`FlowLayout`）；`HBox`、`VBox`、`Grid`、`Frame`、`Dock`、`Border`、`Stack`、`Overlay`、`Paned`、`Splitter`、`Expander`、`Accordion`；声明式的 `Node` 构建器 |
| 导航 | `Menu`、`MenuBar`（支持 `Alt`+字母）、`ContextMenu`、`Popover`、`CommandPalette`、`Dialog`、`MessageDialog`、`Wizard`、`Notebook`、`ViewSwitcher`、`Carousel` |
| 反馈 | `ProgressBar`、`ProgressCircle`、`LevelBar`、`Spinner`、`Tooltip`、`Notification`、`Toast`、`Banner`、`Alert`、`Badge`、`Skeleton` |
| 数据 | `Table`（单元格编辑、冻结列、分组行、聚合、行展开器）、`TreeTable`、`PropertyGrid`、`Kanban`、`Gantt`、`Agenda`（周、月、季度、年；多个日历） |
| 图表 | `LineChart`、`BarChart`、`PieChart`、`AreaChart`、`ScatterChart`、`RadarChart`、`Gauge`、`Sparkline` |
| 复合 | `FileChooser`、`ColorChooser`、`ColorPicker`、`FontChooser`、`Calendar`、`DatePicker`、`DateRangePicker`、`TimePicker`、`MarkdownView`、`MarkdownEditor`、`TerminalView` |
| 外壳 | 带客户端装饰的 `Window`、`HeaderBar`、`Toolbar`、`Ribbon`、`Statusbar`、`StatusIcon`、`StatusArea`、`Wallpaper`、`Thumbnail` |

每一个控件的实时演示：[画廊（gallery）](https://go-widgets.github.io/gallery/)。

## 功能区 {#ribbon}

`Ribbon` 是 Office 风格的命令功能区（ribbon）：由若干选项卡组成，每个选项卡含带标签的分组，
分组中有大按钮（图标在文字上方）、小按钮（图标在文字旁）、切换按钮、拆分按钮、下拉框和一个小型图库。
它遵循 Windows 功能区指南：窗口变窄时，每个分组按优先级逐级降级——完整、紧凑、仅图标，
最后是一个弹出按钮——因此功能区总是显示能容纳的最大组合；它可以折叠为仅剩选项卡
（Ctrl+F1，或点击箭头），此时选项卡会临时展开其内容；按下 Alt 显示快捷键提示；
工具提示带有快捷键和说明。选项卡和工具栏导航遵循 WAI-ARIA 模式，其边界和焦点环在浅色和深色主题下都保持 3:1 的对比度。

每个控件都是对某个 `Action` 的视图——菜单、命令面板和功能区共享的同一条命令——
`NewCommandAction` 可由视图模型的 `mvvm.Command` 创建一个 `Action`，
因此其 `CanExecute` 会使控件变灰。当前选项卡、折叠状态、切换按钮以及下拉框的选择都是
`mvvm.Observable`。把功能区放在 `PopoverHost` 之下，它的菜单就会绘制在页面之上，无需任何宿主代码。

`RibbonGrid` 是一种控件类型，它的弹出层是由 `GridCols` 乘 `GridRows` 个单元格组成的网格
（为零时为 8 乘 6，最大 16 乘 16），就像文字处理软件的“表格”按钮那样。指针或方向键会从左上角起
点亮到当前所指单元格为止的所有单元格，并以标题显示“3 × 2”；点击或按 Enter 即确定并调用
`OnGrid(cols, rows)`；Home 和 End 跳转，Escape 关闭。它在浅色和深色下都能绘制，
并向辅助技术报告为一个按钮，网格打开时处于展开状态。

```go
&toolkit.RibbonControl{Kind: toolkit.RibbonGrid, Label: "Table",
	GridCols: 8, GridRows: 8,
	OnGrid: insertTable}
```

## 分隔条 {#splitter}

`Splitter` 把 N 个窗格并排（或在 `vertical` 为 true 时上下）排列，并在每对相邻窗格之间放置
一个可拖动、可聚焦的分隔条，遵循 WAI-ARIA 窗口分隔条模式。窗格要么是**固定**的——其 `Size`
以逻辑像素为单位，是由分隔条写入的 `mvvm.Observable[int]`——要么是**弹性**的，按其 `Weight`
（同样是一个 `Observable`）分享固定窗格剩下的空间。每个窗格都有 `Min` 和 `Max`。
`Collapsible` 的固定窗格在分隔条被拖到小于其 `Min` 的一半时、或双击或按 Enter 时关闭（大小为 0），
并以同样的方式重新打开。窗格的 `Visible` 可观察对象可以同时开关该窗格及其分隔条。
由于尺寸存放在可观察对象中，视图模型可以绑定、持久化和恢复它们；`OnResized` 在每次完成的手势后
只调用一次，参数是用户改变过的窗格，对于仅仅跟随窗口缩放而变化的布局则从不调用。

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

分隔条可以获得焦点：方向键（垂直时为上和下）按 `Step` 逻辑像素移动它，按住 Shift 时为四倍；
Home 和 End 把它前面的窗格调到最小和最大；Enter 折叠或恢复它所控制的固定窗格；
Tab 在分隔条之间切换。每个分隔条都是带有 `aria-valuemin`、`aria-valuemax` 和 `aria-valuenow` 的
`RoleSeparator`，名称为“Resize”加窗格的 `Name`。`Cursor(x, y)` 在分隔条上方返回 `"col-resize"`
或 `"row-resize"`，`SplitterColors(theme)` 给出分隔线和强调色，二者与背景和表面的对比度在两个默认主题下都保持
3:1，焦点环亦然。

## 主题 {#themes}

一个 `Theme` 值会在整棵树中层层生效：在根上改变一种颜色，每个控件都会随之重绘。
`DefaultLight()` 和 `DefaultDark()` 是内置的，`LoadGTKTheme(css)` 会把任何 GTK 3 或 libadwaita
主题（Adwaita、WhiteSur、Solarized）的 `@define-color` 块读入一个 `Theme`。

## 文本 {#text}

编译进去的默认字体是 5×7 点阵字体，因此像素测试能保持其几何形状。一次调用即可把整个界面切换为
抗锯齿的、经过文本整形（shaping）的文本：

```go
toolkit.UseOpenTypeText() // Atkinson Hyperlegible, bundled; js/wasm-safe
```

该字体来自 [go-opentype/fonts](https://github.com/go-opentype/fonts)，由
[go-opentype](https://github.com/go-opentype/opentype) 用 Go 光栅化，不使用 C 字体库，也不查找系统字体。
`NewTrueTypeFont(ttf, px)` 加载另一种字体；`NewFallbackFont` 把若干字体串成回退链，
例如添加 CJK 或阿拉伯文字体；控件自己的 `Font` 字段只对该控件覆盖全局字体。

## 输入 {#input}

鼠标、键盘、输入法组合、触摸（`EventTouchStart`/`Move`/`End`，并有 `GestureRecognizer` 识别轻点、
长按和滑动）、拖放（`DragSource`、`DropTarget`）。`ListBox`、`Table` 和 `TreeView` 中的多行选择
可用鼠标和键盘完成，并且除 Ctrl 之外也认 ⌘，因为在 macOS 上 Ctrl 加点击是次要点击。

## 二进制体积：程序只链接它用到的部分 {#binary-size}

工具包是一个很大的包，这对链接器来说没有问题：Go 的死代码消除只保留从 `main.main` **以及每个被导入包的初始化任务**
可达的部分。程序从未碰到的控件会被丢弃。不会被丢弃的，是包级初始化器或被导入包的 `init` 所能到达的一切，
因此自 v0.345.0 起，工具包让自己的初始化器不做任何工作，并避免导入其 `init` 会拖入代码的包：

- `Material` 和 `Thumbnail` 使用不含编解码器的
  [`github.com/go-images/images/ops`](https://github.com/go-images/images)，
  而不是 `github.com/go-images/images`——后者的解码器会在 `init` 中自行注册（约 1.7 MB 的 WebAssembly 代码）。
- `CollabText`、`NewCollabText`、`MetaName` 和 `MetaColor` 移到了
  **`github.com/go-widgets/toolkit/collabui`**：它们需要一个运行中的 `go-crdt/collab` 会话，
  其传输层会引入 `net/http` 和 `crypto/tls`。
  `CollabSheet` 和 `IsoCRDTDocument` 只需要 `go-crdt/crdt`，仍留在 `toolkit` 中。
- 电子邮件 `Rule` 的正则表达式、内置的 iso 图标注册表以及 `RichEditorToolbar` 的按钮表，
  都在首次使用时才构建，而不是由初始化器构建。

⛔ **这次移动是破坏性变更，没有弃用的转发层。** 把上述四个名称的导入改为 `collabui`；其他一概不变。

`GOOS=js GOARCH=wasm`、使用 `-ldflags="-s -w"` 时的体积，原始大小和 gzip -9 后的大小：

| 程序 | 之前（原始 / gzip） | 之后（原始 / gzip） |
|---|---|---|
| 一个 `Label`（v0.345.0） | 10.49 / 2.88 MB | 7.48 / 2.13 MB |
| 一个 `Label`（v0.345.1） | 10.49 / 2.88 MB | 5.66 / 1.61 MB |
| go-wasmcontainers/dashboard | 20.42 / 5.17 MB | 17.72 / 4.58 MB |
| the relay flow map | 22.62 / 5.44 MB | 19.95 / 4.85 MB |

剩下的主要是运行时和文本渲染器所需的 OpenType 栈。要查看是什么让某个符号留在你自己的程序里，
请使用 `-ldflags=-dumpdep` 构建，并沿着第一个父节点链从该符号追溯到 `main.main` 或 `go:main.inittasks`。

v0.345.0 还新增了 `ColorHex` 和 `ParseColorHex`，用于把颜色写成十六进制并读回。

## 无障碍 {#accessibility}

自己绘制像素的工具包，如果不说明画了什么，屏幕阅读器就无从读取。控件实现 `Accessible`——角色、名称、值——
并由 `CollectA11y` 收集起来交给宿主发布：在 Linux 上通过 [`window`]({{< relref "/surfaces/native-window.md" >}})
发布 AT-SPI，在 [Android]({{< relref "/surfaces/android.md#accessibility" >}}) 上则是一个虚拟视图层级。
