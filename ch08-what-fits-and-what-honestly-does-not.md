# Chapter 8 — What Fits, and What Honestly Does Not

*(draft v0, 2026-08-28 — written by rogerai-dj for RogerAI Labs; unverified. Numbers
with a `[LAB:]` marker resolve into the lab record. Claims without one are labeled
unmeasured.)*

## Fit is the product

A model that does not load is not a model. A model that loads only on a flag combination
you cannot remember is not a deployment. A model that loads on a 128 GB box and is sold
as "edge" for a Pi without a recipe is marketing.

This chapter is a fit map for the reference class used throughout the book, plus an
honest lower bound on what smaller machines can claim. It is also the place where the
book keeps a hard linguistic promise: **Pico is not an MCU**, and microcontroller-class
hardware is not a failed GPU box.

## The 128 GB VRAM class: working recipes

On the reference machine (4× RTX PRO 4500 Blackwell, 128 GB VRAM, 128 GB host RAM), the
matrix's fit table is the shopping list `[LAB: RESULTS-MATRIX §F]`:

| Artifact | Working recipe | Failure modes if you miss |
|---|---|---|
| DeepSeek IQ3 102 GB | n-cpu-moe 4; split 31,25,24,20 | — |
| community Q4 175 GB | n-cpu-moe 24; split 25,6,6,6; `--no-repack`; mmap; lean host RAM | VRAM-OOM; segfault; host-OOM >125 GB |
| Q8-MTP ~160 GB | n-cpu-moe 14; split 20,8,8,8; no-repack; mmap | compute-buffer OOM on "close" splits |
| Q3-MTP 143 GB | n-cpu-moe 10–11; prod split 18,9,9,8; PAR 2 | — |

Production soak on Q3-MTP added the live constraints: PAR=2, 64K ctx/slot, MTP n_max=1,
headroom 3.3–6.6 GB/card, 28K long-context recall checked `[LAB: RESULTS-MATRIX §G]`.

**Fit means recipe + headroom + a soak**, not a green load banner.

## What the same class should not pretend

Even on 128 GB VRAM:

- A 175 GB Q4 is a **conditional** citizen, not a casual default.
- Sideways requants that grow experts are not a path to fit (chapter 2).
- Long context × high parallel × heavy spill × aggressive MTP can un-fit a model that
  "loaded fine" at c=1 (chapters 3 and 4).
- vLLM versus llama.cpp changes the fit surface; engine is part of the recipe (chapter
  5).

If a vendor says "runs on 128 GB" without flags, context, and concurrency, they said
almost nothing.

## Dense versus MoE fit intuition

From §C, a dense Qwen3.6-27B Q8_0 at 29 GB is a different class of object than a 149 GB
MoE master `[LAB: RESULTS-MATRIX §C]`. It fits more places, fails differently, and still
does not repeal KV budgets at long context.

Use dense small models when:

- you need simple residency
- your quality bar matches their suite
- you want fewer placement foot-guns

Use large MoE masters when:

- the product suite needs them
- you can staff the recipe (splits, offload, MTP)
- you can pay the operational complexity

Do not sneak a MoE master into a dense-shaped runbook.

## Below the GPU: honesty without cosplay

Smaller hardware is real. It is not a moral failure. It is a different envelope.

**Prosumer single-GPU boxes.** A 24–48 GB card can host smaller dense models and some
quantized medium models with tight context. It will not host the 149 GB master recipe
from §F. Do not quote this book's DeepSeek production rows as if they transfer.

**CPU-heavy or unified-memory machines.** Possible for smaller models and serious
prompt-eval patience. Bandwidth accounting from chapter 1 still rules; expect decode far
below the reference GPU table unless the model is small enough to stay hot in memory.

**Raspberry Pi–class boards.** Good for tiny models, controllers, gateways, and demo
chat with constrained expectations. Bad for pretending a plant-floor MoE lives there.
Measure tok/s and context honestly; publish DNF when appropriate (chapter 5).

