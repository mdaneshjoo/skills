# Agent Skills

Reusable skills for AI coding agents (Claude Code, Cursor, Codex, Copilot, Windsurf, Gemini, Cline, …).

```bash
npx skills add mdaneshjoo/skills
```

## Skills

### `/iq` — Daily Interview Prep

A daily interview quiz **tailored to your own stack and the projects you actually shipped** — not generic trivia.

- **Built once per day, resumable.** Re-running the same day continues where you left off instead of regenerating.
- **Deduped against everything you've ever been asked**, semantically — a reworded repeat gets dropped.
- **Spaced repetition built in.** Each day = N new questions + 1 carry-over (something you skipped) + 1 review (something you answered before).
- **Project-grounded questions.** Give it your resume and it asks what an interviewer would actually ask *you*: *"On <project> you used Kafka with CQRS — what happens to read-after-write consistency, and how did you handle it?"*
- **Teaches on a miss.** A wrong or skipped answer gets a proper explanation — mechanism, worked example, caveats — not a definition dump.
- **Writes to your notes.** Obsidian vault, plain folder, or a repo. Answers and feedback are saved so the history compounds.

#### Setup

```
/iq init
```

Five questions, one at a time: where to store things, how many questions a day, your resume (optional but recommended), and target seniority. It writes an `iq-config.md` you can hand-edit afterwards — the stack is a checkbox list, so you tick what you want to be drilled on.

Re-run `/iq init` any time to change something. It walks the same steps with your current values as defaults, and never touches your saved questions.

#### Daily use

```
/iq
```

One question at a time. Answer it, get graded, move on. Say *"make this an article"* on anything worth keeping and it writes a full write-up into your notes, cross-linked with the question.

#### Nothing is hardcoded

Storage path, question volume, stack, seniority, and link style (`[[wikilink]]` vs relative markdown) all live in `iq-config.md`, scoped **global** or **per-project**. Its path is registered in your agent-instruction file (`CLAUDE.md` / `AGENTS.md` / `GEMINI.md`) so it loads on `/iq`.

Your `iq-config.md` is machine-local and is **not** part of this repo — it holds your paths and your resume-derived stack.

## Adding a skill to this repo

One folder per skill under `skills/`, each with a `SKILL.md` carrying `name` + `description` frontmatter.

## License

MIT
