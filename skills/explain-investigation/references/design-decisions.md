# Explain Investigation — Design Decisions

Captured from the initial user-guided `grill-me` refinement session.

## Source resolution

Priority order:

1. User-provided file path/address.
2. Uploaded attachment.
3. Pasted investigation text.
4. URL/link to investigation text.
5. Current session context.

Do not search Obsidian, project folders, or the filesystem automatically unless the user explicitly asks to search. If no source exists in explicit input or current session context, say no investigation source was found and ask the user to provide one.

## Walkthrough flow

Default mode is one finding at a time. After each finding:

- If the user agrees and says to continue/go next, move to the next finding.
- If the user asks a question, answer it from the investigation source/context, then ask whether to continue.
- If the user disagrees with the explanation, revise the explanation only.
- If the user says the investigation source itself is wrong and explicitly asks for correction/update, confirm exact edit scope before modifying the source.

Full/all-at-once explanation is allowed only when explicitly requested.

## Implementation boundary

During investigation explanation, mention only implications directly supported by evidence. Do not write code, edit files, or create a full implementation plan unless the user asks after the explanation.

## Evidence and safety

Preserve technical evidence as exactly as possible: status codes, endpoint names, response shape, error messages, config keys, table/entity names, and non-sensitive values.

Redact usable secrets/credentials and sensitive personal data while keeping field names and structure, e.g. `client_secret: [REDACTED]`.

## Confidence

Each finding should label confidence:

- Confirmed
- Likely
- Unknown / needs validation
- Not proven

Use the label to avoid turning auth failures, timeouts, firewall blocks, missing credentials, or environment-specific probe failures into confirmed code bugs.
