# Chapter 3 — Speculative Decoding Economics

*(draft v0, 2026-08-28 — written by rogerai-dj for RogerAI Labs; unverified. Numbers
with a `[LAB:]` marker resolve into the lab record. Claims without one are labeled
unmeasured.)*

## The slogan

Speculative decoding says: let a cheap draft propose several tokens, let the strong
model verify them in a batch, keep the prefix that matches. If the draft is good, you
bought multiple tokens per expensive step. If the draft is bad, you paid overhead for
rejects.

As a slogan, it is almost always sold as a free speedup. As an accounting problem, it
is a bet on **accepted tokens per heavy read**, minus the cost of drafting and verifying
under your real spill and batch constraints.

This chapter is the bet, priced on the reference box.

## The only equation that matters

Let:
- \(H\) be the cost of one heavy (target) step without speculation
- \(D\) be the cost of drafting a candidate span
- \(V(n)\) be the cost of verifying \(n\) draft tokens in a batch
- \(a\) be the number of draft tokens actually accepted on average

A speculative step is a win when:

**\((D + V(n)) / a < H\)**

Everything else is commentary.

People lose money on speculation three ways:

1. **\(a\) collapses** — draft quality is poor, so you accept ~1 token after paying \(D+V\).
2. **\(V(n)\) explodes** — verification is not a cheap batch on your hardware recipe;
   it rereads spilled experts \(n\) ways.
3. **\(D\) is not actually cheap** — the draft model or draft head is large, cold, or
   contending for the same bus.

You do not need Greek letters in production. You need to notice which term broke.

## What the lab implemented

On the DeepSeek V4 line, the lab ran multi-token prediction (MTP) heads — draft tokens
from a head trained for that job — through llama.cpp on the reference box. The matrix
labels the DeepSeek rows as the lab's MTP implementation and includes a dense Qwen
stock-MTP row as a near-zero-spill contrast `[LAB: RESULTS-MATRIX §E]`.

This chapter does not claim a survey of every speculative system (Medusa, EAGLE,
independent draft models, etc.). It claims a measured economic pattern: **speedup
tracks spill and acceptance, not enthusiasm.**

## The §E table, in full

| Model / spill | Baseline | n_max=1 | n_max=2 | n_max=3 |
|---|---|---|---|---|
| Qwen3.6-27B Q8 (zero spill, stock MTP) | 27.0 | — | — | **59.3 (2.2×)** |
| DeepSeek Q8-MTP · 24 layers spilled | 19.3 | — | — | 16.6 (0.86×) @ 82% |
| DeepSeek Q8-MTP · 14 layers spilled | 22.2 | **26.3 (1.18×) @ 100%** | 21.6 @ 89% | 18.8 @ 78% |
| DeepSeek Q3-MTP · 10 layers spilled | 23.5 | **30.5 (1.30×) @ 100%** | 24.8 @ 93% | — |

`[LAB: RESULTS-MATRIX §E]`

The matrix's own law:

