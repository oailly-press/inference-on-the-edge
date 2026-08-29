# Chapter 2 — Quantization without Folklore

*(draft v0, 2026-08-28 — written by rogerai-dj for RogerAI Labs; unverified. Numbers
with a `[LAB:]` marker resolve into the lab record. Claims without one are labeled
unmeasured.)*

## The story everyone already knows

Quantization is supposed to be simple. Store the weights with fewer bits. The file
shrinks. The model fits. Maybe you lose a little quality. Maybe you do not notice.

That story is not entirely wrong. It is the right story for a first afternoon with a
dense 7B. It becomes folklore the moment you treat "Q4" as a moral category — as if
the label on the file were the same thing as the precision that actually mattered
inside the network.

This chapter is the lab's refusal of that folklore. On a mixture-of-experts model big
enough to hurt, **which tensors kept their precision** moved quality more than **how
small the file looked**. A community Q4 that was larger on disk lost to a master build
that preserved expert precision and used speculative decoding to pay the spill back.
A sideways requant into a friendlier-looking Q4 was aborted when the conversion log
showed the experts growing while quality had nowhere to go but down.

If chapter 1 said a token costs bytes moved, chapter 2 says: not all bytes are equal,
and the spreadsheet column titled "quant" is not a substitute for knowing which bytes
you kept.

## What quantization is, without romance

A trained weight is a number. Numbers can be stored at different precisions. Full
training might live in high precision. Inference often ships lower: 8-bit, 4-bit,
exotic typed formats, mixtures across layers.

Two bills arrive when you lower precision:

1. **A memory bill, usually smaller.** Fewer bits per weight means a smaller file and,
   if the weights stay resident, fewer bytes to stream per step.
2. **A quality bill, sometimes sharp.** Some parts of a network tolerate coarse
   quantization. Some do not. The damage is not evenly distributed just because the
   average bit-width looks neat.

Folklore collapses those bills into one sentence: "Q4 is fine." Engineering keeps them
separate: which tensors, which method, which calibration, which eval, which hardware
recipe.

This book will not teach every quant method. It will teach the reading habit that
survived contact with one hard MoE stack on one reference box.

## The reference comparison that broke the slogan

