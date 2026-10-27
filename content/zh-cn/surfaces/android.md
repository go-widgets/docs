---
title: "Android"
linkTitle: "Android"
weight: 50
description: "go-widgets/android：一个可安装的 APK，其整个界面由 CGO_ENABLED=0 的 Go 进程完成布局和绘制。"
tags: [surfaces, android]
---

Android 是唯一一个不使用 cgo 的进程无法拥有窗口的平台：通往可绘制表面的每条路径都要经过 JNI，
而 JNI 需要 cgo。这里没有像 X11 和 Wayland 那样可以改用的线路协议。

因此 [`go-widgets/android`](https://github.com/go-widgets/android) 由两个进程组成：

| | |
|---|---|
| **Java 宿主**（`host/`） | 拥有 `Activity`、`SurfaceView`、触摸、按键和生命周期。负责传送像素；对控件一无所知。 |
| **Go 应用**（`cmd/gwapp`） | 一个普通的 `CGO_ENABLED=0 GOOS=android` 可执行文件。负责布局、控件、主题、命中测试和焦点，与其他每个后端完全一致。 |

Java 宿主所处的位置，相当于 Linux 上的 X 服务器。像素通过应用创建、并经套接字交给宿主的 **memfd** 传递；
Go 一侧写入 RGBA_8888，这与 Android 的 ARGB_8888 逐字节相同，因此传送只是一次不带转换的复制，
而且只复制受损矩形。输入、边衬区（insets）、输入法文本和无障碍树通过抽象的 `LocalSocket` 上的成帧协议传递。

```go
c, err := android.Dial("my app", nil) // nil theme: toolkit.DefaultDark()
if errors.Is(err, android.ErrUnsupported) {
	return nil // not under a host
}
defer c.Close()
return c.Run(myWidgetTree())
```

`Client` 满足 `window` 的 `Backend`，因此应用在 Android 和桌面后端之间切换时，窗口之上的代码无需改动一行——
当宿主的套接字存在时，[`window.Open`]({{< relref "/surfaces/native-window.md#how-open-chooses" >}})
会自动选中它。

## 仅限 arm64 {#arm64-only}

`android/arm64` 是 Go 不使用 cgo 就能链接的唯一 Android 目标；`arm`、`amd64` 和 `386` 需要外部链接。
CI 对两半都做了断言——能构建的那个和不能构建的那三个——这样，哪天 Go 取消了这一限制，构建会告诉我们。

## 无障碍 {#accessibility}

屏幕阅读器会把 `SurfaceView` 看成一个不透明的矩形。宿主转而为它提供一个虚拟视图层级：
每个无障碍控件一个节点，带有 Android 据以选择播报内容的 `android.widget.*` 类、文本、边界以及一个激活操作。
这棵树是在有东西读取它时被**拉取**的，从不主动推送，因此没有连接无障碍服务的应用永远不会构建它。
激活会作为一次发生在元素中心的普通点击返回，走的是触摸所用的同一段代码。

## 触摸与动画 {#touch-and-animation}

每个触摸采样先作为触摸事件到达控件树，再作为鼠标事件到达：前者给感知手势的控件，
后者给众多只监听点击的控件。带动画的控件在帧循环上推进，帧循环在有东西开始动画时启动，在没有东西动画时停止。
