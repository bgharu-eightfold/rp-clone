# RP-Clone Engine — Algorithm Design (v2: routine optimizer)

A local web app that takes **your current training program as input**, diagnoses how it stacks up against evidence-based training principles (volume landmarks, RIR proximity, frequency, SFR, balance), and plans a structured block of training to optimize it. Over successive blocks, the engine tunes per-user landmarks, recovery profiles, and per-exercise priors from your logged data.

**Framing**: this is a routine *optimizer*, not a program generator. It works for any starting state — chaos to tight UL split — and never forces a rewrite. It proposes a minimal delta and explains why.

---

## 1. End-to-end loop

```
[Ingest current routine] → [Diagnose] → [Plan next block]
                                              ↓
[Update per-user model] ← [Block ends] ← [Log + autoregulate within block]
                                              ↑
                                       (loop forever)
```

Two timescales:

- **Within a block (mesocycle)**: RIR ladder, double-progression load, set autoregulation matrix, deload triggers. Reactive to per-session feedback.
- **Between blocks**: re-diagnose, refine per-user landmarks and exercise priors, decide whether to bias the next block toward weak muscles, address imbalances, change frequency or split.

---

## 2. Core concepts (one-screen recap)

- **Mesocycle** — 4–6 weeks: accumulation + 1 deload.
- **Microcycle** — one week.
- **Volume landmarks** per muscle:
  - **MV** maintenance (below this you lose muscle)
  - **MEV** minimum effective (lowest dose that grows)
  - **MAV** maximum adaptive (sweet spot range)
  - **MRV** maximum recoverable (ceiling; past this fatigue beats stimulus)
- **RIR** reps in reserve; proxy for proximity to failure.
- **SFR** stimulus-to-fatigue ratio of an exercise.
- These are evidence-based defaults; the engine personalizes them with your logged data over time.

---

## 3. Ingestion — bootstrap from your current routine

Ingestion uses the **same UX shape as daily logging**: a session-by-session card that renders prescribed exercises with planned weight / reps / RIR, and per-set actual fields the user fills after the workout.

For ingestion, the user pre-fills the "prescribed" side of each session to represent their current routine. "Actuals" stay blank until they actually train that day.

```
CurrentRoutine
  daysPerWeek
  rotationLengthDays: 7 | 14 | other      # how many calendar days to complete one full rotation
  frequencyPerMusclePerWeek: derived       # critical input for the diagnostic
  unit: kg                                  # locked
  sessions: [
    {
      label: "Pull (Back+Biceps)" | "Push" | etc.
      orderInRotation: int
      exercises: [
        {
          name: "Pull-ups"
          sets: int
          repRange: [lo, hi]               # or single rep target
          plannedLoad: number?              # absent for bodyweight unless +ext load
          rirEstimate: int?
          notes?
        }
      ]
    }
  ]
```

**The session card UI** (used for both ingestion and daily training):

```
Today: Pull (Back + Biceps)         Week 2 / 5    RIR target: 2

Pull-ups            3 sets × 6–10 reps        BW              RIR 2
  Set 1   [weight]   [reps]   [rir]
  Set 2   [weight]   [reps]   [rir]
  Set 3   [weight]   [reps]   [rir]

Chest-supported DB rows    3 sets × 8–12     22.5 kg         RIR 2
  Set 1   [weight]   [reps]   [rir]
  ...
```

Before workout: read-only "today's plan" view. During/after: same card with editable actual fields.

The app matches `name` against a curated exercise library (~80 movements tagged with primary/secondary muscles, category, SFR rating, joint-sensitivity, default rep range, smallest load increment). Unknown exercises get a 30-second tag flow (1 tap for primary muscle, 1 tap for category).

**Output**: a normalized routine the diagnostic engine can analyze.

---

## 4. Diagnostic — what the app analyzes

Given the normalized routine, compute per muscle:

- **Weekly direct sets** — count only sets where the muscle is the **prime mover** (a 1.0× direct set). Secondary contributions from compound work (e.g., biceps during pull-ups, triceps during bench) are **not counted by default**, because RP's volume landmarks are calibrated against direct sets and most compound work doesn't load the secondary muscle close enough to failure to be productive volume. A per-exercise override lets the curator mark a near-primary case (e.g. chin-ups crediting 0.5× to biceps, close-grip bench crediting 0.5× to triceps); these overrides live in the exercise library, not in user settings.
- **Frequency** — sessions/week hitting the muscle hard.
- **Per-session max sets** — flags `junk_volume` if any session exceeds ~10 direct sets for one muscle.
- **Exercise SFR mix** — % of sets from high vs. low SFR exercises.
- **Compound/isolation ratio**.
- **Rep-range spread** — heavy only, light only, or balanced.
- **Intensity proxy** — average reps vs. exercise default range (rough proxy for how hard sets are taken).

