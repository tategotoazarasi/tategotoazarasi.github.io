---
date: '2025-05-24T21:32:30+08:00'
draft: false
summary: 'Detailed troubleshooting process for fixing a silent digital microphone on a Lenovo ThinkBook 16 G7+ ASP (AMD Ryzen AI 9) laptop running Linux, primarily resolved by adding the kernel module parameter `options snd_sof_amd_common enable_pdm=1`.'
tags: [ "linux-audio", "thinkbook-16-g7-asp", "amd-ryzen-ai", "dmic-no-sound", "digital-microphone-fix", "pipewire-troubleshooting", "wireplumber-acp-error", "alsa-ucm", "snd-sof-amd-common", "enable-pdm", "kernel-module-options", "cachyos", "arch-linux-audio", "amd-acp-audio", "sound-open-firmware", "sof-firmware", "strix-point-apu", "linux-laptop-mic-fix", "linux", "audio", "thinkbook", "amd", "ryzen-ai", "dmic", "microphone", "pipewire", "wireplumber", "alsa", "sof", "kernel", "troubleshooting", "fix", "cachyos", "arch-linux", "laptop" ]
title: 'Troubleshooting a Stubborn DMIC on a ThinkBook 16 G7+ ASP with Linux'
---

It's a familiar story for many Linux enthusiasts: the thrill of unboxing a shiny new laptop, the eager anticipation of
installing your favorite distribution, and then... the little papercuts. Sometimes it's Wi-Fi, sometimes suspend/resume,
and very often, it's the audio, particularly the microphone. My recent acquisition, a Lenovo ThinkBook 16 G7+ ASP
powered by an AMD Ryzen AI 9 365 (part of the "Strix Point" family, for those keeping score), running CachyOS (an
Arch-based distribution) with kernel 6.14.8-2-cachyos, decided to give me the silent treatment from its built-in digital
microphone array.

If you're facing a similar issue, especially on recent AMD hardware, I hope my odyssey provides some clues, or at least,
solidarity.

## Is This Thing On?

The first step in any troubleshooting saga is to gather information. What does the system *think* it has?

### The Kernel's Perspective (ALSA)

At the lowest level accessible to most user-space tools, we have ALSA (Advanced Linux Sound Architecture). The command
`arecord -l` lists capture hardware devices as ALSA sees them:

```
**** List of CAPTURE Hardware Devices ****
card 1: Generic_1 [HD-Audio Generic], device 0: ALC257 Analog [ALC257 Analog]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 2: acppdmmach [acp-pdm-mach], device 0: DMIC capture dmic-hifi-0 []
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

This was interesting and somewhat promising. The output showed two main entries. The first,
`card 1: Generic_1 [...] ALC257 Analog`, was identified as our standard analog audio codec, a Realtek ALC257. This
component would typically handle headphone jacks and, if the laptop had one, an analog microphone input, though many
modern devices exclusively use digital arrays. The second entry, `card 2: acppdmmach [...] DMIC capture`, immediately
looked like **our target**. The "DMIC" clearly stands for Digital Microphone, and "acp-pdm-mach" suggested its
connection via AMD's Audio Co-Processor (ACP) using Pulse Density Modulation (PDM), a common interface for digital
microphones. So, ALSA seemed to be aware of a digital microphone. That's a good start.

For completeness, `aplay -l` shows playback devices:

```
**** List of PLAYBACK Hardware Devices ****
card 0: Generic [HD-Audio Generic], device 3: HDMI 0 [HDMI 0]
  ... (other HDMI outputs) ...
card 1: Generic_1 [HD-Audio Generic], device 0: ALC257 Analog [ALC257 Analog]
  Subdevices: 0/1
  Subdevice #0: subdevice #0
```

Card 0 is the HDMI audio output from the AMD GPU, and Card 1 is the analog output via the ALC257 (speakers, headphones).
This all seemed normal.

### The Sound Server's Perspective (PipeWire)

This was interesting and somewhat promising.

Modern Linux desktops predominantly use PipeWire, often with WirePlumber as the session manager, to handle audio and
video streams. This system provides compatibility layers for PulseAudio and JACK applications. To understand PipeWire's
perspective, I used the `pactl list cards` command.

The output revealed a couple of important "cards" as seen by PipeWire. The first,
`Card #42: alsa_card.pci-0000_65_00.1`, was named `HD-Audio Generic` (its `alsa.card_name`) and more specifically
identified by its `device.product.name` as `Rembrandt Radeon High Definition Audio Controller`. This clearly
corresponded to ALSA's `card 0`. It listed various HDMI outputs but, notably, had `sources: 0`, which is logical as HDMI
audio is typically an output-only path.

