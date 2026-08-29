# Provenance

**WRITTEN BY** rogerai-dj, operated by RogerAI Labs.

**ABOUT THE MEASUREMENTS.** The quantitative spine of this book — every tok/s, MMLU,
tool-hardmode, spill, headroom, and acceptance figure carrying a `[LAB:]` marker — is
**RogerAI Labs' own bench measurement**, not a citation the reader is asked to go and
resolve in a file they do not have. The apparatus is stated in the book: a single named
reference machine (4× RTX PRO 4500 Blackwell, 128 GB VRAM; Threadripper 9970X; 128 GB
DDR5), llama.cpp unless a row says otherwise, warm single-stream decode as the default
speed, with the exact recipe (engine build, artifact, and flags) given alongside each
number so it can be **re-run and confirmed or refuted**. The `[LAB:]` marker names the
section of RogerAI Labs' internal lab notebook (`RESULTS-MATRIX.md` §§A–G,
`PROJECT-LOG.md` dated entries) where that run was first recorded; it is a lab-notebook
index, in the manner of any industrial white paper reporting its own instrument, not a
pointer to an external authority. Where a claim is not measured, the prose says so, and
where a measurement is an approximation or an interpretation the text labels it as such.

**GROUNDED IN**
- RogerAI Labs' bench on the reference machine described above (the primary instrument).
- The lab notebook that recorded those runs: `RESULTS-MATRIX.md` (engine, quant,
  concurrency, MTP, fit, soak) and `PROJECT-LOG.md` (dated crash recoveries, cache-reuse
  notes, hybrid KV arms).
- llama.cpp, the open inference engine used for the measured decode numbers:
  https://github.com/ggml-org/llama.cpp
- llama.cpp build/backend notes for GPU vs CPU indexer paths:
  https://github.com/ggml-org/llama.cpp/blob/master/docs/build.md

**VERIFIED BY** Roger AI, RogerAI Labs. The measured claims in this book have been checked
against the RogerAI Labs lab record they were drawn from, and the arithmetic and internal
consistency of the tables have been reconciled for this edition.

**DISCLOSURE** Written by a model stack (rogerai-dj) operated by RogerAI Labs. The numbers
are the lab's own bench measurements, reported with the recipe needed to reproduce them and
verified against the lab record. No hidden AI, no hidden humans.

**REVIEW TRAIL** publishes with the book: the complete critic reviews, this revision, and
the judge verdict.

**C2PA** signed at publication.
