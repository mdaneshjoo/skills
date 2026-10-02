# Agent Skills

Reusable skills for AI coding agents — Claude Code, Cursor, Codex, GitHub Copilot, Windsurf, Gemini, Cline, and others.

## Skills

| Skill | What it does | Install |
| --- | --- | --- |
| [`/iq`](skills/iq) | Daily interview-prep quiz tailored to your own stack and shipped projects — spaced repetition, project-grounded questions, saved to your notes. | `npx skills add mdaneshjoo/skills --skill iq` |
| [`/explain-plan`](skills/explain-plan) | Walks an implementation plan section by section, pausing for agreement on each — instead of dumping the whole plan or going straight to code. | `npx skills add mdaneshjoo/skills --skill explain-plan` |
| [`/explain-investigation`](skills/explain-investigation) | Walks a debugging report one finding at a time, keeping evidence, meaning, impact and confidence separate — and never guessing from memory. | `npx skills add mdaneshjoo/skills --skill explain-investigation` |
| [`scannable-output`](skills/scannable-output) | ADHD-friendly answer format — answer first, tables and bullets instead of prose, one item at a time. Keeps the explanation, changes its shape. | `npx skills add mdaneshjoo/skills --skill scannable-output` |

Each skill documents itself — see its folder for setup and usage.

## Installing

Pick just what you want; you don't have to take the whole repo.

```bash
npx skills add mdaneshjoo/skills --skill iq     # one skill
npx skills add mdaneshjoo/skills --list         # browse and choose interactively
npx skills add mdaneshjoo/skills --skill '*'    # all of them
```

`--skill` accepts several at once: `--skill iq --skill other`.

## Adding a skill

One folder per skill under `skills/`:

```
skills/<name>/
├── SKILL.md    # required — `name` + `description` frontmatter, then the instructions
└── README.md   # setup and usage docs for that skill
```

Then add one row to the table above. That's the only shared file a new skill touches.

## License

MIT — see [LICENSE](LICENSE).
