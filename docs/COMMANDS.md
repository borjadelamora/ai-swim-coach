# Commands

The system exposes three slash-commands, defined in `.claude/commands/`. Each is a small pipeline that delegates to the specialist agents described in [`ARCHITECTURE.md`](ARCHITECTURE.md). This document describes what each one does, what it expects as input, and what it produces.

---

## `/plan` — plan today's session

**When:** before you swim.

**Input:** free-form. Typically a time budget, optionally with how you feel. Feelings are treated as data, not orders.

```
/plan 45 minutes
/plan 1 hour, legs feel heavy after yesterday
/plan 30 min, short on time, want something sharp
```

**What it does:**

1. Delegates to the **historian** for recent load, the set shapes due for variation, the active skill's recency and status, and any data gaps.
2. Delegates to the **physiologist** — sequentially, with the historian's findings in hand — to set today's target: energy system, zone, and work-to-rest.
3. Hands the physiologist's prescription, the profile, and the recent set shapes to the **coach**, with an instruction to make today's session meaningfully different in structure from recent ones while honouring the purpose, the swimmer's real paces, and every constraint.
4. Passes the coach's set verbatim to the **critic**, which returns *approve* or *revise*. On a blocker, the coach is handed its own set plus the single fix and told to change only that. The loop is bounded.

**Output:** two artefacts.

- A **readable set** — plain language, with an effort feel-cue on each block, per-block and total distance, and a short "why today" rationale. No coaching jargon. If the plan overrides what you asked for, it says so and explains why.
- A **cheatsheet** — the bare set in shorthand notation, to copy onto a waterproof deck card. See [`SET-NOTATION.md`](SET-NOTATION.md).

Overrides are not watered down. If the data says today should be hard when you asked for easy, you get the hard session with its reasoning, not an apology.

---

## `/log` — record the session just completed

**When:** after you swim.

**Input:** free-form. Report what you actually did, in the order you did it, with whatever times, efforts, and notes you have. Mention anything you skipped, and any pain — if you do not mention pain, the system records none.

```
/log felt strong. kick 4x25 came in 38/39/39/40. pull held pace, all turns clean.
     skipped the last 100 easy, ran out of time.
```

**What it does:** delegates to the **logger**, which:

1. Appends one structured JSON object to `sessions_scm.jsonl` — facts only, blocks in swum order — and verifies it landed (valid JSON, `id` one greater than the previous, a real date set). This raw line is the source of truth and is written first.
2. Appends one row to `TRAINING_LOG.md` (the glance view).
3. Updates `prs.json` if a record fell.
4. Refreshes `current_state.md` (benchmarks and next bounds).
5. Refreshes `swimmer_profile.md` (durable changes only, preserving existing nuance).

**Output:** a short, honest summary — what changed, any record, and the one-sentence priority for the next session. If your self-rating and your actual times disagree, it says so, without over-reading a single session.

---

## `/review` — weekly or phase review

**When:** at the end of a week, or at a suspected phase boundary.

**Input:** optional period. Defaults to the last seven sessions.

```
/review
/review last 2 weeks
```

**What it does:** delegates to the **historian** for the data and the **physiologist** for the load read, then covers total volume, the easy-to-hard balance across the week (the ~80/20 target, or drift into the middle), the active skill's progression, any record movement, whether a recovery microcycle or phase change is due, and the focus for the next block.

**Output:** a calibrated review that separates what the clock proved from what the data merely suggests, and says how much evidence backs each read.

**Scope:** `/review` is read-only over the per-session record. It does not rewrite `current_state.md`, `swimmer_profile.md`, `prs.json`, or the log — those belong to `/log`. The only file it may change is `phase.json`, and only when a transition is genuinely warranted.

---

## Design notes that apply across all three

- **Stated feelings are data.** Every command weighs what you report against the logged record and overrides when the evidence warrants.
- **Conviction in prescription, honesty in analysis.** The set you read is delivered with full conviction and no hedging. The uncertainty lives in the analysis commands (`/log`, `/review`), which are explicit about how sure they are.
- **Self-contained.** Each command reads its state from `memory/`, so it behaves identically in a fresh terminal or mid-conversation.
