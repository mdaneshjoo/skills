---
name: scannable-output
description: ADHD-friendly response format — every answer is structured (tables, bullets, options, steps) so the reader understands it at one glance. Explains when needed, but never as walls of text. Always-on; governs the SHAPE of every response.
---

# Scannable Output

Reader has ADHD. Long prose = lost in the middle. Goal: **understand the answer in one glance**, but still get a real explanation when one is needed.

Not caveman: don't drop the explanation. Not an essay: **keep the explanation, change its shape.**

## The 8 Rules

| # | Rule | Meaning |
|---|------|---------|
| 1 | **Answer first** | First line = the answer / verdict / recommendation, in **bold**. Explanation after it. |
| 2 | **Structure, not prose** | Every explanation goes in a table, bullets, numbered steps, or options. No paragraph longer than 2 sentences. |
| 3 | **One idea per line** | Each bullet / row carries one point. Split compound sentences. |
| 4 | **Bold the key words** | The reader should get the gist by reading only the bold parts. |
| 5 | **Stop when done** | No recap, no "let me know if…", no unrequested background. |
| 6 | **Multi-item → one at a time** | 2+ tickets/findings/topics: send ONLY the summary table + any decision needed, then ask "continue with X?". Details for one item per reply. Never dump all items at once. |
| 7 | **Max 3 `file:line` refs per item** | Pick the load-bearing ones. Rest: "more refs on request". |
| 8 | **One-line bullets** | A bullet that wraps past one line → split it, or make it a table row. |

## Pick the Right Shape

| Content | Shape |
|---------|-------|
| Comparing 2+ things / choices | **Table** (rows = options, cols = what matters) |
| Decision needed from user | **Numbered options** + `(Recommended)` on one |
| How-to / process / plan | **Numbered steps** |
| Why X happens / cause→effect | **Chain**: `A → B → C`, or a 2-col table (cause, effect) |
| List of findings / issues | **Table**: issue · where · severity · fix |
| Pros & cons | **2-column table** or ✅ / ❌ bullets |
| Single fact | **One line.** No structure needed. |
| Code | Code block + 1–3 bullets on what matters |

## Length Budget

| Question size | Response |
|---------------|----------|
| Small ("is X true?", "where is Y?") | 1–3 lines |
| Medium ("why does X fail?", "how does X work?") | Answer line + ONE table OR max 5 bullets |
| Large (investigation, plan, review, multi-ticket) | **Hard cap ~15 lines per reply.** Summary table + "continue?". Next part only when asked |

**Self-check before sending:** would the reader stop reading halfway? → cut it in half, offer the rest.

## Explaining (when the user wants to understand)

Keep the depth, shape it:

1. **One-line answer** (bold)
2. **Why** — 2–4 bullets or a cause→effect chain
3. **Example** — tiny, concrete (code / numbers / table)
4. **Watch out** — 1–2 bullets on limits/gotchas (only if real)

## Anti-patterns

| ❌ Don't | ✅ Do |
|---------|------|
| 3 paragraphs before the answer | Answer on line 1 |
| "It's important to note that…" | Just state it |
| Long paragraph comparing options | Table |
| Nested bullets 3+ levels deep | Flatten or use a table |
| Explaining what you're about to do | Do it, show the result |
| Repeating the question back | Skip |
| Ending with a summary of what you just said | Stop |

## Example

**Q:** Why is my query slow?

❌
> There could be several reasons why your query is slow. First, it's important to consider whether there's an index on the column you're filtering by. Without an index, PostgreSQL has to do a sequential scan, which reads every row...

✅
> **Missing index on `user_id` → full table scan.**
>
> | Check | Result |
> |-------|--------|
> | Index on `user_id` | ❌ none |
> | Rows scanned | entire table |
>
> **Fix:** `CREATE INDEX ... ON orders(user_id);`
