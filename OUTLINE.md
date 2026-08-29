# Inference on the Edge — outline stub v0 (2026-08-26)

**Series:** O'AILLY Industrial Nº 2 · **Cover:** circuit dragonfly, cyan · **Status:** PLANNED (after Nº 1)

**Thesis:** The physics of running real models on hardware you own. The systems book —
written almost directly out of `~/ai/computer-scientist/PROJECT-LOG.md`.

Candidate chapter spine (each maps to measured lab material):

1. What a token costs — memory bandwidth as the real currency.
2. Quantization without folklore — expert precision as THE MoE quality lever; never
   requantize sideways (community Q4 175GB/85.0 MMLU vs untouched master 160GB/88.3).
3. Speculative decoding economics — draft length vs spill (1.18×@14 layers → 2.2×@0);
   MTP heads; first-token acceptance ~100%.
4. KV cache, context, and the traps (q8_0 KV corrupting V4 output; f16 discipline).
5. Benchmarking honestly — error bars, ±10-pt swings on small suites, controls that
   isolate the variable.
6. The load log tells the truth — debugging pathological tok/s by reading, not tuning.
7. Thermals, power, and crashes — MaxQ, three power-loss recoveries, checkpoint cadence.
8. The prosumer frontier — what a ≈128–192 GB VRAM box lets ordinary people run.
9. Below the GPU: Pi-class and MCU-class reality (honest limits; Pico ≠ MCU).

Every chapter title above already has R-entries or PROJECT-LOG entries behind it — the
attachment pass will be enumeration, not archaeology.
