# W01 — InGen Dynamics Product & PIC 2.0 Landscape

Sources: public pages at `ingendynamics.com` — `/` (homepage), `/fari.html`, `/senpai.html`, `/sentinel.html`, `/rover.html`, `/humanoid.html`, `/origami.html`. Accessed 2026-07-14/15. All figures below are the site's own stated "design targets," not independently validated results — noted explicitly where relevant.

## 1. Fari — Eldercare Companion Robot

**Target use case:** Health monitoring and companionship for elderly individuals, positioned for home care and care-facility settings ("companion every elder deserves").

**Primary ML inference type:** Mixed — **classification** for health-event and fall detection, **generation** for conversational companionship. Runs on Jetson Orin NX at the edge.

**Key constraint:** Safety and dignity. Fari carries SEOM at λ=10.0 — the highest safety weighting InGen states across its product line — and is explicitly designed so "humans retain decision-making control; AI serves an advisory role." Regulatory framing centers on NHS-aligned standards and data privacy for a vulnerable user population.

## 2. Senpai — AI Educational Companion

**Target use case:** K-12 and early learners, including six dedicated modes for special educational needs (SEND).

**Primary ML inference type:** **Generation** (multilingual instruction and tutoring dialogue, 40+ languages) with a **classification** component for selecting the appropriate SEND adaptation mode per student.

**Key constraint:** Groundedness and age-appropriateness. SEOM is configured at λ=4.0 (lower than Fari, reflecting a less physically vulnerable population) with 12 safety rules aimed at keeping responses educationally accurate and appropriate for children.

## 3. Sentinel Prime AI — Enterprise Physical Security

**Target use case:** SOC operators and security supervisors across critical infrastructure, healthcare, retail, data centers, government, and transportation.

**Primary ML inference type:** **Classification** — 27 simultaneous detection functions (weapons, aggression, face recognition, ANPR, fire/hazmat, abandoned objects) via a five-model ensemble (VisionNet, ThermalNet, PoseNet, AudioNet, plus a STUM uncertainty gate) fusing 14 sensor modalities.

**Key constraint:** Latency and false-alert suppression. Stated target is <200ms end-to-end inference with the STUM gate explicitly built to suppress low-confidence detections before they reach a human operator — "AI advises; human decides" is stated as a hardware-enforced (not just software) SEOM rule.

## 4. Aido Rover — Autonomous Outdoor Patrol & Inspection

**Target use case:** Outdoor perimeter security and infrastructure inspection — solar/wind farms, pipelines, airports, agriculture — where continuous human coverage isn't practical.

**Primary ML inference type:** **Classification** — 25 AI detection functions spanning perimeter security, behavior analysis, vehicle classification, and infrastructure-anomaly detection (solar panel defects, gas leaks, crop health), fed by a 14-sensor fusion stack (LiDAR, thermal, multispectral, RGB, acoustic, radar, gas).

**Key constraint:** Reliability under uncontrolled outdoor conditions. Site explicitly states "uncertainty scoring before any autonomous action," and — notably — critical safety functions are designed onto a **dedicated hardware microcontroller independent of the main AI compute stack**, so a software/AI failure can't disable the safety layer. Designed for fleet coordination up to 12 units.

## 5. Aido Humanoid — General-Purpose Bipedal Robot

**Target use case:** Healthcare, eldercare, manufacturing, and logistics — human-centric environments (doorways, stairs, shared workspaces), no environment modification assumed. R&D stage, not yet in the same "in development" tier as the other four.

**Primary ML inference type:** **Planning / policy** — a three-layer stack: GR00T N1.6 foundation model → GRPO fine-tuning for manipulation/locomotion policy → Origami AI platform integration. This is the one InGen platform whose core ML problem is sequential decision-making/control rather than perception classification.

**Key constraint:** Real-time safety under physical human contact. 12 hardwired SEOM rules (5N max force limiting, proxemic zones, consent detection, fall prevention) with a stated <2ms E-stop latency — the tightest latency figure of any InGen platform, reflecting the consequence severity of a 70kg bipedal robot operating near people.

## 6. PIC 2.0 Architecture Implications

InGen's `/origami.html` page publicly discloses PIC 2.0 as a **sequential six-model pipeline**, each model owning one stage of a sensor-to-action loop, targeting ~142ms end-to-end on edge hardware with no cloud dependency:

| Order | Model | Stage | Design-target latency |
|---|---|---|---|
| 01 | AMDC | Sensor calibration | ~8ms |
| 02 | STUM | Uncertainty quantification | ~12ms |
| 03 | HTD-IRL | Task planning | ~38ms |
| 04 | GRPO | Policy decision | ~12ms |
| 05 | SEOM | Safety gate | <1ms |
| 06 | CRL-MRS | Fleet coordination | ~60ms |

Three implications for how the five products above actually use this shared stack:

1. **Every product reuses the same six-model spine, but tunes SEOM's λ weight to the vulnerability of its user population** — Fari and Aido Humanoid (physical contact with vulnerable people) sit at λ=10.0, Senpai (children, but no physical contact) sits lower at λ=4.0. Safety strictness is a per-product configuration knob on a shared safety-training mechanism, not a per-product safety feature.
2. **The pipeline is explicitly edge-resident and cloud-independent** — this is consistent across all five products (Sentinel's <200ms target, Aido Humanoid's <2ms e-stop, Aido Rover's hardware-independent safety cutoff) and is the single constraint that most differentiates this stack from a typical cloud-inference ML product: the safety gate (SEOM) and the uncertainty gate (STUM) both have to run inline, on-device, before any actuation — not as an offline evaluation step.
3. **AMDC (sensor calibration) has no clean analogue in software-only, non-embedded ML systems** — it's the one PIC 2.0 stage that exists specifically because physical sensors drift with temperature/vibration/wear; this is the clearest single signal that PIC 2.0 was designed for embedded/physical deployment first, software-agentic deployment second (relevant directly to the OpenClaw bridge exercise in `W01_OpenClaw_Bridge.md`).
