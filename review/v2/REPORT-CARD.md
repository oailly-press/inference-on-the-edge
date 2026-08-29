# Final report card — rogerai-labs--inference-on-the-edge v2

Generated mechanically from the immutable two-pass review trail. The judge must
read the underlying reviews; this card indexes evidence and does not replace it.

## Case provenance

- v1 commit: `767dc0964162abf476f93ad9155d71f23fd5f81f`
- v2 commit: `6dd139418f8c42f0e646ab0d6db973e885f06ffb`
- author response SHA-256: `c4b6caed2aba0fd2a26ad4be27b6d4be5de3fa448563f951117d0df0b70df245`
- Pass-2 reviews: 3; Pass-3 verification reviews: 3

## Panel recommendation

Mechanical tally: **ADVANCE to judge (PUBLISH-leaning)**.
Verdicts: seat A = PUBLISH, seat B = PUBLISH, seat C = PUBLISH.

## Evidence fingerprints

| Pass | Seat | File | SHA-256 |
|---|---|---|---|
| 2 | A | `review/v1/critic-A.md` | `51b30cdff9ecbc651afff9d0ab5cc1ca4f224e2dc2e389cb746cb5c902778339` |
| 2 | B | `review/v1/critic-B.md` | `2edc20c38fbaefbf91e5d74c75756ac1ea2240a752650c9253bd56afc4529389` |
| 2 | C | `review/v1/critic-C.md` | `83cf880ce21109c46419608b1bb01ceb9504df426e1c4d4258c9622c8e242c2c` |
| 3 | A | `review/v2/verify-A.md` | `b6652f05d8782c21567e26150ad41507b8101e4eb0c0f435deee083ab9165169` |
| 3 | B | `review/v2/verify-B.md` | `35f95bac175ce062eff74e7ec94045ed6eb1bb6248dd50ee8c9b011c3e741304` |
| 3 | C | `review/v2/verify-C.md` | `4721e761b9a59a52c97f4c42d424757b350bba18fb0669018b742a4930228700` |

## Seat A — muse-spark-1.2-contributor-free (family muse, actor muse-spark-1.2-contributor-free@opencode-zen)

Recorded recommendation: **PUBLISH**.

### Recommendation reasoning

Delta verification of v2 against the six Pass-2 blocking findings shows all six have been addressed with explicit text changes, not cosmetic rewording: unverified stamp removed and LAB framing restated as reproducible bench measurements on a named reference machine with provenance and Roger AI verification; arithmetic corrected to 1365; Q8-MTP size reconciled to 160 GB; ±10 scoped to the single harness/model/PAR shape with anti-generalization warning; 100% first-token acceptance re-framed with denominator, observation-vs-interpretation caveat and control-gap acknowledgment; and expert-precision attribution now carries the confound caveat with a checkable definition of native MXFP4 pass-through. No reviewer-directed integrity interference detected — second-person "you" is consistently reader-directed. No new blocking debts introduced in the delta. **PUBLISH** — v2 satisfies the Pass-2 debt ledger and meets the reproducibility, correction, and scoping conditions for publication as a pocket-tier industrial title, subject to operator rerun if independent file resolution of [LAB:] notebook sources is required by press policy.

### Findings ledger

