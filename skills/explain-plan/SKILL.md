---
name: explain-plan
description: Walk the user through an implementation plan section by section — one small piece at a time, pausing for agreement after each — instead of dumping a whole plan or diving straight into code. Calibrates technical depth to the user's knowledge and keeps a shared glossary in sync. Use when the user says "explain the plan", "walk me through it first", "don't dump the whole plan", "go section by section", "explain before you build", or before starting any non-trivial implementation.
---

# Explain Plan

Reveal an implementation plan **incrementally**, in digestible pieces, getting agreement on each before moving on — and only write code once every piece is agreed. This fixes two failure modes: (1) a full upfront plan is too long to reason about, and (2) silently coding means the user never sees the intent until it's already built.

## Step 0 — Calibrate depth (ask ONCE, first thing)

Before any section, ask exactly one question about the user's baseline with this part of the stack:

> "How familiar are you with this part of the system? Should I show actual code and technical detail, or explain in plain terms (the flow and what to expect)?"

Lock the answer in for the whole walkthrough:

- **Knows the tech** → you ARE authorized to use code, signatures, and terminology throughout.
- **Doesn't** → explain ONLY non-technically: the *flow* of what's being built and what the *end result* will look like. No code, no jargon.

Do **not** re-ask after each section. Only revisit if the plan clearly crosses into a *different* domain the user may not know.

## Step 1 — Load the shared vocabulary (before explaining anything)

- Look for a domain-glossary / context document in the project (commonly `CONTEXT.md`, `GLOSSARY.md`, or similar).
- If it exists: read it. Use those exact terms while explaining. Do **not** ask the user to redefine terms already documented.
- If none exists: offer to create one. Create it only on approval.

## Step 2 — Decompose, then reveal one section at a time

- Break the work into the **smallest meaningful sections**. Pick the natural breakdown for *what's being built* — sections are domain-appropriate, not fixed.
  - *Illustration (an API):* §1 endpoint name → §2 decorators/annotations → §3 input validation → §4 core logic → §5 response shape. (Just an example — a UI screen, schema change, or script breaks down differently.)
- For **each** section, in order:
  1. State your proposal for this one piece.
  2. Give a **short** rationale (1–2 sentences).
  3. **STOP.** Wait for the user to agree or discuss before the next section.
- Keep each section short and readable. Never reveal the whole plan at once. Never combine sections to "save time."

## Step 3 — Watch for new domain terms (while walking through)

When the user uses a word that is NOT generic technical vocabulary AND is NOT already in the glossary:

1. **Pause** and surface it:
   > "We don't have **{term}** defined. Is it the same as **{existing-term}**, or is it new? If new, can we add it?"

   Always check for an existing synonym first — avoid duplicate/competing terms.
2. On confirmation it's new, propose an entry in the document's existing style, e.g.:
   `- **TermName** — one-sentence definition. _Example: usage context._`
3. **Show the diff/preview.** Append to the glossary only on the user's explicit approval.

Never invent or add terms unilaterally. Never auto-commit the doc change — always preview and wait.

## Step 4 — Implement

Only after **all** sections are agreed, proceed to implementation — following exactly what was agreed, in the agreed terminology.

## Guardrails

- One question in Step 0; one section per round-trip in Step 2. No walls of text.
- Stay in the depth mode chosen in Step 0 until the domain genuinely changes.
- Glossary edits are preview-then-approve, never silent.
