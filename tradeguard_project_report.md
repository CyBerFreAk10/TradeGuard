# TradeGuard
### A Modular Edge-AI Wearable for Real-Time Safety and Diagnostic Training in Skilled Trades

**Smart India Hackathon 2025 — Idea Submission | Technical Project Report**

| | |
|---|---|
| **Problem Statement Title** | AI-Enabled Diagnostic and Sensing Wearables for Skilled Trades |
| **Problem Statement ID** | _to be filled_ |
| **Theme** | Smart Automation |
| **PS Category** | Hardware |
| **Team Name / Team ID** | _to be filled_ |
| **Document Version** | v1.0 |

---

> **One core. Many trades. Zero blind spots.**
> TradeGuard turns safety training from *"the trainer noticed after it happened"* into *"the trainee felt it before it happened"*, at about ₹1,300 per unit.

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [The Problem](#2-the-problem)
3. [Solution Overview](#3-solution-overview)
4. [System Architecture](#4-system-architecture)
5. [Hardware Design](#5-hardware-design)
6. [Edge AI and Firmware](#6-edge-ai-and-firmware)
7. [Software Ecosystem](#7-software-ecosystem)
8. [The Objective Safety Score and NSQF Integration](#8-the-objective-safety-score-and-nsqf-integration)
9. [Feasibility and Viability](#9-feasibility-and-viability)
10. [Risk Analysis and Mitigation](#10-risk-analysis-and-mitigation)
11. [Competitive Landscape](#11-competitive-landscape)
12. [Impact and Benefits](#12-impact-and-benefits)
13. [Implementation Roadmap](#13-implementation-roadmap)
14. [Validation Plan and Success Metrics](#14-validation-plan-and-success-metrics)
15. [Scalability and Sustainability](#15-scalability-and-sustainability)
16. [Conclusion](#16-conclusion)
17. [Research and References](#17-research-and-references)

---

## 1. Executive Summary

India's Industrial Training Institutes (ITIs) train lakhs of plumbers, fitters and welders every year. They learn in workshops full of pressurised pipes, open flames, UV-intense welding arcs and toxic fumes. Today, safety in these workshops is taught **reactively**: an instructor spots an unsafe act after it has happened. Practical skills like leak finding are graded **subjectively**, by watching.

**TradeGuard** is a low-cost, modular wearable system that fixes both problems at once:

- A universal **Core Band** (ESP32 + on-device TinyML + haptic motor + tri-colour LED ring) is worn on the wrist or collar.
- Cheap, hot-swappable **Trade Pods** plug into the band to add trade-specific sensing: an acoustic + gas **Leak Pod** for plumbers and gas-fitters, and a heat + UV + fume **Weld Pod** for welders.
- An on-device ML model classifies hazards and leak signatures **offline, in well under a second**, and warns the trainee instantly through vibration and light.
- Every session is logged over BLE to a **Trainer Dashboard** that produces an objective, auditable **Safety & Diagnostic Score**, designed to plug into **NSQF** practical assessment.

| Key figure | Value |
|---|---|
| Target unit cost (prototype BOM) | **~₹1,300 (≈ US$15–16)** |
| Cost vs. typical commercial single-purpose detectors (US$200–1,000+) | **~92% lower or more** |
| Addressable institutions | **13,000+ ITIs** |
| Addressable learners | **~14.7 lakh trainees (2025-26 enrolment)** |
| Internet required for alerts | **None — fully on-device inference** |

---

## 2. The Problem

### 2.1 Context

Skilled-trade training in India runs through a very large ITI network. Hands-on modules such as **Plumber**, **Fitter** and **Welder (Gas & Electric)** expose trainees to real hazards from their first weeks: gas leaks, hot work, arc radiation and fume build-up.

### 2.2 Three gaps in current practice

| Gap | What happens today | Consequence |
|---|---|---|
| **1. Reactive safety teaching** | The instructor corrects an unsafe act *after* it is observed. One instructor supervises many trainees at once. | Unsafe habits form unnoticed, and near-misses go unrecorded. |
| **2. Subjective assessment** | Practical skills (leak finding, safe stance, ventilation awareness) are graded by visual judgement. | Grades are inconsistent between assessors and centres, and there is no data trail. |
| **3. Unaffordable instrumentation** | Each diagnostic or safety instrument (ultrasonic leak detector, gas detector, fume monitor) is a separate, costly device. | Most ITIs cannot equip every trainee or every trade, so trainees rarely train with instruments at all. |

### 2.3 The opportunity

If every trainee wore a cheap, trade-aware sensor that **warned them in real time** and **recorded what actually happened**, safety would be taught *before* the incident and skill would be assessed with *evidence*. That is what TradeGuard does.

---

## 3. Solution Overview

### 3.1 Design principles

| Principle | How TradeGuard applies it |
|---|---|
| **Modular, not monolithic** | One Core Band serves every trade. Only the inexpensive sensor pod changes. |
| **Edge-first** | All hazard detection runs on the device. Alerts never depend on Wi-Fi, a phone or the cloud. |
| **Felt, not watched** | Haptic alerts get through noise, gloves and welding helmets, where screens and buzzers fail. |
| **Evidence by default** | Every alert, acknowledgement and diagnostic result is time-stamped and logged. |
| **ITI-buildable** | Off-the-shelf parts, a 3D-printed enclosure and through-hole-friendly assembly. Trainees can build and repair it themselves. |

### 3.2 What the trainee experiences

| LED state | Haptic pattern | Meaning |
|---|---|---|
| 🟢 **Green** | none | Safe. Normal operation. |
| 🟡 **Yellow** | 2 short pulses | Approaching a threshold (heat proximity, rising fumes, UV dose at 75%), or a leak is "getting warmer". |
| 🔴 **Red** | continuous strong pulse | Danger threshold crossed, or a leak is confirmed. Act now. |

### 3.3 What the trainer gets

A live classroom view of every band, a per-trainee session timeline, and an end-of-session **Safety & Diagnostic Score** with the evidence behind it.

---

## 4. System Architecture

```
 ┌───────────────────────────┐        ┌───────────────────────────┐
 │  POD A — LEAK POD         │        │  POD B — WELD POD         │
 │  Piezo contact mic        │        │  MLX90614 IR temperature  │
 │  + op-amp preamp          │        │  ML8511 UV sensor         │
 │  MQ-2 / MQ-4 gas sensor   │        │  MQ-135 fume sensor       │
 │  Pod-ID resistor          │        │  Pod-ID resistor          │
 └─────────────┬─────────────┘        └─────────────┬─────────────┘
               │      USB-C keyed connector         │
               │  (power, I²C, analog, pod-ID)      │
               └─────────────────┬──────────────────┘
                                 ▼
 ┌──────────────────────────────────────────────────────────────────┐
 │  CORE BAND                                                       │
 │  ESP32  ─  Pod detection → sensor drivers → DSP → TinyML         │
 │          → sensor-fusion state machine → alert engine            │
 │  Haptic motor  ·  RGB LED ring  ·  Li-Po + charger  ·  RTC log   │
 └─────────────────────────────────┬────────────────────────────────┘
                                   │  BLE (GATT notifications)
               ┌───────────────────┴───────────────────┐
               ▼                                       ▼
 ┌───────────────────────────┐        ┌───────────────────────────┐
 │  TRAINEE APP (Flutter)    │        │  TRAINER HUB (tablet)     │
 │  Live gamified HUD        │        │  Multi-band BLE gateway   │
 │  Hot/cold leak dial       │        │  Class live view          │
 └───────────────────────────┘        └─────────────┬─────────────┘
                                                    │  sync when online
                                                    ▼
                                      ┌───────────────────────────┐
                                      │  TRAINER DASHBOARD        │
                                      │  React + Firebase         │
                                      │  Scores, reports, NSQF    │
                                      │  export                   │
                                      └───────────────────────────┘
```

### 4.1 Data flow — the five-stage pipeline

| Stage | Where it runs | What happens | Target latency |
|---|---|---|---|
| **1. Sense** | Pod → Core Band | Sensors are sampled: audio frames, gas resistance, IR temperature, UV index. | continuous |
| **2. Classify** | ESP32 | DSP feature extraction, then TinyML inference and rule-based fusion. | < 250 ms per window |
| **3. Alert** | Core Band | The state machine drives the haptic motor and LED ring. | < 100 ms after a decision |
| **4. Log** | Core Band → Hub | Events are buffered on the band and pushed over BLE when connected. | best-effort, no data loss |
| **5. Assess** | Dashboard | The session is scored, checked against the rubric and exported. | end of session |

---

## 5. Hardware Design

### 5.1 Core Band

| Block | Component | Role | Notes |
|---|---|---|---|
| MCU | **ESP32** (dual-core, 240 MHz, Wi-Fi + BLE) | Sensor I/O, DSP, TinyML, BLE | An ESP32-S3 can be used later for faster vector maths in ML. |
| Feedback | Coin **ERM/LRA vibration motor** + MOSFET driver | Primary alert channel | Several pulse patterns encode severity. |
| Feedback | **RGB LED ring** (addressable) | Secondary, glanceable alert | Green / Yellow / Red states. |
| Power | **Li-Po cell** + USB-C charging IC + protection | Full-shift operation target | Fuel-gauge reading sent to the app. |
| Power | 5 V boost rail (pod-switched) | Powers the MQ sensor heaters | Enabled only when a gas-capable pod is attached. |
| Interface | **Keyed USB-C pod connector** | Power, I²C, analog line, pod-ID | A resistor divider identifies the pod type when it is plugged in. |
| Enclosure | 3D-printed, flame-retardant filament, strap or collar clip | Wearable housing | Designed for gloved operation. |

### 5.2 Pod A — Leak Pod (Plumber / Gas-fitter)

| Sensor | Measures | How it is used |
|---|---|---|
| **Piezoelectric contact transducer** + op-amp preamp | Structure-borne vibration in the pipe wall | Touched to the pipe, it picks up the turbulent-flow "hiss" signature of a pressurised leak and rejects much of the airborne workshop noise. |
| **MQ-2 / MQ-4** metal-oxide gas sensor | Combustible gases (LPG, methane) | Confirms gas leaks and drives a proximity gradient on the "hot/cold" dial. |

**Diagnostic workflow:** the trainee sweeps the probe along a pipe run. The acoustic classifier's confidence and the gas reading combine into a **proximity score**. This drives the Yellow → Red transition and the hot/cold dial in the app. A pinpointed leak is logged with its time-to-locate.

### 5.3 Pod B — Weld Pod (Welder)

| Sensor | Measures | How it is used |
|---|---|---|
| **MLX90614** non-contact IR thermometer | Surface / object temperature in the field of view | Tracks how close the trainee is to the hot work zone and flags unsafe heat exposure. |
| **ML8511** UV sensor | UV intensity (UV-A/UV-B band) | Integrated over time into a **cumulative UV dose** for each session, with an alert at 75% and 100% of a set limit. |
| **MQ-135** air-quality sensor | Broad response to fumes and pollutant gases | Trend alarm for fume build-up, which signals poor ventilation. |

### 5.4 Indicative power budget (design targets)

| Load | Mode | Notes |
|---|---|---|
| ESP32 + BLE | Duty-cycled inference | Main continuous load. |
| Haptic motor | Event-driven | Only runs during alerts. |
| LED ring | Low-brightness steady state | Brightness steps up on alert. |
| **MQ heater** | **Largest single load** | Mitigated by heater duty-cycling and powering pods only while attached (see §10). |

> **Design target:** a full practical session (3–4 h) on a single charge with the Weld Pod attached, and one full session with the Leak Pod's gas heater duty-cycled. These targets will be checked with real current measurements in Phase 1.

---

## 6. Edge AI and Firmware

### 6.1 Firmware architecture

```
 ┌──────────────────────────────────────────────────────────┐
 │  APPLICATION     Alert engine · Session logger · BLE API │
 ├──────────────────────────────────────────────────────────┤
 │  INTELLIGENCE    TinyML classifier · Fusion state machine│
 ├──────────────────────────────────────────────────────────┤
 │  SIGNAL          Band-pass filter · FFT / MFCC · Baseline│
 ├──────────────────────────────────────────────────────────┤
 │  DRIVERS         I²S/ADC audio · I²C sensors · Pod-ID    │
 ├──────────────────────────────────────────────────────────┤
 │  RTOS            FreeRTOS tasks (ESP-IDF / Arduino core) │
 └──────────────────────────────────────────────────────────┘
```

Sampling, inference, alerting and BLE run as separate FreeRTOS tasks. A slow BLE link therefore never delays a safety alert.

### 6.2 Acoustic leak model — ML pipeline

| Step | Approach |
|---|---|
| **Data collection** | A test rig of PVC and metal pipe runs with drilled pinhole leaks at varied pressures (air compressor), recorded through the actual piezo pod. |
| **Negative data** | Real ITI workshop noise: grinders, hammering, compressors, voices, flowing pipes with no leak. |
| **Labels** | `leak`, `no_leak`, `handling_noise` (probe movement), `impact` |
| **Features** | Band-pass filtering, then FFT spectral features / MFCCs over short overlapping windows. |
| **Model** | A small 1-D CNN or dense network trained in **Edge Impulse**, quantised to **int8** and deployed as a C++ library on the ESP32. |
| **Post-processing** | Temporal smoothing (majority vote over N windows) to suppress single-window false positives. |
| **Target metrics** | ≥ 90% leak recall at the pinhole sizes used in training. False-alarm rate low enough not to cause alert fatigue (tuned in pilot). |

Published work supports this direction. Acoustic-emission and machine-learning approaches have reported high classification accuracy for pipeline leaks and pinhole sizes in controlled settings [1], and sound-based ML devices have been demonstrated for detecting water leakage [2].

### 6.3 Hazard fusion — Weld Pod logic

The Weld Pod uses a **transparent rule-plus-model state machine**, not a black box. Instructors therefore know exactly why an alert fired:

| Input | Yellow condition | Red condition |
|---|---|---|
| IR temperature | Rising quickly toward the unsafe proximity threshold | Above the unsafe proximity threshold |
| Cumulative UV dose | ≥ 75% of the session limit | ≥ 100% of the session limit |
| Fume trend (MQ-135) | Sustained rise above the session baseline | Sustained high level, meaning the ventilation has failed |

All thresholds are **configurable per trade and per exercise** from the Trainer Dashboard.

### 6.4 Event schema (logged per alert)

```json
{
  "band_id": "TG-0417",
  "trainee_id": "ITI-WLD-2026-112",
  "session_id": "S-20261021-03",
  "pod": "WELD",
  "event": "UV_DOSE_RED",
  "value": 1.02,
  "ts": "2026-10-21T10:42:17+05:30",
  "acknowledged_ms": 1800
}
```

---

## 7. Software Ecosystem

### 7.1 Trainee App (Flutter, Android)

- **Live HUD:** current pod, live readings, alert state.
- **Hot/cold leak dial:** turns leak hunting into a game with real-time proximity feedback.
- **Post-session replay:** a timeline of every alert, so trainees learn from their own data.

### 7.2 Trainer Hub and Dashboard (React + Firebase)

| Feature | Description |
|---|---|
| **Class live view** | Every connected band in one grid. Red events surface immediately. |
| **Exercise configuration** | Pick the trade, exercise and thresholds, then push them to all bands. |
| **Session reports** | Per-trainee timeline, alert counts, response times, diagnostic results. |
| **Safety & Diagnostic Score** | Auto-computed and explainable (see §8). |
| **NSQF export** | Assessment-ready reports for the practical-exam record. |
| **Offline-first** | The hub caches sessions locally and syncs to the cloud when connected. |

---

## 8. The Objective Safety Score and NSQF Integration

### 8.1 Scoring model (proposed)

Each session produces two scores from 0 to 100.

**Safety Score**

```
Safety = 100 − (w_R × RedEvents) − (w_Y × UnacknowledgedYellowEvents) − (w_T × SlowResponsePenalty)
```

**Diagnostic Score** (Leak Pod exercises)

```
Diagnostic = LeakFound × (Base + TimeBonus) − FalseIdentificationPenalty
```

The weights (w) and time limits are set by the institute or the assessing body. **Every deduction links back to a logged event**, so a score can be audited and explained to the trainee.

### 8.2 Current vs. proposed NSQF practical assessment

| Aspect | Current practice | With TradeGuard |
|---|---|---|
| Evidence | The assessor's observation | Time-stamped sensor log plus the assessor's observation |
| Consistency | Varies by assessor and centre | Same thresholds and rubric everywhere |
| Safety coverage | Only what the assessor happens to see | Every exposure event, the whole session |
| Feedback to trainee | Verbal, after the fact | Instant haptic during the task, plus a replay afterwards |

### 8.3 Proposed "Digital Safety Compliance" module — example criteria

- **Welder:** finish a gas-cutting exercise with **zero Red-zone heat or fume violations**.
- **Plumber / Gas-fitter:** locate a **hidden pipe leak within 3 minutes** using the acoustic pod.
- **All trades:** acknowledge every Yellow alert within a set response time.

---

## 9. Feasibility and Viability

### 9.1 Technical feasibility

- Every core technology is **mature and well documented**: ESP32, Edge Impulse deployment to ESP32, I²C IR and UV sensors, and MQ-series gas sensors.
- The **Phase 1 proof-of-concept** (the acoustic leak demo) needs only a PVC pipe, a pinhole and an air compressor. These are standard ITI workshop items.
- The team's skill set covers embedded firmware, analog front-end design, ML and app development.

### 9.2 Bill of Materials (prototype, per unit)

| Component | Purpose | Est. cost (INR) |
|---|---|---:|
| ESP32 development board | Processing, BLE | ₹350 |
| MLX90614 IR + ML8511 UV sensors | Heat and UV sensing (Weld Pod) | ₹300 |
| Li-Po battery + charging/protection | Power | ₹200 |
| Piezo contact transducer + op-amp | Acoustic leak sensing (Leak Pod) | ₹150 |
| MQ-series gas sensors | Gas and fume detection | ₹120 |
| Haptic motor + LED ring | Trainee feedback | ₹100 |
| 3D-printed enclosure | Housing | ₹80 |
| **Total (Core Band + both pods)** | | **~₹1,300 (≈ US$15–16)** |

### 9.3 Economic viability

| Item | TradeGuard | Typical alternative |
|---|---|---|
| Instruments needed for 2 trades | 1 Core Band + 2 pods | 2+ separate commercial instruments |
| Approximate cost | **~₹1,300** | **US$200–1,000+ per instrument** |
| Data logging for assessment | Built in | Usually absent or proprietary |
| Repairable in-house by ITI | Yes | Rarely |

At volume (custom PCB instead of dev boards, bulk sensors), unit cost is expected to **fall further**. A class set of 20 bands with pods costs roughly what **one** mid-range commercial instrument does.

---

## 10. Risk Analysis and Mitigation

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|:---:|:---:|---|
| R1 | Workshop noise masks the leak signature | High | High | Structure-borne contact piezo (not an air mic), band-pass filtering, training on real workshop noise, temporal smoothing. |
| R2 | MQ sensor warm-up, drift and cross-sensitivity | High | Medium | Pre-heat at session start, per-session baseline, used as a **trend alarm** rather than an absolute concentration meter. |
| R3 | Battery drain from MQ heaters | Medium | Medium | Heater duty-cycling, pod-switched 5 V rail, low-power BLE intervals. |
| R4 | Heat and spatter near the welding arc | Medium | High | Non-contact IR from the collar, flame-retardant enclosure, pod positioned away from the spatter zone. |
| R5 | False alarms cause alert fatigue | Medium | High | Yellow "caution" stage before Red, per-trade thresholds, smoothing, pilot-tuned limits. |
| R6 | Trainer and institutional adoption | Medium | High | Pilot in 1–2 ITIs; score introduced **alongside** the existing rubric, not instead of it. |
| R7 | Over-reliance on the device | Low | High | Positioned as a **training aid**, not certified PPE. Standard PPE and supervision stay mandatory. |
| R8 | Trainee data privacy | Low | Medium | Pseudonymous IDs on the band, role-based dashboard access, data used only for training and assessment. |

> **Important:** TradeGuard is a **training and assessment aid**. It does **not** replace certified gas detectors, welding PPE or instructor supervision.

---

## 11. Competitive Landscape

| Capability | Commercial ultrasonic leak detector | Personal gas monitor | Fume / air-quality monitor | **TradeGuard** |
|---|:---:|:---:|:---:|:---:|
| Leak localisation | ✅ | ⚠️ gas only | ❌ | ✅ |
| Heat / UV exposure tracking | ❌ | ❌ | ❌ | ✅ |
| Fume trend detection | ❌ | ⚠️ | ✅ | ✅ |
| Wearable haptic alerts | ❌ | ⚠️ | ❌ | ✅ |
| Multi-trade with one device | ❌ | ❌ | ❌ | ✅ |
| Training scores and session logs | ❌ | ❌ | ❌ | ✅ |
| Built for ITI budgets | ❌ | ❌ | ❌ | ✅ |

**Our differentiator:** existing tools are *instruments for professionals*. TradeGuard is a *training system for learners*. It combines sensing, coaching and assessment in one modular, affordable platform.

---

## 12. Impact and Benefits

### 12.1 Reach

| Metric | Value |
|---|---|
| ITIs in India | **13,000+** |
| ITI trainees enrolled (2025-26) | **~14.7 lakh** |
| Trades covered at launch | Plumber, Gas-fitter, Welder |
| Trades addressable with new pods | Electrician, Fitter, Machinist, and more |

### 12.2 Benefits by stakeholder

| Stakeholder | Benefit |
|---|---|
| **Trainees** | Instant, private feedback. Safety habits formed before the first job. A data-backed record of competence. |
| **Trainers** | One instructor can watch the whole class's safety state at once. Objective evidence instead of guesswork. |
| **ITIs / DGT** | Affordable instrumentation for every trainee. Consistent, auditable assessment across centres. |
| **Employers** | Safety-aware, job-ready hires with verifiable practical records. |

### 12.3 Benefits by dimension

| Dimension | Benefit |
|---|---|
| **Social** | Fewer burns, fume exposures and gas incidents during training. A stronger safety culture across the workforce. |
| **Economic** | One core serves every trade. Less wasted gas and fewer damaged tools in labs. Low capital cost. |
| **Educational** | An objective Safety & Diagnostic Score, ready for NSQF integration. Replayable sessions for learning. |
| **Environmental** | Faster leak detection cuts gas and water loss. Modular, repairable pods reduce e-waste. Fume alerts push better ventilation. |

---

## 13. Implementation Roadmap

| Phase | Duration | Deliverables |
|---|---|---|
| **Phase 0 — Design** | Weeks 1–2 | Requirements frozen, schematics, enclosure CAD, pipe test rig built. |
| **Phase 1 — Leak Pod PoC** | Weeks 3–6 | Core Band + Leak Pod working. Dataset collected. TinyML model deployed. **Live pinhole-leak demo.** |
| **Phase 2 — Weld Pod + App** | Weeks 7–10 | Weld Pod fusion logic, Trainee App HUD, power measurements. |
| **Phase 3 — Dashboard + Scoring** | Weeks 11–13 | Trainer Hub, Dashboard, Safety & Diagnostic Score, report export. |
| **Phase 4 — Pilot** | Weeks 14–20 | Field trial in 1–2 ITIs, threshold tuning, trainer feedback, v2 hardware. |
| **Phase 5 — Scale** | Beyond | Custom PCB, more trade pods, partnership with DGT / state skill missions. |

### Hackathon demo plan

1. A pressurised PVC pipe with a hidden pinhole leak on the table.
2. A judge wears the band and sweeps the probe. The hot/cold dial climbs, the LED turns Yellow, then Red with a strong buzz at the leak.
3. A grinder runs nearby to show the system rejects workshop noise.
4. The dashboard updates live with the event log and the session score.

---

## 14. Validation Plan and Success Metrics

| Area | Metric | Target (prototype) |
|---|---|---|
| Leak detection | Recall on test-rig pinhole leaks | ≥ 90% |
| Leak detection | False alarms under workshop noise | Low enough to avoid alert fatigue (tuned in pilot) |
| Responsiveness | Sensor event → haptic alert | < 0.5 s |
| Endurance | Runtime with the Weld Pod | One full practical session (3–4 h) |
| Usability | Trainees who notice haptic alerts with gloves on and while welding | ≥ 95% in pilot |
| Learning outcome | Change in Red-zone events from first to last pilot session | Measurable reduction |
| Assessment | Agreement between the TradeGuard score and expert assessor grades | Strong correlation in pilot |

---

## 15. Scalability and Sustainability

- **Pod ecosystem:** new trades need only a new pod plus a new model. Possible pods include an electrician pod (voltage proximity), a machinist pod (vibration and noise dose) and a confined-space pod (O₂/CO).
- **Open hardware approach:** published schematics and enclosures let ITIs build and repair bands *as part of the curriculum*. The device becomes a training project in its own right.
- **Deployment model:** class sets (e.g. 20 bands + 1 hub) procured through institute modernisation budgets. ITI upgrade programmes such as **PM-SETU** are a natural route for this.
- **Data at scale:** anonymised aggregate data can show which exercises cause the most hazard events. This feeds back into curriculum design.

---

## 16. Conclusion

TradeGuard brings three things into a ₹1,300 wearable that ITIs have never had together: **real-time hazard coaching**, **trade-specific diagnostic training** and **objective, evidence-based assessment**. Its modular core-and-pod design makes it affordable at the scale of India's 13,000+ ITIs. Its edge-AI architecture makes it reliable in the noisy, offline conditions of a real workshop. Its scoring model gives NSQF a practical path to *Digital Safety Compliance*.

**Safety learned before the incident. Skill proven with evidence. One core for every trade.**

---

## 17. Research and References

1. Ullah, N. et al. "Pipeline Leakage Detection Using Acoustic Emission and Machine Learning Algorithms." *Sensors*, 2023. https://pmc.ncbi.nlm.nih.gov/articles/PMC10057666
2. "Water Flow Detection Device Based on Sound Data Analysis and Machine Learning to Detect Water Leakage." *arXiv*, 2025. https://arxiv.org/abs/2501.11151
3. National Council for Vocational Education and Training (NCVET). "National Skills Qualifications Framework (NSQF)." https://ncvet.gov.in/?p=462
4. National Qualification Register. "NSQF Levels." https://nqr.gov.in/nsqf
5. All India Radio News. "PM-SETU completes one year, aims to modernise Government ITIs." October 2026. https://newsonair.gov.in/pm-setu-completes-one-year-aims-to-modernise-government-itis/
6. Edge Impulse. Documentation — deploying TinyML models to microcontrollers including ESP32. https://docs.edgeimpulse.com
7. Espressif Systems. ESP32 SoC. https://www.espressif.com/en/products/socs/esp32
8. Melexis. MLX90614 Infrared Thermometer. https://www.melexis.com/en/product/MLX90614

---

*Prepared for Smart India Hackathon 2025 — Idea Submission. All costs are prototype estimates. Performance figures are design targets to be validated in Phases 1–4.*
