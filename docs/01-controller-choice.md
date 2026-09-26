# 1. Controller Choice, Firmware & Setup

## Controller: RadioMaster Pocket 2

A compact, pocket-sized ELRS 2.4GHz transmitter — good for both sim use (via USB) and real flying (built-in ELRS module), which is what makes it a fit for both halves of this build.

## The firmware problem

Out of the box, switches aren't presented to a PC sim as buttons. EdgeTX 2.10's **Advanced** USB Joystick mode added per-channel axis/button assignment, but it no longer exists for this radio from 2.11 on (see below). On current firmware, switches reach the PC through **Classic** mode's fixed layout instead.

**Confirmed factory build** (read off the radio's own SD card, `\FIRMWARE\`):

> `POCKET-V2.9.0-PROD-2023.7.27-C.bin` — the filename says 2.9.0, but the binary actually contains **edgetx-pocket-2.10.0-RM** (a RadioMaster-branded 2.10.0 build). Corrected 2026-09-26 — don't trust the filename over the binary.

> ✅ **Resolved — the earlier "unverified" flag was correct to raise, and the claim checks out.** [EdgeTX issue #6434](https://github.com/EdgeTX/edgetx/issues/6434) confirms Advanced USB Joystick (`USBJ_EX`) was **intentionally disabled starting in 2.11.x** for STM32F4 targets with only 512KB flash — the feature doesn't fit alongside everything else once compiled in. The maintainer's comment names the TX12 MK2 as an example; users in the same thread name the **RadioMaster Pocket**, and one reports that enabling `USBJ_EX` on the `pocket` build overflows flash by 24 bytes. The definitive proof is the build file itself: at `v2.12.4`, [`radio/src/targets/taranis/CMakeLists.txt`](https://github.com/EdgeTX/edgetx/blob/v2.12.4/radio/src/targets/taranis/CMakeLists.txt) sets the Pocket to `CPU_TYPE_FULL STM32F407xE` (512KB), and that CPU branch sets `USBJ_EX OFF`. The USB Joystick setup page is compiled out entirely, for every model. The generic manual pages for "USB Joystick" and "Configure Advanced Joystick" still exist because larger-flash radios keep the feature — they just don't apply to this one. So the 2.10.6 downgrade was not a misdiagnosis: it was the correct call *if* Advanced per-channel mapping is the priority.

Originally flashed here: **EdgeTX 2.10.6 "Centurion"** (released 2025-01-28), specifically to get Advanced mode.

## Firmware update: 2.10.6 → 2.12.4 (2026-09-26)

Updated anyway, priority shifted to running current EdgeTX over keeping Advanced joystick mode:

- Flashed **EdgeTX 2.12.4 "Queen Anne's Revenge"** (latest stable, released 2026-09-02) via the same SD-card/bootloader method below.
- Firmware file: `pocket-def35ad.bin` from `edgetx-firmware-v2.12.4.zip`; SHA256 verified against GitHub's published value.
- SD card contents updated in step: **2.8 → 2.12.3** (bw128x64 pack + English sounds 2.12.3) — see [SD card contents version](#sd-card-contents-version) below for why this matters separately from firmware.
- Full SD backup taken first: `Downloads\pocket-sd-backup-20260926-101904` (1,097 files, verified) — same discipline as the original flash.
- A pre-existing model, `model04.yml` (unnamed, the original USB-Joystick test model with Advanced mode and trim offsets Ail −4 / Ele +2 / Rud +4), was **not** touched and is kept for later rather than deleted.
- A new stock model named `000` was created and selected instead of reusing `model04`.
- Post-flash, Windows sees a **Classic-mode joystick (6 axes, 24 buttons)**. On 2.12.4 that is the only mode this radio has; `model04.yml`'s Advanced joystick settings are inert too, since the feature is missing from the firmware rather than from the model.
- **Not yet verified:** that the radio actually reports `2.12.4` cleanly, and that model `000` saved correctly. Check by connecting as USB Storage and reading `RADIO/radio.yml` (firmware semver) and the `MODELS/` folder contents.

## The DFU vs. Mass Storage trap

The costliest mistake in this build: STM32-based radios like the Pocket 2 can present as **two different, non-interchangeable** USB devices depending on how bootloader mode is entered.

| Mode | USB ID | What it looks like to your PC | Flash method |
|---|---|---|---|
| DFU | `0483:DF11` | A DFU programming interface | EdgeTX Buddy "Connect" (DFU flash) |
| Mass Storage | `0483:5720` | A plain USB flash drive with a `FIRMWARE` folder | Copy `.bin` to `\FIRMWARE\`, flash from the radio's own menu |

The Pocket 2's bootloader came up as **Mass Storage** (`0483:5720`), not DFU. EdgeTX Buddy's "Could not connect to DFU interface" was misread as a driver problem, and Zadig was used to bind WinUSB to `0483:5720` — which was never a DFU endpoint. That rebinding only broke the correct, already-working mass-storage path.

**How to prove it in one command** (Windows) — read the USB interface class rather than guessing:

```powershell
Get-PnpDevice -InstanceId 'USB\VID_0483&PID_5720\<serial>' |
  Get-PnpDeviceProperty -KeyName DEVPKEY_Device_CompatibleIds
```

- `USB\Class_08&SubClass_06&Prot_50` → **Mass Storage**. No DFU interface exists. `dfu-util` and Buddy's DFU flash will both fail, and no driver change will help.
- `USB\Class_FE&SubClass_01` → **DFU**. DFU tools will work.

Also check whether the device is *composite* (child `MI_` interfaces). A non-composite device reporting only `Class_08` has no hidden DFU interface.

## Undoing a mistaken Zadig binding

> ⚠️ **Correction to the earlier draft.** It gave a single command hardcoding `oem38.inf`. That is wrong on two counts.

**Zadig installed *two* driver packages, not one.** Deleting `oem38.inf` left the device still bound to WinUSB through `oem37.inf`, and the symptom persisted through a replug. Both had to go.

**The `oemNN.inf` numbers are machine-specific.** Never copy them from a guide. Find yours:

```powershell
pnputil /enum-drivers        # look for: Provider Name = libwdi
```

Then remove **every** libwdi package it lists (elevated):

```powershell
pnputil /delete-driver oemNN.inf /uninstall
```

Then **physically unplug and replug the radio.** Windows will not re-evaluate the driver until the device re-enumerates — deleting the package alone leaves the running device on the old binding. Confirm it flipped:

```powershell
Get-PnpDevice -PresentOnly | Where-Object { $_.InstanceId -match 'VID_0483' } |
  Get-PnpDeviceProperty -KeyName DEVPKEY_Device_Service    # want: USBSTOR
```

When `Service = USBSTOR` and the SD card appears as a drive letter, the mass-storage path is healthy again.

## Working flash procedure (SD-card / mass-storage method)

1. **Back up the SD card first** — plain recursive copy of the whole card.
2. Download the EdgeTX `.bin` for the `pocket` target and verify its SHA256 against the published source.
   - Current build: `edgetx-firmware-v2.12.4.zip` on the [EdgeTX releases page](https://github.com/EdgeTX/edgetx/releases/tag/v2.12.4) contains `pocket-def35ad.bin`. The earlier 2.10.6 flash used `pocket-14adf04.bin` from `edgetx-firmware-v2.10.6.zip`; go back to that only if you need Advanced USB Joystick more than current firmware.
   - **There is exactly one `pocket` target** per release. No separate "Pocket 2" build exists; the Pocket 2 uses the `pocket` target.
   - Update the SD card contents to the matching version in the same session (see [SD card contents version](#sd-card-contents-version)).
3. Enter **bootloader mode**: hold both horizontal trims inward + power on. The radio shows "Write Firmware / Exit."
4. Connect via USB — the radio should mount as a plain USB drive.
5. Copy the `.bin` file into the `FIRMWARE` folder on that drive (create it if needed, e.g. `F:\FIRMWARE\<file>.bin`).
6. Unplug the USB cable — the radio needs to write from its own SD card, not while tethered.
7. On the bootloader screen, select **Write Firmware**, pick the `.bin` file, confirm.
8. Wait a couple of minutes for the flash, then the radio reboots into the new EdgeTX automatically.

> The bootloader validates the image before writing, which makes this materially safer than DFU flashing. There are [reports of radios bricked by DFU flashes via Buddy](https://github.com/EdgeTX/edgetx/issues/5112).

## SD card contents version

Separate from firmware. Check `edgetx.sdcard.version` in the card's root — it was **2.8** against the original 2.10.6 firmware (mismatch produces a warning on boot plus missing sounds and themes), and was updated to **2.12.3** alongside the 2.12.4 firmware update on 2026-09-26. SD contents are a separate download from the [edgetx-sdcard releases](https://github.com/EdgeTX/edgetx-sdcard/releases) — always match it to the firmware version, they don't auto-sync.

## Post-flash configuration

On EdgeTX 2.12.4 this radio has **Classic** USB Joystick mode only, and there is no USB Joystick page in `MDL` → Setup. Classic mode is a fixed layout, verified against [`radio/src/usb_joystick.cpp`](https://github.com/EdgeTX/edgetx/blob/v2.12.4/radio/src/usb_joystick.cpp) at `v2.12.4`:

| Radio channel | Appears on the PC as | Rule |
|---|---|---|
| CH1–CH8 | 8 axes (X, Y, Z, Rx, Ry, Rz, Slider, Dial) | channel value maps to axis position |
| CH9–CH32 | Buttons 1–24 | **pressed when the channel output is above 0** |

Windows' legacy joystick API only reports the first 6 of those axes, which is why `joyGetDevCapsW` shows `6 axes / 24 buttons`.

**Channel mapping (MIXES):** Channel 1 = Aileron/Roll, Channel 2 = Elevator/Pitch, Channel 3 = Throttle, Channel 4 = Rudder/Yaw.

> The 2.10.6 setup used **Advanced** mode (`MDL` → Setup → USB Joystick → per-channel `Btn` / `2POS` / button number). That page is compiled out on 2.11+ for this radio, so those steps no longer apply.

### Sticks vs. switches

The radio sends numbered **channels** to the PC.

- **Sticks are wired to channels 1–4 at the factory.** They work immediately.
- **Switches are wired to nothing.** In Classic mode there is only one hop: put the switch on a channel from 9 up, and that channel *is* a button.

**Switch → button.** `MDL` → **Mixes** → **CH9** → set **Source** to the switch (`SA`, `SB`, …). CH9 becomes button 1, CH10 button 2, and so on.
Use **press-and-hold** to open the entry for editing; a short press only moves the cursor, which is an easy way to believe you saved something you didn't.

With a 100% weight mix, a 2-position switch outputs −100 in one position (button released) and +100 in the other (button pressed). A 3-position switch's middle position outputs 0, which also counts as released, so only one end presses the button. To flip which end presses, set the mix weight to −100.

**Replug the USB cable if the joystick layout seems stale.** The radio only re-describes itself to Windows on re-enumeration. Button presses themselves are live.

### Practical tips

- Testing on a spare/new model slot keeps an existing working config untouched while experimenting.
- To delete a bad mix/channel entry: long-press the entry → **Delete**.

### Verifying what the radio is actually sending

Don't trust the sim's binding screen to tell you whether the radio is working — read it from Windows directly. `joyGetDevCapsW` reports declared capability, `joyGetPosEx` reports live values:

- **Axes at a flat `32767`** = dead center. With sticks *moving*, that means no channel data. With sticks *at rest*, it means nothing at all — make sure the sticks are actually being moved during a test.
- In Classic mode `wNumButtons` is always 24, so it says nothing about configuration. **No press events when the switch flips** means the switch isn't mixed onto CH9+, or the mix output never goes above 0.

Reading capability *and* live values separately is what distinguishes "not configured" from "configured but not wired."

## Lessons for next time

1. If a flashing tool throws a DFU-specific error, **check the actual USB VID:PID and interface class first** (Device Manager or equivalent) before touching driver bindings. `0483:DF11` = DFU, `0483:5720` = Mass Storage — two unrelated device modes, not two states of the same one.
2. **Zadig is not a general fix.** It rebinds drivers, and pointing it at a non-DFU device actively breaks a working path. Never apply it to a HID game controller.
3. **Measure before concluding.** Hours went into diagnoses built on a test where the sticks weren't being moved, and another that assumed no SD card was inserted. Confirm the input to a test before trusting its output.

## Open items

1. **Switch → button mapping not yet done on 2.12.4.** The 2026-09-22 attempt used Advanced mode, which no longer exists here. Redo it the Classic way on model `000`: mix a 2-position switch onto CH9, check the radio's **Channel Monitor** (`MDL` → Channels) shows CH9 swinging between −100 and +100, then confirm button 1 presses in Windows.
2. **Firmware version and model save not yet verified on-device.** Confirm by connecting as USB Storage and reading `RADIO/radio.yml` (should report `semver: 2.12.4`) and the `MODELS/` folder (should contain the `000` model).
