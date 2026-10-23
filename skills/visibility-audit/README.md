# `visibility-audit` — Will Your App Be Found?

One master checklist for being found and trusted — search engines, AI assistants, link previews, PWA install, app stores — copied into each project and ticked from real evidence (curl, Lighthouse, code).

```bash
npx skills add mdaneshjoo/skills --skill visibility-audit
```

Installs only this skill — nothing else from the repo comes with it.

---

## What it does

| | |
| --- | --- |
| **Master checklist** | [`checklist.md`](checklist.md): 12 sections — indexing, on-page SEO, social preview, structured data, AEO, GEO, performance, accessibility, security & trust, PWA install, app stores, measurement & off-page. |
| **Per project** | Copies it to `docs/visibility-checklist.md` and keeps it there; re-runs update it in place. |
| **Evidence or nothing** | Every tick has a URL, `path:line` or tool output. Every gap has where + a fix. |
| **Report** | Counts per section (measured), then the top fixes by impact. |
| **Security first** | Suggests the [`security-audit`](../security-audit) skill before it, if the project hasn't run one. |

## Use

Ask in your own words, for example:

- "Will my app be found? Check visibility"
- "SEO / GEO check before launch"
- "Is the landing ready to share on Telegram?"

## Make it a habit

Add a line to your global agent rules (e.g. `~/.claude/CLAUDE.md`):

```markdown
# Visibility checks → visibility-audit skill (all projects)
When the user asks for a marketing / SEO / visibility / launch-readiness check:
use the `visibility-audit` skill and fill `docs/visibility-checklist.md` in the project.
```
