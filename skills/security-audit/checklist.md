# Security checklist

Reusable for any web app, API, PWA or mobile backend. Copy it into the project as `docs/security-checklist.md` and fill it from evidence (code read, live `curl`, tests, tool output). Not from memory.

Marks: `[x]` done · `[ ]` missing · `[~]` partial · `N/A` (with a reason). Every `[ ]` / `[~]` gets **where** (`path:line` or URL) and a **one-line fix**.

Examples name common stacks (NestJS, Express, Next.js, TypeORM, Drizzle, Prisma); use the equivalent in yours. Thresholds are the cited standards' values (OWASP, NIST), not our measurements.

**Order of work:** 0 → 1 → 2 → 3 → 9 first (what an attacker hits first), then 4–8, 10–13, then the rest. Re-run before every release.

---

## 0. Live surface (check the deployed app, not only the code)

- [ ] List every public host that reaches the API or app (custom domains, platform hosts like `*.vercel.app`, `*.liara.run`, `*.herokuapp.com`). Each one is attack surface; remove the ones nothing needs
- [ ] Internal services (DB, Redis, admin, API reached only by the web app) are on a private network, not public
- [ ] Probe from outside with `curl`: unauthenticated data route → `401`; another origin's preflight → no `Access-Control-Allow-Origin`; wrong login repeated → eventually `429`
- [ ] No debug / docs / admin endpoints open in production (`/swagger`, `/graphql` playground, `/admin`, `/.env`, `/debug`), or they are behind auth
- [ ] Error pages and `404`s reveal no framework, version or stack trace

## 1. Authentication

- [ ] Passwords hashed with **argon2id** or **bcrypt** (cost ≥ 12). Never MD5 / SHA-1 / plain
- [ ] Password policy on sign-up and change (minimum length; breach check e.g. HIBP if possible)
- [ ] Same error for unknown user and wrong password, and similar timing (no user enumeration)
- [ ] Account lockout or growing delay after repeated failed logins
- [ ] Session token: short-lived access token, or a session the server can revoke
- [ ] Refresh tokens rotated on every use and stored server-side (revocable, bound to device)
- [ ] Token in an **httpOnly + Secure + SameSite** cookie, not `localStorage`
- [ ] JWT secret ≥ 256 bits (or RS256 / EdDSA keys) from env; algorithm pinned (no `none`); `exp`, `iat`, `sub` (and `iss`, `aud` if used) validated
- [ ] Logout ends the session server-side (not only clears the cookie)
- [ ] Password change / reset ends other sessions (session version, `passwordChangedAt`, or token family)
- [ ] OTP / magic links: single use, short expiry (≤ 5 min for OTP), hashed at rest, rate-limited per phone/email and per IP
- [ ] MFA available for admin and sensitive accounts
- [ ] Verification of phone/email before actions that need it

## 2. Authorization

- [ ] Auth guard applied globally; every public route is marked on purpose and listed
- [ ] **Object-level checks (IDOR / BOLA):** every read and write filters by the owner (`userId`, tenant) on the server. Prove it with tests: user B cannot read, change or delete user A's rows by id
- [ ] Function-level checks: admin / privileged routes gated by role or permission, server-side only
- [ ] Never trust ids, roles or prices from the client
- [ ] Shared resources (shared budgets, teams, orgs): membership and role checked on every endpoint
- [ ] Least privilege for DB users, service accounts, cloud keys

## 3. Input validation and output encoding

- [ ] Every body, query and route param validated with a schema (zod, class-validator, …); unknown fields stripped or rejected
- [ ] Limits on string lengths, array sizes, numbers (amounts, counts, pages)
- [ ] Malformed ids return `404` / `400`, never `500`
- [ ] Env config validated at startup (missing secret = crash, not a silent default)
- [ ] Output encoded in HTML / email templates; no raw HTML from users (`dangerouslySetInnerHTML`, `v-html`) without a sanitizer
- [ ] User input never reaches shell commands, regex construction, file paths, LDAP / XPath without escaping

## 4. Injection and data access

