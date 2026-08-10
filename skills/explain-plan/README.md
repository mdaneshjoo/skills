# `/explain-plan` — Walk a Plan, Section by Section

Stops the agent from dumping a 40-section plan at you, or skipping the plan entirely and
starting to write code. One piece at a time, pausing for your agreement before moving on.

```bash
npx skills add mdaneshjoo/skills --skill explain-plan
```

Installs only this skill — nothing else from the repo comes with it.

---

## What it does

| | |
| --- | --- |
| **One section at a time** | You agree with each piece before the next arrives. No wall of text to skim and pretend you read. |
| **Depth matched to you** | Calibrates to what you already know, so it neither over-explains basics nor hand-waves the part you actually need. |
| **Keeps a shared glossary** | Terms introduced along the way stay consistent instead of drifting between sections. |
| **Disagreement is cheap** | Catching a wrong assumption in section 2 costs a sentence. Catching it after the code is written costs the afternoon. |

---

## When it triggers

Say any of these and it takes over:

- "explain the plan"
- "walk me through it first"
- "don't dump the whole plan"
- "go section by section"
- "explain before you build"

It's also worth invoking yourself before any non-trivial implementation — the point is to
find the disagreement *before* the code exists, not after.

---

## Why it exists

The default failure mode of a planning agent is one of two extremes: a giant plan you
skim and approve without really reading, or no plan at all and straight to code. Both
push the disagreement to the most expensive possible moment — after everything is built.

Sectioned delivery forces the small, cheap "wait, no" to happen early, which is the only
time it's actually cheap.

---

## Pairs well with

- [`/explain-investigation`](../explain-investigation) — same pacing, but for a debugging
  report rather than a forward-looking plan.