Plus structural checks:

- **Imbalances** — push/pull set ratio, knee-dom/hip-dom set ratio, anterior/posterior delt ratio.
- **48h spacing** — same muscle on consecutive days flagged.
- **Frequency feasibility** — if weekly target > 10 × frequency, flag `frequency_too_low`.

Output is a **diagnostic report** with per-muscle status:

| Status        | Meaning                                |
|---------------|----------------------------------------|
| `under_MV`    | losing muscle; bump immediately        |
| `under_MEV`   | not growing; needs more volume         |
| `inside_MAV`  | sweet spot                              |
| `near_MRV`    | hold or slight reduce                   |
| `over_MRV`    | overshooting; reduce                    |

Plus structural flags (`junk_volume`, `low_SFR_dominant`, `imbalance_pull_under`, `frequency_too_low`, etc.) with concrete fixes attached.

---

## 5. Proposing the first planned block

The planner generates a 4–6 week mesocycle that is a **minimal delta** from your current routine. Defaults:

1. **Set targets per muscle**:
   - `under_MEV` → ramp to MEV by W1, then progress toward MAV across the block.
   - `inside_MAV` → keep current W1 sets, progress as normal.
   - `over_MRV` → reduce to top of MAV for W1.
2. **Frequency**: if any muscle needs more sets than `10 × current_frequency`, propose adding a session for that muscle (or splitting one).
3. **Exercise selection**: keep your existing exercises unless flagged. Suggested swaps are *proposals* with rationale; you accept or decline.
4. **Loads & rep ranges**: use what you already do as W1 prescription. Progression starts from there.
5. **W1 RIR seed**: from your reported current effort. Push-close-to-failure type → start RIR 2. Conservative → start RIR 3–4.
6. **Priority levels** (see §7d): the user assigns `Specialize`/`Normal`/`Maintenance`/`Off` per muscle group at block creation. Default is `Normal` for everything. This is the lever for focused growth blocks and for rotating focus across mesos.

The plan is shown as a **side-by-side diff** against your current routine, with a short "why" beside each change. Priority assignments live alongside the diff so you can see exactly which muscles are being pushed, held, or maintained this block.

---

## 6. Within-block autoregulation

### 6a. RIR ladder (default 5-week meso)

| Week | Target RIR | Notes                                 |
|------|------------|---------------------------------------|
| 1    | 3          | Ease in. Bottom of each rep range.    |
| 2    | 2          | First real overload.                   |
| 3    | 1          | Peak fatigue starts to bite.           |
| 4    | 0–1        | Last hard week.                        |
| 5    | 4–5        | **Deload**: 50% sets, ~60% load.       |

Seed-week RIR may be biased ±1 by diagnostic.

**Per-exercise RIR floor**: each exercise in the library carries a `minRIR` cap.

| Exercise category                                                                  | `minRIR` | Behavior                              |
|------------------------------------------------------------------------------------|----------|---------------------------------------|
| CNS-taxing compounds (squat, deadlift, bench, OHP, RDL, heavy hip thrust)          | 1        | Never to failure, even in W4.         |
| Standard compounds (rows, pull-ups, dips, lunges)                                  | 0        | Failure allowed in W4 if joints OK.   |
| Isolations and machines (curls, extensions, raises, leg curls, calf raises)        | 0        | Failure allowed any week.             |

The week's RIR is the *muscle-level* target; the per-exercise floor is enforced on top. In W4 (target RIR 0–1): squats target RIR 1, quad extensions target RIR 0.

**Why this matters**: compound, CNS-taxing exercises taken to true failure deplete recovery capacity disproportionately to the muscle stimulus gained. A single Smith-squat session taken to RIR 0 every set can require 2–3 days of recovery, leak into the next session's performance, and cause adherence breakdown elsewhere in the week. Most "one leg day needs two rest days" cases are this exact pattern.

### 6b. Load + rep progression — double progression

Per-exercise rule, evaluated within a microcycle:

