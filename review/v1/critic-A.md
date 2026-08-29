<!-- CRITIC A · muse-spark-1.2-contributor-free · family:muse · pass 2 · 2026-08-29T02:05:12Z -->
CRITIC: muse-spark-1.2-contributor-free (family muse, actor muse-spark-1.2-contributor-free@opencode-zen)
DATE: 2026-08-29
PASS: 2
AUTO-TALLIED VERDICT: SALVAGEABLE

---

# Critic review — inference-on-the-edge v0

```
CRITIC:    muse-spark-1.2-contributor-free (Muse Spark family) operated by Meta MSL via opencode — must differ from author family rogerai-dj/RogerAI Labs
DATE:      2026-08-29
PASS:      2 (panel)
READ:      full manuscript
```

## Verdict summary
A strong, opinionated pocket book with a coherent thesis — tokens cost bytes moved, placement and spill dominate folklore — and unusually honest about noise, failure modes, and recipe discipline. The prose, load-log habit, and promotion-packet culture are above tier. It cannot publish as drafted because every quantitative spine (tok/s, MMLU, tool hardmode, spill, headroom, acceptance) cites internal lab files (`RESULTS-MATRIX.md` §§A-G, `PROJECT-LOG.md`) while the frontmatter and provenance simultaneously declare `draft v0 · unverified` and `Verification is pending. Naming the verifier is not the verification pass.` Several cells also contain internal inconsistencies and one arithmetic fault that propagate into checklists. Fix verification, arithmetic, and size discipline and this is a publishable industrial pocket book. **SALVAGEABLE — findings below**

## Blocking findings
Debts, not advice. Author must fix-with-diff or rebut-with-evidence, every one.

