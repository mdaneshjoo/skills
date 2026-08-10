# `/iq` — Daily Interview Prep

A daily interview quiz **tailored to your own stack and the projects you actually shipped** — not generic trivia.

```bash
npx skills add mdaneshjoo/skills --skill iq
```

Installs only this skill — nothing else from the repo comes with it.

---

## What it does

| | |
| --- | --- |
| **Built once per day, resumable** | Re-running the same day continues where you left off instead of regenerating. |
| **Deduped against your whole history** | Semantic dedup — a reworded repeat gets dropped, not re-asked. |
| **Spaced repetition** | Each day = N new + 1 carry-over (something you skipped) + 1 review (something you answered before). |
| **Project-grounded questions** | Give it your resume and it asks what an interviewer would actually ask *you*. |
| **Teaches on a miss** | A wrong or skipped answer gets mechanism → worked example → caveats, not a definition dump. |
| **Writes to your notes** | Obsidian vault, plain folder, or a repo. Answers and feedback are saved, so history compounds. |

A project-grounded question looks like:

> *"On **&lt;your project&gt;** you used Kafka with CQRS to split reads and writes — what does that do to read-after-write consistency, and how did you handle a user who doesn't see their own write?"*

That's the difference from a generic quiz: it drills the things you'll actually be asked to defend.

---

## Setup

```
/iq init
```

Five questions, **one at a time**:

1. **Scope** — global (same stack everywhere) or project-local (per-repo stack).
2. **Storage** — an existing notes vault, a new plain folder, or the current repo. Auto-detects Obsidian and switches link style accordingly.
3. **Daily volume** — how many *new* questions per day, and whether to enable the carry-over and review slots.
4. **Resume** *(optional, recommended)* — paste it or give a path. The skill extracts your stack, shows it back as a checklist for you to confirm, and pulls out your projects for project-grounded questions. Skip it and you get generic technical questions only.
5. **Seniority** — mid / senior / staff. Drives difficulty.

It writes an `iq-config.md` and registers its path in your agent-instruction file (`CLAUDE.md`, `AGENTS.md`, or `GEMINI.md`) so it loads whenever you run `/iq`.

### Re-configuring

```
/iq init
```

Walks the same steps with your **current values as defaults**, shows a diff of what changed, and **never touches your saved questions**. Or just say what you want — *"tick Go"*, *"3 per day"* — and it changes only that.

You can also hand-edit `iq-config.md`. The stack is a checkbox list, so ticking `- [ ] Go` → `- [x] Go` is enough to start getting Go questions.

---

## Daily use

```
/iq
```

One question at a time — topic, difficulty, and whether it's new, carried, or a review. Answer it, or say *"show answer"* / *"I don't know"* / *"skip"*.

You get graded, the answer and feedback are written to today's note, and it moves on.

**Follow-up questions are part of the same question.** Ask *"explain more"*, *"give an example"*, *"what if it were 5 columns"* — it keeps teaching and doesn't push you forward until you say next.

### Turning an answer into an article

Say *"make this an article"* on anything worth keeping. It writes a full write-up into your notes, placed by topic, with:

- frontmatter linking back to the question,
- a `📄 Article` link added to that question in the day file,
- registration in the relevant index note,
- stub links (`[[cluster]]`) for sibling topics you haven't written yet.

The day file stays compact; the article carries the depth.

---

## Configuration reference

`iq-config.md` — markdown, hand-editable, machine-local (never commit it).

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

## Stack
- [x] Node.js / TypeScript
- [ ] Go

## Projects
- **<name>** — <one line> · _<tech>_
```

| Key | Meaning |
| --- | --- |
| `root` | Where everything is written. **The only absolute path anywhere.** |
| `questionsDir` | Quiz data subfolder. |
| `articlesRoot` | Where write-ups go. |
| `linkStyle` | `wikilink` → `[[Note]]` (Obsidian/Logseq). `markdown` → relative links (plain repos, GitHub). |
| `newPerDay` | Count of **new** questions. Carry-over and review are counted separately. |
| `carryOver` / `review` | Toggle those two slots. |
| `projectQuestionsPerDay` | How many of the new ones are grounded in your real projects. Needs a resume. |
| `seniority` | Difficulty. |
| `alwaysOnTopics` | Asked regardless of stack — real interviews ask these of everyone. |

**Only ticked stack items produce questions.** That's the point: no Kubernetes questions if you've never run it.

---

## Files it creates

```
<root>/<questionsDir>/
├── days/2026-08-10.md    # today's set: questions, your answers, feedback
├── index.md              # every question ever asked — dedup source + pool tracker
└── resume.md             # optional, feeds project-grounded questions
```

`index.md` is the engine: `- [ ]` lines are the carry-over pool, `- [x]` lines are the review pool. Answering a question flips its line, moving it from one pool to the other.

---

## Notes

- **Cost only when it runs.** No background token use.
- **Idempotent per day.** Any number of sessions on the same date reuse the same set.
- **Self-repairing.** On each build it reconciles `index.md` against the day files, so a missed checkbox can't silently drain your review pool.
- **Portable.** Nothing is hardcoded — the skill works the same on someone else's machine with their vault, their stack, their resume.
