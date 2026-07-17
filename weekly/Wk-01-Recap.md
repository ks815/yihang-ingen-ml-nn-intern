# Week 1 Recap

This week, I mainly focused on learning about InGen's products, reviewing the PIC 2.0 vocabulary, and comparing it with systems I have built before. I mainly wanted to understand which InGen products were closer to my Psychologist Agent project and which were closer to AURA.

The Psychologist Agent is a local LLM system that uses DPO and QLoRA fine-tuning, RAG using FAISS, and several safety checks. Its goal is to produce responses that are helpful, grounded in reliable sources, and able to escalate when a conversation may involve risk. AURA is a different type of system. It uses sensor readings from an ESP32 setup and trains a classifier to predict one of four room modes. It focuses on sensor data and classification in real time rather than language generation.

Fari and Senpai are more closely related to the Psychologist Agent. Fari needs safe companion dialogue, health monitoring, and clear boundaries around when a caregiver should make the final decision. Senpai is especially close to the RAG side of the project because its tutoring responses need to remain grounded in curriculum content and the student's current learning state. The Concepts Primer also helped me connect these products with semantic retrieval, grounding, and escalation logic.

Sentinel Prime AI and Aido Rover are more similar to AURA. Both depend on sensor fusion and classification on edge devices. The basic pattern is similar: collect sensor readings, process them over time, and predict an event or operating state. However, Sentinel and Rover use more sensor types and must work under stricter latency and reliability constraints. This connects to the Primer's ideas of state and sequence modelling, as well as inference under edge constraints. AURA used a simpler classifier and did not include all of these features.

Aido Humanoid does not fit neatly into either group. Its main challenges involve hierarchical task planning, adaptive decision-making, and physical coordination. These areas are not directly covered by my previous projects, especially the inverse reinforcement learning and coordination across multiple robots mentioned in the Primer.

Overall, Week 1 helped me see what I can carry over from my previous projects and what I still need to learn. I already have relevant experience in safe dialogue, RAG, sensor classification, task routing, and system checks. The main new areas are physical state modelling, adaptive robot decision-making, and coordination across multiple robots.
