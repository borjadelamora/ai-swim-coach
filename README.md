# AI Swim Coach

A personal, multi-agent swimming coach built on [Claude Code](https://claude.com/claude-code). It plans each training session before you swim, records what you actually did afterwards, and reviews your progress across the week — behaving like a real, opinionated coach rather than a passive log.

The system is deliberately small, file-based, and transparent. There is no database, no server, and no cloud state: everything the coach knows lives in a handful of plain-text files under `memory/`, and every decision is made by a pipeline of specialised agents whose prompts are version-controlled in this repository.

> **Note on data.** This repository contains the *system* only — the agent definitions, commands, documentation, and empty templates. It deliberately ships with no real training history. The files that would hold an individual's performance data are git-ignored, and the repository provides clearly-marked templates in their place. See [Privacy and data](#privacy-and-data).

---

## Contents

- [Why this exists](#why-this-exists)
- [How it works](#how-it-works)
- [The agent pipeline](#the-agent-pipeline)
- [Commands](#commands)
- [Set notation](#set-notation)
- [The memory model](#the-memory-model)
- [Design principles](#design-principles)
- [Getting started](#getting-started)
- [Repository layout](#repository-layout)
- [Privacy and data](#privacy-and-data)
- [Documentation](#documentation)
- [Licence](#licence)

---

## Why this exists

Most training apps are spreadsheets with a timer bolted on. They store numbers but hold no opinion: they will not tell you that your last three sessions were all easy, that your turn quality collapses under fatigue, or that the interval you asked for is too soft to produce any adaptation.

This project takes the opposite stance. It treats the athlete's stated feelings as **data, not instructions**, cross-references them against the logged record, and overrides them when the evidence warrants — then explains why. It is built around a single guiding idea: a good coach is firm, expert, and accurately calibrated; never timid, never overshooting.

It is also an exercise in **agent orchestration**. Rather than asking one model to do everything, the work is divided among five narrow specialists, each with a single responsibility, its own tool permissions, and its own model tier. They are composed into deterministic pipelines by three slash-commands. This separation is what keeps the coaching honest: the historian may only report facts, the critic may only approve or reject, and no single agent can both invent a number and act on it.

---

## How it works

The athlete interacts with the system through three commands:

| Command | When | What it does |
| --- | --- | --- |
| `/plan` | Before a session | Produces today's set, prescribed and ready to take to the pool. |
| `/log` | After a session | Records what was actually done and updates every derived view. |
| `/review` | Weekly, or at a phase boundary | Assesses load, balance, skill progression, and whether the training phase should change. |

Each command spins up a fresh pipeline of agents. Crucially, every agent reads its context from the files in `memory/` rather than from the chat — so a command run in a brand-new terminal produces exactly the same quality of output as one run mid-conversation. The length or history of the surrounding chat has no bearing on the result.

---

## The agent pipeline

Five agents, each defined in [`.claude/agents/`](.claude/agents). They are intentionally constrained: an agent's tool access reflects exactly what its job requires, and nothing more.

| Agent | Role | Tools | Model tier | Authority |
| --- | --- | --- | --- | --- |
| **Historian** | Reports facts from the log — recent load, set shapes due for variation, skill recency, patterns, and genuine data gaps. Never prescribes. | Read-only | Fast | None — facts only |
| **Physiologist** | Translates recent load and current state into today's target: energy system, training zone, and work-to-rest structure. Sets constraints; does not write the set. | Read-only | Mid | Constrains the coach |
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

## Commands

Defined in [`.claude/commands/`](.claude/commands). Full detail in [`docs/COMMANDS.md`](docs/COMMANDS.md).

### `/plan`

Plans today's session. Input is free-form — typically a time budget, optionally with how you feel ("45 minutes, legs feel heavy"). The pipeline reads the recent log, sets a physiological target, writes a set that is meaningfully different in structure from recent sessions, and runs it past the critic before presenting it. Stated feelings are weighed against the data; if you ask for an easy day after three easy days, you will be told why today is not easy.

The output is two things: a **readable set** in plain language with effort cues and a short rationale, and a terse **cheatsheet** in the athlete's shorthand to copy onto a waterproof deck card.

### `/log`

Records the session just completed. The logger appends one structured object to the raw log (facts only — what was done, times, efforts, turns, anything skipped), verifies it landed, and only then refreshes the derived views. Interpretation is kept strictly out of the raw record and confined to the profile.

### `/review`

Produces a training review over a named period (default: the last seven sessions): total volume, the easy-to-hard balance across the week, skill progression, any record movement, and whether a recovery microcycle or a phase change is due. It is read-only over the per-session record; the one file it may change is the periodisation state, and only when a transition is actually warranted.

---

## Set notation

The coach writes sets in a compact, unambiguous shorthand. Full specification in [`docs/SET-NOTATION.md`](docs/SET-NOTATION.md).

Within a block:

```
n x distance Stroke <goal @ send-off
```

- `<` is the **goal** time — come in under it.
- `@` is the **send-off** — the interval on the pace clock; leave on it.
- `nB` denotes breath-controlled rest instead of a clock (e.g. `4x25 Free 4B` is four 25s with four breaths' rest between each).
- Broken efforts are allowed: a 100 may be written `1x50 + 2x25`.

For example:

```
4x25 Kick <:40 @ :45
```

reads as *four 25s of kick, aim to come in under 40 seconds, leaving every 45 seconds.*

Breaks between blocks are marked separately from the within-block send-off:

```
— break —          untimed, athlete's discretion
— break 2:00 —     fixed timed break
— break 2-3 min —  timed range
```

Two **permanent** format rules encode the geometry of the pool and the pace clock:

- **Send-offs are multiples of :15** (`:15 / :30 / :45 / 1:00 / 1:15 …`), because the clock is tracked in fifteen-second increments.
- **Every block finishes on the start wall.** In a 25 m pool this means each block's total distance is a multiple of 50 m, so the swimmer is never stranded at the far wall during a break.

These constraints are enforced by the critic, not left to chance.

---

## The memory model

All state is plain text under [`memory/`](memory). Full detail in [`docs/MEMORY-MODEL.md`](docs/MEMORY-MODEL.md).

There is a strict distinction between the **source of truth** and **derived views**:

| File | Role | Status |
| --- | --- | --- |
| `sessions_scm.jsonl` | The live training log — one JSON object per session, append-only, facts only. | **Source of truth** |
| `archive_scy.jsonl` | A frozen archive from a prior tool, kept for calibration only. | **Source of truth** (frozen) |
| `current_state.md` | Current capacity: benchmarks, bounds, and the next allowable progression. | Derived |
| `swimmer_profile.md` | The durable athlete profile: identity, the active skill project, set-design preferences, safety notes. | Derived |
| `prs.json` | Personal records. | Derived |
| `phase.json` | Periodisation state — the current training phase and its transition trigger. | Derived |
| `TRAINING_LOG.md` | A human-readable, one-row-per-session glance view. | Derived |

The raw logs are the irreplaceable record; every derived view can be rebuilt from them if it drifts or is lost. This is why the logger always writes the raw line first and never edits a past line.

Two units coexist by design. The frozen archive is in short-course **yards**; live training is in short-course **metres**. A 25 m length is roughly 9% longer than 25 yd, so yard times run about 10–11% faster than the equivalent metre times. The system keeps the two explicitly tagged and never compares them without applying the conversion — a small but realistic detail that prevents a whole class of silent errors.

---

## Design principles

A few ideas run through the whole system and are worth calling out, because they are what make it behave like a coach rather than a logger.

**Feelings are data, not commands.** What the athlete reports is evidence to be weighed against the record, not an instruction to be obeyed. The coach overrides when the data warrants and explains the reasoning, because informed athletes train better.

**Separate what is proven from what is suspected.** The analysis agents are disciplined about epistemic honesty: they distinguish what the clock *proved* (measured times and distances), from what the data *suggests* (a trend the evidence supports but has not confirmed), from what genuinely *cannot be pinned down yet*. A thing seen once is an observation; a thing seen every session is a pattern — and the system refuses to harden the former into the latter prematurely. This discipline governs analysis only; the prescription the athlete reads is delivered with full conviction and no hedging.

**One fact, one place.** Every file is kept lean. A fact lives in exactly one file; nothing is restated across files; stale information is overwritten rather than accumulated. "Lean" means tight, not lossy — hard-won facts such as benchmarks and calibration corrections are preserved on every rewrite.

**Self-contained commands.** Each command reads its state from `memory/`, never from the chat history. Agents are spawned fresh and read the files, so the system is reproducible and stateless from the user's point of view.

**Calibrated, never timid.** The coaching is prescribed at the genuine edge of demonstrated capacity. The system explicitly guards against both failure modes: padding sets with junk volume, and overshooting into work the athlete cannot complete.

---

## Getting started

### Prerequisites

- [Claude Code](https://claude.com/claude-code) installed and configured.
- A terminal opened in this repository's directory.

### Set-up

1. **Clone the repository** and open it in your terminal.

2. **Initialise your memory files** from the supplied templates. The real data files are git-ignored, so you create your own private copies:

   ```sh
   cp memory/swimmer_profile.template.md  memory/swimmer_profile.md
   cp memory/current_state.template.md    memory/current_state.md
   cp memory/phase.template.json          memory/phase.json
   cp memory/prs.template.json            memory/prs.json
   cp memory/TRAINING_LOG.template.md     memory/TRAINING_LOG.md
   touch memory/sessions_scm.jsonl
   ```

3. **Fill in your profile.** Edit `memory/swimmer_profile.md` and `memory/current_state.md` with your own identity, benchmarks, preferences, and the one skill you are actively working on. These drive every prescription.

4. **Plan your first session:**

   ```
   /plan 45 minutes, feeling fresh
   ```

5. **After you swim, log it:**

   ```
   /log felt good, kick set on the boundary, all turns clean
   ```

6. **At the end of the week, review:**

   ```
   /review
   ```

The supplied templates are documented inline and show the exact schema each file expects.

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
│   └── commands/                 The three slash-commands.
│       ├── plan.md
│       ├── log.md
│       └── review.md
├── docs/
│   ├── ARCHITECTURE.md           The pipelines and the reasoning behind them.
│   ├── COMMANDS.md               Detailed behaviour of each command.
│   ├── SET-NOTATION.md           The full set-notation specification.
│   └── MEMORY-MODEL.md           File roles, source-of-truth rules, schemas.
└── memory/
    ├── *.template.*              Documented, empty templates.
    └── sessions_scm.example.jsonl  One synthetic example session.
```

---

## Privacy and data

This repository is the **system**, not anyone's training record. That separation is enforced, not merely intended:

- **No real performance data is included.** The files that would hold an individual's history, benchmarks, and records are listed in [`.gitignore`](.gitignore) and are never committed.
- **Templates ship in their place.** Every memory file has a `*.template.*` counterpart that documents its schema with placeholder values only.
- **One example, clearly synthetic.** `memory/sessions_scm.example.jsonl` contains a single fabricated session purely to illustrate the log schema. It is not real.

If you adopt this system, your own `memory/` data stays on your machine and out of git by default. Review `.gitignore` before committing if you change the file layout.

---

## Documentation

| Document | Subject |
| --- | --- |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | The agent pipelines, sequencing, and the reasoning behind the design. |
| [`docs/COMMANDS.md`](docs/COMMANDS.md) | The full behaviour of `/plan`, `/log`, and `/review`. |
| [`docs/SET-NOTATION.md`](docs/SET-NOTATION.md) | The complete set-notation grammar, with examples. |
| [`docs/MEMORY-MODEL.md`](docs/MEMORY-MODEL.md) | Every memory file, its role, its schema, and the source-of-truth rules. |
| [`CLAUDE.md`](CLAUDE.md) | The system specification that governs all agent behaviour. |

---

## Licence

No licence has been chosen yet, so all rights are reserved by default. The code is published for reference and review; reuse terms will be decided in due course. If you would like to use any part of this work, please get in touch.
