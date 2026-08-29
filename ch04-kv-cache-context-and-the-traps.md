# Chapter 4 — KV Cache, Context, and the Traps

*(draft v0, 2026-08-28 — written by rogerai-dj for RogerAI Labs; unverified. Numbers
with a `[LAB:]` marker resolve into the lab record. Claims without one are labeled
unmeasured.)*

## The second model

People budget the weight file and forget the working set that grows with every token of
context: the **KV cache**.

Weights are the encyclopedia on the shelf. The KV cache is the desk where the model
keeps the conversation it is having *right now*. Desk space is not free. It scales with
layers, heads, head dimension, precision, batch, and sequence length. It can exceed the
weight footprint on long contexts. It can silently shrink when you raise concurrency. It
can corrupt outputs when precision or cache-reuse flags are wrong.

Chapter 1 said a token costs bytes moved. Chapter 4 says: some of those bytes are the
past you are re-reading every step. Treat the cache as a second model you are also
serving.

## What the cache is for

Autoregressive generation reuses attention keys and values from earlier tokens so the
model does not recompute the whole prompt on every step. That reuse is the KV cache.

Two operator facts follow:

1. **Prefill writes a lot of cache at once** (prompt evaluation).
2. **Decode appends a little cache per token** and reads a growing history.

If your latency pain is time-to-first-token on long prompts, you are often in prefill and
cache allocation. If your pain is tokens-per-second on long generations, you are often in
decode bandwidth against weights **and** against an ever-larger cache.

Confusing those pains produces the wrong fix: quantizing weights harder will not cure a
context budget you divided by `--parallel`, and buying another GPU will not cure a
cache-reuse corruption flag.

## Context is a budget, not a vibe

Engines expose a context ceiling (`n_ctx` and friends). That ceiling is RAM and VRAM
policy, not a personality trait of the model.

On the lab's production burn-in line, the promoted Q3-MTP recipe ran **PAR=2, 64K
ctx/slot** among other flags `[LAB: RESULTS-MATRIX §G]`. That is a deliberate allocation:
two slots, each with a large ceiling. It is also a reminder that context numbers in a
server log are **per-slot** after parallelism divides the pie.

The matrix records the trap in plain language during K3-Encode work `[LAB:
RESULTS-MATRIX H.4.2 / concurrency notes]`:

> `--parallel N` DIVIDES the context budget — `n_ctx_slot = CTX/N`. A first attempt at
> PAR=6 with CTX=8192 silently produced **1536 tokens per slot**, far too little for a
> model that spends 1200 tokens thinking. Size CTX as `PAR × per_slot_need`, and read
> `n_ctx_slot` in the startup log to confirm.

That is one of the highest-value sentences in this book. Silent undersizing does not
error loudly. It thinks in a closet and then loops, truncates, or looks "dumb."

**Habit:** after every parallel or context flag change, read `n_ctx_slot` (or equivalent)
from the startup log before you trust a single quality sample.

## Parallelism confounds: do not compare unequal desks

When the lab first compared PAR=1 and PAR=4 behavior, unequal per-slot context
confounded the story. Re-running with matched `n_ctx_slot` fixed the science `[LAB:
RESULTS-MATRIX concurrency / control notes]`:

| Server | slots | n_ctx_slot | control-fact | control-code |
|---|---|---|---|---|
| PAR=1, CTX=16384 | 1 | 16384 | answered | answered |
| PAR=1, CTX=4096 | 1 | 4096 | answered | answered |
| PAR=4, CTX=16384 | 4 | 4096 | **loops** | loops at 4-in-flight |

The point is not that PAR=4 is cursed. The point is that a fair comparison holds the desk
size fixed. Earlier rows that mixed PAR=1 at 16384 with PAR=4 at 4096 per slot were
comparing different products.

Chapter 5 will call this a control. Chapter 4 calls it furniture: **if the desks differ,
the exam results are not about the student alone.**

## Precision: the cache has a bit-width too

Weights are quantized in public. Cache precision is often a quiet flag (`f16`, `q8_0`,
etc.) that still moves both memory and behavior.

The outline for this book flagged a hard lab lesson in one line: **q8_0 KV corrupting
V4 output; f16 discipline.** The operational stance that survived is conservative on the
cache when quality is on the line: prefer a known-good KV precision for the architecture
you are serving, and treat "more quantized KV" as an experiment with a product suite
attached, not as a free default.

You will also see recipes that pin f16 KV explicitly beside other carefully chosen flags
— for example a Kimi-K3 streaming bring-up line that lists `f16 KV` next to `--no-repack`,
mmap, and `--cache-reuse 0` `[LAB: RESULTS-MATRIX H bring-up flags]`. That is not
decoration. It is a stack of foot-guns with the safeties written in the on position.

