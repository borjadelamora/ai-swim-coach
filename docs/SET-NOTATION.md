# Set notation

The coach writes every session in a compact, unambiguous shorthand. The same notation is used in the deck-card cheatsheet and in the raw log. This document is the complete specification.

## A block

A *block* is one group of repeats. The canonical form is:

```
n x distance Stroke <goal @ send-off
```

| Token | Meaning |
| --- | --- |
| `n x distance` | `n` repeats of `distance` metres (e.g. `4x50` is four 50s). |
| `Stroke` | `Kick` or `Pull` where it matters. Plain freestyle is left unlabelled. |
| `<goal` | The **goal time** — come in under it. The `<` reads literally as "under". |
| `@ send-off` | The **send-off** — the pace-clock interval. Leave on it, whatever your time. |

Example:

```
4x25 Kick <:40 @ :45
```

> Four 25s of kick. Aim to come in under 40 seconds. Leave every 45 seconds.

The gap between your time and the send-off is your rest. If you swim the 25 in :38 on a :45 send-off, you get 7 seconds before the next one leaves.

## Breath-controlled rest (`nB`)

Instead of a clock, rest can be counted in breaths. `nB` means *n breaths of rest between reps within the block*. Breath length is the swimmer's discretion — it scales rest to how the swimmer actually feels rather than to a fixed clock.

```
4x25 Free 4B       Four 25s, four breaths' rest between each.
2x100 Free 12B     Two 100s, twelve breaths' rest between them.
```

This format is the system's preferred vehicle for building aerobic volume: real pace with self-regulated rest, rather than one long continuous swim.

## Broken efforts

A single distance may be broken into shorter pieces with brief rest, to hold a faster pace than the whole would allow:

```
1x50 + 2x25        A "broken 100" — swim it as a 50 and two 25s.
```

## Breaks between blocks

A *set* is a sequence of blocks. The break **between** blocks is separate from the within-block send-off `@`:

```
— break —          Untimed, the swimmer's discretion. The usual default.
— break 2:00 —     A fixed, timed break.
— break 2-3 min —  A timed range, for firmer sets.
```

In the cheatsheet, breaks are written as horizontal rules instead:

```
---            Untimed break.
--- 2:00 ---   Timed break.
--- 2-3min --- Timed range.
```

## Effort tags

In the readable set, effort is always plain language with a feel-cue (easy / steady / strong / hard, plus how it should feel). In the terse cheatsheet, short tags are allowed: `ez`, `build`, `80%`, `strong`, and per-rep notes such as `1,3 ez 2,4 80%`.

## The two permanent format rules

These encode the geometry of the pool and the pace clock. They never change, and the critic enforces them.

### 1. Send-offs are multiples of :15

`:15 / :30 / :45 / 1:00 / 1:15 / 1:30 …` only. The swimmer reads the pace clock in fifteen-second increments, so an odd interval like `@:42` is unusable and is flagged as a blocker.

### 2. Every block finishes on the start wall

In a 25 m pool, the swimmer pushes off from one wall and must return to it before a break. This means **every block's total distance is a multiple of 50 m** — 50, 100, 150, 200, and so on. (150 m is fine: six lengths, finishing where you started.)

The failure cases are pieces that strand the swimmer at the far wall:

- a lone 75 (three lengths — ends across the pool);
- a rep-count that sums to an odd number of lengths, such as `5x25` (= 125 m, five lengths).

The coach computes block and session totals explicitly and the critic checks them, so these never reach the deck card.

## A worked example

Readable form:

```
WARMUP — 200 m
200 Free easy — loosen up
— break —

KICK — 100 m
4x25 Kick — strong, under 40 each, leaving every 45
— break —

MAIN SET — 400 m
2x100 then 4x50, twelve breaths' rest — steady, hold your pace
— break —

COOL DOWN — 100 m
100 Free easy
```

Cheatsheet form:

```
200 ez
---
4x25 Kick <:40 @ :45
---
2x100 Free 12B
4x50 Free 12B
---
100 ez
```

Both describe the same 800 m session; the readable form carries the cues, the cheatsheet carries the bare work.
