# 2. Sim Choice, Settings, Tips & Calibration

## Sim #1: Uncrashed (Steam)

General-purpose FPV racing sim, used first to learn basic stick control and arm/disarm flow once the Pocket 2 was flashed and configured (see [Controller Choice](01-controller-choice.md)).

**Binding:** Uncrashed's controller binding screen auto-detects movement per control — wiggle each stick/switch when prompted and it registers correctly once EdgeTX's Advanced USB Joystick mode and channel mapping are set up right.

**Calibration reality check:** full calibration (endpoints/centering) plus getting comfortable took about 4 hours of practice before the first complete, crash-free lap.

## Keyboard shortcuts worth knowing

Bound by default, cost nothing to try. Two of them solve problems that look like they need configuration.

| Key | Action | What it does |
|---|---|---|
| **`R`** | `Reset` | Resets the drone **only**. Damage and race timer persist. |
| **`T`** | `RestartRace` | **Full restart** — damage cleared, timer reset, back to the start. |
| **`A`** | `Acro` | Toggles acro ↔ self-leveling. |
| `Q` | `SaveSpawn` | Sets the respawn point |
| `C` | `SwitchCameraMode` | Cycles camera |
| `M` | `MiniMap` | Toggles minimap |
| `Tab` | `ToggleUI` | Hides/shows HUD |
| `F9` / `F10` | Recording | Start recording / save last 2 min |

**`R` vs `T` is the one that trips people up.** If a reset seems to "keep your damage," you're on `R` and you want `T`. No mod or map reload is needed — the action already exists.

**`A` is the real beginner mode.** Toggling *out* of acro gives self-leveling: release the sticks and the quad levels itself instead of continuing to rotate. For learning this matters far more than any throttle curve — expo makes hover smoother, but self-leveling is what stops you tumbling into the ground.

Every action has **two binding slots**, and the second is empty by default, so a controller binding can be added without losing the keyboard one.

## Uncrashed throttle curve (sim-side only)

Uncrashed stores no flight-tuning of its own — there's no in-game throttle curve setting — so any throttle-feel adjustment has to happen on the radio side instead.

Verified by inspecting `%LOCALAPPDATA%\Uncrashed\Saved\SaveGames\Settings\MyOptions.sav`: zero matches for `rate`, `expo`, `curve`, `throttle`, `angle`, `horizon`, `assist`, `smooth` or `sensitivity`. The complete section list is `SaveGraphicIndex`, `SaveControls`, `SaveSpawn`, `SaveLast2Min`, `SaveAudio`, `SaveUI`, `SaveUI_Weather` — graphics, key bindings, spawn point, clip recording, audio, UI, weather. **No flight tuning of any kind.**

**Fix:** in EdgeTX, reduce **CH3 Weight to 75–80%** (from its default of 100%). This flattens/softens the throttle response coming out of the radio before it ever reaches the sim, compensating for Uncrashed's default throttle feel. Optionally add **Expo ~20–30%** on the same mix to stretch the low-throttle range where a quad actually hovers.

Write down the existing values before changing them — that is the undo, and EdgeTX won't remember them for you.

**Important — this is sim-only.** This CH3 Weight change lives on the radio and is not related to Betaflight throttle-curve tuning (Mid/Expo) on the real Meteor75 Pro II's flight controller — see [Drone, Battery & Spares](05-drone-battery-spares.md#real-flight-tuning-betaflight-throttle-curve) for that, entirely separate topic. **Undo the CH3 Weight change (set it back to 100%) before flying real hardware** — carrying a sim-only compensation over to the real drone will make it fly wrong.

## Backing up sim settings

Uncrashed keeps everything in one file, so a byte-exact copy is a more reliable "remember my defaults" than transcribing numbers:

```
%LOCALAPPDATA%\Uncrashed\Saved\SaveGames\Settings\MyOptions.sav
```

Worth copying alongside it: `Saved\Config\WindowsNoEditor\Input.ini` and `GameUserSettings.ini`. Restore by copying back.

