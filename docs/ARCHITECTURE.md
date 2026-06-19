# Architecture

AI Swim Coach is a thin orchestration layer over [Claude Code](https://claude.com/claude-code). It has no runtime of its own: the "application" is a set of agent definitions and command definitions, and the "database" is a directory of plain-text files. This document explains how those pieces fit together and why the design is shaped the way it is.

## The two moving parts

| Concept | Lives in | What it is |
| --- | --- | --- |
| **Commands** | `.claude/commands/*.md` | Entry points the athlete invokes (`/plan`, `/log`, `/review`). Each defines a pipeline. |
| **Agents** | `.claude/agents/*.md` | Specialists with a single responsibility, scoped tools, and a model tier. Commands delegate to them. |
| **Memory** | `memory/*` | All state. Read by every agent; written only by the logger. |
| **Specification** | `CLAUDE.md` | The rules every agent obeys — notation, memory rules, epistemic discipline. |

A command is a short script of delegation. It spawns agents in sequence, passes each one's output to the next, and presents the result. The intelligence lives in the agent prompts; the command is the wiring.

## Why five agents instead of one

A single model asked to "be a swim coach" will cheerfully invent a benchmark, prescribe against it, and praise the result — all in one breath, with nothing to catch the error. Splitting the work into narrow specialists makes that failure structurally harder:

- The **historian** may only *report*. It has read-only tools and is forbidden from prescribing, so the facts that enter the pipeline are not contaminated by opinion.
- The **physiologist** may only *constrain*. It sets the target zone and work-to-rest structure but does not write the set.
- The **coach** may only *write the set*, and only within the physiologist's constraints.
- The **critic** may only *approve or reject*. It is the independent check that sees the set before the athlete does.
- The **logger** is the only agent that may *write to memory*. Everyone else is read-only.

This is separation of powers applied to coaching. No single agent can both fabricate a number and act on it, because the agent that establishes facts is not the agent that prescribes, and the agent that prescribes is not the agent that records.

### Tool scoping

Each agent is granted exactly the tools its job requires:

| Agent | Tools | Rationale |
| --- | --- | --- |
| Historian | `Read`, `Grep`, `Glob` | Needs to search the log; must not modify it. |
| Physiologist | `Read` | Reasons from state; produces text only. |
| Coach | `Read` | Reads state and the brief; produces text only. |
| Critic | `Read` | Reviews against the record; produces a verdict only. |
| Logger | `Read`, `Write`, `Edit` | The sole writer to memory. |

### Model tiers

Model choice is matched to the cognitive load of each role, so cost and latency are spent where they matter:

| Agent | Tier | Why |
| --- | --- | --- |
| Historian | Fast | Mechanical fact-retrieval and summarisation. |
| Physiologist | Mid | Reasoning over load and zones. |
| Coach | Top | The hardest creative task: a correct, varied, constraint-satisfying set. |
| Critic | Mid | Structured checking against explicit failure modes. |
| Logger | Mid | Careful, schema-bound writing. |

## The planning pipeline (`/plan`)

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

1. **Historian first.** It summarises recent load, the set shapes that have repeated and are due for variation, the recency of the active skill, and any genuine data gaps.
2. **Physiologist second, sequentially.** It reasons *from the historian's load summary* to today's target — energy system, zone, work-to-rest. Because it depends on the historian's facts, the two cannot run in parallel.
3. **Coach third.** Given the physiologist's constraints, the profile, and the recent set shapes, it writes a session deliberately different in structure from recent ones.
4. **Critic fourth.** It checks the set against the brief and the log for specific failure modes (soft intervals, mushy-middle intensity, skill placed after fatigue, broken format rules, unrequested changes on a revision). It returns *approve* or *revise*.

The critic loop is **bounded**. A set is revised at most once or twice; if a blocker still stands, the command applies the mechanical fix directly rather than looping the coach indefinitely. When the coach revises, it is handed its own prior set plus the single fix and instructed to change only that — preventing it from drifting and rewriting blocks that were already correct.

## The logging pipeline (`/log`)

The logger writes five things, in a strict order, and the **raw session line lands first**:

```
1. raw session line   →  sessions_scm.jsonl   (append-only; SOURCE OF TRUTH)
2. glance-view row    →  TRAINING_LOG.md      (append-only)
3. records            →  prs.json             (only if a record fell)
4. current capacity   →  current_state.md     (benchmarks + next bounds)
5. durable profile    →  swimmer_profile.md   (only what genuinely changed)
```

The ordering is the whole point: items 2–5 are **derived views**, each rebuildable from item 1. By writing and verifying the raw line before touching anything derived, the system guarantees that the irreplaceable record exists before any interpretation is layered on top. If a derived file ever disagrees with the raw log, the log wins.

The logger also enforces the **facts-versus-interpretation** boundary: the raw line contains only what happened (times, efforts, turns, anything skipped); all assessment lives in the profile.

## The review pipeline (`/review`)

`/review` is **read-only** over the per-session record. It delegates to the historian for the data and the physiologist for the load read, then assesses volume, the weekly easy-to-hard balance, skill progression, record movement, and whether a recovery microcycle or phase change is due. The single file it may write is `phase.json`, and only when a transition is genuinely warranted.

## Statelessness and reproducibility

Every agent is spawned fresh and reads its context from `memory/`, never from the surrounding chat. A consequence worth stating plainly: a command run in a brand-new terminal produces the same quality of output as one run deep in a long conversation. The length or content of the parent chat has no bearing on the result. The files are the program's memory; the chat is not.

## Adapting the system to a different athlete

Because every personal fact lives in `memory/` and the agent prompts hold only reusable method, adapting the system to a new athlete means editing the profile and current-state files — not the prompts. When a memory fact and a prompt ever conflict, memory wins.