> Speedup grows as spill shrinks (1.18× → 1.30× → 2.2×); on spill-bound MoE,
> batch-verify costs ~N× DDR5 expert reads → n_max=1 is the sweet spot; first-token
> drafts accept at 100% (the head's training objective).

Memorize the shape, not just the hero cell.

## Zero spill is a different planet

The dense Qwen row is the optimistic planet: **2.2×** at n_max=3, baseline 27.0 → 59.3,
with zero spill. Verification can batch without shipping expert bodies across host
memory for each candidate. The draft length can open up.

If your mental model of speculation was formed on dense, resident models, you will
over-promise on MoE-with-spill. The Qwen row is real. It is also not a license to quote
2.2× on a DeepSeek recipe that spills fourteen layers.

## Spill turns verification into a bill

Watch the DeepSeek Q8-MTP 14-spill row as draft length grows:

- n_max=1: **1.18×** at **100%** acceptance
- n_max=2: 0.97×-ish territory (21.6 from 22.2) at 89% acceptance
- n_max=3: 18.8 at 78% acceptance — slower than baseline

The draft is not "getting stupid" in a narrative sense alone. Acceptance falls, and
verification cost rises with \(n\) while each verify still risks DDR5 expert traffic.
The matrix is blunt: batch-verify costs scale with N times those reads.

Now the 24-spill row at n_max=3: **0.86×** at 82% acceptance. Speculation loses
outright. You paid to go slower.

This is chapter 1's bandwidth thesis wearing an MTP hat. If verification multiplies
expensive host reads, longer drafts are not brave — they are leveraged debt.

## Why n_max=1 won on the MoE recipes

On both DeepSeek MTP rows that show n_max=1, first-token acceptance is **100%**, and
that cell is the local optimum:

- Q8-MTP 14 spill: 1.18× @ 100%
- Q3-MTP 10 spill: 1.30× @ 100%

The matrix attributes 100% first-token acceptance to the head's training objective.
Practically, n_max=1 means: take the head's best single guess, verify it cheaply
relative to a long draft, keep the win rate high, do not open a large batch-verify
surface against spilled experts.

Longer drafts are not forbidden forever. They become rational when spill shrinks (more
resident experts, different placement) or when the architecture is dense and resident
like the Qwen row. The economic mistake is copying n_max=3 from a blog about a dense
model onto a spilled MoE and calling the slowdown "weird."

## Acceptance rate is a first-class metric

Tok/s without acceptance is how folklore hides losses.

A run can show busy GPUs, high internal throughput, and still lose end-to-end if rejects
dominate. The §E cells pair speedup with acceptance percentages for a reason. On the
14-spill Q8 line, acceptance slides 100% → 89% → 78% as n_max climbs, and speedup slides
with it into a loss.

Instrument acceptance beside tok/s:

- mean accepted length
- per-position accept rate if you have it
- reject cost (time spent on discarded drafts)

If you only watch tok/s, a bad draft policy can look "busy-fast" while users wait
longer.

## Speculation and quality: the tool-suite control

Speed features that quietly wreck behavior are not speed features. The lab checked MTP
against tool hardmode on Q3-MTP: three runs at MTP n=1 (40, 47) and an MTP-off control
(50). Within the suite's ±10 noise, MTP did not measurably harm tool use `[LAB:
RESULTS-MATRIX §C footnote]`.

That is not a universal safety certificate for all speculative methods. It is a
recorded control on this implementation and harness. When you adopt any draft scheme,
keep a product-critical suite on a leash. Speculation that buys 1.2× and loses your
tool contracts is a bad trade even if the latency dashboard celebrates.

## Coupling to the production promotion

Chapter 2's promotion story depends on this chapter's economics. The Q8 master kept
native expert precision (quality) and used MTP n_max=1 to land at 26–27 tok/s — old
production speed — despite a larger resident footprint and fourteen layers of spill in
the related MTP rows `[LAB: RESULTS-MATRIX headline before/after + §E]`.

Without speculation, the quality-preserving master might have been "too slow" under a
naive decode comparison. With speculation priced correctly (n_max=1, not n_max=3),
quality and latency could cohabit.

That is the industrial pattern: **buy quality in the weights; buy back latency with a
speculation recipe that respects spill; do not buy latency by vandalizing experts.**

## Failure modes checklist

1. **Blog-default draft length.** n_max=3 copied from dense zero-spill onto spilled MoE.
2. **Unmeasured acceptance.** Tok/s reported alone.
3. **Verification across host spill.** Batch-verify multiplies DDR traffic.
4. **Draft contention.** Draft and target fight for the same scarce bandwidth.
5. **Silent quality drift.** No tool/product suite paired with the speed claim.
6. **Cold vs warm confusion.** Speculation numbers measured only in a warm steady state
   that production never sees (unmeasured here as a general warning; pin your own).

## How to price a speculation change on your box

A minimal recipe:

1. Fix model, engine build, placement flags, and context length.
2. Measure baseline single-stream decode (range of ≥3 runs if noisy).
3. Enable speculation at n_max=1; measure tok/s and acceptance.
4. Step n_max upward only while acceptance stays high and tok/s rises.
5. Stop at the first step that loses end-to-end speed or breaks the product suite.
6. Record spill / residency (how much of the model is off-GPU) beside the winning cell.

If step 4 never leaves n_max=1, that is a result, not a failure of nerve.

## What this chapter refuses to claim

- We do not claim MTP is the only speculative method worth using.
- We do not claim 2.2× is available on MoE with heavy spill.
- We do not claim 100% first-token acceptance generalizes beyond the recorded head and
  setup.
- We do not claim speculation replaces the need for fit recipes (chapter 8) or load-log
  literacy (chapter 6).
- We have not published a full cost model in microseconds for \(D\) and \(V(n)\) on every
  row; the table is the evidence, the inequality is the frame.



## Worked reading: three cells, three decisions

Take the Qwen zero-spill 2.2× cell first. Decision: open draft length. Reason: verification
is not paying host expert traffic, acceptance stays useful enough to clear a double, and
the baseline is already a clean dense decode. Copying this cell into a spilled MoE runbook
is a category error.

Take the DeepSeek Q3-MTP 1.30× @ n_max=1 cell second. Decision: keep drafts short, keep
the head's first token, bank a solid single-stream gain, promote only if tool controls stay
inside noise. This is the cell that made a quality-preserving production story possible at
old latency.

Take the DeepSeek Q8-MTP 24-spill 0.86× cell third. Decision: stop. Reason: acceptance is
not terrible (82%), but the bandwidth math still loses. The correct response is not "train
a braver draft." It is "reduce spill or shorten drafts until the inequality flips."

If your postmortem after a slow rollout starts with sampler tweaks instead of spill and
n_max, you are debugging the wrong layer.

## Interaction with concurrency

Chapter 1 separated single-stream decode from aggregate throughput. Speculation complicates
both.

A single stream may show a clean 1.2–1.3× while multi-slot serving changes batch-verify
economics, cache pressure, and scheduler behavior. The concurrency table in §B was measured
on the promoted engine path without turning this chapter into a full factorial, and the
matrix still records an unresolved c=2/c=3 dip on PAR=4 `[LAB: RESULTS-MATRIX §B]`. That
dip is a reminder: scheduler seams exist even before MTP.

Practical rule: **re-measure speculation at the concurrency you will actually serve**, not
only at c=1 on a quiet box. If you only ever bench solitary warm decode, you will ship a
lab speedup and a production shrug.

## Prefill versus decode under speculation

Speculation is usually a decode-path bet. Prefill still builds the KV cache and still
dominates short-prompt / long-generation handoffs differently than long-prompt / short-
generation jobs. The §A prefill note (about 130 tok/s GPU indexer vs 50–80 CPU indexer on
the IQ3 builds) remains the prefill story `[LAB: RESULTS-MATRIX §A]`.

Do not advertise an MTP decode multiplier as an end-to-end latency multiplier for
prompt-heavy workloads without measuring time-to-first-token and time-to-last-token
separately. Operators who crush those into one "tok/s" will mis-price queues.

## A short field worksheet

When someone proposes enabling or widening speculation, fill this before merging flags:

1. Model identity and precision recipe (master? Q3? community Q4?).
2. Spill / residency summary (what is on GPU vs host).
3. Baseline decode range at target context.
4. n_max tried; acceptance at each step.
5. Product-suite delta (tools or equivalent) on/off.
6. Concurrency target and re-bench at that c.
7. Decision: keep, widen, narrow, or disable — with the inequality term that dominated.

If the worksheet cannot be filled, the change is not priced. Unpriced speculation is how
free-speedup folklore re-enters through a side door.



## Accounting worksheet you can copy

When a teammate says "just turn on MTP," make them fill this table before the flag ships:

| Field | Baseline | Candidate | Notes |
|---|---|---|---|
| engine build |  |  | pin SHA |
| model artifact |  |  | path + checksum |
| spill / n-cpu-moe |  |  | from load log |
| n_max | off |  | |
| mean accept length | n/a |  | |
| warm decode tok/s (3 runs) |  |  | |
| tool suite runs |  |  | |
| production concurrency |  |  | re-bench mandatory |
| decision |  | keep/reject | owner name |

If the candidate column cannot beat baseline on the product's real concurrency without wrecking the suite range, the feature is not a feature. It is a lab toy. The §E rows that fall below 1.0× exist to give you permission to reject `[LAB: RESULTS-MATRIX §E]`.

## Draft quality is a systems property

People say "the draft model is weak" when acceptance is low. Sometimes that is true. Often the draft is fine and the system is feeding it a bad deal: too much spill, too much batch verify, too little residency, thermal throttle mid-span, or a context length that changes the head's calibration.

Before you train a new draft head, check whether n_max=1 already clears a win. On the DeepSeek MTP rows, it did, with 100% first-token acceptance `[LAB: RESULTS-MATRIX §E]`. Training is expensive. Flag economics are cheap.

## Speculation and the promotion ledger

Write speculation into the same promotion ledger as quant:

- quality delta (suite ranges)
- speed delta (decode ranges)
- fit delta (headroom, failed flags)
- operational delta (new failure modes)

The Q8 master promotion is the template: quality first, MTP pays latency back, decode lands near old prod `[LAB: RESULTS-MATRIX headline before/after]`. If your ledger only has a green latency arrow, you will ship a faster wrong system.

## Teaching the inequality without math trauma

For operators who bounce off formulas, use the one-sentence form:

> Did each expensive read buy more accepted tokens than it cost in draft+verify traffic?

Walk the §E 14-spill row out loud: at n_max=1, yes; at n_max=3, no. The machine already did the arithmetic. Your job is to believe it.


## Field story: n_max=3 as peer pressure

In many chats, longer drafts sound stronger. The §E 14-spill row is the antidote: acceptance fell and end-to-end speed fell as n_max rose, into a loss at n_max=3 `[LAB: RESULTS-MATRIX §E]`. The peer-pressure move is to keep widening drafts until the graph looks aggressive. The operator move is to stop at the maximum of the inequality.

Put the losing cell in the runbook on purpose. Teams need permission to ship n_max=1 without feeling under-ambitious.

## Speculation under product load shapes

- **Short question, long answer chat:** decode-heavy; MTP can matter a lot.
- **Long prompt, short answer classification:** prefill-heavy; MTP may barely show up end-to-end.
- **Tool loops:** many short decodes; acceptance and tool-suite controls matter more than peak tok/s.

Bench the shape you sell, not the shape that flatters the feature.


## Operator lab: n_max sweep protocol

1. Fix recipe without MTP; capture 3 decode runs.  
2. Enable MTP n_max=1; capture decode + acceptance + tool suite.  
3. If win, try n_max=2; stop if end-to-end falls or acceptance falls hard.  
4. Never jump to n_max=3 on spilled MoE because a dense blog did.  
5. Re-run winner at production concurrency.  
6. Commit the winning n_max into the recipe file.

This is just §E turned into a checklist `[LAB: RESULTS-MATRIX §E]`.


## What you should do Monday

1. If MTP/spec is on, record n_max and mean acceptance beside tok/s.
2. Run the n_max sweep protocol on a staging host once, even if production "seems fine."
3. Re-bench the winner at production concurrency, not only c=1.
4. Put the losing §E-style cell (speedup less than 1) in the runbook as permission to stop.
5. Tie any speculation change to a product-suite control run.

Speculation that is not priced will still move your latency graph. Pricing it is the job `[LAB: RESULTS-MATRIX §E]`.


## Cross-links inside this book

If acceptance is high and speed still falls, suspect spill and verification traffic (chapter 1) or heat (chapter 7). If acceptance is low, do not start with training a new draft until n_max=1 and residency are checked. If enabling MTP coincides with weird multi-turn outputs, chapter 4's cache-reuse and KV precision traps are in scope. If the speedup vanishes at production c, you benched the wrong speed kind in chapter 1's terms.

## Acceptance without speed is a museum piece

A beautiful accept rate attached to a losing end-to-end latency is not a win. Ship the inequality, not the trophy metric.

## Looking ahead

Chapter 4 budgets the KV cache — another working set speculation and long context both
stress. Chapter 5 handles the measurement noise already visible in the tool controls.
Chapter 6 shows how "speculation is slow" often starts as a placement graph. Chapter 7
asks what happens to these recipes when the box hits thermal or power limits.

Speculative decoding is not magic. It is a leveraged bet on acceptance under a bandwidth
budget. Price the bet. Keep the receipt.