```
target = exercise.defaultRepRange       # e.g. [8, 12]
plan_load = last_session.load
plan_reps = bottom of range on new exercise

After each logged session of an exercise:
  if all working sets hit target.hi AND actual_RIR ≤ target_RIR:
      next_load = plan_load + loadIncrement
      next_reps_target = target.lo
  elif all working sets hit ≥ target.lo AND actual_RIR ≤ target_RIR + 1:
      next_load = plan_load
      next_reps_target = min(last_reps + 1, target.hi)
  else:
      if perf flat-or-down for ≥2 consecutive sessions:
          flag for deload check
      next_load = plan_load
      next_reps_target = plan_reps
```

Why double progression: reps absorb fatigue from accumulating weekly volume without forcing premature load jumps.

### 6c. Set autoregulation matrix (the heart of the engine)

After each session, for each muscle trained, compute Δ-sets for that muscle's **next exposure**.

Inputs (asked at session start or end, one tap each):

- `soreness` ∈ {0 healed early, 1 healed just in time, 2 still sore, 3 very sore}
- `jointPain` ∈ {0 none, 1 mild, 2 moderate, 3 high}
- `perf` ∈ {0 better, 1 same, 2 worse} — derived from logged sets vs. last exposure
- `pump` ∈ {0 none, 1 low, 2 moderate, 3 excellent}
- `workload` ∈ {0 easy, 1 pretty good, 2 hard, 3 crushed}

Decision table, top-down, first match wins:

| #  | Condition                                                | Δ sets | Rationale                                       |
|----|----------------------------------------------------------|--------|--------------------------------------------------|
| 1  | `jointPain ≥ 2`                                          | −1, rotate exercise | Joint dominates. Suggest SFR-equiv swap. |
| 2  | `perf == 2` AND `soreness == 3`                          | −2, flag deload     | MRV exceeded.                              |
| 3  | `perf == 2` OR `soreness == 3`                           | −1                  | Strong recovery alarm.                     |
| 4  | `workload == 3`                                          | 0                   | Crushed — hold.                            |
| 5  | `soreness == 2` AND `workload ≥ 2`                       | 0                   | Slightly under-recovered.                  |
| 6  | `soreness ≤ 1` AND `pump == 3` AND `workload == 2`       | 0                   | Inside MAV sweet spot.                     |
| 7  | `soreness ≤ 1` AND `pump ∈ {1,2}` AND `workload ∈ {1,2}` | +1                  | Room to grow.                              |
| 8  | `soreness == 0` AND `pump ≤ 1` AND `workload ≤ 1`        | +2                  | Below MEV. Under-dosed.                    |
| 9  | (anything else)                                          | +1                  | Default mild progression.                  |

After applying Δ, clamp to `[max(MV, currentSets − 3), MRV_user]`.

### 6d. Deload triggers

Any one fires:

1. **Calendar**: meso has reached its planned final week.
2. **Performance regression**: 2 consecutive sessions for the same muscle with `perf == 2` AND `soreness ≥ 2`.
3. **MRV stall**: any muscle at MRV for ≥ 2 weeks with flat/down session scores.
4. **Joint cluster**: 2+ muscles flagged for joint pain in the same week.
5. **User override**: "I'm cooked" button.

Deload prescription: 1 week, 50% sets, ~60% loads, RIR 4–5. Auto-start next meso at MEV.

### 6e. Floating sets and catch-up sessions

Real-world adherence is rarely perfect. The engine supports a **fixed schedule + optional catch-up day** pattern so missed volume can be recovered within the same week instead of lost.

**Set priority tags** (set at block creation, per prescribed set):

| Tag        | Meaning                                                                                              |
|------------|------------------------------------------------------------------------------------------------------|
| `required` | Must be done on its scheduled day. Missed = logged as missed.                                        |
| `floating` | Allowed to slip from its scheduled day to a catch-up slot later in the week.                         |

Defaults: compounds and primary exercises for a muscle are `required`. Isolation accessory work (arms, lateral raises, rear delts, calves) is `floating` by default. The block planner sets these; the user can override per exercise.

**Catch-up slot** (typically Sunday, configurable in user prefs):

The engine reserves a non-prescribed weekday as the catch-up slot. At the start of that day, the engine evaluates each muscle:

```
for each muscle:
  actual_sets = sum of logged sets this week
  prescribed_sets = sum of required + floating sets this week

  if actual_sets >= 0.75 * prescribed_sets:
      no catch-up needed for this muscle

  else:
      missing = prescribed_sets - actual_sets
      catch-up exercises = floating sets not yet done, capped at `missing` sets
      add to catch-up session, respecting:
        - 48h spacing rule (skip if muscle was hit yesterday with any compound work)
        - per-exercise RIR floor and current week's RIR target
        - max ~10 direct sets per muscle on the catch-up day
```

