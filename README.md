# AI Swim Coach

A personal, multi-agent swimming coach built on [Claude Code](https://claude.com/claude-code). It plans each training session before you swim, records what you actually did afterwards, and reviews your progress across the week — behaving like a real, opinionated coach rather than a passive log.

The system is deliberately small, file-based, and transparent. There is no database, no server, and no cloud state: everything the coach knows lives in a handful of plain-text files under `memory/`, and every decision is made by a pipeline of specialised agents whose prompts are version-controlled in this repository.

> **Note on data.** This repository contains the *system* only — the agent definitions, commands, documentation, and empty templates. It deliberately ships with no real training history. The files that would hold an individual's performance data are git-ignored, and the repository provides clearly-marked templates in their place. See [Privacy and data](#privacy-and-data).

---

## Contents

- [Why this exists](#why-this-exists)
- [How it works](#how-it-works)
- [The agent pipeline](#the-agent-pipeline)
- [The knowledge library](#the-knowledge-library)
- [Commands](#commands)
- [Set notation](#set-notation)
- [The memory model](#the-memory-model)
- [Clearing the chat between commands](#clearing-the-chat-between-commands)
- [Design principles](#design-principles)
- [Getting started](#getting-started)
- [Repository layout](#repository-layout)
- [Privacy and data](#privacy-and-data)
- [A note on the advice](#a-note-on-the-advice)
- [Licence](#licence)

---

## Why this exists

Most training apps are spreadsheets with a timer bolted on. They store numbers but hold no opinion: they will not tell you that your last three sessions were all easy, that your turn quality collapses under fatigue, or that the interval you asked for is too soft to produce any adaptation.

This project takes the opposite stance. It treats the athlete's stated feelings as **data, not instructions**, cross-references them against the logged record, and overrides them when the evidence warrants — then explains why. It is built around a single guiding idea: a good coach is firm, expert, and accurately calibrated; never timid, never overshooting.

It is also an exercise in **agent orchestration**. Rather than asking one model to do everything, the work is divided among five narrow specialists, each with a single responsibility, its own tool permissions, and its own model tier. They are composed into deterministic pipelines by the day-to-day commands. This separation is what keeps the coaching honest: the historian may only report facts, the critic may only approve or reject, and no single agent can both invent a number and act on it.

---

## How it works

The athlete interacts with the system through one onboarding command and three recurring ones:

| Command | When | What it does |
| --- | --- | --- |
| `/setup` | First run (and to reconfigure) | Interviews you, ingests any previous logs, and builds your personalised memory files. Guarded on later runs so it will not overwrite a working profile. |
| `/plan` | Before a session | Produces today's set, prescribed and ready to take to the pool. |
| `/log` | After a session | Records what was actually done and updates every derived view. |
| `/review` | Weekly, or at a phase boundary | Assesses load, balance, skill progression, and whether the training phase should change. |

Each of the recurring commands spins up a fresh pipeline of agents. Crucially, every agent reads its context from the files in `memory/` rather than from the chat — so a command run in a brand-new terminal produces exactly the same quality of output as one run mid-conversation. The length or history of the surrounding chat has no bearing on the result.

---

## The agent pipeline

Five agents, each defined in [`.claude/agents/`](.claude/agents). They are intentionally constrained: an agent's tool access reflects exactly what its job requires, and nothing more.

| Agent | Role | Tools | Model tier | Authority |
| --- | --- | --- | --- | --- |
| **Historian** | Reports facts from the log — recent load, set shapes due for variation, skill recency, patterns, and genuine data gaps. Never prescribes. | Read-only | Fast | None — facts only |
| **Physiologist** | Translates recent load, current state, and the periodisation phase into today's target: energy system, training zone, and work-to-rest structure. Sets constraints; does not write the set. | Read-only | Mid | Constrains the coach |
| **Coach** | Writes the actual session within the physiologist's constraints, in the athlete's exact notation. Authoritative — overrides athlete requests when warranted, and explains why. | Read-only | Top | Writes the prescription |
| **Critic** | Reviews the coach's set against the brief and the log, hunting specific failure modes. Returns *approve* or *revise* with blockers. | Read-only | Mid | Veto before the athlete sees the set |
| **Logger** | After a session, writes the raw record first, then refreshes every derived view (records, current state, profile, glance log). | Read / Write | Mid | Owns all writes to memory |

The division of labour is the point. The historian establishes ground truth, the physiologist reasons from it to a target, the coach turns the target into a concrete set, and the critic acts as an independent check before anything reaches the athlete. Writes to memory are funnelled exclusively through the logger, so the source-of-truth record can never be corrupted by an agent whose job was only to read it.

### The planning pipeline (`/plan`)

```
        ┌───────────┐   facts    ┌──────────────┐  constraints  ┌─────────┐
input ─▶│ Historian │ ─────────▶ │ Physiologist │ ────────────▶ │  Coach  │
        └───────────┘            └──────────────┘               └────┬────┘
                                                                     │ proposed set
                                                                     ▼
                                                               ┌──────────┐
                                                               │  Critic  │
                                                               └────┬─────┘
                                                  approve ◀─────────┴─────────▶ revise
                                                     │                            │
                                                     ▼                            ▼
                                              present to athlete        coach edits, re-check
```

The historian and physiologist run **sequentially**, not in parallel: the physiologist reasons from the historian's load summary, so it must see those facts first. The critic loop is bounded — a set is revised at most once or twice before a mechanical fix is applied directly, so the pipeline can never spin indefinitely.

### The logging pipeline (`/log`)

The logger writes in a strict order, and the raw session line lands **first**:

```
raw session line (append-only)  ──▶  glance-view row  ──▶  records  ──▶  current state  ──▶  profile
        source of truth                          ── derived views, rebuildable from the raw log ──
```

If a derived view ever disagrees with the raw log, the log wins.

---

## The knowledge library

Agent prompts define roles and guardrails, but the coaching depth lives in [`knowledge/`](knowledge) — a version-controlled library of swimming method: energy systems and training zones, set architecture (how an intent becomes a structure), breath control and underwater work, turns and walls, and pace/time mathematics. The planning agents read the files their task needs: the physiologist reasons from the energy-system model, the coach designs from the set-design method plus the file matching the day's focus, and the critic recomputes the arithmetic.

The library is deliberately generic — it contains method, never athlete data. The design rule: **knowledge holds how to coach; memory holds who is being coached.** When they disagree, memory wins. This separation is what lets the coach translate a stated focus ("I want to hold my breakouts when tired") into a mechanism, a session structure, and a measurable standard — rather than replaying whatever worked last week.

---

## Commands

Defined in [`.claude/commands/`](.claude/commands). A one-time `/setup` builds your files; the three recurring commands run your training day to day.

### `/setup`

Run on first use to build your system, and again later only when you want to reconfigure it.

**First run** is a thorough, build-from-zero pass. It interviews you — your level and background, your pool and course (length, and metres or yards), the strokes you swim, your weekly schedule, strengths and limiters, goals, the one skill you most want to work on (a turn, a stroke to learn or refine, starts, underwater kick, a breathing pattern, or pacing), any health constraints, and whatever benchmarks you already know — asking in small, logical groups rather than as one long form. If you have **previous training logs** (an app export, a spreadsheet, a notebook, or pasted text), it ingests them: each past session is parsed into the log schema, records are extracted into your personal bests, and your current capacity is derived from the result. Sessions from an older pool or unit are kept as a separate, frozen calibration baseline rather than mixed with live data. It then writes every memory file from the templates, personalised to you, and leaves anything you did not know as a clearly-named data gap rather than a fabricated number.

**Later runs are guarded.** Setup detects that you are already configured, says so, and switches to a conservative reconfigure mode: it asks what you want to change — a new goal or phase, a different pool or schedule, a new active skill, updated benchmarks, or some additional old logs to ingest — and touches only that, preserving everything else and never editing the append-only raw log. So re-running is safe: it will not silently overwrite a working profile. Nothing is ever invented, and setup only builds or adjusts state — it does not prescribe a set.

### `/plan`

Plans today's session. Input is free-form — typically a time budget, optionally with how you feel or what you want to work on ("45 minutes, legs feel heavy", "60 min, I want to hold my breakouts when tired"). The pipeline reads the recent log, translates your stated focus into a physiological target, designs a set from that intent (meaningfully different in structure from recent sessions), computes the session's expected duration so it actually fills your time budget, and runs it past the critic — which recomputes the arithmetic — before presenting it. Stated feelings are weighed against the data; if you ask for an easy day after three easy days, you will be told why today is not easy.

The output is two things: a **readable set** in plain language with effort cues and a short rationale, and a terse **cheatsheet** in the athlete's shorthand to copy onto a waterproof deck card. The approved prescription is also saved to `memory/planned_session.md`, so `/log` can later compare prescribed against done without needing the chat history.

### `/log`

Records the session just completed. The logger reads the saved prescription to work out what was skipped, arithmetic-checks the session line (each block's rep maths must equal its recorded distance, and the blocks must sum to the session total — any mismatch is resolved with the athlete before writing), appends one structured object to the raw log (facts only — what was done, times, efforts, turns, anything skipped, and the athlete's own words on how it felt), verifies it landed, and only then refreshes the derived views. Those notes make the raw log the permanent home for the detail that matters; interpretation is kept strictly out of the raw record and confined to the profile.

### `/review`

Produces a training review over a named period (default: the last seven sessions): total volume, the load trajectory and recovery pattern (judged by load and recovery evidence, not a fixed easy:hard ratio), skill progression, any record movement, and whether a recovery microcycle or a phase change is due. It is read-only over the per-session record; the one file it may change is the periodisation state, and only when a transition is actually warranted.

---

## Set notation

The coach writes sets in a compact, unambiguous shorthand.

Within a block, a line reads:

```
n x distance Stroke <goal @ send-off
```

| Symbol | Meaning | Example |
| --- | --- | --- |
| `n x distance` | `n` repeats of that distance, in your pool's unit | `4x25` — four 25s |
| `Stroke` | the **swim mode**: `Kick` or `Pull`. Plain freestyle is left unlabelled | `4x25 Kick`, `4x50 Pull`, `4x50` (free) |
| `<goal` | the **goal** time — come in under it | `<:40` — under 40 seconds |
| `@ send-off` | the **send-off** — the pace-clock interval you leave on, whatever your time | `@ :45` — leave every 45 s |
| `nB` | breath-controlled rest *instead of* a clock: `n` breaths between reps | `4B` — four breaths' rest |

So `4x25 Kick <:40 @ :45` reads as *four 25s of kick, aim to come in under 40 seconds, leaving every 45 seconds* — while `4x25 Free 4B` is *four 25s of freestyle with four breaths' rest between each* (you choose how long a breath is).

Broken efforts are allowed: a 100 may be written `1x50 + 2x25` — swum as a 50 then two 25s with brief rest, to hold a faster pace than the whole 100 would allow.

Breaks between blocks are marked separately from the within-block send-off:

```
— break —          untimed, athlete's discretion
— break 2:00 —     fixed timed break
— break 2-3 min —  timed range
```

Two **permanent** format rules encode the geometry of the pool and the pace clock:

- **Send-offs are multiples of :15** (`:15 / :30 / :45 / 1:00 / 1:15 …`), because the clock is tracked in fifteen-second increments.
- **Every block finishes on the start wall.** Each block's total must be an *even number of lengths*, so the swimmer is never stranded at the far wall during a break. That is the only wall constraint — any rep distance is legal, including odd-length reps, when the count keeps the block total even: in a 25 m pool `2x75` works (six lengths) while `3x75` and `5x25` do not.

These constraints are enforced by the critic, not left to chance.

---

## The memory model

All state is plain text under [`memory/`](memory).

There is a strict distinction between the **source of truth** and **derived views**:

| File | Role | Status |
| --- | --- | --- |
| `sessions.jsonl` | The live training log — one JSON object per session, append-only, facts only. | **Source of truth** |
| `archive.jsonl` | An optional frozen baseline of older logs, kept for calibration only. | **Source of truth** (frozen) |
| `current_state.md` | Current capacity: benchmarks, bounds, and the next allowable progression. | Derived |
| `swimmer_profile.md` | The durable athlete profile: identity, the active skill project, set-design preferences, safety notes. | Derived |
| `prs.json` | Personal records. | Derived |
| `phase.json` | Periodisation state — the current training phase and its transition trigger. | Derived |
| `TRAINING_LOG.md` | A human-readable, one-row-per-session glance view. | Derived |
| `planned_session.md` | The latest `/plan` prescription, overwritten each run; `/log` reads it to compare prescribed vs done, then marks it consumed. | Ephemeral |

The raw logs are the irreplaceable record; every derived view can be rebuilt from them if it drifts or is lost. This is why the logger always writes the raw line first, arithmetic-checks it (block maths must equal the recorded distances), and never edits a past line — corrections are appended `amends` objects.

Your **course** (pool length and unit) is chosen at setup: short-course metres (25 m), short-course yards (25 yd), long-course metres (50 m) — or any other length; 30 m outdoor pools are real pools, and rep distances simply become whole lengths of whatever you swim in. Every figure is tagged with its course, and each logged session records its pool, so even a mid-log pool change stays unambiguous. If you import older logs from a different course, they are kept as a separate, frozen archive and never compared with live data without applying the conversion — a small but realistic detail that prevents a whole class of silent errors.

---

## Clearing the chat between commands

Every command rebuilds its own context from the files in `memory/`. The agents do not read the conversation above them, so once a command has finished and saved its work, the chat history serves no further purpose. Leaving it to grow is in fact harmful: the more text the model must take in each turn, the more thinly its attention is spread, and earlier detail is diluted. Starting each command from an empty chat keeps every run as sharp as the first.

So after any command that writes to `memory/`, clear the conversation before running the next one:

1. Run `/setup`, `/log`, or `/review`, and wait for it to confirm what it wrote.
2. Run `/clear`. This command is built into Claude Code: it empties the conversation but leaves your files untouched.
3. Run the next command. It reads the current state from `memory/` and begins clean.

`/plan` saves its prescription to `memory/planned_session.md` and touches nothing else, so there is nothing to lose by clearing after it as well. The natural moment to clear is at the end of a session's cycle, once `/log` has saved the work. The commands themselves remind you of this on completion.

---

## Design principles

A few ideas run through the whole system and are worth calling out, because they are what make it behave like a coach rather than a logger.

**Feelings are data, not commands.** What the athlete reports is evidence to be weighed against the record, not an instruction to be obeyed. The coach overrides when the data warrants and explains the reasoning, because informed athletes train better.

**Separate what is proven from what is suspected.** The analysis agents are disciplined about epistemic honesty: they distinguish what the clock *proved* (measured times and distances), from what the data *suggests* (a trend the evidence supports but has not confirmed), from what genuinely *cannot be pinned down yet*. A thing seen once is an observation; a thing seen every session is a pattern — and the system refuses to harden the former into the latter prematurely. This discipline governs analysis only; the prescription the athlete reads is delivered with full conviction and no hedging.

**One fact, one place.** Every file is kept lean. A fact lives in exactly one file; nothing is restated across files; stale information is overwritten rather than accumulated. The raw log holds the history; the derived views hold only the *current* read and never replay it session by session, so they stay bounded as the log grows. "Lean" means tight, not lossy — hard-won facts such as benchmarks and calibration corrections are preserved on every rewrite.

**Measure what you are building.** A subtle failure mode is training a limiter in an untimed format — breath-controlled sets, drills — and so never measuring whether it is improving. The system guards against it: the historian tracks how long the focus quality has gone unmeasured, and the physiologist periodically calls for a measurable benchmark, keeping a phase's progress and its transition trigger testable rather than a matter of feel.

**Self-contained commands.** Each command reads its state from `memory/`, never from the chat history. Agents are spawned fresh and read the files, so the system is reproducible and stateless from the user's point of view.

**Calibrated, never timid.** The coaching is prescribed at the genuine edge of demonstrated capacity. The system explicitly guards against both failure modes: padding sets with junk volume, and overshooting into work the athlete cannot complete.

---

## Getting started

### Prerequisites

- [Claude Code](https://claude.com/claude-code) installed and configured.
- A terminal opened in this repository's directory.

### Set-up

1. **Clone the repository** and open it in your terminal.

2. **Run setup.** This is the recommended path — it interviews you, ingests any previous logs, and writes all of your memory files for you:

   ```
   /setup
   ```

   Setup creates your private, git-ignored data files from the supplied templates and personalises them to you. If you have prior training records, have them to hand (a file path or text to paste) and setup will format them into the log.

3. **Plan your first session:**

   ```
   /plan 45 minutes, feeling fresh
   ```

4. **After you swim, log it:**

   ```
   /log felt good, kick set on the boundary, all turns clean
   ```

5. **At the end of the week, review:**

   ```
   /review
   ```

### Setting up by hand (optional)

If you would rather not use `/setup`, copy the templates to their real filenames and edit them yourself. The real data files are git-ignored, so these copies stay private:

```sh
cp memory/swimmer_profile.template.md  memory/swimmer_profile.md
cp memory/current_state.template.md    memory/current_state.md
cp memory/phase.template.json          memory/phase.json
cp memory/prs.template.json            memory/prs.json
cp memory/TRAINING_LOG.template.md     memory/TRAINING_LOG.md
touch memory/sessions.jsonl
touch memory/planned_session.md
```

The templates are documented inline and show the exact schema each file expects.

---

## Repository layout

```
ai-swim-coach/
├── README.md                     This file.
├── CLAUDE.md                     System specification — the rules every agent obeys.
├── .gitignore                    Keeps real training data out of version control.
├── .claude/
│   ├── agents/                   The five specialist agents.
│   │   ├── historian.md
│   │   ├── physiologist.md
│   │   ├── coach.md
│   │   ├── critic.md
│   │   └── logger.md
│   └── commands/                 The slash-commands.
│       ├── setup.md
│       ├── plan.md
│       ├── log.md
│       └── review.md
├── knowledge/                    The coaching method library (generic — no athlete data).
│   ├── energy-systems.md
│   ├── set-design.md
│   ├── breath-underwater.md
│   ├── turns-walls.md
│   └── pace-and-time.md
└── memory/
    ├── *.template.*              Documented, empty templates.
    ├── sessions.example.jsonl    One synthetic example session.
    └── archive.example.jsonl     One synthetic example archive line.
```

---

## Privacy and data

This repository is the **system**, not anyone's training record. That separation is enforced, not merely intended:

- **No real performance data is included.** The files that would hold an individual's history, benchmarks, and records are listed in [`.gitignore`](.gitignore) and are never committed.
- **Templates ship in their place.** Every memory file has a `*.template.*` counterpart that documents its schema with placeholder values only.
- **One example, clearly synthetic.** `memory/sessions.example.jsonl` contains a single fabricated session purely to illustrate the log schema. It is not real.

If you adopt this system, your own `memory/` data stays on your machine and out of git by default. Review `.gitignore` before committing if you change the file layout.

The full system specification — notation, memory rules, and the discipline every agent obeys — lives in [`CLAUDE.md`](CLAUDE.md).

---

## A note on the advice

This is an AI coach. Like any generative system, its output is not guaranteed to be correct, and it should be read as informed guidance rather than the final word. It is built to earn trust the honest way — it reasons from your own logged record, calibrates to the capacity you have actually demonstrated, checks each set against its own constraints, and is careful to separate what it has measured from what it is only inferring. Used as intended — to generate sessions, keep an honest log, and think through your training — it is a genuinely useful training partner.

That said, it does not see you swim, and it cannot weigh everything an experienced coach standing on the deck can. Listen to your body, apply your own judgement, and for anything involving pain, injury, or health, treat a qualified coach or clinician as the authority. If their guidance and the app's ever disagree, follow theirs.

---

## Licence

Released under the [MIT Licence](LICENSE) — you are free to use, modify, and distribute this work, provided the copyright notice is retained. See the [`LICENSE`](LICENSE) file for the full text.
