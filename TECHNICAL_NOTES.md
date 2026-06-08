# Project North Star — Hardware Modification & Electronics Engineering Notes

> **Author:** airbate
> **Base:** Leap Motion / UltraLeap Project North Star (Release 3 + Deck X)
> **License:** Apache 2.0
> **Scope:** Optical alignment, display driver electronics, mechanical chassis, thermal management, cable routing

---

## Preface

Project North Star is the most ambitious open-source AR headset design to date — a $100 BOM reference platform that delivers a >100° diagonal FOV at 1600×1440 per eye running at 120 Hz. That's roughly 3× the FOV of a HoloLens 2 at 1/35th the cost.

But "open-source reference design" is a polite way of saying "here's a starting point, you figure out the rest." The stock build works. Barely. Every builder hits the same walls: reflector alignment drifts after 20 minutes of wear, the display driver board picks up noise from the USB 3.0 bus, the halo flexes under the weight of the optics, and thermal soak from the driver ICs degrades panel uniformity after extended sessions.

This document catalogs what I changed, why I changed it, and what I learned over multiple hardware revision cycles. It is written for the next builder who stares at a bag of McMaster-Carr fasteners and wonders if they're in over their head.

---

## 1. Display Driver Electronics

### 1.1 The Stock Problem

The Analogix ANX7530 display driver board is the nervous system of the headset. It receives DisplayPort signals from the host, splits them into dual MIPI-DSI streams, and drives the two BOE VS035ZSM-NW0 panels at 120 Hz with low-persistence backlight strobing.

In the stock configuration, three problems converge:

- **Power rail noise.** The 3.3V and 1.8V LDO regulators on the driver board share a ground plane with the USB 3.0 hub. When the Leap Motion hand tracking module polls at 150 Hz, the resulting ground bounce injects 40–80 mVpp of noise into the panel VCOM reference. This manifests as subtle horizontal banding in uniform-color scenes.
- **Signal integrity on the FPC ribbons.** The 30-pin flex PCB cables between the driver board and the panels run unshielded for ~120 mm through the halo channel. At 1.2 Gbps/lane MIPI speeds, the crosstalk margin is already tight — any impedance discontinuity converts to visible pixel clock jitter.
- **Thermal throttling with no telemetry.** The driver IC junction temperature crosses 85°C within 15 minutes of operation with no heatsinking and no thermal pad contact to the chassis. The chip self-throttles by dropping the panel refresh to 90 Hz, but there is no indication this is happening — the user just sees degraded motion clarity and assumes the panels are bad.

### 1.2 Modifications Applied

**Power delivery overhaul:**

```
Stock: USB 5V → onboard LDO → 3.3V / 1.8V (shared ground, no filtering)
Mod:   USB 5V → external LTC3440 buck-boost → 3.6V → onboard LDO → 3.3V
                                  └→ TPS7A4700 ultra-low-noise LDO → 1.8V (analog)
                                  └→ separate 1.8V rail (digital) via on-board LDO
```

The key change is splitting the 1.8V rail into analog (panel VCOM, backlight reference) and digital (MIPI PHY, timing controller) domains with a star-ground topology. The TPS7A4700 on the analog rail brings output noise down to 4.17 μVrms (measured at 10 Hz–100 kHz bandwidth). The stock LDO was pushing ~120 μVrms across the same band. This single change eliminated the horizontal banding artifact entirely.

**FPC signal conditioning:**

Inserted a small interposer PCB at the driver-board end of each flex cable with:

- Common-mode chokes (TDK ACM2012H-900) on each MIPI lane pair
- 100 Ω differential termination matched to within ±2%
- A solid copper fill on the bottom layer tied to the clean analog ground

This is not re-timing — the signal still runs native MIPI-DSI — but the impedance match and common-mode rejection clean up enough clock jitter to visibly sharpen the pixel grid, especially in high-contrast text rendering.

**Thermal solution:**

Milled a small aluminum heat spreader (40 × 25 × 3 mm) and bonded it to the driver IC with thermal epoxy (Arctic Silver, 7.5 W/m·K). The spreader makes contact with the aluminum halo bar stock through a 0.5 mm thermal gap pad. Result: junction temperature stabilizes at 62°C under continuous 120 Hz operation (ambient 25°C). The driver never throttles.

