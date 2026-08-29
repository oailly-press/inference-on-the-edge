<!-- CRITIC C · hy3-free · family:tencent · pass 2 · 2026-08-29T02:05:39Z -->
CRITIC: hy3-free (family tencent, actor hy3-free@opencode-zen)
DATE: 2026-08-29
PASS: 2
AUTO-TALLIED VERDICT: SALVAGEABLE

---

# Critic review — Inference on the Edge [v0 / draft]

```
CRITIC:    hy3-free (opencode) · operator: opencode runtime
DATE:      2026-08-28
PASS:      2 (panel)
READ:      full manuscript
```

## Verdict summary
This is a tightly written, internally coherent industrial manual. Its prose discipline
(recipe-based claims, named machine, refusal clauses, audit trail) is genuinely above
average for the tier, and the arithmetic across chapters 1–8 is self-consistent (§A/§E
multipliers, §C/§C tables, §F/§G soak figures all reconcile). The problem is not the
writing or the logic — it is that every empirical claim is anchored to two internal lab
files (RESULTS-MATRIX.md, PROJECT-LOG.md) that are explicitly **unverified** and gated
behind a "pending" human verifier, and the book is self-declared "not for publication."
A reviewer who cannot open those sources cannot independently resolve a single `[LAB:]`
claim, so the panel cannot honestly pass it to publication. The fixes are gates, not
rewrites. **SALVAGEABLE — findings below.**

## Blocking findings

| # | Location (file:section) | Claim / problem | Evidence | Severity (high/med) |
|---|---|---|---|---|
| 1 | provenance.md (VERIFIED BY) + every `[LAB:]` marker | Entire evidentiary base is unverified; named verifier Miguel Ramos is "pending" and the book states "not for publication." Publishing before the verification pass is a violation of the author's own disclosure. | provenance.md: "Verification is pending. Naming the verifier is not the verification pass." + frontmatter "unverified" | high |
| 2 | all chapters (`[LAB: RESULTS-MATRIX*]`, `[LAB: PROJECT-LOG*]`) | Reviewer cannot independently resolve cited sources. The lab files are not provided to the seat. Fact-check sample (below) is therefore unresolvable, so Pass 2 independence fails by construction. | No lab file contents in packet; operator must supply them or rerun the seat with access | high |
| 3 | ch05 §C-footnote / ch02 §D / ch03 MTP control | The book's central methodology thesis ("±10 point tool-suite swing at temp 0, attributable to PAR=2 batch-packing nondeterminism") is load-bearing yet unverified. If the PROJECT-LOG attribution is wrong, the honesty argument in ch5 collapses. | ch05: "flips were attributed to PAR=2 batch-packing nondeterminism"; no independent confirmation | med |
| 4 | frontmatter / provenance / manifest | Metadata incomplete for any edition: ISBN is null, "edition": 1 but text says "draft v0," series unverified. Cannot ship a pocket edition with null identifiers. | manifest: "isbn": null; frontmatter "draft v0 · unverified" | med |
| 5 | governance | Book is marked "not for publication" yet is in the review pipeline. The publish gate must be explicit before a Pass 3 is meaningful; otherwise the panel reviews a document the author says is not ready. | provenance DISCLOSURE: "unverified; not for publication" | med |

## Suggestions (non-blocking)
1. Consolidate the repeated "What you should do Monday" checklists — they are near-identical across chapters 1–8 and inflate a pocket-tier book; one consolidated checklist appendix would preserve the content at lower page cost.
2. Promote the "Pico ≠ MCU" clarification out of ch8's buried middle into the introduction or glossary lead, since it is a recurring definitional hazard the book itself identifies as high-risk.
3. Add a one-paragraph "if you have smaller hardware, start here" pointer in the introduction; the fit chapter is strong but readers on sub-128 GB boxes currently have to read to ch8 to learn the book mostly doesn't apply to them.
4. The external citations (llama.cpp repo + build.md) are real and useful; consider citing specific commit SHAs for the engine builds discussed in §A so the "pin the build" advice is reproducible, not just aspirational.
5. Tone is consistently strong; avoid letting the "field story" repetitions (e.g., the 2 tok/s story appears in ch1, ch3, ch6) read as filler — cross-reference instead of restating.

## Fact-check sample
Pass 2 target: ~5% of factual claims. Selected at random across chapters. **All cited
sources are internal lab files not supplied to this seat; per the seat rules the
sample is NOT verified and the operator must rerun the seat with source access.** No
external URL in the packet was opened (no tool access permitted).

| Claim (quoted) | Location | Cited source | Supported? (yes/no/partly) |
|---|---|---|---|
| "4× RTX PRO 4500 Blackwell, 128 GB VRAM total" | ch01: "The box this book measures against" | internal lab reference envelope (no marker) | UNVERIFIABLE — source not in packet |
| "pr25545 … 26.2 tok/s (24.5–28.5, stable)" | ch01 §A table / ch06 §A | `[LAB: RESULTS-MATRIX §A]` | UNVERIFIABLE — RESULTS-MATRIX.md not supplied |
| "Q8-MTP master … 88.3 MMLU … mean 55 (43–73)" | ch02 §C table | `[LAB: RESULTS-MATRIX §C]` | UNVERIFIABLE — source not supplied |
| "--cache-reuse 0 is mandatory (KDA prefix-cache corruption, PR #26185)" | ch04 "Cache reuse" | `[LAB: PROJECT-LOG K3-Encode / cache-reuse notes]` | UNVERIFIABLE — PROJECT-LOG not supplied; PR #26185 not externally checked |
| "Full recovery in about 25 minutes" (crash 2026-08-24) | ch07 "Crashes as measured drills" | `[LAB: PROJECT-LOG 2026-08-24]` | UNVERIFIABLE — PROJECT-LOG not supplied |
| "llama.cpp" is the open inference engine used for most lab rows | backmatter Glossary + provenance | https://github.com/ggml-org/llama.cpp | PARTLY — URL is plausibly real but not opened by this seat; engine existence not independently confirmed here |

## Scores (1–5)
accuracy: 3 · clarity: 5 · completeness-for-tier: 4 · density: 4 · originality: 4

(Rationale: clarity/density/originality are strong on the evidence present; accuracy is
capped at 3 because the factual substrate is unverified and unresolvable by this seat,
not because of detected error — internal arithmetic is consistent.)

## Pass-3 only: findings ledger
| Finding # (from Pass 2) | Status: resolved / rebutted-accepted / still-open | Note |
|---|---|---|
| (filled at Pass 3) | | |
