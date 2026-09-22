# 1. Controller Choice, Firmware & Setup

## Controller: RadioMaster Pocket 2

A compact, pocket-sized ELRS 2.4GHz transmitter — good for both sim use (via USB) and real flying (built-in ELRS module), which is what makes it a fit for both halves of this build.

## The firmware problem

Factory firmware ships without a usable configurable "USB Joystick" setup, so switches can't be presented to a PC sim as buttons out of the box. EdgeTX 2.10's **Advanced** USB Joystick mode adds per-channel axis/button assignment.

**Confirmed factory build** (read off the radio's own SD card, `\FIRMWARE\`):

> `POCKET-V2.9.0-PROD-2023.7.27-C.bin` — EdgeTX 2.9.0, built 2023-07-27

> ⚠️ **Unverified claim — check before relying on it.** An earlier draft said EdgeTX 2.11+ *removed* the configurable USB Joystick page, making 2.10.6 the last version to have it. The official EdgeTX manual contradicts this: it publishes a [USB Joystick page for 2.11](https://manual.edgetx.org/color-radios/model-settings/model-setup/usb-joystick) and a [v2.11 "Configure Advanced Joystick" how-to](https://manual.edgetx.org/v2.11/edgetx-how-to/configure-advanced-joystick-with-edgetx). If the feature still exists in 2.11, **the downgrade to 2.10.6 was unnecessary.** Verify before repeating this path.

Version flashed here regardless: **EdgeTX 2.10.6 "Centurion"** (released 2025-01-28).

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
2. Download the EdgeTX 2.10.6 "Centurion" `.bin` for the `pocket` target and verify its SHA256 against the published source.
   - The firmware bundle is `edgetx-firmware-v2.10.6.zip` on the [EdgeTX releases page](https://github.com/EdgeTX/edgetx/releases/tag/v2.10.6).
   - **There is exactly one `pocket` target** — `pocket-14adf04.bin`. No separate "Pocket 2" build exists; the Pocket 2 uses the `pocket` target.
3. Enter **bootloader mode**: hold both horizontal trims inward + power on. The radio shows "Write Firmware / Exit."
4. Connect via USB — the radio should mount as a plain USB drive.
5. Copy the `.bin` file into the `FIRMWARE` folder on that drive (create it if needed, e.g. `F:\FIRMWARE\<file>.bin`).
6. Unplug the USB cable — the radio needs to write from its own SD card, not while tethered.
7. On the bootloader screen, select **Write Firmware**, pick the `.bin` file, confirm.
8. Wait a couple of minutes for the flash, then the radio reboots into EdgeTX 2.10.6 automatically.

> The bootloader validates the image before writing, which makes this materially safer than DFU flashing. There are [reports of radios bricked by DFU flashes via Buddy](https://github.com/EdgeTX/edgetx/issues/5112).

## SD card contents version

Separate from firmware. Check `edgetx.sdcard.version` in the card's root — it was **2.8** here against 2.10.6 firmware. A mismatch produces a warning on boot plus missing sounds and themes. SD contents are a separate download from the [edgetx-sdcard releases](https://github.com/EdgeTX/edgetx-sdcard/releases).

## Post-flash configuration

> ⚠️ **Correction to the earlier draft.** It gave the path as `SYS` → **Hardware** → USB Joystick. Per the [EdgeTX 2.10 manual](https://manual.edgetx.org/v2.10/bw-radios/model-select/setup), USB Joystick lives under **Model** settings, not Radio settings:
>
> **`MDL` → Setup → USB Joystick**

Set **Mode = Advanced** (not Classic) for per-channel axis/button assignment.

**Channel mapping (MIXES):** Channel 1 = Aileron/Roll, Channel 2 = Elevator/Pitch, Channel 3 = Throttle, Channel 4 = Rudder/Yaw.

### Sticks vs. switches: two hops, not one

This is the part that actually costs time. The radio sends 16+ numbered **channels** to the PC.

- **Sticks are wired to channels 1–4 at the factory.** They work immediately.
- **Switches are wired to nothing.** A switch is invisible to the PC until *both* of these are true:

**Hop 1 — switch → channel.** `MDL` → **Mixes** → pick a free channel → set **Source** to the switch (`SA`, `SB`, …).
Use **press-and-hold** to open the entry for editing; a short press only moves the cursor, which is an easy way to believe you saved something you didn't.

**Hop 2 — channel → button.** `MDL` → Setup → USB Joystick → that channel → Mode = **Btn**. This is what gives a sim's binding screen a discrete on/off signal to bind arm/disarm to.

| Field | Value |
|---|---|
| Mode | `Btn` |
| Positions | `2POS` for a 2-position switch, `Push` for momentary |
| Button No. | 1 (or next free) |
| Inversion | Off |

Button sub-modes: `Normal` (held while switch is in position), `Pulse` (brief press on change), `SWEmu` (toggle emulation). `Normal` is fine if whatever reads it does edge detection.

⚠️ A **3-position** switch in `Normal` mode consumes **three** button numbers, one per position. Use a 2-position switch for a single clean trigger.

**Channel convention:** CH1–8 for axes, CH9–32 for buttons. Keeps things from colliding if you ever switch back to Classic mode.

**Replug the USB cable after any change.** The radio only re-describes itself to Windows on re-enumeration. Config edits are invisible until you do.

### Practical tips

- Testing on a spare/new model slot keeps an existing working config untouched while experimenting.
- To delete a bad mix/channel entry: long-press the entry → **Delete**.

### Verifying what the radio is actually sending

Don't trust the sim's binding screen to tell you whether the radio is working — read it from Windows directly. `joyGetDevCapsW` reports declared capability, `joyGetPosEx` reports live values:

- **Axes at a flat `32767`** = dead center. With sticks *moving*, that means no channel data. With sticks *at rest*, it means nothing at all — make sure the sticks are actually being moved during a test.
- **`wNumButtons = 0`** = no switch is assigned. **`wNumButtons > 0` with no press events** = a button slot is declared but nothing drives it → Hop 1 is missing.

Reading capability *and* live values separately is what distinguishes "not configured" from "configured but not wired."

## Lessons for next time

1. If a flashing tool throws a DFU-specific error, **check the actual USB VID:PID and interface class first** (Device Manager or equivalent) before touching driver bindings. `0483:DF11` = DFU, `0483:5720` = Mass Storage — two unrelated device modes, not two states of the same one.
2. **Zadig is not a general fix.** It rebinds drivers, and pointing it at a non-DFU device actively breaks a working path. Never apply it to a HID game controller.
3. **Measure before concluding.** Hours went into diagnoses built on a test where the sticks weren't being moved, and another that assumed no SD card was inserted. Confirm the input to a test before trusting its output.

## Open item

**Switch → button mapping is not working yet** (as of 2026-09-22). Post-flash the radio declares `1 button / 32 max`, but no switch produces a press event, and the descriptor is byte-identical before and after the config attempt — suggesting Hop 1 (the mix) or Mode = Advanced didn't commit. Next check: the radio's **Channel Monitor** (`MDL` → Channels). If the channel's bar doesn't move when the switch is flipped, the mix was never saved.
