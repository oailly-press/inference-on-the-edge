# Chapter 1 — What a Token Costs

*(draft v0, 2026-08-28 — written by rogerai-dj for RogerAI Labs; unverified. Numbers
with a `[LAB:]` marker resolve into the lab record. Claims without one are labeled
unmeasured.)*

## The wrong unit

People talk about local inference in the units marketing handed them: parameters,
tokens per second, "a 70B on your desk." Those numbers are not fake. They are just the
wrong place to start. A parameter count tells you how large the weight file is. A
tok/s number tells you how the run felt on one machine, one day, one engine. Neither
tells you what the machine was actually paying.

The machine pays in **bytes moved**.

Every generated token is the result of reading a large weight tensor, combining it with
a smaller activation, and writing a little state forward. The arithmetic is cheap on
modern silicon. The memory movement is not. That is the single fact this chapter is
built on, and every later chapter is a consequence of it: quantization exists to move
fewer bytes; speculative decoding exists to make the bytes you do move buy more than
one token; the KV cache is a second working set you are also streaming; a bad load log
is almost always a placement story about which bytes landed on which bus.

If you already knew that, stay anyway. The chapter is not the slogan. It is the
measurement of what the slogan costs on one real box, and the habits that keep you from
lying to yourself with a tok/s number.

## The box this book measures against

Unless a sentence says otherwise, the numbers in this book come from one laboratory
machine:

- 4× RTX PRO 4500 Blackwell, 128 GB VRAM total
- Threadripper 9970X
- 128 GB DDR5-4800 host memory
- llama.cpp unless a row says vLLM

That envelope is not universal. It is a **reference**. A laptop will be slower. A
datacenter H100 node will be faster. A Raspberry Pi will be a different regime entirely,
and chapter 8 will say so without romance. The point of a reference is not to pretend
your hardware matches it. The point is that every comparison in the book is pinned to a
named machine, so a claim can be re-run or rejected.

Warm single-stream decode is the default tok/s figure. Prefill (prompt evaluation) is
called out when it matters. Aggregate throughput under concurrency is a different
number again, and chapter 3 will spend time on why confusing those three is how people
buy the wrong GPU.

## A tok/s is a receipt, not a personality

Take one model, hold it fixed, and change only the engine. The lab did that with a
102 GB UD-IQ3_XXS build of DeepSeek-V4-Flash:

| Build | Indexer | Warm decode |
|---|---|---|
| pre-#25545 era | CPU | ~2 tok/s |
| mainline CUDA | CPU | ~10.8 tok/s (bimodal) |
| taco build | CPU/disabled | 13.1 tok/s (bimodal) |
| pr25545 | **GPU** | **26.2 tok/s (24.5–28.5, stable)** |
| combined prototype | GPU | 28.4 tok/s (±0.04) |

`[LAB: RESULTS-MATRIX §A]`

Same weights. Same cards. Same host. Decode moved from a cratered ~2 tok/s to a stable
26 tok/s because the lightning indexer stopped living on the CPU. Prefill moved with
it: roughly 50–80 tok/s on CPU-indexer builds versus ~130 tok/s once the indexer was on
the GPU.

If tokens-per-second were a property of "the model," that table would be illegal. It
is not a property of the model. It is a property of **which bytes crossed which bus on
each step**. The early builds were not "bad at language." They were paying a host-side
tax on every token that no amount of prompt engineering could refund.

This is the first practical habit of the book:

> When a tok/s number is surprising, ask what moved, not what the model "is."

Chapter 6 will turn that habit into a load-log discipline. Here it is enough to notice
that a twelve-fold swing appeared before any quantization experiment, any speculative
decoding trick, or any change to the weights at all.

## Bandwidth as a budget you can feel

You do not need a cycle-accurate simulator to use the bandwidth idea. You need a
back-of-envelope that keeps you honest.

Suppose a decode step must touch roughly the active weights for the layers involved,
plus a slice of KV cache, plus overhead. The exact fraction depends on architecture
(dense versus mixture-of-experts), batch size, and how much of the model lives on GPU
versus host. What does not depend on those details is the shape of the limit:

**tokens per second cannot exceed (effective bytes per second) / (bytes per token).**

Raise effective bandwidth — better placement, fewer host round-trips, less spill — and
tok/s rises. Raise bytes per token — higher-precision weights, fatter KV, wider active
expert sets — and tok/s falls. Everything marketed as a "speed tip" is one of those two
moves in costume.

