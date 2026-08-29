# Chapter 5 — Benchmarking Honestly

*(draft v0, 2026-08-28 — written by rogerai-dj for RogerAI Labs; unverified. Numbers
with a `[LAB:]` marker resolve into the lab record. Claims without one are labeled
unmeasured.)*

## Why this chapter exists

Every prior chapter leaned on tables. Tables can lie without any one cell being false.

A true 47 on a tool suite becomes a lie when it is sold as "the model is 47" without the
range, the harness noise, the parallel setting, the context per slot, or the control that
isolates the change you claim to have made. Honest benchmarking is not etiquette. It is
how you keep chapters 1–4 from laundering folklore through arithmetic.

## The scar: ±10 points at temperature 0

The capability matrix carries a footnote that should be taught in every local-inference
shop `[LAB: RESULTS-MATRIX §C footnote]`:

Q3-MTP tool hardmode, three runs on 07-13: **40**, **47**, **50** (MTP off control).
Five of fifteen scenarios flipped between identical back-to-back runs. Temperature was
0.0. The flips were attributed to **PAR=2 batch-packing nondeterminism amplified by MoE
routing**, not to sampling.

**Treat single-run hardmode numbers as ±10.**

If your promotion threshold is "beat 45," a 40 and a 50 are different religions. If your
marketing quotes the 50 without the 40, you are not benchmarking. You are fishing.

## What "temperature 0" does not guarantee

Temperature 0 removes one randomness source. It does not freeze:

- batch packing across concurrent slots
- MoE routing ties and implementation details
- GPU reduction order and kernel nondeterminism on some stacks
- cache-reuse paths
- any server-side scheduling that changes which requests share a batch

So "we set temp 0" is a necessary note, not a sufficiency proof. The lab had to name PAR=2
packing before the tool flips made sense. Your harness notes should be equally specific.

## Controls beat vibes

A control holds the thing you are not studying fixed.

Examples already in this book:

- **Parser vs quant on tools** `[LAB: RESULTS-MATRIX §D]`: template/parser patch scored 43
  versus baseline 47; Q4 experts scored 60. Without the parser control, someone would have
  "fixed tools" by rewriting prompts forever.
- **MTP on vs off for quality** `[LAB: RESULTS-MATRIX §C footnote]`: MTP-off control at 50
  sits inside noise of MTP-on 40/47 — so a speed feature was not purchased with a silent
  tool regression large enough to see through ±10.
- **Matched n_ctx_slot across PAR** `[LAB: RESULTS-MATRIX concurrency controls]`: PAR=1 at
  4096 versus PAR=4 at 4096 per slot, after unequal desks had confounded earlier reads.

If you cannot name the control, you are not measuring a change. You are measuring a
different Tuesday.

## Isolate one variable

The matrix is usable because rows try to change one axis at a time: engine build on the
same IQ3 file; expert precision on the same hardmode; n_max on the same spill class;
PAR on a fixed recipe.

Industrial failures often change five axes at once: new quant, new engine binary, new
parallel, new context, new prompt template — then declare victory. Maybe something
improved. You will not know what to keep when it regresses next month.

**Rule:** one change per claim. Bundle changes only as a named recipe promotion, and then
accept that the unit of claim is the recipe, not a single flag.

## Ranges, not heroes

Report:

- n runs
- min / median / max or mean ± spread
- known nondeterminism sources
- suite size (15 scenarios is not 1500)

The Q8-MTP master tool runs of 73 / 50 / 43 (mean 55) are more honest than "55" alone
because the spread is visible, and because scenario-consistent wins (TC-71, TC-78, TC-70)
were called out separately `[LAB: RESULTS-MATRIX §C notes]`. Means hide structure; structure
is often what you ship.

## Suite identity is part of the claim

"MMLU 88.3" and "tool hardmode 55" are different products. On §C they do not even rank
models the same: Qwen3.6-35B-A3B posts 71.0 MMLU and **87** tools; DeepSeek Q8-MTP posts
**88.3** MMLU and mean 55 tools `[LAB: RESULTS-MATRIX §C]`.

If your customers buy tool reliability, promoting on MMLU is a category error. If your
customers buy general knowledge chat, promoting on a 15-scenario tool suite is a category
error. Honest benchmarking starts with **which decision the number is allowed to drive**.

## Methodology findings are results

Sometimes the finding is that the suite cannot answer the question. DeepSeek IFEval was
marked DNF because long-form outputs at ~6 tok/s/slot made the method impractical on that
model class `[LAB: RESULTS-MATRIX §C]`. Publishing DNF beats publishing a fantasy score.