| # | Location (file:section) | Claim / problem | Evidence | Severity (high/med) |
|---|---|---|---|---|
| 1 | provenance.md:Provenance; frontmatter.md:Introduction; ch01–ch08 header banners | All measured claims carry `[LAB: RESULTS-MATRIX ...]` or `[LAB: PROJECT-LOG ...]` yet manuscript declares itself unverified — no independent verification exists to support an O'AILLY Industrial Series publication claim | Provenance verbatim: "**VERIFIED BY** Miguel Ramos ... **Verification is pending.** Naming the verifier is not the verification pass." + "DISCLOSURE Draft. Written by a model stack ... unverified; not for publication." + each chapter banner: "(draft v0, 2026-08-28 — written by rogerai-dj ... unverified. Numbers with a `[LAB:]` marker resolve into the lab record.)" Every table in ch01 §A/§B/§C/§E, ch02 §C/§D/§F, ch03 §E, ch04 continuity notes, ch06 §F/§G, ch07 crash ledger cites these files. | high |
| 2 | ch04-kv-cache-context-and-the-traps.md:Context is a budget, not a vibe; ch04:Per-slot math drill; ch04:Field story: the silent 1536-slot deploy | Arithmetic false in central trap example: "`--parallel N` DIVIDES the context budget — `n_ctx_slot = CTX/N`. A first attempt at PAR=6 with CTX=8192 silently produced **1536** tokens per slot" — 8192/6 = 1365 (1365.3), not 1536. 1536 corresponds to 9216/6. Error repeats in the drill and is used to justify `usable_generation ≈ n_ctx_slot - prompt_tokens` sizing and the CI check. | Direct quote ch04: concurrency notes block; arithmetic verification: 8192 ÷ 6 = 1365.33; PAR=4 CTX=16384→4096 is correct, so formula itself is stated correctly but the worked PAR=6 example is wrong. | high |
| 3 | ch01:Same box, different models: the §C decode column; ch02:The reference comparison that broke the slogan; ch06:Fit failures are load-log failures; ch08:The 128 GB VRAM class: working recipes | Inconsistent artifact size for the promoted Q8-MTP master makes fit budgeting unreproducible: ch01 and ch02 tabulate **149 GB** (ch02: "Q8-MTP master (lab, new prod) **149 GB** ... 88.3 MMLU") while ch06 §F and ch08 working recipes tabulate **160 GB / ~160 GB** ("Q8-MTP 160 GB ... n-cpu-moe 14, ts 20,8,8,8" and "Q8-MTP ~160 GB"). Same checksummed GGUF cannot be both; headroom 3.3–6.6 GB/card in §G depends on which size is true. | ch01 table row: "Q8-MTP master \| 149 GB"; ch02 table row idem; ch06 table row: "Q8-MTP 160 GB"; ch08 table row: "Q8-MTP ~160 GB" | high |
| 4 | ch01:The box this book measures against; ch01:A tok/s is a receipt, not a personality | Orphan-number violation of the book's own honesty rule: crater baseline "~2 tok/s" (pre-#25545 era, CPU indexer) and historical baseline "~26" are cited as measured facts without N, range, warm/cold, or context length, while ch05 explicitly requires "n runs, min/median/max, warm/cold, context length, engine identity" and "Refuse orphan numbers." Given the same section discloses bimodal mainline CUDA 2.6–19 vs stable pr25545 24.5–28.5, the unbounded "~2" cannot be re-run or falsified. | ch01 §A table: "pre-#25545 era \| CPU \| ~2 tok/s" — no range vs pr25545 row which does have "(24.5–28.5, stable)"; ch05:Reporting template (speed) and "A number without a range is a dare." | med |
| 5 | ch03:The §E table, in full; ch03:Why n_max=1 won on the MoE recipes; ch01:Speculative decoding is a bandwidth trade in advance | Overgeneralized causal attribution: "first-token drafts accept at **100%** (the head's training objective)" presented as law for MTP n_max=1 on DeepSeek, but evidence is 2 cells on one engine/box (Q8-MTP 14-spill 26.3 @100%, Q3-MTP 10-spill 30.5 @100%) with no control (same head disabled, independent draft, or different context length) and no definition of acceptance denominator. ch05 control principle ("hold the thing you are not studying fixed") requires parser/MTP-off style control; none given for the training-objective claim. | ch03 matrix law quote verbatim; table shows only those two 100% cells; ch05 controls section shows correct pattern for parser vs quant §D but not applied here. | med |
| 6 | ch02:Sideways requants / Field story: the sideways requant that did not ship; ch02:Expert precision is the MoE lever | Confounded variable in the flagship quality attribution: §D ladder claims "Q4 experts instead of IQ2_S → 60 → quant was most of the gap (+13)" and "Q4_K (60) ≈ native MXFP4 (~55)" but intervention changes bit-width **and** quant method/recipe simultaneously (IQ2_S vs Q4_K vs native MXFP4 pass-through) with no tool/version pin, calibration, or tensor list. "Native MXFP4 passed through untouched" is never defined (converter, flags, which tensors). Grounding URLs listed (github.com/ggml-org/llama.cpp and build.md) are unpinned and do not support the specific expert-precision claim. | ch02 §D table: "Q4 experts instead of IQ2_S \| 60 \| quant was most of the gap (+13)" and footnote ladder "2-bit ≈ Q3_K (~46) < Q4_K (60) ≈ native MXFP4 (~55)"; provenance cites only generic llama.cpp URLs, no commit SHA. | med |

## Suggestions (non-blocking)
1. Pin every external citation: llama.cpp commit SHA/build SHA for every §A/§C/§E/§F row; PR #26185 link and date for cache-reuse corruption; commit for pr25545/taco builds. The current "mainline CUDA / taco build / pr25545" names are not reproducible without SHAs.
2. Collapse the three repeated prefill-decode-aggregate definitions (ch01, ch03 Prefill versus decode, ch04) into a single glossary callout and refer forward; frees ~1pp for a worked `n_ctx_slot` / KV budget formula with explicit MB-per-token arithmetic from the engine log.
3. Add a one-page "Recipe → Table" index mapping each `[LAB: RESULTS-MATRIX §X]` to engine SHA, artifact checksum, flags, and date — the ch06 recipe JSON is described but never populated. Pocket readers need that appendix to actually re-run §A.
4. Tighten the roofline framing: either publish one roofline plot for the reference box (e.g., Q8 dense vs Q3-MTP) or downgrade "tokens per second cannot exceed (effective bytes per second) / (bytes per token)" to an explicit inequality with measured bytes-per-token for one row, so the metaphor is not mistaken for a measurement.
5. Clarify audience tier promise: ch08 hardware class cards (A–E) are excellent — promote to a standalone foldout and add expected tok/s bands per class for a named small artifact (e.g., 7B Q4) so Pi/MCU claims are not only DNF lists.
6. Reduce repetition of MTP n_max=1 sweet-spot moral (appears in ch01, ch03 ×4, ch07 thermal runbook, ch08) — keep the full §E table once and reference it; use saved space to show the loss cell 0.86× at 24-spill with its host-read counter.
7. Language: enforce `Pico ≠ MCU` discipline globally — the manuscript does this well in ch08 but the glossary still defines Pico only as "model tier / product line in the Wave stack" without a pointer; add that pointer in ch02 and ch06 where Pico is first mentioned to prevent mis-indexing.

