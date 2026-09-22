# 5. Analog Meteor75 Pro II — Drone, Battery & Spares

## Drone: BetaFPV Meteor75 Pro II — ✅ purchased ($99.99, Quadcopter/pre-built, analog)

- Original plan was the **Meteor75 Pro** ($109.99) — found sold out across BetaFPV's own store, RaceDayQuads, and Pyrodrone (only digital HD/O4 variants existed elsewhere, and those were sold out too).
- **Meteor75 Pro II** is the current, in-stock successor — arguably better spec for the same/lower price. Chose the **analog** VTX version to match the SKY04X Pro goggles.
- **Developer Kit vs. Quadcopter:** the bundle defaulted to "Developer Kit" (unassembled components requiring soldering) at the same price as "Quadcopter" (pre-built, ready to fly). Since the goal is flying immediately, chose **Quadcopter**. Final confirmed order variant ID: `44333605945478`.
- Confirmed compatible with the RadioMaster Pocket 2 (built-in ELRS 2.4GHz binds directly to the drone's onboard ELRS receiver — no extra module needed) and the Skyzone SKY04X Pro (analog VTX matches the goggles' native analog receiver; the separate "Pro II O4" digital variant would not have worked here).

## Weight comparison across the Meteor75 analog lineup

| Model | Dry weight (no battery) | All-up weight (with battery) | Battery used |
|---|---|---|---|
| Meteor75 (original) | not listed separately | 24.98g | BT2.0 450mAh 1S |
| Meteor75 Pro | 31.1g | 45.3g | 1S 550mAh |
| Meteor75 Pro II (this build) | 27.98g | ~41g (est.) | LAVA II 1S 580mAh |

BetaFPV markets the Pro II frame as "2g lighter than the previous version," which lines up with the 31.1g → 27.98g dry-weight drop. The original Meteor75 is the lightest overall, but it's a smaller/lower-power frame (weaker motors, smaller props) and not really in the same performance class as the two Pro versions. The Pro II uses GF 1811 3-blade props, 1102 motors, and an 80mm wheelbase; BetaFPV rates 10:30 min flight time on 580mAh (11:30 on 680mAh).

Sources: [BetaFPV Meteor75 Pro II product page](https://betafpv.com/products/meteor75-pro-ii-brushless-whoop-quadcopter), [BetaFPV Meteor75 Pro product page](https://betafpv.com/products/meteor75-pro-brushless-whoop-quadcopter), [Oscar Liang Meteor75 Pro (Analog) review](https://oscarliang.com/betafpv-meteor75-pro-whoop-analog/), [BetaFPV Meteor75 (original) product page](https://betafpv.com/products/meteor75-brushless-whoop-quadcopter-1s)

## Battery: LAVA II 1S 580mAh — ✅ purchased

- Cart briefly had **LAVA II 1S 680mAh** selected — corrected to **580mAh**, the spec-matched size for the Pro II. Official flight-time and thrust-to-weight figures for this drone are calculated around 580mAh; 680mAh only adds ~1 minute of flight time while adding weight that reduces agility — not worth it for a beginner.
- Considered LAVA 1S 450mAh 75C ($17.99) as a lighter alternative but stuck with 580mAh as the balanced choice.
- Bought enough packs to allow charging rotations without waiting mid-session.

## Charger: BetaFPV 6-Port 1S Battery Charger — ✅ purchased (BetaFPV website)

- The Meteor75 Pro II bundle itself includes only a USB-C adapter/4-pin cable, **not** a standalone charger.
- Bought separately, direct from the BetaFPV website: the **6-port 1S battery charger**, which lets multiple LAVA II packs charge in rotation without bottlenecking practice sessions.

## Spare parts

- **Props:** Meteor-series replacement props sourced for crash spares.
- **Screws:** initial pick ("BetaFPV Drone Screw Pack," $2.50) was for Pavo20/Aquila/O4 frames using **M2** screws — incompatible with the Meteor75 Pro II's **M1.4mm** screw spec. Corrected to the **Meteor Series Motor Fixing Screws Pack (40pcs)**.
- **Camera:** confirmed a 16:9 camera option is the right aspect ratio match for the SKY04X Pro goggles.

## Real-flight tuning: Betaflight throttle curve

This is separate from any radio-side sim throttle trick (EdgeTX CH3 Weight adjustments used only for Uncrashed) — this section is Betaflight, tuned on the drone's own flight controller, for real flight only.

| Airframe | Typical hover throttle | Why | Betaflight "Mid" setting |
|---|---|---|---|
| 5" freestyle/race quad | ~45–55% (near center stick) | Higher all-up weight relative to thrust, needs more throttle input just to hover | Mid closer to 0.5 |
| Tiny whoop (Meteor75-class) | ~20–35% (lower on the stick) | Very light airframe, ducted props, high thrust-to-weight — hovers on much less throttle | Mid lower, e.g. 0.20–0.30 |

A noob-friendly starting curve for the Meteor75 Pro II, per a 5"-quad pilot's shared settings: **Mid 0.25, Expo 0.35**. Putting the curve's inflection point down in the lower third of stick travel keeps fine hover control where a tiny whoop actually hovers, rather than wasting stick resolution up near 50% (which is correct for a 5" quad, not a whoop).

Key point: **midrange is not the same across airframes.** Copying a 5"-quad throttle curve onto a tiny whoop puts the sensitive part of the curve in the wrong place — the whoop's real hover zone sits lower on the stick than a 5" quad's does.

## Pricing / timing considerations

- Watched for Black Friday timing vs. buying now — general 2025–2027 US-China tariff pressure on FPV components pushing retailer prices up over time, plus the Meteor75 Pro already observed sold out, tipped the decision toward buying now rather than waiting.

## Totals (approximate, as ordered)

| Item | Price |
|---|---|
| Skyzone SKY04X Pro goggles pack | $533 |
| Meteor75 Pro II (Quadcopter) bundle w/ batteries | $99.99–$155.78 depending on bundle config |
| Extra batteries / spares / screws | ~$20–30 |
| 6-Port 1S Battery Charger | ~$20 |