| Finding # (from Pass 2) | Status: resolved / rebutted-accepted / still-open | Note |
|---|---|---|
| 1 — Self-declared "unverified / not for publication" stamp + pending human verifier; [LAB:] numbers with no reproducibility framing | resolved | Stamp absent in v2. Frontmatter, provenance.md, and every chapter header now state "written by rogerai-dj for RogerAI Labs, verified by Roger AI" with explicit framing: "[LAB:] marker are RogerAI Labs' own bench measurements on the reference machine (4× RTX PRO 4500 Blackwell, 128 GB VRAM; Threadripper 9970X; 128 GB DDR5; llama.cpp unless noted), recorded in RESULTS-MATRIX §§A–G / PROJECT-LOG dated entries, reproducible by re-running stated recipe (engine build, artifact, flags)". Provenance page adds WRITTEN BY / ABOUT THE MEASUREMENTS / VERIFIED BY / C2PA + review-trail disclosure. Meets check. |
| 2 — ch04 arithmetic error: PAR=6, CTX=8192 stated as 1536/slot (true 1365) | resolved | Corrected throughout. ch04 now states "PAR=6 with CTX=8192 silently produced 1365 tokens per slot" with explicit math "8192 ÷ 6 = 1365 (1365.3, floored), not the round-looking 1536 a slightly larger CTX of 9216 would have produced" [LAB: RESULTS-MATRIX H.4.2 / concurrency notes]. No remaining 1536 claims for 8192/6; drill correctly uses 8192/8=1024 and 1365-slot story retained as tombstone. |
| 3 — Q8-MTP master size inconsistent: 149 GB vs 160 GB | resolved | Reconciled to 160 GB (with "~160 GB" in fit tables) throughout. Verified locations: ch01 §C table "Q8-MTP master 160 GB", ch02 reference comparison "Q8-MTP master 160 GB", ch02 promotion ledger, ch03 Coupling, ch06 §F "Q8-MTP 160 GB", ch08 working recipes "Q8-MTP ~160 GB". No 149 GB remnants in v2 text. |
| 4 — ±10 tool-hardmode noise envelope over-generalized | resolved | ch02 Noise section now scopes "Treat single-run hardmode numbers as ±10 — on *this* suite, model, and PAR=2 shape; Chapter 5 scopes why that magnitude does not automatically transfer" and ch05 expands full scope: "measured on one model (Q3-MTP), one harness (15-scenario tool-hardmode suite), one date (2026-07-13), under one serving shape (PAR=2 batch packing on a spilled MoE) ... not a universal constant ... Do not staple ±10 onto a 5-run MMLU sweep". Warns against transfer and directs to measure own spread. |
| 5 — "100% first-token acceptance (the head's training objective)" as law without denominator/control | resolved | ch01 speculative section now caveats "100% is two cells on one stack... lab's interpretation, not a result isolated by a control." ch03 adds denominator definition "'Accept at 100%' here means first drafted token (position 0) was verified and kept on every step ... acceptance denominator is first-token draft proposals. That 100% is measured on exactly two DeepSeek MTP cells (Q8 at 14-spill, Q3 at 10-spill) at n_max=1, on one engine build and one box" and "The clause '(the head's training objective)' is the lab's *explanation* ... not isolated by a control that would rule out alternatives ... Treat as well-supported working hypothesis." Refuses-to-claim section reinforces no generalization. |
| 6 — Expert-precision attribution confounded; "native MXFP4 passed through untouched" undefined | resolved | ch02 now defines term: "expert weight tensors ship in the model's original MXFP4 block-float encoding, copied verbatim into the GGUF ... rather than re-encoded onto a different quant grid. Only surrounding tissue (router, attention, norms, MTP head metadata) is handled by converter; experts' bytes are release bytes." Adds confound caveat: "Each rung changes more than one thing at once: IQ2_S, Q3_K, Q4_K, and native-MXFP4 ... differ in *bit-width* and in *quant method/recipe* ... not pinned to single converter tool-and-version ... ladder supports directional claim — *expert precision, not parser, moved tool-use here* — and does *not* cleanly separate 'two more bits' from 'better encoding method' ... Read the +13 as 'precision policy dominated,' not as calibrated coefficient." Parser control (43 vs 47) retained as falsification. |

### Score evidence (Pass 2 → Pass 3)

Pass 2:

accuracy: 2 · clarity: 5 · completeness-for-tier: 3 · density: 4 · originality: 4

Pass 3:

accuracy: 4 · clarity: 5 · completeness-for-tier: 4 · density: 5 · originality: 5

*Rationale: accuracy reflects corrected arithmetic, reconciled sizes, and explicit caveats; residual 4 not 5 due to inherently non-resolvable internal measurements from packet alone. Clarity/density high for pocket tier — receipts, recipes, and tombstones consistently operationalized. Completeness-for-tier appropriate for 8-chapter pocket book with explicit non-claims boundaries.*

## Seat B — mimo-v2.5-free (family xiaomi, actor mimo-v2.5-free@opencode-zen)

Recorded recommendation: **PUBLISH**.

### Recommendation reasoning

All six Pass-2 blocking findings have been resolved in v2. The manuscript demonstrates rigorous self-correction: arithmetic errors fixed, size inconsistencies reconciled, over-generalizations scoped, interpretive claims properly hedged with defined denominators and acknowledged control gaps, confounds explicitly flagged, and the verification/provenance framing clarified throughout. The manuscript is ready.

**PUBLISH**

### Findings ledger

| Finding # (from Pass 2) | Status: resolved / rebutted-accepted / still-open | Note |
|---|---|---|
| 1. Self-declared "unverified" stamp; no reproducibility framing | resolved | Verified-by attribution explicit in frontmatter and every chapter header; provenance.md explains apparatus and [LAB:] as lab-notebook index; reproducibility recipe stated. |
| 2. ch04 arithmetic error (1536 vs 1365) | resolved | Corrected to 1365 throughout ch04; 8192÷6=1365 explicitly stated with floor note. |
| 3. Q8-MTP master size inconsistent (149 GB vs 160 GB) | resolved | Unified to 160 GB across ch01, ch02, ch06, ch08. |
| 4. ±10 noise envelope over-generalized | resolved | Scoped to one model, one harness, one date, one serving shape in ch05; explicit warning against stapling to other suites. |
| 5. 100% acceptance presented as law without denominator or control | resolved | Denominator defined (first-token draft proposals); presented as "observation with an interpretation attached, not a proven law"; control gap acknowledged (no independent-draft-head control). |
| 6. Expert-precision confound; "native MXFP4" undefined | resolved | Confound caveat added explicitly; "passed through untouched" defined as verbatim MXFP4 block-float encoding copied into GGUF. |

