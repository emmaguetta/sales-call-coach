---
name: sales-call-coach
description: >-
  Coach a sales call from its transcript. Score the rep on 8 execution
  dimensions, cite exact moments, and write one structured coaching file per
  call so progress can be tracked over time. Use when the user pastes a
  transcript and asks to "coach this call", "review this sales call", or
  invokes /sales-call-coach.
---

# Sales Call Coach

Turn one sales call transcript into structured, grounded coaching, and persist
it so the `sales-coach-rollup` skill can track progress across calls.

One pass:

1. Read the transcript and the call's metadata.
2. Score the rep on 8 dimensions, each grounded in a specific moment.
3. Write one coaching file with a machine-readable header so trends can be
   aggregated later without re-reading transcripts.

## File this skill writes

- Coaching folder: `calls/` (auto-created)
- One file per call: `calls/YYYY-MM-DD-[rep-kebab]-[account-kebab].md`
  (lowercase, spaces → `-`, accents stripped, special chars removed)

This skill never overwrites an existing call file. If the target path already
exists, append `-2`, `-3`, etc. The skill only writes its own per-call file. It
does not modify any shared file, so no validation gate is needed before writing.

## Required inputs

From the user or inferable from the transcript with high confidence:

- `date` — call date (YYYY-MM-DD). Default to today if absent.
- `rep` — the sales rep being coached. Ask if not inferable.
- `account` — the prospect company. Ask if not inferable.
- `call_type` — discovery | demo | follow-up | negotiation | other.
- Transcript — pasted text. If absent, stop and ask for it. Do not coach from
  notes alone unless the user explicitly says the notes are the only record.

Do not invent a rep or account name. A wrong name corrupts the rollup.

## Phase 1 — Write the coaching file

Path: `calls/YYYY-MM-DD-[rep-kebab]-[account-kebab].md`.

The file has a machine-readable YAML frontmatter (parsed by the rollup skill)
followed by the human coaching report.

### Frontmatter — exact schema

```yaml
---
coaching_record: v1
date: YYYY-MM-DD
rep: <rep-kebab>
account: <account-kebab>
call_type: discovery | demo | follow-up | negotiation | other
seats_mentioned: <integer or null>      # team size / seat count the prospect named
scores:
  opening_upfront_contract: pass | fail
  pain_discovery: pass | fail
  qualification_process: pass | fail
  listening_talk_ratio: pass | fail
  value_articulation: pass | fail
  objection_handling: pass | fail | na  # na only if no hesitation/pushback occurred
  buying_signals: pass | fail
  next_steps_close: pass | fail
score_total: <count of pass>
score_applicable: <8 minus count of na>
talk_ratio_estimate: "<rep ~X% / prospect ~Y%>" | "unknown - transcript not speaker-labeled"
focus_next_call: "<one sentence, the single highest-leverage change>"
outcome: pending        # pending | won | lost | no-show | next-step-booked | disqualified
outcome_notes: ""
---
```

Rules for the frontmatter:

- `pass` / `fail` only. `na` is allowed **only** for `objection_handling`, and
  only when no hesitation, pushback, or concern was raised by the prospect. Do
  not fabricate an objection to force a verdict.
- `score_total` = number of `pass`. `score_applicable` = 8 minus number of `na`.
- `talk_ratio_estimate`: estimate only if the transcript has speaker labels or
  the speakers are unambiguous. Otherwise write the literal string
  `unknown - transcript not speaker-labeled`. Never guess a ratio from an
  unlabeled wall of text.
- `seats_mentioned`: the number of seats / team size the prospect said they
  would need, or `null` if not mentioned.
- `outcome` is always `pending` at write time. The user edits it later when the
  deal moves. Never set anything other than `pending`.

### Body — exact structure

Output exactly these 8 sections, always in this order, each headed with one
emoji (`✅` if the rep did it well, `❌` if not; for objection handling, `➖`
if not applicable). 2 sentences max per section. Every verdict must quote or
tightly paraphrase a specific moment from the transcript — no generic advice.

```
# Call coaching — <Account>, <date> (<rep>)

Seats / deal-size signal: <number the prospect mentioned, or "not mentioned">

## 1. OPENING & UPFRONT CONTRACT ✅ / ❌
Did the rep establish credibility, set a clear agenda, AND get the prospect to
agree on the call's structure, timing, and desired outcome in the first 2 min?

## 2. PAIN DISCOVERY (DEPTH) ✅ / ❌
Did the rep uncover the real pain, its business impact, and the quantified cost
of inaction? Or did they stop at a surface "we have a problem"?

## 3. QUALIFICATION & PROCESS ✅ / ❌
Did the rep surface the decision process, economic buyer, budget / seats,
timeline, and alternatives considered? Or did they pitch before qualifying?

## 4. ACTIVE LISTENING & TALK RATIO ✅ / ❌
Did the rep acknowledge and build on what the prospect said? Estimate the
rep-vs-prospect talk ratio and flag if the rep dominated the call.

## 5. VALUE ARTICULATION ✅ / ❌
Did the rep connect benefits to the specific pains raised, in the prospect's
own words? Or was it a generic feature dump?

## 6. OBJECTION HANDLING ✅ / ❌ / ➖
How did the rep respond to hesitation or pushback? Did they probe to understand
the objection before answering? Mark ➖ if no objection was raised.

## 7. BUYING SIGNALS ✅ / ❌
Which buying signals (budget mentioned, urgency, a competitor named, "how soon
could we start") did the rep catch and act on, and which were missed?

## 8. NEXT STEPS & CLOSE ✅ / ❌
Did the rep secure a concrete next step with an owned date on the calendar? Was
the close confident or vague?

---

SUMMARY: <score_total>/<score_applicable>. The single highest-leverage thing to
change on the next call: <one specific, actionable sentence tied to this call>.
```

## Tone

Be direct and specific. Cite exact moments (quote or tight paraphrase). Balance
praise and improvement. Write as if giving live feedback right after the call —
no fluff, no preamble, no closing pep talk.

## Rules

1. **Ground every verdict.** No section without a cited moment from the
   transcript. If the transcript has no evidence for a dimension, mark `fail`
   and say so explicitly ("no agenda was set" is a finding, not a guess).
2. **Stay faithful.** Do not invent objections, signals, or numbers. `na`/`➖`
   for objection handling is correct when nothing was raised — use it.
3. **One focus, not five.** The SUMMARY names exactly one change. The rollup
   depends on a single, comparable focus per call.
4. **Frontmatter is data.** It must parse: valid YAML, exact keys, allowed
   values only. The rollup skill reads it verbatim.
5. **Be concise.** 2 sentences max per section. No intro, no conclusion.
