# Chapter 6 — The Load Log Tells the Truth

*(draft v0, 2026-08-28 — written by rogerai-dj for RogerAI Labs; unverified. Numbers
with a `[LAB:]` marker resolve into the lab record. Claims without one are labeled
unmeasured.)*

## When the knobs lie

A model that should do ~26 tok/s does ~2. The chat is fine. The GPU utility graph is a
Rorschach test. Someone lowers temperature. Someone raises batch. Someone rebuilds with
"performance flags." Someone blames the quant.

The load log already knows.

Pathological decode is usually not a personality problem and not a prompt problem. It is
a **placement problem**: which tensors landed on which device, which layers spilled, which
indexer runs where, which flags turned a resident recipe into a host-memory commute. The
startup log and the per-layer placement summary are the primary sources. Sampling flags
are secondary literature.

## The pure case: 2 tok/s with the weights right there

Chapter 1 already showed the cleanest load-log story in the matrix. Same 102 GB UD-IQ3
DeepSeek file, same cards, different engine builds `[LAB: RESULTS-MATRIX §A]`:

| Build | Indexer | Warm decode |
|---|---|---|
| pre-#25545 | CPU | ~2 tok/s |
| mainline CUDA | CPU | ~10.8 bimodal |
| taco | CPU/disabled | 13.1 bimodal |
| **pr25545** | **GPU** | **26.2 stable** |

The weights were not "slow." The lightning indexer was on the CPU. Every token paid a
host-side tax. No amount of prompt craft refunds that tax. Reading the build identity and
the indexer placement ends the mystery; turning knobs extends it.

**First rule of this chapter:** when tok/s collapses by an order of magnitude, look for a
bus, not a better temperature.

## What to read before you tune

A useful startup sequence answers:

1. **Engine identity** — build SHA / binary name / backend.
2. **Model identity** — path, quant tag, size on disk.
3. **Device map** — which GPUs, tensor split, n-cpu-moe / offload counts.
4. **Per-slot context** after parallel division (chapter 4).
5. **KV precision and cache-reuse flags.**
6. **mmap / repack / no-mmap choices.**
7. **Any warning about fallbacks** (CPU layers, failed CUDA graphs, OOM recoveries).

If your server prints a placement table or layer device list, archive it with the run. If
it does not, improve the logging before you improve the model. A regression without a
placement snapshot is a ghost story.

## Fit failures are load-log failures

Section F is a catalog of recipes that look like "model issues" and are actually flag
issues `[LAB: RESULTS-MATRIX §F]`:

| Model | Works when… | Fails when… |
|---|---|---|
| IQ3 102 GB | n-cpu-moe 4, split 31,25,24,20 | — |
| community Q4 175 GB | n-cpu-moe 24, split 25,6,6,6, **--no-repack**, mmap, lean RAM | n-cpu-moe 14 VRAM-OOM; ≥18 without --no-repack **segfault**; --no-mmap **host-OOM >125 GB** |
| Q8-MTP 160 GB | n-cpu-moe 14, split 20,8,8,8 (+ no-repack, mmap) | split 21,8,8,7 → **compute-buffer OOM card0** |
| Q3-MTP 143 GB | n-cpu-moe 10… / prod 11 with split 18,9,9,8, PAR 2 | — |

Three different "the model crashed" reports, three different log truths:

- **VRAM-OOM** — too much forced on-GPU for this split.
- **segfault without --no-repack** — a packaging/runtime interaction, not a user prompt.
- **host-OOM without mmap** — the host RAM budget was the real GPU.
- **compute-buffer OOM on card0** — a one-card buffer cliff from a split that looked
  "almost the same" (20,8,8,8 works; 21,8,8,7 dies).

The load log and the OOM killer messages are more informative than a screenshot of the
chat UI. Save them.

## Tiny split changes are not tiny

