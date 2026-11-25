---
title: "原生窗口"
linkTitle: "原生窗口"
weight: 10
description: "go-widgets/window：覆盖 X11、Wayland、GTK4、Cocoa、Win32、Android、wasmbox 和浏览器标签页的统一 Open/Run API，全部为 CGO_ENABLED=0。"
tags: [surfaces, window, x11, wayland, macos, windows]
---

[`go-widgets/window`](https://github.com/go-widgets/window) 打开一个真实的窗口并在其中运行控件树：
布局、绘制、呈现、把输入转换为 `toolkit.Event`，如此循环，直到窗口关闭。

```go
w, err := window.Open(window.Config{Title: "Demo"})
if err != nil {
	return err
}
defer w.Close()
return w.Run(root) // root is any toolkit.Widget
```

`Config` 还接受以**逻辑点**为单位的初始大小（零表示请求一个可读的默认值）、`WM_CLASS` 的实例和类、
一个 `Theme`，以及一个 `RenderScale`。

## Open 如何选择 {#how-open-chooses}

| 平台和环境 | 后端 | 如何到达屏幕 |
|---|---|---|
| Linux，设置了 `$GO_WIDGETS_GTK` | GTK4 | 由 GTK 拥有窗口；帧缓冲区是一个 `GtkPicture`，原生控件是位于其上的真正 GTK 控件。需要 libgtk-4 运行时。 |
| Linux，设置了 `$WAYLAND_DISPLAY` | Wayland | 通过合成器的 unix 套接字使用 xdg-shell |
| Linux，其他情况下使用 `$DISPLAY` | X11 | 通过 unix 套接字使用 X11 核心协议，并配合 MIT-SHM |
| macOS | Cocoa | 通过 [go-macos/objc](https://github.com/go-macos/objc)（purego）使用 `NSWindow` 和 `NSView` |
| Windows | Win32 | 通过 user32 和 gdi32 系统调用创建顶层 `HWND`，使用 `StretchDIBits` |
| Android，设置了 `$GW_ANDROID_SOCKET` | Android 宿主 | 与 [Java 宿主]({{< relref "/surfaces/android.md" >}})之间的成帧协议，像素放在共享的 memfd 中 |
| Android，其他情况 | Wayland 或 X11 | Termux 下的 shell 仍然有显示服务器可以连接 |
| `js/wasm`，在 wasmdesk 内 | wasmbox | [wasmdesk 合成器]({{< relref "/surfaces/browser.md#a-window-on-the-wasmdesk-desktop" >}})的客户端 |
| `js/wasm`，普通页面 | 浏览器标签页 | 通过 `webcanvas` 使用页面的 `<canvas>`（`Config.Canvas`，默认为 `"screen"`）：无需合成器，无需 `SharedArrayBuffer`，无需跨源隔离 |
| 其他任何情况 | — | `window.ErrUnsupported`，因此交叉构建仍能编译 |

每个后端都是 `CGO_ENABLED=0`。X11 后端是在套接字之上从零写成的核心协议——没有 Xlib，没有 XCB——
支持两种字节序、Xauthority cookie、keysym 映射、按服务器请求大小分块的 `PutImage`，
以及 MIT-SHM 快速路径，共享段通过 `SCM_RIGHTS` 传递。Windows 通过进程自己的 DLL，
使用 `syscall.NewLazyDLL` 和 `syscall.NewCallback` 窗口过程来访问；macOS 则通过 Objective-C 运行时和 purego。

每个后端中与平台无关的部分——事件映射、坐标运算、像素打包、受损矩形——都放在一个覆盖率为 100% 的编解码器中，
在每个操作系统上都是如此。很薄的平台胶水代码则做实时验证：参见[状态]({{< relref "/status.md" >}})。

## 只绘制变化的部分 {#only-what-changed}

实现了 `DamageRenderer` 的根（如 `toolkit/scene.HostRoot`）会报告它重绘的矩形，
每个后端都只呈现这些矩形：X11 通过 MIT-SHM 的 `ShmPutImage`，Wayland 通过 `wl_shm` 的 damage，GTK4 通过部分纹理更新，
以及 Cocoa、Win32、Android、wasmbox 和浏览器标签页。第一帧、窗口缩放和 X11 的 `Expose` 仍然呈现整个绘制表面。

在 GTK4 上这需要 libgtk-4 4.16 或更新版本：帧成为带有 `set_update_texture` 和 `set_update_region` 的
`GdkMemoryTextureBuilder` 纹理，因此 GTK 只重新渲染并上传受损区域。较旧的运行时保留完整上传，在运行时选择。
帧缓冲位于纹理共享的 GLib 内存中，因此一次呈现在 CPU 一侧不复制任何字节（以前每次都复制整帧，200x160 时为 128000 字节）。GTK4 窗口会跟随大小调整：图片填满窗口，帧缓冲被重新分配，以新尺寸整帧呈现一次，之后恢复按受损区域呈现。

## 在 GPU 上绘制 {#on-the-gpu}

在 macOS 上，`Config{GPU: true}` 让帧在 GPU 上绘制。
- **方式：** 控件通过 [gpupainter](https://github.com/go-widgets/gpupainter) 记录到
  [go-gfx/gpu2d](https://github.com/go-gfx/gpu2d) 列表中，由它的 Metal
  渲染器绘制帧并呈现在 `CAMetalLayer` 中，帧不会回到 CPU。
- **精确性：** 像素与 CPU 路径逐字节相同。
- **开销：** 每帧占用的处理器时间在 960 × 640 下从 1.58 ms 降到 0.59 ms，在
  1920 × 1200 下从 5.96 ms 降到 0.99 ms（Apple M4 Max）。
- **回退：** 带半透明材质的窗口仍走 CPU 路径，Metal 无法启动的机器也一样，二者都不算
  错误。Linux 见下文；其他后端忽略此选项。

在 Linux（X11 和 Wayland）上，`Config{GPU: true}` 使用 Vulkan 绘制帧。
- **方式：** 同样的记录，由 go-gfx/gpu2d 的 Vulkan 渲染器绘制。它的像素被复制到窗口的帧缓冲区，
  并照旧呈现：X11 上用 MIT-SHM（或 `PutImage`），Wayland 上用 `wl_shm`。帧会经过 CPU 一次，
  目前还没有通过 dma-buf 与显示服务器共享。
- **精确性：** 在软件 Vulkan（lavapipe）上，像素与 CPU 路径逐字节相同，并已在两种协议上实机证明：
  从 X 服务器读回的窗口，以及窗口提交的 `wl_shm` 缓冲区，都与 CPU 路径的一致。
  go-gfx/gpu2d 已在硬件 GPU（AMD Radeon PRO W7700，RADV）上测量其 Vulkan 渲染器：
  在 1080p 下 `Draw` 比单核 CPU 的 `Rasterize` 快 52 倍；窗口本身（包括复制到帧缓冲区）
  尚未在硬件上测量。
- **回退：** 当 Vulkan 不可用，或某一帧被拒绝（对设备来说太大、设备丢失）时，窗口改用 CPU 绘制并继续运行。
  原因由后端的 `GPUError()` 给出，此后的每一帧都是 CPU 帧。
- **开销（仅软件）：** lavapipe 是 CPU 光栅器，所以 GPU 路径占用的 CPU 时间约为 CPU 路径的十倍：
  每帧在 960 × 640 下为 25.8 ms 而不是 2.48 ms，在 1920 × 1200 下为 89.7 ms 而不是 8.46 ms
  （X11，Apple M4 Max 上 arm64 容器中 16 线程的 lavapipe）。在真实 GPU 上是否划算尚未测量。

## HiDPI {#hidpi}

默认是每个逻辑点对应一个帧缓冲像素：界面以可读的大小布局和绘制，由合成器对其放大。
在 Windows 上，窗口声明按显示器的 DPI 感知，由操作系统把逻辑帧缩放到物理客户区。
`RenderScale` 请求一个具有面板真实分辨率的帧缓冲区，这只对以设备像素进行布局的根才是正确的。

## 屏幕，以及显示和隐藏 {#screens-and-showing-and-hiding}

`window.Screens()` 列出已连接的显示器，主显示器排在最前，以逻辑点为单位，并排除桌面的面板，
并且可以在 `Open` 之前调用。屏幕的名称取自面板自身（`"DELL U2720Q"`，来自其 EDID），
当面板没有给出名称时回退为连接器名（`"HDMI-1"`）——当两块已连接的面板给出相同的名称时也是如此，
因为无法区分二者的名称不算名称。

`window.Show`、`Hide` 和 `Raise` 可以让一个已打开的窗口离开屏幕而不关闭它，并再把它带回来——
这正是[托盘应用]({{< relref "/surfaces/application.md#the-tray" >}})所需要的。

| 后端 | Show | Hide | Raise |
|---|---|---|---|
| X11 | `MapWindow` | ICCCM withdraw，因此它会离开任务栏 | map、raise、`_NET_ACTIVE_WINDOW` |
| Wayland | 再次 commit | 空缓冲区并 commit | `ErrNotSupported`：xdg-shell 没有这样的请求 |
| macOS | `orderFront:` | `orderOut:` | 激活应用，`makeKeyAndOrderFront:` |
| Windows | `SW_SHOWNA` | `SW_HIDE` | `SW_RESTORE` 和 `SetForegroundWindow` |
| GTK4 | `gtk_widget_set_visible(true)` | `gtk_widget_set_visible(false)` | `gtk_window_present` |
| Android、wasmbox、浏览器标签页 | `ErrNotSupported` | `ErrNotSupported` | `ErrNotSupported` |

焦点由平台决定是否授予：在有防抢焦点机制的情况下，`Raise` 可能只能把窗口标记为需要关注，并且无法得知结果
（GTK4 下的 Wayland 合成器正是如此）。

`Config.HideOnClose` 由 X11、Wayland、GTK4、macOS 和 Windows 支持：窗口自带的关闭控件会隐藏窗口而不是
关闭它，`Show` 或 `Raise` 可以把它带回来。

## 原生控件上的上下文菜单和图片 {#context-menu-and-picture-on-native-controls}

`NativeControl.Menu` 以及 `Image` / `ImageOnly` 由每个原生后端应用，而不只是 Cocoa。在 GTK4 上，
菜单是由次要点击打开的弹出框，图片位于按钮内，可以单独显示，也可以与标题并排。在 Win32 上，菜单是在
`WM_CONTEXTMENU` 时用 `TrackPopupMenu` 显示的真实 `HMENU`，图片是按钮上的位图。已禁用的条目或分隔线
不可选择。由于描述符每帧都会重建，两者只在发生变化时才重新应用。

PNG 有上限：最大 1 MiB，每边 1024 像素，在分配任何像素之前先根据文件头检查；无法读取的图片会让按钮保持
原样。`painter.IconSize` 和 `painter.IconBGRAOver` 会把图片缩放到按钮减去 4 像素边距的范围内，绝不放大。在
Windows 上，与标题并排的图片需要启用视觉样式的通用控件；没有清单（manifest）时，按钮只显示图片。
Windows 的编辑框保留它自己的系统菜单。

## Windows 上的原生控件 {#native-controls-on-windows}

`toolkit.NativeControl` 会成为 Win32 窗口的真实子窗口，每个 `Key` 一个，并在每一帧进行协调：
只有当值与控件最后一次报告的不同时才会写入控件，因此不会与用户自己的编辑相冲突。除文本、按钮、
滑块、进度条和步进器外，还托管另外五种：

| 类型 | 窗口类 | 值 |
|---|---|---|
| `NativeList` | `LISTBOX` | `Number` 是所选行，无选择时为 `-1` |
| `NativeLink` | `SysLink` | 点击或按回车时调用 `OnActivate` |
| `NativeDate` | `SysDateTimePick32` | `Text` 为 `YYYY-MM-DD` |
| `NativeColor` | `GoWidgetsColorWell` | `Text` 为 `#RRGGBB`；点击会打开系统颜色对话框 |
| `NativeSegmented` | `SysTabControl32` | `Text` 是所选分段的标题 |

列表的行和分段在创建控件时就已固定；要更改它们，请更改 `Key`。链接自己从不打开任何东西：
它只调用 `OnActivate`，因此跟随哪些地址（仅 http 和 https）仍由 `application.OpenURL` 决定。
`SysLink` 只存在于带视觉样式的通用控件中，没有清单的程序无法获得它；窗口会在自带的清单下创建链接，
若不可用，则显示一个以相同方式激活的扁平按钮。

## 原生表格、树和标签页

另有三种类型承载结构化数据，用 `NewNativeTable(columns, rows, selected)`、
`NewNativeTree(nodes, selectedPath)` 和 `NewNativeTabs(titles, selected)` 创建；
为此 `NativeControl` 新增了 `Columns`、`Rows`、`Tree` 和 `Revision`，
之后可用 `Native.SetRows` 替换表格的行。

| 类型 | Cocoa | GTK4 | Win32 | 选择 |
|---|---|---|---|---|
| `NativeTable` | `NSTableView` | `GtkColumnView` | `SysListView32`（`LVS_REPORT`、`LVS_OWNERDATA`） | `Number` / `OnNumber` 是行，`-1` 表示无 |
| `NativeTree` | `NSOutlineView` | 基于 `GtkTreeListModel` 的 `GtkListView` | `SysTreeView32` | `Text` / `OnText` 是形如 `2.0.1` 的路径 |
| `NativeTabs` | `NSTabView` | `GtkNotebook` | `SysTabControl32` | `Number` / `OnNumber` 是标签页，`-1` 表示无 |

排序尚未实现：行按给定顺序显示。标签页控件只是标签条；标签页下的内容仍由工具包照常布局。
绘制的后备实现（`Table`、带 `HideRoot` 的 `TreeView`、`TabBar`）在没有承载原生控件的地方绘制相同的数据。
承载这些类的窗口后端（由 `go-widgets/window` 承载）将在下一个窗口版本中提供；
Cocoa 一侧需要 `go-macos/appkit` 新增的 `GridView`、`OutlineView` 和 `TabView` 类型。

数据有上限：最多 `MaxNativeRows`（100000）行、`MaxNativeColumns`（64）列、
`MaxNativeTreeDepth`（32）层、`MaxNativeTreeNodes`（100000）个节点和 `MaxNativeTabs`（64）个标签页；
超出的值会被拒绝，而不是被截断。

协调只处理变化。帧以 `Native.Revision()` 为键：修订号不变时完全不遍历行，只有它变化时才把行推给控件。
在 Cocoa 上，10,000 行的一帧在修订号不变时耗时 160 us，行变化时为 687 us（含构建这 10,000 行）；
由于数据源是虚拟的，`SetGrid` 本身只需 1.3 us。在 Win32 上，修订号不变时耗时 0.26 ns 且无分配，
遍历 10,000 行则需 43.8 us（Cocoa 在 M4 Max 上单核测得）。
