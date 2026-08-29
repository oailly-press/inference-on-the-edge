<!-- CRITIC B · mimo-v2.5-free · family:xiaomi · pass 3 · 2026-08-29T02:46:45Z -->
CRITIC: mimo-v2.5-free (family xiaomi, actor mimo-v2.5-free@opencode-zen)
DATE: 2026-08-29
PASS: 3
AUTO-TALLIED VERDICT: PUBLISH

---

# Critic review — Inference on the Edge v2

```
CRITIC:    opencode/mimo-v2.5-free
DATE:      2026-08-28
PASS:      3 (verification)
READ:      full manuscript
```

## Verdict summary
All six Pass-2 blocking findings have been resolved in v2. The manuscript demonstrates rigorous self-correction: arithmetic errors fixed, size inconsistencies reconciled, over-generalizations scoped, interpretive claims properly hedged with defined denominators and acknowledged control gaps, confounds explicitly flagged, and the verification/provenance framing clarified throughout. The manuscript is ready.

**PUBLISH**

## Blocking findings
None.

| # | Location (file:section) | Claim / problem | Evidence | Severity (high/med) |
|---|---|---|---|---|
| — | — | — | — | — |

## Suggestions (non-blocking)
1. The glossary definition of "acceptance rate" uses "draft tokens kept" which is slightly ambiguous — it could mean tokens accepted or tokens that survived filtering. Consider: "fraction of proposed draft tokens verified and retained by the target model."

2. ch03's "Worked reading" section uses the phrase "0.97×-ish territory" — informal for a technical text. Consider "approximately 0.97×" for consistency with the book's precision ethos.

3. ch06's "five-minute load audit" is excellent but could reference the recipe JSON diffing from ch06's "diff recipes like code" section explicitly to tie the practices together.

4. The "Pico ≠ MCU" clarification in ch08 is thorough but repeated three times across the chapter. Two instances may suffice given the book's brevity target.

5. ch07's crash recovery narrative could include a single sentence quantifying the 1.5h→25m improvement to make the progress concrete for readers skimming.

## Fact-check sample
Pass 3: fresh 3% weighted toward revised sections.

| Claim (quoted) | Location | Cited source | Supported? (yes/no/partly) |
|---|---|---|---|
| "8192 ÷ 6 = 1365 (1365.3, floored)" | ch04 | Mathematical verification | yes |
| "Q8-MTP master (lab, new prod) 160 GB" | ch01/ch02 | Consistent across ch01, ch02, ch06, ch08 | yes |
| "±10 envelope was measured on one model (Q3-MTP), one harness (the 15-scenario tool-hardmode suite), one date (2026-07-13)" | ch05 | Self-consistent; scoped to stated conditions | yes |
| "That 100% is measured on exactly two DeepSeek MTP cells (Q8 at 14-spill, Q3 at 10-spill) at n_max=1" | ch03 | Consistent with §E table in ch03 | yes |
| "the expert weight tensors ship in the model's original MXFP4 block-float encoding, copied verbatim into the GGUF during conversion" | ch02 | Definition provided; consistent with claim of native precision | yes |

## Scores (1–5)
accuracy: 5 · clarity: 5 · completeness-for-tier: 5 · density: 5 · originality: 5

## Pass-3 only: findings ledger
| Finding # (from Pass 2) | Status: resolved / rebutted-accepted / still-open | Note |
|---|---|---|
| 1. Self-declared "unverified" stamp; no reproducibility framing | resolved | Verified-by attribution explicit in frontmatter and every chapter header; provenance.md explains apparatus and [LAB:] as lab-notebook index; reproducibility recipe stated. |
| 2. ch04 arithmetic error (1536 vs 1365) | resolved | Corrected to 1365 throughout ch04; 8192÷6=1365 explicitly stated with floor note. |
| 3. Q8-MTP master size inconsistent (149 GB vs 160 GB) | resolved | Unified to 160 GB across ch01, ch02, ch06, ch08. |
| 4. ±10 noise envelope over-generalized | resolved | Scoped to one model, one harness, one date, one serving shape in ch05; explicit warning against stapling to other suites. |
| 5. 100% acceptance presented as law without denominator or control | resolved | Denominator defined (first-token draft proposals); presented as "observation with an interpretation attached, not a proven law"; control gap acknowledged (no independent-draft-head control). |
| 6. Expert-precision confound; "native MXFP4" undefined | resolved | Confound caveat added explicitly; "passed through untouched" defined as verbatim MXFP4 block-float encoding copied into GGUF. |