Operators often nudge tensor split "a little" to chase headroom. The Q8-MTP row is the
warning label: `20,8,8,8` loads; `21,8,8,7` compute-buffer OOMs card0 `[LAB:
RESULTS-MATRIX §F]`.

Treat split vectors as **discrete recipes**, not continuous knobs. When you change them,
you are not "tuning." You are shipping a new placement. Re-run:

- load success
- smoke decode
- product suite if quality-sensitive
- soak if production-bound (chapter 7)

## mmap, repack, and the host as a silent GPU

The community Q4 recipe only behaves when mmap is on and repack is off, with lean host
RAM discipline. Disable mmap and the host can OOM above 125 GB RAM trying to hold what
you thought was a GPU problem `[LAB: RESULTS-MATRIX §F]`.

This is why chapter 1 refused to treat file size as speed. A 175 GB artifact can thrash
the host path and look like a slow GPU model. The log line about mmap and the host OOM
trace are the explanation. "Q4 is slow" is the folklore paraphrase.

## Offload counts: n-cpu-moe as a bandwidth valve

For MoE stacks, how many expert layers live on CPU is a bandwidth valve. Too few on CPU
and you VRAM-OOM. Too many on CPU and every token walks DDR. The working recipes in §F
are not aesthetics; they are measured compromises.

When tok/s is mediocre rather than cratered, compare:

- n-cpu-moe / offload layers
- measured spill traffic if the engine exposes it
- whether speculation is multiplying spilled verifies (chapter 3)

Mediocre is often "too much commute," not "bad GPU silicon."

## Parallel and context: the silent closet

Chapter 4's trap belongs in the load log checklist again: `--parallel N` divides context.
If the startup log says `n_ctx_slot=1536` while the model thinks for 1200 tokens, quality
failures are placement/budget failures `[LAB: RESULTS-MATRIX concurrency notes]`.

Read the slot size before you file a "model loops" ticket.

## A debug order that saves days

When decode is wrong, walk this order:

1. **Confirm binary and model path** (wrong file is undefeated).
2. **Read placement / split / offload** from startup.
3. **Read n_ctx_slot, KV precision, cache-reuse.**
4. **Compare to last known good recipe** (diff the flags, not the vibes).
5. **Check host RAM and mmap/repack** if the artifact is huge.
6. **Only then** touch sampler, batch, or prompt template.
7. **Only then** rebuild engines or re-quant.

Teams that start at step 6 donate weekends to the machine.

## Production soak as a load-log continuation

A load that succeeds is not a load that lasts. The promoted Q3-MTP soak recorded 86
requests in 12 minutes, 0 errors, 100% mean acceptance, **+2 MiB VRAM drift** `[LAB:
RESULTS-MATRIX §G]`. Drift is a load-log story stretched across time: leaks, fragmentation,
cache growth, mis-sized pools.

If your "it got slow after an hour" report has no VRAM/host series, you are debugging
without the patient chart.

## Case patterns (short)

**Pattern A — cratered tok/s, GPUs look idle-ish.** Suspect host-side tax (indexer, spill,
mmap miss). Historical twin: §A ~2 tok/s era.

**Pattern B — hard crash on load.** Suspect split, compute-buffer, repack, VRAM. Twin: §F
failed configs.

**Pattern C — loads, answers garbage on multi-turn.** Suspect KV precision or cache-reuse.
Twin: chapter 4 corruption notes.

**Pattern D — loops / short thinking.** Suspect n_ctx_slot too small under parallel. Twin:
PAR context division.

**Pattern E — fine at c=1, dies under load.** Suspect concurrency scheduling, KV budget,
or aggregate placement. Twin: §B dips and chapter 3's re-bench-at-c rule.

## What this chapter refuses to claim

- We do not claim every engine prints perfect placement logs; we claim you should not
  trust speeds without them.
- We do not claim one universal split vector for all MoE models.
- We do not claim segfaults are always --no-repack; we claim the matrix has a documented
  case you should know exists.
- We do not replace vendor debuggers — we replace knob-first superstition.



## A worked false trail