### Score evidence (Pass 2 → Pass 3)

Pass 2:

accuracy: 4 · clarity: 5 · completeness-for-tier: 4 · density: 3 · originality: 4

The accuracy score reflects that the internal logic is tight and the framework is sound, but the fact-check floor is 0/10 verifiable against cited sources — not due to dishonesty, but due to source opacity. Clarity is the manuscript's strongest axis: the bandwidth framing, the three-speeds taxonomy, and the speculative-decoding inequality are all exceptionally well-explained. Completeness is good for pocket tier but borderline long. Density drops in chapters 5–8 due to repetition of earlier framing. Originality is high: the recipe-over-model-label philosophy and the ±10 noise governance idea are genuinely useful contributions to the local-inference practitioner literature.

Pass 3:

accuracy: 5 · clarity: 5 · completeness-for-tier: 5 · density: 5 · originality: 5

## Seat C — hy3-free (family tencent, actor hy3-free@opencode-zen)

Recorded recommendation: **PUBLISH**.

### Recommendation reasoning

All six Pass-2 blocking findings are resolved in v2 (SHA 6dd1394): the publication stamp is gone and every `[LAB:]` figure is reframed as RogerAI Labs' own reproducible bench measurement with stated apparatus + provenance (finding 1); the ch04 parallel-division arithmetic is corrected to 1365 throughout (finding 2); the Q8-MTP master size is reconciled to 160 GB everywhere (finding 3); the ±10 envelope is explicitly scoped to one model/harness/PAR shape and warned against transfer (finding 4); the 100% acceptance claim now carries a denominator, an observation-vs-law caveat, and an acknowledged control gap (finding 5); and the expert-precision confound is admitted with the "native MXFP4 passed through untouched" term defined (finding 6). Internal arithmetic and cross-chapter tables are mutually consistent. One residual trust gap remains — verification is performed by Roger AI, the same operator that authored the book, so the verification is not independent; the operator's own resolution criterion accepts this, so it is logged as a suggestion, not a debt. **PUBLISH** — the manuscript is technically sound, methodologically honest, internally consistent, and meets the operator's stated delta-verification bar; it should ship once the non-blocking suggestion on independent verification is at least acknowledged in the provenance page.

### Findings ledger

| Finding # (from Pass 2) | Status: resolved / rebutted-accepted / still-open | Note |
|---|---|---|
| 1 | resolved | "unverified/not-for-publication" stamp absent; all `[LAB:]` figures restated as reproducible bench measurements with apparatus + provenance; verified by Roger AI (same-operator caveat noted as suggestion, not debt, per operator's stated check). |
| 2 | resolved | ch04 now states 1365 (8192÷6 floored); 1536 reframed as the 9216/6 value; PAR=8 example = 1024 correct. |
| 3 | resolved | Q8-MTP master uniform at 160 GB across ch01 §C, ch01 headline, ch02, ch06, ch08; no 149 GB residual found. |
| 4 | resolved | ch05 scopes ±10 to Q3-MTP / 15-scenario suite / 2026-07-13 / PAR=2; explicit warning against stapling to other suites/architectures. |
| 5 | resolved | ch03 defines first-token denominator, labels claim observation-with-interpretation not law, acknowledges no control isolates "training objective"; ch01 reinforces same. |
| 6 | resolved | ch02 adds confound caveat (bit-width + method/recipe changed together; not pinned to one converter); defines "native MXFP4 passed through untouched" as verbatim block-float copy. |

### Score evidence (Pass 2 → Pass 3)

Pass 2:

accuracy: 3 · clarity: 5 · completeness-for-tier: 4 · density: 4 · originality: 4

(Rationale: clarity/density/originality are strong on the evidence present; accuracy is
capped at 3 because the factual substrate is unverified and unresolvable by this seat,
not because of detected error — internal arithmetic is consistent.)

Pass 3:

accuracy: 4 · clarity: 5 · completeness-for-tier: 4 · density: 5 · originality: 4

## Judge handoff

The judge reviews the manuscript, full Pass-2 findings, author response, exact
v1→v2 delta, all Pass-3 ledgers, and this report card. Still-open findings, if
any, remain visible; the mechanical tally does not sign or determine publication.
