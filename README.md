# InGen ML/NN Analyst Internship — Yihang Sun

4-week, fully-remote Machine Learning & Neural Network Analyst internship at InGen Dynamics Inc. (Origami AI / Physical Intelligence Platform 2.0), July 6 – August 1, 2026.

## Intern

Yihang Sun — M.S. Electrical Engineering, Columbia University (Expected Dec 2026); B.A. Computer Science (Minor Economics), Rutgers University.

## Skills demonstrated

- Neural network engineering: PyTorch classifier design, training, and evaluation on held-out test splits
- Edge / embedded ML: post-training quantization (INT8), latency benchmarking against a stated real-time budget
- LLM fine-tuning: QLoRA on small open-licensed base models (peft, bitsandbytes), rank/learning-rate ablations
- Retrieval-augmented generation: FAISS-based dense retrieval, sentence-transformers embeddings, hit-rate evaluation
- Safety / evaluation discipline: labeled held-out test sets, reproducible seeds and configs, documented hyperparameters

## Deliverable index

| Wk | Artifact | Description | Due |
|----|----------|--------------|-----|
| 1 | `week01_landscape/W01_InGen_NN_Landscape.md` | Platform brief: one page per InGen product + PIC 2.0 architecture map | Fri Jul 11 |
| 1 | `week01_landscape/W01_OpenClaw_Bridge.md` | OpenClaw agent roles → PIC 2.0 foundation model bridge | Fri Jul 11 |
| 1 | `week01_landscape/W01_env_check.ipynb` | Environment check: PyTorch, transformers, peft, faiss, FastAPI + GPU check | Fri Jul 11 |
| 2 | `week02_edge_classifier/W02_Edge_Classifier.ipynb` | Lightweight quantized NN classifier on synthetic sensor data | Fri Jul 18 |
| 2 | `week02_edge_classifier/W02_Edge_Deploy_Memo.md` | On-device classifier design memo | Fri Jul 18 |
| 3 | `week03_finetune_rag/W03_LoRA_FineTune.ipynb` | QLoRA fine-tune + ablation | Fri Jul 25 |
| 3 | `week03_finetune_rag/W03_RAG_Pipeline/` | FAISS-based RAG pipeline + retrieval hit-rate eval | Fri Jul 25 |
| 3 | `week03_finetune_rag/W03_FineTune_RAG_Memo.md` | Fine-tuning + RAG design memo | Fri Jul 25 |
| 4 | `week04_capstone/W04_Capstone_Report.docx` | 12-16 page capstone report | Fri Aug 1 |
| 4 | `week04_capstone/W04_Capstone_Deck.pptx` | 12-slide executive deck | Fri Aug 1 |
| 4 | `week04_capstone/W04_Retrospective.md` | Retrospective + career narrative | Fri Aug 1 |
| 1-4 | `weekly/Wk-NN-Recap.md` | 300-500 word weekly recap | Each Fri |

## Constraint note

No access to InGen internal systems, customer data, financials, or proprietary technical documentation. All work uses public data, open-source models, and InGen's published product portfolio. Nothing in this repository is InGen-confidential.
