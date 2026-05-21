# Diagnostic — Current Routine (paper run, 2026-05-17, **v4: compound-RIR finding**)

The engine's analysis applied to your stated routine, by hand, to validate the rules before code. **v4 incorporates the CNS-fatigue signal from Wed legs and reframes Block 1 around fixing compound-RIR first**, before adding any volume or frequency. v3's direct-set counting is still in effect.

**Reported recovery pattern**: Wed legs sometimes cost 2–2.5 days of recovery; Fri Pull is occasionally skipped because of residual CNS fatigue. User's own hypothesis: "maybe I'm going too close to failure on Wednesdays." Engine agrees — this is the textbook signature of compound work taken to true failure.

---

## 1. Routine and schedule

| Day | Session                                                                       |
|-----|-------------------------------------------------------------------------------|
| Mon | Pull (Back + Biceps)                                                          |
| Tue | Push (Chest + Shoulders + Triceps)                                            |
| Wed | Legs                                                                          |
| Thu | Rest                                                                          |
| Fri | Pull                                                                          |
| Sat | Push                                                                          |
| Sun | Rest, **or** catch-up arm day (6–8 tris, 6–8 bis, 4 lat raises) **if** arm work was skipped earlier in the week. Sat triceps work omitted when Sun is run. |

Frequency per muscle (typical week):

| Muscle group                    | Sessions / week              |
|---------------------------------|------------------------------|
| Back, Biceps                    | 2 (Mon, Fri)                 |
| Chest, Front delts, Rear delts  | 2 (Tue, Sat)                 |
| Side delts                      | 2 (Tue, Sat) or 3 if Sun ran |
| Triceps                         | 2 (split Tue+Sat or Tue+Sun) |
| Quads, Hams, Glutes, Calves     | **1** (Wed only)             |

Units: kg.

---

## 2. Set counting — corrected methodology

