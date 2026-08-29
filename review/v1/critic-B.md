<!-- CRITIC B · mimo-v2.5-free · family:xiaomi · pass 2 · 2026-08-29T02:05:27Z -->
CRITIC: mimo-v2.5-free (family xiaomi, actor mimo-v2.5-free@opencode-zen)
DATE: 2026-08-29
PASS: 2
AUTO-TALLIED VERDICT: SALVAGEABLE

---

# Critic review — inference-on-the-edge v0

```
CRITIC:    opencode/mimo-v2-5-free
DATE:      2026-08-28
PASS:      2 (panel)
READ:      full manuscript
```

## Verdict summary
The manuscript builds a genuinely strong framework: bandwidth-as-budget thinking, recipe-level engineering over model-label folklore, and honest treatment of measurement noise (±10 at temp 0). The speculative decoding economics chapter and the quantization-vs-tissue argument are above average for practitioner ML writing. However, every empirical claim traces to lab files (RESULTS-MATRIX.md, PROJECT-LOG.md) and URLs that this critic cannot independently resolve. The fact-check sample returns 0 of 10 claims verified against source material — not because the claims are wrong, but because the cited sources are opaque to review. This is a structural debt that blocks publication: a book whose spine is lab evidence must make that evidence auditable by the panel. **SALVAGEABLE — findings below**

## Blocking findings

| # | Location (file:section) | Claim / problem | Evidence | Severity (high) |
|---|---|---|---|---|
| 1 | provenance.md / all chapters | All empirical claims cite RESULTS-MATRIX.md and PROJECT-LOG.md as primary sources; these are local lab files not accessible to the reviewer or reader. The GitHub URLs (ggml-org/llama.cpp) may also be unreachable for verification. No claim in the fact-check sample could be verified against its cited source. | Every `[LAB:]` marker resolves to an opaque file. A book whose authority rests on measured evidence must make that evidence auditable. | high |
| 2 | ch01:§A, ch02:§C footnote, ch05:§C footnote | The ±10-point tool-hardmode noise claim is presented as established fact ("Treat single-run hardmode numbers as ±10") but rests on three runs of one model (Q3-MTP) on one harness. The manuscript does not establish that this noise envelope generalizes to other models, quants, or suites. A reader applying ±10 to a 5-run MMLU sweep on a dense 7B would be miscalibrated. | Three runs, one model, one harness, one date (07-13). No cross-model or cross-suite replication cited. | high |
| 3 | ch07:Crash #1 and Crash #2 | Recovery timelines (crash #1 forced process changes; crash #2 full recovery in ~25 min, 9.4 GPU-hours redone) are cited to PROJECT-LOG 2026-08-22 and 2026-08-24. These are unverifiable. The claim that crash-#1 fixes held in crash #2 is a strong reliability claim with no accessible evidence. | PROJECT-LOG entries dated 2026-08-22 and 2026-08-24, opaque to reviewer. | high |
| 4 | ch04:KV precision | "q8_0 KV corrupting V4 output; f16 discipline" is stated as a hard lab lesson but cited only to the book's own outline and a general reference to RESULTS-MATRIX H bring-up flags. No specific failure mode, reproduction steps, or output corruption example is provided. This is the kind of claim that, if wrong, leads operators to avoid a safe configuration. | Outline reference + opaque RESULTS-MATRIX section. No reproduction recipe or corrupted output sample. | high |
| 5 | ch04:cache-reuse | "--cache-reuse 0 is mandatory (KDA prefix-cache corruption, PR #26185)" — the PR number is cited but the manuscript does not describe the corruption mode, affected architectures, or whether the bug is fixed upstream. An operator reading this may disable a feature that is safe on current builds. | PR #26185 cited by number; corruption mode and fix status unstated. | med |

## Suggestions (non-blocking)

