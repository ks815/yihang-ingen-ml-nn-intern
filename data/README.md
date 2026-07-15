# Data

Synthetic sensor data (Week 2) and small curated dialogue datasets (Week 3) only. No InGen internal or customer data.

Generated data files are gitignored beyond seeds and config (see `.gitignore`). To regenerate:

1. Week 2 synthetic sensor dataset: run the generator cell in `../week02_edge_classifier/W02_Edge_Classifier.ipynb` with the committed seed/config — reproduces IMU, motor current, and proximity sensor readings labeled with operational modes (PATROL, ALERT, CHARGING, FAULT).
2. Week 3 dialogue dataset: run the dataset construction cell in `../week03_finetune_rag/W03_LoRA_FineTune.ipynb` with the committed config.