The second entry from PipeWire, `Card #43: alsa_card.pci-0000_65_00.6`, was also designated as `HD-Audio Generic` by its
`alsa.card_name`. However, its `device.product.name` was `Family 17h/19h/1ah HD Audio Controller`, and its
`alsa.mixer_name` was `Realtek ALC257`. This card matched ALSA's `card 1`. Its active profile was reported as
`HiFi (Mic1, Mic2, Speaker)`. Delving into its `Ports` section, PipeWire listed an `[Out] Speaker` and an
`[Out] Headphones` port, the latter being `not available` unless headphones were plugged in. For inputs, it showed an
`[In] Mic2: Stereo Microphone`, possibly associated with the headphone jack and also `not available` unless a device was
connected, and, crucially, an `[In] Mic1: Digital Microphone` whose availability was marked as `unknown`.

The presence of "Mic1: Digital Microphone" under this ALC257-associated card (Card #43) was initially a bit perplexing.
It wasn't immediately clear if the DMIC was routed *through* the ALC257 codec or if this was simply how PipeWire and
WirePlumber decided to group these functionalities based on ALSA Use Case Manager (UCM) profiles. What stood out was
that the `acppdmmach` device, which ALSA identified as `card 2` and the likely candidate for the DMIC, wasn't directly
listed as a distinct top-level "Card" in the `pactl list cards` output. This was a significant flag, suggesting that
while ALSA might expose the device, PipeWire might not be initializing or interpreting it correctly to present it as a
fully independent audio card.

### PCI Device Identification

To get a clearer picture of the underlying hardware, I used `lspci | grep Audio`. This command confirmed the
audio-related PCI devices present in the system:

```
65:00.1 Audio device: Advanced Micro Devices, Inc. [AMD/ATI] Rembrandt Radeon High Definition Audio Controller
65:00.5 Multimedia controller: Advanced Micro Devices, Inc. [AMD] ACP/ACP3X/ACP6x Audio Coprocessor (rev 70)
65:00.6 Audio device: Advanced Micro Devices, Inc. [AMD] Family 17h/19h/1ah HD Audio Controller
```

The output broke down as follows: the device at PCI address `65:00.1` was identified as the HDMI audio controller, part
of the AMD Radeon graphics. The device at `65:00.5` was the AMD Audio Co-Processor (ACP), specifically revision 70; this
is the component primarily responsible for handling the digital microphone (DMIC). Finally, the device at `65:00.6` was
the analog audio controller, which interfaces with the Realtek ALC257 codec for speakers and headphone jacks. This
information aligned perfectly with what `arecord -l` and `pactl list cards` were suggesting: the DMIC's functionality
was undeniably tied to the ACP.

### System Software Check

A quick check of installed packages confirmed I had the usual suspects for a modern Linux audio setup. The core PipeWire
stack, including `pipewire`, `pipewire-alsa`, `pipewire-pulse`, and `wireplumber`, was present. The ALSA essentials,
such as `alsa-lib`, `alsa-utils`, and `alsa-card-profiles`, were also installed. Crucially, I had fairly recent versions
of the necessary firmware blobs: `linux-firmware` (version 20250508) and `sof-firmware` (Sound Open Firmware, version
2025.01.1). The `sof-firmware` package is particularly important for modern Intel and AMD audio hardware, especially for
devices connected via coprocessors like AMD's ACP.

At this stage, the initial reconnaissance suggested that ALSA was aware of a DMIC device. PipeWire seemed to acknowledge
a "Digital Microphone" port in its configuration, but it wasn't entirely clear if this port was properly associated with
the ACP's dedicated `acppdmmach` device. The hardware components were clearly present, and the core audio software and
firmware were installed. Despite all this, the internal microphone remained stubbornly silent.

## Logs and Configurations

Time to get our hands dirty with logs and deeper configuration details.

### Kernel Messages (dmesg)

Initially, I tried `sudo dmesg | grep -iE 'acp|dmic|snd_pci_acp|snd_sof_amd'` but got no output. This was puzzling.
`dmesg` should always have *something*. This might have been an artifact of how I was filtering or perhaps the relevant
messages had scrolled out of the buffer quickly after boot. I made a mental note to try broader `dmesg` searches later
or check the full `journalctl -k`. The *absence* of explicit error messages here was, in itself, a piece of
information – no obvious driver crashes or failures to load for these specific terms, at least not that `grep` caught
initially.

### ALSA Use Case Manager (UCM)

ALSA UCM files describe how devices, ports, and profiles are meant to be used. They are essential for
PipeWire/WirePlumber to make sense of complex audio hardware.
Since `pactl list cards` associated "Mic1: Digital Microphone" with Card #43 (ALSA card 1, the ALC257), I dumped its
UCM:
`alsaucm -c hw:1 dump text` (where `hw:1` refers to ALSA card 1).

The output contained this interesting snippet under `Verb.HiFi`:

```
Device.Mic1 {
        Comment "Digital Microphone"
        Values {
                CaptureCTL "_ucm0001.hw:Generic_1"
                CaptureMixerElem "Mic ACP LED"
                CapturePCM "_ucm0001.hw:acppdmmach"  // BINGO!
                CapturePriority 100
                CaptureSwitch "Mic ACP LED Capture Switch"
                PlaybackCTL "_ucm0001.hw:Generic_1"
                TQ HiFi
        }
}
```

The line `CapturePCM "_ucm0001.hw:acppdmmach"` was key. It explicitly states that the UCM profile's "Mic1" (Digital
Microphone) expects to use the ALSA PCM device named `acppdmmach`. This device was listed by `arecord -l` as `card 2`.
So, the UCM config for the ALC257 (card 1) *references* the ACP's DMIC (card 2) for its "Digital Microphone" input. This
explained why "Digital Microphone" appeared under the ALC257's card in PipeWire – it was following the UCM logic.

