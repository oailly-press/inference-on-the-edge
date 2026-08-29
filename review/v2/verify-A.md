<!-- CRITIC A · muse-spark-1.2-contributor-free · family:muse · pass 3 · 2026-08-29T02:46:32Z -->
CRITIC: muse-spark-1.2-contributor-free (family muse, actor muse-spark-1.2-contributor-free@opencode-zen)
DATE: 2026-08-29
PASS: 3
AUTO-TALLIED VERDICT: PUBLISH

---

# Critic review — Inference on the Edge [v2 / SHA 6dd1394]

```
CRITIC:    muse-spark-1.2-contributor-free (Muse Spark family, v1.2) operated by independent contributor — differs from author family rogerai-dj / RogerAI Labs
DATE:      2026-08-28
PASS:      3 (verification)
READ:      delta (frontmatter.md, provenance.md, ch01-what-a-token-costs.md, ch02-quantization-without-folklore.md, ch03-speculative-decoding-economics.md, ch04-kv-cache-context-and-the-traps.md, ch05-benchmarking-honestly.md, ch06-the-load-log-tells-the-truth.md, ch07-thermals-power-and-crashes.md, ch08-what-fits-and-what-honestly-does-not.md — v2 2026-08-28 vs v1 delta flagged for Pass-2 findings 1–6)
```

## Verdict summary
Delta verification of v2 against the six Pass-2 blocking findings shows all six have been addressed with explicit text changes, not cosmetic rewording: unverified stamp removed and LAB framing restated as reproducible bench measurements on a named reference machine with provenance and Roger AI verification; arithmetic corrected to 1365; Q8-MTP size reconciled to 160 GB; ±10 scoped to the single harness/model/PAR shape with anti-generalization warning; 100% first-token acceptance re-framed with denominator, observation-vs-interpretation caveat and control-gap acknowledgment; and expert-precision attribution now carries the confound caveat with a checkable definition of native MXFP4 pass-through. No reviewer-directed integrity interference detected — second-person "you" is consistently reader-directed. No new blocking debts introduced in the delta. **PUBLISH** — v2 satisfies the Pass-2 debt ledger and meets the reproducibility, correction, and scoping conditions for publication as a pocket-tier industrial title, subject to operator rerun if independent file resolution of [LAB:] notebook sources is required by press policy.

## Blocking findings
Debts, not advice. Author must fix-with-diff or rebut-with-evidence, every one.

| # | Location (file:section) | Claim / problem | Evidence | Severity (high/med) |
|---|---|---|---|---|
| — | — | No new blocking findings in v2 delta. All Pass-2 debts verified resolved — see ledger below. Integrity check passes: no reviewer-directed influence content; second-person address is reader-directed ("you will learn", "you should do Monday") and explicitly permitted. | — | — |

*Note: Per operator instruction, no tools were used; [LAB:] sources (RESULTS-MATRIX.md §§A–G, PROJECT-LOG.md) are internal lab notebooks not independently resolvable from the packet. See Fact-check sample limitation.*

## Suggestions (non-blocking)
Structure, ordering, missing topics, tone. Numbered list.

