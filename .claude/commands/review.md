---
description: Weekly / phase review.
---

Produce a training review for the period the swimmer names (default: last 7
sessions). Delegate to the historian for the data (from sessions.jsonl;
archive.jsonl is calibration-only) and the physiologist for the load read.
Output TWO layers, in this fixed structure (drop none).

PRIMARY — the last 7 sessions (the block):
  Period: what's covered.
  Load & recovery: the load trajectory (vs the recent baseline), the recovery pattern, and the
    intensity mix (label sessions easy<6 / mod 6-<8 / hard 8-<9 / peak>=9, for WORDING only).
    Judge by the Physiologist's decision order — flag only unrecovered hard work, a sharp ramp,
    monotony, or an outcome trigger (forced deload, rpe over plan, splits regressing, benchmark
    stalling). Do NOT judge against a fixed easy:hard ratio; the right distribution scales with
    the athlete's volume, and a high hard-fraction they recover from is fine.
  Active skill: progression this period.
  Records: any movement.
  Phase: hold, or transition/microcycle change — with why; have the logger update
    phase.json ONLY if a transition is warranted.
  Next block: the focus going forward.

SECONDARY — a fixed "Wider picture" section over ALL sessions (the historian's history_arc):
the long-run skill trend, key benchmark movement, the whole-history load trajectory (is overall
load/intensity drifting?), and how the recent block fits it.

/review is READ-ONLY over the per-session files — it does NOT rewrite current_state.md,
swimmer_profile.md, prs.json, or the log (those are the logger's job under /log). The only
file it may change is phase.json, and only when a transition or microcycle change is warranted.

Speak with calibrated confidence — keep the depth and conviction, but separate what the
clock PROVED (times, PRs, what was swum) from what the data SUGGESTS (a trend the evidence
supports) from what can't be pinned down yet, and say how much backs each. A thing seen
once is not yet a pattern; a thing seen every session is — don't harden a one- or
two-session read into a firm conclusion. Pain/fatigue default: if the swimmer didn't
report any, there is none — do not list it as a data gap.

Once the review is done, this chat is safe to clear — the next command starts fresh from memory/.
