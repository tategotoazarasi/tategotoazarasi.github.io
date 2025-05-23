---
date: '2025-05-24T21:32:30+08:00'
draft: false
summary: '记录了在联想 ThinkBook 16 G7+ ASP (AMD Ryzen AI 9) 笔记本上解决 Linux 系统下数字麦克风无声问题的详细过程，主要通过添加内核模块参数 `options snd_sof_amd_common enable_pdm=1` 实现'
tags: [ "linux-audio", "thinkbook-16-g7-asp", "amd-ryzen-ai", "dmic-no-sound", "digital-microphone-fix", "pipewire-troubleshooting", "wireplumber-acp-error", "alsa-ucm", "snd-sof-amd-common", "enable-pdm", "kernel-module-options", "cachyos", "arch-linux-audio", "amd-acp-audio", "sound-open-firmware", "sof-firmware", "strix-point-apu", "linux-laptop-mic-fix", "linux", "audio", "thinkbook", "amd", "ryzen-ai", "dmic", "microphone", "pipewire", "wireplumber", "alsa", "sof", "kernel", "troubleshooting", "fix", "cachyos", "arch-linux", "laptop" ]
title: '解决 ThinkBook 16 G7+ ASP 在 Linux 系统下数字麦克风的顽固问题'
---

对于许多 Linux 爱好者来说，这是一个熟悉的故事：拆开一台崭新笔记本电脑包装时的兴奋，安装好心仪发行版的热切期待，然后……就是那些令人头疼的小问题。有时是
Wi-Fi，有时是睡眠/唤醒，而很多时候，是音频问题，尤其是麦克风。我最近入手了一台联想 ThinkBook 16 G7+ ASP，搭载 AMD Ryzen AI 9
365（属于 "Strix Point" 系列，供参考）处理器，运行着 CachyOS（一个基于 Arch 的发行版），内核版本为
6.14.8-2-cachyos。然而，它内置的数字麦克风阵列却对我“沉默以待”。

如果你也面临类似问题，尤其是在较新的 AMD 硬件上，我希望我的这段经历能提供一些线索，或者至少，让你感受到并非孤军奋战。

## 这玩意儿开着吗？—— 初步侦察

任何故障排除的第一步都是收集信息。系统*认为*它拥有什么？

### 内核视角 (ALSA)

在大多数用户空间工具可以访问的最底层，我们有 ALSA（Advanced Linux Sound Architecture，高级 Linux 声音架构）。命令
`arecord -l` 会列出 ALSA 所能识别的捕获硬件设备：

```
**** List of CAPTURE Hardware Devices ****
card 1: Generic_1 [HD-Audio Generic], device 0: ALC257 Analog [ALC257 Analog]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 2: acppdmmach [acp-pdm-mach], device 0: DMIC capture dmic-hifi-0 []
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

这个结果很有趣，也带来了一些希望。输出显示了两个主要条目。第一个，`card 1: Generic_1 [...] ALC257 Analog`
，被识别为我们的标准模拟音频编解码器，一块 Realtek ALC257。这个组件通常会处理耳机插孔，如果笔记本有模拟麦克风输入的话（许多现代设备仅使用数字阵列麦克风），也会由它负责。第二个条目，
`card 2: acppdmmach [...] DMIC capture`，**看起来就是我们的目标**。“DMIC” 显然代表数字麦克风，而 "acp-pdm-mach" 暗示它通过
AMD 的音频协处理器 (ACP) 使用脉冲密度调制 (PDM) 连接，这是一种数字麦克风的常用接口。所以，ALSA 似乎意识到了数字麦克风的存在。这是一个好的开始。

为了完整起见，`aplay -l` 显示了播放设备：

```
**** List of PLAYBACK Hardware Devices ****
card 0: Generic [HD-Audio Generic], device 3: HDMI 0 [HDMI 0]
  ... (其他 HDMI 输出) ...
card 1: Generic_1 [HD-Audio Generic], device 0: ALC257 Analog [ALC257 Analog]
  Subdevices: 0/1
  Subdevice #0: subdevice #0