## Mapping a radio switch to a race restart (AutoHotkey)

Uncrashed's own binding UI handles axes well; for driving a *keyboard action* from a radio switch, an AutoHotkey bridge is more robust and survives game updates. It also shares as a single text file.

Requires [AutoHotkey v2](https://www.autohotkey.com/) and a switch mapped to a joystick button (see [Controller Choice](01-controller-choice.md) — "two hops").

```ahk
#Requires AutoHotkey v2.0
#SingleInstance Force

JOYSTICK := 1                        ; 1 = first controller
BUTTON   := 5                        ; set to your switch's button number
SENDKEY  := "t"                      ; t = RestartRace
WINSPEC  := "ahk_exe Uncrashed.exe"  ; only fires while Uncrashed is focused

prev := false
SetTimer(Poll, 25)

Poll() {
    global prev, JOYSTICK, BUTTON, SENDKEY, WINSPEC
    cur := GetKeyState(JOYSTICK . "Joy" . BUTTON)
    ; Press edge only, so a toggle switch restarts once per flip
    ; instead of repeating while held.
    if (cur && !prev) {
        ; Without the window check, flipping the switch while
        ; alt-tabbed types "t" into whatever app is focused.
        if WinActive(WINSPEC) {
            Send("{" SENDKEY " down}")
            Sleep(50)                ; UE4 can drop presses shorter than this
            Send("{" SENDKEY " up}")
        }
    }
    prev := cur
}
```

Three details that matter: **edge detection** (so an ordinary toggle switch works, not just a momentary one), the **window check** (so it can't type into other apps), and the **50 ms hold** (UE4 intermittently drops shorter presses).

The **scroll/select button cannot be used** — EdgeTX reserves the rotary encoder and its push for menu navigation, so it is never exposed as a joystick button. Use a spare switch.

## Not worth modding

Uncrashed is UE4.27 with roughly 15 GB of content across `pakchunk0`–`pakchunk105`. A mod means unpacking, editing blueprint logic and repacking, and UE4 shipping paks are frequently encrypted or signature-checked. It would break on every game update. Since `T` already provides the full restart, the effort/reward is bad — the AHK route reaches the same outcome in ~15 lines with nothing to maintain.

## Sim #2: Liftoff: Micro Drones (Steam)

- **Price:** $15.99
- **Released:** August 4, 2025 (out of Early Access; had been in EA since November 2021)
- **Focus:** the micro/whoop class specifically — turns ordinary indoor spaces into flyable tracks, which better mirrors realistic first-flight conditions for a Meteor75-class whoop than a general racing sim.
- **Features:** single-player + online multiplayer, a dedicated drone and track editor, 18 Steam achievements, Family Sharing supported.
- **Requirements:** broadband internet required; integrated Intel HD graphics not recommended; ~15 GB storage.
- **Controller:** recommends an RC transmitter; no specific brand callouts, but the same EdgeTX Advanced USB Joystick config used for Uncrashed should carry over — just rebind axes in Liftoff's own controller settings screen.

## Why run both

Uncrashed built the general stick-coordination and crash-recovery reflexes; Liftoff adds whoop-specific practice — tight indoor turns, prop-wash recovery, low-speed hover control — that maps more directly onto the Meteor75 Pro II's actual flight characteristics before risking real props.

## Practice plan

1. Install Liftoff on the same PC.
2. Reuse the existing EdgeTX Advanced USB Joystick config from Uncrashed — rebind in Liftoff's controller settings.
3. Use the drone/track editor to build a track approximating your expected first real-world flying space (room size, obstacles).
4. Focus reps on whoop-specific skills: tight turns, prop-wash recovery, low-speed hover — more relevant to a 1S brushless whoop than open-track racing practice.

## Sources

- [Liftoff®: Micro Drones on Steam](https://store.steampowered.com/app/1432320/Liftoff_Micro_Drones/)
- [Liftoff: Micro Drones — official product page](https://www.liftoff-game.com/our-products/liftoff-micro-drones)
