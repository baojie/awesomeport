# 2026-01-29 LAN Mouse - Wayland 环境下最简单的键鼠共享方案

在上一篇介绍 [x2x](2026-01-28-x2x.md) 的文章中，我们提到了它是一个 X11 专属工具。随着 Wayland 逐渐成为主流，很多用户发现 `x2x` 无法在新的显示服务器上工作。今天，我们来介绍 Wayland 环境下最简单易用的键鼠共享方案：`LAN Mouse`。

### 简介

`LAN Mouse` 是一个现代的跨平台键鼠共享工具，类似于苹果设备上的 Universal Control。它让你能用一套键盘和鼠标控制多台电脑，通过屏幕边缘"穿越"自动切换——就像多个显示器连在一起一样自然。

### 主要特点

* **图形界面**: 有 GTK GUI，无需手写配置文件
* **边缘穿越**: 鼠标移到屏幕边缘自动切换到另一台电脑
* **Wayland 原生**: 支持 wlroots 系（Hyprland、Sway）和 GNOME（通过 libei）
* **跨平台**: 主要支持 Linux Wayland，也部分支持 Windows、macOS 和 Xorg
* **安全加密**: 使用 DTLS 加密所有网络通信

### 安装

```bash
# 通用方式 (需要 Rust 工具链)
cargo install lan-mouse

# Arch Linux (AUR)
yay -S lan-mouse

# 或从 GitHub Releases 下载预编译二进制
# https://github.com/feschber/lan-mouse/releases
```

在被控机上，需要有权限访问 `/dev/uinput` 来模拟输入设备：
```bash
sudo usermod -aG input $USER
```
修改后需要重新登录才能生效。

### 使用方法

#### 1. 启动程序

在两台电脑上都运行 `lan-mouse`，会自动打开 GTK 图形界面。

#### 2. 添加设备

在主控机上点击 **Add** 按钮，输入被控机的主机名或 IP 地址。

#### 3. 授权连接

在被控机上，找到 **Incoming Connections** 区域，点击 **Authorize** 按钮授权主控机。授权时需要核对指纹（在主控机的 General 区域可以看到，格式如 `aa:bb:cc:...`）。

#### 4. 设置屏幕位置

在 GUI 中拖拽设置屏幕的相对位置（左、右、上、下）。

#### 5. 开始使用

鼠标移到屏幕边缘就能自动切换到另一台电脑，键盘输入也会跟随切换。

### 注意事项

* 确保防火墙开放 UDP 端口 **4242**
* Sway 1.8 以下版本需要打补丁才能支持指针锁定

### 守护进程模式

如果想后台运行或开机自启：

```bash
lan-mouse daemon
```

可以配合 systemd 服务使用。

### 其他替代方案

如果 `LAN Mouse` 不能满足你的需求，还有以下选择：

#### Wey - 热键切换方案

如果你更喜欢通过热键而非边缘穿越来切换，`Wey` 是一个轻量的选择。它通过配置文件驱动，使用 `Super+方向键` 等热键精准切换。

```bash
cargo install wey
```

缺点是需要手写 TOML 配置文件，配置相对繁琐。

#### Input Leap - 成熟稳定

Barrier 的继任者，有完整的图形界面，跨平台支持好：

```bash
# Arch Linux
yay -S input-leap

# Ubuntu/Debian
sudo apt install input-leap
```

#### Deskflow - 跨平台首选

Synergy/Barrier 的新分支，Windows/macOS/Linux 全平台支持：

```bash
flatpak install flathub org.deskflow.deskflow
```

### 总结

对于 Wayland 用户来说，键鼠共享的推荐选择：

* **首选 `LAN Mouse`**: 有 GUI、边缘穿越、配置简单、开箱即用
* **需要热键切换**: 选 `Wey`，配置文件驱动，精准高效
* **需要跨平台（Windows/macOS）**: 选 `Deskflow` 或 `Input Leap`

`LAN Mouse` 完美填补了 `x2x` 在 Wayland 生态中的空白，是目前最推荐的方案。