```

声卡 0 是来自 AMD GPU 的 HDMI 音频输出，而声卡 1 是通过 ALC257（扬声器、耳机）的模拟输出。这一切看起来都很正常。

### 声音服务器视角 (PipeWire)

这个结果很有趣，也带来了一些希望。

现代 Linux 桌面主要使用 PipeWire（通常与 WirePlumber 作为会话管理器一起）来处理音频和视频流。该系统为 PulseAudio 和 JACK
应用程序提供了兼容层。为了了解 PipeWire 的视角，我使用了 `pactl list cards` 命令。

输出显示了 PipeWire 所见的几个重要“声卡”。第一个，`Card #42: alsa_card.pci-0000_65_00.1`，名为 `HD-Audio Generic` (其
`alsa.card_name`)，并通过其 `device.product.name` 更具体地标识为 `Rembrandt Radeon High Definition Audio Controller`
。这显然对应于 ALSA 的 `card 0`。它列出了各种 HDMI 输出，但值得注意的是，它有 `sources: 0`，这很合理，因为 HDMI 音频通常是仅输出路径。

PipeWire 输出的第二个条目，`Card #43: alsa_card.pci-0000_65_00.6`，其 `alsa.card_name` 也被指定为 `HD-Audio Generic`。然而，它的
`device.product.name` 是 `Family 17h/19h/1ah HD Audio Controller`，其 `alsa.mixer_name` 是 `Realtek ALC257`。这张卡匹配
ALSA 的 `card 1`。其活动配置文件报告为 `HiFi (Mic1, Mic2, Speaker)`。深入其 `Ports` 部分，PipeWire 列出了一个
`[Out] Speaker`（扬声器）和一个 `[Out] Headphones`（耳机）端口，后者在未插入耳机时为 `not available`（不可用）。对于输入，它显示了一个
`[In] Mic2: Stereo Microphone`（立体声麦克风），可能与耳机插孔相关，并且在未连接设备时也为 `not available`，以及至关重要的一个
`[In] Mic1: Digital Microphone`（数字麦克风），其可用性标记为 `unknown`（未知）。

