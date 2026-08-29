# Chapter 7 — Thermals, Power, and Crashes

*(draft v0, 2026-08-28 — written by rogerai-dj for RogerAI Labs; unverified. Numbers
with a `[LAB:]` marker resolve into the lab record. Claims without one are labeled
unmeasured.)*

## The environment is in the critical path

Local inference does not run in a vacuum. It runs in a chassis with finite cooling, a
wall circuit with finite power, and a human world that loses electricity.

If chapters 1–6 are about bytes and recipes, chapter 7 is about what happens when the
building disagrees. Thermals throttle tok/s without changing a single flag. Power loss
turns an un-checkpointed day into heat. Recovery time is a product feature whether you
sell it or not.

## Crashes as measured drills

The lab lost power and wrote the recovery down. That is the correct culture.

**Crash #1 — 2026-08-22.** Recovery of pretraining runs; fstab and mount discipline
became part of the story; recovery cost was large enough to force process changes `[LAB:
PROJECT-LOG 2026-08-22]`.

**Crash #2 — 2026-08-24.** Full recovery in about **25 minutes**. The crash-#1 fixes
held: data volumes auto-mounted, zero filesystem work. New failure: `disable` did not
keep a production DeepSeek unit down because other units pulled it in via `Wants=`; the
durable fix was a condition-gated hold using a marker file. Training resumed from
checkpoints with bounded loss (thousands of steps, not the whole run). Cumulative crash
cost that week: about **9.4 GPU-hours** redone. Checkpoint cadence (5k/4k steps)
continued to bound each loss under roughly 4.5 hours `[LAB: PROJECT-LOG 2026-08-24]`.

Read those entries as inference operators, not only as trainers:

1. **Recovery time is measurable** and should fall after each incident.
2. **Filesystem and service graphs** are part of the model stack.
3. **Checkpoints** convert disasters into bills you can pay.
4. **Dependency pulls** ignore your mental model of "disabled."

## UPS seconds versus checkpoint hours

A UPS that outlasts a graceful flush is wonderful. A UPS that outlasts your ego is rare.
The lab moral is blunt: a UPS buys seconds; a checkpoint buys the day.

For inference servers, the analogs are:

- **model file integrity** on disk (you can reload weights)
- **KV is disposable** (you cannot checkpoint a conversation cheaply in many setups —
  design clients to retry)
- **config and unit files** in version control
- **artifact mirrors** so a dead disk does not strand a one-off GGUF

Do not confuse "the GPU service restarts" with "the system recovered." Recovery means the
service returns to a known recipe with known quality, not merely that a process exists.

## Thermals: the invisible n_max

Heat is a silent speculation killer and a silent concurrency killer. As power limits
engage, clocks fall, tok/s falls, and your carefully priced chapter 3 multiplier becomes
a weather report.

Honest practice:

- Log GPU temperature and power draw beside soak tok/s.
- Bench at sustained load, not only at a 30-second burst.
- Treat Max-Q / power-cap modes as different hardware for SLO purposes.
- If a node sits in a hot aisle or a closet, write that down in the runbook like it was a
  flag.

This book will not invent a thermal curve it did not measure on the reference box in the
cited tables. It will insist you measure yours before you publish a latency SLO.

## Service holds and accidental resurrection

The 08-24 incident where a "disabled" DeepSeek returned because of `Wants=` pulls is an
inference-ops classic `[LAB: PROJECT-LOG 2026-08-24]`. Training jobs, sidecars, and share
units can resurrect the thing you are trying to starve for bandwidth.

Patterns that work:

- **condition-gated units** (`ConditionPathExists=!.../TRAINING_ACTIVE`) so a marker file
  is the source of truth
- **explicit conflicts** in systemd where appropriate
- **one capacity owner** per GPU set in the runbook

If two automation systems can start the same heavyweight server, they will do so on the
worst morning.

## Soak tests are environmental tests

§G's 12-minute soak (86 requests, 0 errors, +2 MiB VRAM drift) is a start, not a
climate chamber `[LAB: RESULTS-MATRIX §G]`. Extend soaks until they match your real duty
cycle: hours for a shop server, days for unattended plant boxes.

Watch:

- tok/s trend
- VRAM/host trend
- temperature/power trend
- error rate
- restart count

A flat error rate with rising temperature and falling tok/s is still a failure if your
SLO is latency.

## Power-loss checklist for inference boxes

After an outage:

1. Confirm disks mounted as expected (fstab nofail vsfail policy intentional).
2. Confirm the **intended** model service is the one that came back.
3. Diff runtime flags against last-known-good (chapter 6).
4. Run a smoke quality suite, not only `curl` health.
5. Run a short soak before reopening traffic.
6. Write the recovery time and the surprise into the log.