- [ ] Only parameterized queries / query builders. No string-built SQL (`sql.raw`, `$queryRawUnsafe`, template strings into `.query()`)
- [ ] NoSQL operator injection blocked (`$where`, `$ne` in user input) where applicable
- [ ] ORM auto-sync off outside local dev; schema changes only through migrations
- [ ] App DB role is not superuser
- [ ] Pagination has a stable tiebreaker (`ORDER BY …, id`); no unbounded list endpoints
- [ ] Timestamps stored as `timestamptz` (UTC)

## 5. Secrets and configuration

- [ ] No secrets in git, now or in history (`gitleaks detect`); anything ever committed is rotated
- [ ] `.env*` ignored; `.env.example` has placeholders only
- [ ] Secrets in the platform's env / secret manager, different per environment
- [ ] `process.env` read only in config code, not scattered through features
- [ ] Rotation plan for JWT secret, DB password, API keys, push / VAPID keys
- [ ] Public-by-design keys (VAPID public key, analytics site id) are the only keys shipped to the browser

## 6. Transport and network

- [ ] HTTPS only; HTTP → HTTPS redirect (301) at the edge
- [ ] HSTS (`max-age ≥ 31536000; includeSubDomains`; `preload` when ready)
- [ ] TLS ≥ 1.2, no weak ciphers
- [ ] DB connection over TLS or a private network
- [ ] Service-to-service traffic on a private network

## 7. HTTP security headers and CORS

- [ ] `Content-Security-Policy` (start report-only, then enforce); `frame-ancestors 'none'` or `X-Frame-Options: DENY`
- [ ] `X-Content-Type-Options: nosniff`
- [ ] `Referrer-Policy: strict-origin-when-cross-origin`
- [ ] `Permissions-Policy` turns off what the app doesn't use (camera, mic, geolocation, …)
- [ ] `Cross-Origin-Opener-Policy: same-origin`
- [ ] Version headers removed (`X-Powered-By`, `Server` versions): helmet / `poweredByHeader: false`
- [ ] **CORS:** none if the app is same-origin; otherwise an explicit allow-list, never `*` with credentials

## 8. CSRF

- [ ] Cookie-auth state changes protected: `SameSite=Lax/Strict` cookie **and** JSON-only bodies, or a CSRF token, or an `Origin` check
- [ ] `GET` never changes state

## 9. Rate limiting and abuse

- [ ] Strict limits on login, sign-up, password change / reset, OTP send (per IP **and** per account / phone)
- [ ] General limit per IP and per user on the rest of the API
- [ ] Request body size limit (e.g. 1 MB)
- [ ] Expensive endpoints (exports, AI calls, uploads, reports) have their own limits or quotas
- [ ] CAPTCHA or proof-of-work where bots are a real risk (sign-up, contact forms)
- [ ] Limits work behind the proxy (trust the right `X-Forwarded-For` hop, not any client header)

## 10. Logging, monitoring, alerting

- [ ] Structured logs; no `console.log` left in production paths
- [ ] Secrets and PII redacted from logs (password, token, cookie, authorization, OTP, phone, card)
- [ ] Request id on every log line
- [ ] Security events logged: login success/failure, permission denied, password / role change, admin actions
- [ ] Error monitoring in production (Sentry / GlitchTip) with PII scrubbing
- [ ] Alerts on auth-failure spikes and 5xx surges
- [ ] Log retention and access rules

## 11. Error handling

- [ ] One error shape for all errors; messages safe to show
- [ ] No stack traces, SQL or internal paths in production responses
- [ ] DB / provider errors mapped to generic errors before the response
- [ ] Status codes don't reveal whether a user / resource exists to someone without access

## 12. Sessions and cookies

- [ ] Cookie flags: `HttpOnly`, `Secure` (prod), `SameSite`, path / domain scoped, expiry set. Check the live `Set-Cookie`, not only the code
- [ ] Sessions end on password change, role change, account deletion
- [ ] **Push / device tokens follow the session, server-side:** each push subscription (Web Push, FCM, APNs token) is stored with the session that registered it. Logout, password change (other sessions), session expiry and account deletion stop notifications on the server — never rely only on the app unsubscribing (it can be offline, crash, or be uninstalled). Test it: log out, then send a test push; nothing should arrive
- [ ] Users can see and end their other sessions (when the app has multiple devices)