Symptom: "DeepSeek Q4 is broken; chat is slow and sometimes dies."

False trail: try GGUF from another packer, change temperature, disable tools, rebuild
from `main`.

Load-log trail:

1. File is ~175 GB community Q4.
2. Startup shows no `--no-repack`, mmap off, n-cpu-moe 14.
3. Host RAM climbs past 125 GB; OOM killer lands — or a segfault hits on repack paths.
4. Compare to §F working recipe: n-cpu-moe 24, split 25,6,6,6, `--no-repack`, mmap, lean
   host RAM `[LAB: RESULTS-MATRIX §F]`.

After the recipe matches, decode is merely slow relative to a resident master (16.5 vs
26+), which is now a chapter 1/2 problem, not a haunted model.

The difference between the trails is a day of life.

## Diff recipes like code

Keep a `last-known-good` flag file next to the unit:

```
BINARY=...
MODEL=...
N_CPU_MOE=...
TENSOR_SPLIT=...
PAR=...
CTX=...
KV=...
CACHE_REUSE=...
MMAP=...
REPACK=...
MTP=...
```

When something breaks, `diff` that file against today's flags before you touch weights.
Placement regressions love to hide in "temporary" environment variables that became
permanent.

## Headroom is part of the log

§G's production line recorded 3.3–6.6 GB headroom per card on the promoted Q3-MTP recipe
`[LAB: RESULTS-MATRIX §G]`. Headroom is not vanity. It is the difference between surviving
a concurrent spike and dying on compute-buffer allocation when a second stream arrives.

If load succeeds with 0.1 GB free, you have a demo, not a service. Log headroom beside
tok/s in soaks.


## Reading a bad load like a postmortem

Write the incident in four lines before you change anything:

1. Symptom (tok/s, crash, loop, garbage multi-turn).
2. Last-known-good recipe hash or flag file.
3. Diff of today's flags vs that file.
4. First log line that disagrees with the good run.

Most "mysteries" die on line 3. The rest die on line 4. If you cannot produce a last-known-good, your operational problem is upstream of inference physics: you are flying without a flight data recorder.

## Engine upgrades are placement events

Upgrading llama.cpp or swapping a backend is not a no-op. The §A table is an engine-upgrade story that moved decode from ~2 to ~26 without touching weights `[LAB: RESULTS-MATRIX §A]`. The inverse also happens: a new binary can reintroduce CPU paths, change default offload, or alter cache behavior.

Policy:

- pin engine versions in the unit file
- on upgrade, force a full recipe re-validation (load, smoke, suite, soak)
- never roll engine and model file in the same change window unless the change unit is explicitly "recipe vNext"

## When the log is quiet and the speed is still wrong

Sometimes placement looks correct and tok/s is still mediocre. Then expand the investigation sideways, still without sampler folklore:

- thermal throttle (chapter 7)
- power cap / Max-Q mode
- another process on the GPUs (training pull-ins from chapter 7)
- speculation settings multiplying spill (chapter 3)
- context so long that KV bandwidth dominates (chapter 4)

The load log remains the first read. It is not the only read. It is the read that prevents you from spending Tuesday on temperature 0.7.

## Operator drill

Once a month, on a non-production clone:

1. Break one flag deliberately (mmap off on a huge artifact, or a bad split).
2. Capture the log.
3. Restore last-known-good.
4. Time the diagnosis.

If diagnosis takes longer than restore, your logging is insufficient. Fix logging before the real outage.



## Load log annex: fields worth parsing

If you automate one thing, automate extraction of:

- model path and size
- GPU UUID list
- tensor split
- offload / n-cpu-moe
- n_ctx and n_ctx_slot
- KV type
- cache reuse
- mmap/repack
- warnings and fallbacks
- estimated VRAM per device

Store them as JSON next to the systemd unit start. When tok/s regresses, diff JSON. Humans miss flags; diffs do not.

## The "almost same split" postmortem

