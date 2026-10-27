---
title: "在哪里绘制"
linkTitle: "绘制表面"
weight: 30
description: "go-widgets 控件树可以显示在其上的每一种绘制表面，以及由什么来选择它们。"
tags: [surfaces, back-ends]
---

| 绘制表面 | 模块 | 绘制器 |
|---|---|---|
| X11、Wayland、macOS、Windows 上的[原生窗口]({{< relref "/surfaces/native-window.md" >}}) | `window` | 像素 |
| 带托盘图标和系统外观的[应用程序]({{< relref "/surfaces/application.md" >}}) | `application`、`tray` | 像素 |
| [浏览器标签页]({{< relref "/surfaces/browser.md" >}})：一个普通的 `<canvas>` | `webcanvas` 或 `window` | 像素 |
| [浏览器桌面]({{< relref "/surfaces/browser.md#a-window-on-the-wasmdesk-desktop" >}})：wasmdesk 合成器的一个窗口 | `window`（wasmbox） | 像素 |
| [终端]({{< relref "/surfaces/terminal.md" >}}) | `tui` | 单元格 |
| [Android APK]({{< relref "/surfaces/android.md" >}}) | `android`、`window` | 像素 |
| [图片]({{< relref "/surfaces/snapshots.md" >}})：SVG 或 PNG | `svg` | 像素 |

控件树在它们之间保持不变。变化的是谁拥有缓冲区、谁传递输入——显示服务器、浏览器、终端、Java 宿主——
而每个后端都会把该输入转换成同样的 `toolkit.Event`。

## 由现有环境决定 {#chosen-by-what-is-there}

`window.Open` 不接受后端参数。它会查看环境——构建时的目标平台、`$WAYLAND_DISPLAY`、`$DISPLAY`、
Android 宿主导出的套接字——然后选出能工作的那一个。为 Linux 构建的二进制文件在 Wayland 会话下打开 Wayland 窗口，
在 X 下打开 X11 窗口；同一个 Android 二进制文件在 APK 内连接它的宿主，在 Termux 下则打开普通的 Linux 窗口。
参见[`Open` 如何选择]({{< relref "/surfaces/native-window.md#how-open-chooses" >}})。