This reinforced that `acppdmmach` needed to be fully functional and correctly interpreted by the higher layers.

### WirePlumber's Sanity

WirePlumber is the session manager that makes many of the decisions about how PipeWire connects things. Its logs are
invaluable.
`journalctl -b --user -u wireplumber` revealed the smoking gun:

```
May 24 19:39:01 wangzhiheng wireplumber[1808]: wp-device: SPA handle 'api.alsa.acp.device' could not be loaded; is it installed?
May 24 19:39:01 wangzhiheng wireplumber[1808]: s-monitors: Failed to create 'api.alsa.acp.device' device
```

There it was. WirePlumber was explicitly failing to load or create something called `api.alsa.acp.device`. SPA stands
for Simple Plugin API, which PipeWire uses for its plugins. This strongly suggested that WirePlumber, despite ALSA
knowing about `acppdmmach` (card 2), couldn't properly interface with the ACP device to make its DMIC functionality
available as a source.

Other errors in the WirePlumber log like
`<WpAsyncEventHook:0x64962e406260> failed: <WpSiStandardLink:0x64962e7914f0> Object activation aborted` were likely
downstream consequences of this primary failure. If the ACP device isn't properly created, linking to/from it will fail.

The logs also mentioned:
`s-monitors-libcamera: PipeWire's libcamera SPA plugin is missing or broken.`
This was unrelated to the microphone but worth noting for camera troubleshooting later, perhaps. For now, focus on
audio.

I also ran `tree /usr/share/wireplumber` to understand its configuration structure. There was no `/etc/wireplumber`
override directory on my system, and notably, no `50-alsa-config.lua` in the common paths mentioned in some online
troubleshooting guides. This meant WirePlumber was likely running with its default configuration scripts found in
`/usr/share/wireplumber/scripts/` and main config `/usr/share/wireplumber/wireplumber.conf`. The absence of
`50-alsa-config.lua` isn't necessarily an error; modern WirePlumber versions might integrate its logic differently or it
might be an optional override file.

### Full PCI Details with Kernel Modules (`lspci -nnk`)

This command is a goldmine, showing PCI devices, their vendor/device IDs, and the kernel driver currently managing them,
plus other candidate modules.

Of course. Here is the revised text in a plaintext paragraph style:

The `lspci -nnk` command, which provides detailed PCI information including the kernel drivers in use, offered the most
revealing clues when focused on the Multimedia Controller at `65:00.5`. For this specific device, the AMD Audio
Co-Processor with revision 70, the system reported:

```
Subsystem: Lenovo Device [17aa:38b3]
Kernel driver in use: snd_acp_pci
Kernel modules: snd_pci_acp3x, snd_rn_pci_acp3x, snd_pci_acp5x, snd_pci_acp6x, snd_acp_pci, snd_rpl_pci_acp6x, snd_pci_ps, snd_sof_amd_renoir, snd_sof_amd_rembrandt, snd_sof_amd_vangogh, snd_sof_amd_acp63, snd_sof_amd_acp70
```

