# Visibility checklist

Reusable for any web app / PWA. Goal: the app is found in search engines, cited by AI assistants, previewed nicely when shared, and listed in app stores.

Tick per app. Sizes and thresholds below are the platforms' own published specs, not our measurements.

**Order of work:** 1 → 2 → 3 → 4 first (foundation), then 5–8, then 9–11 when the store app ships, 12 forever.

---

## 1. Domains and indexing (foundation)

- [ ] Final domain on HTTPS with a valid certificate (bare + `www` both work, one redirects to the other with **301**)
- [ ] Marketing site (landing) and app on clear hosts (e.g. `example.com` = landing, `app.example.com` = app)
- [ ] Platform default hosts (`*.vercel.app`, `*.liara.run`, …) send `X-Robots-Tag: noindex` or redirect to the real domain
- [ ] `robots.txt` on every host
  - [ ] Landing: allow all, `Sitemap:` line
  - [ ] App: `Disallow: /` (private pages) — or allow only public pages
- [ ] `sitemap.xml` on the landing (every public URL, `lastmod`)
- [ ] Every public page has a `<link rel="canonical">` to its final URL
- [ ] Login / signup / app pages: `noindex, follow` (so the landing ranks for the brand)
  - `robots.txt Disallow` alone is not enough: a blocked URL is never crawled, so its `noindex` is never seen and shared links get indexed URL-only. Send `X-Robots-Tag: noindex` and let it be crawled
- [ ] No temporary (302/307) redirects left on URLs that should rank
- [ ] `favicon.ico` + PNG icon + `apple-touch-icon` (180×180)
- [ ] Registered and verified in **Google Search Console** and **Bing Webmaster Tools**; sitemap submitted
- [ ] Bing matters twice: it also feeds ChatGPT search and Copilot

## 2. SEO — on-page

- [ ] `<html lang dir>` correct (`lang="fa" dir="rtl"` for Persian)
- [ ] Unique `<title>` per page: brand + main keyword
- [ ] Unique `meta description` per page, written as an answer to "what is this"
- [ ] Exactly one `<h1>`, then `h2`/`h3` in order
- [ ] Main keywords in the h1, first paragraph and headings, in the words users actually search (research them, e.g. «مدیریت قسط», «یادآور چک»)
- [ ] Real text in HTML (not only images / canvas / client-rendered after load)
- [ ] Images: meaningful `alt`, decorative ones `alt=""` / `aria-hidden`
- [ ] Internal links between landing sections/pages; descriptive link text
- [ ] `hreflang` only if more than one language
- [ ] Fonts self-hosted (no third-party font CDN that may be blocked or slow in the target country)

## 3. Social preview (Telegram, WhatsApp, X, LinkedIn, Slack)

- [ ] `og:title`, `og:description`, `og:url`, `og:type=website`, `og:site_name`, `og:locale` (`fa_IR`)
- [ ] `og:image` **1200×630** PNG/JPG, absolute URL, text baked into the image, plus `og:image:width/height/alt`
- [ ] `twitter:card=summary_large_image` (+ title, description, image)
- [ ] App pages that get shared (login/signup) also have OG tags
- [ ] Next.js: `metadataBase` set so relative image URLs resolve
- [ ] Tested by actually pasting the link in Telegram and WhatsApp (they cache — test before announcing)

## 4. Structured data (AEO + rich results)

- [ ] JSON-LD `SoftwareApplication` / `WebApplication`: name, `applicationCategory` (e.g. `FinanceApplication`), `operatingSystem`, `offers` (price, currency), `inLanguage`, url, image
- [ ] JSON-LD `Organization`: name (in every script the brand uses, e.g. native + Latin), url, logo, `sameAs` (social profiles, store pages)
- [ ] JSON-LD `WebSite`
- [ ] JSON-LD `FAQPage` matching a **visible** FAQ (text must be identical)
- [ ] Validated with Google Rich Results Test and validator.schema.org

## 5. AEO — answer engines (featured snippets, voice)

- [ ] A visible FAQ: real user questions as headings, 1–3 sentence direct answers first, details after
- [ ] Each "how" section opens with a short direct answer, then steps as an ordered list
- [ ] Comparison / "is it free / is it safe / does it work on iPhone" answered in plain sentences
- [ ] Tables for facts (plans, platforms, features)

## 6. GEO — being cited by AI assistants (ChatGPT, Perplexity, Gemini, Claude, AI Overviews)

- [ ] `robots.txt` allows AI crawlers you want: `GPTBot`, `OAI-SearchBot`, `ChatGPT-User`, `PerplexityBot`, `ClaudeBot`, `Google-Extended`, `Bingbot`
- [ ] One quotable "What is X" paragraph with all key facts: what it does, who it's for, price, platforms, privacy stance, maker
- [ ] Facts are consistent everywhere (site, stores, social bios, directories) — AI trusts repeated, consistent claims
- [ ] About page: who builds it, contact, since when
- [ ] `llms.txt` at the root (proposed standard; cheap, adoption by engines unconfirmed)
- [ ] Mentions on third-party sites: blog posts, reviews, directories, Q&A sites, community posts (see 12)
- [ ] Periodically ask the assistants "best app for <problem> in <language>" and check if/how you're cited

