# W02 — Edge Deployment Memo

**Product anchor:** Aido Rover (primary — the synthetic sensor set is built to mirror its IMU / motor current / proximity stack), with the same conclusions applying to Sentinel Prime AI's edge-classification role.

## Design choices

I built a small feedforward NN (210 → 64 → 32 → 4) trained on a synthetic Aido Rover sensor dataset that I generated myself, not a real one — full methodology is documented in `W02_Edge_Classifier.ipynb` §1. The short version: 30-timestep windows × 7 features (IMU, motor current L/R + diff, front/rear proximity), 4 classes (PATROL/ALERT/CHARGING/FAULT), generated as class-specific AR(1) processes with added measurement noise and ~4% label noise. I deliberately reused AURA's 30×7=210-dim window shape and its "paired sensor + difference" feature pattern, since the whole point of this task is extending AURA's methodology rather than starting over.

I chose a feedforward NN over a 1D-CNN mainly because post-training quantization of a pure `nn.Linear` stack is the most reliable path in PyTorch — I wanted the quantization and latency numbers in this memo to be trustworthy, not something I had to caveat because the quantization step itself was shaky.

One thing worth flagging up front: my first pass at the dataset generator produced classes that were too easy to separate — 100.000% test accuracy on every metric, which isn't a result I'd trust. I added measurement noise and label noise (justified by AURA's own labels coming from a manually-pressed button, which is imperfect at mode transitions) and re-ran everything. The numbers below are from that second, more honest version.

## Accuracy–latency trade-off

Measured on a held-out test split (never used for training or tuning), 200 warm-up-free single-sample (batch=1) CPU inference runs after 20 untimed warm-up calls:

| | FP32 | INT8 (dynamic PTQ) |
|---|---|---|
| Accuracy | 87.22% | 87.22% (identical predictions) |
| Precision / Recall / F1 (macro) | 0.876 / 0.873 / 0.871 | same |
| Latency (mean ± std) | 0.0171 ± 0.0009 ms | 0.4666 ± 0.0577 ms |
| Model size | 0.066 MB | 0.021 MB (−68.6%) |

Two things stand out, and I want to be upfront about both rather than only reporting the flattering one:

1. **Quantization didn't cost any accuracy** — same predictions on the test set, so there's no accuracy penalty to weigh against anything.
2. **Quantization made inference slower here, not faster** — about 27x slower. This isn't a mistake in the benchmark; dynamic quantization computes activation quantization parameters at runtime on every call, and at batch=1 on a network this tiny (three `Linear` layers, sub-millisecond either way), that overhead is bigger than what int8 matrix multiplication saves. A larger model would very likely show the opposite result. I'd rather state this plainly than quietly pick the framing that makes quantization look better than it measured.

## Latency budget and recommendation

**Assumed budget: 20ms per inference**, for a real-time decision loop on Aido Rover or Sentinel Prime AI (consistent with the plan's own example figure). Both configurations clear this by a wide margin — FP32 by roughly 1,170x, INT8 by roughly 43x — so latency alone doesn't decide anything here; either model is instant relative to the budget.

**Recommendation: deploy the INT8 model anyway**, on model size rather than speed. The 68.6% size reduction, with zero accuracy cost, is the real reason to prefer it — not latency, which quantization actually made worse here. This matters more once this classifier isn't the only model on the device: Sentinel Prime AI's own public spec runs a five-model ensemble on one Jetson AGX Orin (see `W01_InGen_NN_Landscape.md` §3), and Aido Rover's 14-sensor detection stack implies something similar. Memory and compute budget shared across several models is a more binding constraint at this parameter count than per-model latency, so I'd rather ship the smaller model when it costs nothing in accuracy — even knowing it isn't the faster one.

If this classifier were meaningfully larger — closer to what Sentinel's actual ensemble members probably are, not a 210-input toy network — I'd expect quantization to win on both size and latency, and I'd want to re-run this exact benchmark rather than assume the conclusion carries over.

## Connection back to AURA

AURA's actual deployed pipeline (`ESP32/main.py` + `model_training/server.py`, documented in `EVALUATE_AND_ITERATE_WORKFLOW.md`) never quantizes anything and never benchmarks latency — it retrains an sklearn `MLPClassifier` on the full accumulated dataset and calls `.predict()` directly, with no accuracy/precision/recall/F1 step and no deployment-sizing decision at all. This task is effectively the deployment-engineering half AURA's on-device interaction loop is missing: AURA proved the sensor-windowing and lightweight-classifier approach works end-to-end on real hardware, but never had to answer "should this be quantized, and does it fit the latency budget" — because ESP32 inference happens on the *server*, not on the microcontroller itself, so there was never a hard edge-latency constraint forcing that question. Aido Rover and Sentinel Prime AI, running inference on the device itself, do have that constraint, which is exactly what this memo's benchmark is answering that AURA's design never needed to.