1. **Repetition.** The bandwidth-as-budget thesis is restated in nearly identical language across chapters 1, 2, 3, 6, and 8. A single canonical statement with cross-references would tighten the text by ~15% without losing content.
2. **Monday checklists.** Every chapter ends with a "What you should do Monday" list. By chapter 6 these feel formulaic and many items overlap across chapters (e.g., "pin recipe identity" appears in chapters 1, 5, 6, and 8). Consider a single consolidated Monday appendix.
3. **Field stories.** Several are effective (ch01 "day the indexer moved," ch04 "silent 1536-slot deploy"). Others (ch07 "afternoon slump," ch08 "128 GB demo that was a 4 GB headroom prayer") add length without new insight. Trim or consolidate.
4. **Scope vs tier.** The manifest says "pocket" tier but the manuscript is long and covers thermals, power loss, systemd dependency graphs, and MCU boundaries. Consider whether chapter 7's crash-recovery material belongs in a separate ops companion.
5. **Density.** Chapters 1–3 are tight. Chapters 5–8 repeat methodology framing (controls, ranges, recipes) that was already established. A denser recap in chapter 5 referencing earlier definitions would reduce redundancy.
6. **Glossary overlap.** The glossary in backmatter.md repeats definitions already given inline in chapters 1 and 4. Keep one canonical location.

## Fact-check sample

| Claim (quoted) | Location | Cited source | Supported? (yes/no/partly) |
|---|---|---|---|
| "warm decode jumped to 26.2 and the range tightened to something you could plan around (24.5–28.5)" | ch01:§A | RESULTS-MATRIX §A | no — source opaque; cannot verify |
| "Qwen3.6-27B dense Q8_0 sits at 29 GB, 79.0 MMLU, 67 tool hardmode, ~27 tok/s" | ch02:§C | RESULTS-MATRIX §C | no — source opaque; cannot verify |
| "DeepSeek Q8-MTP with 14 layers spilled gets 1.18× at n_max=1 with 100% first-token acceptance" | ch03:§E | RESULTS-MATRIX §E | no — source opaque; cannot verify |
| "Q3-MTP tool hardmode, three runs on 07-13: 40, 47, and 50 (MTP off control)" | ch05:§C footnote | RESULTS-MATRIX §C footnote | no — source opaque; cannot verify |
| "--cache-reuse 0 is mandatory (KDA prefix-cache corruption, PR #26185)" | ch04:cache-reuse | PROJECT-LOG K3-Encode / PR #26185 | no — PR not accessible; corruption mode unstated |
| "A 175 GB Q4 that spills hard into host memory can lose to a 149 GB master that stays resident" | ch01:bandwidth | RESULTS-MATRIX headline before/after | no — source opaque; cannot verify |
| "Q8-MTP master's three runs (73 / 50 / 43, mean 55)" | ch02:noise | RESULTS-MATRIX §C notes | no — source opaque; cannot verify |
| "PAR=6 with CTX=8192 silently produced 1536 tokens per slot" | ch04:parallel | RESULTS-MATRIX concurrency notes | no — source opaque; cannot verify |
| "Full recovery in about 25 minutes" (crash #2) | ch07:crash | PROJECT-LOG 2026-08-24 | no — source opaque; cannot verify |
| "tokens per second cannot exceed (effective bytes per second) / (bytes per token)" | ch01:bandwidth | Unmeasured (theoretical claim) | partly — well-established bandwidth-bound principle, but the manuscript labels it unmeasured, which is honest |

## Scores (1–5)
accuracy: 4 · clarity: 5 · completeness-for-tier: 4 · density: 3 · originality: 4

The accuracy score reflects that the internal logic is tight and the framework is sound, but the fact-check floor is 0/10 verifiable against cited sources — not due to dishonesty, but due to source opacity. Clarity is the manuscript's strongest axis: the bandwidth framing, the three-speeds taxonomy, and the speculative-decoding inequality are all exceptionally well-explained. Completeness is good for pocket tier but borderline long. Density drops in chapters 5–8 due to repetition of earlier framing. Originality is high: the recipe-over-model-label philosophy and the ±10 noise governance idea are genuinely useful contributions to the local-inference practitioner literature.
