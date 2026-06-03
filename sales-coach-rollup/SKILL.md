---
name: sales-coach-rollup
description: >-
  Aggregate the per-call coaching files written by sales-call-coach into a
  scorecard over a period, for one rep or a whole team. Shows trend vs the
  previous period, the top recurring weaknesses, whether last period's focus
  improved, and how coaching scores correlate with deal outcomes. Use when the
  user asks to "run the coaching rollup", "how is <rep> trending", "team
  coaching review", or invokes /sales-coach-rollup.
---

# Sales Coach Rollup

Turn the `calls/` folder into continuous training: not a static report, but a
loop that checks whether last period's flagged weakness actually improved.

## Files this skill reads and writes

- Reads: every `calls/*.md` written by `sales-call-coach` (frontmatter only —
  never re-reads transcripts).
- Reads: the most recent matching file in `rollups/` to close the loop.
- Writes: one rollup file `rollups/YYYY-Www-[rep-kebab | team].md`.

The rollup never modifies a `calls/` file. It only reads their frontmatter and
writes its own summary file.

## Arguments

- **Scope** — a specific `rep` (kebab name), or `team` / `all` for everyone.
  Default: `team` if not specified.
- **Period** — a date range, a number of days (e.g. "last 14 days"), or a call
  count (e.g. "last 10 calls"). Default: last 7 days.

If the scope names a rep with no matching files, say so and stop. Do not
invent data.

## Phase 1 — Collect

Read the frontmatter of every file in `calls/` whose `date` falls in the
period and whose `rep` matches the scope. Parse only the YAML frontmatter; do
not load the coaching body or any transcript.

Build two sets:

- **Current period** — calls in the requested window.
- **Prior period** — the immediately preceding window of equal length, for the
  trend comparison.

If there are fewer than 3 calls in the current period, still produce the
rollup but flag it: "low sample (N calls) — trends are indicative only".

## Phase 2 — Compute

### Per-dimension scorecard

For each of the 8 dimensions, compute the pass rate over the current period
(`pass / applicable`, excluding `na`). Compute the same for the prior period.
Show each as a percentage with a trend arrow:

- `▲` improved by ≥10 points
- `▼` dropped by ≥10 points
- `→` within ±10 points (stable)

### Recurring weaknesses

Rank dimensions by current-period fail rate. The top 1-2 are the recurring
weaknesses to call out.

### The loop (continuous training)

Open the most recent prior rollup file in `rollups/` for this scope. Read its
`focus_set` field (the dimension it told the rep/team to work on). Then check
the current period:

- Did that dimension's pass rate improve vs the period the focus was set on?
  State it plainly: "Focus last period was PAIN DISCOVERY (40% → target). This
  period: 71%. Improved."
- Did a different dimension regress while attention was on the focus? Flag it.

If no prior rollup exists, skip the loop and state: "Baseline period — no prior
focus to check against."

### Outcome correlation

If at least 3 calls in the period have an `outcome` other than `pending`,
compute, per dimension: pass rate among `won` / `next-step-booked` calls vs
`lost` / `disqualified` / `no-show` calls. Report only dimensions with a clear
gap (≥20 points), phrased as a correlation, not causation: "Calls that passed
PAIN DISCOVERY booked a next step 80% of the time vs 30% when it failed."

If fewer than 3 non-pending outcomes exist, write: "Not enough logged outcomes
yet to correlate coaching with results — log `outcome` in the call files as
deals move."

## Phase 3 — Write the rollup

Path: `rollups/YYYY-Www-[rep-kebab | team].md` (`Www` = ISO week, e.g.
`2026-W21`). If the file exists, append `-2`.

### Frontmatter

```yaml
---
rollup_record: v1
scope: <rep-kebab | team>
period_start: YYYY-MM-DD
period_end: YYYY-MM-DD
calls_in_period: <integer>
focus_set: <dimension key for next period, e.g. pain_discovery>
---
```

`focus_set` is the single dimension this rollup tells the rep/team to work on
next period. The next rollup reads it to close the loop. Pick the dimension
with the worst current pass rate that is also actionable, not one already
improving fast.

### Body — exact structure

```
# Coaching rollup — <scope> — <period_start> to <period_end>

<N> calls. <low-sample flag if applicable>

## Scorecard (this period vs previous)

| Dimension                     | This period | Prev | Trend |
|-------------------------------|-------------|------|-------|
| Opening & upfront contract    | <X%>        | <Y%> | ▲/▼/→ |
| Pain discovery (depth)        | ...         |      |       |
| Qualification & process       | ...         |      |       |
| Active listening & talk ratio | ...         |      |       |
| Value articulation            | ...         |      |       |
| Objection handling            | ...         |      |       |
| Buying signals                | ...         |      |       |
| Next steps & close            | ...         |      |       |

## The loop
<Did last period's focus improve? One or two sentences. Or "Baseline period.">

## Recurring weakness
<Top 1-2 dimensions by fail rate, one sentence each, with the pattern.>

## Outcome correlation
<Per-dimension gap vs deal outcome, or the not-enough-data line.>

## Focus for next period
<One dimension. One concrete drill. Example: "PAIN DISCOVERY — on every call,
ask the cost-of-inaction question before showing the product.">
```

For `team` scope, also add a one-line per-rep ranking on the focus dimension so
the manager sees who needs the most help on it. Do not rank reps overall — rank
only on the focus dimension to keep it actionable, not a leaderboard.

## Rules

1. **Frontmatter only.** Never re-read transcripts or coaching bodies. The
   whole point of the structured header is to make aggregation cheap.
2. **Close the loop.** The rollup is training, not reporting: always check
   whether the last focus moved before naming a new one.
3. **One focus out.** Exactly one `focus_set`. Spreading attention across
   dimensions is why coaching doesn't stick.
4. **Correlation, not causation.** Outcome gaps are reported as associations.
   Do not claim a dimension "causes" wins.
5. **Honest about sample size.** Small N is flagged, not hidden. Do not present
   a 2-call period as a trend.
6. **No invented data.** Missing outcomes, missing prior rollup, thin period —
   state the gap, do not fill it.
