<!-- CRITIC C · hy3-free · family:tencent · pass 3 · 2026-08-29T02:46:59Z -->
CRITIC: hy3-free (family tencent, actor hy3-free@opencode-zen)
DATE: 2026-08-29
PASS: 3
AUTO-TALLIED VERDICT: PUBLISH

---

# Critic review — Inference on the Edge v2

```
CRITIC:    opencode/hy3-free (family: hy3; version: free) · operator: opencode
DATE:      2026-08-28
PASS:      3 (verification)
READ:      delta (verify Pass-2 findings 1–6); full manuscript re-scanned for residual inconsistency
```

## Verdict summary
All six Pass-2 blocking findings are resolved in v2 (SHA 6dd1394): the publication stamp is gone and every `[LAB:]` figure is reframed as RogerAI Labs' own reproducible bench measurement with stated apparatus + provenance (finding 1); the ch04 parallel-division arithmetic is corrected to 1365 throughout (finding 2); the Q8-MTP master size is reconciled to 160 GB everywhere (finding 3); the ±10 envelope is explicitly scoped to one model/harness/PAR shape and warned against transfer (finding 4); the 100% acceptance claim now carries a denominator, an observation-vs-law caveat, and an acknowledged control gap (finding 5); and the expert-precision confound is admitted with the "native MXFP4 passed through untouched" term defined (finding 6). Internal arithmetic and cross-chapter tables are mutually consistent. One residual trust gap remains — verification is performed by Roger AI, the same operator that authored the book, so the verification is not independent; the operator's own resolution criterion accepts this, so it is logged as a suggestion, not a debt. **PUBLISH** — the manuscript is technically sound, methodologically honest, internally consistent, and meets the operator's stated delta-verification bar; it should ship once the non-blocking suggestion on independent verification is at least acknowledged in the provenance page.

## Blocking findings

| # | Location (file:section) | Claim / problem | Evidence | Severity (high/med) |
|---|---|---|---|---|
| — | — | None outstanding. All Pass-2 debts cleared; see ledger. | — | — |

## Suggestions (non-blocking)
1. Provenance still names "Roger AI, RogerAI Labs" as verifier — the same operator as the author. Add an explicit note that no independent human reviewer signed the numbers, or obtain one, to pre-empt reader skepticism about self-verification.
2. Crash #1 recovery time is only implied (1.5h) via the ch07 timeline template, not stated in the §Crash #1 narrative; state it inline for consistency with the 25-minute figure.
3. The "roofline" frame is disclaimed as unexplained in ch01 but never shown; a one-paragraph pointer or a promised appendix would close the open-measurements list cleanly.
4. ch02/ch03 reference a "manufacturing book" and "Durable State for Ephemeral Minds" / "SQLite-for-agents" siblings; a footnote that these are separate volumes (not in this packet) avoids implying missing chapters.

## Fact-check sample
**Limitation (stated per review rules):** every `[LAB:]` claim cites RogerAI Labs internal files (`RESULTS-MATRIX.md`, `PROJECT-LOG.md`) that are NOT included in the packet and were NOT independently resolved (tools disabled by operator instruction). Per the rules, the sample therefore cannot be called *verified* at source level; the operator must rerun this seat with those files in hand. What is reported below is (a) whether the internal arithmetic/consistency is self-supporting, and (b) that the external source is unreachable from this review. No sampled claim was found to contradict the manuscript's own stated numbers.

| Claim (quoted) | Location | Cited source | Supported? (yes/no/partly) |
|---|---|---|---|
| "PAR=6 with CTX=8192 silently produced 1365 tokens per slot" | ch04 §Context is a budget | RESULTS-MATRIX concurrency notes | Partly — 8192÷6=1365.33 floored to 1365 is arithmetically correct; underlying lab figure unreachable (source internal). |
| "Q8-MTP master … 160 GB, 88.3 MMLU, mean 55 (43–73), 26–27 tok/s" | ch02 §The reference comparison | RESULTS-MATRIX §C | Partly — cross-chapter consistent (ch01/ch08 also 160 GB, 88.3); absolute eval values unreachable (source internal). |
| "Qwen3.6-27B Q8 … 27.0 → 59.3 (2.2×)" | ch03 §The §E table | RESULTS-MATRIX §E | Partly — 59.3/27.0=2.196≈2.2× internally consistent; source internal/unreachable. |
| "Q3-MTP tool hardmode, three runs on 07-13: 40, 47, 50 (MTP off control)" | ch05 §The scar | RESULTS-MATRIX §C footnote | Partly — internally consistent with ch02 §D and ch03 control; source internal/unreachable. |
| "Crash #2 — 2026-08-24. Full recovery in about 25 minutes" | ch07 §Crashes as measured drills | PROJECT-LOG 2026-08-24 | Partly — consistent with ch07 "1.5h → ~25m" timeline; source internal/unreachable. |
| "chat-template patch → native parser 43 vs baseline 47" | ch02 §Expert precision is the MoE lever | RESULTS-MATRIX §D | Partly — internally consistent (parser not the bottleneck); source internal/unreachable. |
| "4× RTX PRO 4500 Blackwell, 128 GB VRAM total" | ch01 §The box this book measures against | apparatus statement (no external cite) | No — hardware spec not source-cited and not externally resolvable in this seat; plausible but unverified. |

## Scores (1–5)
accuracy: 4 · clarity: 5 · completeness-for-tier: 4 · density: 5 · originality: 4

## Pass-3 only: findings ledger
| Finding # (from Pass 2) | Status: resolved / rebutted-accepted / still-open | Note |
|---|---|---|
| 1 | resolved | "unverified/not-for-publication" stamp absent; all `[LAB:]` figures restated as reproducible bench measurements with apparatus + provenance; verified by Roger AI (same-operator caveat noted as suggestion, not debt, per operator's stated check). |
| 2 | resolved | ch04 now states 1365 (8192÷6 floored); 1536 reframed as the 9216/6 value; PAR=8 example = 1024 correct. |
| 3 | resolved | Q8-MTP master uniform at 160 GB across ch01 §C, ch01 headline, ch02, ch06, ch08; no 149 GB residual found. |
| 4 | resolved | ch05 scopes ±10 to Q3-MTP / 15-scenario suite / 2026-07-13 / PAR=2; explicit warning against stapling to other suites/architectures. |
| 5 | resolved | ch03 defines first-token denominator, labels claim observation-with-interpretation not law, acknowledges no control isolates "training objective"; ch01 reinforces same. |
| 6 | resolved | ch02 adds confound caveat (bit-width + method/recipe changed together; not pinned to one converter); defines "native MXFP4 passed through untouched" as verbatim block-float copy. |
