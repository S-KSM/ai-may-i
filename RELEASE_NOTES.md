# Release Notes

## v1.0 — 2026-04-19 — Initial Launch

First public deploy of `ai-may-i.com`: a personal portfolio for AI-inspired side projects. Two projects ship with it.

### What went live

| URL | What | Repo |
|---|---|---|
| https://ai-may-i.com | Landing page | [`S-KSM/ai-may-i`](https://github.com/S-KSM/ai-may-i) |
| https://ai-may-i.com/classics/ | Illustrated Classics — 19 public-domain books, 3 visual modes | [`S-KSM/illustrated-classics`](https://github.com/S-KSM/illustrated-classics) (submodule) |
| https://card-stitcher.ai-may-i.com | Card Stitcher — PWA for stitching scanned greeting cards | [`S-KSM/card-stitcher`](https://github.com/S-KSM/card-stitcher) |

### Landing page

- Warm off-white palette (`#fdf8ef` / ink black / coral / citrus / mint).
- Fraunces italic display, Inter body, Caveat handwritten accents.
- Floating SVG shapes drifting in the background.
- Pulse-dot logo cycling through three accent colors.
- Two project cards (Illustrated Classics + Card Stitcher) with tilt-on-hover.
- Instagram link ([@ink.visions](https://www.instagram.com/ink.visions/)). TikTok reserved for later.

### Illustrated Classics

- 19 adult editions (Cinematic + Anime styles) + 19 kids editions.
- New third visual mode `astro-nomad` added and set as default — matches the landing aesthetic end-to-end.
- Existing `minimal` and `maximal` modes retained and toggleable.
- Book reading pages use a reading-friendly variant of `astro-nomad` (warm bg, soft shadows, no floating shapes) so illustrations remain the focus.
- Regeneration pipeline unchanged: `python3 build_site.py` in `book_transformer/` writes to the submodule working copy.

### Card Stitcher

- Phase 1 of PRD v2.0 shipped — PWA public beta.
- React 18 + TypeScript 5 strict + Vite 5 + Tailwind 3.
- Features: import → arrange → metadata → export; CSS 3D page-flip viewer; IndexedDB persistence; PDF / GIF / ZIP export; multi-provider AI autofill (Anthropic / OpenAI / Gemini) with BYOK; install prompt.
- Deployed as its own Cloudflare Pages project on a dedicated subdomain (`card-stitcher.ai-may-i.com`) to keep SPA routing and PWA service worker scope clean.

### Infrastructure decisions

- **Host:** Cloudflare Pages (free tier). Two Pages projects: `ai-may-i` (landing + classics) and `card-stitcher` (PWA).
- **DNS + SSL:** Cloudflare. `ai-may-i.com` zone is authoritative; Squarespace nameservers unchanged (domain remains registered there).
- **Landing → Classics integration:** git submodule. `ai-may-i` repo pins an exact `illustrated-classics` SHA; Cloudflare Pages clones with `GIT_SUBMODULE_STRATEGY=recursive`. Classics updates ship by bumping the submodule ref in `ai-may-i`.
- **Subdomain vs subpath:** Classics lives at `/classics/` (static, no SPA). Card Stitcher lives at `card-stitcher.*` because SPA routing + PWA service worker scope behave better under a dedicated subdomain.
- **`www` → apex:** 301 redirect via Cloudflare Redirect Rules (path + query preserved). `www` was removed from Pages custom domains to let the zone-level rule take precedence.
- **Uptime monitoring:** Betterstack, plain HTTP status check, 3-minute interval, SSL expiry alerts, on all three production URLs.
- **Decommissioned:** GitHub Pages deploy of `illustrated-classics` — Cloudflare Pages is now authoritative.

### Update workflows

**Landing:** edit `~/Code/ai-may-i/index.html` → push → auto-deploys.

**Classics content:**
```bash
cd ~/Code/Gutenberg/book_transformer
python3 build_site.py
cd site && git add -A && git commit -m "..." && git push
cd ~/Code/ai-may-i
git submodule update --remote classics
git commit -am "bump classics" && git push
```

**Card Stitcher:** edit `~/Code/card-stitcher/` → push → auto-deploys (`npm run build` on CF).

### Known gotchas captured for next time

- Cloudflare Pages custom domain bindings bypass Redirect Rules for attached hostnames. If a redirect rule needs to fire against a hostname, remove it from Pages custom domains first.
- Redirect Rule match values are whitespace-sensitive — a trailing space in the hostname value will silently skip the rule.
- The classics `site/` working copy doubles as the `illustrated-classics` git repo. Don't add a second `.git` inside it.