Hold the machine fixed (chapter 1's box). Look at DeepSeek-V4-Flash builds from the
capability table `[LAB: RESULTS-MATRIX §C]`:

| Build | Size | MMLU | Tool hardmode | Warm tok/s |
|---|---|---|---|---|
| UD-IQ3_XXS (old prod) | 102 GB | 79.6 | 47 | ~26 |
| community Q4_K_M-XL (teamblobfish) | 175 GB | 85.0 | 60 | 16.5 |
| Q3_K_M-MTP (lab, morning prod) | 143 GB | 84.0 | 40–50 | **30.5** @ MTP n=1 |
| **Q8-MTP master (lab, new prod)** | **149 GB** | **88.3** | mean **55** (43–73) | 26–27 @ MTP n=1 |

Read it slowly.

The community Q4 is the folklore champion on paper: "Q4," widely shared, 85.0 MMLU,
60 tool hardmode. It is also **175 GB**, slower on decode (16.5 tok/s), and not the
end of the story.

The lab master keeps experts at their **original release precision** (native MXFP4
passed through untouched in the conversion). It is **149 GB**, scores **88.3 MMLU**,
posts the best tool-scenario floor in the DeepSeek series on that harness, and still
lands at **26–27 tok/s** once MTP n_max=1 is on.

So the smaller-looking "more quantized" community artifact did not win. The build that
refused to mistreat the experts won on quality and, with speculation, tied old
production on speed.

That is not a vibe. It is a row.

## Expert precision is the MoE lever

Mixture-of-experts models do not spend all parameters on every token. They route each
token to a subset of experts. The router and the expert bodies are different kinds of
tissue. Smashing both with the same blunt quant policy is convenient for packaging and
often wrong for behavior.

The lab's tool-gap attribution series on DeepSeek hardmode made the lever visible
`[LAB: RESULTS-MATRIX §D]`:

| Intervention | Hardmode | Verdict |
|---|---|---|
| baseline IQ3 (generic parser) | 47 | baseline |
| chat-template patch → native parser | 43 | parser was **not** the bottleneck |
| **Q4 experts instead of IQ2_S** | **60** | **quant was most of the gap (+13)** |
| residual vs gpt-oss 73 | — | remaining ~13 looks like model gap |

Changing the parser did not fix tool use. Restoring expert precision did most of the
repair. The matrix's own ladder on tools reads: 2-bit ≈ Q3_K (~46) < Q4_K (60) ≈
native MXFP4 (~55, better scenario floor) `[LAB: RESULTS-MATRIX §C notes]`.

MMLU, meanwhile, recovered earlier than tools. Q3_K experts already sat near Q4 on
MMLU (84.0 vs 85.0) while tool use stayed in the IQ3 band (~40–50) until experts were
right. **Knowledge and tool-following did not share a single bit-width story.**

If you only watch a general knowledge sample, you can promote a quant that still
butchers the behavior your product actually needs. If you only watch a glossy file
size, you can reject a master that is both better and, under a sane recipe, fast
enough.

## Sideways requants: the conversion log as a stop sign

Folklore loves a sideways move: take a master, emit a Q4, ship the friendly label.
The lab tried the spirit of that move and aborted it for a boring, decisive reason.

On the record, a Q4_K_M requant attempt was stopped mid-run when the log showed
MXFP4→Q4_K conversion **growing expert tensors while adding requant loss**. The matrix
states the rule in operator English: requantizing below the master only makes sense to
**shrink** (Q3), never **sideways** (Q4) `[LAB: RESULTS-MATRIX headline before/after]`.

That sentence should be on a sticky note above every conversion job:

> If the conversion does not buy residence or bandwidth headroom, it is not a
> quant — it is vandalism with a progress bar.

Sideways requants are attractive because they match community naming. They are
dangerous because they can add loss without buying fit. The stop condition is in the
log, not in the marketing name of the output file.

## IQ3, Q3, Q4, Q8: labels are not a ladder of virtue

The same matrix teaches a second anti-folklore lesson: the alphabetical soup is not a
moral ladder.

- IQ3 old production: 102 GB, weakest MMLU of the DeepSeek set shown (79.6), fine as a
  historical baseline, not as an aspirational end state.
- Q3-MTP: 143 GB, MMLU 84.0, tools still noisy in the 40–50 band, but decode **30.5**
  tok/s with MTP n=1 — a speed/quality compromise that was real morning production.
- community Q4: 175 GB, strong headline tools (60), slower decode, not master quality.
- Q8 master: 149 GB, best MMLU, best DeepSeek tool floor on the harness, old-prod speed
  with MTP.

Notice that "higher Q number" did not monotonically mean "better" or "slower" or
"larger." The community Q4 is larger than the Q8 master and slower than the Q3-MTP
build. The only safe reading is per-row: size, quality suite, decode, recipe.

If your team sorts artifacts by the substring `Q4` versus `Q8` as if that were a total
order, you are sorting labels, not systems.

## Noise: the ±10 point tax before you brag

Before anyone turns a single hardmode number into a brand, read the footnote the matrix
carries like a scar `[LAB: RESULTS-MATRIX §C footnote]`.

Q3-MTP hardmode was measured three times on 07-13: 40, 47, and 50 (MTP off control).
Five of fifteen scenarios flipped between identical back-to-back runs. The harness sends
temperature 0.0. The flips were attributed to PAR=2 batch-packing nondeterminism
amplified by MoE routing, not to sampling temperature.

**Treat single-run hardmode numbers as ±10.** Conclusions that survived that noise:

1. Q3_K experts land near IQ3-level tool use, not near Q4's 60.
2. MTP speculation did not measurably harm tool use (MTP-off control within noise).
3. Some scenarios were consistent wins or losses across runs; the mean hides them if you
   only quote the mean.

Q8-MTP master's three runs (73 / 50 / 43, mean 55) likewise hide scenario-consistent
structure: TC-71 passed 3/3 after failing all five prior DeepSeek runs; TC-78 3/3;
TC-70 3/3.

Chapter 5 is the methodology chapter. Chapter 2 only needs the quant-facing moral:

> Do not promote a quant on a one-shot tool score. Demand a range, a control, and a
> scenario story.

## Dense models still quantize — they just fail differently

Not every row in §C is MoE. Qwen3.6-27B dense Q8_0 sits at 29 GB, 79.0 MMLU, 67 tool
hardmode, ~27 tok/s. Qwen3.6-35B-A3B Q8_K_XL sits at 38 GB, 71.0 MMLU, **87** tool
hardmode. gpt-oss-120b MXFP4 under vLLM TP=4 posts 71.0 MMLU, 73 tools, 60 tok/s
wall-clock `[LAB: RESULTS-MATRIX §C]`.

These rows exist here to prevent a false universal:

- **Dense Q8 can be "boring good"** — small enough to place, strong enough to use, not
  the subject of the expert-precision drama.
- **Tool rank and MMLU rank disagree across families.** The 35B-A3B line wins tools on
  this harness while losing MMLU to DeepSeek masters. Ranking quants by a single suite
  will reshuffle your heroes.
- **Engine identity is part of the quant story.** gpt-oss numbers above are vLLM TP=4,
  not llama.cpp. A quant comparison that silently changes engines is not a quant
  comparison.

If your deployment is a dense 7B–30B, you may never meet the MoE expert lever. You
still meet bytes, residence, and eval noise. Do not import MoE folklore into dense
stacks, or dense folklore into MoE stacks.

## Fit is a quant feature

Quantization that does not load is not a quant; it is a brick.

Section F of the matrix is the fit companion to §C `[LAB: RESULTS-MATRIX §F]`:

| Model | Working recipe | Failed configs |
|---|---|---|
| IQ3_XXS 102 GB | n-cpu-moe 4, ts 31,25,24,20 | — |
| blobfish Q4 175 GB | n-cpu-moe 24, ts 25,6,6,6, **--no-repack**, mmap, lean RAM | n-cpu-moe 14 VRAM-OOM; ≥18 without --no-repack segfault; --no-mmap host-OOM (>125 GB RAM) |
| Q8-MTP 160 GB | n-cpu-moe 14, ts 20,8,8,8 (+ --no-repack, mmap) | ts 21,8,8,7 → compute-buffer OOM card0 |
| Q3-MTP 143 GB | n-cpu-moe 10… / prod n-cpu-moe 11, ts 18,9,9,8, PAR 2 | — |

The community-shaped 175 GB Q4 does not merely "run slower." It runs only inside a
narrow recipe. Miss `--no-repack` and you can segfault. Miss mmap and you can host-OOM
beyond 125 GB RAM. Choose the wrong n-cpu-moe and you VRAM-OOM.

A quant card that omits the recipe is incomplete. A blog screenshot of file size is not
a recipe.

Chapter 8 returns to fit as a first-class problem. Chapter 2 needs only this coupling:
**precision choices change both quality and the placement surface.** The master that
preserved experts was not only a quality win; it was a different spill and flag story
than the 175 GB Q4.

## Promotion is a multi-objective decision

The lab did not promote the master because MMLU is sacred. The recorded rationale is
explicit: **tool quality over the last 4 tok/s** `[LAB: RESULTS-MATRIX headline
before/after]`. MTP n_max=1 recovered old-production speed. The master dominated the
community Q4 on the axes they cared about (+3.3 MMLU, comparable tools with a better
floor, +60% speed versus that Q4's 16.5, −26 GB).

That is what non-folklore promotion looks like:

1. Name the product-critical suite (here: tools, not only MMLU).
2. Measure a range, not a hero run.
3. Measure decode on the target engine recipe.
4. Measure fit and flags.
5. Accept an explicit trade (quality > last few tok/s) and write it down.

If your promotion story is "the Q4 is popular," you are doing release engineering by
folklore.

## What quantization cannot buy you

Plain boundaries:

- Quantization cannot fix a wrong engine placement. Chapter 1's 2 tok/s crater was not
  a quant problem.
- Quantization cannot invent eval honesty. ±10 points of harness noise remains ±10
  after you quant.
- Quantization cannot make a task the model cannot do into a task it can. Chapter 5
  and the manufacturing book's abstention work are about refusal and coverage; bits
  will not substitute.
- Quantization cannot repeal architecture. MoE routing pathologies and dense attention
  costs remain themselves at every bit-width.

If a pitch says "we Q4'd it, so it should be fine," ask: fine on which suite, which
range, which machine, which flags?

## Practical checklist for a quant decision

1. **Write the active tensors you care about.** For MoE, experts and router at minimum.
2. **Pick the suite that matches the product.** MMLU alone is not a tool product.
3. **Run at least three times** when the harness has known nondeterminism. Keep the
   range.
4. **Record decode and fit on the target box**, including failed flags.
5. **Reject sideways requants** that add loss without buying residence.
6. **Prefer masters that preserve fragile tissue**, then buy speed with speculation or
   placement — not with hopeful bit-chopping.
7. **Document the trade** you accepted (quality vs tok/s vs VRAM).



## A quant review board agenda

When someone proposes a new GGUF:

1. What product suite does it need to hold?
2. What is the range across ≥3 runs?
3. What is decode on the target engine?
4. What is the fit recipe and failed recipes?
5. What tissue changed precision (experts, attn, KV)?
6. Is this a shrink, a sideways move, or a master preserve?
7. Who owns the tombstone if it fails in a week?

If the proposer cannot answer 5 and 6, stop. That is how folklore enters — as an unnamed tissue change.

## Documenting expert policy

For MoE artifacts, keep a one-liner in the manifest of the service:

`EXPERT_PRECISION=native-MXFP4 preserved; router=...; attn=...`

The §D result that Q4 experts recovered tools is the reason this line exists `[LAB: RESULTS-MATRIX §D]`. If you cannot say what happened to experts, you do not know what you shipped.


## Field story: the sideways requant that did not ship

The aborted MXFP4→Q4_K attempt is easy to under-teach because it never became a hero row. That is why it matters. Most bad quants do not fail the demo. They fail the conversion log while still producing a file someone could have published `[LAB: RESULTS-MATRIX headline before/after]`.

A culture that only celebrates successful GGUFs will keep shipping sideways losses. A culture that files aborted conversions with reasons will not.

Add aborted experiments to the tombstone file with:

- source artifact
- tool and flags
- log line that triggered the stop
- who stopped it
- date

The master that did ship is partly protected by the requant that did not.

## What to say when someone asks for "just Q4"

Answer with questions:

1. Q4 of which tensors?
2. Compared to which master?
3. On which suite range?
4. On which machine recipe?
5. Does the conversion shrink bytes that were actually the bottleneck?

If they cannot answer, your job is to refuse the label until it becomes a recipe.


## Operator lab: read a GGUF like a bill of materials

When a new file arrives, do not only look at the filename's Q-tag. Inspect:

- total size
- whether experts are native precision or requantized
- attention/KV related tensors if exposed
- converter tool and version
- any log from the conversion job

The master versus community Q4 story is a bill-of-materials story: native experts versus a different tissue policy, different size, different speed, different suite range `[LAB: RESULTS-MATRIX §C/§D]`. Filenames compress that into a token like "Q4" that cannot carry the truth.

If your organization accepts GGUFs from chat links without conversion logs, you are consuming unmarked food.

## Suite pairing for quants

Always pair:

- one general suite (MMLU-class or your domain knowledge)
- one product behavior suite (tools, JSON schema, abstention)

§C shows why: ranks disagree across families and across quants `[LAB: RESULTS-MATRIX §C]`. A quant that only wins the suite you do not sell is a trophy, not a product.


## What you should do Monday

1. For every MoE artifact you serve, write one line on expert precision policy.
2. Refuse a new GGUF that arrives without conversion notes.
3. Pair your trophy suite with a product suite before any promote meeting.
4. Add aborted conversions to the tombstone file on purpose.
5. Re-check whether any "Q4 default" in your docs is actually a sideways requant.

The master versus community Q4 row is your teaching aid when someone argues labels over tissue `[LAB: RESULTS-MATRIX §C/§D]`.


## Cross-links inside this book

Quant changes that "should be faster" but are not are often chapter 1 placement or chapter 6 mmap/repack stories. Quant changes that are faster but flaky under tools are chapter 5 range problems. Quant changes that only load on sacred flags are chapter 8 fit problems. Speculative decoding can pay back a quality-preserving master's latency — that bridge is chapter 3, and it is not optional if you want the promotion ledger to close.


## Last word

Labels are not tissue. Preserve what hurts to lose; shrink what buys residence; never requant sideways out of peer pressure.

## Looking ahead

Chapter 3 takes the MTP rows seriously as economics: draft length, spill, acceptance,
and when speculation loses. Chapter 4 adds the KV cache as a second precision surface
people forget to budget. Chapter 5 formalizes the noise and controls already haunting
this chapter's footnotes. Chapter 6 shows how a "quant is slow" report often turns into
a load-log placement report under daylight.

The folklore version of quantization is a label. The engineering version is a claim
about **which numbers survived, on which tensors, under which recipe, on which
machine**, with a range attached.

Ship the second one.
