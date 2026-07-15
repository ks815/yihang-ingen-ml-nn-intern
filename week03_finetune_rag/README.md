# Week 3 — Fine-Tuning & RAG for Companion Dialogue Safety

Jul 21–25, 2026. Objective: apply QLoRA fine-tuning and a FAISS-based RAG pipeline to an InGen companion-robot dialogue safety task, extending the Psychologist AI Agent methodology to a new domain.

## Deliverables

- [ ] `W03_LoRA_FineTune.ipynb` — QLoRA fine-tuning notebook: dataset construction, training loop, rank/learning-rate ablation, loss curves
- [ ] `W03_RAG_Pipeline/` — embedding generation, FAISS indexing, retrieval evaluation script, top-k hit-rate report
- [ ] `W03_FineTune_RAG_Memo.md` — 2-page memo: ablation summary, retrieval quality, safety evaluation

## Self-check

- Does the QLoRA ablation report training loss per configuration plus a qualitative response comparison, not just a final number?
- Does the RAG retrieval evaluation use a held-out query set, distinct from any queries used to tune retrieval?
- Does the safety evaluation use a labeled test set not used to design the safety-check logic itself?
