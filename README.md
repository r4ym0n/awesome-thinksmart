# Awesome ThinkSmart [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated list of firmware, tools, guides, and projects for hacking, flashing, and repurposing the Lenovo ThinkSmart View (CD-18781Y) — the discontinued Microsoft Teams appliance that became a cult-favorite Home Assistant dashboard, Linux tablet, and family smart display for $20–$40.

[中文版本 / Chinese Version](https://github.com/r4ym0n/awesome-thinksmart/blob/main/README.zh-CN.md)

ThinkSmart View is NOT a consumer tablet — it is an 8" Teams display that flopped as an appliance and is now dirt cheap on the second-hand market. Its hardware is surprisingly good (great speaker, solid IPS touchscreen, metal build), and because it runs a leaked userdebug Android image with an EDL-reachable Qualcomm SoC, the community has turned it into one of the most hackable smart displays around. This list collects everything you need to join in.

## Contents

- [Why the ThinkSmart View](#why-the-thinksmart-view)
- [The Device](#the-device)
- [Choose Your Path](#choose-your-path)
- [Operating Systems & ROMs](#operating-systems--roms)
- [Flashing Guides](#flashing-guides)
- [Firmware & Images](#firmware--images)
- [Source Code & Development](#source-code--development)
- [Tools](#tools)
- [Home Assistant & Smart Home](#home-assistant--smart-home)
- [Projects & Use Cases](#projects--use-cases)
- [Hardware Hacking & Research](#hardware-hacking--research)
- [Troubleshooting](#troubleshooting)
- [Community](#community)

## Why the ThinkSmart View

- Cheap — ~$350 retail, now commonly $20–$40 new-in-box on eBay/Amazon.
- Well built — metal body, quality 8" IPS touchscreen, and a genuinely good speaker (TAS5782M Class-D amp).
- Hackable — leaked Lenovo userdebug image enables ADB/root; Qualcomm EDL mode allows full reflashing even with an OEM-locked bootloader.
- Useful — near-perfect Home Assistant wall dashboard, family learning terminal, Linux touch panel, kiosk, digital frame, camera monitor.

## The Device

| Component    | Details                                                                 |
| ------------ | ----------------------------------------------------------------------- |
| Model        | Lenovo ThinkSmart View **CD-18781Y** (Teams / Zoom variants exist)      |
| SoC          | Qualcomm APQ8053 / MSM8953 (Snapdragon 625-class, octa-core Cortex-A53) |
| RAM          | 2 GB LPDDR3                                                             |
| Storage      | 8 GB eMMC                                                               |
| Display      | 8" IPS touchscreen, DSI                                                 |
| Audio amp    | Texas Instruments TAS5782M (I2S Class-D)                                |
| WiFi / BT    | Qualcomm QCA9379 (ath10k_sdio)                                          |
| Sensors      | BMA253 accelerometer, VCNL4200 proximity/ALS                            |
| Camera       | Hardware points to Samsung S5KC505A (driver research in progress)       |
| Ports        | USB-C (under rubber foot on the bottom), barrel power                   |
| Cell/battery | None — Teams appliance, mains powered                                   |

**Known limitation**: 2 GB RAM. Heavy web apps can be slow; for dashboards and kiosk use it is fine.

**Getting into EDL (Emergency Download) mode** — the single most important trick:

1. Unplug power.
2. Hold **Vol+ and Vol−** together.
3. Plug power back in (a smart plug helps).
4. Release after ~5 s / when the host USB enumerates. Screen stays black — that is normal.

Note: fastboot exists but **cannot flash** — the bootloader is OEM-locked and enforces signed images. Only EDL works.

## Choose Your Path

| You want                         | Go with                    | Notes                                                                  |
| -------------------------------- | -------------------------- | ---------------------------------------------------------------------- |
| Home Assistant dashboard / kiosk | Lineage 15.1 (Android 8.1) | Most mature; runs Fully Kiosk, View Assist, Play Store (pico GApps)    |
| A real Linux desktop             | Ubuntu 24.04 + Plasma 6    | Working audio/mic/BT/rotation/proximity wake; camera still in research |
| Mainline Linux / Alpine, hacking | postmarketOS               | FelixKa's original PMOS bring-up; basis of the Ubuntu port             |
| HA voice satellite with a screen | Debian 13 + home-companion | Minimal native stack (~190 MB base system); wake word, timers, HA dashboard, AirPlay |
| Minimal changes, keep stock OS   | Root / Magisk route        | Play Store via microG, updated WebView, custom launcher                |

## Operating Systems & ROMs

- LineageOS 15.1 for ThinkSmart View (`starfire`) - Unofficial Lineage 15.1 (Android 8.1) build by @deadman (deadman96385) with @electimon. Fixes the audio-in-browser, touchscreen timeout, and A11 launcher-crash issues that plagued earlier builds. **The recommended OS for dashboard/kiosk use.** Still receives occasional fixes; do not expect official support.
- [Ubuntu 24.04 for ThinkSmart View](https://github.com/rickx/ubuntu-thinksmart) - Custom Ubuntu 24.04 ARM64 port, kernel 6.19.5-msm8953. Primary image is Plasma Mobile 6 (KDE neon 6.7.0). Speaker audio, mic, Bluetooth, accelerometer auto-rotation, brightness slider, and proximity wake all working; hardware video decode via Venus V4L2 (H.264 1080p ~1.26× realtime, HEVC ~5.7×); camera driver research in progress.
- [thinksmart-ubuntu-image (xiaozhi voice assistant)](https://github.com/justhalfbit/thinksmart-ubuntu-image) - Ubuntu 24.04 image built on rickx's port with the Plasma desktop removed: boots straight into a local voice assistant (「小麦小麦」 wake word, on-device speech recognition, Claude / DeepSeek and other LLM backends, web admin panel). Includes device fixes (both mics, speaker, screen blanking, proximity wake, WiFi stability, GPU white-screen patches) and a complete flashing tutorial written for both humans and AI agents. UI and wake word are Chinese-focused for now.
- [ThinkSmart View Debian](https://github.com/m0s4ik/thinksmart-view-debian) - Minimal Debian 13 (trixie) on the postmarketOS msm8953 kernel, with no desktop and no browser: ~190 MB of RAM for the whole system, ~24 s boot. Screen, touch, Adreno 506 GPU (reduced Mesa), speaker (TAS5782M), both microphones, Wi-Fi, accelerometer, light/proximity sensor and the Venus hardware decoder working; optional WPE WebKit and AirPlay runtime layers. Flashed over EDL; meant as a light base for your own application.
- [postmarketOS device: lenovo-starfire](https://wiki.postmarketos.org/wiki/Lenovo_ThinkSmart_View_%28lenovo-starfire%29) - Original PMOS (Alpine Linux) bring-up by FelixKa, including the TAS5782M audio driver reference that later work builds on. Device package: `device-lenovo-starfire` in pmaports.
- Stock Android (Teams / "Teams & Others") - The base firmware everything starts from. Flash the `200628.084` build first to get a known-good state (see the firmware table below); the leaked userdebug build enables ADB unlock via Settings → About → tap firmware version repeatedly.

## Flashing Guides

- [pgale/lineage_15.1_Installation_TSV](https://github.com/pgale/lineage_15.1_Installation_TSV) - The definitive step-by-step Lineage 15.1 flashing guide, collated from the XDA thread, forums, and Discord. Covers Windows (QFIL/QPST) and Linux (bkerler/edl, Raspberry Pi recommended), sideloading OpenGApps, Android setup for HA use, newer WebView install, and troubleshooting. Start here.
- [rickx/ubuntu-thinksmart FLASHING.md](https://github.com/rickx/ubuntu-thinksmart/blob/main/FLASHING.md) - Full stock-Android to Ubuntu flow via EDL: flash `lk2nd`, flash an SSH-capable PMOS bootstrap image, rewrite the GPT on-device with `apply-ubuntu-gpt.sh`, then flash the Ubuntu image. Includes WiFi pre-injection so the device is SSH-able on first boot.
- [XDA Guide: Root and Improve Your ThinkSmart View](https://xdaforums.com/t/guide-root-unlocking-and-improving-your-lenovo-thinksmart-view-cd-18781y.4584033/) - The original root guide (Chewie610, building on deadman96385's firmware/tools): QFIL flashing, TWRP, Magisk, microG + Play Store, WebView Manager. Use this if you want to stay on stock-ish Android.
- [Flashing from within a VirtualBox VM](https://github.com/pgale/lineage_15.1_Installation_TSV/discussions/43) - Alternative when you can't get USB passthrough working on your host.
- thinksmart-ubuntu-image flashing guide - Route A (stock → lk2nd → bootstrap → GPT rewrite → image) and Route B (re-flash over an existing Ubuntu) walkthrough with checksum verification, full-eMMC backup steps, first-boot setup, troubleshooting, and user-data backup/restore. The repo is linked in the Operating Systems section above.

### Flashing cheat-sheet (condensed from the guides)

```text
1. Enter EDL: hold Vol+ + Vol−, plug in power. Screen stays black.
2. Windows: QFIL → Flat Build → prog_emmc_firehose_8953_ddr.mbn → rawprogram_unsparse.xml + patch0.xml → Download.
   Linux:  edl qfil rawprogram_unsparse.xml patch0.xml image/ --loader=prog_emmc_firehose_8953_ddr.mbn
3. Write recovery + aboot:
   edl w aboot image/emmc_appsboot.mbn
   edl w abootbak image/emmc_appsboot.mbn
   edl w recovery lineage-15.1-XXXX-UNOFFICIAL-starfire-recovery.img
4. Boot recovery: unplug, hold Vol+, plug power. Touch doesn't work in recovery —
   navigate with Vol+/−, select with the camera privacy slider.
5. Factory reset → Apply update → Apply from ADB →
   adb sideload lineage-15.1-XXXX-UNOFFICIAL-starfire.zip
   adb sideload open_gapps-arm-8.1-pico-20220215.zip   (re-select ADB sideload between each)
6. Factory reset again → Reboot.
```

> USB caveats: use a **USB 2.0 port and cable** (black, not blue). USB 3.x hosts commonly cause Sahara errors and mid-flash failures. Flash promptly after entering EDL — waiting too long triggers `Sahara protocol failed`. On Windows, install the Qualcomm HS-USB QDLoader driver and expect Test Mode for unsigned drivers.

## Firmware & Images

| File                                          | What                                                               | Link                                                                                                                                              |
| --------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CD-18781Y.200628.084.zip`                    | Stock "Teams & Others" firmware (known-good base for all flashing) | [wasabisys mirror](https://s3.wasabisys.com/filestash-buk/lenovo-thinksmart-view/CD-18781Y.200628.084.zip)                                        |
| `lineage-15.1-...-starfire.zip`               | Lineage 15.1 system                                                | [wasabisys mirror](https://s3.us-west-1.wasabisys.com/rom-release/LineageOS/15.1/starfire/lineage-15.1-20240531-UNOFFICIAL-starfire.zip)          |
| `lineage-15.1-...-starfire-recovery.img`      | Lineage recovery (flash to `recovery` partition)                   | [wasabisys mirror](https://s3.us-west-1.wasabisys.com/rom-release/LineageOS/15.1/starfire/lineage-15.1-20240531-UNOFFICIAL-starfire-recovery.img) |
| `open_gapps-arm-8.1-pico-20220215.zip`        | Google Play (pico is the largest that fits stock partition)        | [OpenGApps SourceForge](https://sourceforge.net/projects/opengapps/files/arm/20220215/)                                                           |
| `plasma6-ubuntu-qcom-msm8953.img`             | Ubuntu 24.04 + Plasma Mobile 6 image                               |
| `thinksmart-ubuntu-*.img.xz`                  | Voice-assistant Ubuntu image (xiaozhi, no desktop)                 | [GitHub Releases](https://github.com/justhalfbit/thinksmart-ubuntu-image/releases)                                                                |     | Link in the ubuntu-thinksmart README (MEGA) |
| `bootstrap-pmos-ssh-generic-qcom-msm8953.img` | SSH-capable PMOS bootstrap for GPT rewrite                         | [GitHub Release](https://github.com/rickx/ubuntu-thinksmart/releases) (asset `bootstrap-2026-05-22`)                                              |
| `lk2nd.img`                                   | lk2nd secondary bootloader                                         | [prebuilt](https://github.com/rickx/ubuntu-thinksmart/tree/main/prebuilt)                                                                         |
| Rescue & Smart Assistant (RSA)                | Lenovo's official tool to restore stock firmware                   | [Lenovo Support](https://support.lenovo.com)                                                                                                      |

## Source Code & Development

- [Starfire-development/android_kernel_lenovo_apq8053](https://github.com/Starfire-development/android_kernel_lenovo_apq8053) - Kernel source for the Lineage 15.1 `starfire` build.
- [Starfire-development/android_device_lenovo_starfire](https://github.com/Starfire-development/android_device_lenovo_starfire) - Android device tree (audio, BT, GPS, keylayout, lights, overlays, sepolicy).
- [Starfire-development/proprietary_vendor_lenovo_starfire](https://github.com/Starfire-development/proprietary_vendor_lenovo_starfire) - Vendor blobs (some graphics/audio/DRM/gatekeeper from Motorola G6 "ali", rest from stock).
- [rickx/ubuntu-thinksmart sources](https://github.com/rickx/ubuntu-thinksmart/tree/main/sources) - Ubuntu port sources: TAS5782M kernel driver, ath10k patches, kernel/DTS patches, userspace daemons (idle blanker, proximity wake, auto-rotation), build/deploy scripts.
- [msm8953-mainline/linux](https://github.com/msm8953-mainline/linux) - Mainline Linux kernel for MSM8953 devices (build base for the Ubuntu port).
- [msm8953-mainline/lk2nd](https://github.com/msm8953-mainline/lk2nd) - lk2nd second-stage bootloader used to boot mainline from the `boot` partition.
- [msm8953-mainline/alsa-ucm-conf](https://github.com/msm8953-mainline/alsa-ucm-conf) - ALSA UCM2 audio profiles.
- [msm8953-mainline/linux-mdss-dsi-panel-driver-generator](https://github.com/msm8953-mainline/linux-mdss-dsi-panel-driver-generator) - Generator for DSI panel drivers.
- [postmarketOS pmaports](https://gitlab.postmarketos.org/postmarketOS/pmaports) - PMOS ports tree; contains the `device-lenovo-starfire` package.
- Lenovo open-source Android kernel release (`lenovo_cd-18781y` series) - Kernel sources published by Lenovo for this device (mirrored/referenced in community repos).

## Tools

- [bkerler/edl](https://github.com/bkerler/edl) - Open-source Qualcomm EDL/Sahara/Firehose flashing tool (`edl`, `edl.py`). The Linux equivalent of QFIL.
- QPST / QFIL - Qualcomm's official Windows flashing suite (with the HS-USB QDLoader driver). Install with Test Mode for unsigned drivers.
- ADB & Fastboot++ - Friendly Windows ADB/fastboot installer used in the guides.
- [Magisk](https://github.com/topjohnwu/Magisk) - Root for the stock-Android route.
- [OpenGApps](https://opengapps.org) - Google Apps packages (use `arm` 8.1, pico or nano).
- [Android System WebView APKs](https://www.apkmirror.com) - Version 125.x supports Android 8.1; noticeably speeds up HA dashboards. Also installable via the WebView Manager Magisk module.
- [Fully Kiosk Browser](https://www.fully-kiosk.com) - The kiosk browser most people run for HA dashboards (screensaver, motion detection, REST API, WebView selection).
- [scrcpy](https://github.com/Genymobile/scrcpy) - Mirror/control the device over ADB, handy during setup.
- Day-2 management: `adb` on Android; `ssh` / `nmtui` on Ubuntu (USB networking at `172.16.42.1`).

## Home Assistant & Smart Home

- [dinki/View-Assist](https://github.com/dinki/View-Assist) - The flagship project that popularized the TSV in the HA world: voice-assistant satellite dashboards with views for timers, weather, camera streams, and more. Runs on Android (Fully Kiosk) and ESPHome satellites.
- [home-companion](https://github.com/m0s4ik/home-companion) - Native (Rust) Home Assistant voice satellite with a screen, built for the TSV on ThinkSmart View Debian: on-device wake word ("Okay Alfred") and an animated butler character, local timers, alarms and reminders that work offline, HA's own dashboard full screen, AirPlay music and screen mirroring. Set up from the screen with a QR code (Wyoming satellite and HA's built-in MCP client, no add-ons). Italian and English.
- Fully Kiosk Browser + HA companion dashboards - The most common TSV setup (see pgale's guide for the recommended Android settings: no screen lock, no battery icon, volume buttons wake, auto-rotate toggle bug workaround).
- [My take on a home dashboard running on the TSV (Fully Kiosk multi-device management)](https://community.home-assistant.io/t/my-take-on-a-home-dashboard-running-on-the-lenovo-thinksmart-view-and-managing-multiple-devices-running-fully-kiosk/718355) - Dashboard design + managing a fleet of TSVs.

## Projects & Use Cases

- pgale's seven-device household - The author of the flashing guide runs seven TSVs as whole-house HA display panels (details in the guide repo above).
- [samyeei/thinksmart-learning-station](https://github.com/samyeei/thinksmart-learning-station) - Turn a TSV into a family learning terminal: official ChatGPT app with manual Chinese (Fcitx) keyboard toggle, offline dashboard (clock, Wi-Fi, daily sentence), MusicFree music entry, and a LAN-only parent-control web page (screen-time estimate, on-demand screenshot, password management). MIT-licensed source, alpha quality.
- dinki/View-Assist - HA voice satellite with visual feedback (see the Smart Home section).
- Home dashboard / wall panel - The classic use (see the guides above).
- Linux touch panel - Ubuntu/PMOS images make it a generic ARM64 tablet (USB networking + SSH for headless use).
- Camera monitor / RTSP viewer - Venus hardware decode handles 1080p H.264 camera streams (encode without B-frames: `-bf 0`).
- Digital photo frame / kiosk / info display - Fully Kiosk screensavers or a plain browser in PMOS/Ubuntu.
- Local voice assistant - The xiaozhi-based Ubuntu image turns the TSV into an always-on, wake-word voice assistant with on-device ASR and a choice of LLM backends (repo linked in the Operating Systems section).

## Hardware Hacking & Research

Deep-dives from [rickx/ubuntu-thinksmart research](https://github.com/rickx/ubuntu-thinksmart/tree/main/research) (excellent reading if you want to contribute):

- WiFi GTK rekey fix - QCA9379 firmware drops keys with `-110` errors on rekey; fixed by a patched `ath10k_core.ko` that swallows `-ETIMEDOUT`. `cryptmode=1` is not a workaround on this firmware (no raw-mode).
- Audio (TAS5782M) - Custom I2S Class-D driver + DSP firmware + UCM2 HiFi profile + WirePlumber policy; S16LE must be forced (S24 sounds near-silent, S32 rejected). KCFI kernel requires clang-built modules.
- Screen blanking on Plasma 6 - DSI panel can't survive DRM Off; powerdevil timeouts must stay 0. Three userspace daemons handle it: `idle-blanker`, `vcnl4200-proximity-daemon` (hand-approach wake), `rotation-daemon` (BMA253 auto-rotation).
- Hardware video decode - Venus V4L2 M2M: H.264 1080p 1.26× realtime, HEVC 1080p 5.7×; no B-frame support.
- Camera - Live hardware points to Samsung S5KC505A; bring-up in progress under `research/camera/`.
- Proximity sensor (VCNL4200) - Userspace daemon approach; don't use the upstream vcnl4000 driver (drives IR LED at full power continuously).

## Troubleshooting

| Symptom                                               | Fix                                                                                                            |
| ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| QFIL: `Sahara` error / flash pauses ~1 min then fails | Use USB 2.0 port + cable; start download immediately after EDL; disable driver signing (Test Mode)             |
| QFIL doesn't see the port                             | Reinstall HS-USB QDLoader driver; check Device Manager; secure boot can block the driver (disable temporarily) |
| `adb: sideload connection failed` / `E1001`           | ADB sideload times out fast — re-select it right before each sideload; reboot to recovery and retry            |
| Lineage setup hangs at "Checking for updates"         | Press Back after a few seconds (another check page slides in behind; Back at that moment continues setup)      |
| Lineage Wi-Fi drops until reboot                      | Toggle Wi-Fi off/on; running Fully Kiosk seems to prevent it; some users schedule a nightly kiosk restart      |
| Rotation stuck on Lineage                             | Toggle auto-rotate off/on (known bug, no permanent fix yet)                                                    |
| Stock firmware keeps resetting your settings          | Uninstall the MDM/kiosk/managed-settings packages: `adb shell pm uninstall -k --user 0 <pkg>`                  |
| Ubuntu: WiFi disconnects on GTK rekey                 | Already fixed in the image (patched ath10k_core.ko); a dmesg warning line may remain — ignore it               |
| Ubuntu: screen won't wake after sleep                 | Never enable powerdevil display timeouts (DSI DRM Off is unrecoverable); use the built-in daemons              |
| Bricked / want to go back to stock                    | Lenovo Rescue & Smart Assistant (USB 2.0 port again)                                                           |

## Community

- XDA Forums ThinkSmart View root/unlock thread - Where the firmware leak and first mods landed.
- [Home Assistant Community — TSV ROM/OS development thread](https://community.home-assistant.io/t/lenovo-thinksmart-view-rom-os-development/676324) - The main living discussion.
- Home Assistant & View Assist Discord, postmarketOS Matrix/IRC - Where @deadman, @electimon, FelixKa and co hang out.
- Reddit (r/homeassistant, r/selfhosted) - Periodic "cheap Teams display" posts with real-world experience.

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](https://github.com/r4ym0n/awesome-thinksmart/blob/main/CONTRIBUTING.md). In short: only add things that work or are genuinely informative, link to the source (not a repost), use the existing format, and note if a resource is unofficial/abandoned.

## Footnotes

This list is unofficial and not affiliated with or endorsed by Lenovo, Microsoft, Google, or the LineageOS project. All custom ROMs and mods are community efforts with no warranty and no support — you can brick the device, and nobody is obligated to help you. The ThinkSmart View is a discontinued product; flashing it is entirely at your own risk. Several linked files come from community mirrors — verify checksums where provided and scan anything you download.
