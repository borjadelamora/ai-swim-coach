# The memory model

All of the system's state lives in plain-text files under `memory/`. There is no database and no server. This document describes each file, its role, and the rules that keep the whole thing coherent.

## Source of truth versus derived views

The single most important distinction in the system:

| File | Role | Status | Written by |
| --- | --- | --- | --- |
| `sessions_scm.jsonl` | The live training log — one JSON object per session, in swum order. | **Source of truth** | `/log` (append-only) |
| `archive_scy.jsonl` | A frozen archive from a prior tool, in yards. Calibration reference only. | **Source of truth** (frozen) | never |
| `current_state.md` | Current capacity: benchmarks, bounds, the next allowable step. | Derived | `/log` |
| `swimmer_profile.md` | The durable profile: identity, the active skill project, preferences, safety. | Derived | `/log` |
| `prs.json` | Personal records, tagged by pool. | Derived | `/log` |
| `phase.json` | Periodisation state and the phase transition trigger. | Derived | `/review` (and `/log`) |
| `TRAINING_LOG.md` | A human-readable, one-row-per-session glance view. | Derived | `/log` (append-only) |

The two `.jsonl` logs are the irreplaceable record. Every other file is an *interpretation* that could be rebuilt from them if it drifted or were lost. Three rules follow from this:

1. **The raw line is written first.** `/log` appends and verifies the session object in `sessions_scm.jsonl` before touching any derived file.
2. **Past lines are never edited.** The log is append-only. A correction is a new line, not a mutation of an old one.
3. **The log wins.** If a derived file ever disagrees with the raw log, the log is correct by definition.

## Facts versus interpretation

The raw log holds **facts only**: what was done, in the order swum, with times, efforts, turns, and anything skipped. No inference, no coaching commentary, no narrative.

All **interpretation** — "the limiter has shifted", "self-rating outran output today", "do X next" — lives in `swimmer_profile.md` and `current_state.md`. A single fact lives in exactly one file and is never restated across files.

## Leanness without loss

Every file is kept tight. There is no narrative padding, and stale content is overwritten rather than allowed to accumulate. But "lean" means *tight*, not *lossy*: when a derived file is refreshed, durable facts — benchmarks, calibration corrections, evidence-tagged strengths and weaknesses, stated preferences — are preserved. Only what a new session genuinely supersedes is overwritten.

## Units: two pools, never mixed

Two units coexist by design:

- **`archive_scy.jsonl`** is short-course **yards** (a 25 yd pool). Calibration-only.
- **`sessions_scm.jsonl`** and the live benchmarks are short-course **metres** (a 25 m pool).

A 25 m length is roughly 9% longer than 25 yd, so yard times run about **10–11% faster** than the equivalent metre times. The system tags every figure with its pool and never compares the two without applying the conversion. Live sessions always print the unit (m) explicitly.

## Progression discipline

`current_state.md` records, for each benchmark, a **next bound** — the single allowable next step. A benchmark advances by exactly one variable at a time:

- more **volume** (an extra rep), **or**
- a tighter **goal** time, **or**
- a tighter **send-off**.

Never two at once, and only when the current standard was met cleanly. This keeps progression honest and traceable: each step is a deliberate, single change rather than an optimistic leap.

## The session object schema

Each line of `sessions_scm.jsonl` is one JSON object:

```jsonc
{
  "id": 1,                       // increments by one each session
  "date": "YYYY-MM-DD",          // required; a real date, never null
  "pool": "SCM",
  "duration_min": 45,
  "total_m": 1200,               // sum of metered blocks only
  "rpe": 7,                      // 1-10, or null if not stated
  "rating": 4,                   // 1-5, or null if not stated
  "turns_in_sets": "flip",       // "wall" | "flip" | "mixed"
  "pain": null,                  // null, or a short factual string
  "blocks": [
    {
      "i": 1,                    // order swum
      "kind": "kick",            // warmup|drill|kick|pull|swim|aerobic|sprint|main|cooldown
      "set": "4x25 Kick <:40 @:45",
      "m": 100,                  // metered distance; OR "min" for an unmetered drill
      "goal": "<:40",            // or null
      "times": [38, 39, 39, 40], // or a per-distance object, or null
      "effort": "high",          // easy|steady|high|very high, or null
      "turns": "flip",           // only when notable
      "note": "on the boundary"  // optional, factual
    }
  ],
  "skipped": []                  // prescribed work not done, with the reason
}
```

Rules: `total_m` counts metered blocks only — unmetered drills carry `min`, never invented metres. Ratings record exactly what the swimmer stated; if none was given, the field is `null` rather than a fabricated number, and any inference goes in the profile instead.

## Privacy

A real swimmer's `memory/` files contain their entire performance history and are deliberately **not** committed to this repository — they are listed in [`.gitignore`](../.gitignore). What ships here instead is:

- a `*.template.*` file for each derived file, documenting its schema with placeholders;
- a single synthetic example line in `sessions_scm.example.jsonl` and `archive_scy.example.jsonl`.

To start your own, copy the templates to their real filenames (see the README's *Getting started*) and fill them in. Your data then stays on your machine.
