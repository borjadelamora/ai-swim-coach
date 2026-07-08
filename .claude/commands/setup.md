---
description: Set up the system for a new swimmer, or reconfigure an existing profile.
---

You configure this swim-coaching system for whoever is using it. On a first run you
interview the swimmer and build their `memory/` files (and the swimmer snapshot in
CLAUDE.md) from scratch; on a later run you make targeted changes to an existing profile
without wiping it. The system holds no built-in swimmer — everything personal comes from
this interview, so any swimmer (any level, any pool, any stroke, any goal) can make it
their own.

The swimmer's free-form input (may be empty): $ARGUMENTS

Read CLAUDE.md first — it defines the notation, the memory rules, the schemas, and the
permanent pool/clock constraints. Everything you write must obey it.

## First, detect the mode (safety guard)
Look at `memory/` and decide which mode you are in. Treat a file as "real data" only if it
exists, is non-empty, and is NOT still the unedited template/example.

- **FRESH SETUP** — no real data yet. Run the full interview below and build everything.
- **RECONFIGURE** — already set up (a populated `current_state.md` / `swimmer_profile.md`,
  or a non-empty `sessions.jsonl`). Do NOT wipe and rebuild and do NOT re-run the whole
  interview. State what you found, ask what they want to change (a new goal or phase, a
  different pool/course or schedule, a new active skill, updated benchmarks, more old logs),
  and touch ONLY that, preserving everything else. Never overwrite past lines in
  `sessions.jsonl` — it is append-only. When in doubt, ask rather than overwrite.

The interview and write steps below describe the FRESH SETUP pass in full. In RECONFIGURE
mode, draw on only the parts that match what the swimmer wants to change.

## How to run the interview
Ask in small, logical GROUPS, not one giant wall of questions. Wait for answers before
moving on. Accept "I don't know" / "skip" gracefully: record what's known and leave the
rest as a NAMED data gap. Never fabricate a benchmark, a time, or a date. Restate your
understanding before writing files, and let them correct it.

## What to find out (cover all of these)
1. Identity & level — swimming background, rough level (new / returning / club /
   competitive), years in the water, training solo or with a squad. Age only if offered.
2. POOL & COURSE (important) — the pool length and the unit (metres or yards). Commonly
   25 or 50, but ANY length works — 30 m and 33⅓ m pools exist; a leisure pool is a real
   pool. Record the COURSE: SCM (25 m), SCY (25 yd), LCM (50 m), or "<N>m"/"<N>yd" for a
   nonstandard length. Everything downstream prints and tags this, and rep distances are
   whole lengths of THIS pool. Ask if they ALSO have older logs from a different
   pool/unit — if so, capture that second course for the calibration archive. Also ask their REGION for the
   DATE FORMAT — US = MM/DD/YYYY, else DD/MM/YYYY (default) — and record it as date_format
   in CLAUDE.md's "## Date format".
3. STROKES — which strokes they actually swim (freestyle, backstroke, breaststroke,
   butterfly, individual medley). This frames what the coach can program.
4. Schedule — sessions per week and a typical time budget per session.
5. Strengths & limiters — what comes easily (sprint / distance / kick / pull / a stroke /
   technique) and what holds them back. Tag anything only self-reported as "stated,
   unverified" until the log proves it.
6. THE ONE ACTIVE SKILL PROJECT (important) — the single thing they most want to learn or
   improve right now. The system tracks exactly ONE at a time. Offer suggestions and let
   them pick or name their own:
   - a turn: the freestyle flip turn, backstroke flip, or open turns
   - a stroke to learn or refine: butterfly, breaststroke, backstroke, or freestyle technique
   - starts and dives
   - underwater dolphin kick off the walls
   - a breathing pattern (e.g. bilateral breathing)
   - pacing, negative-splitting, or distance-per-stroke
   Also note any skill they already consider MASTERED, so it is excluded from the targets.
7. Goals — the target (event + distance + date, or a general aim: endurance / speed /
   technique / fitness) and the timeframe. This seeds phase.json.
8. Set-design preferences — warmup/cooldown length, broken vs continuous, descending/ladder
   formats, whether to log kick times every session, etc. Sensible defaults are fine.
9. Health & safety — any current injury or constraint. If they report none, record none —
   do NOT leave it "unknown" or hedge (per the pain/fatigue default in CLAUDE.md).
10. Current benchmarks — any known times (e.g. 50 / 100 free, kick, pull), a CSS estimate,
    longest continuous swim. Capture only what they actually know.
11. Previous logs — ASK whether they have any prior training records (an app export, a
    spreadsheet, a notebook, pasted text). If yes, ingest them (below).

## Ingesting previous logs (only if they have them)
- Parse each past session into the live schema (mirror memory/sessions.example.jsonl and
  the schema in CLAUDE.md): one JSON object per line, FACTS ONLY, in swum order, ids
  incrementing, real dates. Preserve their notation; never invent a time.
- Put sessions in the swimmer's CURRENT course into sessions.jsonl. Put records from an
  OLDER pool/course into archive.jsonl as a frozen, calibration-only baseline (tag the
  course). Keep courses strictly separate — never mix them without the conversion in
  CLAUDE.md.
- If the logs are messy or partial, ingest what is reliable and list the rest as a data
  gap. After ingesting, DERIVE the rest: append rows to TRAINING_LOG.md, record PRs in
  prs.json, and write the resulting benchmarks into current_state.md.

## What to write
Create the real memory files (copy from the `*.template.*` files if present; otherwise
follow the schemas in CLAUDE.md and the example log line). Personalise each. Obey the
memory rules: facts in the log, interpretation in the profile, one fact in one place, lean.
- memory/swimmer_profile.md  — snapshot (level, strokes, course); strengths / weaknesses
  (evidence-tagged, or "stated, unverified"); the ONE active skill project and its current
  limiter; set-design preferences; safety; next-session priority.
- memory/current_state.md    — the COURSE and unit at the top, then the benchmarks they
  gave, each as Standard / Volume / Trend / Next bound; mark genuine unknowns as data gaps.
- memory/phase.json          — an honest starting phase (usually a base build) and a
  transition trigger tied to their goal.
- memory/prs.json            — any records they stated or that fell out of ingested logs,
  each tagged by course.
- memory/TRAINING_LOG.md     — the header plus one row per ingested session (none if they
  had no logs).
- memory/sessions.jsonl      — the ingested current-course sessions, or an empty file.
- memory/planned_session.md  — an empty placeholder ("No active prescription."); /plan overwrites it.
- memory/archive.jsonl       — an older-course archive if they provided one; otherwise leave
  it absent.
- CLAUDE.md "## The swimmer"  — replace the generic pointer with a short personalised
  snapshot: level, course, strokes, the active skill, and the limiter. Keep DETAILED facts
  in memory/, not here. Set "## Date format" date_format from their region. Touch the
  "## Health note" only if they reported a real, current constraint; otherwise leave the
  low-key default.

## Finish
Confirm, one line per file, what you created, populated, or changed — and what you left as
a data gap. On a fresh setup, tell them they are ready and to run `/plan <time budget>` for
their first session; on a reconfigure, confirm only what changed. Do NOT prescribe a set
here — setup only builds and adjusts state.

Once setup is complete, this chat is safe to clear — every command starts fresh from memory/.