## 13. Data protection and privacy

- [ ] Sensitive columns never serialized (password hash, tokens, OTP, internal flags)
- [ ] PII encrypted at rest where the law or risk needs it
- [ ] Backups automatic and encrypted; **a restore was tested**
- [ ] Data minimization: collect and log only what is needed
- [ ] Privacy policy and terms published and true to what the app does
- [ ] User can export and **delete** their account and data (needed by GDPR-style laws and app stores)
- [ ] Analytics / tracking: no money amounts, titles or PII sent to third parties

## 14. Sensitive data in responses (per module)

Method from a real audit: list sensitive fields once, then check every module's responses against the list.

- [ ] Table of sensitive fields per model (e.g. user: `passwordHash`, `phone` of other users, tokens, internal ids) with risk level
- [ ] Each module's responses checked: returns only fields the caller may see (owner vs. other user vs. public)
- [ ] Explicit response mappers / DTOs (`toUser()`), never raw DB rows or `...row`
- [ ] Lists that embed other users (members, participants, invites) return the public subset only
- [ ] API docs (Swagger / OpenAPI) match the safe shapes

## 15. Dependencies and supply chain

- [ ] `npm/pnpm audit` (or the stack's equivalent) clean or triaged, run in CI
- [ ] Lockfile committed; installs are frozen in CI (`npm ci`, `pnpm install --frozen-lockfile`)
- [ ] Renovate / Dependabot on
- [ ] No unmaintained or typo-squatted packages; new packages reviewed
- [ ] Licenses compatible

## 16. Containers and deployment

- [ ] Base image pinned (tag + digest), rebuilt regularly
- [ ] Runs as a non-root user
- [ ] Multi-stage build; no build tools, `.env`, `.git` or source maps with secrets in the final image (`.dockerignore` / platform ignore file)
- [ ] Image scanned (Trivy / Grype)
- [ ] Migrations run once per deploy, safely (no destructive auto-migration)
- [ ] Health check and resource limits set

## 17. CI/CD and repository

- [ ] Default branch protected: no force push, required checks (lint, types, tests, secret scan)
- [ ] Deploy credentials scoped per environment and rotated
- [ ] Staging / preview never uses production data or secrets
- [ ] `SECURITY.md` with a contact for reporting issues

## 18. API design

- [ ] No secrets or PII in URLs / query strings (they end up in logs)
- [ ] Idempotency on money-moving or side-effect POSTs (retries can't double-pay)
- [ ] Consistent pagination; maximum page size
- [ ] Versioning or a deprecation path for breaking changes (needed once mobile apps ship)

## 19. Outbound requests (SSRF) and third parties

- [ ] Any URL the server fetches that came from a user (webhooks, avatars from URL, link previews, **push subscription endpoints**) goes through an allow-list or blocks private ranges (`127.0.0.0/8`, `10/8`, `172.16/12`, `192.168/16`, `169.254/16`, `::1`, `fc00::/7`)
- [ ] Redirects off or re-checked; timeouts and max response size on every outbound call
- [ ] Incoming webhooks verify signature + timestamp (no replays)
- [ ] Third-party keys scoped to the minimum

## 20. Business logic

- [ ] Money as integers (minor units) or decimal, never float
- [ ] Totals and prices computed on the server
- [ ] Race conditions on balances / payments / quotas: transactions + row locks or optimistic locking
- [ ] State transitions validated (can't pay a cancelled item, can't undo twice)
- [ ] Limits on what one user can create (rows, files, invites) to stop storage abuse

## 21. Web app and PWA (browser side)

- [ ] No tokens or secrets in `localStorage`, `sessionStorage`, JS bundles or source maps
- [ ] Source maps not public in production (or no secrets in them)
- [ ] Third-party scripts minimized; Subresource Integrity on CDN scripts
- [ ] Service worker never caches authenticated API responses for other users; cache cleared on logout
- [ ] Logout clears client state (SWR / React Query caches, IndexedDB) and unsubscribes push on the device (on top of the server-side rule in §12)
- [ ] Push / notification payloads carry no sensitive data beyond what the lock screen may show
- [ ] Open redirects blocked (`?next=` / `?url=` only to own paths)
- [ ] Links with `target="_blank"` use `rel="noopener noreferrer"`

## 22. Uploads (if any)

- [ ] Size and count limits at the edge and in the app
- [ ] Type checked by magic bytes against an allow-list; SVG / HTML / executables rejected
- [ ] Random server-side names; stored outside the web root (object storage), served via short-lived signed URLs
- [ ] Images re-encoded (strips EXIF / GPS and payloads); pixel limit before decode
- [ ] Malware scan where users share files with others
- [ ] Per-user quotas; uploads rate-limited

## 23. Cryptography

- [ ] `crypto.randomBytes` / `randomUUID` for anything secret; never `Math.random()`
- [ ] Constant-time compare for tokens, HMACs, OTPs
- [ ] AES-GCM / ChaCha20-Poly1305; never ECB; unique IV per message
- [ ] No home-made crypto

## 24. Deserialization, prototype pollution, mass assignment

- [ ] No `eval`, `new Function`, `vm` with user input
- [ ] Safe YAML / XML parsing (no external entities)
- [ ] User input never spread into DB writes (`{ ...body }`); pick fields explicitly
- [ ] User-controlled object keys use `Map` / `Object.create(null)`

## 25. Realtime / WebSocket / GraphQL (if any)

- [ ] Same auth as HTTP on connect; re-checked on reconnect; origin checked
- [ ] Every event / resolver authorized; message size and rate limits
- [ ] GraphQL: depth and complexity limits, introspection off in production

## 26. Email / SMS (if any)

- [ ] SPF, DKIM, DMARC on the sending domain
- [ ] Reset / verify links: single use, expiring, ≥ 128-bit tokens, hashed at rest
- [ ] No secrets or full card numbers in messages; templates escape user values
- [ ] SMS / OTP sending rate-limited (cost and abuse)

## 27. Cache

- [ ] Auth-scoped responses `Cache-Control: private, no-store`; `Vary: Cookie, Authorization`
- [ ] CDN cache keys don't mix users
- [ ] Redis / caches private and password-protected

## 28. Testing

- [ ] Tests for auth flows and guards (login, logout, expired / forged token)
- [ ] Cross-user access tests (§2)
- [ ] Negative tests: malformed, oversized, unauthorized, forbidden
- [ ] SAST in CI (Semgrep, `eslint-plugin-security`, CodeQL)
- [ ] DAST / ZAP baseline against staging before major releases
- [ ] Pentest before a big launch or yearly

## 29. Incident response

- [ ] Runbook: leaked secret, DB breach, account takeover, vulnerable dependency, DoS
- [ ] Kill switches: revoke all sessions, force-logout one user, disable sign-up
- [ ] Who gets alerted, and how

---

## OWASP map

| OWASP | Sections |
|---|---|
| API1 BOLA · A01 Broken access control | §2, §14 |
| API2 Broken auth · A07 Auth failures | §1, §12 |
| API3 Property-level auth | §14, §24 |
| API4 Resource consumption | §9, §20, §22 |
| API5 Function-level auth | §2 |
| API6 Sensitive business flows | §9, §20 |
| API7 · A10 SSRF | §19 |
| API8 Misconfiguration · A05 | §0, §5, §7, §16 |
| API9 Inventory | §0, §18 |
| API10 Unsafe API consumption | §19 |
| A02 Cryptographic failures | §6, §13, §23 |
| A03 Injection | §3, §4 |
| A04 Insecure design | §20 |
| A06 Vulnerable components | §15 |
| A08 Integrity failures | §15, §17, §24 |
| A09 Logging failures | §10 |