That is why a smaller file is not automatically faster. A 175 GB Q4 that spills hard
into host memory can lose to a 149 GB master that stays resident, even though the Q4
looks "more quantized" on a spreadsheet. The lab's promotion decision later in the
matrix is exactly that story: tool quality preferred over the last few tok/s, but only
after measuring that the master could still land at old-production speed with
speculative decoding paying back the spill `[LAB: RESULTS-MATRIX headline
before/after + §E]`. Chapter 2 takes the quality side. Chapter 3 takes the speculation
side. This chapter only needs the bandwidth reading: **residence beats folklore.**

## Three speeds people crush into one number

Local-inference conversations mash three different speeds together. Separate them or
you will misread every table in this book.

**Prefill (prompt eval).** The cost of reading the prompt and building the initial KV
cache. Dominated by large matrix multiplies over many tokens at once. On the reference
box, GPU-indexer builds prefilled the same DeepSeek IQ3 around 130 tok/s; CPU-indexer
builds sat nearer 50–80 `[LAB: RESULTS-MATRIX §A]`. Prefill cares about batchable work
and memory throughput into big GEMMs.

**Decode (single-stream).** The cost of producing the next token after the prompt is
in. Often memory-bandwidth bound because each step re-reads weights for a tiny amount
of arithmetic. This is the number people quote as "tok/s." On the same IQ3 model, the
promoted engine held 26.2 warm decode with a tight range (24.5–28.5) once the indexer
tax was gone `[LAB: RESULTS-MATRIX §A]`.

**Aggregate throughput under concurrency.** Several requests at once. Per-stream decode
usually drops; total tokens across streams may rise. On the same pr25545 IQ3 build with
PAR=4, c=1 measured 26.2 tok/s while c=4 measured 46.1 aggregate `[LAB: RESULTS-MATRIX
§B]`. That is not a contradiction. It is two different receipts.

If a vendor quotes "60 tok/s," ask which of the three they measured, on what batch, at
what context length, after what warm-up, and whether the range across runs was smaller
than the claim. Chapter 5 is about that honesty. Chapter 1 only installs the split.

## Same box, different models: the §C decode column

Hold the engine culture roughly fixed and look across models on the reference machine
`[LAB: RESULTS-MATRIX §C]`:

| Model / quant | Size | Warm tok/s | Notes |
|---|---|---|---|
| DeepSeek-V4-Flash IQ3 (old prod) | 102 GB | ~26 | baseline production |
| DeepSeek-V4-Flash community Q4 | 175 GB | 16.5 | bigger file, slower decode |
| DeepSeek-V4-Flash Q3-MTP | 143 GB | **30.5** @ MTP n=1 | speculation on |
| DeepSeek-V4-Flash Q8-MTP master | 149 GB | 26–27 @ MTP n=1 | quality ceiling, old speed |
| Qwen3.6-27B dense Q8_0 | 29 GB | ~27 | much smaller dense model |
| gpt-oss-120b MXFP4 (vLLM TP=4) | ~60 GB | 60 wall-clock | different engine |

Read it as bandwidth, not as a leaderboard.

The community Q4 is larger than the IQ3 and slower on decode. That is the bytes-per-token
term biting. The Q3-MTP build is larger than IQ3 and *faster* on decode because
speculative decoding changed how many tokens each weight read bought — not because Q3
magically streams better than IQ3 in isolation. The Q8 master is larger still and lands
back near the old ~26 tok/s once MTP n_max=1 is on: quality recovered without stranding
latency. The dense 27B Q8 sits near 27 tok/s at 29 GB: a different architecture with a
much smaller working set, in the same speed neighborhood as a heavily engineered MoE
stack an order of magnitude larger in file size.

None of those sentences require you to believe a vendor blog. They require you to
believe a table measured on one box and labeled with the recipes that produced it.

## Why "parameters" keep surviving

Parameter counts survive because they are easy. They fit on a slide. They sort a
Hugging Face page. They are also a decent proxy for *file size at a given precision*,
which is a decent proxy for *bytes you might have to move*, which is why they are not
useless.

They become harmful when they smuggle in assumptions:

1. **That all parameters are active on every token.** Mixture-of-experts models break
   this. Active parameters per token can be a fraction of total parameters; total still
   drives storage and often drives how much spill you accept to make the model fit.
