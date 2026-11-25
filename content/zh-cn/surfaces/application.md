---
title: "应用程序及其托盘"
linkTitle: "应用程序与托盘"
weight: 20
description: "go-widgets/application 负责窗口之上的生命周期；go-widgets/tray 在菜单栏或通知区域放置一个图标和一个菜单。"
tags: [surfaces, application, tray]
---

## 应用程序 {#the-application}

[`go-widgets/application`](https://github.com/go-widgets/application) 是位于
[`window`]({{< relref "/surfaces/native-window.md" >}}) 之上的一层：它打开窗口、驱动窗口的循环、
在第一帧出现之后挂上托盘图标、把实时的系统外观（深色或浅色、强调色、系统字体）传给应用、
发布无障碍树，并在第一帧显示到屏幕上之后调用 `onReady`。

```go
spec := application.Spec{
	Name:       "News Reader",
	Identifier: "com.example.reader",
	Version:    "1.0.0",
	Tray: func() *tray.Menu {
		return tray.NewMenu().Add(
			tray.Item("Refresh", refresh),
			tray.Item("Quit", quit),
		)
	},
}
cfg := application.Config{Title: "News Reader", Width: 1200, Height: 800}
err := application.Run(spec, cfg, handler, func() { log.Println("on screen") })
```

`handler` 实现 `Handler`——`Frame`、`Resize`、鼠标、滚动和按键事件——并且可以实现任何可选能力：
外观、快捷键、带修饰键的点击和按键、次要点击、上下文菜单、剪贴板、无障碍、原生控件。
循环从不包装 handler；它对每项能力做类型断言，并遵从它找到的那些。有一个测试会遍历该包中的这些断言，
如果包文档没有点出每一项，就会失败。

无法挂上的托盘是 `Run` 的错误，而不是静默：`errors.Is(err, tray.ErrNoBackend)` 会说明原因。
无论托盘如何都想要窗口的应用，把 `Spec.Tray` 留空，自己运行一个托盘。

`Run` 在循环返回时关闭它的窗口，因此托盘应用的“打开”可以再次调用 `Run` 来得到一个新窗口。
对于不是窗口的宿主——桌面外壳、浏览器标签页——`Bind` 和 `BindScaled` 返回一个与同一 handler 相连的
`*toolkit.Surface`。

## 绘制控件的无障碍 {#painted-accessibility}

自行绘制[工具包]({{< relref "/rendering/toolkit.md#accessibility" >}})控件的处理器，除非实现 `Accessible`，否则在平台的无障碍层看来只是一个不透明的矩形。`application.A11yElementsOf(root)` 根据控件树生成这份描述（每个控件的角色、标签、值和边界，最多 `MaxA11yElements` = 1000 个）：

```go
func (v *View) A11yElements() []application.A11yElement {
	return application.A11yElementsOf(v.root)
}
```

在 macOS 上，绘制出的按钮随后会以带标签的 `AXButton` 出现在 AX 树中，对其执行 `AXPress` 即相当于点击；处理器嵌入的原生控件仍留在树中它们旁边。Windows（UI Automation）和 Linux（AT-SPI）后端使用同一份描述。元素的禁用状态尚未发布，Windows 桥接也拒绝元素矩形。

## 托盘 {#the-tray}

托盘图标是操作系统集成，而不是绘制出来的控件，因此
[`go-widgets/tray`](https://github.com/go-widgets/tray) 驱动每个平台自己的 API，同样是 `CGO_ENABLED=0`：

| 平台 | API | 通过 |
|---|---|---|
| macOS | `NSStatusItem`、`NSMenu` | purego 和 Objective-C 运行时 |
| Windows | `Shell_NotifyIcon`、`TrackPopupMenu` | `golang.org/x/sys/windows` |
| Linux | `StatusNotifierItem`、`com.canonical.dbusmenu` | DBus，用 Go 实现 |

```go
menu := tray.NewMenu().Add(
	tray.Item("Open", open),
	tray.Checkbox("Notifications", true, setNotify),
	tray.SubMenu("Recent", tray.NewMenu().Add(tray.Item("file.txt", nil))),
	tray.Separator(),
	tray.Item("Quit", func() { t.Quit() }),
)
t := tray.New(iconPNG).SetTooltip("My App").SetMenu(menu)
t.Run() // blocks on the platform's loop until Quit
```

已经在运行窗口循环的程序无法把主线程交给 `Run`：`Attach` 会显示同一个图标并立即返回，`Quit` 则把它撤下。

原生后端默认开启；没有需要记住的构建标签。在没有任何后端的平台上，`Run` 返回 `ErrNoBackend`——
这就是“这里没有托盘”和“托盘悄悄什么也不做”之间的区别。测试或无界面服务传入
`WithBackend(tray.NewHeadless())`。

### 通知

`(*Tray).Notify(tray.Notification{Title, Body, OnClick})` 发出一条原生系统通知：macOS 上是 UNUserNotificationCenter，Linux 上是 `org.freedesktop.Notifications`，Windows 上是 Shell_NotifyIcon 气泡。标题最多 63 个 rune，正文最多 255 个（`MaxNotificationTitle`、`MaxNotificationBody`）；标题为空则返回 `ErrEmptyTitle`。这些错误会指明原因：`ErrNoNotifier`（没有后端）、`ErrNotRunning`、`ErrNotBundled`（macOS 只为应用包投递）以及 `ErrNotAuthorized`（用户拒绝）。`tray.NewHeadless()` 会把发出的内容记录在 `Headless.Notified` 中，供测试使用。

## 打开链接

`application.OpenURL(rawURL)` 在用户的浏览器中打开一个 `http` 或 `https` URL：macOS 上用 NSWorkspace，Windows 上用 ShellExecuteW，Linux 上先用 OpenURI 门户，再用 `xdg-open`。URL 需要有主机，端口（如果有）在 1-65535 之间，且不超过 8 KiB；其他情况一律以 `ErrURLScheme` 或 `ErrURLInvalid` 拒绝，没有启动器的平台返回 `ErrOpenURLUnsupported`。
