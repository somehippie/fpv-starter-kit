# 1. Controller Choice, Firmware & Setup

## Controller: RadioMaster Pocket 2

A compact, pocket-sized ELRS 2.4GHz transmitter — good for both sim use (via USB) and real flying (built-in ELRS module), which is what makes it a fit for both halves of this build.

### Alternative considered: Jumper Bumblebee

Revisited after the Classic-mode limitation above became clear. Note up front: "analog vs. digital" describes the video system (goggles/VTX), not the radio — both the Pocket and the Bumblebee are plain ELRS transmitters and work fine with an analog setup.

| | RadioMaster Pocket 2 | Jumper Bumblebee |
|---|---|---|
| Firmware | EdgeTX (preinstalled) | EdgeTX (preinstalled) |
| MCU / flash | STM32F407xE, 512KB — compiles out Advanced USB Joystick from 2.11 on (see above) | STM32F407VGT6, 1MB — likely keeps Advanced mode available |
| Internal RF power | ELRS 2.4GHz, standard power; external Nano module bay for more | Built-in ELRS up to 1W (1000mW), no module needed |
| Gimbals (stock) | Hall-effect (X5 Nano, quad ball-bearing) — reviewed on par with the Jumper T-Lite V2 | Hall-effect (RDC50) — Jumper's in-house unit, similar tier |
| Gimbal upgrade path | **AG01 Nano**, $59.99/pair — CNC aluminum, reviewed as a real step up from stock | No equivalent upgrade part found |
| Screen | 128×64 monochrome LCD | 1.3" OLED, 128×64 |
| Weight | 288g | not confirmed |
| Price | ~$60–72 | ~$72–129 (varies by retailer/region) |

**Verdict:** for someone starting fresh, the Bumblebee is arguably the stronger general pick — more stock RF power and more firmware headroom for the same rough price, at the cost of RadioMaster's larger community and module-bay expandability. For this build specifically, the Pocket 2 stays: the Classic-mode CH9 workaround is already documented and nearly working, and switching radios now would mean re-debugging already-solved setup for a gimbal/screen upgrade and a joystick feature a workaround already covers. Not independently verified: whether the Bumblebee's 1MB flash actually keeps Advanced USB Joystick mode on current EdgeTX — that's inferred from flash size, not source-checked the way the Pocket's limitation was.

## Firmware

### Factory build

Read off the radio's own SD card, `\FIRMWARE\`: `POCKET-V2.9.0-PROD-2023.7.27-C.bin`. The filename says 2.9.0, but the binary contains **edgetx-pocket-2.10.0-RM**, a RadioMaster-branded 2.10.0 build. Trust the version string inside the binary, not the filename.

### Why only Classic USB Joystick mode (2.11 and later)

EdgeTX 2.10's **Advanced** USB Joystick mode adds per-channel axis/button assignment. From 2.11 on, it is **compiled out** of the Pocket's firmware:

- [EdgeTX issue #6434](https://github.com/EdgeTX/edgetx/issues/6434): Advanced USB Joystick (`USBJ_EX`) was intentionally disabled for STM32F4 targets with only 512KB flash, because it no longer fits. The maintainer's comment names the TX12 MK2 as an example; users in the thread name the Pocket, and one reports that enabling `USBJ_EX` on the `pocket` build overflows flash by 24 bytes.
- **Source-verified:** at `v2.12.4`, [`radio/src/targets/taranis/CMakeLists.txt`](https://github.com/EdgeTX/edgetx/blob/v2.12.4/radio/src/targets/taranis/CMakeLists.txt) sets the Pocket to `CPU_TYPE_FULL STM32F407xE` (512KB), and that CPU branch sets `USBJ_EX OFF`. There is no USB Joystick page in `MDL` → Setup, for any model.

The generic manual pages for [USB Joystick](https://manual.edgetx.org/color-radios/model-settings/model-setup/usb-joystick) and [Configure Advanced Joystick](https://manual.edgetx.org/v2.11/edgetx-how-to/configure-advanced-joystick-with-edgetx) still exist because larger-flash radios keep the feature. They don't apply to this one.

On Classic mode, switches still reach the PC as buttons, just through a fixed channel layout. See [Post-flash configuration](#post-flash-configuration).

> Issue-thread comments are not the same as a confirmed fact. The first write-up here treated "users mention the Pocket" as "maintainer confirmed"; the build file is what settles it.

### Firmware history

| Date | Version | Why |
|---|---|---|
| Factory | 2.10.0-RM (RadioMaster build) | as shipped |
| 2026-09-22 | **2.10.6 "Centurion"** (released 2025-01-28) | last version with Advanced USB Joystick on this radio |
| 2026-09-26 | **2.12.4 "Queen Anne's Revenge"** (released 2026-09-02) | current stable; Advanced joystick no longer a priority |

Details of the 2.12.4 update:

- Flashed via the [SD-card method](#working-flash-procedure-sd-card--mass-storage-method): `pocket-def35ad.bin` from `edgetx-firmware-v2.12.4.zip`, SHA256 verified against GitHub's published value.
- SD card contents updated in the same session: **2.8 → 2.12.3** (bw128x64 pack + English sounds 2.12.3).
- Full SD backup taken first: `Downloads\pocket-sd-backup-20260926-101904` (1,097 files, verified).
- The original USB Joystick test model, `model04.yml` (unnamed; Advanced mode, trim offsets Ail −4 / Ele +2 / Rud +4), was kept rather than deleted. Its Advanced settings are inert on 2.12.4.
- A new stock model named `000` was created and selected.
- Post-flash, Windows sees a **Classic-mode joystick (6 axes, 24 buttons)**.

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

## The DFU vs. Mass Storage trap

If the flash procedure above fails to connect, check this first. It was the costliest mistake in this build: STM32-based radios like the Pocket 2 can present as **two different, non-interchangeable** USB devices depending on how bootloader mode is entered.

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
3. **Sticks not yet re-verified on 2.12.4.** They were confirmed on 2.10.6 (four axes at full range, read from Windows with `joyGetPosEx`). Repeat that check while actually moving the sticks.
