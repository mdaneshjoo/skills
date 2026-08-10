---
name: iq
description: Daily interview-prep quiz tailored to the user's own stack and shipped projects. Builds today's set once per day — N new deduped questions + 1 carry-over (unanswered) + 1 review (spaced repetition) — and runs an interactive attempt-then-grade loop, storing everything in a notes folder the user chooses. All settings (storage, volume, stack, seniority) live in an iq-config.md created by `/iq init`; nothing is hardcoded, so the skill is shareable. Use when the user says "quiz me", "interview questions", "interview prep", "/iq", or "/iq init".
---

# /iq — Daily Interview Prep

Interactive daily interview practice, **tailored to the user's own stack and shipped projects**. The daily set is built once and reused all day, then drilled one question at a time with grading.

## Daily set
| # | Kind | Source |
|---|---|---|
| 1 … N | **new** | freshly generated + deduped. `N = config.newPerDay` (default 5). Up to `projectQuestionsPerDay` of these are grounded in the user's real projects. |
| N+1 | **carried** | 1 **random unanswered** question from history (clears backlog). Skipped if `carryOver: false`. |
| N+2 | **review** | 1 **random answered** question from history (spaced repetition). Skipped if `review: false`. |

The carry-over and review questions are chosen **randomly and frozen for the day** (they live in the day file, so they don't reshuffle across sessions). **If a pool is empty, skip that slot** — early on a day may have only `N` questions; it grows as history builds.

`index.md` is the pool tracker: `- [ ]` lines = the **unanswered** pool, `- [x]` lines = the **answered** pool.

## Configuration — `iq-config.md`

**Nothing about this skill is hardcoded.** Storage, question count, stack, seniority all come from a single human-editable markdown file: **`iq-config.md`**.

It's markdown, not JSON, on purpose — the user should be able to tick a stack checkbox in their editor without asking an agent.

### Where it lives (user picks at init)

| Scope | Config | Pointer registered in |
| --- | --- | --- |
| **global** | `~/.claude/iq-config.md` | `~/.claude/CLAUDE.md` (or the tool's equivalent) |
| **project** | `<project>/iq-config.md` | `<project>/CLAUDE.md` (or equivalent) |

Global = same stack everywhere. Project = different stacks per repo. Project config wins when both exist.

### Registering the pointer (tool-agnostic)

So the config is loaded when the user runs `/iq`, init appends a pointer to whichever **agent-instruction file** the tool reads. Detect what exists; ask only if ambiguous:

| Tool | File |
| --- | --- |
| Claude Code | `CLAUDE.md` |
| Codex | `AGENTS.md` |
| Gemini CLI | `GEMINI.md` |
| Cursor / generic | `AGENTS.md` |

The appended block is small and stable (it sits in the prompt prefix, so keep it short):

```markdown
## /iq — Interview Prep
Config: `<path>/iq-config.md`. Read it when the user runs `/iq` or asks for interview practice.
```

### Config schema

```markdown
---
root: /absolute/path/to/notes
questionsDir: Interview Questions
articlesRoot: RoadMap/Backend
linkStyle: wikilink          # wikilink | markdown
newPerDay: 5
carryOver: true
review: true
projectQuestionsPerDay: 1    # 0 disables
seniority: senior            # mid | senior | staff
alwaysOnTopics: [system design, architecture & design patterns]
---

# /iq Configuration

## Stack
- [x] Node.js / TypeScript
- [ ] Go

## Projects
- **<name>** — <one line> · _<tech>_

## Notes
<free text: focus areas, target role, things to avoid>
```

| Key | Meaning |
| --- | --- |
| `root` | Absolute path to notes/vault/repo. **The only machine-specific value.** |
| `questionsDir` | Quiz data subfolder. Default `Interview Questions`. |
| `articlesRoot` | Article subfolder. Default `<questionsDir>/articles`. |
| `linkStyle` | `wikilink` → `[[Note]]` (Obsidian/Logseq). `markdown` → relative links. |
| `newPerDay` | Count of **new** questions. Carry-over/review are counted separately. |
| `carryOver` / `review` | Toggle the Q(n+1) / Q(n+2) slots. |
| `projectQuestionsPerDay` | How many of `newPerDay` are grounded in the user's real projects. Requires a resume. `0` = off. |
| `seniority` | Drives difficulty. |
| `alwaysOnTopics` | Topics generated regardless of stack — real interviews ask these of everyone. |

**Stack** = only ticked boxes are eligible for tech-specific questions. **Projects** = the source for project-grounded questions.

Derived paths:
- Day files: `<root>/<questionsDir>/days/<YYYY-MM-DD>.md`
- Index (dedup source + pool tracker + TOC): `<root>/<questionsDir>/index.md`
- Resume: `<root>/<questionsDir>/resume.md`

`iq-config.md` is **machine-local**. Share the skill, not the config — the recipient gets `/iq init` on their first run.

## Topic selection (driven by config — no fixed buckets)

Questions are generated from **two sources**, both from `iq-config.md`:

1. **Ticked stack items** — only these. If Go isn't ticked, no Go questions. This is the whole point: stop asking about things the user doesn't work with.
2. **`alwaysOnTopics`** — system design, architecture & design patterns, etc. Real interviews ask these of everyone regardless of resume, so they stay on unless explicitly removed.

Plus, when `projectQuestionsPerDay > 0`, that many of the `newPerDay` questions are **project-grounded** — drawn from the `## Projects` section: *"On <project> you used <tech> for <thing> — what problems does that create, and how would you handle <scenario>?"* These are the highest-signal questions, because they're what an interviewer would actually ask **this** candidate.

Rules:
- **English only.** Difficulty from `seniority`.
- **Never all from one topic** — spread across the ticked stack + always-on topics. Rotate so the same area doesn't dominate consecutive days.
- Weight toward what the user uses most (a stack item appearing across many projects is more interview-likely than one listed once).
- Project questions must be **answerable from the project description** — don't invent details the resume doesn't support.

---

## Step 0 — Load config (or run init)

Look for `iq-config.md` — project first (`./iq-config.md`, `./.claude/iq-config.md`), then global (`~/.claude/iq-config.md`).

**Found** → load it, say nothing, go to Step 1.

**Not found** → **do NOT start a quiz.** Say:

> "`/iq` isn't set up yet. Run **`/iq init`** and I'll walk you through it (about 4 questions) — or say *go* and I'll start now."

Then run the init wizard below on confirmation.

---

## `/iq init` — the setup wizard

Runs on first use, and any time the user asks (`/iq init`, "reconfigure /iq", "change my stack", "change where /iq stores things").

### How to run it — ALWAYS one step at a time

**Ask ONE question, stop, wait for the answer. Then the next.** Never print all the steps at once, never dump a config summary and ask "what do you want to change". The wizard is a conversation, not a form.

Each step shows:
- the **current value** (on re-init) or the **recommended default** (on first run), and
- how to accept it in one word (`keep` / `yes` / enter).

```
Step 1/5 — Scope
Currently: global (~/.claude/iq-config.md)
Keep global, or switch to project-local?
→ [wait for answer]

Step 2/5 — Storage
Currently: <root> → Interview Questions/  (Obsidian, wikilinks)
New path, or keep?
→ [wait for answer]
```

Number the steps (`Step 2/5`) so the user knows how far in they are.

**Incremental and never destructive:**
- Unanswered steps keep their existing value — no step silently resets anything.
- **Never delete existing questions, history, or `index.md`.** A changed `root` offers to *move* files, never wipe them.
- At the end, show a **diff of what actually changed** (or "nothing changed") and confirm before writing.
- If the user says "skip"/"keep" on every step, write nothing and say so.

**Jumping straight to one setting is still allowed** — if the user opens with a specific ask ("tick Go", "3 per day"), do just that, confirm, and don't run the wizard. The step-by-step walk is what `/iq init` with no further detail means.

### Step A — Scope

> Global (`~/.claude/`, same stack in every project) or project-local (`./`, per-repo stack)? **Recommended: global.**

Then detect the agent-instruction file (`CLAUDE.md` / `AGENTS.md` / `GEMINI.md`) at that scope and append the pointer block. If several exist, ask. If none exists, create the one matching the tool in use.

### Step B — Storage

> Where should the questions live?
> **(a)** an existing notes vault / Obsidian folder — give the path
> **(b)** a plain folder (e.g. `~/interview-prep`) — I'll create it
> **(c)** the current repo (`./interview-prep`)

- Expand `~`, resolve absolute, `mkdir -p` the questions dir.
- **Auto-detect `linkStyle`**: a `.obsidian/` dir at or above `root` → `wikilink`, else `markdown`. State the choice so it can be overridden.
- Don't ask about `articlesRoot` here — defer it until the first article is written.

### Step C — Daily volume

> How many **new** questions per day? **Recommended: 5.**
> Plus 1 carry-over (an unanswered one from history) and 1 review (spaced repetition)? **Recommended: both on.**

Store as `newPerDay`, `carryOver`, `review`. Make it explicit that the number means *new* questions — the other two slots are counted separately.

### Step D — Resume (recommended — say why)

> "Do you want to give me your resume? **Recommended.** It lets me ask about the projects you actually shipped — 'on X you used Y, what broke, how did you handle Z' — which is what a real interview does. Without it I can only ask generic tech questions.
> Paste it, give me a file path, or say skip."

**If given:**
1. Save to `<questionsDir>/resume.md` (if one already exists there, offer to read that instead of re-pasting).
2. **Extract the stack from it** — languages, frameworks, databases, cloud, infra, tooling.
3. **Present the extracted list and ask the user to confirm** — select a subset, or all. Never assume everything on a resume is interview-ready; people list things they touched once.
4. **Extract the projects** too — name, one line, tech used. These feed `projectQuestionsPerDay`.
5. Ask: how many of the daily questions should be project-grounded? **Recommended: 1 of 5.** `0` disables.

**If skipped:**
> "Then just tell me what you've actually worked with — language(s), database(s), infra, anything else you'd expect to be interviewed on."

Build the stack from that. Set `projectQuestionsPerDay: 0` — there's nothing to ground project questions in. Mention the resume can be added later.

### Step E — Seniority

> Target level: **mid / senior / staff**? Drives difficulty. **Recommended: senior.**

### Then

- Write `iq-config.md`.
- Append the pointer to the agent-instruction file.
- Show a compact summary (scope, path, counts, stack, seniority) and confirm.
- Offer to start today's set.

> **Portability rule:** `root` in `iq-config.md` is the ONLY absolute path anywhere. Never hardcode one in this file, never write one into a note. Everything else is `<root>`-relative so the vault can move, be renamed, or belong to someone else.

## Step 1 — Determine today's date
Run `date +%F` → `<DATE>`. Compute `DAYFILE = "<root>/<questionsDir>/days/<DATE>.md"`.

## Step 2 — Ensure today's set exists (build only if missing)
**If `DAYFILE` already exists, SKIP building** and go to Step 3 (resume the loop). This is what makes it idempotent — any number of sessions on the same day reuse the same set.

If `DAYFILE` does NOT exist, build it:

0. **Reconcile `index.md` against the day files FIRST** (drift repair). `index.md` is the pool tracker, but it's edited separately from the day files, so it can fall out of sync — a question answered in `days/<date>.md` whose `index.md` line was never flipped silently vanishes from the review pool (and stays wrongly in the carry-over pool). Before reading the pools:
   - `grep` the day files for `status: answered` question headings.
   - For each, verify its `index.md` line is `- [x]`. If it's still `- [ ]`, **flip it**.
   - Do the reverse check too: a line marked `- [x]` whose day-file question is `status: pending` → un-tick it.
   - Report the repairs in one line (`Fixed N index drift entries`), don't make it a ceremony.
   - Only then read the pools. Otherwise the very first build after any drift picks the wrong Q6/Q7 — or skips Q7 entirely because the answered pool looks empty.
1. **Read `index.md`.** Split existing entries into two pools:
   - `unansweredPool` = every `- [ ]` line (with its text + `[[days/<date>]]` link)
   - `answeredPool`   = every `- [x]` line
2. **Generate the `newPerDay` NEW questions.** Produce ~1.5× that many candidates, drawn from the ticked **stack** + `alwaysOnTopics`, at `seniority` difficulty, English — each with topic, difficulty, question text, and a strong model answer. Of these, `projectQuestionsPerDay` must be **project-grounded** (from the config's `## Projects`).
3. **Dedup with a cheap agent.** Spawn ONE sub-agent on the **Haiku** model (Agent tool, `model: haiku`, `subagent_type: general-purpose`). Give it the candidate texts + every question text from `index.md`; it returns the candidates that are **NOT semantic duplicates** (reworded repeats count), as JSON indices + a one-line reason per drop. Keep unique ones; regenerate + re-check until you have `newPerDay` unique new questions.
4. **Pick the CARRIED question** (skip if `carryOver: false`). If `unansweredPool` is non-empty, choose **one at random**. Open its linked day file, locate that question by its text, and copy its question text + model answer. Remember its exact `index.md` line. If the pool is empty, **skip the slot**.
5. **Pick the REVIEW question** (skip if `review: false`). If `answeredPool` is non-empty, choose **one at random** (necessarily different from the carried one, since the pools are disjoint). Open its linked day file, copy its question text + model answer. Remember its `index.md` line. If empty, **skip the slot**.
6. **Compute `total`** = `newPerDay` + (carried present ? 1 : 0) + (review present ? 1 : 0).
7. **Write `DAYFILE`** from the template — the new ones `kind: new`, then `kind: carried` (note origin date), then `kind: review` (note origin date); every question `status: pending`; frontmatter `answered: 0`, `total: <total>`.
8. **Append ONLY the new questions to `index.md`** as `- [ ]` lines (Index template). Do this **after** picking carried/review so today's new ones can't be selected as today's carry-over. **Do NOT append the carried/review ones** — they're already in the index.

Generation stays in the **main model** (quality matters); only dedup runs on Haiku.

## Step 3 — Run the quiz loop (one question at a time)
Find the first question in `DAYFILE` with `status: pending`. For each, in order:
1. Show **only** the question — number, topic, difficulty, its **kind tag** (`new` / `carried` / `review`), and text. **Do not reveal the answer.**
2. Wait for the user. They may answer, or say "show answer" / "I don't know" / "skip".
3. Reveal the **model answer**, then give **feedback**: what they nailed, what they missed, one tip. **Calibrate the depth to the miss** (see *Teach-on-miss* below). (If skipped, just show the model answer.)
4. **Persist, by kind:**
   - **`new` and `carried`:** fill **Your answer** + **Feedback** in `DAYFILE`, set that question's `status: answered`, increment frontmatter `answered:` by 1, and **flip its `index.md` line `- [ ]` → `- [x]`** (for a carried question, that's the original prior-day line you remembered).
   - **`review`:** fill **Your answer** + **Feedback** in `DAYFILE`, set `status: answered`, increment `answered:` — but **DO NOT touch the original question's saved answer.** Leave its `index.md` line `- [x]`; just append ` (reviewed: <DATE>)` to that line for spaced-repetition tracking.
5. Move to the next pending question.

When `answered == total`, congratulate and stop. Tomorrow is a new date → new set automatically.

---

## Teach-on-miss (depth calibration)

The stored **Model answer** is a *reference*, not a script. How much to expand depends on what the user actually did:

| Situation | Response |
| --- | --- |
| **Correct + well-reasoned** | Confirm briefly, add one senior nuance. Don't pad. |
| **Correct conclusion, weak reasoning** | Name the stronger framing explicitly — "you said X *works*; the reason interviewers listen for is Y". |
| **Partially right / one misconception** | Isolate the misconception, kill it with a concrete counter-example, THEN give the model answer. Misconception-first — don't bury it under correct material. |
| **"I don't know" / "show answer"** | **Teach it properly.** Motivation → intuition → mechanism → worked example → caveats. Not a definition dump. |
| **Unfamiliar term in the question itself** | Stop, explain the term from first principles with an example, then re-ask the question. Don't reveal the answer yet — the user hasn't attempted it. |

Follow-up questions ("explain more", "give an example", "what if N columns") are **part of the same question** — keep teaching, don't push to the next question. Advance only when the user says next/skip.

Use concrete, runnable examples over abstractions: real tables with real rows, real code, real `EXPLAIN` output, a timeline of events. Prefer showing the mechanism over stating the rule.

**Persisted feedback should capture the correction, not the lecture.** Write the 2–5 sentences that would let a future re-read reconstruct the gap — the misconception, the fix, and the phrase that signals seniority. The full explanation belongs in an article (below), not in the day file.

---

## Article flow (on request)

When the user asks to turn an explanation into an article ("make this an article", "write this in obsidian", "save this"):

1. **Place it by topic** under `<root>/<articlesRoot>/<Area>/<Topic>.md`, next to its siblings. **Read the target folder first** and match whatever convention is already there (naming, nesting, index notes). If `articlesRoot` isn't set in `iq-config.md` yet, ask once where articles should live, then persist it.
2. **Frontmatter** — `title`, `created: <DATE>`, `tags`, `source` (pointing at the day file), and a `related:` list of links.
3. **Backlink to the question** — a line near the top: `Source: quiz question Q<n> in <link-to-day-file> (/iq).`
4. **Forward-link from the question** — add to that question's block in `DAYFILE`, directly above **Feedback**:
   ```
   **📄 Article:** <link-to-article>
   ```
5. **Register it in the folder's index note** if one exists (e.g. a topic MOC), matching the existing list style. If none exists, skip — don't invent one.
6. **Stub links are encouraged** (`linkStyle: wikilink` only). If the article mentions a sibling topic with no note yet, link it anyway — an unresolved wikilink marks future work, and when that note is written the link resolves and content can move there. Mention which stubs were created. Under `markdown` link style, write the sibling as **plain bold text** instead — a dangling relative link is just a 404, not a TODO.

**Link rendering** — obey `config.linkStyle` everywhere (day files, index, articles):

| `linkStyle` | Link | Backlink to day file |
| --- | --- | --- |
| `wikilink` | `[[Article Title]]` | `[[<DATE>]]` |
| `markdown` | `[Article Title](../../<path>.md)` | `[<DATE>](../days/<DATE>.md)` — relative to the file being written |

The article carries the full teaching depth; the day file's Feedback stays compact and links out.

---

## Templates

### Day file — `days/<DATE>.md`
```
---
date: <DATE>
type: interview-quiz
total: <5–7>
answered: 0
---

# Interview Quiz — <DATE>

## Q1 · [<topic> · <difficulty>] · new · status: pending
**Question:** <question text>

**Your answer:** _(pending)_

**Feedback:** _(pending)_

**Model answer:** <model answer>

---
```
(`**📄 Article:** [[…]]` is inserted above **Feedback** only when an article is written for that question — see *Article flow*.)
(Q2–Q5 = `new`. Then, if present:)
```
## Q6 · [<topic> · <difficulty>] · carried from <origin-date> · status: pending
...same fields...

## Q7 · [<topic> · <difficulty>] · review from <origin-date> · status: pending
...same fields...
```

### Index line — append to `index.md` (the NEW questions only)
```
- [ ] <DATE> · <topic> · <question text> — <link to day file>
```
Link per `config.linkStyle`: `[[days/<DATE>]]` (wikilink) or `[<DATE>](days/<DATE>.md)` (markdown).

Append-only. `- [ ]` = unanswered pool, `- [x]` = answered pool. The question text is what the dedup agent compares against.

---

## Resume-intensive session (on demand)
Project-grounded questions already appear in the daily set (`projectQuestionsPerDay`). But if the user wants a **full resume drill** before a specific interview, read `<root>/<questionsDir>/resume.md`, generate a whole set grounded in those projects, dedup against `index.md`, and save to a labeled file (e.g. `<questionsDir>/resume-<DATE>.md`). Keep it out of the daily flow and out of the daily pools. If `resume.md` doesn't exist, offer `/iq init` to add one.

## Notes
- Cost happens **only when this skill runs**, never on plain session starts. (An optional session-start nudge can remind the user to run `/iq` — it's a shell hook, not part of this skill, so it costs nothing.)
- If `index.md` is missing, create it from the standard header before appending.
- Pools come straight from `index.md` checkboxes, so carry-over/review need no extra bookkeeping file.
- **Nothing outside `iq-config.md` may contain an absolute path.** Its `root` key is the only machine-specific value — everything else is portable.
- On a fresh install the only required input is `root`; every other key has a working default.