If step 6 does not happen, crash #3 will rhyme with crash #1.

## What this chapter refuses to claim

- We do not claim a universal UPS sizing guide.
- We do not claim training checkpoint intervals transfer unchanged to all inference
  products.
- We do not claim the reference box's thermal behavior is measured in full here.
- We do not claim systemd is the only process supervisor — only that dependency graphs
  always exist.


## What training crashes teach inference operators

The 08-22 and 08-24 entries are training-colored, but the failure classes map cleanly onto inference fleets `[LAB: PROJECT-LOG 2026-08-22/24]`:

- **Mount policy** → model store and adapter volumes must come back without human fsck heroics.
- **Checkpoint cadence** → for inference, think config + artifact mirrors + client retry; do not pretend in-flight KV is durable unless you built that.
- **Service dependency edges** → the heavyweight server you stopped for capacity will return if something else Wants it.
- **Bounded loss** → measure "minutes to healthy smoke" the way training measured steps lost.

If you only rehearse happy-path deploys, your first outage is also your first curriculum. Prefer scheduled drills.

## Capacity contention is an environmental hazard

On a shared box, "environmental" includes other jobs. The same physical GPUs cannot honestly serve a training run and a latency-sensitive inference SOP without a written policy. The marker-file hold from 08-24 is one mechanism: a visible, greppable source of truth that prevents accidental co-tenancy `[LAB: PROJECT-LOG 2026-08-24]`.

Write the policy as if it were a safety interlock:

- who may start large models
- what must be stopped first
- how contention is detected (utilization, memory, unexpected processes)
- how long a hold lasts

## Thermal runbooks (minimum)

Even without a full lab thermal curve in the cited tables, a minimum runbook is not optional:

1. Alert on GPU temp and power before users alert on latency.
2. Define a throttle response (shed concurrency, disable MTP, reduce max context) that is better than silent SLO death.
3. Distinguish "hot and stable" from "hot and climbing."
4. After a thermal event, run a quality smoke — some stacks behave oddly near limits.

If you have no sensors, you have no thermal control. Buy sensors before you buy another slogan about edge reliability.

## Incident timeline template

- T0: power loss / thermal trip / OOM storm detected
- T1: process down confirmed
- T2: mounts confirmed
- T3: last-known-good recipe restored
- T4: smoke suite green
- T5: limited traffic
- T6: full traffic
- T7: postmortem notes (what surprised you)

Track T7→T0 improvements across incidents. The 1.5h → ~25m recovery improvement in the lab log is the kind of curve operators should demand of themselves `[LAB: PROJECT-LOG 2026-08-22/24]`.

## Client-side honesty

Not all recovery is server-side. Clients should:

- treat mid-generation disconnects as retryable with idempotent request IDs when the product allows
- avoid assuming KV state survived a restart
- surface "model restarted, context cleared" rather than silently continuing a half-dead session

A perfect server recovery still looks broken if the UI pretends the desk still holds the conversation.

## Boundaries again

This chapter does not turn into a facilities-engineering manual. It insists that power, heat, and dependency graphs are first-class inference inputs, and that the lab's crash ledger is the model for how to talk about them: dated, measured, and corrected in public.


## Power budget as a product constraint

A wall-circuit limit is a concurrency limit in disguise. If the rack or the office circuit
cannot sustain all four GPUs at the boost the decode table assumed, your chapter 1 numbers
are from a machine you do not actually own under load.

Write power the way you write VRAM:

- rated circuit capacity
- measured draw at production concurrency
- headroom for startup surges
- whether other tenants share the circuit

When draw and latency rise together, believe the PDU. When a node "gets slow every
afternoon," check whether the afternoon is also when the HVAC loses the aisle.

## Checkpoint philosophy for inference configs

Training checkpoints save optimizer state. Inference checkpoints are boring and therefore
neglected:

- unit files and drop-ins in git
- model file checksums
- recipe flag files (chapter 6)
- a smoke-suite script pinned next to the unit
- a known-good artifact mirror on a second disk

After 08-24, the lab could resume training because checkpoints existed `[LAB: PROJECT-LOG
2026-08-24]`. After an inference host crash, you can resume service only if the boring
artifacts exist. If rebuilding the unit requires tribal memory, your RTO is "until the
right person wakes up."

## Coordination with speculative decoding under heat

Chapter 3 priced MTP under bandwidth. Heat changes the price. If clocks fall, the heavy
step \(H\) gets larger, and a draft policy that was barely winning can start losing without
any flag change. A thermal runbook that disables MTP or shortens n_max under sustained high
temp is not cowardice; it is re-solving the inequality from chapter 3 with new inputs.

Pair thermal alerts with a known degraded mode:

