---
name: explain-investigation
description: Walk the user through an investigation/debugging report from an explicit source or current-session context, one finding at a time by default, separating evidence, meaning, impact, confidence, and uncertainty without jumping into implementation.
version: 1.1.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [investigation, debugging, root-cause-analysis, explanation, walkthrough]
    related_skills: [systematic-debugging, obsidian, grill-me, grill-with-docs]
---

# Explain Investigation

## Overview

Use this skill to explain an investigation report incrementally. The goal is to help the user understand **what was checked**, **what evidence was found**, **what it means**, **how confident we are**, and **what decisions it supports** — without dumping the whole report or prematurely starting implementation.

This is the investigation counterpart to an implementation-plan walkthrough. It is for explaining findings, not designing code changes.

See `references/design-decisions.md` for the user-guided decisions that shaped source resolution, walkthrough flow, source-edit boundaries, evidence handling, and confidence labels.

## When to Use

Use when the user asks for:

- "Explain this investigation"
- "Walk me through the findings"
- "Explain the root cause"
- "Read the investigation result"
- "Go section by section"
- "Explain what Codex/Claude found"
- "Help me understand this audit/debug report"
- "What does this Obsidian investigation mean?"

Do **not** use this when:

- The user is asking to write a new implementation plan → use `writing-plans` or `explain-plan`.
- The user is asking to implement the fix now → first confirm scope, branch/Jira/base rules if project rules require it.
- The user only wants a one-line summary → answer directly.

## Source Rules

Do **not** hunt around the filesystem automatically. Use explicit sources first, then current-session context only.

Source priority:

1. **User-provided file path/address** — read exactly that source.
2. **Uploaded attachment** — use the attached investigation/report.
3. **Pasted investigation text** — use the pasted content.
4. **URL/link to investigation text** — fetch/read the linked source if tools allow.
5. **Current session context** — use only investigation material already present in the active chat/session.

If no source is found in those places, say clearly:

> I couldn't find any investigation source. Please provide a file path, attachment, pasted text, or link.

Do not explain from memory. Do not search Obsidian, project folders, or docs unless the user explicitly gives a path/source or asks you to search.

## Domain Docs and Glossary Rules

Use `grill-with-docs` style domain awareness, but do not silently mutate docs during an explanation.

- If project/domain docs were already provided or loaded in the session, use them.
- If the user provides a project path together with the investigation source, read obvious local context docs such as `CONTEXT.md`, `CONTEXT-MAP.md`, glossary docs, or domain docs near that project.
- If no project path or docs are provided, do **not** search broadly.
- If the investigation uses unclear, overloaded, or conflicting terms, ask one clarification question before continuing.
- Do **not** update `CONTEXT.md` automatically during explanation.
- Only offer doc/glossary updates when the user asks, or when a term is clearly resolved and worth preserving.

## Explanation Modes

Default mode is **one finding at a time**.

- Explain one finding or tightly related group.
- Stop after each section.
- Ask whether the reading is correct before continuing.

Full mode is allowed only when the user explicitly asks for it, e.g.:

- "full explanation"
- "all at once"
- "summarize everything"
- "give me the whole thing"

Even in full mode, keep the structure clear: evidence → meaning → impact → confidence/uncertainty → decision enabled.

## Required Order

1. **Resolve the investigation source** using the Source Rules above.
2. **Read the investigation source** before explaining it.
   - Read only the relevant sections if the user says "updated parts" or gives line ranges.
   - Preserve file paths and line ranges when useful.
3. **Use project/domain context** only when already loaded/provided, or when a project path/source was explicitly supplied.
4. **Ask the depth question once** unless the user already gave a clear preference:
   > "How detailed should I go — code/data-level detail, or plain flow and impact?"