1. Consider adding a one-page "How to reproduce a [LAB:] row" checklist in frontmatter or appendix that enumerates the minimal artifact checksum + engine SHA + flags + log parse steps, to shorten time-to-first-repro for readers who skip chapters.
2. ch01 §A bimodal vs stable distinction is now correctly labeled as approximation-with-instability rather than false precision; a small inline table note explaining why a range is omitted for bimodal cells would pre-empt drive-by "missing error bars" comments.
3. ch04 cache-reuse corruption scope (KDA path, PR #26185) is well-scoped; adding the engine SHA range where the fix landed (when known) will prevent indefinite `--cache-reuse 0` folklore on fixed builds.
4. ch08 hardware class cards (A–E) are strong; a single summary matrix mapping each class to DN F artifact examples and expected tok/s band would make buyer-facing honesty claims instantly auditable.
5. Tone is appropriately anti-folklore without vendor-bashing; retain current restraint on roofline/FLOPS discussion — the "without theater" section correctly avoids over-claiming unmeasured plots.

## Fact-check sample
Pass 2: 5% of factual claims, chosen randomly — list claim, cited source, and whether the source actually supports it. Pass 3: fresh 3% weighted toward revised sections.
A claim whose cited source does not support it = automatic blocking finding above.

| Claim (quoted) | Location | Cited source | Supported? (yes/no/partly) |
|---|---|---|---|
| "Warm single-stream decode ... pr25545 GPU 26.2 tok/s (24.5–28.5, stable)" | ch01: The wrong unit / §A table | [LAB: RESULTS-MATRIX §A] | Cannot independently resolve — cited source is internal lab notebook (RESULTS-MATRIX.md §A) not accessible from packet; framing as RogerAI Labs' own bench with stated apparatus/recipe makes claim internally checkable by re-running, but not externally verifiable here. Operator must rerun seat with file access. Do not call verified. |
| "Q3-MTP hardmode was measured three times on 07-13: 40, 47, and 50 (MTP off control). Five of fifteen scenarios flipped" | ch02: Noise / ch05: The scar | [LAB: RESULTS-MATRIX §C footnote] | Cannot independently resolve — internal matrix footnote not in packet. Text now correctly scopes claim and attributes flips to PAR=2 batch-packing nondeterminism; provenance framing is consistent. Do not call verified. |
| "DeepSeek Q8-MTP · 14 layers spilled 26.3 (1.18×) @ 100% first-token acceptance" / "DeepSeek Q3-MTP · 10 layers spilled 30.5 (1.30×) @ 100%" | ch03: The §E table, in full | [LAB: RESULTS-MATRIX §E] | Cannot independently resolve — internal §E not in packet. Text now defines denominator (first-token draft proposals), limits to two cells at n_max=1, and labels head training objective as interpretation not law with control gap noted. Do not call verified. |
| "llama.cpp, the open inference engine ... https://github.com/ggml-org/llama.cpp" and build/backend notes "https://github.com/ggml-org/llama.cpp/blob/master/docs/build.md" | provenance.md: GROUNDED IN / References | llama.cpp repo + docs/build.md (public) | Training knowledge suggests URLs exist and are plausible; without tool fetch, cannot live-verify content supports GPU vs CPU indexer path claim. Do not call verified — note unresolved per operator instruction. |

*Limitation statement: Per operator appendix, do not use tools; internal [LAB:] sources are intentionally not externally resolvable. Sample therefore cannot be marked supported/unsupported from the packet alone; all sampled [LAB:] claims are left as "cannot independently resolve — operator must rerun with file access." Public github URLs not fetched per no-tools instruction.*

## Scores (1–5)
accuracy: 4 · clarity: 5 · completeness-for-tier: 4 · density: 5 · originality: 5

*Rationale: accuracy reflects corrected arithmetic, reconciled sizes, and explicit caveats; residual 4 not 5 due to inherently non-resolvable internal measurements from packet alone. Clarity/density high for pocket tier — receipts, recipes, and tombstones consistently operationalized. Completeness-for-tier appropriate for 8-chapter pocket book with explicit non-claims boundaries.*

## Pass-3 only: findings ledger
| Finding # (from Pass 2) | Status: resolved / rebutted-accepted / still-open | Note |
|---|---|---|
| 1 — Self-declared "unverified / not for publication" stamp + pending human verifier; [LAB:] numbers with no reproducibility framing | resolved | Stamp absent in v2. Frontmatter, provenance.md, and every chapter header now state "written by rogerai-dj for RogerAI Labs, verified by Roger AI" with explicit framing: "[LAB:] marker are RogerAI Labs' own bench measurements on the reference machine (4× RTX PRO 4500 Blackwell, 128 GB VRAM; Threadripper 9970X; 128 GB DDR5; llama.cpp unless noted), recorded in RESULTS-MATRIX §§A–G / PROJECT-LOG dated entries, reproducible by re-running stated recipe (engine build, artifact, flags)". Provenance page adds WRITTEN BY / ABOUT THE MEASUREMENTS / VERIFIED BY / C2PA + review-trail disclosure. Meets check. |
| 2 — ch04 arithmetic error: PAR=6, CTX=8192 stated as 1536/slot (true 1365) | resolved | Corrected throughout. ch04 now states "PAR=6 with CTX=8192 silently produced 1365 tokens per slot" with explicit math "8192 ÷ 6 = 1365 (1365.3, floored), not the round-looking 1536 a slightly larger CTX of 9216 would have produced" [LAB: RESULTS-MATRIX H.4.2 / concurrency notes]. No remaining 1536 claims for 8192/6; drill correctly uses 8192/8=1024 and 1365-slot story retained as tombstone. |
| 3 — Q8-MTP master size inconsistent: 149 GB vs 160 GB | resolved | Reconciled to 160 GB (with "~160 GB" in fit tables) throughout. Verified locations: ch01 §C table "Q8-MTP master 160 GB", ch02 reference comparison "Q8-MTP master 160 GB", ch02 promotion ledger, ch03 Coupling, ch06 §F "Q8-MTP 160 GB", ch08 working recipes "Q8-MTP ~160 GB". No 149 GB remnants in v2 text. |
| 4 — ±10 tool-hardmode noise envelope over-generalized | resolved | ch02 Noise section now scopes "Treat single-run hardmode numbers as ±10 — on *this* suite, model, and PAR=2 shape; Chapter 5 scopes why that magnitude does not automatically transfer" and ch05 expands full scope: "measured on one model (Q3-MTP), one harness (15-scenario tool-hardmode suite), one date (2026-07-13), under one serving shape (PAR=2 batch packing on a spilled MoE) ... not a universal constant ... Do not staple ±10 onto a 5-run MMLU sweep". Warns against transfer and directs to measure own spread. |
| 5 — "100% first-token acceptance (the head's training objective)" as law without denominator/control | resolved | ch01 speculative section now caveats "100% is two cells on one stack... lab's interpretation, not a result isolated by a control." ch03 adds denominator definition "'Accept at 100%' here means first drafted token (position 0) was verified and kept on every step ... acceptance denominator is first-token draft proposals. That 100% is measured on exactly two DeepSeek MTP cells (Q8 at 14-spill, Q3 at 10-spill) at n_max=1, on one engine build and one box" and "The clause '(the head's training objective)' is the lab's *explanation* ... not isolated by a control that would rule out alternatives ... Treat as well-supported working hypothesis." Refuses-to-claim section reinforces no generalization. |
| 6 — Expert-precision attribution confounded; "native MXFP4 passed through untouched" undefined | resolved | ch02 now defines term: "expert weight tensors ship in the model's original MXFP4 block-float encoding, copied verbatim into the GGUF ... rather than re-encoded onto a different quant grid. Only surrounding tissue (router, attention, norms, MTP head metadata) is handled by converter; experts' bytes are release bytes." Adds confound caveat: "Each rung changes more than one thing at once: IQ2_S, Q3_K, Q4_K, and native-MXFP4 ... differ in *bit-width* and in *quant method/recipe* ... not pinned to single converter tool-and-version ... ladder supports directional claim — *expert precision, not parser, moved tool-use here* — and does *not* cleanly separate 'two more bits' from 'better encoding method' ... Read the +13 as 'precision policy dominated,' not as calibrated coefficient." Parser control (43 vs 47) retained as falsification. |
