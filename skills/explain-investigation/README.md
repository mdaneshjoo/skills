# `/explain-investigation` — Walk an Investigation, Finding by Finding

Takes a debugging or investigation report and walks it one finding at a time, keeping
**evidence**, **meaning**, **impact**, **confidence**, and **uncertainty** as separate
things instead of blending them into one confident-sounding paragraph.

```bash
npx skills add mdaneshjoo/skills --skill explain-investigation
```

Installs only this skill — nothing else from the repo comes with it.

---

## What it does

| | |
| --- | --- |
| **One finding at a time** | You absorb and question each before the next arrives. |
| **Evidence kept apart from interpretation** | "The log shows X" and "which means Y" are different claims with different reliability. Blending them is how a guess gets treated as a fact. |
| **States confidence explicitly** | Says which parts are solid and which are inference, instead of narrating everything in the same assured tone. |
| **Names what's still unknown** | Unresolved questions stay visible rather than getting smoothed over. |
| **Doesn't jump to the fix** | Understanding first. Proposing an implementation mid-explanation is how a wrong diagnosis gets built on. |

---

## Reads only what you give it

It works from an **explicit source you point at** or the **current session's context** —
nothing else. It will not go searching your vault, project folders, or filesystem unless
you ask it to.

If no source is available it says so and asks for one, rather than reconstructing an
investigation from memory. That constraint is deliberate: a plausible explanation
assembled from recall is worse than none, because it reads exactly like a real one.

---

## When it triggers

- "explain this investigation"
- "what does this report mean?"
- "walk me through the findings"
- After any debugging session that produced a written report

---

## Pairs well with

- [`/explain-plan`](../explain-plan) — same pacing, for a forward-looking plan rather
  than a post-mortem.

---

## License

MIT.