在这个与 ALC257 关联的声卡 (Card #43) 下出现 "Mic1: Digital Microphone"，最初让人有些困惑。尚不清楚 DMIC 是通过 ALC257
编解码器路由，还是这仅仅是 PipeWire 和 WirePlumber 根据 ALSA Use Case Manager (UCM) 配置文件对这些功能进行分组的方式。引人注目的是，ALSA
识别为 `card 2` 并且很可能是 DMIC 的 `acppdmmach` 设备，并没有直接作为顶层“声卡”列在 `pactl list cards`
的输出中。这是一个重要的信号，表明尽管 ALSA 可能暴露了该设备，但 PipeWire 可能未能正确初始化或解释它，从而无法将其呈现为一个完全独立的声卡。

### PCI 设备识别

为了更清楚地了解底层硬件，我使用了 `lspci | grep Audio`。该命令确认了系统中存在的与音频相关的 PCI 设备：

```
65:00.1 Audio device: Advanced Micro Devices, Inc. [AMD/ATI] Rembrandt Radeon High Definition Audio Controller
65:00.5 Multimedia controller: Advanced Micro Devices, Inc. [AMD] ACP/ACP3X/ACP6x Audio Coprocessor (rev 70)
65:00.6 Audio device: Advanced Micro Devices, Inc. [AMD] Family 17h/19h/1ah HD Audio Controller
```

输出结果细分如下：PCI 地址为 `65:00.1` 的设备被识别为 HDMI 音频控制器，隶属于 AMD Radeon 显卡。地址为 `65:00.5` 的设备是
AMD 音频协处理器 (ACP)，具体为修订版 70；这是主要负责处理数字麦克风 (DMIC) 的组件。最后，地址为 `65:00.6` 的设备是模拟音频控制器，它与
Realtek ALC257 编解码器接口，用于扬声器和耳机插孔。此信息与 `arecord -l` 和 `pactl list cards` 的提示完全一致：DMIC 的功能无疑与
ACP 相关联。

### 系统软件检查

快速检查已安装的软件包，确认我拥有现代 Linux 音频设置的常用组件。核心 PipeWire 堆栈，包括 `pipewire`、`pipewire-alsa`、
`pipewire-pulse` 和 `wireplumber` 都已安装。ALSA 的基本组件，如 `alsa-lib`、`alsa-utils` 和 `alsa-card-profiles`
也已安装。至关重要的是，我拥有相当新版本的必要固件包：`linux-firmware` (版本 20250508) 和 `sof-firmware` (Sound Open
Firmware，版本 2025.01.1)。`sof-firmware` 软件包对于现代 Intel 和 AMD 音频硬件尤其重要，特别是对于通过像 AMD 的 ACP
这样的协处理器连接的设备。

在这个阶段，初步侦察表明 ALSA 知道 DMIC 设备的存在。PipeWire 似乎在其配置中承认了一个“数字麦克风”端口，但尚不清楚此端口是否与
ACP 专用的 `acppdmmach` 设备正确关联。硬件组件清晰可见，核心音频软件和固件也已安装。尽管如此，内置麦克风仍然顽固地保持沉默。

## 日志与配置探查

是时候深入研究日志和更深层次的配置细节了。

### 内核消息 (dmesg)

最初，我尝试使用 `sudo dmesg | grep -iE 'acp|dmic|snd_pci_acp|snd_sof_amd'`，但没有得到任何输出。这令人困惑。`dmesg` 应该总会显示
*一些*内容。这可能是我过滤方式的问题，或者相关消息在启动后很快就从缓冲区中滚出去了。我暗自记下，稍后尝试更广泛的 `dmesg`
搜索，或者检查完整的 `journalctl -k`。这里*没有*明确的错误消息本身也是一条信息——至少对于这些特定术语，没有明显的驱动程序崩溃或加载失败，至少
`grep` 最初没有捕捉到。

### ALSA Use Case Manager (UCM)

ALSA UCM 文件描述了设备、端口和配置文件的预期使用方式。它们对于 PipeWire/WirePlumber 理解复杂的音频硬件至关重要。
由于 `pactl list cards` 将 "Mic1: Digital Microphone" 与 Card #43 (ALSA card 1，即 ALC257) 相关联，我转储了它的 UCM：
`alsaucm -c hw:1 dump text` (其中 `hw:1` 指的是 ALSA card 1)。

输出在 `Verb.HiFi` 下包含了这个有趣的片段：

```
Device.Mic1 {
        Comment "Digital Microphone"
        Values {
                CaptureCTL "_ucm0001.hw:Generic_1"
                CaptureMixerElem "Mic ACP LED"
                CapturePCM "_ucm0001.hw:acppdmmach"  // 找到了！
                CapturePriority 100
                CaptureSwitch "Mic ACP LED Capture Switch"
                PlaybackCTL "_ucm0001.hw:Generic_1"
                TQ HiFi
        }
}
```

`CapturePCM "_ucm0001.hw:acppdmmach"` 这一行是关键。它明确指出 UCM 配置文件的 "Mic1" (数字麦克风) 期望使用名为
`acppdmmach` 的 ALSA PCM 设备。这个设备在 `arecord -l` 的输出中被列为 `card 2`。因此，ALC257 (card 1) 的 UCM 配置*引用*了
ACP 的 DMIC (card 2) 作为其“数字麦克风”输入。这解释了为什么“数字麦克风”会出现在 PipeWire 中 ALC257 的声卡下——它遵循了 UCM
的逻辑。

这进一步证实了 `acppdmmach` 需要完全正常工作，并被更高层正确解释。

### WirePlumber 的状态

WirePlumber 是会话管理器，它负责制定 PipeWire 如何连接事物的许多决策。它的日志非常宝贵。
`journalctl -b --user -u wireplumber` 揭示了确凿的证据：

```
May 24 19:39:01 wangzhiheng wireplumber[1808]: wp-device: SPA handle 'api.alsa.acp.device' could not be loaded; is it installed?
May 24 19:39:01 wangzhiheng wireplumber[1808]: s-monitors: Failed to create 'api.alsa.acp.device' device
```

就是它了。WirePlumber 明确指出无法加载或创建名为 `api.alsa.acp.device` 的东西。SPA 代表简单插件 API (Simple Plugin API)
，PipeWire 使用它来实现插件。这强烈暗示，尽管 ALSA 知道 `acppdmmach` (card 2) 的存在，但 WirePlumber 无法正确地与 ACP
设备接口，以使其 DMIC 功能作为源可用。

WirePlumber 日志中的其他错误，如
`<WpAsyncEventHook:0x64962e406260> failed: <WpSiStandardLink:0x64962e7914f0> Object activation aborted`
，很可能是这个主要故障的下游后果。如果 ACP 设备没有正确创建，与其相关的链接操作自然会失败。

日志中还提到：
`s-monitors-libcamera: PipeWire's libcamera SPA plugin is missing or broken.`
这与麦克风无关，但或许值得在以后排查摄像头问题时注意。目前，专注于音频。

我还运行了 `tree /usr/share/wireplumber` 来了解其配置结构。我的系统上没有 `/etc/wireplumber`
覆盖目录，并且值得注意的是，在一些在线故障排除指南中提到的常见路径下，并没有 `50-alsa-config.lua` 文件。这意味着 WirePlumber
很可能使用其在 `/usr/share/wireplumber/scripts/` 中的默认配置脚本和主配置文件 `/usr/share/wireplumber/wireplumber.conf`
运行。缺少 `50-alsa-config.lua` 并不一定是一个错误；现代 WirePlumber 版本可能以不同的方式集成其逻辑，或者它可能是一个可选的覆盖文件。

### 完整的 PCI 详细信息与内核模块 (`lspci -nnk`)

该命令是一个宝库，显示 PCI 设备、它们的供应商/设备 ID、当前管理它们的内核驱动程序，以及其他候选模块。

`lspci -nnk` 命令提供了详细的 PCI 信息，包括正在使用的内核驱动程序，当聚焦于 `65:00.5`
的多媒体控制器时，它提供了最具启发性的线索。对于这个特定的设备，即修订版为 70 的 AMD 音频协处理器，系统报告如下：

```
Subsystem: Lenovo Device [17aa:38b3]
Kernel driver in use: snd_acp_pci
Kernel modules: snd_pci_acp3x, snd_rn_pci_acp3x, snd_pci_acp5x, snd_pci_acp6x, snd_acp_pci, snd_rpl_pci_acp6x, snd_pci_ps, snd_sof_amd_renoir, snd_sof_amd_rembrandt, snd_sof_amd_vangogh, snd_sof_amd_acp63, snd_sof_amd_acp70
```

`Kernel driver in use: snd_acp_pci` 这一行确认了通用的 ACP PCI 驱动程序已加载并正确绑定到该设备。然而，`Kernel modules`
这一行则非常引人入胜。这个列表显示了系统认为可以处理此硬件的所有潜在内核模块。关键是，它包含了几个重要的 SOF (Sound Open
Firmware) 驱动程序。其中包括 `snd_sof_amd_rembrandt`，这很合乎逻辑，因为我的 "Strix Point" APU 是 Rembrandt
架构的后续产品。最重要的是，它列出了像 `snd_sof_amd_acp63` 和 `snd_sof_amd_acp70` 这样的特定驱动程序。由于我的 ACP 被识别为
`rev 70`，`snd_sof_amd_acp70` 模块立刻成为提供正确操作 DMIC 所需的专业 SOF 层的非常强有力的候选者。

其他音频设备：

`65:00.1 Audio device [0403]: ...Rembrandt Radeon High Definition Audio Controller [1002:1640]`
`Kernel driver in use: snd_hda_intel` (HDMI 音频的标准驱动)

`65:00.6 Audio device [0403]: ...Family 17h/19h/1ah HD Audio Controller [1022:15e3]`
`Kernel driver in use: snd_hda_intel` (模拟 HDA 编解码器如 ALC257 的标准驱动)

这证实了标准驱动程序已为 HDMI 和模拟音频部分加载。焦点仍然集中在 ACP 以及 `snd_acp_pci` 如何与像 `snd_sof_amd_acp70` 这样的
SOF DSP 驱动程序或更通用的 `snd_sof_amd_common` 交互（或需要交互）。

## 连接线索

好了，总结一下目前收集到的线索，描绘出了一幅相当清晰的图景。首先，ALSA 这个基础声音层正确地将一个 `acppdmmach` 设备识别为
`card 2`，这是我们数字麦克风的主要嫌疑对象。其次，Realtek ALC257 的“数字麦克风”配置文件的 ALSA Use Case Manager (UCM)
配置明确地将这个 `acppdmmach` 设备指向其捕获 PCM。这意味着系统*期望*使用该设备。

然而，在 PipeWire/WirePlumber 层面出现了一个关键问题：WirePlumber 的日志显示无法加载或创建 `api.alsa.acp.device`
。这表明上层声音服务器在尝试与 ACP 硬件接口时发生了故障。在硬件和驱动程序方面，我们知道 ACP 硬件 (`1022:15e2 rev 70`)
存在，并且通用的 `snd_acp_pci` 内核驱动程序已加载并激活。此外，必要的 SOF (Sound Open Firmware) 固件已安装，并且相关的 SOF
内核模块，如 `snd_sof_amd_acp70`，在系统上可用。

这些点强烈暗示，虽然基本组件都已就位，但通用 ACP 驱动程序 (`snd_acp_pci`) 与 DMIC 所需的更专业的 SOF 层之间的交互或初始化序列没有正确发生，导致
WirePlumber 无法正确利用 ACP 设备。

我的假设是：`snd_acp_pci` 驱动程序本身可能不够，或者它的初始化方式没有完全向 WirePlumber 期望的 `api.alsa.acp.device` 所需的
SOF 层暴露 PDM/DMIC 功能。本质上，ACP 中处理 DMIC 阵列的 DSP 部分，可能没有为 PipeWire 的使用而正确“激活”。

这在较新的 AMD（和 Intel）笔记本电脑上是一种常见模式，其中 DMIC 由在音频协处理器上运行的专用 DSP 固件 (SOF)
处理。如果此固件未正确加载，或者驱动程序未配置为将其用于 PDM 麦克风，就会出现静音。

## 修复方案

许多基于 SOF 的驱动程序，尤其是用于 PDM 麦克风的驱动程序，都有模块选项来启用或配置特定功能。一个常见选项与启用 PDM 麦克风支持有关。

考虑到 `lspci -nnk` 输出的内核模块列表，特别是 `snd_sof_amd_acp70`，以及一个更通用的 `snd_sof_amd_common` 模块的存在（该模块通常作为各种
AMD ACP SoF 驱动程序的包装器或通用代码库），我搜索了与这些模块相关的选项。

根据社区论坛和错误报告，一个经常被建议用于解决 AMD ACP DMIC 问题的修复方法是使用 `enable_pdm`
内核模块选项。主要问题是，这个选项应该针对哪个特定模块？可能的选项包括 `snd_sof_amd_acp70`，这是针对我的 ACP 修订版的更具体的
SOF 驱动程序；`snd_sof_amd_common`，它通常在更高度定制的驱动程序完全成熟或进入主线之前，充当较新 AMD 平台的通用代码库或伞形结构；甚至
`snd_acp_pci` 本身，尽管这种可能性较小，因为 `enable_pdm` 通常是 SOF 特有的功能。对于近期的 AMD 平台，普遍的看法通常指向使用
`snd_sof_amd_common` 来启用 PDM 麦克风支持。

因此，提议的解决方案是添加这个内核模块选项。第一步是创建一个新的配置文件，例如，通过运行
`sudo nano /etc/modprobe.d/99-thinkbook-mic-fix.conf`。`99-` 前缀有助于确保此配置加载较晚，尽管简单 `options`
行的加载顺序通常不重要，除非存在直接冲突；然而，`.conf` 后缀是系统识别该文件所必需的。

在这个新文件中，需要添加的关键行是：

```
options snd_sof_amd_common enable_pdm=1
```

这条指令告诉 `snd_sof_amd_common` 内核模块在系统启动加载时明确启用 PDM 麦克风支持。对于这个布尔选项，值 `1` 等同于 `true`。

保存此配置文件后，下一个关键步骤是重建 initramfs (初始 RAM 文件系统)。模块选项可能会影响设备在引导过程的早期被探测的方式，因此更新
initramfs 可以确保这些新选项在该阶段可用。在像我的 CachyOS 安装这样的基于 Arch 的系统上，这通常通过以下命令完成：

```bash
sudo mkinitcpio -P
```

该命令会重建所有预设的 initramfs 镜像，并包含新的 modprobe 配置。

最后，需要**完全重启系统**。这使得内核能够加载新的模块选项，从而可能改变音频硬件的初始化方式。这种方法感觉是一个很有希望的修复方案，因为它直接解决了数字麦克风阵列的
PDM (脉冲密度调制) 问题，针对了一个相关的 SOF 模块 (`snd_sof_amd_common`)，并且是针对 AMD 硬件上类似音频问题的广泛报道的解决方案。

## 验证

重启后，到了见证奇迹的时刻。
我打开了一个录音应用，对着笔记本电脑说话。然后它出现了——输入电平表跳动了起来！麦克风工作了。

`sudo dmesg | grep -Ei 'sof|acp|dmic|snd_sof_amd_common|snd_sof_amd_acp70'`

```
[    0.000000] BIOS-e820: [mem 0x0000000009f00000-0x0000000009f37fff] ACPI NVS
... (许多 ACPI 表行) ...
[    0.411425] ACPI: \_SB_.PCI0.GPPA.ACP_.PWRS: New power resource  // ACPI 中定义的 ACP 电源资源
...
[    5.676187] snd_acp_pci 0000:65:00.5: enabling device (0000 -> 0002) // snd_acp_pci 驱动程序启用设备。
```

这里的关键行是 `[ 5.676187] snd_acp_pci 0000:65:00.5: enabling device (0000 -> 0002)`。这表明通用的 ACP PCI 驱动程序确实在初始化硬件。

理想情况下，我们*希望*在成功的基于 SOF 的 DMIC 初始化（`enable_pdm=1` 选项旨在触发此初始化）后，在 `dmesg`
输出中看到更具体的日志行。这些可能包括来自 `sof-audio-pci-intel`（或其 AMD 等效模块，如 `snd_sof_amd_common` 或
`snd_sof_amd_acp70`）的消息，表明它们已成功探测或初始化了数字信号处理器 (DSP)。我们也可能寻找确认 SOF 驱动程序检测到 PDM
设备或 DMIC 的行。此外，指示 `acp-pdm-mach` ALSA 设备现在正由 SOF 层注册的日志，将是成功初始化序列的有力证据。

这次故障排除虽然具体针对一台搭载 AMD "Strix Point" APU 的 ThinkBook 16 G7+ ASP，但它突显了 Linux
音频问题解决中的几个共同主题。音频堆栈的层级复杂性，从硬件和内核驱动（ALSA、SOF）到声音服务器（PipeWire）和会话管理器（WirePlumber）再到应用程序，意味着问题可能出现在交互的许多环节。因此，检查日志至关重要：
`dmesg`（或 `journalctl -k`）用于内核消息，用户级服务日志用于 WirePlumber 和 PipeWire，这些都是不可或缺的工具。确保固件，特别是
`linux-firmware` 和 `sof-firmware` 的更新，对于现代系统而言是不容忽视的。ALSA UCM 文件在 PipeWire
如何理解复杂音频设备方面也扮演着至关重要的角色，虽然它们有时可能需要针对新硬件进行修补，但在本例中 UCM 似乎是正确的。通过
`/etc/modprobe.d/` 配置的内核模块参数是启用功能或解决硬件怪癖的强大工具，尽管找到正确的模块和选项通常需要一番研究。专用
DSP 和音频协处理器（如 AMD 的 ACP，运行 SOF 固件以处理 DMIC 阵列等任务）日益普及，这引入了另一个需要正常工作的层面；
`enable_pdm=1` 选项正是这种架构转变的直接结果。此外，ACPI 表对操作系统如何发现和配置硬件（包括音频组件）有重大影响。最后，Linux
社区在论坛、维基和错误跟踪器中积累的集体智慧是巨大的资源。如果应用的修复方法，例如
`options snd_sof_amd_common enable_pdm=1`，未能解决问题，接下来的步骤将包括尝试对更具体的模块（如 `snd_sof_amd_acp70`）使用
`enable_pdm=1` 选项，寻找完全不同的模块选项，测试更新的内核版本（因为驱动程序支持在不断改进），检查笔记本电脑制造商提供的
BIOS/UEFI 更新，或者作为最后手段，向相关的上游项目提交详细的错误报告。鉴于这款 ThinkBook 型号及其 APU
都相当新，最新的硬件通常需要此类有针对性的调整，直到更广泛的 Linux 支持完全成熟，并且这些配置成为默认设置或集成到 UCM
配置文件中。