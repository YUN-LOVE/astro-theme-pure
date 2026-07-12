---
title: '如何使用scrcpy'
description: '简短地介绍了scrcpy极其使用教程'
publishDate: '2026-07-12 17:39:34'
tags:
  - Technology
---

# 一、引言

**Scrcpy**（**Screen Copy**）是一个开源、免费且无需Root权限的安卓设备投屏与控制工具。它通过**ADB**（**Android调试桥**）传输视频流，以低延迟和高画质著称。  

核心特点：  

· 高效稳定：延迟可低至30-50ms，资源占用极低（内存约45MB）。  
· 双向控制：支持键盘、鼠标操作，以及剪贴板双向复制粘贴。  
· 功能全面：支持文件拖拽安装、录屏（MP4/MKV）和摄像头镜像（**Android 12+**）。  

# 二、驱动安装

你需要安装adb驱动，在[驱动包](https://one.031312.xyz/api/raw/?path=/%E6%96%87%E6%A1%A3%2B%E5%8E%8B%E7%BC%A9%E5%8C%85/%E9%AA%81%E9%BE%99%E5%BC%BA%E9%99%8Dmiui.rar)下载 

你需要下载adb sdk，在[Google adb sdk](https://developer.android.google.cn/tools/releases/platform-tools?hl=zh-cn)

# 三、基础使用

1. 环境准备  
手机需开启“**开发者选项**”和“**USB调试**”，并确认电脑已安装ADB工具。可通过 `adb devices` 验证连接状态。  

2. 有线连接（最常用）  
直接执行 `scrcpy` 即可开始投屏。  

3. 无线连接  

1. 切换TCP/IP：先有线连接，执行 `adb tcpip 5555`。  
2. 连接设备：断开USB，执行 `adb connect 设备IP:5555`。  
3. 启动：再次执行 `scrcpy`。  


# 四、摄像头模式（Android 12+）

通过 `--video-source=camera` 将摄像头画面镜像到电脑，而非屏幕。默认音频源会自动切换为麦克风。  

1. 基础开启  

```bash
scrcpy --video-source=camera
```

2. 选择与设置  

· 查看摄像头：`scrcpy --list-cameras`（*列出ID和分辨率*）。  
· 选择摄像头：  
  - `--camera-id=0`（*指定ID*）。  
  - `--camera-facing=front` 或 `back`（**指定前后置**）。  
- 调整画质：  
  - `--camera-size=1920x1080`（**指定分辨率**）。  
  - `--camera-fps=60`（**设定帧率**）。  
  - `-m 1024`（**限制最大尺寸**）。  

# 五、常用参数整理

| 类别 | 参数示例 | 作用说明 |
| :--- | :--- | :--- |
| 画面优化 | `-m 1024` / `-b 8M` / `--max-fps 30` | 限制分辨率/码率/帧率，改善性能。 |
| 窗口控制 | `-f` / `--always-on-top` / `-S` | 全屏/置顶/启动时关闭手机屏幕。 |
| 录屏 | `--record demo.mp4` | 保存投屏录像（也支持 `--no-display` 后台录制）。 |
| 输入控制 | `--mouse=uhid` / `--keyboard=uhid` | 模拟物理HID设备，适合游戏操作。 |
| 多设备 | `-s 设备序列号` | 连接特定设备（当有多个设备时）。 |

 # 六、快捷键（MOD键）
 
 默认为 Ctrl，常用组合：  

· MOD + f：全屏切换。  
· MOD + o：关闭手机屏幕（保持镜像）。  
· MOD + r：旋转手机屏幕。  
· 拖拽APK文件：直接安装应用。  

# Other：使用硬件编解码器

## 一、通过 ADB 获取编码器列表  

### 1. 查看所有编码器（含软件+硬件）  

```bash
adb shell media-codecs --list
```

输出示例（**重点关注带 `OMX.` 的硬件编码器和 `c2.` 的框架**）：  

```
OMX.qcom.video.encoder.avc      # 高通硬件 H.264
OMX.qcom.video.encoder.hevc     # 高通硬件 H.265
c2.qti.avc.encoder              # C2 框架 H.264
OMX.google.h264.encoder         # 软件编码器（性能差）
```

### 2. 只筛选视频编码器（更精确）  

```bash
adb shell media-codecs --list | grep -E "encoder.*(avc|hevc|h264|h265)"
```

### 3. 通过 scrcpy 直接查看（最便捷）  

```bash
scrcpy --list-encoders
```

会直接输出设备支持的编码器 ID，复制即可使用。  

## 二、在 scrcpy 中指定编码器

拿到编码器名称后，用 `--video-codec` 和 `--video-encoder` 指定：  

### 示例 1：指定硬件 H.264 编码器  

```bash
scrcpy --video-codec=h264 --video-encoder=OMX.qcom.video.encoder.avc
```

### 示例 2：指定硬件 H.265 编码器（画质更好）

```bash
scrcpy --video-codec=h265 --video-encoder=OMX.qcom.video.encoder.hevc
```

### 示例 3：强制使用软件编码器（兼容性测试）

```bash
scrcpy --video-codec=h264 --video-encoder=OMX.google.h264.encoder
```
