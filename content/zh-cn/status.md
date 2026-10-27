---
title: "状态"
weight: 60
description: "哪些已在真实显示器、真实设备或真实终端上得到验证；哪些只是通过了编译；以及 CI 尚未覆盖的部分。"
tags: [status]
---

这些页面已对照[模块]({{< relref "/modules.md" >}})中所列标签处的模块核对过。当某个模块的 README 与其代码不一致时，页面以代码为准。

## 已在真实环境验证 {#proven-live}

进程内测试的编解码器，证明的只是编解码器本身。以下这些是向平台本身发出过询问的地方：

| 项目 | 验证环境 |
|---|---|
| X11 窗口 | CI，在 Xvfb 下：呈现一幅已知图案，捕获并采样；用 `xdotool` 合成一次点击和一次按键，并断言所分发的 `toolkit.Event`；`Show`/`Hide`/`Raise` 从服务器回读 |
| Cocoa 窗口 | macOS CI 通道：打开并渲染一个窗口，采样像素，合成一次点击和一次按键；从另一个 goroutine 对真实的 `NSWindow` 调用 `Show`/`Hide`/`Raise`，并从 AppKit 回读 |
| Win32 窗口 | Windows 11 arm64 虚拟机：一个真实窗口渲染 `VBox`、`Label` 和 `Button`，注入三次点击，经由窗口过程把它的计数器从 0 推到 3 |
| Wayland 窗口 | 针对一个强制执行 xdg-shell 映射与取消映射规则的模拟合成器 |
| wasmbox 窗口 | 无头 Chromium 驱动真实的 wasmdesk 合成器：回读合成后的像素，一次真实点击被路由到控件 |
| Android | 一台 Android 15 arm64 设备：测量仅脏区域的块传送，测量 memfd 脏内存，并在设备上对无障碍树做基准测试 |
| macOS 托盘 | 一个真实的 macOS 会话：程序停止时图标离开菜单栏，启动时回来，并能打开它的菜单 |
| Linux 托盘 | CI 中针对真实会话总线：名称被占用，菜单响应 `GetLayout`，`Quit` 撤销该名称 |
| Windows 托盘 | Windows 11 虚拟机上的图标及其菜单 |
| 终端演示 | CI 中的真实 pty：发送按键字节，断言渲染出的画面 |

## 已编译，未运行 {#compiled-not-run}

- Win32 的 `Show`/`Hide`/`Raise`：决策表有单元测试，而在窗口线程上的调用只是通过了编译。
- Windows 托盘的 `Attach`。
- `webcanvas` 的 DOM 循环（`run_js.go`）：位于 `js && wasm` 构建标签之后，因此不在所测的覆盖率之内；使用它的应用会运行到它。
- `application` 的原生 `Run`：打开真实窗口的部分被排除在其覆盖率门禁之外；契约、事件转换和就绪计数器则在覆盖之内。

## 尚未完成 {#not-done}

- Windows 和 Linux 托盘会忽略 `MenuItem.Icon`：带图标的行会不带图标地绘制。
- `window` 的 Wayland 后端无法 `Raise`：xdg-shell 没有这样的请求，而 xdg-activation 需要来自用户自身输入的令牌。
- Android：仅支持 `android/arm64`，因为这是 Go 在没有 cgo 的情况下唯一能链接的 Android 目标。
- `window.Screens` 在 `js/wasm` 上返回 `ErrScreensUnsupported`。

## CI 覆盖的内容 {#what-ci-does-not-cover-yet}

2026-10-10，本页此前列出的五处缺口已全部补上：

| 模块 | 之前 | 现在 |
|---|---|---|
| `mvvmtk` | 没有覆盖率门禁；仅 amd64 和 arm64 | 采用与工具包相同的严格 100% 门禁（它本来就是 100.0%）；Linux 覆盖全部六种架构 |
| `application` | Linux、macOS 和 Windows 仅 amd64 | Linux 覆盖全部六种架构；macOS 和 Windows 覆盖 amd64 和 arm64 |
| `tray` | 五个操作系统/架构组合 | Linux 覆盖全部六种架构，macOS 两种，Windows 一种 |
| `mvvmlint` | 没有交叉构建 | Linux 覆盖全部六种架构，另有 macOS 和 Windows |
| `bricolint` | 一个“6 种架构”的步骤实际只构建五种 | 加入 loong64 |

因此，全部十七个库和工具现在都交叉构建到六种 64 位 Linux 架构，全部十九个模块都以 100% 语句覆盖率为门禁。`app-template` 和 `gallery` 是浏览器应用，构建到 `js/wasm`，这是它们唯一的目标。

## 安全 {#security}

于 2026-10-10 使用 `govulncheck`（符号级别：只统计模块可达的代码）测量：

- 19 个模块中有 15 个可达到存在漏洞的代码：位于 go1.27.1 的标准库中（`html/template`、`net/http`、`crypto/tls`、`mime/multipart`；已在 go1.27.2 中修复），以及 `golang.org/x/net` v0.58.0 中（`http2`，经由 `data` 中的 gRPC；已在 v0.60.0 中修复）。
- 当晚即已修复。每个模块的 CI 都使用 go1.27.2 构建，每个模块都要求 `x/net` v0.60.0。在大多数模块中它是间接依赖，Renovate 不会更新，因此是手工升级的。对全部 19 个模块再做一轮 `govulncheck`，没有发现可达的漏洞。
- 已发布：android v0.16.0、app-template v0.5.0、application v0.8.0、bricolint v0.4.1、data v0.4.0、mvvm v0.15.0、mvvmlint v0.4.1、mvvmtk v0.15.0、painter v0.16.0、skin v0.3.0、svg v0.7.0、tray v0.15.0、webcanvas v0.4.1、window v0.89.0，各版本号均由 `gorelease` 根据其 API 推导。2026-10-11：toolkit v0.329.0，以及 tui v0.62.0、desktop v0.20.0、gallery v0.7.0 和 isoicons v0.4.0。对于后四个，`gorelease` 无法加载模块图，因为它会触及 `github.com/ajroetker/go-highway/hwy/goat` v0.0.0，这是一个从未发布过的版本（[ajroetker/go-highway#73](https://github.com/ajroetker/go-highway/issues/73)），所以它们的版本号改为在 linux、darwin 和 windows 上用 `apidiff` 推导，遵循同一规则。
- 全部 23 个仓库的工作流（31 个文件）都用 `actionlint` 和 `wfaudit` 审计过：没有特权触发器，工作流级别没有写权限授予，上传的检出内容中没有遗留令牌，shell 步骤中没有不受信任的表达式。每个工作流都声明了自己的权限，因此没有哪个依赖仓库的默认令牌。
