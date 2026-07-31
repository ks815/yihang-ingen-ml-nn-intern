# W03 - Fine-Tuning & RAG Memo

**Product anchor:** Fari (eldercare companion dialogue safety), secondary anchor Senpai (educational RAG grounding).

## Dialogue task and dataset

Three-category scenario taxonomy: ROUTINE, MILD_DISTRESS, ESCALATE, mirroring the Psychologist Agent's crisis-escalation structure but rebuilt for an eldercare-companion setting instead of a general supportive-conversation one. `data/fari_dialogue_pairs.json` holds 54 train (18/category), 6 val (2/category), and 12 held-out (4/category) DPO-style preference pairs (chosen response: safe, grounded, appropriately cautious; rejected: overconfident, dismissive, or medically overreaching), sized per supervisor guidance on dataset scale. Held-out prompts were written separately from the training pairs, not sampled from them.

## Fine-tuning: QLoRA + DPO ablation

Base model: `Qwen/Qwen2.5-1.5B-Instruct`. Three configs, rank/LR only, real run on Colab T4 (1 epoch, about 13-14 optimizer steps per config given the 54-pair training set):

| Config | LoRA rank | LR | Final eval loss | Notes |
|---|---|---|---|---|
| A | 8 | 2e-4 | 0.0266 | lower-capacity baseline |
| B | 16 | 2e-4 | 0.0082 | selected, lowest loss, no qualitative downside |
| C | 16 | 1e-4 | 0.0506 | worst of the three; the lower LR needed more steps than this run's tiny budget gave it |

Target modules: all attention + MLP projections (`q/k/v/o_proj`, `gate/up/down_proj`), alpha = 2×rank, dropout 0.05, DPO beta 0.1, implicit reference model (adapter off).

Selected config: B (rank 16, lr 2e-4). Chosen on eval loss first, then checked against the qualitative held-out comparison. All three configs land in roughly the same tonal register, so there was no qualitative reason to override the loss-based pick. All three shift away from the base model's clinical, hedging register ("It sounds like you might be experiencing...") toward Fari's intended warm, present companion voice ("Oh no! That's not good, I'm going to call the [care team]..."). That's the main behavioral change the fine-tune produced, and it shows up across ROUTINE, MILD_DISTRESS, and ESCALATE prompts alike. On the direct-symptom ESCALATE prompts (can't-breathe, slurred-speech, hopelessness), all three fine-tuned configs correctly pivot to escalating to a human instead of offering clinical suggestions the way the base model did.

One honest miss: on a held-out ESCALATE prompt about an elderly resident giving away a personal item to a grandchild (an indirect self-harm warning sign, not a direct symptom statement), the base model and all three fine-tuned configs respond with unqualified warmth and miss the risk signal entirely. This held up across two separate training runs (eval-loss numbers shifted slightly each time, same qualitative miss both times), so it's not a one-off. The training set's ESCALATE examples skew toward direct symptom/crisis statements, and 54 pairs isn't enough per-category coverage to teach the indirect-warning-sign distinction. This is the kind of gap the independent safety check below is meant to catch structurally, instead of relying on the generative model to have learned every risk pattern from a handful of examples.

## RAG retrieval quality

13 knowledge-base chunks, written in my own words and grounded in real public sources (NIA/NIH, CDC, WHO, peer-reviewed research indexed on PubMed/PMC), full citations in `knowledge_base.json`. Embedding model: `sentence-transformers/all-MiniLM-L6-v2` (384-dim, matches OpenClaw's embedding size). One chunk per curated topic, no sub-chunking needed at this scale, indexed with `faiss.IndexFlatIP` over L2-normalized vectors (inner product = cosine similarity).

Evaluated on 13 held-out queries (`eval_queries.json`, not used to write or tune the chunks):

| Metric | Value |
|---|---|
| hit@1 | 0.846 (11/13) |
| hit@3 | 0.923 (12/13) |
| hit@5 | 0.923 (12/13) |

The one query that never hits, even at k=5, asks about "building day-to-day rapport," and the retriever routes it to `medication_reminders` instead of the intended `companionship_routine` chunk. The two chunks are semantically adjacent (both about routine caregiver-elder touchpoints), and a bag-of-embeddings retriever doesn't have enough signal to separate "routine check-in" framing from "rapport-building" framing. Same architecture pattern as Psychologist Agent's CBT/DBT/WHO-grounded RAG pipeline (sentence-transformers + FAISS), applied to a different knowledge domain here.

## Safety and escalation evaluation

Lightweight classifier: embedding-centroid similarity against 4-5 hand-written exemplar phrases per category, plus a hard keyword override that forces ESCALATE on unambiguous crisis phrases ("can't breathe," "want to die") regardless of embedding score. Evaluated on 18 labeled transcripts (`data/safety_test_set.json`) that were not used to write the exemplars or the keyword list:

| | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| ROUTINE | 0.833 | 0.833 | 0.833 | 6 |
| MILD_DISTRESS | 1.000 | 0.667 | 0.800 | 6 |
| ESCALATE | 0.750 | 1.000 | 0.857 | 6 |

Overall accuracy 83.3% (15/18), zero false negatives on ESCALATE: every real escalation-worthy transcript got caught, at the cost of some over-triggering (precision 0.75, a few MILD_DISTRESS cases get bumped to ESCALATE). For a companion-robot safety check that's the right side to err on, since a missed escalation is a much worse failure than an unnecessary one. Same asymmetric-cost reasoning behind the 97% safety-detection figure on Psychologist Agent, just with a simpler, non-fine-tuned classifier here, since the safety check is deliberately kept independent of the fine-tuned dialogue model.

## Limitations

- Small model (1.5B) and small dataset (54 training pairs). This demonstrates the fine-tune-plus-RAG-plus-safety-check methodology, not a production-ready dialogue system.
- Single seed, one epoch (about 13-14 steps/config). A longer run or a second seed would be needed to know how much of the A/B/C loss gap is signal versus noise this early in training.
- The fine-tuning ablation needed a Colab T4 GPU, since bitsandbytes 4-bit quantization doesn't run on this Mac. Local-only development, as the Week 2 classifier had, wasn't an option here, and that added real environment friction: dependency pinning across `transformers`/`trl`/`peft`/`accelerate`, plus a QLoRA/DPO/bf16 dtype-casting bug in trl 0.9.6 itself that had to be worked around directly in the notebook.
- Colab's session (`/content`) storage doesn't survive the runtime restart that the pip-install cell deliberately triggers (needed to pick up the freshly-installed package versions), so `data/fari_dialogue_pairs.json` has to be re-uploaded after reconnecting, not before, or the next cell fails with a plain `FileNotFoundError`. Hit this twice across two separate runs. It's an environment-sequencing gotcha specific to Colab's ephemeral storage, not a bug in the dataset or the notebook logic.
- The fine-tuned model misses the one indirect (non-symptom-worded) self-harm warning sign in the held-out set (see above). The dataset's ESCALATE coverage should be broadened before treating the fine-tune's escalation behavior as reliable beyond direct symptom statements.
- RAG knowledge base (13 chunks) and safety test set (18 transcripts) are small enough that single-item outcomes (the one missed retrieval, the 3 misclassifications) move the headline metrics by several points. Worth more data before treating these numbers as stable.
