# Pace math, pool conversion & the session time model

Reference for the Coach and Critic (both MUST apply the time model every /plan) and the
Physiologist (tests, conversions). GENERIC method — the swimmer's actual paces and any
validated conversion live in current_state.md and win on conflict.

## Think per-length, print per-rep
Internally reason in seconds per LENGTH at a named effort — it survives pool changes and
odd rep distances. A rep's expected time = lengths × per-length pace (minus ~1s on the
first length for the push start). Print times per rep in the swimmer's notation.

## Converting times between pool lengths
Per-length swim time scales with the length; each wall is worth a saving (push-off +
glide beat swimming) of roughly 0.5-1.0s per turn.
- Operational rule: T_new ≈ T_old × (L_new / L_old) per length, then round UP to the
  swimmer's clock granularity on first exposure. The lost-wall correction going to a
  longer pool (fewer turns per 100) is under ~1% — real, but smaller than send-off
  rounding; fold it into the round-up.
- Converted numbers are PROVISIONAL: valid for seeding goals, replaced by measured times
  as soon as swims land in the new pool. Never advance a bound off a converted number.
- Never compare raw times across pool lengths in analysis — convert or speak per-length.

## Goal & send-off construction
- goal = anchored benchmark pace under MATCHED conditions (fatigue state, overlay, rep
  length) ± the one progression dial being moved. No anchor = first probe, eased.
- send-off = goal + intended rest, snapped to the :15 grid. Round rest UP on first
  exposures and overlay work; rest defines the zone (energy-systems.md), so pick the
  rest first and let the send-off follow — not the reverse.
- Rest reference: breath-count rest ≈ 4s per breath (a calm breathing cycle) — so 8B is
  ~30s, 12B ~50s. Working estimate; calibrate from logged durations when they disagree.

## Test protocols (run one when a gate or a new pool demands data)
- Two-distance CSS test: two honest max-effort swims, short (≈2-4 lengths) and long
  (≈8-16 lengths), same day, full recovery between. CSS pace = (D_long − D_short) /
  (T_long − T_short). Distances flex to pool geometry; record conditions.
- Timed gate piece: one measured swim mid-session under DEFINED conditions (effort,
  overlay, position in session) — the repeatable low-cost benchmark.
- Broken benchmark: fixed splits + fixed brief rest (e.g. 4-length swim as 2x2-length
  with 10s) — measures pace capacity beyond current continuous range.
- New pool: the first 1-2 sessions are calibration-first — moderate-effort timed swims
  across the working strokes/modes to replace converted numbers with measured ones.
  Benchmarks migrate by measurement, not conversion.

## THE SESSION TIME MODEL — compute it, never eyeball it
The historical failure mode is UNDER-FILLING: sessions that compute short leave the
swimmer inventing work at the pool. The plan must fill the stated budget.

Per block:
- Clocked block: (n−1) × send-off + last-rep goal time.
- Breath-rest (nB) block: n × rep_time + (n−1) × (B × ~4s).
- Broken pieces: swim parts + declared inner rests.
- Drills: their stated minutes. Untimed easy swims: distance × easy pace (use the
  swimmer's logged easy pace; if unmeasured, aerobic pace + ~15% and say it's estimated).
- Breaks between blocks: as written if timed; untimed break ≈ 1:30 each.
- Session overhead (getting in, first push-off, gear shuffles): +2-3 min once.

Sum the blocks + breaks + overhead = computed duration. Print it in the session header.
Acceptance: computed duration ∈ [budget − 5 min, budget + 2 min]. Under by more →
add purposeful work (not filler); over → trim the lowest-value block, never the purpose.
The Critic recomputes independently; a plan outside the window is a blocker.

Calibration loop: the log records actual duration_min. When actual vs computed drifts
≥10% across sessions, adjust the constants (break length, breath seconds, easy pace) in
current_state.md — the model is calibrated to THIS swimmer, not defended.