**v3 rule (matches RP's app):** a set counts toward a muscle only when that muscle is the *prime mover* and is loaded near failure. Pull-ups don't credit biceps; bench doesn't credit triceps; squats credit quads but not glutes unless the variant is hip-dominant.

A small set of exercises have explicit "near-primary" overrides in the library — e.g. chin-ups credit biceps at 0.5×, close-grip bench credits triceps at 0.5× — but these are exceptions, not the default.

The exercises in your routine all use direct counting (no overrides apply).

---

## 3. Weekly direct sets per muscle

| Muscle      | Calc                              | Sets | MEV | MAV     | MRV | Status                              |
|-------------|-----------------------------------|------|-----|---------|-----|-------------------------------------|
| Back        | 3×2 + 3.5×2 + 3×2                 | 19   | 10  | 14–22   | 25  | inside MAV, room ~3 to MRV          |
| Biceps      | 3×2 + 3×2  (+7 Sun if run)        | 12 (19 with Sun) | 8 | 14–20 | 26 | A: at MAV bottom. B: mid-MAV. Neither over MRV. |
| Triceps     | 3×2 + 3×2  *or* 3×1 + 7 Sun        | 12 (13 with Sun) | 6 | 10–14 | 18 | inside MAV either way               |
| Chest       | 3×2 + 3×2                         | 12   | 10  | 12–20   | 22  | at MAV bottom                       |
| Front delts | 3×2                               | 6    | 0   | 6–16    | 18  | at MAV bottom                       |
| Side delts  | 4×2  (+4 Sun if run)              | 8 (12 with Sun) | 10 | 16–22 | 26 | A: **just under MEV**. B: above MEV. |
| Rear delts  | 3×2                               | 6    | 6   | 10–20   | 25  | at MEV                              |
| Quads       | 3 + 3                             | 6    | 8   | 12–18   | 20  | **under MEV**                       |
| Hamstrings  | 3                                 | 3    | 6   | 10–16   | 20  | **under MEV (big gap)**             |
| Glutes      | 0 direct (Smith squats credit quads only) | 0 | 0 | 4–12 | 16 | **no direct work**                  |
| Calves      | 3                                 | 3    | 8   | 12–16   | 20  | **under MEV (big gap)**             |
| Abs / Traps | 0                                 | 0    | —   | —       | —   | untrained (info)                    |

Your instinct was right — arms aren't past MRV. They're sitting at the bottom-to-middle of MAV. The Sunday catch-up brings them to mid-MAV; without Sunday, biceps and triceps both land at the MAV floor.

---

## 4. Structural flags (severity-ordered) — v4

| # | Flag                                       | Detail                                                                                                                                                  | Severity |
|---|--------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| 1 | **`RIR_too_aggressive_on_compounds`**      | Smith squats (and likely other compounds) taken too close to failure. Signature: one low-volume leg day costs 2–2.5 recovery days; leaks into Fri Pull adherence. Engine rule (now in `ALGORITHM.md §6a`): compounds carry `minRIR = 1` — never to failure, even in W4. | **High** |
| 2 | `leg_volume_under_MEV` (gated by #1)       | Quads 6, hams 3, glutes 0, calves 3. All under MEV. But fixing #1 first is the prerequisite — adding volume on top of CNS-saturating sessions makes things worse. | High     |
| 3 | `under_MEV: side delts` (Pattern A only)   | 8 sets vs MEV 10 on weeks Sunday isn't run. +4 sets across the two push days closes it.                                                                | Med-High |
| 4 | `no_direct_glute_work`                     | Smith squats credit quads only under direct counting. Need hip-dominant work for glute hypertrophy.                                                     | Medium   |
| 5 | `hams_no_hinge`                            | Seated curls only; no RDL/good morning for the long head.                                                                                                | Medium   |
| 6 | `calves_gastroc_neglect`                   | Seated calf raises only (soleus). Missing standing variant for gastrocnemius.                                                                            | Low      |
| 7 | `RIR_too_aggressive: pull-ups`             | Same family as #1, isolation/standard-compound version: pull-ups to failure is fine in W4 but should be RIR 3 in W1.                                    | Low      |
| 8 | `adherence_variance: arms`                 | Sunday catch-up pattern. Working as intended — the app's logged-vs-prescribed delta quantifies this automatically.                                      | Info     |

**What's correctly tuned:**

- Push : Pull set balance is fine in direct-set counting (≈ 36 push, ≈ 31 pull excluding arms; arms ~24 split 12/12).
- Frequency on every upper-body muscle is at the 2×/week target.
- Arms are *not* overdosed — your sense of your routine was correct.
- Exercise selection has solid SFR overall.
- Rep ranges sensibly cover 6–10, 8–12, 12–15.

---

## 5. The big insight (v4)

The routine has two real problems and one structural pattern:

1. You're going too close to failure on CNS-taxing compounds. Your hypothesis was right. A 6-set quad session costing 2.5 recovery days isn't a volume problem — it's an RIR problem. Compounds should stop at RIR 1 even on the hardest week of a meso. Going to true failure on Smith squats depletes the CNS far more than it grows the muscle, and the cost leaks across the rest of the week (Fri Pull skipped).

2. Lower body volume is under MEV — but #1 has to be fixed first. Adding a Lower B day on top of a 1×/week session that already over-fatigues you would be a disaster. Get the RIR right, see how recovery responds, then the engine decides whether to add frequency.

3. Sunday catch-up is informal autoregulation on arms — keep it; the app can formalize it as floating sets.

This is exactly the class of decision the engine should be making block-by-block: don't change two things at once if you can't tell which one helped. Block 1 fixes the RIR problem in isolation; Block 2 reads the data and decides on frequency.

---

## 6. What Block 1 would propose (revised)

Block 1 is now a **recovery-recalibration block**. Goal: validate that fixing compound RIR restores Fri adherence and drops Wed recovery cost. No new training days, no specialization.

### 6a. No schedule change

| Day | Current                | Proposed Block 1     |
|-----|------------------------|----------------------|
| Mon | Pull                   | Pull                 |
| Tue | Push                   | Push                 |
| Wed | Legs                   | Legs (revised below) |
| Thu | Rest                   | Rest                 |
| Fri | Pull                   | Pull                 |
| Sat | Push                   | Push                 |
| Sun | Rest / catch-up arms   | Rest / catch-up arms |

Lower B is **deferred to Block 2 as a data-driven decision**.

### 6b. Compound RIR fix — the headline change

| Exercise              | Current effort  | W1 → W4 RIR target            | W1 weight adjustment        |
|-----------------------|-----------------|--------------------------------|------------------------------|
| Smith machine squats  | ~failure        | 3 → 2 → 1 → **1** (compound cap; never failure) | Drop ~5–10 kg from current top weight |
| Pull-ups              | failure         | 3 → 2 → 1 → 0 (failure OK W4)  | Drop one rep per set W1      |
| Incline Smith bench   | hard (assumed)  | 3 → 2 → 1 → **1** (compound cap) | Weight that leaves 3 reps tank W1 |
| Standing DB OHP       | hard (assumed)  | 3 → 2 → 1 → **1** (compound cap) | Same                         |
| All isolations / cables | unclear        | 3 → 2 → 1 → 0 (failure OK W4)  | Standard double progression  |

Applying the RIR cap to compounds alone should cut Wed recovery cost from 2.5 days to 1–1.5 and fix Fri Pull adherence without any other change.

### 6c. Modest volume bumps — all within current schedule

| Muscle      | Now | W1 | W4 | Action                                                       |
|-------------|-----|----|----|--------------------------------------------------------------|
| Quads       | 6   | 6  | 8  | Hold W1 to validate RIR fix; +1 squat + 1 ext set by W4       |
| Hamstrings  | 3   | 6  | 9  | Add RDL 3 × 8–10 on Wed                                       |
| Glutes      | 0   | 3  | 6  | Add hip thrust 3 × 8–12 on Wed                                |
| Calves      | 3   | 6  | 10 | Add standing calf raise 3 × 10–15 on Wed                      |
| Side delts  | 8   | 10 | 14 | +1 set lateral raise per push day                             |
| Chest       | 12  | 13 | 15 | +1 set incline Smith                                          |
| Rear delts  | 6   | 8  | 12 | +1 set face pull per push day                                 |
| Back, Bi, Tri, FD | hold | hold | hold | All inside productive range; no change                |

Wed session goes from ~12 working sets to ~18 by W4, but with RIR-1 on Smith squats and fresh isolations at standard progression, total fatigue cost should be similar or lower than current.

### 6d. RIR ladder (5-week meso, per-exercise floors applied)

| Week | Muscle target | Compound effort     | Isolation effort     |
|------|---------------|---------------------|----------------------|
| 1    | 3             | RIR 3               | RIR 3                |
| 2    | 2             | RIR 2               | RIR 2                |
| 3    | 1             | RIR 1               | RIR 1                |
| 4    | 0–1           | **RIR 1 (capped)**  | RIR 0 (failure OK)   |
| 5    | 4–5           | Deload              | Deload               |

### 6e. Suggested priority levels for Block 1

- `Specialize`: **none**. Block 1 is recovery-recalibration. Specializing on top confounds the signal.
- `Normal`: every muscle currently trained.
- `Maintenance` / `Off`: none.
- Block 2 will likely specialize quads + hams + calves and add Lower B if Block 1 data shows headroom.

### 6f. Required vs floating sets (the 5-day + Sunday catch-up shape)

Block 1 runs as **5 fixed training days (Mon/Tue/Wed/Fri/Sat) + Thu rest + Sun = rest or catch-up**. The engine evaluates Saturday night and prescribes a Sunday session only if any muscle is below 75% of its weekly target.

Per-exercise priority defaults for Block 1:

| Session | Exercise | Required | Floating |
|---------|----------|----------|----------|
| Pull (Mon, Fri) | Pull-ups | ✓ |  |
| Pull | Chest-supp DB row | ✓ |  |
| Pull | Seated cable row | ✓ |  |
| Pull | Standing DB curls | ✓ |  |
| Pull | Preacher curls |  | ✓ |
| Push (Tue, Sat) | Incline Smith bench | ✓ |  |
| Push | Cable fly | ✓ |  |
| Push | DB OHP | ✓ |  |
| Push | Cable lateral raise | ✓ (first 4 sets) | ✓ (added 5th set) |
| Push | Face pulls | ✓ (first 3 sets) | ✓ (added 4th set) |
| Push | Tricep rope ext | ✓ |  |
| Push | Tricep pushdowns |  | ✓ |
| Legs (Wed) | Smith squats | ✓ |  |
| Legs | RDL (new) | ✓ |  |
| Legs | 45° back extension (new — replaces hip thrust per user pref) |  | ✓ |
| Legs | Quad extension | ✓ |  |
| Legs | Hamstring curls | ✓ |  |
| Legs | Standing calf raise (new) |  | ✓ |
| Legs | Seated calf raise | ✓ |  |

**Hip thrust** remains in the exercise library; swap from back extension any time via settings. Block 1 defaults to back extension because it's lower-friction setup.

Effect on a typical week:

- If you complete every session as prescribed Mon→Sat → Sunday is rest. ✓
- If you skip preacher curls on Mon and Fri because of time → engine sees biceps at ~75% of prescribed → Sunday prescribes ~3 sets of preacher (or substitute curl). The arm catch-up day you were doing intuitively, now planned.
- If you skip Wed standing calf raises → Sunday prescribes 3 sets of standing calf (calves aren't hit anywhere else, so 48h spacing is fine).
- If Wed feels too long, drop the standing calf and one tricep pushdown set → both are floating → both auto-roll into Sunday.

### 6g. What Block 1 success looks like

Engine reads these at the between-block review and decides Block 2:

1. Wed recovery cost drops from 2.5 days to ≤ 1.5 (per logged `soreness` and `perf` on Fri).
2. Fri Pull session completion ≥ 80%.
3. Smith squat working weight does **not** drop despite the RIR cap — should sustain or increase as recovery improves.
4. Quad / ham / calf sets land in prescribed range without crash signals.

If 1–4 hold, Block 2 adds Lower B and specializes lower body. If not, Block 2 investigates further (likely: drop a couple of upper-body sets, hold leg volume, examine recovery factors outside training).

---

## 7. What you need to decide

1. **Accept the compound-RIR cap?** Smith squats / bench / OHP capped at RIR 1, never to failure even in W4.
2. **Accept the in-place Wed leg additions?** RDL, hip thrust, standing calf — added inside the existing Wed session, with one full block to validate before considering Lower B.
3. **Floating-set support for arms?** Lets the app defer prescribed bicep/tricep sets to Sunday only if missed during the week, instead of fixed daily prescriptions.
4. **Specialization cap for future blocks**: default 3 muscles per block — keep, raise to 4, lower to 2?

Frequency increase (Lower B / Sunday legs) is now off the table for Block 1 and will be decided by Block 1's recovery data.

---

## 8. Engine validity check (v4)

The v4 iteration showed the engine doing what it should:

- A real-world recovery signal ("one leg day costs 2.5 days") flipped the diagnosis from "frequency too low" to "RIR too high on compounds."
- The proposal changed from "add a leg day" (high-risk under current conditions) to "fix effort, then measure, then decide" (data-driven).
- The compound-RIR cap is now a permanent rule in the spec (`ALGORITHM.md §6a`), not a one-off.
- Block 1 is reframed as a **diagnostic block** — minimal changes, maximum signal — which is the right move when a recovery alarm is active.

This is also the strongest argument for the app itself: a person trying to manage all of this in their head almost always changes too many things at once. The engine forces the discipline of "one variable per block when a signal is unclear."

The v3 set-counting fix (direct sets only) remains in effect. The v4 RIR-floor rule is additive, not a replacement.