1. full recipe
2. no MTP
3. reduced parallel
4. reduced max context

Document the mode transitions so on-call does not invent them at 03:00.

## Human factors

The worst crash recovery failures are social: two people start conflicting services, a
"temporary" hold is forgotten, a training job is re-enabled by a timer nobody owns. Put
names on holds. Put expiry on holds. Put the active capacity owner in the status page.

The marker-file pattern works partly because it is greppable and physical on disk `[LAB:
PROJECT-LOG 2026-08-24]`. Prefer boring visibility over clever automatic healing that
nobody can see.




## The afternoon slump

A common local-inference ticket: "it's fine in the morning, slow after lunch." Possibilities:

- thermal soak
- concurrent human users
- scheduled jobs
- grid or UPS mode changes
- someone started a training run

The 08-24 resurrection story is the social version of this ticket `[LAB: PROJECT-LOG 2026-08-24]`. Without a capacity owner and a greppable hold, the afternoon slump is permanent.

## RTO/RPO for inference

Borrow the backup vocabulary:

- **RTO** — minutes until smoke suite green
- **RPO** — how much conversation or job state you accept losing (often: all in-flight KV)

Write targets. Measure against the last drill. The lab's crash recovery curve (hours toward ~25 minutes) is the kind of progress bar leadership understands `[LAB: PROJECT-LOG 2026-08-22/24]`.

## Degraded modes as first-class configs

Keep three unit templates:

- `model-full.service`
- `model-degraded-no-mtp.service`
- `model-emergency-small.service`

Document when on-call may switch. Degraded modes that exist only in someone's head do not exist.

## After-action: the only five questions

1. What failed first (power, heat, OOM, deps)?
2. What did the logs say before the failure?
3. What was the measured recovery time?
4. What tombstone recipe or monitor did we add?
5. What false fix did we avoid?

If question 4 is empty, the incident will recur.


## Field story: the dependency graph that undid a disable

The DeepSeek unit that returned despite disable because of Wants= pulls is a perfect inference fable `[LAB: PROJECT-LOG 2026-08-24]`. The operator believed in a boolean. The system believed in a graph.

Draw the graph once. Put it in the runbook. Include sidecars. Include "helpful" share units. Include anything that can call start.

## Drills beat dashboards

A green dashboard never practiced a black start. Schedule a quarterly hard stop on a spare box: kill power or drop the unit, then measure T0–T6 from chapter 7's timeline. If you cannot spare a box, you are already running a higher risk than you admit.


## Operator lab: black-start drill script

On a spare host:

1. Start production-like unit.  
2. Run smoke suite.  
3. Record steady tok/s and temp.  
4. Hard kill the unit (or pull power on a dedicated lab PDU if safe).  
5. Bring host back.  
6. Do not use tribal memory — use only runbook.  
7. Time until smoke green.  
8. Write five-line after-action.

Compare to the lab's published recovery improvement curve as inspiration, not as a leaderboard `[LAB: PROJECT-LOG 2026-08-22/24]`. Your hardware differs. Your process should still improve.

## Environmental SLOs

Consider adding SLOs that are not user-facing latency:

- max GPU temp at production concurrency
- min headroom VRAM during soak
- max unplanned restarts / week
- max minutes to smoke after hard kill

These SLOs prevent a culture where only chat latency is "real."


## What you should do Monday

1. Schedule a black-start drill on a non-critical host and record minutes to smoke.
2. Draw the service dependency graph that can resurrect heavyweight model units.
3. Define degraded modes (no MTP / reduced PAR / emergency small model) as real units.
4. Add temp/power panels next to latency panels.
5. Name a capacity owner for each GPU set and an expiry for every hold.

The 08-24 marker-file hold is a pattern you can steal without stealing the rest of the lab's stack `[LAB: PROJECT-LOG 2026-08-24]`.


## Cross-links inside this book

After recovery, chapter 6's five-minute audit is mandatory before traffic. Degraded modes that disable MTP must still pass chapter 5 smokes. Power and heat change chapter 3's inequality without changing flags — re-measure. If a crash returns a different dependency graph, chapter 8's capacity owner and class cards need an update, not only a process restart.


## One-page recovery pledge

After any hard failure:

- minutes to smoke recorded
- dependency graph checked
- degraded mode documented if used
- tombstone written if a new failure class appeared

The 25-minute recovery is not a legend to admire. It is a standard to beat with process `[LAB: PROJECT-LOG 2026-08-24]`.

## Looking ahead

Chapter 8 closes the book with fit: what a ~128 GB VRAM class machine can hold under
real recipes, what fails, and what smaller machines can honestly promise without cosplay.

The best inference stack is the one that returns from a bad Wednesday with a receipt.
