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
除 GTK4 之外的每个后端都只呈现这些矩形：X11 通过 MIT-SHM 的 `ShmPutImage`，Wayland 通过 `wl_shm` 的 damage，
以及 Cocoa、Win32、Android、wasmbox 和浏览器标签页。第一帧、窗口缩放和 X11 的 `Expose` 仍然呈现整个绘制表面。

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
| GTK、Android、wasmbox、浏览器标签页 | `ErrNotSupported` | `ErrNotSupported` | `ErrNotSupported` |

焦点由平台决定是否授予：在有防抢焦点机制的情况下，`Raise` 可能只能把窗口标记为需要关注，并且无法得知结果。