**Microcontrollers / MCU class.** Hard real-time controllers, DSP-ish budgets, kilobytes
to megabytes of memory. They run control firmware and carefully designed tiny-ML, not
general LLM decode stacks from this book's tables.

**Pico ≠ MCU.** The lab's "Pico" naming in model work refers to a small language-model
tier / product line in the Wave stack, **not** a Raspberry Pi Pico microcontroller. This
book will not claim microcontroller deployment for LLM inference on the strength of a
model nickname. If a sentence ever seems to blur those, prefer the stricter reading:
microcontrollers are out of scope for the MoE recipes measured here.

## A fit decision worksheet

1. Target hardware: VRAM, host RAM, CPU, power, cooling.
2. Target context and concurrency.
3. Product suite (tools? knowledge? abstention?).
4. Candidate artifacts with **measured** sizes.
5. Engine identity.
6. Working flags from a real load log.
7. Failed flags (keep the tombstones).
8. Soak result at duty cycle.
9. Recovery plan (chapter 7).
10. Go / no-go with the trade named.

If you cannot fill rows 6–8, you are still in demo land.

## Connecting the whole book

- **Chapter 1** — if it fits but crawls, find the bus.
- **Chapter 2** — if it fits only by wrecking experts, it does not fit your product.
- **Chapter 3** — if it fits only at n_max that loses, price speculation again.
- **Chapter 4** — if it fits weights but not desks, shrink parallel or context.
- **Chapter 5** — if you cannot measure fit failures reproducibly, fix the instrument.
- **Chapter 6** — the load log is the fit oracle.
- **Chapter 7** — fit at time zero is not fit after heat and outages.

## What this chapter refuses to claim

- We do not publish a universal SKU list for 2027 hardware.
- We do not claim Pi-class boards are useless — only that they are not 128 GB VRAM.
- We do not claim MCUs will "run LLMs soon enough" as a substitute for engineering.
- We do not claim the reference recipes are optimal — only that they worked and failed in
  the recorded ways.


## Reading §C as a fit catalog, not a trophy case

The capability table is also a fit preview `[LAB: RESULTS-MATRIX §C]`:

- ~29–38 GB dense/quant rows are "many machines" objects.
- ~60 GB class objects need serious prosumer or multi-GPU recipes.
- ~100–175 GB MoE rows are reference-class citizens with explicit spill discipline.

Promote models down this list only when the product suite forces you up. The cost is not only money. It is operational surface area: more flags, more ways to mis-place, more ways to lie with a single tok/s number.

## Headroom targets

§G's 3.3–6.6 GB per-card headroom on a working production recipe is a qualitative guide: leave room `[LAB: RESULTS-MATRIX §G]`. Exact targets depend on concurrency and context. A practical stance:

- if headroom is <2 GB/card under production PAR and ctx, you are one feature flag from pain
- if host RAM is near full with mmap recipes, you are one leak from OOM
- if you need perfect packing to load, you do not have a spare for diagnostics

Fit without headroom is a screenshot.

## Tombstones you should keep forever

Failed configs are assets. Keep a `FAILED-RECIPES.md` next to the service:

- flags
- error line
- date
- engine version
- who reproduced

§F already models this culture (segfault, host-OOM, compute-buffer OOM) `[LAB: RESULTS-MATRIX §F]`. Teams that delete failure notes rebuy the same outage.

## Smaller machines: sample honest claims

Honest claim patterns:

- "7B-class Q4, 4k context, ~N tok/s on hardware H, suite S range R."
- "Does not run 70B+ MoE masters; DNF on artifact set A."
- "Pi-class board: assistant demos only; not a multi-user production target."

Dishonest claim patterns:

- "Runs GPT-class models" without names, quants, or tok/s.
- "Edge" as a synonym for "we quanted it until the demo fit."
- "MCU LLM" without defining whether you mean firmware-class tiny models or chat models.

## Pico naming discipline (again, harder)

