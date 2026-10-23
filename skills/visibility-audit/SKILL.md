---
name: visibility-audit
description: Visibility / launch-readiness check of a web app or PWA against a reusable checklist — copy the master checklist into the project as docs/visibility-checklist.md, audit every section from real evidence (curl, Lighthouse, code read), tick each item with where + fix, and report the top fixes by impact. Use when the user asks for a marketing, visibility, discoverability, SEO, GEO, AEO, ASO, social-preview, launch-readiness or "will my app be found" check.
---

# Visibility Audit

Will people (and search engines, AI assistants, link previews, app stores) find and trust the app? Fill a per-project checklist from **evidence**, not memory.

The master checklist is [`checklist.md`](checklist.md) next to this file.

## Before you start

Security first: if the project has no recent `docs/security-checklist.md`, suggest running the `security-audit` skill before this one. A site that ranks but leaks data is worse than one that doesn't rank.

## Steps

1. **Copy** `checklist.md` into the project as `docs/visibility-checklist.md`.
   - Already there → update it in place. Never untick or re-tick an item without re-checking it.
   - Header line: date, which URLs were checked (landing, app, store pages), and how (curl, Lighthouse version, mobile/desktop, from which country — results from outside the target market are low confidence).
2. **Audit section by section** in the checklist's order (1 → 2 → 3 → 4 first). For each item record one mark and its evidence:

   | Mark | Meaning | Must include |
   |---|---|---|
   | `[x]` | done | evidence: URL + what was seen, `path:line`, or tool output |
   | `[~]` | partial | what is missing |
   | `[ ]` | missing | **where** (`path:line` or URL) + **one-line fix** |
   | `N/A` | not applicable | reason (e.g. "§11 — no store app yet") |

3. **Report**: a summary table per section (done / partial / missing / N/A — counted), then the **top fixes by impact**. Details one section at a time when asked.
4. **Offer** to turn the top fixes into issues / tasks. Don't create them unasked.
5. A rule worth keeping that the master lacks → **propose adding it to `checklist.md`** (master), not only to the project copy.

## Specialist skills per section

Use them when installed; they are optional.

| Section | Skills |
|---|---|
| 1–2 Indexing, on-page SEO | `seo-audit`, `seo` |
| 3 Social preview | `fixing-metadata`, `seo-aeo-best-practices` |
| 4 Structured data | `schema`, `seo-aeo-best-practices` |
| 5–6 AEO, GEO | `ai-seo`, `seo-aeo-best-practices` |
| 7 Performance | `web-quality-audit`, `core-web-vitals`, `performance` |
| 8 Accessibility | `accessibility-audit` |
| 9 Security & trust | `security-audit` (full check), `site-launch-checklist`, `best-practices` |
| 10 PWA install | the checklist items directly |
| 11 App stores | `aso` |
| 12 Measurement, off-page | `directory-submissions`, `site-launch-checklist` |

## How to gather evidence

| Kind | How |
|---|---|
| Headers, redirects, robots | `curl -sI` each host (bare, `www`, app, platform host); follow redirects and note 301 vs 302/307 |
| Meta, OG, JSON-LD | `curl -s` the HTML and read the `<head>`; validate JSON-LD (Rich Results Test / Schema validator) |
| Performance, a11y | Lighthouse (state version, device, network, location); one run = low confidence |
| Previews | Share the URL in the real apps (Telegram, WhatsApp, X) or their debuggers |
| Code | `path:line` for metadata, manifest, robots, sitemap, service worker |

## Rules

- **Numbers only when measured.** Lighthouse scores, sizes, counts: say what was measured, where and how. No "should load in ~2s".
- **Platform specs are theirs, not ours**: write "Google's threshold is 2.5 s", not "2.5 s is good".
- **Don't fix during the audit** unless asked. Audit first, then the user picks fixes.
