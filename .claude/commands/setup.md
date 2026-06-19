---
description: First-run setup. Interview the swimmer and build their memory files.
---

You are running the one-time setup for this swim-coaching system. Your job is to interview
the swimmer, then populate `memory/` (and the swimmer snapshot in CLAUDE.md) so that
`/plan`, `/log`, and `/review` work for them from the very next session.

The swimmer's free-form input (may be empty): $ARGUMENTS

Read CLAUDE.md first — it defines the notation, the memory rules, the schemas, and the
permanent pool/clock constraints. Everything you write must obey it.

## How to run the interview
- Ask in small, logical GROUPS, not one giant wall of questions. Wait for answers before
  moving on. Keep it conversational and efficient — onboarding a fit, motivated swimmer,
  not a medical intake.
- Accept "I don't know" / "skip" gracefully: record what's known and leave the rest as a
  NAMED data gap. Never fabricate a benchmark, a time, or a date.
- Briefly restate your understanding before you write any files, and let them correct it.
- Default the pace-clock and finish-on-the-start-wall rules from CLAUDE.md silently; don't
  quiz the swimmer on pool geometry.

## What to find out (cover all of these)
1. Identity & level — swimming background, rough level (new / returning / club /
   competitive), years in the water, training solo or with a squad. Age only if offered.
2. Pool & units — pool length and units: 25 m, 25 yd, or 50 m. This fixes whether the live
   log is metres or yards and the unit every session prints. If they have BOTH a current
   pool and an older record in a different unit, capture both.
3. Schedule — sessions per week and a typical time budget per session.
4. Strengths & limiters — what comes easily (sprint / distance / kick / pull / technique)
   and what holds them back. These seed the profile's evidence-tagged lists; tag anything
   only self-reported as "stated, unverified" until the log proves it.
5. Goals — the target (event + distance + date if there is one, or a general aim:
   endurance / speed / technique / fitness) and the timeframe. This seeds phase.json.
6. The ONE active skill project — the single technique to work on now (e.g. flip turns,
   bilateral breathing, starts, underwaters). The system tracks exactly one at a time.
   Also note any skill already MASTERED, so it is excluded from the targets.
7. Set-design preferences — anything they care about: warmup/cooldown length, broken vs
   continuous, descending/ladder formats, whether to log kick times every session, etc.
   Sensible defaults are fine where they have no opinion.
8. Health & safety — any current injury or constraint. If they report none, record none —
   do NOT leave it "unknown" or hedge on it (per the pain/fatigue default in CLAUDE.md).
9. Current benchmarks — any known times (e.g. 50 / 100 free, kick, pull), a CSS estimate,
   longest continuous swim. Capture only what they actually know.
10. Previous logs — ASK whether they have any prior training records (an app export, a
    spreadsheet, a notebook, pasted text, a coach's plan). If yes, ingest them (below).

## Ingesting previous logs (only if they have them)
- Parse each past session into the live schema (mirror memory/sessions_scm.example.jsonl
  and the schema in CLAUDE.md): one JSON object per line, FACTS ONLY, in swum order, ids
  incrementing, real dates. Preserve their notation where you can; never invent a time.
- Put sessions in the swimmer's CURRENT unit/pool into sessions_scm.jsonl. Put records from
  an OLDER pool/unit into archive_scy.jsonl as a frozen, calibration-only baseline (tag the
  unit). Keep the two strictly separate — never mix yards and metres without the ~10-11%
  conversion in CLAUDE.md.
- If the logs are messy or partial, ingest what is reliable and list the rest as a data
  gap. Do not guess at missing fields.
- After ingesting, DERIVE the rest: append matching rows to TRAINING_LOG.md, record any
  PRs in prs.json, and write the resulting benchmarks into current_state.md.

## What to write (mirror how an experienced user has tuned this system)
Create the real memory files (copy from the `*.template.*` files if present; otherwise
follow the schemas in CLAUDE.md and the example log line). Personalise each to the swimmer.
Obey the memory rules: facts in the log, interpretation in the profile, one fact in one
place, lean files, no padding.
- memory/swimmer_profile.md  — snapshot; strengths / weaknesses (evidence-tagged, or
  "stated, unverified"); the ONE skill project; set-design preferences; safety;
  next-session priority.
- memory/current_state.md    — the benchmarks they gave, each as Standard / Volume / Trend
  / Next bound; mark genuine unknowns as data gaps.
- memory/phase.json          — an honest starting phase (usually a base build) and a
  transition trigger tied to their goal.
- memory/prs.json            — any records they stated or that fell out of ingested logs,
  each tagged by pool.
- memory/TRAINING_LOG.md     — the header plus one row per ingested session (none if they
  had no logs).
- memory/sessions_scm.jsonl  — the ingested current-pool sessions, or an empty file.
- memory/archive_scy.jsonl   — an older-pool archive if they provided one; otherwise leave
  it absent.
- CLAUDE.md "## The swimmer"  — replace the generic pointer with a short personalised
  snapshot (level, strengths, the active skill, the limiter), the way a tuned setup reads.
  Keep DETAILED facts in memory/, not here. Touch the "## Health note" only if they
  reported a real, current constraint; otherwise leave the low-key default.

## Finish
Confirm, one line per file, what you created or populated and what you left as a data gap.
Then tell them they are ready, and to run `/plan <time budget>` for their first session.
Do NOT prescribe a set here — setup only builds state.
