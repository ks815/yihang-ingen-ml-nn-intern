# Week 2 Recap

Week 2 was the edge classifier: synthetic Aido Rover sensor data, a small feedforward net, held-out eval, INT8 quantization, CPU latency benchmark. Kept comparing this to the 4-bit GGUF call I made on Psychologist Agent, since on paper it's the same kind of decision but played out almost the opposite way.

Psychologist Agent's Llama-3.1-8B is about 16GB in fp16, way too big for an 8GB GPU. Quantizing to Q4_K_M (4.6GB, ~43.5 tok/s) wasn't really a choice, it's what made local deployment work at all.

This week was the opposite situation. The fp32 classifier is tiny, 17K parameters, 0.066MB, and it was already running in 0.035ms, something like 578x under the 20ms budget before I even touched quantization. INT8 cost a small real accuracy hit this time (92.50% down to 92.33%) and also came out slower, not faster, because dynamic quantization's per-call overhead beats whatever a network this small saves on the matmul. The only real win was file size, 68.6% smaller, and that only matters because Sentinel and Rover apparently run several models on one chip at once.

Also worth noting: the dataset itself changed a lot between drafts. Scaling from 300 to 1,000 samples per class (per the supervisor's guidance) pushed accuracy from 87.22% up to 92.50%, and FAULT recall specifically jumped from 0.717 to 0.900. That class is defined by irregularity rather than a clean steady state, so it makes sense it needed the most extra data to actually learn.

So the takeaway isn't "quantization helps." It's that whether it helps depends entirely on what's actually the bottleneck. For Psychologist Agent, memory and speed were both genuinely the bottleneck. For this classifier, nothing was, so there wasn't much for quantization to fix, just a smaller file that happens to matter once other models are sharing the same chip. I probably would've assumed it made things faster if I hadn't actually timed it, which is the part of this week I'd want to remember going into Week 3.
