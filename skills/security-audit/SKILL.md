---
name: security-audit
description: Security check of a project against a reusable checklist — copy the master checklist into the project as docs/security-checklist.md, audit every section from real evidence (code read, live curl, tests, tool output), tick each item with where + fix, and report the top fixes. Use when the user asks for a security check, security audit, security review, "is my app secure", "check security before release / launch / availability check", OWASP check, pentest prep, or a hardening pass. Also use before a visibility / launch check.
---

# Security Audit

Fill a per-project security checklist from **evidence**, not memory. Same flow as a visibility check: one master list, copied into each project, ticked with proof.

The master checklist is [`checklist.md`](checklist.md) next to this file.

## Steps

1. **Copy** `checklist.md` into the project as `docs/security-checklist.md`.
   - Already there → update it in place. Never untick or re-tick an item without re-checking it.
   - Add a header line: date, commit (`git rev-parse --short HEAD`), and which hosts were probed.
2. **Map the stack** (one table at the top of the project file): language/framework, auth method, DB + ORM, hosting, public hosts, what talks to what. Mark whole sections `N/A` with a reason (e.g. "§22 Uploads — N/A: no file upload endpoint").
3. **Audit section by section**, order from the checklist (§0 → §1 → §2 → §3 → §9 first). For each item record one mark and its evidence:

   | Mark | Meaning | Must include |
   |---|---|---|
   | `[x]` | done | evidence: `path:line`, command + output, or test name |
   | `[~]` | partial | what is missing |
   | `[ ]` | missing | **where** (`path:line` or URL) + **one-line fix** |
   | `N/A` | not applicable | reason |

4. **Report**: a summary table per section (done / partial / missing / N/A — counted, not estimated), then the **top fixes by risk**. Details one section at a time when asked.
5. **Offer** to turn the top fixes into issues / tasks. Don't create them unasked.
6. If a useful rule comes up that the master lacks, **propose adding it to `checklist.md`** (master), not only to the project copy.

## How to gather evidence

| Kind | How |
|---|---|
| Code | Read the code; search for the pattern (`enableCors`, `helmet`, `sql.raw`, `Math.random`, `dangerouslySetInnerHTML`, `localStorage.setItem`, `...body`). Cite `path:line` |
| Live (§0, §6, §7, §9, §12) | `curl -sI` for headers and cookies; preflight with a foreign `Origin`; unauthenticated data route; a few wrong logins to see if `429` comes |
| Sessions & push (§1, §12) | Log in on two devices; change the password on one → the other gets `401` and **no more pushes**; log out → a test push to that user reaches no device |
| Ownership (§2, §14) | Two test accounts: B tries A's ids on every `:id` route. Prefer adding these as e2e tests |
| Dependencies (§15) | `pnpm audit` / `npm audit` / stack equivalent; paste counts by severity |
| Secrets (§5) | `gitleaks detect` (history too) or a grep for key patterns; list env var **names** only |
| Platform (§0, §16) | Hosting CLI: env var names, domains, ignore files (`.dockerignore`, platform ignore) |

## Specialists per section

Use these when available; they are optional.

| Sections | Skill / agent |
|---|---|
| §1, §2, §12, §14 | an AppSec / identity agent; `security-review` skill |
| §5 | a secrets / credential-hygiene agent; `gitleaks` |
| §7, §21 | `best-practices` skill (headers, CSP, browser security) |
| §15, §16, §17 | a DevOps / cloud-security agent |
| Whole codebase (AI-written code) | an AI-generated-code security auditor agent |
| Bug sweep alongside | `code-review` skill |

## Rules

- **Numbers only when measured.** Counts come from the filled file; "x of y items" with the real y. No "~80% secure".
- **Never print secret values.** Env var names only; mask anything that looks like a key or URL with credentials.
- **Live probing stays gentle.** Production: a handful of requests per check, your own test accounts only, no load tests, no exploit payloads against real users' data. Heavier tests (rate-limit floods, ZAP, pentest) go to staging or need the user's explicit OK.
- **Don't fix during the audit** unless asked. Audit first, then the user picks fixes.
- **Clean up** test accounts / data you created, and say you did.
