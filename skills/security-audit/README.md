# `security-audit` — A Security Checklist You Fill With Evidence

One master security checklist, copied into each project and ticked from real proof (code, live `curl`, tests, tool output). The agent tells you what is done, what is missing, where, and the one-line fix.

```bash
npx skills add mdaneshjoo/skills --skill security-audit
```

Installs only this skill — nothing else from the repo comes with it.

---

## What it does

| | |
| --- | --- |
| **Master checklist** | [`checklist.md`](checklist.md): 30 sections from live surface, auth and authorization to PWA, uploads, SSRF and incident response, with an OWASP map. |
| **Per project** | Copies it to `docs/security-checklist.md` and keeps it there; re-runs update it in place. |
| **Evidence or nothing** | Every tick has `path:line`, a command and its output, or a test. Every gap has where + a fix. |
| **Live checks** | Gentle `curl` probes of the deployed app: headers, cookies, CORS, `401`s, rate limits. |
| **Report** | Counts per section (measured), then the top fixes by risk. |

## Use

Ask in your own words, for example:

- "Run a security check on this project"
- "Is the API safe before we launch?"
- "Security audit before the availability check"

## Make it a habit

Add a line to your global agent rules (e.g. `~/.claude/CLAUDE.md`) so security checks always go through the skill:

```markdown
# Security checks → security-audit skill (all projects)
When the user asks for a security check / audit / review, or before a launch or visibility check:
use the `security-audit` skill and fill `docs/security-checklist.md` in the project.
```

## Credits

Built from checklists used in real NestJS / Next.js projects, plus a per-module "sensitive data in responses" audit method.
