# Portfolio Redesign — Design Spec
**Date:** 2026-06-27  
**Owner:** Rohan RJ  
**Status:** Approved for implementation

---

## Vision

A single-file (`index.html`) portfolio that feels like an Apple product launch page built by a founder — not a developer template. Pure black background, white space as design, 10 interactions, photos animated via Higgsfield, and a "manifesto" scroll section that stops US startup hiring managers cold.

**Audience:** US startup GTM leads, founders, hiring managers  
**Goal:** One clear action — start a conversation with Rohan

---

## Style: Apple Clean Dark

- **Background:** `#000000` (pure black) with `#0A0A0F` for card surfaces
- **Accent:** `#2563EB` (electric blue) + `#06B6D4` (cyan) for highlights only
- **Text:** `#FFFFFF` primary, `rgba(255,255,255,0.45)` secondary, `rgba(255,255,255,0.22)` muted
- **Buttons:** White pill buttons (`background:#fff; color:#000; border-radius:100px`) for primary CTAs
- **Typography:** `-apple-system, 'Segoe UI'` body; `Georgia, serif` for display headings
- **Borders:** `rgba(255,255,255,0.06)` — barely-there separators
- **Cards:** `#0a0a0a` background, `border-radius:20px`, no heavy borders

---

## 10 Interactions

1. **Magnetic cursor** — custom cursor div tracks mouse; near buttons/links it is attracted with CSS transform. Deforms to an oval on hover.
2. **Word-by-word scroll reveal (manifesto)** — each `<span>` word starts `color: rgba(255,255,255,0.12)` and transitions to full white as it enters the scroll viewport via IntersectionObserver with threshold steps.
3. **Parallax photo layers** — hero photo and about photo move at 0.4× scroll speed via `requestAnimationFrame`.
4. **Higgsfield video loop** — `<video autoplay muted loop playsinline>` placed as the hero's right-side background; falls back to static photo if no video.
5. **Counter animations** — stats (6, 3, 1) count up from 0 using `requestAnimationFrame` easing when they scroll into view.
6. **Typewriter contact CTA** — email address types itself letter by letter when the contact section enters view.
7. **Smooth scroll inertia** — Lenis-style custom smooth scroll using `requestAnimationFrame` with lerp (no library; ~30 lines of JS).
8. **3D card tilt** — project cards respond to `mousemove` with `perspective(1000px) rotateX() rotateY()` capped at ±8°; resets on `mouseleave`.
9. **Scroll progress bar** — 2px fixed bar at top of viewport, width = `scrollY / (docHeight - viewportHeight) * 100%`, color `linear-gradient(90deg, #2563EB, #06B6D4)`.
10. **Stagger scroll reveals** — sections and cards use IntersectionObserver; children animate in with `opacity: 0 → 1` + `translateY(24px → 0)` with 80ms stagger between siblings.

---

## Page Sections (in order)

### 1 · Navigation (fixed)
- Logo: "Rohan RJ" left-aligned, 14px, weight 700
- Links: Work · Stack · About — `rgba(255,255,255,0.38)`, 11px
- CTA pill: "Let's Talk →" — white pill, black text, right-aligned
- `backdrop-filter: blur(20px)` on scroll; border-bottom appears after 80px scroll

### 2 · Hero (100vh)
- **Left column (55%):** availability chip (green dot + "Open to remote GTM roles"), name `ROHAN / RJ.` at 72–80px Georgia, role line, tagline with cyan `<em>`, two pill CTAs, stat counters (6 / 3 / 1)
- **Right column (45%):** `<video>` or `<img>` of Rohan — Higgsfield-animated photo, parallax on scroll, blue radial glow behind subject
- Animated grid background (radial dot + line grid, CSS `animation: breathe`)
- Scroll progress bar at very top

### 3 · Manifesto (full-width, centred)
- Black section, centred text
- Large statement: *"Most businesses don't lose revenue from bad marketing. They lose it from broken execution."*
- Word-by-word colour scroll reveal (dim → white)
- Sub-line: `I BUILD THE INFRASTRUCTURE THAT FIXES THIS.` — small caps, muted

### 4 · What I Build (3 columns)
- GTM & Revenue Systems / Business Operations Software / Workflow Automation
- Cards stagger in from bottom; hover triggers subtle inner glow
- Arrow list items inside each card

### 5 · Production Systems (projects)
- Two-column card layout (left: name + status badge + URL meta; right: problem quote + what was built + tech tags)
- 3D tilt on `mousemove`
- Architecture diagram for Aarna (CSS nodes)
- Three projects: Aarna, MENZO, STEMLYN

### 6 · Stats Strip (full-width dark band)
- Three giant counters: **6** Production Systems · **3** Live URLs · **1** Backend
- Numbers animate up from 0 on scroll entry

### 7 · About (split — photo left, text right)
- **Left:** real photo of Rohan, full height, `object-fit: cover`, parallax on scroll
- **Right:** two paragraphs, text reveals paragraph by paragraph on scroll
- MBA + location sidebar card

### 8 · Tech Stack
- Grouped by category (GTM Tools / AI & Automation / Dev / Business)
- Tags spring in on scroll entry (stagger)
- Hover: lift + blue border

### 9 · Contact (centred, large)
- Big serif headline: "Let's Talk."
- One-line CTA copy
- Email typed out via typewriter effect
- Two link buttons: email + LinkedIn

### 10 · Footer
- Minimal: copyright + tagline + LinkedIn

---

## Technical Constraints

- **Single file:** all CSS and JS inline in `index.html`. No imports, no CDN.
- **No frameworks:** vanilla JS only. No React, no Vue, no Lenis library.
- **No external fonts:** system font stack only.
- **No external images:** Higgsfield video/photo embedded as `src` path or base64 (user will supply files).
- **Performance:** smooth scroll and cursor use `requestAnimationFrame`, not `setInterval`. IntersectionObserver for all reveals.
- **Mobile:** all interactions degrade gracefully — cursor disabled on touch, parallax disabled on mobile, smooth scroll disabled on `prefers-reduced-motion`.

---

## Photo / Video Assets Needed

- [ ] Rohan's photo(s) — for hero right column and about section
- [ ] Higgsfield-animated video loop (generated from photo above)

Implementation can proceed with placeholder silhouette; assets dropped in when ready.

---

## What's NOT in scope

- CMS or dynamic content
- Blog section
- Dark/light mode toggle
- Contact form (email link only)
- Analytics (user can add later)