---

## 2. Optical Alignment System

### 2.1 The Stock Problem

The ellipsoidal reflectors are the soul of North Star. They're diamond-turned PMMA with a 50-50 transmissive-reflective coating — effectively a half-silvered mirror bent into a precise ellipsoid section. The display panel sits at one focus; the user's pupil sits at the other. When aligned correctly, a collimated virtual image appears to float at optical infinity.

When aligned incorrectly — by even 0.5° of tilt or 1 mm of translation — the stereo overlap collapses. One eye sees a slightly rotated image. The user gets a headache in under five minutes.

The stock optics bracket sets reflector position with four M2.5 screws through elongated slots. This is adjustable, yes, but it is also unrepeatable. Every time you remove the bracket to access the electronics, you lose alignment. Re-aligning takes 30–45 minutes of iterative tweaking with a test pattern.

### 2.2 Modifications Applied

**Kinematic mount conversion:**

Replaced the stock four-screw adjustment with a three-point kinematic mount per reflector:

- Point A: ball-tip set screw (McMaster-Carr 94125A210) seated in a conical detent → sets X/Y position
- Point B: ball-tip set screw in a V-groove → constrains rotation
- Point C: flat-tip set screw against a polished pad → sets Z (tilt)

This is the same principle used in optical table mirror mounts, just miniaturized. The reflectors now return to within 0.05 mm of calibrated position after removal and reinstallation. Alignment time dropped from 45 minutes to under 5 minutes.

**Calibration jig:**

Built a simple alignment jig from aluminum extrusion and a calibration target printed on matte photo paper:

1. Mount headset on jig, facing a retroreflective target 2.0 m away
2. Project a crosshatch test pattern to each eye independently
3. Adjust each reflector's three kinematic points until the crosshatch overlay is pixel-perfect at center and corners
4. Lock set screws with low-strength threadlocker (Loctite 222, purple)

The jig is not precise enough for production QC, but it turns "impossible to reproduce" into "good enough to wear for two hours without eye strain."

---

## 3. Mechanical Chassis

### 3.1 The Stock Problem

The stock chassis is an impressive piece of parametric CAD optimized for consumer-grade FDM printers. But PLA has a flexural modulus of roughly 3.5 GPa. The optics bracket weighs ~180 g loaded. When the wearer turns their head, inertial flex in the bracket introduces ~1.5° of peak-to-peak reflector wobble — enough to momentarily break stereo fusion during quick head movements.

The halo (electronics tray) has a different problem: it's the right shape but the wrong material. PLA creeps under sustained load. After ~50 hours of wear, the USB and DisplayPort connectors develop enough play to cause intermittent disconnects.

### 3.2 Modifications Applied

**Optics bracket — hybrid material strategy:**

| Component | Stock Material | Modified Material | Rationale |
|---|---|---|---|
| Reflector mounts | PLA | PETG-CF (carbon fiber filled, 15% wt.) | Flexural modulus 6.2 GPa vs 3.5 GPa; nearly double the stiffness for the same mass |
| Facial interface | PLA | TPU 95A (flexible filament) | Compliant, conforms to face geometry, doesn't crack |
| Structural bridges | PLA | 6061-T6 aluminum, waterjet-cut, hand-finished | Zero creep, dimensionally stable, acts as a secondary heatsink |

The PETG-CF reflector mounts are the biggest single mechanical win. They're printed on a standard Ender 3 with a hardened steel nozzle (0.4 mm), 260°C hot end, 80°C bed, 0.15 mm layer height. The carbon fiber fill makes the surface slightly matte — it looks and feels like a professional camera body part, not a 3D print.

**Halo connector reinforcement:**

Designed and printed a small TPU strain-relief block that clamps both the USB 3.0 and Mini DisplayPort cables against the halo body. The block is contoured to match the cable bend radius (minimum 35 mm for USB 3.0 to avoid insertion loss at 5 Gbps). The cables now exit the halo at a fixed 45° downward angle and route through the headgear pivot — no connector stress, zero disconnects after 200+ hours.

**Weight distribution:**

The stock design places the entire electronics payload behind the forehead. This is convenient for cable routing but terrible for comfort — the center of mass sits ~60 mm forward of the wearer's center of head rotation, creating a constant downward torque that the headgear must counter.

