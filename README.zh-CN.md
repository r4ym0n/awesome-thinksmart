# Awesome ThinkSmart [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> 一份精心整理的清单，收录了破解、刷机和改造 **Lenovo ThinkSmart View (CD-18781Y)** 所需的固件、工具、教程与项目——这款已经停产的微软 Teams 智能显示屏，如今以 20–40 美元的价格成为了 Home Assistant 仪表盘、Linux 平板和家庭智能屏圈子的宠儿。

**[English Version](./README.md)**

ThinkSmart View 并不是消费级平板——它是一台 8 英寸的 Teams 会议设备，作为商业产品失败后，在二手市场被爱好者们发掘。硬件素质意外地好（扬声器出色、IPS 触摸屏扎实、金属机身），加上它运行的是泄露的 Lenovo userdebug 安卓镜像、高通芯片支持 EDL 刷机，于是它成了可玩性最高的智能屏之一。这份清单收集了入坑所需的全部资料。

## 目录

- [为什么是 ThinkSmart View?](#为什么是-thinksmart-view)
- [设备规格](#设备规格)
- [选择你的路线](#选择你的路线)
- [操作系统与 ROM](#操作系统与-rom)
- [刷机教程](#刷机教程)
- [固件与镜像下载](#固件与镜像下载)
- [源码与开发](#源码与开发)
- [工具](#工具)
- [Home Assistant 与智能家居](#home-assistant-与智能家居)
- [项目与玩法](#项目与玩法)
- [硬件研究与折腾](#硬件研究与折腾)
- [常见问题排错](#常见问题排错)
- [社区](#社区)
- [贡献](#贡献)
- [许可](#许可)

## 为什么是 ThinkSmart View?

- **便宜**:发售价约 $350，现在 eBay/亚马逊上常见 $20–$40 的全新库存机。
- **做工好**:金属机身、8 英寸 IPS 触摸屏、扬声器是真不错(TAS5782M Class-D 功放)。
- **可折腾**:泄露的 userdebug 镜像可开 ADB/ROOT；高通 EDL 模式可以在 bootloader 不解锁的情况下完整刷机。
- **实用**:近乎完美的 HA 墙面仪表盘、家庭学习终端、Linux 触控面板、Kiosk、电子相框、摄像头监视器。

## 设备规格

| 部件        | 详情                                                       |
| ----------- | ---------------------------------------------------------- |
| 型号        | Lenovo ThinkSmart View **CD-18781Y**(有 Teams / Zoom 版本) |
| SoC         | 高通 APQ8053 / MSM8953(骁龙 625 级别,八核 Cortex-A53)      |
| 内存        | 2 GB LPDDR3                                                |
| 存储        | 8 GB eMMC                                                  |
| 屏幕        | 8 英寸 IPS 触摸屏,DSI 接口                                 |
| 音频功放    | 德州仪器 TAS5782M(I2S Class-D)                             |
| WiFi / 蓝牙 | 高通 QCA9379(ath10k_sdio)                                  |
| 传感器      | BMA253 加速度计、VCNL4200 距离/环境光传感器                |
| 摄像头      | 硬件指向三星 S5KC505A(驱动仍在研究中)                      |
| 接口        | USB-C(在底部橡胶垫下)、圆口电源                            |
| 电池/基带   | 无——会议一体机,市电供电                                    |

**已知短板**:2 GB 内存。复杂网页应用会偏慢；跑仪表盘和 Kiosk 完全够用。

**进入 EDL(紧急下载)模式**——最重要的技巧:

1. 拔掉电源。
2. 同时按住 **音量+ 和 音量−**。
3. 重新插上电源(用智能插座更方便)。
4. 约 5 秒后/电脑识别到 USB 后松开按键。屏幕保持黑屏——这是正常的。

注意:fastboot 存在但**不能刷机**——bootloader 处于 OEM 锁状态且强制校验签名。只有 EDL 可用。

## 选择你的路线

| 你的目标                          | 推荐方案                                     | 说明                                                         |
| --------------------------------- | -------------------------------------------- | ------------------------------------------------------------ |
| HA 仪表盘 / Kiosk                 | [Lineage 15.1(Android 8.1)](#操作系统与-rom) | 最成熟;可跑 Fully Kiosk、View Assist、Play Store(pico GApps) |
| 真正的 Linux 桌面                 | [Ubuntu 24.04 + Plasma 6](#操作系统与-rom)   | 扬声器/麦克风/蓝牙/旋转/靠近唤醒均可用;摄像头还在研究        |
| Mainline Linux / Alpine、深度折腾 | [postmarketOS](#操作系统与-rom)              | FelixKa 的原始 PMOS 移植,也是 Ubuntu 移植的基础              |
| 带屏幕的 HA 语音卫星              | [Debian 13 + home-companion](#操作系统与-rom) | 极简原生方案(系统本身约 190 MB 内存);唤醒词、计时器、HA 仪表盘、AirPlay |
| 最小改动、保留原生系统            | [ROOT / Magisk 路线](#刷机教程)              | 通过 microG 装 Play Store、更新 WebView、换启动器            |

## 操作系统与 ROM

- [LineageOS 15.1 for ThinkSmart View(`starfire`)](https://s3.us-west-1.wasabisys.com/rom-release/LineageOS/15.1/starfire/lineage-15.1-20240531-UNOFFICIAL-starfire.zip) — @deadman(deadman96385)与 @electimon 制作的非官方 Lineage 15.1(Android 8.1)。修复了早期版本浏览器无声音、触控超时、Android 11 换启动器崩溃等问题。**仪表盘/Kiosk 用途的首选系统。** 偶尔还有更新,但不要指望官方支持。
- [Ubuntu 24.04 for ThinkSmart View](https://github.com/rickx/ubuntu-thinksmart) — 定制 Ubuntu 24.04 ARM64 移植,内核 6.19.5-msm8953。主推镜像为 Plasma Mobile 6(KDE neon 6.7.0)。扬声器、麦克风、蓝牙、加速度计自动旋转、亮度滑条、靠近唤醒全部可用;Venus V4L2 硬解(H.264 1080p 约 1.26 倍实时,HEVC 约 5.7 倍);摄像头驱动研究中。
- [thinksmart-ubuntu-image(小智语音助手)](https://github.com/justhalfbit/thinksmart-ubuntu-image) — 基于 rickx 移植构建的 Ubuntu 24.04 镜像,去掉 Plasma 桌面,开机直达本地语音助手:「小麦小麦」唤醒词、本机语音识别、可接 Claude / DeepSeek 等大模型,带网页管理后台。包含设备修复(双麦克风、扬声器、熄屏、接近唤醒、WiFi 稳定性、GPU 白屏补丁)和一份同时面向人类与 AI 助手的完整刷机教程。界面与唤醒词目前以中文为主。
- [ThinkSmart View Debian](https://github.com/m0s4ik/thinksmart-view-debian) — 基于 postmarketOS msm8953 内核的极简 Debian 13(trixie),无桌面、无浏览器:整个系统约占 190 MB 内存,约 24 秒开机。屏幕、触控、Adreno 506 GPU(精简版 Mesa)、扬声器(TAS5782M)、双麦克风、WiFi、加速度计、光线/接近传感器和 Venus 硬解均可用;可选 WPE WebKit 与 AirPlay 运行时。通过 EDL 刷入,适合作为自己应用的轻量底座。
- [postmarketOS 设备页:lenovo-starfire](<https://wiki.postmarketos.org/wiki/Lenovo_ThinkSmart_View_(lenovo-starfire)>) — FelixKa 的原始 PMOS(Alpine Linux)移植,包含后来所有工作所依赖的 TAS5782M 音频驱动参考实现。设备包:`device-lenovo-starfire`(pmaports)。
- **原厂安卓(Teams / "Teams & Others")** — 一切刷机的基础。先刷 `200628.084` 版本回到已知良好的状态(见[固件下载](#固件与镜像下载));泄露的 userdebug 版本可在 设置 → 关于 → 连点固件版本 后开启 ADB。

## 刷机教程

- [pgale/lineage_15.1_Installation_TSV](https://github.com/pgale/lineage_15.1_Installation_TSV) — 最权威的 Lineage 15.1 刷机教程,整理自 XDA 帖、论坛和 Discord。涵盖 Windows(QFIL/QPST)和 Linux(bkerler/edl,推荐树莓派)两条路线、OpenGApps 侧载、HA 用途的安卓设置、新版 WebView 安装与排错。**从这里开始。**
- [rickx/ubuntu-thinksmart — FLASHING.md](https://github.com/rickx/ubuntu-thinksmart/blob/main/FLASHING.md) — 完整的 原厂安卓 → Ubuntu 流程(走 EDL):刷 `lk2nd` → 刷带 SSH 的 PMOS 引导镜像 → 在设备上运行 `apply-ubuntu-gpt.sh` 重写 GPT → 刷 Ubuntu 镜像。支持预注入 WiFi,首次开机即可 SSH。
- [XDA:[GUIDE][ROOT] Unlocking and Improving Your Lenovo ThinkSmart View (CD-18781Y)](https://xdaforums.com/t/guide-root-unlocking-and-improving-your-lenovo-thinksmart-view-cd-18781y.4584033/) — 最初的 ROOT 教程(Chewie610,基于 deadman96385 的固件/工具):QFIL 刷机、TWRP、Magisk、microG + Play Store、WebView Manager。想留在类原生安卓就走这条。
- [Home Assistant 社区:Lenovo ThinkSmart View ROM/OS Development](https://community.home-assistant.io/t/lenovo-thinksmart-view-rom-os-development/676324) — 大部分生态动态所在的大型讨论帖(安卓 8.1/11 固件、PMOS、Ubuntu、仪表盘)。很长但必读。
- [在 VirtualBox 虚拟机里刷机](https://github.com/pgale/lineage_15.1_Installation_TSV/discussions/43) — 宿主机 USB 直通搞不定时的替代方案。
- thinksmart-ubuntu-image 刷机教程 — 路线 A(原厂 → lk2nd → bootstrap → 改分区表 → 刷镜像)与路线 B(在已有 Ubuntu 上重刷)的完整流程,含校验和核对、整盘 eMMC 备份、首次开机设置、排错和用户数据备份/还原。仓库链接见上文操作系统一节。

### 刷机速查表(摘自上述教程)

```text
1. 进入 EDL:按住 音量+ + 音量−,插上电源。屏幕保持黑屏。
2. Windows: QFIL → Flat Build → prog_emmc_firehose_8953_ddr.mbn → 载入 rawprogram_unsparse.xml + patch0.xml → Download。
   Linux:  edl qfil rawprogram_unsparse.xml patch0.xml image/ --loader=prog_emmc_firehose_8953_ddr.mbn
3. 写入 recovery 与 aboot:
   edl w aboot image/emmc_appsboot.mbn
   edl w abootbak image/emmc_appsboot.mbn
   edl w recovery lineage-15.1-XXXX-UNOFFICIAL-starfire-recovery.img
4. 进 recovery:拔电,按住 音量+,插电。recovery 里触屏不可用——
   用音量键移动,用摄像头隐私滑块确认选择。
5. Factory reset → Apply update → Apply from ADB →
   adb sideload lineage-15.1-XXXX-UNOFFICIAL-starfire.zip
   adb sideload open_gapps-arm-8.1-pico-20220215.zip(每次侧载前都要重新选择 ADB sideload)
6. 再 Factory reset 一次 → Reboot。
```

> ⚠️ **USB 注意事项**:使用 **USB 2.0 接口和线材**(黑色口,不是蓝色)。USB 3.x 主机常导致 Sahara 错误和刷机中途失败。进入 EDL 后尽快开始刷机——等待过久会报 `Sahara protocol failed`。Windows 上需安装高通 HS-USB QDLoader 驱动,未签名驱动需要测试模式。

## 固件与镜像下载

| 文件                                                     | 说明                                                | 链接                                                                                                                                            |
| -------------------------------------------------------- | --------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `CD-18781Y.200628.084.zip`                               | 原厂 "Teams & Others" 固件(所有刷机的已知良好起点)  | [wasabisys 镜像](https://s3.wasabisys.com/filestash-buk/lenovo-thinksmart-view/CD-18781Y.200628.084.zip)                                        |
| `lineage-15.1-20240531-UNOFFICIAL-starfire.zip`          | Lineage 15.1 系统包                                 | [wasabisys 镜像](https://s3.us-west-1.wasabisys.com/rom-release/LineageOS/15.1/starfire/lineage-15.1-20240531-UNOFFICIAL-starfire.zip)          |
| `lineage-15.1-20240531-UNOFFICIAL-starfire-recovery.img` | Lineage recovery(刷入 `recovery` 分区)              | [wasabisys 镜像](https://s3.us-west-1.wasabisys.com/rom-release/LineageOS/15.1/starfire/lineage-15.1-20240531-UNOFFICIAL-starfire-recovery.img) |
| `open_gapps-arm-8.1-pico-20220215.zip`                   | 谷歌服务(pico 是不扩分区能装的最大版本;nano 也可以) | [OpenGApps SourceForge](https://sourceforge.net/projects/opengapps/files/arm/20220215/)                                                         |
| `plasma6-ubuntu-qcom-msm8953.img`                        | Ubuntu 24.04 + Plasma Mobile 6 镜像                 |
| `thinksmart-ubuntu-*.img.xz`                             | 语音助手 Ubuntu 镜像(小智,无桌面)                   | [GitHub Releases](https://github.com/justhalfbit/thinksmart-ubuntu-image/releases)                                                              |     | 见 [rickx/ubuntu-thinksmart](https://github.com/rickx/ubuntu-thinksmart) README(MEGA) |
| `bootstrap-pmos-ssh-generic-qcom-msm8953.img`            | 带 SSH 的 PMOS 引导镜像(用于重写 GPT)               | [GitHub Release](https://github.com/rickx/ubuntu-thinksmart/releases)(资产 `bootstrap-2026-05-22`)                                              |
| `lk2nd.img`                                              | lk2nd 二级引导                                      | [rickx/ubuntu-thinksmart prebuilt](https://github.com/rickx/ubuntu-thinksmart/tree/main/prebuilt)                                               |
| Rescue & Smart Assistant (RSA)                           | 联想官方工具,可恢复原厂固件                         | [联想支持站点](https://support.lenovo.com)                                                                                                      |

## 源码与开发

- [Starfire-development/android_kernel_lenovo_apq8053](https://github.com/Starfire-development/android_kernel_lenovo_apq8053) — Lineage 15.1 `starfire` 构建的内核源码。
- [Starfire-development/android_device_lenovo_starfire](https://github.com/Starfire-development/android_device_lenovo_starfire) — 安卓设备树(音频、蓝牙、GPS、按键布局、灯效、overlay、sepolicy 等)。
- [Starfire-development/proprietary_vendor_lenovo_starfire](https://github.com/Starfire-development/proprietary_vendor_lenovo_starfire) — 厂商闭源组件(部分图形/音频/DRM/gatekeeper 来自摩托罗拉 G6 "ali",其余来自原厂)。
- [rickx/ubuntu-thinksmart](https://github.com/rickx/ubuntu-thinksmart) — Ubuntu 移植源码:TAS5782M 内核驱动、ath10k 补丁、内核/DTS 补丁、用户态守护进程(息屏、靠近唤醒、自动旋转)、构建/部署脚本。
- [msm8953-mainline/linux](https://github.com/msm8953-mainline/linux) — MSM8953 设备的 mainline Linux 内核(Ubuntu 移植的构建基础)。
- [msm8953-mainline/lk2nd](https://github.com/msm8953-mainline/lk2nd) — 用于从 `boot` 分区引导 mainline 的 lk2nd 二级引导。
- [msm8953-mainline/alsa-ucm-conf](https://github.com/msm8953-mainline/alsa-ucm-conf) — ALSA UCM2 音频配置。
- [msm8953-mainline/linux-mdss-dsi-panel-driver-generator](https://github.com/msm8953-mainline/linux-mdss-dsi-panel-driver-generator) — DSI 面板驱动生成器。
- [postmarketOS pmaports — device-lenovo-starfire](https://gitlab.postmarketos.org/postmarketOS/pmaports) — PMOS 设备包。
- 联想开源发布 `lenovo_cd-18781y.201218.2018_opensource` — 联想公开的安卓内核源码(在社区仓库中有镜像/引用)。

## 工具

- [bkerler/edl](https://github.com/bkerler/edl) — 开源高通 EDL/Sahara/Firehose 刷机工具(`edl`、`edl.py`),Linux 下的 QFIL 等价物。
- QPST / QFIL — 高通官方 Windows 刷机套件(配 HS-USB QDLoader 驱动)。未签名驱动需测试模式。
- ADB & Fastboot++ — 教程推荐的 Windows ADB/fastboot 一键安装器。
- [联想 Rescue & Smart Assistant (RSA)](https://support.lenovo.com) — 搞砸后恢复原厂固件。
- [Magisk](https://github.com/topjohnwu/Magisk) — 原生安卓路线的 ROOT 方案。
- [OpenGApps](https://opengapps.org) — 谷歌应用包(用 `arm` 8.1 版,pico 或 nano)。
- WebView Manager(Magisk 模块)/ 新版 [Android System WebView APK](https://www.apkmirror.com) — 例如 125.x 支持 Android 8.1;HA 仪表盘明显更快。
- [Fully Kiosk Browser](https://www.fully-kiosk.com) — HA 仪表盘最常使用的 Kiosk 浏览器(屏保、人体感应、REST API、WebView 选择)。
- [scrcpy](https://github.com/Genymobile/scrcpy) — 通过 ADB 镜像/操控设备,装软件时很方便。
- `adb` / `nmtui` / `ssh` — 安卓与 Ubuntu 的日常管理(Ubuntu 在 `172.16.42.1` 提供 USB 网络)。

## Home Assistant 与智能家居

- [dinki/View-Assist](https://github.com/dinki/View-Assist) — 让 TSV 在 HA 圈走红的旗舰项目:带可视反馈的语音助手卫星设备,支持计时器、天气、摄像头画面等视图。跑在安卓(Fully Kiosk)和 ESPHome 卫星上。
- [home-companion](https://github.com/m0s4ik/home-companion) — 为 TSV 打造的原生(Rust)Home Assistant 带屏语音卫星,运行在 ThinkSmart View Debian 上:本机唤醒词("Okay Alfred")和一位动画管家角色,离线可用的本地计时器、闹钟与提醒,全屏显示 HA 自带仪表盘,AirPlay 音乐与投屏。在屏幕上扫二维码即可完成设置(Wyoming 卫星 + HA 内置 MCP 客户端,无需加载项)。支持意大利语和英语。
- [Fully Kiosk Browser](https://www.fully-kiosk.com) + HA 仪表盘 — 最常见的 TSV 组合(安卓设置建议见 pgale 教程:无锁屏、隐藏电池图标、音量键唤醒、自动旋转开关的 bug 规避)。
- [HA 社区:基于 TSV 的家庭仪表盘与多台 Fully Kiosk 设备管理](https://community.home-assistant.io/t/my-take-on-a-home-dashboard-running-on-the-lenovo-thinksmart-view-and-managing-multiple-devices-running-fully-kiosk/718355) — 仪表盘设计与机群管理。
- [HA 社区:Lenovo ThinkSmart View ROM/OS Development](https://community.home-assistant.io/t/lenovo-thinksmart-view-rom-os-development/676324) — 上述一切的源头讨论帖。
- [Channels 社区:ThinkSmart Views——又一款可破解的联想设备](https://community.getchannels.com/t/thinksmart-views-another-hack-able-lenovo-device/38911) — 把 TSV 用作 Channels 客户端显示屏。

## 项目与玩法

- [pgale/lineage_15.1_Installation_TSV](https://github.com/pgale/lineage_15.1_Installation_TSV) — 除了是刷机教程,还记录了用 **七台 TSV** 做全屋 HA 显示屏的实战。
- [samyeei/thinksmart-learning-station](https://github.com/samyeei/thinksmart-learning-station) — 把 TSV 变成**家庭学习终端**:官方 ChatGPT App + 手动中文(Fcitx)键盘开关、离线桌面(大时钟、WiFi、每日一句)、MusicFree 音乐入口、局域网家长管理页(前台时长估算、按需查看屏幕、密码管理)。源码 MIT 协议,alpha 阶段。
- [dinki/View-Assist](https://github.com/dinki/View-Assist) — HA 语音卫星 + 可视反馈。
- 家庭仪表盘 / 墙面面板 — 最经典的玩法(见上述教程)。
- Linux 触控面板 — Ubuntu/PMOS 镜像使其成为通用 ARM64 平板(USB 网络 + SSH 可无人值守)。
- 摄像头监视器 / RTSP 播放器 — Venus 硬解可播 1080p H.264 摄像头流(编码时不要用 B 帧:`-bf 0`)。
- 电子相框 / Kiosk / 信息展示屏 — Fully Kiosk 屏保,或 PMOS/Ubuntu 下的纯浏览器方案。
- 本地语音助手 — 基于小智(xiaozhi)的 Ubuntu 镜像让 TSV 变成常驻唤醒词语音助手:本机语音识别,大模型后端可自选(仓库链接见上文操作系统一节)。

## 硬件研究与折腾

来自 [rickx/ubuntu-thinksmart/research](https://github.com/rickx/ubuntu-thinksmart) 的深度分析(想参与开发必读):

- **WiFi GTK 重连修复** — QCA9379 固件在重连时以 `-110` 错误丢密钥;通过修补 `ath10k_core.ko`(吞掉 `-ETIMEDOUT`)解决。此固件无 raw-mode,`cryptmode=1` 不可行。
- **音频(TAS5782M)** — 定制 I2S Class-D 驱动 + DSP 固件 + UCM2 HiFi 配置 + WirePlumber 策略;必须强制 S16LE(S24 几乎无声,S32 直接被拒)。内核启用了 KCFI,模块必须用 clang 构建。
- **Plasma 6 息屏问题** — DSI 面板无法从 DRM Off 恢复;powerdevil 超时必须设为 0。由三个用户态守护进程接管:`idle-blanker`、`vcnl4200-proximity-daemon`(挥手亮屏)、`rotation-daemon`(BMA253 自动旋转)。
- **硬件视频解码** — Venus V4L2 M2M:H.264 1080p 1.26 倍实时,HEVC 1080p 5.7 倍;不支持 B 帧。
- **摄像头** — 硬件指向三星 S5KC505A;移植工作见 `research/camera/`。
- **距离传感器(VCNL4200)** — 采用用户态守护进程方案;勿用上游 vcnl4000 驱动(会以全功率常开红外 LED)。

## 常见问题排错

| 症状                                        | 解决办法                                                                          |
| ------------------------------------------- | --------------------------------------------------------------------------------- |
| QFIL 报 `Sahara` 错误 / 刷一分钟左右失败    | 换 USB 2.0 口和线;进入 EDL 后立即下载;禁用驱动签名(测试模式)                      |
| QFIL 识别不到端口                           | 重装 HS-USB QDLoader 驱动;看设备管理器;secure boot 可能拦截驱动(临时关闭)         |
| `adb: sideload connection failed` / `E1001` | ADB sideload 超时很快——每次侧载前重新选择;重启进 recovery 重试                    |
| Lineage 初始设置卡在 "Checking for updates" | 几秒后按返回键(另一层检查页会滑入;此时按返回即可继续)                             |
| Lineage WiFi 掉线直到重启                   | 开关一次 WiFi;开着 Fully Kiosk 似乎不会复现;有人每晚定时重启 Kiosk                |
| Lineage 旋转卡住                            | 开关一次自动旋转(已知 bug,暂无根治)                                               |
| 原生固件总是重置设置                        | 卸载 MDM/ kiosk / managed-settings 包:`adb shell pm uninstall -k --user 0 <包名>` |
| Ubuntu:WiFi 在 GTK 重连时断开               | 镜像已修复(打过补丁的 ath10k_core.ko);dmesg 里的警告可忽略                        |
| Ubuntu:睡眠后屏幕唤不醒                     | 永远不要开启 powerdevil 显示超时(DSI DRM Off 不可恢复);使用内置守护进程           |
| 变砖 / 想刷回原厂                           | 联想 Rescue & Smart Assistant(记得还是用 USB 2.0 口)                              |

## 社区

- [XDA 论坛 — ThinkSmart View root/解锁帖](https://xdaforums.com/t/guide-root-unlocking-and-improving-your-lenovo-thinksmart-view-cd-18781y.4584033/) — 固件泄露与最初魔改的发源地。
- [Home Assistant 社区 — TSV ROM/OS 开发讨论](https://community.home-assistant.io/t/lenovo-thinksmart-view-rom-os-development/676324) — 最主要的活跃讨论。
- Home Assistant 与 View Assist 的 [Discord](https://www.home-assistant.io)、postmarketOS 的 [Matrix/IRC](https://postmarketos.org) — @deadman、@electimon、FelixKa 等人的活动地。
- Reddit(r/homeassistant、r/selfhosted)——周期性的"$30 Teams 显示屏"经验帖。

## 贡献

欢迎贡献!请先阅读 [CONTRIBUTING.md](./CONTRIBUTING.md)。简要规则:只收录确实可用或真正有信息量的内容;链接到原始来源(而非转载);沿用现有格式;非官方/已停更的资源请注明。

## 许可

[CC0-1.0](./LICENSE) — 公有领域贡献。署名是情分,不是义务。

## 免责声明

本清单是非官方的,与 Lenovo、微软、谷歌或 LineageOS 项目均无隶属或认可关系。所有自定义 ROM 与修改都是社区作品,**没有任何保修和支持**——你可能会把设备刷砖,也没有人有义务帮你。ThinkSmart View 已停产,刷机风险完全自负。部分文件来自社区镜像站——如提供校验值请核对,下载内容注意查毒。
