# `scannable-output` — Answers You Can Read in One Glance

For readers who lose the thread in long text (ADHD-friendly). The agent still explains, but every answer comes as tables, bullets, options and steps instead of paragraphs.

```bash
npx skills add mdaneshjoo/skills --skill scannable-output
```

Installs only this skill — nothing else from the repo comes with it.

---

## What it does

| | |
| --- | --- |
| **Answer first** | Line 1 is the answer, in bold. |
| **Structure, not prose** | Tables, bullets, numbered options and steps. No long paragraphs. |
| **Bold key words** | Read only the bold parts and you get the gist. |
| **One item at a time** | Several tickets or findings? You get a summary table, then "continue?". |
| **Length cap** | Large answers are capped at about 15 lines per reply. The rest comes when you ask. |
| **Still explains** | Unlike ultra-terse modes, the depth stays. Only the shape changes. |

---

## Making it always-on

It is a format rule, so it works best loaded every session. In Claude Code, add it to `~/.claude/CLAUDE.md`:

```markdown
@~/.agents/skills/scannable-output/SKILL.md
```

Or load it in an open session with `/scannable-output`.