5. **Break the investigation into finding sections.** Keep the outline private unless the user asks for it.
6. **Reveal one section at a time** by default, or use full mode only when explicitly requested.
7. **Pause after each section** in one-by-one mode.
8. **Produce final synthesis** only after all findings are accepted or corrected.
9. **Do not start implementation** unless the user explicitly asks after the investigation walkthrough is complete.

## Section Format

For each section, use this format:

```markdown
Section N: <short finding name>

Evidence: <what the investigation observed; include concrete APIs, files, counts, errors, or line references when useful.>
Meaning: <what this evidence implies.>
Impact: <why it matters for the product/system/debug target.>
Confidence: <Confirmed | Likely | Unknown / needs validation | Not proven> — <short reason>.
Uncertainty: <what is still unknown, blocked, or needs validation; use "None" if truly none.>

Do you agree with this reading, or should we adjust it? If yes, say "next" to continue, or ask any question about this finding first.
```

Keep each section short. Do not include the next section. Do not dump the full outline unless asked.

## Confidence Labels

Every finding should carry a confidence label:

- **Confirmed** — directly supported by source evidence.
- **Likely** — evidence strongly points there, but the source does not fully prove it.
- **Unknown / needs validation** — blocked by missing access, auth, timeout, environment, incomplete logs, or missing data.
- **Not proven** — tempting conclusion, but the investigation source does not support it.

Use these labels to avoid overclaiming. Do not turn a timeout, firewall block, auth failure, or missing credential into a confirmed code bug.

## How to Break Down Investigation Findings

Choose sections based on the report, not a fixed template. Common section types:

- **Scope checked** — which files, APIs, logs, DB queries, or environments were inspected.
- **Confirmed working paths** — things that are not broken and should not be changed blindly.
- **Broken config/path** — concrete mismatch between config and reality.
- **Blocked/unknown paths** — network block, auth failure, timeout, missing credentials, or environment-specific uncertainty.
- **Data model gap** — schema/model shape cannot represent the real-world data fully.
- **Code behavior gap** — code ignores data that exists, handles only first item, skips pagination, etc.
- **Root cause** — the smallest explanation that connects evidence to the user-visible bug.
- **Product impact** — how this creates confusing or wrong user-facing behavior.
- **Decision enabled** — what decision the investigation supports next.

## Evidence Rules

- Separate **evidence** from **interpretation**.
- Quote concrete facts: file path, line range, HTTP status, DB count, API capability, error string, config key name, entity/table name, or observed behavior.
- Preserve raw technical evidence by default: status codes, endpoint paths, response shapes, error names/messages, provider/API names, and non-sensitive values.
- Redact only values that are usable secrets/credentials or sensitive personal data:
  - API keys
  - bearer/basic tokens
  - passwords
  - client secrets
  - usable auth client IDs when paired with auth context
  - session cookies
  - private keys/certs
  - connection strings
  - PHI/PII unless explicitly safe and necessary
- When redacting, keep field names and structure, and mark the value as `[REDACTED]`.
- Mark uncertain items as uncertain. Use wording like:
  - "Likely"
  - "Needs QA/server validation"
  - "Cannot be confirmed from this machine"
- Do not overstate a timeout or firewall block as a code bug.
- Do not treat missing docs as proof an API cannot work; distinguish "not documented locally" from "live API failed".

## Implementation Boundaries

During explanation:

- Mention only implementation implications directly supported by the investigation.
- Do **not** create a full fix plan.
- Do **not** write code.
- Do **not** start implementation.

Example: say "This implies provider-specific search strategies are needed" if supported by the investigation. Do not jump into "edit file X, add class Y, run test Z" unless the user asks for planning or implementation.

If the user asks "what should we do next?" after the explanation, switch to planning mode (`writing-plans` / `explain-plan`) instead of continuing this skill.

## Disagreement Handling

If the user disagrees with a finding interpretation:

