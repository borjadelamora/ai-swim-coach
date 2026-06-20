# Swimmer profile (durable) — TEMPLATE

> Copy this file to `swimmer_profile.md` and replace every placeholder with your own
> details. This is the durable, slow-changing view: identity, tendencies, the one active
> skill project, set-design preferences, and safety. The current NUMBERS live in
> `current_state.md`, not here — do not duplicate benchmarks across the two.

last_updated: YYYY-MM-DD
role: Slow-changing identity, tendencies, the active skill project, set-design preferences,
and safety. For current numbers (benchmarks, bounds) read current_state.md, not this file.

memory map:
- current_state.md → capacity now (numbers + bounds). Logger rewrites it.
- sessions.jsonl   → live log, in your configured course. Append-only, facts only.
- archive.jsonl    → OPTIONAL frozen older baseline. Calibration-only; never edit.
- TRAINING_LOG.md  → human glance view (one row per session).
- prs.json, phase.json → records and periodisation.

## Snapshot
<One or two lines: who the swimmer is, their level, the course they train in, the strokes
they swim, their broad strengths, and the single biggest limiter to build. Name the ONE
active skill project. Note any skill already mastered and therefore retired from the targets.>

## Strengths (evidence)
- <Strength> — <the measured evidence that supports it, with a date>.
- <Strength> — <evidence>.

## Weaknesses (evidence)
- <Limiter> — <evidence; self-identified and/or shown in the data>.
- <Limiter> — <evidence>.

## Calibration corrections (coach-owned errors, encode permanently)
1. <A mistake the coach made about this swimmer and the correct rule going forward.>
2. <…>

## <Active skill> — the active project
- Status (YYYY-MM-DD): <what is confirmed vs still open, with how much evidence backs it>.
- Current limiter: <the one element being coached now>.
- Method / cue: <the swimmer's own working method, if any>.
- Next: <the next drill or progression>.

## Set-design preferences (how to program for them)
1. Send-offs: :15 multiples only. Never odd intervals.
2. <e.g. lean warmup/cooldown — no long easy padding>.
3. <e.g. varied, punchy, purposeful — every block holds a purpose and a difficulty level>.
4. <e.g. don't repeat recent sessions; vary reps, distances, and formats>.
5. <e.g. how to build the aerobic base — broken/nB sets at real pace, not big continuous swims>.
6. <Technique-by-design preferences.>
7. Finish on the starting wall — every block's total is a multiple of two pool lengths
   (permanent rule in CLAUDE.md).

## Response to intensity
<How the swimmer responds to load: where the ceiling is, how they recover, whether their
self-rating tracks their output, how they handle de-load days.>

## Safety (low-key — do not over-weight)
<Any genuinely current health constraint, kept brief. If there is none, say so. Universal
rule only: don't train through genuine pain; if something persists, see a clinician.>

## Next-session priority
<The single most important focus for the next session, with the specific targets per
discipline (skill / kick / pull / aerobic / sprint).>
