---
description: Weekly / phase review.
---

Produce a training review for the period the swimmer names (default: last 7
sessions). Delegate to the historian for the data (from sessions.jsonl;
archive.jsonl is calibration-only) and the physiologist for the load read.
Output these sections, in this fixed order (drop none):
  Period: what's covered.
  Volume & easy:hard: total volume + the ~80/20 balance (or drifting into the middle?).
  Active skill: progression this period.
  Records: any movement.
  Phase: hold, or transition/microcycle change — with why; have the logger update
    phase.json ONLY if a transition is warranted.
  Next block: the focus going forward.

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
