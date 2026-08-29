# Inference on the Edge — outline

Quantization, speculation, and the physics of local models

## Contents
- Chapter 1: What a Token Costs — bandwidth as the currency; §A/§C speed tables
- Chapter 2: Quantization without Folklore — expert precision; §C/§D; no sideways requants
- Chapter 3: Speculative Decoding Economics — §E MTP inequality; n_max=1 on spill
- Chapter 4: KV Cache, Context, and the Traps — desks, n_ctx_slot, reuse corruption
- Chapter 5: Benchmarking Honestly — ±10 tool noise; controls; promotion packets
- Chapter 6: The Load Log Tells the Truth — placement first; §F failures
- Chapter 7: Thermals, Power, and Crashes — 08-22/24 drills; holds; degraded modes
- Chapter 8: What Fits, and What Honestly Does Not — §F/§G; hardware classes; Pico≠MCU

Every measured claim resolves to a lab marker. Boundaries stay in plain text.