## 7. Performance (ranking signal + conversion)

- [ ] Core Web Vitals pass Google's "good" thresholds on mobile: LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1
- [ ] Lighthouse (mobile) run on landing, login and main app screen; regressions tracked
- [ ] Measured from the target country too (server and CDN location matter)
- [ ] Fonts: only needed weights/subsets, `font-display: swap`, preloaded
- [ ] Images: modern formats, explicit width/height, lazy below the fold
- [ ] Static assets cached `immutable`; HTML not over-cached
- [ ] Service worker doesn't serve stale HTML after deploys (version bump on change)

## 8. Accessibility (WCAG 2.2 AA — also helps SEO and AI parsing)

- [ ] Lighthouse a11y + axe with no serious issues
- [ ] Every input has a label; errors announced
- [ ] Icon-only buttons have accessible names
- [ ] Text contrast ≥ 4.5:1 (large text ≥ 3:1)
- [ ] Full keyboard use; visible focus; dialogs/sheets trap focus and close on Escape
- [ ] Touch targets ≥ 24×24 CSS px (44×44 recommended)
- [ ] `prefers-reduced-motion` respected
- [ ] Zoom to 200% without breaking; no horizontal scroll at 320 px

## 9. Security & trust (browsers, stores and users check these)

Quick trust pass only. The full security check is the `security-audit` skill; run it before this checklist.

- [ ] HTTPS everywhere + `Strict-Transport-Security`
- [ ] `Content-Security-Policy` (at least `frame-ancestors`, `object-src 'none'`, `base-uri 'self'`)
- [ ] `X-Content-Type-Options: nosniff`, `Referrer-Policy`, `Permissions-Policy`
- [ ] No `X-Powered-By` / server version leaks
- [ ] Session cookies `Secure; HttpOnly; SameSite`
- [ ] Rate limiting on login/signup/OTP
- [ ] **Privacy policy** page (required by every store; expected for finance/health apps)
- [ ] **Terms** page
- [ ] **Delete account** path (in-app + public URL; Google Play requires it)
- [ ] Contact email / support page

## 10. PWA install (being on the home screen)

- [ ] Manifest: `id`, `name`, `short_name`, `description`, `start_url`, `scope`, `display`, `lang`, `dir`, `categories`, `theme_color`, `background_color`
- [ ] Icons 192 + 512 `any`, 512 `maskable`, monochrome
- [ ] `screenshots` (narrow + wide) — enables the richer install dialog on Android/desktop
- [ ] `shortcuts` for top actions
- [ ] Install button / guide for iOS, Android, in-app browsers
- [ ] Works offline at least for the shell

## 11. ASO — app stores (Google Play, Cafe Bazaar, Myket, App Store)

- [ ] Store build (TWA via Bubblewrap/PWABuilder for a PWA) — `/.well-known/assetlinks.json` on the final domain with the signing key fingerprint
- [ ] Title with the main keyword (e.g. «مونده – مدیریت قسط و چک»)
- [ ] Short description: the promise in one line, keywords included
- [ ] Long description: features, FAQ-style answers, keywords naturally repeated
- [ ] 512 icon, phone screenshots with captions, feature graphic (Play: 1024×500), optional promo video
- [ ] Privacy policy URL, account-deletion URL, content rating, Play data-safety form
- [ ] Category chosen correctly; developer name + website consistent with the site
- [ ] Ask for ratings at a happy moment (after a success, not on first open)
- [ ] Reply to reviews; update the listing with each release
- [ ] Store pages linked from the landing (badges) and in `Organization.sameAs`

## 12. Measurement & off-page (keeps working after launch)

- [ ] Privacy-friendly analytics (self-hosted Umami / Plausible / Matomo if SaaS is blocked in your region); no PII in events
- [ ] Funnel events: landing view → signup → first key action → install → return (D1/D7)
- [ ] Error monitoring (Sentry / self-hosted GlitchTip) on web and API
- [ ] Search Console / Bing: watch queries, coverage errors, CWV report monthly
- [ ] Brand profiles with the same name, logo, one-line description: Instagram, Telegram channel, X, LinkedIn
- [ ] Launch posts and directories: Product Hunt; local equivalents (for Iran: Virgool, Zoomit/Digiato-style tech blogs, Telegram tech channels)
- [ ] Content: a few genuinely useful articles for real searches (e.g. «چطور قسط‌ها را مدیریت کنیم»), each linking to the app
- [ ] Backlinks from relevant sites; no paid link schemes
- [ ] Google "Preferred Sources" button — only for sites publishing news/articles (Top Stories); N/A for app-only sites
- [ ] Re-run this checklist after every major release