2. **That precision is uniform and free.** It is not. Expert precision moved tool-use
   scores in the lab when total-bit folklore said the models should be close `[LAB:
   RESULTS-MATRIX §C/§D]`. Chapter 2 is the full argument.
3. **That two models with the same parameter count have the same decode cost.** Engine,
   placement, KV precision, batching, and spill can dominate.

Use parameters as a rough size class. Pay for tokens with bandwidth accounting.

## The 2 tok/s lesson, in slow motion

The cratered ~2 tok/s era on the reference box is worth sitting with, because it is the
purest bandwidth failure mode in the matrix.

Operators did what operators do: change GPU splits, poke batch flags, blame the model,
blame the quant, rerun benchmarks that all agreed the machine was "slow." The load path
was the problem. With the indexer on CPU, every token paid a host-side tax. The cards
were present. The weights were present. The bytes were taking the long road.

When the GPU indexer landed in pr25545, warm decode jumped to 26.2 and the range
tightened to something you could plan around (24.5–28.5). Prefill roughly doubled. No
romance, no new weights — a placement fix `[LAB: RESULTS-MATRIX §A]`.

Chapter 6 will teach you to read the load log for the modern versions of this failure:
layers on the wrong device, mmap surprises, repack flags, compute-buffer OOMs. Chapter 1
only needs the moral:

> A pathological tok/s is often a map of where bytes went. Tuning sampling parameters
> will not move that map.

## Concurrency without self-deception

Once single-stream decode is stable, someone will ask whether the box can serve more
than one user. Measure aggregate and per-stream separately.

On the promoted pr25545 IQ3 configuration, PAR=4 concurrency looked like this `[LAB:
RESULTS-MATRIX §B]`:

| c | tok/s (as reported in the matrix) |
|---|---|
| 1 | 26.2 |
| 2 | 16.9 |
| 3 | 24.7 |
| 4 | 46.1 aggregate |

There is a reproducible dip at c=2/c=3 with zero re-prefills logged — a scheduling
quirk the matrix still marks unresolved. That honesty matters more than a smooth curve.
A real system has seams. If your concurrency table is perfectly monotonic in a way your
engine has no right to produce, your methodology may be warmer than your claim.

A taco PAR=8 line in the same section reaches 65.7 aggregate at c=8. A vLLM TP=4
gpt-oss-120b reference climbs from 60.2 at c=1 to 888 at c=16. Those rows are not
"better hardware myths." They are different engines and different models on the same
reporting discipline. Steal the discipline, not a cross-row fantasy that every stack
should hit 888.

For capacity planning, the useful questions are:

1. What per-stream latency does a user still accept?
2. What aggregate tok/s does the box deliver at that concurrency?
3. What fails first — VRAM, host RAM, scheduler quirk, thermal throttle?

Question 3 is why chapter 7 exists. Question 1 and 2 are bandwidth accounting with a
queue attached.

## Speculative decoding is a bandwidth trade in advance

You do not need the full chapter 3 treatment to see the shape. Speculative decoding
asks a cheaper draft to propose tokens and a stronger path to verify them. When it
works, each heavy weight read buys more than one accepted token. When it fails, you
paid extra movement for rejects.

On the reference matrix `[LAB: RESULTS-MATRIX §E]`:

- A dense Qwen3.6-27B Q8 with stock MTP and zero spill reaches **2.2×** (27.0 → 59.3)
  at n_max=3.
- DeepSeek Q8-MTP with 14 layers spilled gets **1.18×** at n_max=1 with **100%**
  first-token acceptance, and slows down if you push n_max while spill stays high.
- DeepSeek Q3-MTP with 10 layers spilled gets **1.30×** at n_max=1, again at 100%
  first-token acceptance.
- The same DeepSeek Q8-MTP with 24 layers spilled falls below baseline at n_max=3
  (0.86×): speculation can lose.

The matrix states the law in one line: speedup grows as spill shrinks; on spill-bound
MoE, batch-verify costs pile into DDR5 expert reads, so n_max=1 is the sweet spot; the
head's training objective shows up as 100% first-token acceptance.

That is pure chapter-1 material. Speculation does not repeal bandwidth limits. It
changes the numerator: accepted tokens per expensive read. If your draft forces extra
spill traffic, you can invent a perpetual-motion engine on a whiteboard and still lose
on the bench.

## Quality and speed are coupled through the same budget

