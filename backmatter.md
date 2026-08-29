# Back Matter

## Glossary

- acceptance rate: fraction of draft tokens kept under speculative decoding
- aggregate throughput: total tokens/second across concurrent streams
- artifact: a concrete weight file (e.g. a GGUF) with identity and checksum
- bandwidth-bound: regime where memory movement, not FLOPS, limits decode
- black start: recovery from full process or power loss
- cache reuse: serving feature that reuses prefix KV across requests
- compute-buffer OOM: allocation failure for workspace, not only weights
- concurrency (c): number of simultaneous in-flight generations
- control: measurement that isolates the variable not under study
- decode: per-token generation after prefill
- degraded mode: intentional reduced recipe under heat or incidents
- DNF: did not finish / not feasible under stated method
- edge: hardware you own near the work; not a synonym for MCU
- expert precision: bit-width policy for MoE expert bodies
- fit: load + headroom + soak under a named recipe
- GGUF: common local weight container used with llama.cpp
- headroom: free VRAM/RAM left after a successful load under target traffic
- host OOM: host memory exhaustion, often mislabeled as a GPU fault
- indexer: engine component whose CPU vs GPU placement moved §A tok/s
- KV cache: per-context key/value working set for attention history
- last-known-good: pinned recipe that last passed smoke and suite
- llama.cpp: open inference engine used for most lab rows in this book
- load log: startup/placement output treated as ground truth
- mmap: memory-map weights from disk; interacts with host RAM budgets
- MoE: mixture of experts; sparse activation, dense packaging problems
- MTP: multi-token prediction / multi-token draft head speculation
- n_ctx_slot: per-slot context after parallel divides the budget
- n_max: maximum draft tokens proposed per speculative step
- n-cpu-moe: offload count for MoE layers onto CPU paths
- placement: device map of tensors/layers/indexers
- prefill: prompt ingestion path that builds initial KV
- promotion packet: quality+speed+fit evidence bundle for a recipe change
- recipe: full binary+artifact+flags bundle that produces a number
- reference box: the lab's 4× RTX PRO 4500 / 128 GB VRAM measurement host
- repack: weight repacking path; can segfault some artifacts if wrong
- residence: weights kept on the intended fast device path
- roofline: hardware model relating intensity to bandwidth ceilings
- RTO: recovery time objective after hard failure
- sideways requant: precision rewrite that does not buy shrink/residence
- soak: sustained load test watching drift, heat, errors
- speculation: draft-and-verify decoding to raise accepted tokens per heavy step
- spill: layers/experts living off the fast path
- suite: versioned evaluation set tied to a product decision
- tensor split: multi-GPU partition vector
- tok/s: tokens per second; always name which speed kind
- tombstone: retained failed recipe or aborted experiment note
- tool hardmode: lab tool-use suite referenced in §C/§D
- vLLM: alternate serving engine appearing in some reference rows
- warm decode: steady-state generation after load, not first-token cold start

## References

- RogerAI Labs RESULTS-MATRIX.md — Complete test matrix, DeepSeek-V4-Flash project and later sections (§A–§G and notes)
- RogerAI Labs PROJECT-LOG.md — 2026-08-22/24 power crash recoveries; K3-Encode cache-reuse notes; hybrid KV arms
- llama.cpp project: https://github.com/ggml-org/llama.cpp
- llama.cpp build docs: https://github.com/ggml-org/llama.cpp/blob/master/docs/build.md
