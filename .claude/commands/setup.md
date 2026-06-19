---
description: Set up the system for a new swimmer, or reconfigure an existing profile.
---

You configure this swim-coaching system. On a first run you interview the swimmer and build
their `memory/` files (and the swimmer snapshot in CLAUDE.md) from scratch; on a later run
you make targeted changes to an existing profile without wiping it. Either way, the goal is
that `/plan`, `/log`, and `/review` work correctly afterwards.

The swimmer's free-form input (may be empty): $ARGUMENTS

Read CLAUDE.md first — it defines the notation, the memory rules, the schemas, and the
permanent pool/clock constraints. Everything you write must obey it.

## First, detect the mode (safety guard)
Look at `memory/` and decide which mode you are in. Treat a file as "real data" only if it
exists, is non-empty, and is NOT still the unedited template/example.

- **FRESH SETUP** — no real data yet (the data files are absent, empty, or still the
  templates). Run the full interview below and build everything. This is the thorough,
  build-from-zero pass.

- **RECONFIGURE** — the swimmer is already set up (a populated `current_state.md` /
  `swimmer_profile.md`, or a non-empty `sessions_scm.jsonl`). Do NOT wipe and rebuild, and
  do NOT re-run the whole interview. Instead:
  1. State plainly what you found (e.g. "You already have a profile and N logged sessions")
     so it is obvious this is not a clean install.
  2. Ask what they want to change — typically one of: a new goal or training phase, a
     different pool / units / schedule, a new active skill project, updated benchmarks, or
     ingesting additional past logs.
  3. Touch ONLY what they ask for. Preserve all existing data and durable nuance
     (benchmarks, calibration corrections, evidence-tagged strengths/weaknesses, stated
     preferences) per the memory rules in CLAUDE.md. Confirm before each write.
  4. NEVER overwrite or rewrite past lines in `sessions_scm.jsonl` — it is append-only.
     Additional logs are appended with continuing ids; the raw record is never edited.
  This pass is deliberately conservative: re-configuration is welcome, wholesale
  replacement of a working profile is not. When in doubt, ask rather than overwrite.

The interview and write steps below describe the FRESH SETUP pass in full. In RECONFIGURE
mode, draw on only the parts that match what the swimmer wants to change.

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
Confirm, one line per file, what you created, populated, or changed — and what you left as
a data gap. On a fresh setup, tell them they are ready and to run `/plan <time budget>` for
their first session; on a reconfigure, confirm only what changed and note that the rest was
left intact. Do NOT prescribe a set here — setup only builds and adjusts state.