1. Stop the walkthrough.
2. Ask which part is wrong: evidence, meaning, impact, confidence, or terminology.
3. Re-read the source if needed.
4. Update the interpretation.
5. Do **not** modify the investigation source unless the user explicitly says the source information is wrong and asks you to correct/update it.
6. If the user only disagrees with your explanation, correct your explanation only; leave the investigation file/text unchanged.
7. Do not continue to the next finding until the current one is resolved.

Do not collect disagreements for the end; resolve them at the point they occur.

## User Response Handling

After each one-by-one finding, the user may respond in different ways:

- If the user agrees and asks to continue, move to the next finding.
- If the user agrees but asks a question, answer that question using the investigation source/context, then ask whether to continue.
- If the user disagrees with the explanation, follow Disagreement Handling and revise the explanation.
- If the user says the investigation source itself is wrong and explicitly asks to correct/update it, confirm the exact change scope before editing the source.
- If the user does not explicitly ask for a source edit, never modify the investigation file/text.
- Do not turn investigation explanation into implementation planning unless the user explicitly asks for a plan; investigations explain evidence/meaning/impact, plans define future code changes.

## Final Synthesis

After all sections are accepted or corrected, provide a short final synthesis:

- Root cause / main conclusion
- What is confirmed
- What remains unknown or needs validation
- What decision this enables
- Optional next step: "Do you want me to turn this into a plan?"

Do not provide final synthesis before the walkthrough is complete unless the user explicitly asks for a summary.

## Technical Depth Modes

If user wants **code/data-level detail**:

- Use exact domain names, config names, APIs, tables, entities, commands, line ranges, and failure modes.
- Explain why a code/config shape causes the observed behavior.
- Include likely implementation implications, but do not start designing full code unless asked.

If user wants **plain terms**:

- Explain flow, cause, and impact without code syntax.
- Avoid jargon unless it is a documented domain term and define it briefly.
- Focus on what users/QA/product will observe.

## Common Pitfalls

1. **Auto-searching for an investigation when no source was provided.** Do not do this. Use the current session only; if not found, ask for a source.
2. **Explaining from memory.** Prior sessions or remembered project facts are not an investigation source.
3. **Dumping the full report by default.** Default to one finding at a time unless the user explicitly asks for full mode.
4. **Turning findings into a fix plan too early.** Mention implications only; planning is a separate step.
5. **Continuing after user disagreement.** Stop and resolve the current finding first.
6. **Modifying the investigation source just because the user disagreed.** Only edit the source if the user explicitly says the investigation information is wrong and asks for a correction/update.
7. **Treating agreement as final completion.** If the user agrees and asks to go next, continue to the next finding; if they ask questions, answer before continuing.
8. **Over-redacting useful technical evidence.** Preserve response shape, status codes, error messages, endpoint names, and non-secret values.
9. **Leaking secrets because the user asks for exact output.** Exact technical output does not override secret/PII redaction.
10. **Silently updating domain docs.** Explanation mode can use docs, but doc edits need explicit user intent/approval.
11. **Overclaiming confidence.** Label blocked, timed-out, missing-auth, or environment-specific evidence as unknown/needs validation.

## Verification Checklist

Before responding:

- [ ] Investigation source was resolved using the explicit source priority.
- [ ] If no source exists, the response says no investigation source was found and asks for one.
- [ ] Relevant source was read or already present in current session context.
- [ ] Project/domain context was used only if already loaded/provided or explicitly pointed to.
- [ ] Depth preference was asked or inferred from the user.
- [ ] Response uses one finding by default, unless full mode was explicitly requested.
- [ ] Evidence, meaning, impact, and confidence are separated.
- [ ] Uncertainty is labeled clearly.
- [ ] Secrets/credentials/PHI/PII are redacted, while non-sensitive technical evidence is preserved.
- [ ] No implementation plan or code is started unless explicitly requested.
- [ ] Investigation source files/text are not modified unless the user explicitly requested a correction/update because the source information is wrong.
- [ ] In one-by-one mode, the response stops with a question asking for agreement or adjustment.
