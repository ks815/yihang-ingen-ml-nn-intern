# Week 2 — Edge-Constrained Neural Network Classifier

Jul 14–18, 2026. Objective: build a lightweight, quantized NN classifier on synthetic InGen robot sensor data, extending the AURA AIoT sensor-classification methodology under an edge-latency constraint. Mid-point review Fri Jul 18.

## Deliverables

- [ ] `W02_Edge_Classifier.ipynb` — synthetic sensor dataset generator, classifier training, evaluation metrics, quantization, latency benchmark
- [ ] `W02_Edge_Deploy_Memo.md` — 2-page memo: design choices, accuracy-latency trade-off, deployment recommendation anchored to a named InGen platform
- [ ] Mid-point evaluation rubric (signed jointly with supervisor)

## Self-check

- Are accuracy/precision/recall/F1 computed on a held-out test split, exactly as done for the Titanic competition models?
- Does the latency benchmark report mean and standard deviation over at least 100 warmup-free runs?
- Does the deploy memo state a specific numeric latency budget and explain why the recommended config meets it?