Likewise, the lab's later instrument defects and retractions (including sections withdrawn
when collectors were wrong) are not stains to hide. They are how you stop living on
corrupted numbers. A benchmark program without retractions is usually a benchmark program
without teeth.

## A practical honesty protocol

1. **Write the decision** the metric will authorize (promote, reject, ship flags).
2. **Write the suite** and its size; do not let a 15-item tool set impersonate a universe.
3. **Pin the recipe:** engine SHA/build, quant identity, parallel, per-slot context, KV
   precision, speculation settings, warm/cold policy.
4. **Run ≥3 times** when noise is known; publish the range.
5. **Hold one control** that would falsify your favorite story (parser, MTP off, matched
   context, single-stream vs batched).
6. **Separate speed claims from quality claims**; each gets its own table.
7. **Record DNF and abort conditions** (too slow, OOM, instrument defect).
8. **Refuse cross-engine leaderboards** unless engine is the variable under study.

## How this rewrites earlier chapters

- Chapter 1's tok/s tables are speed claims under named builds — not model essence.
- Chapter 2's quant ladder is only as strong as the tool ranges and the §D controls.
- Chapter 3's MTP multipliers require acceptance and quality controls, not just peak
  tok/s.
- Chapter 4's parallelism story required matched desks before quality differences were
  speakable.

Honest benchmarking is the load-bearing wall between measurement and myth.

## What this chapter refuses to claim

- We do not claim three runs are always enough; they were the minimum that made ±10
  visible on this harness.
- We do not claim temperature 0 plus fixed seeds makes MoE serving deterministic under
  batching.
- We do not claim MMLU-100 or a 15-scenario tool suite is an industrial certification.
- We do not claim every instrument defect in the wider lab record is fully narrated here;
  where this book cites a number, it cites the surviving one.



## Worked example: a bad promotion paragraph

Bad:

> Our Q3 build scores 50 on hardmode tools and 30 tok/s. We are promoting it.

Better:

> Q3-MTP on llama.cpp build X, PAR=2, per-slot context Y, KV precision Z, MTP n_max=1,
> warm single-stream. Tool hardmode (15 scenarios) over three runs: 40, 47, 50 MTP-off
> control. Spread ±10; temperature 0; flips attributed to batch packing. Decode 30.5
> tok/s warm on the reference box `[LAB: RESULTS-MATRIX §C/§E]`. Decision: morning
> production only, with master-precision follow-up preferred for tool floor.

The second paragraph can be audited. The first can only be believed.

## Speed benches need the same medicine

Quality noise is obvious because scores look like grades. Speed noise hides inside
averages.

Honest speed reports include:

- warm versus cold (first request after load vs steady state)
- context length and generation length
- single-stream versus aggregate under c=N
- whether prefill is mixed into the number
- hardware identity and engine identity
- range across runs, especially near thermal limits (chapter 7)

Chapter 1's engine table already showed a bimodal mainline CUDA decode (~2.6–19) versus a
stable pr25545 band (24.5–28.5) `[LAB: RESULTS-MATRIX §A]`. Publishing 10.8 without the
bimodality would have been a quieter kind of lie — the mean of a broken process.

If your latency SLO is a percentile, bench a percentile. Means flatter than your pager.

## Cross-model leaderboards

Putting gpt-oss under vLLM TP=4 next to DeepSeek under llama.cpp on the same table is
legal only if the caption says engines differ `[LAB: RESULTS-MATRIX §C]`. It is illegal as
a pure model ranking.

The honesty move is either:

1. Fix the engine and vary the model, or
2. Fix the model and vary the engine, or
3. Label a recipe contest and stop pretending it is a model contest.

Most public charts do (3) while claiming (1). Do not import that habit into a plant or a
pager rotation.

## Small suites and scenario notes

A 15-scenario tool suite can still be invaluable. It cannot bear the weight of a general
intelligence claim. What it can do — and what the lab used it for — is catch consistent
scenario failures and wins across runs (TC-70/71/78 notes on the master) `[LAB:
RESULTS-MATRIX §C notes]`.

Practice:

- Publish scenario-level notes for the failures that drive the mean.
- Do not let a single flaky scenario veto a recipe without a rerun policy.
- Do not let a single lucky scenario promote a recipe without a rerun policy.

## When to stop measuring and fix the instrument

If two scorers disagree by huge margins, or a collector can invent reachability, or a rope
base is wrong by orders of magnitude, you are not in model-debugging land. You are in
instrument land. The wider lab record includes retractions forced by bad instruments; the
correct behavior is stop, repair, redo, and leave the tombstone visible.

