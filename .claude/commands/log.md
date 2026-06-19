---
description: Log the session just completed.
---

The swimmer is reporting how the session went: $ARGUMENTS

Delegate to the logger. It writes the irreplaceable raw session object to
sessions_scm.jsonl FIRST — one structured line, blocks in swum order, facts only (times,
effort, turns, skipped; no interpretation) — and verifies it landed (valid JSON, id =
prev + 1, real date set) before the DERIVED updates: the TRAINING_LOG.md row, prs.json
(if a record fell), and the refreshed current_state.md (benchmarks/bounds) and
swimmer_profile.md (durable changes only — preserve existing nuance, don't drop facts).
It confirms each write. (The raw line is the source of truth; the rest are rebuildable
from it, so it must land first.) Then summarize in 2-3 lines: what changed, any record,
and the one-sentence next-session priority. Be honest — if the swimmer's self-rating and actual
outputs disagree, say so — but don't over-read one session: call a single-session
observation what it is, and let trends harden as they repeat. Pain/fatigue default: if
the swimmer didn't mention pain or fatigue, there is none — never log it as unknown or
flag it as a gap.