If no muscle needs catch-up, the slot is rest. If catch-up is needed but the only missing sets violate 48h spacing, the engine rolls those sets into the next week's prescription (not lost — repaid later).

**Why this is a first-class feature, not a hack**:

It lets the engine track adherence honestly. "I skipped a bicep set on Monday" no longer needs to be logged as a missed session — it's logged as a successful deferral. The actual-vs-prescribed delta the engine reads at the between-block review is the *true* delivered volume, not the prescribed-on-paper volume. This makes the personalized-landmark learning in §7 substantially more accurate.

---

## 7. Between-block learning — what makes this *yours*

After each block ends, the engine updates a **per-user model**:

### 7a. Personalized landmarks

RP's published landmarks are priors. After each block, update each muscle's MEV/MAV/MRV with a simple Bayesian update toward the user's observed data:

```
observed_MEV ≈ smallest weekly set count this block where:
  avg(pump) ≥ 2 AND avg(soreness) ≥ 1
observed_MRV ≈ largest weekly set count this block where:
  perf did NOT regress AND soreness did NOT hit 3 two weeks running

learned_X = (prior_X * w_prior + observed_X * w_data) / (w_prior + w_data)
  where w_data increases with block count (e.g., w_data = n_blocks)
```

After 2–3 mesos the engine quietly drifts each muscle's landmarks toward the user's actual recovery profile. Defaults still anchor — no wild swings.

### 7b. Per-exercise priors

- **Load progression rate**: rolling avg of lb (or kg) added per week under double progression. Used to set realistic W1 expectations next block; flags exercises that have plateaued ≥2 blocks for rotation.
- **Effective SFR**: starts at library default; if joint pain or pump-deficit recurs, downgrade for this user.
- **Last-used / staleness**: rotate exercises that hit `(3 sessions no PR) ∨ (joint pain twice consec) ∨ (user flags "bored")`.

### 7c. Frequency tuning

If muscle's learned-MAV midpoint > `10 × current_frequency`, the next-block plan proposes adding a session that hits the muscle. Conversely, if muscle is chronically over-recovering at high frequency, propose consolidating.

### 7d. Block focus & priority levels

Each block, the user assigns a **priority level** per muscle group. Default for a fresh user is `Normal` across the board; specialization is opt-in. Priorities can be changed at every between-block review — running specialization cycles across blocks is a first-class workflow, not a hack.

| Priority     | Volume target                       | Progression                                | Use case                                       |
|--------------|--------------------------------------|--------------------------------------------|------------------------------------------------|
| `Specialize` | Start `MEV + 2`, drive toward MRV    | Aggressive (+1/+2 set bias on green flags) | Lagging muscle, focused growth block            |
| `Normal`     | Start MEV, drive to MAV mid          | Standard autoregulation                    | Default for everything you want to grow         |
| `Maintenance`| Hold at MV all block                 | None (sets flat, RIR ~3 the whole block)   | Free up recovery for specialized muscles        |
| `Off`        | 0 sets                               | —                                          | Injury, deload-extension, deliberate skip       |

**Why maintenance matters**: total systemic recovery is finite. Specialization only works if other muscles give up their share of the recovery budget. Pulling 4 muscle groups back to MV typically frees ~15–25 weekly sets that can be redirected into specialized muscles without overshooting MRV systemically.

**Specialization budget** (enforced by planner):

- **Default cap: 3 muscles on `Specialize` per block.** Going beyond shows a warning. The engine still lets you try, but it autoregulates more aggressively if recovery flags fire — i.e. specialized muscles get cut back first when MRV signals appear.
- Cap is configurable in settings.
- The planner sums projected weekly hard sets across all `Specialize` + `Normal` muscles and warns if the total exceeds a heuristic systemic ceiling.