A clean culture prefers a loud retraction to a quiet wrong leaderboard.



## Building a tiny product suite that is still honest

You do not need a 10,000-item academic suite to make promotion decisions. You need a suite that:

- matches the product risk
- is large enough that one flaky item cannot dominate without being visible
- is small enough to run multiple times
- is versioned and hashed

The 15-scenario tool hardmode set is small and still caught consistent scenario structure across runs when the team looked beyond the mean `[LAB: RESULTS-MATRIX §C notes]`. Copy the attitude, not necessarily the item count.

Version the suite like code. When you change items, you break comparability. Say so.

## Reporting template (quality)

```
suite: tool-hardmode@2026-07-13
recipe: <engine> <model> <flags>
n_runs: 3
scores: [40, 47, 50]
spread_note: ±10 known; temp 0; PAR=2 packing nondeterminism
controls: MTP-off=50
decision: ...
```

## Reporting template (speed)

```
metric: warm_single_stream_decode
context: ...
gen_len: ...
runs: [...]
hardware: 4x RTX PRO 4500 128GB VRAM ...
engine: pr25545
notes: prefill separate = ...
```

If a report cannot fill these, it is not ready for a promotion meeting.

## Nondeterminism inventory

Keep a living list of nondeterminism sources on your stack:

- batch packing
- MoE routing
- kernel reductions
- cache reuse
- network-loaded tokenizer files
- concurrent background jobs

When a score jumps, check the inventory before checking the model mythology. The §C footnote exists because someone did that work `[LAB: RESULTS-MATRIX §C footnote]`.

## Ethics of numbers

Publishing the max of three runs without the min is a choice. Publishing cross-engine ranks without labels is a choice. Publishing DNF as a quiet omission is a choice. Honest benchmarking is ethics for people who ship systems that others will trust with work.


## Field story: three runs that disagreed

40, 47, 50. Same week, same stack family, temperature 0 `[LAB: RESULTS-MATRIX §C footnote]`. The honest response is not to average them into a press release. It is to widen the error bar, name the nondeterminism, and stop making 2-point promotion gates.

If your organization currently promotes on a single run, this footnote is your incident report from the future. Steal it before you earn it.

## Benchmark ownership

Every product suite needs an owner who can answer:

- what decision it authorizes
- how often it runs
- what changed since last week
- what the current range is

Orphan suites become Halloween decorations: visible, scary, not load-bearing.


## Operator lab: the promotion packet

A promotion packet is a single markdown file containing:

- recipe hash
- quality range table
- speed range table
- controls run
- fit/soak notes
- known nondeterminism
- owner signature

No packet, no promote. This is how you stop hallway decisions. The §C footnote and §D controls are the ancestors of a good packet `[LAB: RESULTS-MATRIX §C/§D]`.


## What you should do Monday

1. Ban single-run promotions for any suite with known noise.
2. Create a promotion packet template and reject hallway ships that lack one.
3. Write the nondeterminism inventory for your stack in a shared doc.
4. Label every cross-engine chart as a recipe contest or stop making it.
5. Assign an owner to each product suite with a paging path for suite breakage.

The ±10 footnote is not trivia; it is a governance rule waiting to be adopted `[LAB: RESULTS-MATRIX §C footnote]`.


## Cross-links inside this book

Every table in chapters 1–3 is only as strong as this chapter's ranges and controls. Fit claims in chapter 8 need the same packet discipline as quality promotions. Load-log diffs in chapter 6 are controls too: they isolate recipe change from mythology. If you cannot say which decision a number authorizes, it does not belong in a promote meeting.


## One-page honesty pledge

Before any external claim about local model speed or quality leaves your org, a named person signs:

- recipe identity known
- range published
- controls named
- hardware class named
- failed recipes not deleted

This is not bureaucracy. It is how you keep chapter 5 from becoming optional when marketing is in a hurry. The lab footnote that forced ±10 on a temp-0 tool suite is the pledge's ancestor `[LAB: RESULTS-MATRIX §C footnote]`.

## Looking ahead

Chapter 6 shows what to read when the number is "weird" before you redesign the model:
the load log. Chapter 7 puts environmental nondeterminism — heat, power loss — on the
same honesty ledger. Chapter 8 asks what fits, which is itself a benchmark of residence
and failure flags, not a vibes-based shopping list.

A number without a recipe is a rumor. A number without a range is a dare. A number
without a control is an advertisement.