In this repository universe, Pico-sized language models and Raspberry Pi Pico microcontrollers are different species. The first is a model tier. The second is a microcontroller board. This book measures LLM inference recipes on GPU-class and discusses smaller boards carefully. It does **not** authorize a sentence like "our Pico runs on an MCU" as if those words shared a referent. If you need MCU inference, start a different measurement program; do not borrow this book's MoE tables.

## The last checklist before you call it "edge"

1. Named hardware.
2. Named artifact + engine + flags.
3. Load log archived.
4. Speed range under production concurrency.
5. Quality range on the product suite.
6. Soak long enough to see drift and heat.
7. Recovery drill done once.
8. Failed recipes listed.
9. Claims matched to the envelope (no Pi cosplay as 128 GB).
10. Someone human can explain the recipe without the original author in the room.

If item 10 fails, the deployment is still a research demo — even if it is in production.


## Multi-GPU fit is still fit

"Fits on 128 GB VRAM" might mean four cards. Topology matters: NVLink vs PCIe, split
strategy, whether card0 holds disproportionate compute buffers (the §F card0 OOM is the
cautionary tale) `[LAB: RESULTS-MATRIX §F]`. A model that fits on an idealized fully
connected fabric may not fit on your PCIe bifurcation.

When you change machines, re-run fit even if the VRAM sum matches. Sums are not graphs.

## The manufacturing book parallel

If you also read *Local LLMs for Manufacturing*, notice the shared ethic: local is a
constraint that creates engineering, not a discount SKU. Fit on the plant floor includes
air-gaps, custody, and recovery. Fit on the inference box includes spill, KV, and power.
The same honesty rule applies: do not claim a deployment shape you have not soaked.

## What to tell a buyer

If you are the one writing a proposal, the honest paragraph looks like this:

> On hardware H, artifact A under engine E with flags F loads with G GB headroom, serves
> concurrency C at context K with decode range R tok/s, quality suite S range Q, soak
> duration T with drift D. Failed recipes listed in appendix. Recovery drill RTO measured
> once at M minutes.

Anything much shorter is a brochure. Brochures are fine if labeled brochures. They are
not runbooks.

## Open measurements this book does not pretend to close

- full roofline plots per architecture
- exhaustive Pi and mobile SoC tables
- MCU-class LLM budgets
- every speculative method beyond the recorded MTP rows
- multi-week thermal wear studies

Where those matter to your product, measure them. The point of this book is to make the
missing measurements obvious, not to invent them.




## Fit matrix template for your fleet

Copy this into the repo of every production model:

| Artifact | HW class | Engine | Flags hash | Load | Decode range | Suite range | Soak | Owner |
|---|---|---|---|---|---|---|---|---|
|  |  |  |  |  |  |  |  |  |

Empty cells mean "not a product yet."

## Why "edge" needs a definition per sentence

Edge can mean:

- on-prem VM
- factory server room
- rugged PC on a line
- laptop
- phone
- MCU

This book used edge to mean **hardware you own, near the work, outside a rented frontier API**. It did not mean "tiny silicon." When you use the word, attach a hardware class. Otherwise you will inherit someone else's fantasy envelope.

## Connecting to SQLite-for-agents and manufacturing

Statefulness, custody, and recovery show up in sibling books. Fit is the inference-shaped slice of the same honesty. If you cannot host the model, the rest of the stack is theater. If you can host it only by lying about tok/s or suite ranges, the theater is just more expensive.

## Final operator brief

Before go-live:

1. Recipe file committed.
2. Failed recipes committed.
3. Load log sample committed.
4. Suite ranges committed.
5. Soak notes committed.
6. Recovery drill minutes committed.
7. Capacity owner named.
8. Hardware class named without cosplay.

When those eight exist, you do not need bravado. You have a system.


## Field story: the 128 GB demo that was a 4 GB headroom prayer