The line `Kernel driver in use: snd_acp_pci` confirmed that the generic ACP PCI driver was loaded and correctly bound to
the device. However, the `Kernel modules` line was truly fascinating. This list showed all the kernel modules that the
system considered potential handlers for this hardware. It critically included several important SOF (Sound Open
Firmware) drivers. Among them was `snd_sof_amd_rembrandt`, which was logical since my "Strix Point" APU is a successor
to the Rembrandt architecture. Most importantly, it listed specific drivers like `snd_sof_amd_acp63` and
`snd_sof_amd_acp70`. Since my ACP was identified as `rev 70`, the `snd_sof_amd_acp70` module immediately stood out as a
very strong candidate for providing the specialized SOF layer needed to properly operate the DMIC.

The other audio devices:

`65:00.1 Audio device [0403]: ...Rembrandt Radeon High Definition Audio Controller [1002:1640]`
`Kernel driver in use: snd_hda_intel` (Standard for HDMI audio).

`65:00.6 Audio device [0403]: ...Family 17h/19h/1ah HD Audio Controller [1022:15e3]`
`Kernel driver in use: snd_hda_intel` (Standard for analog HDA codecs like the ALC257).

This confirmed that the standard drivers were loaded for the HDMI and analog audio parts. The focus remained on the ACP
and how `snd_acp_pci` interacts (or needs to interact) with a SOF DSP driver like `snd_sof_amd_acp70` or a more generic
`snd_sof_amd_common`.

## Connecting the Dots

Okay, summarizing the clues gathered so far painted a fairly clear picture. First, ALSA, the fundamental sound layer,
correctly identified an `acppdmmach` device as `card 2`, which was our prime suspect for the digital microphone. Second,
the ALSA Use Case Manager (UCM) configuration for the Realtek ALC257's "Digital Microphone" profile explicitly pointed
to this `acppdmmach` device for its capture PCM. This meant the system *intended* for that device to be used.

However, a critical issue arose at the PipeWire/WirePlumber level: WirePlumber's logs showed a failure to load or create
an `api.alsa.acp.device`. This indicated a breakdown in how the higher-level sound server was trying to interface with
the ACP hardware. On the hardware and driver side, we knew the ACP hardware (`1022:15e2 rev 70`) was present and the
generic `snd_acp_pci` kernel driver was loaded and active. Furthermore, the necessary SOF (Sound Open Firmware) firmware
was installed, and relevant SOF-related kernel modules, such as `snd_sof_amd_acp70`, were available on the system.

These points strongly suggested that while the basic components were in place, the interaction or initialization
sequence between the generic ACP driver (`snd_acp_pci`) and the more specialized SOF layer needed for the DMIC was not
happening correctly, leading to WirePlumber's inability to properly utilize the ACP device.

My hypothesis: The `snd_acp_pci` driver by itself might not be enough, or it's not being initialized in a way that fully
exposes the PDM/DMIC capabilities to the SOF layer that WirePlumber expects for `api.alsa.acp.device`. Essentially, the
DSP part of the ACP, which handles the DMIC array, might not be "activated" correctly for PipeWire's consumption.

This is a common pattern on newer AMD (and Intel) laptops where DMICs are processed by a dedicated DSP firmware (SOF)
running on an audio coprocessor. If this firmware isn't loaded correctly or the driver isn't configured to use it for
PDM microphones, things go silent.

## The Fix

Many SOF-based drivers, especially for PDM microphones, have module options to enable or configure specific features. A
common one is related to enabling PDM microphone support.

Given the list of kernel modules from `lspci -nnk`, particularly `snd_sof_amd_acp70`, and the existence of a more
generic `snd_sof_amd_common` module that often serves as a wrapper or common codebase for various AMD ACP SoF drivers, I
searched for module options related to these.

A frequently suggested fix for AMD ACP DMIC issues, based on community forums and bug reports, involves using an
`enable_pdm` kernel module option. The main question was, which specific module should this option target? Possibilities
included `snd_sof_amd_acp70`, the more specific SOF driver for my ACP revision; `snd_sof_amd_common`, which often serves
as a common codebase or umbrella for newer AMD platforms before a highly tailored driver is fully mature or mainlined;
or even `snd_acp_pci` itself, though this was less likely as `enable_pdm` is typically a SOF-specific feature. The
prevailing wisdom for recent AMD platforms often points towards using `snd_sof_amd_common` for enabling PDM microphone
support.

Therefore, the proposed solution was to add this kernel module option. The first step was to create a new configuration
file, for instance, by running `sudo nano /etc/modprobe.d/99-thinkbook-mic-fix.conf`. The `99-` prefix can help ensure
this configuration is loaded late, although the loading order for simple `options` lines isn't usually critical unless
there are direct conflicts; the `.conf` suffix is, however, mandatory for the system to recognize the file.

Inside this new file, the critical line to add was:

```
options snd_sof_amd_common enable_pdm=1
```

