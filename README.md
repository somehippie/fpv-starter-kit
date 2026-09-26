# FPV Starter Kit — Analog Build Guide

A from-scratch analog FPV starter path: RadioMaster Pocket 2 controller → PC sim practice → real analog goggles and whoop. Written up as a reusable guide for anyone starting the same way.

## Why this path

Analog FPV (SteadyView-class video) is the cheapest, most beginner-friendly way into the hobby — lower latency, cheaper goggles and VTX gear, and a mature parts/secondhand ecosystem, versus locking into a pricier DJI O3/O4 or HDZero digital system on day one.

## Sections

| # | Section | Covers |
|---|---|---|
| 1 | [Controller Choice](docs/01-controller-choice.md) | RadioMaster Pocket 2, factory firmware limitation, EdgeTX flashing, DFU vs. mass-storage trap, USB Joystick config |
| 2 | [Sim Choice, Setup & Calibration](docs/02-sim-setup.md) | Uncrashed and Liftoff: Micro Drones, radio-to-sim binding, in-sim keys, sim-side throttle softening, AutoHotkey switch bridge |
| 3 | [Analog vs. Digital](docs/03-analog-vs-digital.md) | Why analog over DJI O3/O4 and HDZero — mainly price |
| 4 | [Goggle Choice](docs/04-goggle-choice.md) | Skyzone SKY04X Pro vs. HDZero Goggle 2 (close second) vs. DJI Goggles 2 |
| 5 | [Drone, Battery & Spares](docs/05-drone-battery-spares.md) | BetaFPV Meteor75 Pro II, battery sizing, spare parts, weight comparison, Betaflight throttle curve |

## Current build

| Component | Choice |
|---|---|
| Controller | RadioMaster Pocket 2, EdgeTX 2.12.4 "Queen Anne's Revenge" |
| Sims | Uncrashed (Steam), Liftoff: Micro Drones (Steam) |
| Video system | Analog (SteadyView-class) |
| Goggles | Skyzone SKY04X Pro |
| Drone | BetaFPV Meteor75 Pro II — analog, Quadcopter (pre-built) |
| Battery | LAVA II 1S 580mAh |

## Two throttle adjustments, two different places

Easy to conflate, so stated once up front:

| | Where | When | Undo |
|---|---|---|---|
| **CH3 Weight 75–80%** | The radio (EdgeTX) | Sim practice only | Set back to 100% before real flight |
| **Mid 0.25 / Expo 0.35** | The drone (Betaflight) | Real flight only | n/a — this is the real tune |

Details in [section 2](docs/02-sim-setup.md#uncrashed-throttle-curve-sim-side-only) and [section 5](docs/05-drone-battery-spares.md#real-flight-tuning-betaflight-throttle-curve).

## Status

Sim-trained on Uncrashed, hardware purchased and in hand or en route, adding Liftoff: Micro Drones for whoop-specific practice before first real flights.

**Firmware:** flashed to EdgeTX 2.10.6 via the SD-card method on 2026-09-22 (after a detour through a mistaken Zadig/WinUSB driver binding), then updated to **EdgeTX 2.12.4 "Queen Anne's Revenge"** on 2026-09-26, same SD-card method, SD card contents updated to 2.12.3 alongside it. Sticks confirmed working on 2.10.6 — four axes at full range, verified directly from Windows; not yet re-verified on 2.12.4.

**Resolved:** whether EdgeTX 2.11+ dropped the configurable Advanced USB Joystick page — confirmed true for this radio. [EdgeTX issue #6434](https://github.com/EdgeTX/edgetx/issues/6434) shows it was intentionally disabled starting in 2.11.x on 512KB-flash STM32F4 targets, and the RadioMaster Pocket is explicitly named as affected. The 2.10.6 downgrade was the right call for Advanced mode — the update to 2.12.4 was a deliberate trade-off (newer EdgeTX over per-channel joystick config), not a correction of a mistake.

**Open items:** (1) switch → button mapping still not functional as of last test, next diagnostic is the radio's Channel Monitor; (2) whether Advanced USB Joystick mode is even selectable on 2.12.4 for this radio is unconfirmed given the flash-constraint issue; (3) the 2.12.4 flash and new `000` model haven't been verified on-device yet (`RADIO/radio.yml`, `MODELS/` folder). See [Controller Choice](docs/01-controller-choice.md#open-items).
