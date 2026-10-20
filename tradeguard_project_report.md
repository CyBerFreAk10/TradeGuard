# TradeGuard: Modular Safety Wearable
**Smart India Hackathon 2025 Idea Submission**

## Project Overview
**Problem Statement Title:** AI-Enabled Diagnostic and Sensing Wearables for Skilled Trades  
**Theme:** Smart Automation  
**Category:** Hardware  

TradeGuard is a low-cost, modular wearable system designed for vocational training (ITIs). It features a universal "Core Band" and hot-swappable "Trade-Specific Sensor Pods" that detect workshop hazards, provide instant haptic/LED feedback, and log practical sessions for trainers. By replacing expensive industrial diagnostic tools with an affordable Edge AI-powered wearable, TradeGuard shifts safety training from post-incident correction to real-time, pre-incident prevention.

---

## The Solution
The system consists of two main hardware components and a software ecosystem:

1. **The Core Band:**
   * Powered by an ESP32 microcontroller with Bluetooth Low Energy (BLE).
   * Runs TinyML locally for offline, millisecond-response hazard classification.
   * Provides alerts via a haptic motor and a 3-color LED ring (Green/Yellow/Red).
2. **Trade-Specific Sensor Pods (USB-C Plug-in):**
   * **Pod A (Plumber / Gas-fitter):** Uses MQ-2/MQ-4 gas sensors and a piezoelectric contact mic to detect the acoustic "hiss" of pipeline leaks.
   * **Pod B (Welder):** A collar clip featuring an IR heat sensor (MLX90614), UV dose tracking (ML8511), and fume sensing (MQ-135).
3. **Software Ecosystem:**
   * **Trainee App:** A Flutter-based live gamified HUD displaying localized hazards.
   * **Trainer Dashboard:** A React/Firebase web app that logs sessions and generates an objective "Safety & Diagnostic Score" for NSQF assessment.

---

## Technical Approach

### Implementation Flow
1. **Sense:** Pod streams gas, sound, heat, and UV data.
2. **Classify:** TinyML on the ESP32 spots leak/hazard signatures using FFT/MFCC audio features and sensor fusion.
3. **Alert:** Device delivers instant haptic and LED feedback to the trainee.
4. **Log:** Data is synced via BLE to a local tablet or smartphone.
5. **Assess:** The trainer dashboard scores the session based on hazard avoidance and diagnostic speed.

---

## Feasibility and Viability

TradeGuard is designed to be highly affordable, costing ~92% less than entry-level commercial detectors (which often cost $200-$1000+). All components are off-the-shelf and can be assembled using standard ITI lab tools (3D printing, soldering).

### Estimated Bill of Materials (BOM)
| Component | Estimated Cost (INR) |
| :--- | :--- |
| ESP32 Dev Board | ₹350 |
| IR Temp / UV Sensors | ₹300 |
| Li-Po Battery + Charging | ₹200 |
| Piezo Mic + Op-Amp | ₹150 |
| MQ Gas Sensors | ₹120 |
| Haptic Motor & LEDs | ₹100 |
| 3D-Printed Casing | ₹80 |
| **Total per unit** | **~₹1,300 ($15–16)** |

### Challenges & Mitigations
* **Workshop Noise:** Mitigated by using a structure-borne contact piezo mic, band-pass filters, and training the ML model on real workshop noise.
* **Sensor Drift & Cross-reaction:** Mitigated by pre-heating MQ sensors before sessions and establishing per-session baselines.
* **Battery Drain:** Mitigated by heater duty-cycling; pods draw high power only when actively plugged in.
* **False Alarms:** Mitigated by introducing a yellow "caution" stage before triggering a red hazard alert, with thresholds tuned per specific trade.

---

## Impact and Benefits

**Target Audience Reach:** 
* Can be deployed across **13,000+ ITIs** in India.
* Impacts **14.7 lakh+ ITI trainees** enrolled annually (2025-26 data).

**Key Benefits:**
* **Educational:** Replaces subjective guesswork with an objective "Safety & Diagnostic Score," ready to plug into NSQF assessment criteria.
* **Social:** Forms crucial safety habits before trainees enter the workforce, reducing burns, fume inhalation, and gas incidents.
* **Economic:** The unified core band serves every trade, fitting tight ITI budgets while reducing wasted gas and damaged lab tools.
* **Environmental:** Promotes faster leak detection (saving water/gas), encourages better workshop ventilation, and reduces e-waste through modularity.

---

## Research & References
1. Ullah et al., “Pipeline Leakage Detection Using Acoustic Emission and Machine Learning Algorithms”, *Sensors*, 2023. [PMC10057666](https://pmc.ncbi.nlm.nih.gov/articles/PMC10057666)
2. “Water Flow Detection Device Based on Sound Data Analysis and Machine Learning...”, *arXiv*, 2025. [arxiv.org/abs/2501.11151](https://arxiv.org/abs/2501.11151)
3. NCVET – National Skills Qualifications Framework (NSQF). [ncvet.gov.in](https://ncvet.gov.in/?p=462)
4. National Qualification Register – NSQF levels. [nqr.gov.in](https://nqr.gov.in/nsqf)
5. AIR News, “PM-SETU completes one year” (Oct 2026). [newsonair.gov.in](https://newsonair.gov.in)
6. Edge Impulse documentation – TinyML deployment on ESP32. [docs.edgeimpulse.com](https://docs.edgeimpulse.com)
7. Espressif – ESP32 SoC. [espressif.com](https://www.espressif.com)
8. Melexis – MLX90614 sensor. [melexis.com](https://www.melexis.com)