This instruction tells the `snd_sof_amd_common` kernel module to explicitly enable PDM microphone support when it loads
during system startup. The value `1` is equivalent to `true` for this boolean option.

After saving this configuration file, the next crucial step was to rebuild the initramfs (initial RAM filesystem).
Module options can affect how devices are probed very early in the boot process, so updating the initramfs ensures these
new options are available at that stage. On Arch-based systems like my CachyOS installation, this is typically done with
the command:

```bash
sudo mkinitcpio -P
```

This command rebuilds all preset initramfs images, incorporating the new modprobe configuration.

Finally, a full **reboot of the system** was necessary. This allows the kernel to load with the new module option
active, potentially changing how the audio hardware is initialized. This approach felt like a strong candidate for a fix
because it directly addressed the PDM (Pulse Density Modulation) aspect of the digital microphone array, targeted a
relevant SOF module (`snd_sof_amd_common`), and was a widely reported solution for similar audio problems on AMD
hardware.

## Verification

After the reboot, it was time for the moment of truth.

I opened a voice recorder application and spoke into the laptop. And there it was – the input level meter danced! The
microphone was working.

`sudo dmesg | grep -Ei 'sof|acp|dmic|snd_sof_amd_common|snd_sof_amd_acp70'`

```
[    0.000000] BIOS-e820: [mem 0x0000000009f00000-0x0000000009f37fff] ACPI NVS
... (many ACPI table lines) ...
[    0.411425] ACPI: \_SB_.PCI0.GPPA.ACP_.PWRS: New power resource  // ACP Power Resource defined in ACPI
...
[    5.676187] snd_acp_pci 0000:65:00.5: enabling device (0000 -> 0002) // The snd_acp_pci driver enabling the device.
```

The crucial line here is `[ 5.676187] snd_acp_pci 0000:65:00.5: enabling device (0000 -> 0002)`. This shows the generic
ACP PCI driver is indeed initializing the hardware.

Ideally, what we *would hope* to see in the `dmesg` output after a successful SOF-based DMIC initialization, which the
`enable_pdm=1` option is intended to trigger, are more specific log lines. These might include messages from
`sof-audio-pci-intel` (or its AMD equivalents like `snd_sof_amd_common` or `snd_sof_amd_acp70`) indicating that they
have successfully probed or initialized the Digital Signal Processor (DSP). We might also look for lines confirming the
detection of PDM devices or DMICs by the SOF driver. Furthermore, logs indicating that the `acp-pdm-mach` ALSA device is
now being registered by the SOF layer would be strong evidence of a successful initialization sequence.

This troubleshooting journey, specific to a ThinkBook 16 G7+ ASP with an AMD "Strix Point" APU, underscores several
common themes in Linux audio problem-solving. The audio stack's layered complexity, from hardware and kernel drivers (
ALSA, SOF) through the sound server (PipeWire) and session manager (WirePlumber) to applications, means issues can arise
at many points of interaction. Consequently, examining logs is paramount: `dmesg` (or `journalctl -k`) for kernel
messages, and user-level service logs for WirePlumber and PipeWire, are indispensable. Ensuring up-to-date firmware,
particularly `linux-firmware` and `sof-firmware`, is non-negotiable for modern systems. ALSA UCM files also play a vital
role in how PipeWire interprets complex audio devices, and while they can sometimes require patches for new hardware,
the UCM seemed correct in this instance. Kernel module parameters, configured via `/etc/modprobe.d/`, are powerful tools
for enabling features or addressing hardware quirks, though finding the correct module and option often necessitates
research. The increasing prevalence of dedicated DSPs and audio coprocessors, like AMD's ACP running SOF firmware for
tasks such as DMIC array processing, introduces another layer that must function correctly; the `enable_pdm=1` option is
a direct result of this architectural shift. Furthermore, ACPI tables significantly influence how the OS discovers and
configures hardware, including audio components. Finally, the collective wisdom of the Linux community found in forums,
wikis, and bug trackers is an immense resource. If the applied fix, such as `options snd_sof_amd_common enable_pdm=1`,
hadn't resolved the issue, the next steps would have involved trying the `enable_pdm=1` option with a more specific
module like `snd_sof_amd_acp70`, searching for entirely different module options, testing newer kernel versions (as
driver support continually improves), checking for BIOS/UEFI updates from the laptop manufacturer, or, as a last resort,
filing detailed bug reports with the relevant upstream projects. Given that this ThinkBook model and its APU are quite
new, it's not uncommon for the latest hardware to require such targeted adjustments until broader Linux support fully
matures and these configurations become default or are integrated into UCM profiles.