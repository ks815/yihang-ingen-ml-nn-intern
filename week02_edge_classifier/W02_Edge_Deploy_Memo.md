# W02 — Edge Deployment Memo

**Product anchor:** Aido Rover (the sensor set mirrors its IMU / motor current / proximity setup), same conclusions apply to Sentinel Prime AI's edge-classification role.

## Design choices

Small feedforward NN (210 → 64 → 32 → 4) on a synthetic Aido Rover dataset I generated myself, not real data. Full method is in `W02_Edge_Classifier.ipynb` §1. Short version: 30-step windows, 7 features (IMU, motor current L/R + diff, proximity front/rear), 4 classes (PATROL/ALERT/CHARGING/FAULT), generated with AR(1) processes plus measurement and label noise. Reused AURA's 210-dim window shape and paired-sensor-plus-diff pattern since this task is extending AURA, not starting over. Dataset size (4,000, 1,000/class) and the 70/15/15 split were confirmed with the supervisor.

Went with feedforward over 1D-CNN mainly because quantizing a plain Linear stack is the reliable path in PyTorch, and I wanted the latency numbers here to actually mean something.

First pass at the data generator was too easy to separate: 100% test accuracy, not believable. Added noise and reran. Numbers below are from the corrected, full-size run.

## Accuracy-latency trade-off

Held-out test split, 200 warm-up-free runs, batch=1:

| | FP32 | INT8 (dynamic PTQ) |
|---|---|---|
| Accuracy | 92.50% | 92.33% (-0.17pp) |
| Precision / Recall / F1 (macro) | 0.926 / 0.925 / 0.925 | 0.924 / 0.923 / 0.923 |
| Latency (mean ± std) | 0.0346 ± 0.0178 ms | 0.8748 ± 0.1191 ms |
| Model size | 0.066 MB | 0.021 MB (-68.6%) |

Two honest takeaways:

1. Quantization now costs a small real amount of accuracy (0.17pp). At the earlier, smaller test-set size it looked completely free. It isn't, quite.
2. INT8 is slower, about 25x. Dynamic quantization computes the activation scale at runtime on every call, and on a network this tiny that overhead beats whatever the int8 matmul saves.

## Latency budget and recommendation

Budget: 20ms per inference, for a real-time decision loop on Aido Rover or Sentinel Prime AI. Both configs clear it easily (FP32 by ~578x, INT8 by ~23x), so latency alone doesn't decide anything here.

Recommendation: use INT8 anyway, for size not speed. 68.6% smaller for a small accuracy cost seems like a fine trade once this classifier isn't the only model on the device. Sentinel Prime AI's own spec runs a five-model ensemble on one Jetson AGX Orin, and Aido Rover's 14-sensor stack implies something similar. Memory is the more binding constraint at this size, not latency.

If this model were meaningfully bigger, I'd expect quantization to win on both size and latency, and I'd want to re-run this benchmark rather than assume the same conclusion carries over.

## Connection back to AURA

AURA's real pipeline (`ESP32/main.py` + `model_training/server.py`) never quantizes anything and never benchmarks latency. It just retrains an sklearn `MLPClassifier` on the full dataset and calls `.predict()`. This task is basically the deployment-engineering half AURA never needed: its inference runs on the server, not the ESP32 itself, so there was never a hard edge-latency constraint. Aido Rover and Sentinel Prime AI run inference on-device, so they do have that constraint, and that's what this memo is answering.
