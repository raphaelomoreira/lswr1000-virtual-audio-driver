> [!WARNING]
> This software is in early development and currently requires test signing to be enabled
> (`bcdedit /set testsigning on`). Production code signing (via the SignPath Foundation's free
> open-source program) is planned but not yet in place — see "Signing status" below.

# LSWR 1000 Virtual Audio Driver

A Windows virtual speaker and virtual microphone driver for the LSWR 1000 broadcast audio console, giving
other Windows applications (DAWs, playout systems, conferencing/streaming tools) a selectable audio device
to exchange audio with LSWR 1000 through — the same role Lawo's own "R3LAY WDM Driver" plays for the R3LAY
console (which likewise exposes one WDM instance per console channel, not a single shared pair).

This project is a fork of [VirtualDrivers/Virtual-Audio-Driver](https://github.com/VirtualDrivers/Virtual-Audio-Driver)
by MikeTheTech (MIT licensed), itself built on Microsoft's own Sysvad ("Simple Audio Sample") WDM driver
sample. See `THIRD_PARTY_NOTICES.md` for full attribution.

## Overview

The driver package installs **12 fixed device instances**, "LSWR 1000 Channel 1" through "LSWR 1000
Channel 12" — matching the console's maximum configurable channel count (`ConsoleConfiguration.MaxChannelCount`,
4–12). All 12 are always installed; LSWR 1000 itself decides, based on how many channels the user's
session is actually configured with (4 to 12), how many it uses — unused instances simply sit idle, the
same way R3LAY always shows 8 WDM driver instances in Device Manager regardless of how many a given
session uses.

Each channel instance exposes a matched speaker/microphone pair:

- **"LSWR 1000 Channel N Speaker"** ("fake" speaker output) — any Windows app can select this as its
  playback device; audio sent to it is available for LSWR 1000 (or any other software) to read as that
  channel's input.
- **"LSWR 1000 Channel N Mic"** ("fake" mic input) — any Windows app can select this as its recording
  device; audio LSWR 1000 (or any other software) writes to it appears as that app's own microphone input.

By installing this driver, other Windows applications can exchange audio with any individual LSWR 1000
channel without any physical hardware in between — useful for routing a playout system, DAW, or
conferencing app directly into/out of a specific channel strip.

## Key Features

Inherited unchanged from the upstream project — same WDK-based implementation:

| Feature | Virtual Speaker | Virtual Microphone |
|---|---|---|
| **Emulated Device** | Emulates a speaker device recognized by Windows. | Emulates a microphone device recognized by Windows. |
| **Supported Audio Formats** | 8-bit/8000 Hz up to 32-bit/192,000 Hz, including common presets (Telephone, DVD/Studio Quality). | 16-bit/44,100 Hz, 16-bit/48,000 Hz, 24-bit/96,000 Hz, 24-bit/192,000 Hz, 32-bit/48,000 Hz. |
| **Spatial Sound Support** (Speaker only) | Integrates with Windows Sonic. | Integrated with Voice Focus / Background Noise Reduction. |
| **Exclusive Mode and App Priority** | Applications can claim exclusive control. | Same WDK architecture applies. |
| **Volume Level Handling** | Global and per-application volume (Windows mixer). | Adjustable via Windows Sound Settings or audio software. |

## Compatibility

- **OS**: Windows 10 (Build 1903+) and Windows 11
- **Architecture**: x64 (tested); ARM64 (inherited from upstream, not yet verified for this fork)

## Signing status

Not yet production-signed. Plan: apply to the [SignPath Foundation](https://signpath.org/)'s free
code-signing program for open-source projects (the same program the upstream `Virtual-Audio-Driver`
project uses for its own production-signed releases) once this repository has a first tagged release to
point the application at.

Until then, installing requires test signing mode:
```powershell
bcdedit /set testsigning on
```

## Installation

Each of the 12 channel instances is a separate root-enumerated hardware ID
(`ROOT\LSWR1000VirtualAudioCh1` .. `ROOT\LSWR1000VirtualAudioCh12`), all sharing the one signed driver
binary/catalog. On Windows 11, the **Add Legacy Hardware** wizard is known to report "installed
successfully" while creating zero actual device nodes for this kind of root-enumerated audio driver — use
`devcon.exe` (included in the WDK/EWDK Tools folder) instead, which is the installation method Microsoft's
own Sysvad samples document:

1. Enable test signing (see above), if not using a production-signed release.
2. From an elevated prompt, install all 12 instances:
   ```powershell
   $inf = "path\to\package\VirtualAudioDriver.inf"
   1..12 | ForEach-Object { devcon install $inf "ROOT\LSWR1000VirtualAudioCh$_" }
   ```
3. Verify: **Device Manager** → **Sound, video and game controllers** should show "LSWR 1000 Channel 1"
   through "LSWR 1000 Channel 12"; **Audio inputs and outputs** should show the matching
   "Alto-falantes (LSWR 1000 Channel N)" / mic group entries, and all 12 should be selectable in Windows'
   own Sound settings.

To remove an instance: `devcon remove "ROOT\LSWR1000VirtualAudioCh$_"`.

## Usage

Select "LSWR 1000 Channel N" as the playback/recording device — for whichever channel N an application
needs to exchange audio with — exactly like selecting any other real Windows audio device (Sound Settings,
or the app's own audio device picker).

## Attribution

Built with the Microsoft Windows Driver Kit (WDK); includes code derived from Microsoft's own Sysvad
("Simple Audio Sample") driver sample, and is a direct fork of
[VirtualDrivers/Virtual-Audio-Driver](https://github.com/VirtualDrivers/Virtual-Audio-Driver) (MIT).

- **Windows Driver Kit (WDK)**: https://learn.microsoft.com/en-us/windows-hardware/drivers/download-the-wdk
- **Windows Driver Samples (Sysvad)**: https://github.com/microsoft/Windows-driver-samples/tree/main/audio/sysvad
- **Upstream fork base**: https://github.com/VirtualDrivers/Virtual-Audio-Driver

Original code in this repository is provided under the MIT License (see `LICENSE`). Third-party Microsoft
sample code is provided under the Microsoft Public License (MS-PL); see `THIRD_PARTY_NOTICES.md` for the
full license text and details.

Microsoft, Windows, and Windows Driver Kit are trademarks of Microsoft Corporation.