**Cycling specialization across blocks** (the workflow you're asking about):

You can rotate focus every block, every 2 blocks, every 3 — whatever you want. Example:

```
Block 1 (5 wk): Specialize chest + side delts, Maintenance legs + back
Block 2 (5 wk): Specialize back + biceps,     Maintenance chest + side delts
Block 3 (5 wk): Specialize legs + glutes,     Normal upper body
```

At the **between-block review**, the planner surfaces a "rotate focus?" panel that suggests:

- Muscles that haven't been `Specialize`-d in the last N blocks.
- Muscles still measurably behind their personal MAV midpoint after past blocks.
- Muscles flagged as `priority=high` in your settings.

You accept the proposal, override it muscle-by-muscle, or ignore it and set priorities yourself. The engine never forces a rotation.

**Hold a focus for multiple blocks**: fully supported. If you want to specialize chest for three blocks straight, set it that way each between-block review. The engine will warn after 2 consecutive blocks of the same `Specialize` muscle if observed progress has stalled (suggests it may need a rest from peak volume to grow), but it will not block you.

**Lifecycle of a muscle's priority**:

```
[set at block start] → [held all block] → [reviewed between blocks]
                              ↓                       ↓
                       (no mid-block changes)   (changeable freely)
```

Mid-block priority changes are deliberately *not* allowed — switching priorities mid-meso breaks the autoregulation math (volume targets are set relative to the priority at W1). If you really need to bail on a focus mid-block, the right move is to trigger an early deload and start a new meso.

---

## 8. Data model

```
User
  goal, split (mutable), experience, equipment, unit, prefs

ExerciseLibrary  (seeded ~80 entries; user can extend)
  {id, name, primaryMuscle, secondaryMuscles, category,
   defaultRepRange, defaultSFR, jointSensitivity, loadIncrement}

CurrentRoutine
  daysPerWeek, sessions: [...]   (from ingestion; updated when user edits)

Mesocycle
  id, startDate, lengthWeeks, focus, microcycles, status
  plannedRoutineSnapshot         (the diff-from-current routine for this block)

Microcycle
  weekIndex, targetRIR, sessions: [Session]

Session
  id, dateScheduled, dateCompleted
  splitDay, prescribedSets, loggedSets,
  preSessionFeedback, postSessionFeedback

PrescribedSet { exerciseId, targetReps, targetRIR, prescribedLoad }
LoggedSet     { exerciseId, weight, reps, rir, note }

Muscle  { id, name, landmarks:{MV,MEV,MAV[lo,hi],MRV} }

# --- the learning layer ---
UserModel
  perMuscle:
    {muscleId → {
      learnedMV, learnedMEV, learnedMAV, learnedMRV,
      sampleSizeBlocks, priority, recoveryHalflifeDays
    }}
  perExercise:
    {exerciseId → {
      loadProgressionRate, jointFlagCount, effectiveSFR,
      lastUsedDate, plateauStreak
    }}
  blockHistory: [
    {mesoId, startDate, endDate, outcome,
     finalVsW1PerfDelta, deloadCause}
  ]
  routineSnapshots: [{date, routine}]   # for diff/rollback
  diagnosticHistory: [{date, report}]
```

Persistence: `localStorage` (small enough for years) with one-button JSON export/import for backup or moving devices.

---

## 9. UI surfaces (priority order)

1. **Onboarding wizard** — enter your current routine.
2. **Diagnostic report** — per-muscle status, imbalances, structural flags.
3. **Block plan diff** — side-by-side current vs. proposed, with "why" per change.
4. **Today's session card** — prescribed sets/loads/RIR + tap-to-log.
5. **Pre/post-session feedback prompts** — one tap per muscle.
6. **Next-session prescription** — with "why this changed" tooltip.
7. **Block dashboard** — week N of M, per-muscle volume trend, deload risk gauge.
8. **Between-block review** — outcome, what the engine learned, next-block proposal.
9. **History** — per-exercise charts, per-muscle volume across blocks.
10. **Export / import JSON**.

---

## 10. Out of scope for v1

Cardio, nutrition / macro tracker, bodyweight-only progression, multi-meso macrocycles (auto-cycling hypertrophy ↔ strength), cloud sync, wearables (HRV), photo / measurement tracking.

---

## 11. Decisions locked

- **Block length**: 5 weeks (4 accumulation + 1 deload).
- **Units**: kg.
- **Diagnostic posture**: always propose; user accepts or declines per item. Engine never silently rewrites the routine.
- **Ingestion shape**: session-by-session form, same UX shape as daily logging (see §3).

## 12. Open questions (remaining)

1. **Specialization cap** — default `Specialize` slots per block: I've spec'd 3 with a soft warning past that, never a hard block. Keep as-is, raise, or lower?
2. **Frequency clarification** — see DIAGNOSTIC.md; the single biggest unknown about your current routine.
