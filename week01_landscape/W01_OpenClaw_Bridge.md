# W01: OpenClaw to PIC 2.0 Bridge

## Scope

This document is my own working hypothesis based on InGen's Concepts Primer and public product information. It is not a confirmed description of InGen's internal architecture. Where a term is unclear, I keep the mapping tentative instead of forcing a direct match.

| PIC 2.0 model | Working interpretation from the Concepts Primer | Closest OpenClaw component | Why the comparison is reasonable |
|---|---|---|---|
| **HTD-IRL** | Hierarchical task decomposition plus inverse reinforcement learning | **Planner agent** | Both divide a high-level goal into smaller tasks and decide what should happen next. The mapping is only partial because OpenClaw uses routing based on rules and does not learn a reward function from demonstrations. |
| **SEOM** | Semantic or embedding-oriented model | **Knowledge / retrieval agent** | Both represent information by meaning and use retrieval to ground later analysis. OpenClaw uses embeddings and Chroma to find relevant SEC filing passages before generating an answer. |
| **STUM** | State or sequence modelling | **Quant / analysis agent** | Both process changing or sequential inputs and turn them into a structured decision signal. OpenClaw's Quant agent works with market data over time, although it is not a confirmed equivalent of STUM. |
| **AMDC** | Adaptive or hierarchical decision making and governance | **Risk and Critic agents** | The Risk agent measures financial exposure, while the Critic checks evidence, freshness, overstatement, and action boundaries. Together they form a governance layer before the final output is accepted. |
| **CRL-MRS** | Continual or coordination across multiple robots | **Report agent**, at a high level | Both combine outputs from several upstream components into one result. However, the Report agent performs document synthesis, not coordination in real time among physical robots. |
| **GRPO** | One of the six PIC 2.0 foundation models, but not defined in detail in the Primer | **No confident mapping** | The public material is not clear enough for me to connect GRPO to one OpenClaw agent without making a stronger claim than the evidence supports. |

## Main observations

The clearest connection for me is between **HTD-IRL and the Planner agent**. OpenClaw's Planner receives a request, identifies the intent, and decides which agents need to run. This is similar to hierarchical task decomposition because one broad goal is converted into smaller steps. The limitation is that OpenClaw does not use inverse reinforcement learning, so the comparison is about decomposition rather than the full HTD-IRL method.

The connection between **SEOM and the Knowledge agent** also makes sense. The Concepts Primer describes SEOM as likely semantic or embedding-oriented. OpenClaw's Knowledge agent uses embeddings and vector search to retrieve relevant evidence from SEC filings. Both therefore support downstream decisions with semantically related information instead of relying only on a model's internal memory.

The comparison between **STUM and the Quant agent** is broader. STUM is described as state or sequence modelling, while the Quant agent processes time-dependent market and financial data. Both convert a changing stream of observations into a structured input for later decisions, although the internal models may be very different.

For **AMDC**, the closest OpenClaw match is the combination of Risk and Critic. The Risk agent evaluates financial uncertainty, and the Critic applies a final set of governance checks. This resembles adaptive decision-making because the system evaluates whether an output is acceptable before it is released.

The connection between **CRL-MRS and the Report agent** is weaker. The Report agent combines several software outputs into one document, while CRL-MRS is likely concerned with ongoing coordination among multiple physical agents. The shared pattern is synthesis, but the physical coordination problem is much more demanding.

I do not make a firm mapping for **GRPO** because the Concepts Primer lists it without giving a clear published research interpretation. I would rather leave it unresolved than force a weak match with the Quant agent.

## Conclusion

For me, the main value of this bridge is seeing which design patterns transfer. OpenClaw already demonstrates task decomposition, semantic retrieval, analysis of changing data over time, governance checks, and combining results from several components. However, it does not prove that I have built the same models as PIC 2.0, and it does not yet cover inverse reinforcement learning or real-time coordination across multiple robots. This makes it clearer which parts of my previous work transfer and which parts are still new to me.