Take §F's Q8-MTP working `20,8,8,8` versus failing `21,8,8,7` `[LAB: RESULTS-MATRIX §F]`. A human reviewing a PR might approve the change as harmless. A machine comparing against last-known-good would block it until a load test passed.

That is the standard: **placement PRs require load tests**, not just LGTM.

## Correlating dashboards to logs

Dashboards show symptoms. Logs show placement. A good on-call loop:

1. latency spike alert
2. open current recipe JSON
3. open previous good recipe JSON
4. open startup log of current process
5. only then open Grafana for thermals and traffic

If you start at step 5, you will optimize traffic charts while a bad deploy sits in plain sight.

## Training the team

Once a quarter, give each operator a broken recipe on a scratch host and time them to root cause. Keep the broken recipes from §F-like failures. Celebrate fast diagnosis. Do not celebrate heroic silent fixes that leave no notes.


## Field story: host OOM as a "GPU issue"

The §F --no-mmap host-OOM case is a classic mis-tag `[LAB: RESULTS-MATRIX §F]`. The ticket says GPU. The killer says host RAM. The folklore says quant. The fix says mmap and lean host discipline.

Train on-call to read dmesg and host RAM alongside nvidia-smi. A GPU-only dashboard is a partial cockpit.

## Recipe registry

Beyond last-known-good, keep a registry of recipes with states: experimental, candidate, production, retired, failed. Failed is permanent. Retired keeps history. Experimental is allowed to be wild. Production is boring and pinned.

Most outages are an experimental flag that accidentally became production.


## Operator lab: the five-minute load audit

Set a timer for five minutes at every deploy:

Minute 1: confirm binary hash and model checksum.  
Minute 2: capture startup log to `logs/start-$(date -u +%Y%m%dT%H%M%SZ).txt`.  
Minute 3: extract placement fields to JSON.  
Minute 4: diff against last-known-good JSON.  
Minute 5: run a 20-second smoke generation and record tok/s.

If you cannot do this in five minutes, automate it. Humans under pressure skip steps. Scripts do not feel pressure.

## Mapping symptoms to §F

| Symptom | First §F lookalike |
|---|---|
| segfault on load | missing --no-repack on picky artifacts |
| host RAM death | mmap off on huge artifacts |
| VRAM OOM | n-cpu-moe too low / split too greedy |
| compute-buffer OOM | "almost same" split vector |
| slow but stable | spill commute / wrong engine path |

Keep this table in the on-call doc. It is not complete. It is fast.


## What you should do Monday

1. Implement the five-minute load audit on staging, then on production deploys.
2. Diff recipe JSON on every restart; page if production drifts from last-known-good.
3. Store startup logs with timestamps next to the unit, not only in a volatile journal.
4. Practice one §F lookalike failure on a scratch host with the on-call rotation.
5. Require placement diffs on engine upgrades before traffic returns.

If the log is not archived, the truth was available and then thrown away `[LAB: RESULTS-MATRIX §A/§F]`.


## Cross-links inside this book

This chapter is the debug front door. Chapter 1 gives magnitude heuristics. Chapter 8 gives the fit tombstones you should already have on disk. Chapter 7 covers the cases where the log looks fine and the environment does not. Chapter 5 tells you how to record the before/after so the fix becomes a recipe, not a legend.


## One-page load pledge

No production restart without:

- archived startup log
- recipe JSON diff
- smoke tok/s
- owner initials

If any item is missing, the restart is an experiment on users. The §A crater and §F tombstones exist because someone eventually read the log; the pledge makes that the default rather than the rescue `[LAB: RESULTS-MATRIX §A/§F]`.

## Looking ahead

Chapter 7 keeps reading the machine when the failure is heat or power, not placement.
Chapter 8 turns successful recipes into a fit map: what a 128 GB class box can hold, and
what smaller machines can honestly claim.

The load log is not ancillary output. It is the ground truth of local inference. If the
log and the dashboard disagree, believe the log.
