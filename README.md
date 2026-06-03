# Sales Call Coach

Two Claude Code skills that turn every sales call transcript into grounded
coaching, then aggregate it into continuous training. Score one call on 8
execution dimensions, then a weekly rollup tells you what's improving, what
keeps failing, whether last week's focus actually moved, and which behaviors
correlate with closed deals.

## What's in the repo

- **`sales-call-coach/`** — scores a single call from its transcript and writes
  a structured coaching file. 8 dimensions, every verdict cites an exact moment.
- **`sales-coach-rollup/`** — reads every coaching file (frontmatter only,
  never re-reads transcripts) and produces a weekly rollup with trend,
  recurring weakness, outcome correlation, and one focus for next week.

## Install

Each sub-folder is a Claude Code skill. Drop them in your skills folder:

```bash
git clone https://github.com/emmaguetta/sales-call-coach.git
cp -r sales-call-coach/sales-call-coach ~/.claude/skills/
cp -r sales-call-coach/sales-coach-rollup ~/.claude/skills/
```

Restart Claude Code. The skills auto-activate on triggers like "coach this
call", "review this sales call", "run the coaching rollup", "how is <rep>
trending".

## Quick start

1. Get a call transcript. Granola free tier, Fathom, Fireflies, tl;dv, Otter,
   Zoom — any source. Speaker labels help (the talk-ratio estimate needs them).
2. In Claude Code, paste the transcript and say "coach this call". The first
   skill writes `calls/YYYY-MM-DD-rep-account.md` with 8 scores tied to exact
   moments + one focus for next call.
3. At end of week, say "run the coaching rollup for last 7 days". The second
   skill reads the frontmatter of every call file and writes
   `rollups/YYYY-Www-team.md` with the scorecard, the loop check, and the
   focus for next period.

Two folders. No SaaS, no Zapier, $0 a month with Granola free tier.

## Why two skills

The per-call skill writes a YAML frontmatter block with the 8 verdicts, the
score, the talk ratio, the focus, and an outcome field. The rollup reads only
that block — never re-opens a transcript. Aggregating 50 calls by re-reading
50 transcripts would be slow and drift. Aggregating 50 small structured
headers is instant and exact.

## Setup guide

A full setup guide (what to adjust before the first call, the 8 dimensions
explained, why two skills, how to log outcomes) is published as a Notion page
and sent in the LinkedIn DM after you comment on the launch post.

## License

MIT.