I moved the display driver board and USB hub from the front halo to the rear counterweight position, connected via shielded FPC extensions running along the headgear straps. This shifts the center of mass rearward by ~25 mm and reduces the perceived weight by roughly 30% (subjective, but anyone who wears it notices immediately). The forward optics weight is balanced by the rear electronics — it hangs on the head instead of pulling forward.

---

## 4. Cable Management & System Integration

### 4.1 Single-Cable Ambition

The stock Deck X revision consolidates everything to two cables: USB 3.0 (data + power) and Mini DisplayPort (video). That's already good. But every cable is a failure point. I wanted one.

**What I tried:** An MST (Multi-Stream Transport) hub that tunnels DisplayPort over USB-C Alt Mode. The idea is clean — a single USB-C cable carries video, data, and power to the headset, and a small USB-C PD sink + MST breakout inside the halo splits them back out to the driver board and hand tracking module.

**What happened:** It works at 60 Hz. At 120 Hz, the combined bandwidth of dual 1600×1440 streams at 24 bpp exceeds the USB-C Alt Mode 4-lane DP 1.4 ceiling (~25.92 Gbps raw vs ~26.5 Gbps required with overhead). It is tantalizingly close but not quite there.

**Fallback:** Two cables, but both now exit from a single custom-molded exit block at the rear of the halo. From the user's perspective, it's one managed bundle. The block is 3D-printed in TPU with internal cable guides and a ferrite bead channel for common-mode EMI suppression. Clean enough that people ask "is that one cable?" and I get to say "almost."

---

## 5. Lessons for the Next Builder

If you're building a North Star derivative and reading this, here's what I wish someone had told me on day one:

**1. Don't trust the stock reflector alignment.** Print a test pattern. Spend an hour dialing it in. Lock it with threadlocker. You will save yourself days of "why does my head hurt" debugging.

**2. The driver board needs heatsinking.** Not optional. The chip throttles silently and you will blame the panels.

**3. PETG-CF is worth the nozzle wear.** The stiffness improvement over PLA is real and the surface finish is in a different league. Get a hardened steel nozzle and commit.

**4. Cable strain relief is not cosmetic.** Intermittent DisplayPort disconnects at 120 Hz look exactly like a software bug. They're not. They're a mechanical failure that software can't fix.

**5. Move weight to the back.** Even 30 grams moved from front to rear transforms comfort. The welding mask headgear was designed for face shields that weigh almost nothing — it needs help to carry 400 g of optics + electronics comfortably.

**6. You don't need a machine shop.** Everything I did was with a consumer 3D printer, a Dremel, a tap set, a soldering iron, and patience. The kinematic mounts are the only "precision" modification and they cost under $10 in hardware.

---

## 6. What I'd Do Differently Next Time

- **Switch to a Deck X-based optical path with aftermarket reflectors.** The stock reflectors from the Release 3 kit have noticeable coating non-uniformity at the edges. There are community vendors now producing reflectors with ±2% transmittance across the full aperture — better than the original specs.
- **Design a custom driver interposer PCB.** The FPC signal conditioning should be a proper 4-layer board with controlled-impedance traces, not the hand-soldered prototype. I have the schematic roughed out; this is the next hardware spin.
- **Integrate the USB-C MST attempt into a single rigid-flex PCB.** The Alt Mode bandwidth problem is solvable with Display Stream Compression (DSC) — the panels accept DSC 1.2 decode natively but the stock driver board doesn't enable it. A firmware mod to the ANX7530's EDID table might unlock this. If it works, single-cable 120 Hz becomes real.

---

## References

- [Project North Star Official Documentation](https://docs.projectnorthstar.org)
- [Leap Motion Project North Star GitHub](https://github.com/leapmotion/ProjectNorthStar) (Apache 2.0)
- ANX7530 Display Port-to-MIPI Converter Datasheet (NDA)
- BOE VS035ZSM-NW0 Panel Specification, Rev 1.3
- Northstar Next Community Fork — reflector coating improvements
- McMaster-Carr — all fastener part numbers referenced in this document

---

*Last revised: 2026-06-07 · Built in the open, mistakes included.*