It is tempting to put "quality" in one document and "speed" in another. Real deployments
choose a point on a single curve.

The lab's production headline is the cleanest example `[LAB: RESULTS-MATRIX headline
before/after]`:

| | Old IQ3 | Q3-MTP | Q8-MTP master |
|---|---|---|---|
| MMLU | 79.6 | 84.0 | **88.3** |
| Tool hardmode | 43–47 | 40–50 | mean 55 (best floor) |
| Warm decode | ~26 | 30.5 | 26–27 |
| Size / spill | 102 GB / 4 layers | 143 GB / 10 | 149 GB / 14 |

The promotion rationale is on the record: tool quality over the last 4 tok/s. The
master kept native expert precision, speculative decoding paid the spill back, and the
box landed at old-production speed with a better quality floor. A sideways Q4 requant
was aborted when the log showed conversion growing expert tensors while adding loss —
requantizing below the master only made sense to shrink (Q3), never to move sideways
into a "friendlier" label.

Chapter 2 will unpack the quality mechanics. Chapter 1 only needs the coupling: **the
bytes you keep, the bytes you spill, and the tokens you accept per read are one
decision.** Pretending speed is a runtime flag and quality is a training flag is how
teams ship a impressive demo and an unusable service.

## What this chapter refuses to claim

Boundaries, in plain text:

- We do not claim the reference box is the only serious hardware class.
- We do not claim llama.cpp is always the right engine. The matrix itself includes a
  vLLM reference row because engines differ.
- We do not claim that 26 tok/s is "enough" for your product. Enough is a product
  requirement, not a physics constant.
- We do not claim a closed-form bytes-per-token formula for every architecture. The
  accounting is directional and measured; architecture-specific constants belong to
  the model card and the profiler, not to a motivational poster.
- We have not published a full roofline plot for every row in §C. Where this chapter
  speaks in roofline language, it is as an explanatory frame over measured tok/s and
  placement facts, not as a substitute for them.

If a sentence in this chapter sounds like it would survive without the lab tables, it
has gone too far.

## Working habits to take into the rest of the book

1. **Pin the machine.** Write down GPU, host RAM, engine, and build identity before you
   quote tok/s.
2. **Name the speed.** Prefill, single-stream decode, or aggregate under concurrency.
3. **When surprised, find the bus.** Host versus GPU, spill versus resident, indexer
   placement, KV reads — before temperature or prompt shape.
4. **Treat speculation as accepted-tokens-per-read.** If spill rises with draft length,
   you can lose.
5. **Refuse orphan numbers.** A tok/s without context length, warm/cold, and range is a
   rumor.



## A note on rooflines without theater

Hardware people draw rooflines: arithmetic intensity versus bandwidth ceilings. This book stays lower to the ground because the lab evidence is tok/s, placement, and spill, not a full published roofline campaign for every row. Still, the roofline moral applies: once you are bandwidth-bound, cleverer arithmetic does not save you, and once you are bound by host round-trips, more GPU FLOPS do not save you.

If you want a roofline study, do it on your stack and attach it the way this book attaches §A/§C. Do not sprinkle FLOPS marketing onto a 2 tok/s placement bug.

## Cost per million tokens, qualitatively

Operators eventually ask for dollars. This book will not invent cloud list prices. It will say:

- local cost is dominated by capex amortization, power, and engineering time
- waste is dominated by bad recipes (spilled verifies, CPU indexers, oversized context)
- a 12× decode bug is a 12× burn rate bug on the latency path

Fixing placement is often the highest-ROI "FinOps" move available to a local stack, and it shows up in the load log before it shows up in finance.


## Field story: the day the indexer moved

Before pr25545, the reference box could look fully configured and still feel broken. GPUs were present. The model file was present. Clients got answers. The answers arrived at a pace that made tool loops unusable. The team did what every team does: change sampler settings, blame MoE, blame quant, rerun the same dashboard.

The fix was not a new model. It was reading the engine table that became §A and noticing the indexer column `[LAB: RESULTS-MATRIX §A]`. Once the indexer lived on the GPU, warm decode jumped into the mid-20s and the range tightened enough to plan capacity. Prefill roughly doubled. The "model personality" narrative collapsed into a bus narrative.

If you take only one operational memory from chapter 1, take that sequence: surprising tok/s → placement hypothesis → measured recipe change → range re-check. Everything else in this book is a specialization of that loop.

## Teaching bandwidth to a mixed audience

Engineers who write CUDA and engineers who write product prompts need different doors into the same fact.

For systems people: show the §A table and the 12× swing.
For product people: show that a tool loop needing 20 model turns cannot tolerate 2 tok/s if humans are waiting on the loop.
For finance people: show that a placement bug multiplies cost without multiplying value.

Same physics. Three slides. No folklore.


## Operator lab: estimate before you bench

Before you run a new artifact, write a one-page estimate:

1. Artifact size on disk.
2. Expected residency (all GPU / split / heavy host).
3. Expected order-of-magnitude tok/s relative to a known row in §C.
4. Which chapter-1 failure class you will check first if wrong (CPU tax, spill commute, wrong speed metric).

Then bench. If reality is 2× off your estimate, you learned something about either the artifact or your mental model. Both are valuable. The point of estimates is not pride. It is to make surprises legible.

Compare any surprise to the §A swing: a 12× miss is placement until proven otherwise `[LAB: RESULTS-MATRIX §A]`. A 1.2× miss may be thermal, speculation, or context. Use the magnitude to choose the chapter.

## Glossary for chapter 1

- **Prefill** — prompt ingestion; builds initial KV.
- **Decode** — per-token generation after prefill.
- **Aggregate throughput** — total tokens/second across concurrent streams.
- **Spill** — layers or experts living off the fast device path.
- **Residence** — weights kept on the device path you intended.
- **Recipe** — the full flag+binary+artifact bundle that produced a number.

Use these words in tickets. Tickets that say only "slow" waste everyone's cache.


## What you should do Monday

1. Pin your engine build identity in the unit file and in every speed ticket.
2. Split your dashboards into prefill, single-stream decode, and aggregate-under-c.
3. Take one production model and write its bytes story: resident vs spilled, host vs GPU.
4. Re-read the last "model is slow" ticket and mark whether a bus was investigated.
5. Save one warm decode range (three runs) for the current production recipe as baseline.

If Monday ends without a baseline range, Tuesday's incident will invent one under pressure. The §A lesson is that baselines belong to recipes, not to model names `[LAB: RESULTS-MATRIX §A]`.


## Cross-links inside this book

When decode is cratered, go to chapter 6 before chapter 2. When decode is merely mediocre under concurrency, re-read the aggregate section here and then chapter 3's concurrency note. When someone answers a bandwidth problem with a new quant label, send them to chapter 2's sideways-requant stop sign and chapter 8's fit worksheet. When the graph looks good and the users do not, chapter 5's speed template probably asked a different question than production traffic.


## A closing arithmetic habit

When a vendor or a teammate quotes a single tok/s number, force it into a sentence:

> On hardware H, engine E, artifact A, flags F, context K, concurrency C, warm/cold W, the decode range was R across N runs.

If they cannot speak the sentence, they do not have a measurement. They have a mood. This book is a training manual for refusing moods with tables. The reference box rows are examples of sentences that can be audited `[LAB: RESULTS-MATRIX §A/§B/§C]`. Your job is to make your own fleet speak in the same grammar.


## Who this chapter is for

If you have ever stared at a tok/s graph and argued about models when you should have argued about buses, this chapter is the reset. If you are new to local serving, it is the map. If you are experienced, it is a checklist you can hand to the next hire so they do not repeat the 2 tok/s archaeology. The later chapters assume you accept the receipt model of speed: named machine, named recipe, named speed kind, named range. Reject that, and the rest of the book will look like taste. Accept it, and the lab tables become tools instead of trivia `[LAB: RESULTS-MATRIX §A/§C]`.

## Looking ahead

Chapter 2 stays on the same box and asks why expert precision moved quality when total
bits said it should not. Chapter 3 turns the MTP table into economics. Chapter 4 puts
the KV cache on the balance sheet as a second model. Chapter 5 attacks benchmark noise
directly — including the ±10 point tool-suite swing at temperature 0 that already
haunts the §C footnote `[LAB: RESULTS-MATRIX §C footnote]`. Chapter 6 teaches the load
log. Chapter 7 puts heat and power loss into the same accounting. Chapter 8 answers
what fits on a 128 GB class box, and what does not, without insulting smaller machines
by pretending they are broken 128 GB boxes.

The rest of the book is downstream of one sentence:

**A token costs bytes moved, under a recipe, on a named machine.**

Everything else is engineering on top of that receipt.