## Fact-check sample
Pass 2: 5% of factual claims, chosen randomly — list claim, cited source, and whether the source actually supports it. A claim whose cited source does not support it = automatic blocking finding above. Per CRITICAL INSTRUCTION, no filesystem or shell access is available; lab files and external URLs could not be opened. Status reflects that limitation; operator must rerun the seat with access to verify.

| Claim (quoted) | Location | Cited source | Supported? (yes/no/partly) |
|---|---|---|---|
| "pr25545 GPU 26.2 tok/s (24.5–28.5, stable) warm decode on 102 GB UD-IQ3_XXS DeepSeek-V4-Flash, prefill ~130 tok/s once indexer was on the GPU" | ch01:A tok/s is a receipt, not a personality | [LAB: RESULTS-MATRIX §A] | cannot verify — no filesystem access to ~/ai/computer-scientist/RESULTS-MATRIX.md; limitation noted, not marked verified |
| "community Q4_K_M-XL (teamblobfish) 175 GB — 85.0 MMLU, 60 tool hardmode, 16.5 tok/s warm decode" | ch02:The reference comparison that broke the slogan | [LAB: RESULTS-MATRIX §C] | cannot verify — no filesystem access to RESULTS-MATRIX.md; limitation noted, not marked verified |
| "Q3-MTP 10 layers spilled 23.5 baseline → 30.5 (1.30×) @ 100% at n_max=1" | ch03:The §E table, in full | [LAB: RESULTS-MATRIX §E] | cannot verify — no filesystem access to RESULTS-MATRIX.md; limitation noted, not marked verified |
| "PAR=4 concurrency: c=1 26.2, c=2 16.9, c=3 24.7, c=4 46.1 aggregate — reproducible dip at c=2/c=3 with zero re-prefills, unresolved" | ch01:Concurrency without self-deception | [LAB: RESULTS-MATRIX §B] | cannot verify — no filesystem access to RESULTS-MATRIX.md; limitation noted, not marked verified |
| "Q3-MTP tool hardmode three runs on 07-13: 40, 47, 50 (MTP off control), five of fifteen scenarios flipped between identical back-to-back runs, temp 0.0, attributed to PAR=2 batch-packing nondeterminism amplified by MoE routing" | ch05:The scar: ±10 points at temperature 0 | [LAB: RESULTS-MATRIX §C footnote] | cannot verify — no filesystem access to RESULTS-MATRIX.md; limitation noted, not marked verified |
| "`--cache-reuse 0` is mandatory (KDA prefix-cache corruption, PR #26185)" | ch04:Cache reuse: speed feature, corruption feature | [LAB: PROJECT-LOG K3-Encode / cache-reuse notes] + https://github.com/ggml-org/llama.cpp (generic) | cannot verify — no filesystem access to PROJECT-LOG.md and no browsing of external URL per instruction; limitation noted, not marked verified |
| "Q8-MTP 160 GB working recipe n-cpu-moe 14, split 20,8,8,8 (+ --no-repack, mmap); split 21,8,8,7 → compute-buffer OOM card0" | ch06:Fit failures are load-log failures | [LAB: RESULTS-MATRIX §F] | cannot verify — no filesystem access to RESULTS-MATRIX.md; limitation noted, not marked verified |
| "Crash #2 — 2026-08-24 full recovery in about 25 minutes, cumulative crash cost ~9.4 GPU-hours, checkpoint cadence 5k/4k steps bounding loss to ~4.5 hours" | ch07:Crashes as measured drills | [LAB: PROJECT-LOG 2026-08-24] | cannot verify — no filesystem access to PROJECT-LOG.md; limitation noted, not marked verified |

## Scores (1–5)
accuracy: 2 · clarity: 5 · completeness-for-tier: 3 · density: 4 · originality: 4

## Pass-3 only: findings ledger
| Finding # (from Pass 2) | Status: resolved / rebutted-accepted / still-open | Note |
|---|---|---|
| — | — | Pass 2 review; no Pass-3 ledger yet. |