§G's headroom band exists so you do not confuse "it loaded" with "it will survive Monday" `[LAB: RESULTS-MATRIX §G]`. The demo that loads with crumbs of free VRAM is a party trick. Production needs slack for concurrent spikes, longer contexts, and the next engineer debugging with an extra process.

If leadership wants the demo as production, show them the headroom number and the §F tombstones. Make the risk concrete.

## Portfolio fit

A real fleet usually needs more than one artifact:

- small dense for cheap bulk
- medium for default chat
- large MoE master for hard cases

Route between them with honesty, not ego. Fit is easier when not everything needs the master. Chapter 2's quality ladder and chapter 5's suite ownership make that routing measurable.


## Operator lab: portfolio routing sketch

Define three routes:

- **cheap** — small dense, strict schema, bulk traffic
- **default** — medium model, general chat
- **hard** — MoE master, tools, long context, humans waiting

Measure each route's fit envelope separately. The failure mode is routing everything to hard because it scores best on a trophy suite. That is how you turn a 128 GB box into a single-tenant luxury good.

Use chapter 5's suite ownership to decide promote/demote between routes. Use chapter 6's recipe registry to keep each route boring.

## Hardware class cards (fill in for your fleet)

**Class A — reference multi-GPU 128 GB.** Can host §F MoE recipes with headroom if flags are right.  
**Class B — single 24–48 GB.** Dense and medium quants; DNF large MoE masters.  
**Class C — unified memory / workstation.** Case-by-case; bandwidth often the surprise.  
**Class D — Pi-class.** Demo and controller adjacent; not multi-user MoE.  
**Class E — MCU.** Out of scope for this book's LLM recipes.

Put every host into a class before you assign artifacts. Class mismatch is the root failure behind many "edge" disappointments.


## What you should do Monday

1. Classify every host into a hardware class card before assigning artifacts.
2. Fill one fleet fit matrix row for each production model (including blanks).
3. Publish DNF lists for artifacts that will not be attempted on small classes.
4. Enforce Pico≠MCU language in internal docs and customer decks.
5. Demand headroom numbers in every "it fits" claim.

Fit is the gate that keeps the rest of this book honest. Without it, bandwidth essays become cosplay `[LAB: RESULTS-MATRIX §F/§G]`.


## Cross-links inside this book

Fit is where the book becomes a fleet policy. Chapter 2 decides which artifacts deserve a row. Chapter 6 keeps rows honest at deploy time. Chapter 7 keeps rows honest after Wednesday. Chapter 5 prevents trophy suites from assigning masters to every route. If a route cannot fill the fit matrix, it is not a route yet.


## After the matrix, the calendar

Fit is not only a row in a table. It is a calendar:

- weekly recipe drift check
- monthly black-start drill
- quarterly suite recalibration
- every engine upgrade as a mini re-fit

The lab's crash recoveries and soak notes only help if your organization has a cadence that reads them into action `[LAB: PROJECT-LOG 2026-08-22/24; RESULTS-MATRIX §G]`. A static PDF of this book without that cadence becomes another shelf ornament. Use the Monday checklists. Put them on a real calendar.


## One-page fit pledge

No "runs on edge" sentence without:

- hardware class
- artifact id
- recipe hash
- headroom
- DNF list for the classes below it

If the sentence cannot carry those five, it is cosplay. This book ends where the cosplay ends `[LAB: RESULTS-MATRIX §F/§G]`.


## Last word

Measure the box you have. Name the recipe you ship. Keep the failures visible. That is fit; that is edge; that is the whole job.

## Closing the book

Local inference becomes real when three receipts match:

1. **bytes** (bandwidth accounting),
2. **recipes** (flags, placement, speculation, cache),
3. **honesty** (ranges, controls, crash drills, fit tombstones).

The edge is not a place where physics sleeps. It is where physics is close enough to feel
with your hands — on a named box, with a log, under a budget you can explain.

If you carry only one sentence out of these eight chapters, carry this:

**Ship the recipe you measured, on the machine you named, with the failures left visible.**
