# YIELD

**Yield-triggering Illumination & Emission Lightweight Detector**

A low-cost sensing channel that lets an autonomous vehicle detect an energetically active roadway hazards, such as:

- Fire

- Sparks

- Arcing 

and issue an emergency stop, even when the hazard is geometrically indistinguishable from harmless debris to a standard sensor stack.

> **Status: early bench-testing phase.** This repository documents active research, not a finished product. Hardware is being validated on the bench before integration onto a moving platform. See [Status](#status) below for exactly what currently works.

---

## Motivation

Autonomous vehicles make a constant, routine decision: whether to drive over a small object in the road. Geometry-based sensors (lidar, radar) and appearance-based sensors (cameras) handle this well for ordinary debris, yet none of them measure whether an object is **energetically active**. A lit firework and a crushed can can look nearly identical to a LiDAR. On July 4, 2026, this gap was made public when Waymo vehicles in San Francisco encountered lit fireworks in the roadway.

YIELD adds the missing channel: a cheap, fast sensor that listens for the specific light signature of combustion and arcing, and tells the vehicle to yield when that signature is present.

**This is a veto-only system.** YIELD can tell a vehicle *not* to proceed. It never tells a vehicle it is safe to proceed. Absence of a detected hazard is not a safety guarantee.

---

## How it works

Fire and electrical arcs emit light that **flickers** in a characteristic low-frequency band (roughly 1–20 Hz), driven by the turbulence of combustion. Steady light sources - sunlight, streetlights, headlights - do not flicker this way.

A fast photodiode samples incoming light thousands of times per second. The resulting signal is amplified, digitized, and analyzed for that flicker signature. An independent UV sensor (ground-level sunlight is nearly UV-free, so UV presence is a strong, daylight-robust confirmation) cross-checks that the detected light is combustion-type rather than confounders like a strobe, a brake light, dappled sunlight through leaves.

If both signals agree, the system issues a stop command.

```
Light → Photodiode → Transimpedance Amplifier → Digitizer → Flicker Analysis
                                                                     ↓
                                              UV Sensor → Confirmation
                                                                     ↓
                                                            STOP command
```

---

## Hardware

| Component | Role |
|---|---|
| BPW34 photodiode | Core sensing element - converts flicker into current |
| TL072 op-amp (transimpedance config) | Amplifies photodiode current into a readable voltage |
| Feedback resistor (1–10 MΩ, swept experimentally) | Sets circuit gain and detection range |
| Raspberry Pi Pico | Fast digitizer for the flicker signal (the Pi 5 has no analog input) |
| GUVA-S12SD UV sensor | Solar-blind confirmation channel |
| ADS1115 (16-bit ADC) | Digitizes the slower UV sensor signal for the Pi |
| Raspberry Pi 5 | Runs analysis, fusion logic, and issues the stop command |
| Ackermann chassis + encoders | Moving-platform test vehicle (later phase) |

**Deployed sensing cost target: under $50**, excluding instrumentation used only for testing (see below).

**Not part of the deployed sensor, testing instrumentation only:**
- RGB camera - comparison baseline
- Thermal camera / IR thermometer - independent ground truth for trial labeling
- RPLiDAR C1 - geometry baseline, illustrating what existing sensors already see

---

## Repository structure

```
/firmware        MicroPython/C for the Pico - sampling and signal streaming
/analysis        Python - FFT/wavelet flicker analysis, threshold logic, fusion
/hardware        Circuit schematics, breadboard layout notes, BOM
/data            Raw trial recordings (photodiode + UV streams, timestamps, labels)
/docs            Research plan, methodology, mentor notes, safety plan
```

*(Structure will fill in as each stage is built - see Status.)*

---

## Research questions

- Can a passive flicker channel discriminate energetic hazards from geometrically similar inert confounders - better than a geometry-only baseline?
- Is flicker alone sufficient, or is UV pairing required to reject confounders (turn signals, hazard lights, dappled sunlight)?
- Does pyrotechnic combustion (bursty, spark-driven) actually flicker in the same 1–20 Hz band established in the literature for steady flames?
- How much does platform vibration alone (no hazard present) perturb the reading?
- What is time-to-confident-detection, and what stopping distance does that imply at real driving speeds?

Full methodology, hypotheses, and trial design are in `/docs`.

---

## Status

**Currently in Phase 1: bench validation of the core sensing circuit.**

- [ ] Gate 1 - flicker recoverable via FFT at 3+ meters, indoors, controlled lighting
- [ ] UV confirmation channel integrated and bench-tested
- [ ] Confounder rejection testing (turn signals, hazard lights, brake lights, dappled sunlight)
- [ ] Vibration-only baseline (platform moving, no hazard present)
- [ ] Time-to-confidence / stopping-distance analysis
- [ ] Moving-platform integration

This list will be checked off as each stage is completed and verified - not before.

---

## Honest scope and limitations

- YIELD detects hazards that **emit light**: active combustion, arcing, and sparking. It does **not** detect inert hazards (a dead wire, an unlit spill, a crushed can).
- All reported results are a **lower bound**: hobby-grade optics underperform automotive-grade sensors, so positive findings understate what a purpose-built version could achieve.
- This is a single research-grade unit, not a validated safety system. It is not intended for deployment on a public road.

---

## Safety

Any trial involving open flame or pyrotechnics is conducted under adult/mentor supervision, with a charged fire extinguisher visible in frame, in a cleared non-flammable area, using only legal ground-based novelties and flares. No mains-voltage or high-voltage arcing is used at any stage.

---

## Motivation & background reading

- Töreyin et al. (2006), *Computer vision based method for real-time fire and flame detection*
- Erden et al. (2012), *Wavelet based flickering flame detector using differential PIR sensors*
- Khoon et al. (2012), *Autonomous fire fighting mobile platform*

---

## License