If you change KV precision, change **one variable**, keep context and parallel fixed, and
run the same honesty suite you use for weight quants (chapter 2 and chapter 5).

## Cache reuse: speed feature, corruption feature

Prefix cache reuse can avoid re-prefilling shared prompt prefixes. It can also corrupt.

During K3-Encode work the lab recorded a hard requirement: **`--cache-reuse 0` is
mandatory (KDA prefix-cache corruption, PR #26185)** `[LAB: PROJECT-LOG K3-Encode /
cache-reuse notes]`. The flag that looks like free prefill is sometimes a correctness
regression with a delayed fuse.

Rule:

1. Default to reuse off until you have a harness that would catch the corruption mode.
2. When you enable reuse, pin engine version and architecture notes; do not inherit reuse
   across unrelated models because a blog said it was faster.
3. If outputs get weird only on multi-turn or shared-system-prompt traffic, suspect reuse
   before you suspect "the quant is bad."

## KV cost as a first-class design axis (hybrid lesson)

Not every architecture pays the same KV tax. In from-scratch hybrid experiments, the lab
measured bits/byte against KV cost per token and found ordering that was **monotonic in
KV cost at every length** on the reported arms, with a flat-KV hybrid winning on the
combined reading `[LAB: PROJECT-LOG / matrix hybrid KV arms]`.

| arm (sketch) | KV cost class | role |
|---|---|---|
| full attention control | highest KB/token | baseline desk tax |
| hybrids with more full attention | medium | compromise |
| hybrid / flat-KV leaning arms | low or flat | desk tax collapses |

The detailed bits/byte numbers and σ claims live in the lab record; the operator lesson
does not need them memorized. **Architecture choice is sometimes a KV choice.** If your
product is long-context, a model that is slightly worse on a short-context leaderboard
but dramatically cheaper per token of history can win the deployment. If your product is
short prompts, you may be buying flat-KV complexity you will never amortize.

Chapter 8's "what fits" question is partly a KV question once context targets leave the
demo range.

## Long context is a residency plan

A 64K per-slot ceiling is not only an engine number. It is a claim about memory headroom
under real traffic. Production soak on Q3-MTP included 28K-token long-context recall
checks beside tool-calling and dual-stream MTP `[LAB: RESULTS-MATRIX §G]`. That is the
right posture: long context is a **feature you test**, not a slider you max.

When headroom dies, systems start paging, shrinking batches, or OOMing on the next spike.
Chapter 7 will talk about thermal and power. Here the cache-specific failure is quieter:
quality falls first, then speed, then the process dies — and the root cause is that the
desk ate the room.

## Traps, in one checklist

1. **Parallel divides context.** Always compute and log per-slot context.
2. **Unequal desks confound quality comparisons.** Match `n_ctx_slot` before blaming PAR.
3. **KV precision is a quant.** Experiment with suites; do not casual-toggle.
4. **Cache reuse can corrupt.** Require a harness before enabling.
5. **Long context without soak tests is a demo.** Prove recall and stability at target
   length.
6. **Architecture KV tax differs.** Leaderboards at 2K tokens hide 32K economics.
7. **Spill + long context + speculation stack.** Each multiplies bytes; price the stack
   (chapters 1 and 3).

## A minimal measurement recipe

For any new model or flag set:

1. Print startup allocation: weights, KV budget, per-slot context, parallel.
2. Run a short fixed prompt at c=1; capture tok/s and smoke quality.
3. Run a long-prompt prefill case at target context; capture time-to-first-token.
4. Run a long-generation case; watch VRAM/host trend, not only mean tok/s.
5. If enabling parallel, repeat with matched per-slot context and with production-like
   concurrency separately.
6. If enabling cache reuse or KV quant, run a multi-turn / shared-prefix suite designed to
   fail loudly on corruption.

If you cannot state the per-slot context and KV precision of a "bad output" report, you
do not yet have a bug report. You have a mood.

## What this chapter refuses to claim

- We do not claim one universal KV precision for all models.
- We do not claim cache reuse is always unsafe — only that it has been mandatory-off for
  recorded corruption modes on specific stacks.
- We do not claim flat-KV hybrids always win products; they win a measured tax curve in
  the lab record under stated conditions.
- We do not provide a closed-form KV byte formula for every architecture here; use the
  engine's own allocation log and the model card.



## Context length as a product feature with a bill

Product managers love "128k context" on a slide. The bill arrives as:

- KV bytes per token times layers times precision
- prefill time to first token
- decode slowdown as history grows
- sharper failure modes under parallel

If the customer actually uses 2k tokens median, you may be paying a tax for a brochure line. If they truly use 32k, then chapter 1's bandwidth story and this chapter's desk story dominate the design. Measure the real context histogram before you buy the marketing ceiling.

## Prefix-heavy workloads

Many enterprise deployments share a large system prompt or tool schema across requests. That is exactly where cache reuse is tempting and exactly where corruption flags matter. The K3-Encode note that forced `--cache-reuse 0` is your reminder that shared prefixes are not free speed `[LAB: PROJECT-LOG cache-reuse / PR #26185]`.

A safe rollout pattern:

1. ship with reuse off
2. build a multi-turn shared-prefix corruption suite
3. enable reuse on a canary
4. compare suite + latency
5. only then widen

Skipping to step 5 is how silent wrongness enters.

## Conversation durability vs KV durability

Users think conversations live in the model. Usually they live in a client-side or app-side transcript, and the KV is a disposable acceleration of that transcript. After a restart, the desk is empty even if the chat scrollback still shows text.

Design implications:

- reconnect should resend necessary history or accept cold quality
- do not promise "the model remembers" across process death unless you built durable state (see *Durable State for Ephemeral Minds* if you need that stack)
- load-shed by dropping KV and re-prefilling rather than serving half-corrupt desks

## Per-slot math drill

Suppose CTX=8192 and PAR=8. Per slot 1024. If your agent prompt uses 700 tokens of tools+policy and the user question is 200, you have ~124 tokens of generation before you are in trouble. This is not theoretical; the matrix's 1536-slot caution is the same class of foot-gun `[LAB: RESULTS-MATRIX concurrency notes]`.

Always compute:

`usable_generation ≈ n_ctx_slot - prompt_tokens - safety_margin`

If usable_generation is smaller than your product's median answer, your parallel setting is a quality bug.


## Field story: the silent 1536-slot deploy

The matrix note about PAR=6 with CTX=8192 producing 1536 tokens per slot is a complete short story `[LAB: RESULTS-MATRIX concurrency notes]`. Nobody intends to ship a closet-sized desk. The flags look reasonable in isolation. The division does the damage.

Add a CI check if you can: parse startup logs and fail if `n_ctx_slot < product_min_context`. If you cannot CI it, put it in the smoke script. If you cannot smoke it, you will learn from users.

## KV and multi-tenant fairness

When multiple tenants share a server, KV is the fairness battleground. One tenant with a huge context can crowd out others even if weight residency is fine. Per-slot caps, per-tenant max context, and admission control are fit tools as much as they are product tools.

Chapter 8's headroom targets apply to KV too: if the box only works when nobody uses long context, you did not ship long context. You shipped a brochure.


## Operator lab: context admission test

Write a script that:

1. Parses n_ctx_slot from startup.  
2. Builds a prompt of size `n_ctx_slot - 256`.  
3. Asks for a 512-token answer.  
4. Fails if the server loops, truncates immediately, or OOMs.

Run it whenever PAR or CTX changes. The silent 1536-slot failure mode should never reach users twice `[LAB: RESULTS-MATRIX concurrency notes]`.


## What you should do Monday

1. Parse `n_ctx_slot` from every server startup and alert if below product minimum.
2. Confirm KV precision and cache-reuse settings are explicit in the recipe file.
3. Run one matched-desk parallel control before blaming PAR for quality.
4. Measure a real context-length histogram from production logs if you have them.
5. Add a shared-prefix multi-turn case to the smoke suite before enabling reuse.

The silent 1536-slot failure mode is too cheap to leave unguarded `[LAB: RESULTS-MATRIX concurrency notes]`.


## Cross-links inside this book

Context division bugs present as "model quality" and get handed to chapter 2 or 5 by mistake. Always read n_ctx_slot first (chapter 6 audit). Long context without headroom becomes a chapter 8 fit failure mid-week, not at load time. Speculation plus long context multiplies bytes; re-price chapter 3 after any CTX change. Crash recovery rarely restores KV; chapter 7's client honesty section matters for user-visible continuity.

## Desk space is product space

If the desk does not fit the work, the model never gets a fair exam.

## Looking ahead

Chapter 5 turns the honesty problem into a method: error bars, controls, and the ±10
point tool-suite noise already stalking chapters 2 and 3. Chapter 6 stays in the startup
and load logs where context division and placement show up before users do. Chapter 8
asks what fits when weights, KV, concurrent slots, and headroom must cohabit on a 128 GB
class box.

You are never serving only a weight file. You are serving a weight file **and** a growing
desk. Budget both, or the desk will budget itself.
