# Portfolio Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace `index.html` with an Apple Clean Dark single-file portfolio with 10 interactions, Higgsfield video hero, photos, and a manifesto scroll section.

**Architecture:** Single `index.html` with all CSS in a `<style>` block and all JS in a `<script>` block at the bottom. No libraries, no imports, no CDN. Each task builds one vertical section plus its interaction; at the end of every task the file is verified visually in the browser.

**Tech Stack:** Vanilla JS (ES6), CSS custom properties, IntersectionObserver, requestAnimationFrame, CSS animations

**Spec:** `docs/superpowers/specs/2026-06-27-portfolio-redesign.md`

---

## File Map

| File | Action | Responsibility |
|------|--------|----------------|
| `index.html` | Overwrite | Entire portfolio — HTML, inline CSS, inline JS |
| `assets/rohan-photo.jpg` | Add when ready | Portrait for hero + about |
| `assets/rohan-loop.mp4` | Add when ready | Higgsfield animated video loop |

---

## Task 1: Foundation Shell — CSS Tokens, Custom Cursor, Scroll Progress, Smooth Scroll

- [ ] **Step 1: Write the complete foundation shell**

Replace `index.html` entirely with the following. Every subsequent task adds to this file.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Rohan RJ — GTM Engineer & AI Systems Builder</title>
<meta name="description" content="Rohan RJ builds AI-powered GTM systems and revenue operations infrastructure. Based in Mangaluru, India.">
<style>
/* RESET */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

/* TOKENS */
:root {
  --bg:         #000000;
  --bg-surface: #0a0a0a;
  --bg-raised:  #0d0d0d;
  --border:     rgba(255,255,255,0.06);
  --border-mid: rgba(255,255,255,0.12);
  --accent:     #2563EB;
  --accent-dim: rgba(37,99,235,0.15);
  --cyan:       #06B6D4;
  --green:      #22C55E;
  --amber:      #F59E0B;
  --text:       #FFFFFF;
  --text-2:     rgba(255,255,255,0.55);
  --text-3:     rgba(255,255,255,0.28);
  --text-4:     rgba(255,255,255,0.12);
  --radius-sm:  8px;
  --radius-md:  14px;
  --radius-lg:  20px;
  --radius-xl:  28px;
  --ease:       cubic-bezier(0.4, 0, 0.2, 1);
  --font-body:  -apple-system, 'Segoe UI', system-ui, sans-serif;
  --font-serif: Georgia, 'Times New Roman', serif;
}

html { scroll-behavior: auto; }

body {
  background: var(--bg);
  color: var(--text);
  font-family: var(--font-body);
  font-size: 16px;
  line-height: 1.6;
  overflow-x: hidden;
  -webkit-font-smoothing: antialiased;
}

/* Hide real cursor on desktop; show on touch */
@media (hover: hover) { body { cursor: none; } }

/* SCROLL PROGRESS BAR */
#progress-bar {
  position: fixed;
  top: 0; left: 0;
  height: 2px;
  width: 0%;
  background: linear-gradient(90deg, var(--accent), var(--cyan));
  z-index: 9999;
  pointer-events: none;
}

/* CUSTOM CURSOR */
#cursor {
  position: fixed;
  width: 12px; height: 12px;
  background: #fff;
  border-radius: 50%;
  pointer-events: none;
  z-index: 9998;
  transform: translate(-50%, -50%);
  transition: width 0.22s var(--ease), height 0.22s var(--ease),
              background 0.22s var(--ease);
  mix-blend-mode: difference;
}
#cursor.hovering {
  width: 44px; height: 44px;
  background: rgba(255,255,255,0.12);
  border: 1px solid rgba(255,255,255,0.25);
  mix-blend-mode: normal;
}
#cursor.clicking { width: 8px; height: 8px; }

/* SCROLL REVEAL */
.reveal {
  opacity: 0;
  transform: translateY(28px);
  transition: opacity 0.7s var(--ease), transform 0.7s var(--ease);
}
.reveal.in { opacity: 1; transform: none; }
.reveal-delay-1 { transition-delay: 0.1s; }
.reveal-delay-2 { transition-delay: 0.18s; }
.reveal-delay-3 { transition-delay: 0.26s; }
.reveal-delay-4 { transition-delay: 0.34s; }

/* UTIL */
.section-wrap { padding: clamp(72px,10vw,120px) clamp(20px,6vw,100px); }

@keyframes blink { 0%,100%{opacity:1} 50%{opacity:.25} }
@keyframes fade-up { from { opacity:0; transform:translateY(20px); } to { opacity:1; transform:none; } }
@keyframes grid-breathe { 0%,100%{opacity:.35} 50%{opacity:.85} }
</style>
</head>
<body>

<div id="progress-bar" aria-hidden="true"></div>
<div id="cursor" aria-hidden="true"></div>

<!-- NAV, SECTIONS, FOOTER injected in Tasks 2-10 -->

<script>
/* SMOOTH SCROLL (lerp — no library) */
(function() {
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return;
  if ('ontouchstart' in window) return;

  let current = 0, target = 0;
  const EASE = 0.1;

  const scroller = document.createElement('div');
  scroller.style.cssText = 'position:fixed;top:0;left:0;width:100%;overflow-y:scroll;height:100vh;';
  while (document.body.firstChild) scroller.appendChild(document.body.firstChild);
  document.body.appendChild(scroller);
  document.body.style.overflow = 'hidden';

  function lerp(a, b, t) { return a + (b - a) * t; }

  function tick() {
    target = scroller.scrollTop;
    current = lerp(current, target, EASE);
    const max = scroller.scrollHeight - scroller.clientHeight;
    const pct = max > 0 ? (current / max) * 100 : 0;
    document.getElementById('progress-bar').style.width = pct + '%';
    requestAnimationFrame(tick);
  }
  requestAnimationFrame(tick);

  window.__scrollY = () => current;
  window.__scrollEl = scroller;
})();

if (!window.__scrollY) {
  window.__scrollY = () => window.scrollY;
  window.__scrollEl = window;
  window.addEventListener('scroll', () => {
    const max = document.body.scrollHeight - window.innerHeight;
    document.getElementById('progress-bar').style.width = (max > 0 ? window.scrollY / max * 100 : 0) + '%';
  });
}

/* CUSTOM CURSOR */
(function() {
  if ('ontouchstart' in window) return;
  const cur = document.getElementById('cursor');
  let mx = -100, my = -100, cx = -100, cy = -100;

  document.addEventListener('mousemove', e => { mx = e.clientX; my = e.clientY; });
  document.addEventListener('mousedown', () => cur.classList.add('clicking'));
  document.addEventListener('mouseup',   () => cur.classList.remove('clicking'));
  document.addEventListener('mouseover', e => { if (e.target.closest('a,button')) cur.classList.add('hovering'); });
  document.addEventListener('mouseout',  e => { if (e.target.closest('a,button')) cur.classList.remove('hovering'); });

  function tick() {
    cx += (mx - cx) * 0.15;
    cy += (my - cy) * 0.15;
    cur.style.left = cx + 'px';
    cur.style.top  = cy + 'px';
    requestAnimationFrame(tick);
  }
  requestAnimationFrame(tick);
})();

/* SCROLL REVEAL */
const revealObs = new IntersectionObserver((entries) => {
  entries.forEach(e => { if (e.isIntersecting) { e.target.classList.add('in'); revealObs.unobserve(e.target); } });
}, { threshold: 0.12, rootMargin: '0px 0px -40px 0px' });
document.querySelectorAll('.reveal').forEach(el => revealObs.observe(el));
</script>
</body>
</html>
```

- [ ] **Step 2: Open in browser — verify**

Black page, no errors. Move mouse — white dot follows with inertia. Blue progress bar at top.

- [ ] **Step 3: Commit**

```bash
cd "/Users/rohanraj/Rohan RJ/my portfolio"
git init
git add index.html
git commit -m "feat: foundation — tokens, cursor, scroll progress, smooth scroll lerp"
```

---

## Task 2: Navigation

- [ ] **Step 1: Add nav CSS inside `<style>` before `</style>`**

```css
/* NAV */
nav {
  position: fixed; top: 0; left: 0; right: 0; z-index: 200;
  height: 58px;
  display: flex; align-items: center; justify-content: space-between;
  padding: 0 clamp(20px,5vw,80px);
  transition: background 0.3s var(--ease), border-color 0.3s var(--ease);
  border-bottom: 1px solid transparent;
}
nav.scrolled {
  background: rgba(0,0,0,0.85);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-color: var(--border);
}
.nav-logo { font-size: 15px; font-weight: 700; color: var(--text); text-decoration: none; letter-spacing: -0.3px; }
.nav-logo span { color: var(--accent); }
.nav-links { display: flex; gap: 28px; list-style: none; }
.nav-links a { font-size: 12px; color: var(--text-3); text-decoration: none; letter-spacing:.04em; transition: color .2s; }
.nav-links a:hover { color: var(--text); }
.nav-cta {
  padding: 8px 18px;
  background: var(--text); color: #000;
  border-radius: 100px;
  font-size: 12px; font-weight: 700;
  text-decoration: none;
  transition: opacity .2s, transform .2s;
}
.nav-cta:hover { opacity: .88; transform: scale(1.02); }
@media (max-width: 600px) { .nav-links { display: none; } }
```

- [ ] **Step 2: Add nav HTML after `<div id="cursor">`**

```html
<nav id="nav" aria-label="Main navigation">
  <a href="#hero" class="nav-logo">Rohan <span>RJ</span></a>
  <ul class="nav-links">
    <li><a href="#projects">Work</a></li>
    <li><a href="#stack">Stack</a></li>
    <li><a href="#about">About</a></li>
  </ul>
  <a href="#contact" class="nav-cta">Let's Talk →</a>
</nav>
```

- [ ] **Step 3: Add nav JS inside `<script>` after smooth-scroll block**

```js
/* NAV SCROLL */
(function() {
  const nav = document.getElementById('nav');
  const el = window.__scrollEl;
  el.addEventListener('scroll', () => nav.classList.toggle('scrolled', window.__scrollY() > 60));
})();
```

- [ ] **Step 4: Verify — nav invisible at top, gets blur on scroll, pill CTA visible**

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: nav — blur on scroll, pill CTA"
```

---

## Task 3: Hero Section

- [ ] **Step 1: Add hero CSS inside `<style>`**

```css
/* HERO */
#hero {
  min-height: 100vh;
  display: grid;
  grid-template-columns: 55fr 45fr;
  position: relative;
  overflow: hidden;
  padding-top: 58px;
}
.hero-grid {
  position: absolute; inset: 0;
  background-image:
    radial-gradient(circle, rgba(37,99,235,0.25) 1px, transparent 1px),
    linear-gradient(rgba(37,99,235,0.045) 1px, transparent 1px),
    linear-gradient(90deg, rgba(37,99,235,0.045) 1px, transparent 1px);
  background-size: 52px 52px;
  animation: grid-breathe 7s ease-in-out infinite;
  pointer-events: none;
}
.hero-left {
  padding: clamp(48px,8vh,100px) clamp(20px,6vw,100px);
  display: flex; flex-direction: column; justify-content: center;
  position: relative; z-index: 2;
}
.hero-chip {
  display: inline-flex; align-items: center; gap: 7px;
  background: rgba(255,255,255,0.05);
  border: 1px solid rgba(255,255,255,0.1);
  border-radius: 100px;
  padding: 5px 14px 5px 7px;
  font-size: 11px; color: var(--text-2);
  width: fit-content; margin-bottom: 32px;
  animation: fade-up 0.8s 0.2s both;
}
.hero-chip-dot {
  width: 7px; height: 7px; border-radius: 50%;
  background: var(--green); box-shadow: 0 0 7px var(--green);
  animation: blink 2.5s ease infinite;
}
.hero-name {
  font-family: var(--font-serif);
  font-size: clamp(56px,9vw,96px);
  font-weight: normal; line-height: 0.88; letter-spacing: -3px;
  color: var(--text);
  animation: fade-up 0.8s 0.3s both;
}
.hero-name .blue { color: var(--accent); }
.hero-role {
  margin-top: 22px; font-size: 12px; color: var(--text-3);
  letter-spacing: .12em; text-transform: uppercase;
  animation: fade-up 0.8s 0.4s both;
}
.hero-tagline {
  margin-top: 16px; font-size: clamp(14px,1.6vw,16px);
  color: var(--text-2); line-height: 1.65; max-width: 440px;
  animation: fade-up 0.8s 0.5s both;
}
.hero-tagline em { font-style: normal; color: var(--cyan); }
.hero-actions {
  display: flex; gap: 10px; margin-top: 32px; flex-wrap: wrap;
  animation: fade-up 0.8s 0.6s both;
}
.btn-pill-white {
  padding: 12px 24px; background: var(--text); color: #000;
  border-radius: 100px; font-size: 13px; font-weight: 700;
  text-decoration: none; display: inline-flex; align-items: center; gap: 7px;
  transition: opacity .2s, transform .18s;
}
.btn-pill-white:hover { opacity: .88; transform: translateY(-1px); }
.btn-pill-ghost {
  padding: 12px 24px; border: 1px solid var(--border-mid);
  color: var(--text-2); border-radius: 100px; font-size: 13px;
  text-decoration: none; display: inline-flex; align-items: center; gap: 7px;
  transition: border-color .2s, color .2s, transform .18s;
}
.btn-pill-ghost:hover { border-color: rgba(255,255,255,.35); color: var(--text); transform: translateY(-1px); }
.hero-stats {
  display: flex; gap: clamp(20px,4vw,48px);
  margin-top: 52px; padding-top: 28px; border-top: 1px solid var(--border);
  animation: fade-up 0.8s 0.7s both;
}
.stat-num {
  font-family: var(--font-serif); font-size: 36px;
  font-weight: normal; color: var(--text); letter-spacing: -1px; line-height: 1;
}
.stat-label { font-size: 11px; color: var(--text-3); margin-top: 4px; letter-spacing:.05em; }
.hero-right {
  position: relative; overflow: hidden;
  background: linear-gradient(135deg, #04040c 0%, #09091f 100%);
}
.hero-photo-glow {
  position: absolute; bottom: 0; left: 50%; transform: translateX(-50%);
  width: 120%; height: 70%;
  background: radial-gradient(ellipse at 50% 100%, rgba(37,99,235,0.2) 0%, transparent 65%);
  pointer-events: none;
}
.hero-media {
  position: absolute; inset: 0;
  display: flex; align-items: flex-end; justify-content: center;
}
.hero-media video, .hero-media img {
  width: 100%; height: 100%;
  object-fit: cover; object-position: center top;
}
.hero-media-placeholder {
  width: 220px; height: 360px;
  border-radius: 110px 110px 0 0;
  background: linear-gradient(180deg, rgba(37,99,235,0.15) 0%, transparent 100%);
  border: 1px solid rgba(37,99,235,0.18); border-bottom: none;
  position: relative;
}
.hero-media-placeholder::before {
  content: ''; position: absolute;
  top: 36px; left: 50%; transform: translateX(-50%);
  width: 80px; height: 80px; border-radius: 50%;
  background: rgba(37,99,235,0.18); border: 1px solid rgba(37,99,235,0.25);
}
.hf-badge {
  position: absolute; bottom: 24px; right: 20px;
  background: rgba(0,0,0,0.7); backdrop-filter: blur(12px);
  border: 1px solid var(--border-mid); border-radius: 10px;
  padding: 8px 14px; font-size: 10px; color: var(--text-3);
  display: flex; align-items: center; gap: 7px;
}
.hf-badge span { color: var(--cyan); font-weight: 700; }
.parallax-el { will-change: transform; }
@media (max-width: 768px) { #hero { grid-template-columns: 1fr; } .hero-right { display: none; } }
```

- [ ] **Step 2: Add hero HTML after `</nav>`**

```html
<section id="hero">
  <div class="hero-grid" aria-hidden="true"></div>
  <div class="hero-left">
    <div class="hero-chip">
      <div class="hero-chip-dot"></div>
      Open to remote GTM roles
    </div>
    <h1 class="hero-name">ROHAN<br><span class="blue">RJ.</span></h1>
    <p class="hero-role">GTM Engineer &nbsp;·&nbsp; AI Systems Builder &nbsp;·&nbsp; Revenue Operations</p>
    <p class="hero-tagline">I build <em>AI-powered systems</em> that eliminate the execution gaps businesses lose revenue through.</p>
    <div class="hero-actions">
      <a href="#projects" class="btn-pill-white">
        <svg width="13" height="13" fill="none" stroke="currentColor" stroke-width="2.2" viewBox="0 0 24 24" aria-hidden="true"><path d="M12 5v14M5 12l7 7 7-7"/></svg>
        View My Work
      </a>
      <a href="#" class="btn-pill-ghost">
        <svg width="13" height="13" fill="none" stroke="currentColor" stroke-width="2.2" viewBox="0 0 24 24" aria-hidden="true"><path d="M21 15v4a2 2 0 01-2 2H5a2 2 0 01-2-2v-4M7 10l5 5 5-5M12 15V3"/></svg>
        Download CV
      </a>
    </div>
    <div class="hero-stats">
      <div class="hero-stat"><div class="stat-num" data-counter="6">0</div><div class="stat-label">Production Systems</div></div>
      <div class="hero-stat"><div class="stat-num" data-counter="3">0</div><div class="stat-label">Live URLs</div></div>
      <div class="hero-stat"><div class="stat-num" data-counter="1">0</div><div class="stat-label">Backend Controls All</div></div>
    </div>
  </div>
  <div class="hero-right parallax-el" data-parallax="0.35">
    <div class="hero-photo-glow" aria-hidden="true"></div>
    <div class="hero-media" id="hero-media">
      <div class="hero-media-placeholder" aria-label="Photo placeholder"></div>
    </div>
    <div class="hf-badge" aria-hidden="true">▶ <span>Higgsfield</span> · cinematic loop</div>
  </div>
</section>
```

- [ ] **Step 3: Add hero JS inside `<script>`**

```js
/* HERO MEDIA — try video, fall back to photo, fall back to placeholder */
(function() {
  const container = document.getElementById('hero-media');
  if (!container) return;

  function loadPhoto() {
    const img = document.createElement('img');
    img.alt = 'Rohan RJ';
    img.src = 'assets/rohan-photo.jpg';
    img.onload = function() {
      container.textContent = '';
      container.appendChild(img);
    };
    // on error, placeholder silhouette already showing — do nothing
  }

  const vid = document.createElement('video');
  vid.src = 'assets/rohan-loop.mp4';
  vid.autoplay = true; vid.muted = true; vid.loop = true; vid.playsInline = true;
  vid.setAttribute('aria-hidden', 'true');
  vid.oncanplay = function() {
    if (!vid.dataset.loaded) {
      vid.dataset.loaded = '1';
      container.textContent = '';
      container.appendChild(vid);
    }
  };
  vid.onerror = loadPhoto;
  vid.load();
  // If video takes >3s, try photo
  setTimeout(() => { if (!vid.dataset.loaded) loadPhoto(); }, 3000);
})();

/* COUNTER ANIMATION */
(function() {
  const obs = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (!entry.isIntersecting) return;
      const el = entry.target;
      const target = parseInt(el.dataset.counter, 10);
      const start = performance.now();
      const DURATION = 1200;
      function step(now) {
        const p = Math.min((now - start) / DURATION, 1);
        const eased = 1 - Math.pow(1 - p, 3);
        el.textContent = Math.round(eased * target);
        if (p < 1) requestAnimationFrame(step);
      }
      requestAnimationFrame(step);
      obs.unobserve(el);
    });
  }, { threshold: 0.8 });
  document.querySelectorAll('[data-counter]').forEach(el => obs.observe(el));
})();

/* PARALLAX */
(function() {
  if ('ontouchstart' in window) return;
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return;
  const els = document.querySelectorAll('[data-parallax]');
  function tick() {
    const sy = window.__scrollY();
    els.forEach(el => {
      el.style.transform = 'translateY(' + (sy * parseFloat(el.dataset.parallax)) + 'px)';
    });
    requestAnimationFrame(tick);
  }
  requestAnimationFrame(tick);
})();
```

- [ ] **Step 4: Verify — hero renders, grid breathes, chip has green dot, counters animate on scroll**

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: hero — parallax, counter anim, media loader, breathing grid"
```

---

## Task 4: Manifesto Section

- [ ] **Step 1: Add manifesto CSS inside `<style>`**

```css
/* MANIFESTO */
#manifesto {
  padding: clamp(80px,12vw,140px) clamp(20px,8vw,120px);
  text-align: center;
  border-top: 1px solid var(--border);
}
.manifesto-label {
  font-size: 10px; letter-spacing:.22em; text-transform: uppercase;
  color: var(--text-4); margin-bottom: 36px;
}
.manifesto-body {
  font-family: var(--font-serif);
  font-size: clamp(24px,4vw,44px);
  line-height: 1.3; letter-spacing: -0.5px;
  max-width: 800px; margin: 0 auto;
}
.manifesto-body .w {
  display: inline-block;
  color: var(--text-4);
  transition: color 0.5s var(--ease);
  margin-right: 0.22em;
}
.manifesto-body .w.lit { color: var(--text); }
.manifesto-body .w.accent-word.lit { color: var(--accent); }
.manifesto-sub {
  margin-top: 36px; font-size: 11px;
  letter-spacing:.2em; text-transform: uppercase;
  color: var(--text-4);
  opacity: 0; transition: opacity 0.6s var(--ease) 0.3s;
}
.manifesto-sub.in { opacity: 1; }
```

- [ ] **Step 2: Add manifesto HTML after hero `</section>`**

```html
<section id="manifesto">
  <p class="manifesto-label">The reason I exist</p>
  <p class="manifesto-body" id="manifesto-text">Most businesses don't lose revenue from bad marketing. They lose it from broken execution.</p>
  <p class="manifesto-sub" id="manifesto-sub">I build the infrastructure that fixes this.</p>
</section>
```

- [ ] **Step 3: Add manifesto JS inside `<script>`**

```js
/* MANIFESTO WORD REVEAL */
(function() {
  const container = document.getElementById('manifesto-text');
  const sub = document.getElementById('manifesto-sub');
  if (!container) return;

  const accentWords = new Set(['broken', 'execution.']);
  const words = container.textContent.trim().split(/\s+/);

  // Build DOM safely without innerHTML string interpolation
  const frag = document.createDocumentFragment();
  const spans = [];
  words.forEach((word, i) => {
    const span = document.createElement('span');
    span.className = 'w' + (accentWords.has(word.toLowerCase()) ? ' accent-word' : '');
    span.textContent = word;
    frag.appendChild(span);
    if (i < words.length - 1) frag.appendChild(document.createTextNode(' '));
    spans.push(span);
  });
  container.textContent = '';
  container.appendChild(frag);

  const wordObs = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (!entry.isIntersecting) return;
      const idx = spans.indexOf(entry.target);
      setTimeout(() => entry.target.classList.add('lit'), idx * 60);
      wordObs.unobserve(entry.target);
    });
  }, { threshold: 0.8, rootMargin: '0px 0px -60px 0px' });
  spans.forEach(s => wordObs.observe(s));

  const subObs = new IntersectionObserver((entries) => {
    entries.forEach(e => { if (e.isIntersecting) { sub.classList.add('in'); subObs.unobserve(e.target); } });
  }, { threshold: 0.5 });
  subObs.observe(spans[spans.length - 1]);
})();
```

- [ ] **Step 4: Verify — scroll to manifesto, words progressively light up, "broken execution." turns blue**

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: manifesto — word-by-word scroll reveal using safe DOM methods"
```

---

## Task 5: What I Build Section

- [ ] **Step 1: Add CSS inside `<style>`**

```css
/* WHAT I BUILD */
#build { border-top: 1px solid var(--border); }
.build-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1px; background: var(--border);
  border: 1px solid var(--border); border-radius: var(--radius-xl); overflow: hidden;
}
.build-col {
  background: var(--bg-surface);
  padding: 36px 32px 40px;
  transition: background .25s var(--ease);
}
.build-col:hover { background: var(--bg-raised); }
.build-icon {
  width: 44px; height: 44px; border-radius: var(--radius-sm);
  background: var(--accent-dim); border: 1px solid rgba(37,99,235,.25);
  display: flex; align-items: center; justify-content: center;
  font-size: 18px; margin-bottom: 22px; user-select: none;
}
.build-col h3 { font-family: var(--font-serif); font-size: 1.1rem; font-weight: normal; color: var(--text); margin-bottom: 12px; }
.build-col > p { font-size: 13px; color: var(--text-2); line-height: 1.7; }
.build-items { margin-top: 18px; list-style: none; display: flex; flex-direction: column; gap: 4px; }
.build-items li {
  font-size: 12px; color: var(--text-3);
  padding: 5px 0; border-bottom: 1px solid rgba(255,255,255,.03);
  display: flex; align-items: center; gap: 8px;
}
.build-items li::before { content: '→'; color: var(--accent); font-size: 10px; flex-shrink: 0; }
```

- [ ] **Step 2: Add HTML after manifesto `</section>`**

```html
<section id="build" class="section-wrap">
  <div class="reveal">
    <p style="font-size:10px;letter-spacing:.22em;text-transform:uppercase;color:var(--accent);margin-bottom:10px;font-weight:700;">Capabilities</p>
    <h2 style="font-family:var(--font-serif);font-size:clamp(28px,4vw,44px);font-weight:normal;letter-spacing:-.5px;color:var(--text);margin-bottom:40px;">What I Build</h2>
  </div>
  <div class="build-grid">
    <div class="build-col reveal reveal-delay-1">
      <div class="build-icon" aria-hidden="true">◈</div>
      <h3>GTM &amp; Revenue Systems</h3>
      <p>End-to-end systems that connect pipeline to closed revenue — with no gaps in between.</p>
      <ul class="build-items">
        <li>AI-powered lead pipelines</li><li>Outreach automation &amp; sequencing</li>
        <li>CRM integrations &amp; data flows</li><li>WhatsApp commerce agents</li>
      </ul>
    </div>
    <div class="build-col reveal reveal-delay-2">
      <div class="build-icon" aria-hidden="true">⬡</div>
      <h3>Business Operations Software</h3>
      <p>Full-stack BMS platforms that replace paper, memory, and disconnected tools.</p>
      <ul class="build-items">
        <li>POS &amp; billing engines</li><li>Inventory with batch tracking</li>
        <li>CRM &amp; delivery management</li><li>GST reporting &amp; CA-ready financials</li>
      </ul>
    </div>
    <div class="build-col reveal reveal-delay-3">
      <div class="build-icon" aria-hidden="true">⟳</div>
      <h3>Workflow Automation</h3>
      <p>Manual processes replaced with automated pipelines using AI, APIs, and custom logic.</p>
      <ul class="build-items">
        <li>Multi-step API orchestration</li><li>AI decision architectures</li>
        <li>Real-time event pipelines</li><li>Cross-platform data sync</li>
      </ul>
    </div>
  </div>
</section>
```

- [ ] **Step 3: Verify — 3 cards stagger in, hover darkens slightly**

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: what I build — 3-col stagger reveal"
```

---

## Task 6: Projects Section — 3D Tilt + Architecture Diagram

- [ ] **Step 1: Add CSS inside `<style>`**

```css
/* PROJECTS */
#projects { border-top: 1px solid var(--border); }
.project-card {
  background: var(--bg-surface); border: 1px solid var(--border);
  border-radius: var(--radius-xl); overflow: hidden; margin-bottom: 12px;
  transition: border-color .3s var(--ease), box-shadow .3s var(--ease);
  transform-style: preserve-3d; will-change: transform;
}
.project-card:hover { border-color: var(--border-mid); box-shadow: 0 20px 60px rgba(0,0,0,.5); }
.pc-inner { display: grid; grid-template-columns: 280px 1fr; }
.pc-left {
  padding: 32px 26px; border-right: 1px solid var(--border);
  background: rgba(255,255,255,.01);
  display: flex; flex-direction: column; gap: 14px;
}
.pc-num { font-size: 10px; color: var(--text-4); letter-spacing:.18em; }
.pc-name { font-family: var(--font-serif); font-size: 1.3rem; font-weight: normal; color: var(--text); line-height: 1.2; }
.pc-badge {
  display: inline-flex; align-items: center; gap: 6px;
  padding: 4px 10px; border-radius: 6px;
  font-size: 9px; font-weight: 700; letter-spacing:.07em; text-transform: uppercase;
  width: fit-content;
}
.badge-live { background: rgba(34,197,94,.1); color: var(--green); border: 1px solid rgba(34,197,94,.2); }
.badge-live::before { content:''; width:5px;height:5px;border-radius:50%;background:var(--green);box-shadow:0 0 5px var(--green);animation:blink 2s infinite; }
.badge-dev  { background: rgba(245,158,11,.1); color: var(--amber); border: 1px solid rgba(245,158,11,.2); }
.badge-dev::before { content:''; width:5px;height:5px;border-radius:50%;background:var(--amber); }
.pc-url { font-size: 11px; color: var(--cyan); text-decoration: none; opacity:.8; transition: opacity .2s; word-break: break-all; }
.pc-url:hover { opacity: 1; }
.pc-meta { margin-top: auto; font-size: 11px; color: var(--text-4); line-height: 1.6; }
.pc-right { padding: 32px 36px; display: flex; flex-direction: column; gap: 18px; }
.pc-problem-label { font-size: 10px; font-weight: 700; letter-spacing:.18em; text-transform: uppercase; color: var(--text-4); }
.pc-quote { font-family: var(--font-serif); font-size: 1rem; color: var(--text-2); line-height: 1.55; font-style: italic; border-left: 2px solid rgba(37,99,235,.4); padding-left: 16px; }
.pc-built-label { font-size: 10px; font-weight: 700; letter-spacing:.14em; text-transform: uppercase; color: var(--accent); }
.pc-built p { font-size: 13px; color: var(--text-2); line-height: 1.7; }
/* Architecture diagram */
.arch { padding: 18px; background: rgba(0,0,0,.25); border: 1px solid var(--border); border-radius: var(--radius-md); }
.arch-root { display: flex; justify-content: center; }
.arch-root-node { padding: 7px 16px; background: rgba(37,99,235,.18); border: 1px solid var(--accent); border-radius: 6px; font-size: 10px; font-weight: 700; letter-spacing:.08em; text-transform: uppercase; color: var(--accent); }
.arch-lines { display: flex; justify-content: space-around; height: 20px; margin: 0 8%; position: relative; }
.arch-lines::before { content:''; position: absolute; top: 0; left: 0; right: 0; height: 1px; background: linear-gradient(90deg, transparent, rgba(37,99,235,.4), transparent); }
.arch-line { border-left: 1px solid rgba(37,99,235,.3); height: 100%; }
.arch-children { display: grid; grid-template-columns: repeat(4,1fr); gap: 6px; }
.arch-child { padding: 7px 6px; background: rgba(6,182,212,.07); border: 1px solid rgba(6,182,212,.2); border-radius: 6px; font-size: 10px; color: var(--cyan); text-align: center; line-height: 1.3; font-weight: 600; }
/* Subsystems */
.subsystems { display: flex; flex-direction: column; gap: 7px; }
.sub-item { padding: 10px 14px; background: rgba(255,255,255,.02); border: 1px solid var(--border); border-radius: var(--radius-sm); transition: border-color .2s; }
.sub-item:hover { border-color: rgba(6,182,212,.3); }
.sub-name { font-size: 12px; font-weight: 700; color: var(--text); margin-bottom: 2px; }
.sub-url  { font-size: 10px; color: var(--cyan); opacity:.7; }
.sub-desc { font-size: 11px; color: var(--text-3); line-height: 1.55; margin-top: 3px; }
/* Tags */
.pc-tags { display: flex; flex-wrap: wrap; gap: 5px; margin-top: auto; padding-top: 6px; }
.pc-tag { padding: 4px 10px; background: rgba(255,255,255,.04); border: 1px solid rgba(255,255,255,.07); border-radius: 4px; font-size: 10px; color: var(--text-3); transition: all .2s; }
.project-card:hover .pc-tag { border-color: var(--border-mid); color: var(--text-2); }
@media (max-width: 720px) { .pc-inner { grid-template-columns: 1fr; } .pc-left { border-right: none; border-bottom: 1px solid var(--border); } .arch-children { grid-template-columns: repeat(2,1fr); } }
```

- [ ] **Step 2: Add projects HTML after `#build` section**

```html
<section id="projects" class="section-wrap">
  <div class="reveal">
    <p style="font-size:10px;letter-spacing:.22em;text-transform:uppercase;color:var(--accent);margin-bottom:10px;font-weight:700;">Work</p>
    <h2 style="font-family:var(--font-serif);font-size:clamp(28px,4vw,44px);font-weight:normal;letter-spacing:-.5px;color:var(--text);margin-bottom:40px;">Production Systems</h2>
  </div>

  <div class="project-card reveal" data-tilt>
    <div class="pc-inner">
      <div class="pc-left">
        <div class="pc-num">Project 01</div>
        <div class="pc-name">Aarna Business Ecosystem</div>
        <span class="pc-badge badge-live">Live in Production</span>
        <a href="https://aarna.rjglobalgroup.in" target="_blank" rel="noopener" class="pc-url">↗ aarna.rjglobalgroup.in</a>
        <p class="pc-meta">4 sub-systems · 1 shared backend<br>Built solo — no team, no funding</p>
      </div>
      <div class="pc-right">
        <p class="pc-problem-label">The Problem</p>
        <p class="pc-quote">"A flower retail business running entirely on paper, WhatsApp notebooks, and staff memory."</p>
        <p class="pc-built-label">What Was Built</p>
        <div class="pc-built"><p>A complete operating ecosystem — 19-module BMS, WhatsApp AI commerce agent, marketing website, and e-commerce storefront. All four controlled by one backend, built solo.</p></div>
        <div class="arch" aria-label="System architecture diagram">
          <div class="arch-root"><div class="arch-root-node">Node.js / PostgreSQL Backend API</div></div>
          <div class="arch-lines" aria-hidden="true"><div class="arch-line"></div><div class="arch-line"></div><div class="arch-line"></div><div class="arch-line"></div></div>
          <div class="arch-children">
            <div class="arch-child">BMS<br>19 Modules</div>
            <div class="arch-child">WhatsApp<br>AI Agent</div>
            <div class="arch-child">Marketing<br>Site</div>
            <div class="arch-child">E-Commerce<br>Store</div>
          </div>
        </div>
        <div class="subsystems">
          <div class="sub-item"><div class="sub-name">Aarna BMS</div><div class="sub-url">ops.aarna.rjglobalgroup.in</div><div class="sub-desc">19 modules: POS, CRM, 4-tier subscription engine, inventory batch tracking with automated wastage ladder, vendor ledger, delivery management, CA-ready financial reports.</div></div>
          <div class="sub-item"><div class="sub-name">Aarna AI — WhatsApp Commerce Agent</div><div class="sub-desc">Three-layer decision architecture: rule-based state machine → keyword triggers → Llama 3.3 70B via Groq. Hidden action system. Handles full purchase journey without staff.</div></div>
          <div class="sub-item"><div class="sub-name">Aarna Marketing + E-Commerce</div><div class="sub-url">aarna.rjglobalgroup.in · shop.aarna.rjglobalgroup.in</div><div class="sub-desc">BMS-controlled content, GPS lead capture, live inventory ticker. OTP auth, Razorpay checkout, order history.</div></div>
        </div>
        <div class="pc-tags"><span class="pc-tag">Node.js</span><span class="pc-tag">React</span><span class="pc-tag">Next.js 15</span><span class="pc-tag">PostgreSQL</span><span class="pc-tag">Groq</span><span class="pc-tag">Llama 3.3 70B</span><span class="pc-tag">Razorpay</span><span class="pc-tag">Meta WhatsApp API</span><span class="pc-tag">Socket.IO</span><span class="pc-tag">Railway</span><span class="pc-tag">Vercel</span></div>
      </div>
    </div>
  </div>

  <div class="project-card reveal reveal-delay-1" data-tilt>
    <div class="pc-inner">
      <div class="pc-left">
        <div class="pc-num">Project 02</div>
        <div class="pc-name">MENZO Restaurant BMS</div>
        <span class="pc-badge badge-dev">Seeking First Client</span>
        <p class="pc-meta">Full billing → WhatsApp invoice<br>Multi-franchise dashboard<br>Built solo — no funding, no team</p>
      </div>
      <div class="pc-right">
        <p class="pc-problem-label">The Problem</p>
        <p class="pc-quote">"Restaurant owners managing billing, kitchen orders, inventory, and delivery across disconnected tools and paper."</p>
        <p class="pc-built-label">What Was Built</p>
        <div class="pc-built"><p>Production-ready restaurant BMS — full billing engine to WhatsApp invoice, KOT management, visual floor layout, unified delivery board (own riders + Swiggy/Zomato/Direct), recipe-linked inventory, GST reporting, multi-franchise dashboard.</p></div>
        <div class="pc-tags"><span class="pc-tag">Node.js</span><span class="pc-tag">React</span><span class="pc-tag">PostgreSQL</span><span class="pc-tag">WhatsApp Business API</span><span class="pc-tag">Razorpay</span><span class="pc-tag">Railway</span></div>
      </div>
    </div>
  </div>

  <div class="project-card reveal reveal-delay-2" data-tilt>
    <div class="pc-inner">
      <div class="pc-left">
        <div class="pc-num">Project 03</div>
        <div class="pc-name">STEMLYN Revenue Continuity Infrastructure</div>
        <span class="pc-badge badge-dev">Active Development</span>
        <p class="pc-meta">Category: Revenue Continuity Infrastructure</p>
      </div>
      <div class="pc-right">
        <p class="pc-problem-label">The Problem</p>
        <p class="pc-quote">"Growing businesses lose revenue not from bad marketing but from broken execution — delayed responses, inconsistent follow-up, untracked pipeline."</p>
        <p class="pc-built-label">What Was Built</p>
        <div class="pc-built"><p>AI-powered infrastructure that eliminates execution gaps between pipeline and closed revenue. The category: Revenue Continuity Infrastructure — systems that ensure nothing falls through between a warm lead and a signed deal.</p></div>
        <div class="pc-tags"><span class="pc-tag">AI Systems</span><span class="pc-tag">GTM Engineering</span><span class="pc-tag">Revenue Operations</span><span class="pc-tag">Automation</span></div>
      </div>
    </div>
  </div>
</section>
```

- [ ] **Step 3: Add 3D tilt JS inside `<script>`**

```js
/* 3D CARD TILT */
(function() {
  if ('ontouchstart' in window) return;
  document.querySelectorAll('[data-tilt]').forEach(card => {
    card.addEventListener('mousemove', e => {
      const r = card.getBoundingClientRect();
      const x = (e.clientX - r.left) / r.width  - 0.5;
      const y = (e.clientY - r.top)  / r.height - 0.5;
      card.style.transform = 'perspective(1000px) rotateY(' + (x * 7) + 'deg) rotateX(' + (-y * 5) + 'deg) scale3d(1.01,1.01,1.01)';
    });
    card.addEventListener('mouseleave', () => { card.style.transform = ''; });
  });
})();
```

- [ ] **Step 4: Verify — cards stagger in, hover tilts in 3D, arch diagram visible for Aarna**

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: projects — 3D tilt, arch diagram, 3 project cards"
```

---

## Task 7: Stats Strip

- [ ] **Step 1: Add CSS inside `<style>`**

```css
/* STATS STRIP */
#stats {
  border-top: 1px solid var(--border); border-bottom: 1px solid var(--border);
  padding: clamp(60px,8vw,100px) clamp(20px,6vw,100px);
  display: grid; grid-template-columns: repeat(3,1fr);
  background: var(--bg-surface);
}
.stat-block { text-align: center; padding: 20px; }
.stat-big {
  font-family: var(--font-serif);
  font-size: clamp(56px,10vw,100px); font-weight: normal;
  letter-spacing: -3px; line-height: 1; color: var(--text);
}
.stat-big-label { font-size: 12px; color: var(--text-3); margin-top: 10px; letter-spacing:.08em; text-transform: uppercase; }
@media (max-width: 500px) { #stats { grid-template-columns: 1fr; } }
```

- [ ] **Step 2: Add HTML after projects section**

```html
<section id="stats">
  <div class="stat-block reveal"><div class="stat-big" data-counter="6">0</div><div class="stat-big-label">Production Systems</div></div>
  <div class="stat-block reveal reveal-delay-1"><div class="stat-big" data-counter="3">0</div><div class="stat-big-label">Live URLs</div></div>
  <div class="stat-block reveal reveal-delay-2"><div class="stat-big" data-counter="1">0</div><div class="stat-big-label">Backend Controls All</div></div>
</section>
```

*(Counter JS from Task 3 handles `[data-counter]` automatically)*

- [ ] **Step 3: Verify — 3 giant numbers count up from 0 when scrolled into view**

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: stats strip — giant counters"
```

---

## Task 8: About Section

- [ ] **Step 1: Add CSS inside `<style>`**

```css
/* ABOUT */
#about { border-top: 1px solid var(--border); display: grid; grid-template-columns: 1fr 1fr; min-height: 600px; }
.about-photo-col { position: relative; overflow: hidden; background: linear-gradient(135deg, #04040c 0%, #09091f 100%); }
.about-photo-inner {
  position: absolute; inset: 0;
  background: radial-gradient(ellipse at 50% 90%, rgba(37,99,235,.18) 0%, transparent 60%);
  display: flex; align-items: flex-end; justify-content: center;
}
.about-photo-inner img { width: 100%; height: 120%; object-fit: cover; object-position: center top; display: block; position: absolute; top: 0; left: 0; }
.about-sil { width: 200px; height: 300px; border-radius: 100px 100px 0 0; background: linear-gradient(180deg,rgba(37,99,235,.14) 0%,transparent 100%); border: 1px solid rgba(37,99,235,.15); border-bottom: none; position: relative; }
.about-sil::before { content:''; position: absolute; top: 32px; left: 50%; transform: translateX(-50%); width: 70px; height: 70px; border-radius: 50%; background: rgba(37,99,235,.18); }
.about-text-col { padding: clamp(48px,6vw,80px) clamp(32px,5vw,72px); display: flex; flex-direction: column; justify-content: center; gap: 20px; background: var(--bg); }
.about-text-col p { font-size: clamp(14px,1.5vw,16px); color: var(--text-2); line-height: 1.8; }
.about-text-col p strong { color: var(--text); font-weight: 600; }
.about-cards { display: flex; flex-direction: column; gap: 10px; margin-top: 6px; }
.about-card { padding: 16px 18px; background: var(--bg-surface); border: 1px solid var(--border); border-radius: var(--radius-md); }
.about-card-label { font-size: 9px; font-weight: 700; letter-spacing:.2em; text-transform: uppercase; color: var(--text-4); margin-bottom: 6px; }
.about-card p { font-size: 13px; color: var(--text-2); line-height: 1.55; }
.about-card .hl { color: var(--cyan); }
@media (max-width: 768px) { #about { grid-template-columns: 1fr; } .about-photo-col { min-height: 320px; } }
```

- [ ] **Step 2: Add HTML after stats section**

```html
<section id="about">
  <div class="about-photo-col">
    <div class="about-photo-inner parallax-el" data-parallax="0.2" id="about-photo-inner">
      <div class="about-sil" aria-hidden="true"></div>
    </div>
  </div>
  <div class="about-text-col">
    <div class="reveal">
      <p style="font-size:10px;letter-spacing:.22em;text-transform:uppercase;color:var(--accent);margin-bottom:10px;font-weight:700;">Background</p>
      <h2 style="font-family:var(--font-serif);font-size:clamp(28px,3.5vw,40px);font-weight:normal;letter-spacing:-.5px;color:var(--text);">About</h2>
    </div>
    <p class="reveal reveal-delay-1"><strong>I don't come from a big tech company or a funded startup.</strong> I identified real operational problems in real businesses, designed the systems to fix them, and shipped everything solo — from architecture to production deployment.</p>
    <p class="reveal reveal-delay-2">My edge isn't just technical. <strong>I think in business outcomes first,</strong> then build the systems that achieve them. That's what makes me useful to a GTM team.</p>
    <div class="about-cards">
      <div class="about-card reveal reveal-delay-3"><div class="about-card-label">Currently</div><p><span class="hl">MBA in International Business</span> · Golden Gate University</p></div>
      <div class="about-card reveal reveal-delay-4"><div class="about-card-label">Looking For</div><p>Remote GTM Engineer roles with US-based startups — early-stage where speed matters.</p></div>
    </div>
  </div>
</section>
```

- [ ] **Step 3: Add about photo loader JS inside `<script>`**

```js
/* ABOUT PHOTO */
(function() {
  const inner = document.getElementById('about-photo-inner');
  if (!inner) return;
  const img = document.createElement('img');
  img.alt = 'Rohan RJ';
  img.src = 'assets/rohan-photo.jpg';
  img.onload = function() {
    // clear placeholder silhouette, insert photo
    const sil = inner.querySelector('.about-sil');
    if (sil) sil.remove();
    inner.insertBefore(img, inner.firstChild);
  };
})();
```

- [ ] **Step 4: Verify — text reveals staggered, photo col shows silhouette or real photo with parallax**

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: about — split layout, parallax photo col, sidebar cards"
```

---

## Task 9: Tech Stack Section

- [ ] **Step 1: Add CSS inside `<style>`**

```css
/* TECH STACK */
#stack { border-top: 1px solid var(--border); }
.stack-group { margin-bottom: 32px; }
.stack-group:last-child { margin-bottom: 0; }
.stack-cat { font-size: 10px; font-weight: 700; letter-spacing:.2em; text-transform: uppercase; color: var(--text-4); margin-bottom: 12px; padding-bottom: 8px; border-bottom: 1px solid var(--border); }
.stack-tags { display: flex; flex-wrap: wrap; gap: 8px; }
.stack-tag { padding: 8px 16px; background: var(--bg-surface); border: 1px solid var(--border); border-radius: var(--radius-sm); font-size: 13px; color: var(--text-2); transition: background .2s, border-color .2s, color .2s, transform .18s var(--ease); cursor: default; }
.stack-tag:hover { background: var(--accent-dim); border-color: rgba(37,99,235,.4); color: var(--text); transform: translateY(-2px); }
```

- [ ] **Step 2: Add HTML after about section**

```html
<section id="stack" class="section-wrap">
  <div class="reveal">
    <p style="font-size:10px;letter-spacing:.22em;text-transform:uppercase;color:var(--accent);margin-bottom:10px;font-weight:700;">Tools &amp; Technologies</p>
    <h2 style="font-family:var(--font-serif);font-size:clamp(28px,4vw,44px);font-weight:normal;letter-spacing:-.5px;color:var(--text);margin-bottom:40px;">Tech Stack</h2>
  </div>
  <div class="stack-group reveal reveal-delay-1">
    <div class="stack-cat">GTM Tools</div>
    <div class="stack-tags"><span class="stack-tag">Clay</span><span class="stack-tag">Apollo</span><span class="stack-tag">HubSpot</span><span class="stack-tag">Make.com</span><span class="stack-tag">Zapier</span><span class="stack-tag">Instantly</span></div>
  </div>
  <div class="stack-group reveal reveal-delay-2">
    <div class="stack-cat">AI &amp; Automation</div>
    <div class="stack-tags"><span class="stack-tag">Groq</span><span class="stack-tag">Llama 3.3 70B</span><span class="stack-tag">Meta WhatsApp Cloud API</span><span class="stack-tag">Prompt Engineering</span></div>
  </div>
  <div class="stack-group reveal reveal-delay-3">
    <div class="stack-cat">Development</div>
    <div class="stack-tags"><span class="stack-tag">Node.js</span><span class="stack-tag">Next.js 15</span><span class="stack-tag">React</span><span class="stack-tag">Python</span><span class="stack-tag">PostgreSQL</span><span class="stack-tag">Socket.IO</span><span class="stack-tag">Railway</span><span class="stack-tag">Vercel</span></div>
  </div>
  <div class="stack-group reveal reveal-delay-4">
    <div class="stack-cat">Business Systems</div>
    <div class="stack-tags"><span class="stack-tag">Razorpay</span><span class="stack-tag">GST Compliance</span><span class="stack-tag">CRM Architecture</span><span class="stack-tag">Revenue Operations</span></div>
  </div>
</section>
```

- [ ] **Step 3: Verify — tags grouped, hover lifts with blue border**

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: tech stack — grouped tags with hover lift"
```

---

## Task 10: Contact + Typewriter + Footer

- [ ] **Step 1: Add CSS inside `<style>`**

```css
/* CONTACT */
#contact { border-top: 1px solid var(--border); text-align: center; padding: clamp(80px,12vw,140px) clamp(20px,8vw,120px) clamp(60px,8vw,100px); }
.contact-heading { font-family: var(--font-serif); font-size: clamp(40px,8vw,88px); font-weight: normal; letter-spacing: -2px; color: var(--text); line-height: 1; margin-bottom: 16px; }
.contact-sub { font-size: 15px; color: var(--text-3); margin-bottom: 48px; max-width: 500px; margin-left: auto; margin-right: auto; }
.contact-email-wrap { margin-bottom: 32px; font-size: clamp(18px,3vw,28px); }
.contact-email { font-weight: 700; color: var(--text); text-decoration: none; letter-spacing: -.5px; border-bottom: 2px solid var(--accent); padding-bottom: 4px; transition: color .2s; }
.contact-email:hover { color: var(--accent); }
.cursor-blink { display: inline-block; width: 2px; height: 1.1em; background: var(--accent); margin-left: 2px; vertical-align: text-bottom; animation: blink .9s step-end infinite; }
.contact-links { display: flex; justify-content: center; gap: 12px; flex-wrap: wrap; }
.clb { display: inline-flex; align-items: center; gap: 7px; padding: 12px 22px; border-radius: 100px; font-size: 13px; font-weight: 600; text-decoration: none; transition: all .2s; }
.clb-primary { background: var(--text); color: #000; }
.clb-primary:hover { opacity:.88; transform: scale(1.02); }
.clb-secondary { border: 1px solid var(--border-mid); color: var(--text-2); }
.clb-secondary:hover { border-color: rgba(255,255,255,.35); color: var(--text); transform: scale(1.02); }
/* FOOTER */
footer { padding: 24px clamp(20px,5vw,80px); border-top: 1px solid var(--border); display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 12px; }
.footer-copy { font-size: 12px; color: var(--text-4); }
.footer-tagline { font-family: var(--font-serif); font-size: 13px; color: var(--text-4); font-style: italic; }
```

- [ ] **Step 2: Add contact HTML and footer after stack section**

```html
<section id="contact">
  <h2 class="contact-heading reveal">Let's Talk.</h2>
  <p class="contact-sub reveal reveal-delay-1">If your revenue pipeline has execution gaps — I build the infrastructure that closes them.</p>
  <div class="contact-email-wrap reveal reveal-delay-2">
    <a href="mailto:rohanrj4519@gmail.com" class="contact-email" id="contact-email-el"></a><span class="cursor-blink" id="email-cursor" aria-hidden="true"></span>
  </div>
  <div class="contact-links reveal reveal-delay-3">
    <a href="mailto:rohanrj4519@gmail.com" class="clb clb-primary">
      <svg width="14" height="14" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24" aria-hidden="true"><path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22,6 12,13 2,6"/></svg>
      Email Me
    </a>
    <a href="https://linkedin.com/in/rohanrjco" target="_blank" rel="noopener" class="clb clb-secondary">
      <svg width="14" height="14" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24" aria-hidden="true"><path d="M16 8a6 6 0 016 6v7h-4v-7a2 2 0 00-2-2 2 2 0 00-2 2v7h-4v-7a6 6 0 016-6zM2 9h4v12H2z"/><circle cx="4" cy="4" r="2"/></svg>
      LinkedIn
    </a>
  </div>
</section>

<footer>
  <p class="footer-copy">© 2025 Rohan RJ · Mangaluru, India</p>
  <p class="footer-tagline">"I build the systems that keep businesses from bleeding revenue."</p>
</footer>
```

- [ ] **Step 3: Add typewriter JS inside `<script>`**

```js
/* TYPEWRITER — contact email */
(function() {
  const el = document.getElementById('contact-email-el');
  const cur = document.getElementById('email-cursor');
  if (!el) return;

  const EMAIL = 'rohanrj4519@gmail.com';
  let done = false;

  const obs = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (!entry.isIntersecting || done) return;
      done = true;
      obs.unobserve(entry.target);

      let i = 0;
      el.textContent = '';

      function type() {
        if (i <= EMAIL.length) {
          el.textContent = EMAIL.slice(0, i);
          i++;
          setTimeout(type, 52);
        } else {
          setTimeout(() => { if (cur) cur.style.display = 'none'; }, 2800);
        }
      }
      setTimeout(type, 400);
    });
  }, { threshold: 0.6 });

  obs.observe(el);
})();
```

- [ ] **Step 4: Verify — scroll to contact, email types character by character, cursor blinks then disappears**

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: contact — typewriter email, footer"
```

---

## Task 11: Mobile Polish + Reduced Motion

- [ ] **Step 1: Add final responsive CSS inside `<style>` at very bottom**

```css
/* MOBILE */
@media (max-width: 480px) {
  .hero-actions { flex-direction: column; }
  .btn-pill-white, .btn-pill-ghost { justify-content: center; }
  .hero-stats { gap: 16px; }
  .contact-links { flex-direction: column; align-items: center; }
  .footer-tagline { display: none; }
}

/* REDUCED MOTION */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
  .reveal { opacity: 1 !important; transform: none !important; }
  .manifesto-body .w { color: var(--text) !important; }
  .manifesto-body .w.accent-word { color: var(--accent) !important; }
  .manifesto-sub { opacity: 1 !important; }
}
```

- [ ] **Step 2: Test at 390px (iPhone 14)**

Chrome DevTools → device toolbar → 390px. Confirm:
- No horizontal scroll
- Hero is single column (right col hidden)
- Project cards stack to single column
- Stats stack to 1 column
- Contact links stack vertically

- [ ] **Step 3: Final commit**

```bash
git add index.html
git commit -m "chore: mobile pass + reduced-motion accessibility"
```

---

## Task 12: Asset Drop-In + Final QA

- [ ] **Step 1: Create assets folder and drop photos when ready**

```bash
mkdir -p "/Users/rohanraj/Rohan RJ/my portfolio/assets"
# Then copy:
#   assets/rohan-photo.jpg   (portrait photo)
#   assets/rohan-loop.mp4   (Higgsfield animated video)
```

- [ ] **Step 2: Create `.gitignore`**

```
.DS_Store
.superpowers/
node_modules/
```

- [ ] **Step 3: Full QA checklist at 1440px**

| # | What to check | Expected result |
|---|---------------|-----------------|
| 1 | Page load | Black, nav hidden, hero fades in with stagger |
| 2 | Mouse movement | White dot follows with smooth inertia |
| 3 | Hover button | Cursor expands to large soft circle |
| 4 | Scroll 10px | Nav blur appears, progress bar grows |
| 5 | Manifesto section | Words light up sequentially, "broken execution." → blue |
| 6 | Projects hover | Card tilts in 3D on mouse move, resets on leave |
| 7 | Stats strip | Numbers count 0→6, 0→3, 0→1 |
| 8 | About photo | Photo or silhouette moves slower than page scroll |
| 9 | Contact section | Email types out letter by letter, cursor blinks |
| 10 | 390px viewport | All above works, no horizontal overflow |

- [ ] **Step 4: Commit and deploy**

```bash
git add .
git commit -m "feat: complete Apple Clean Dark portfolio — 10 interactions, Higgsfield ready"

# GitHub Pages
git remote add origin https://github.com/YOURUSERNAME/portfolio.git
git push -u origin main
# Enable Pages: repo Settings → Pages → Branch: main → / (root)

# OR Vercel (one command)
npx vercel --yes
```

---

## Spec Coverage Check

- ✅ Interaction 1 — Magnetic cursor: Task 1
- ✅ Interaction 2 — Word-by-word scroll reveal: Task 4
- ✅ Interaction 3 — Parallax photo: Tasks 3 + 8
- ✅ Interaction 4 — Higgsfield video loop: Task 3
- ✅ Interaction 5 — Counter animations: Tasks 3 + 7
- ✅ Interaction 6 — Typewriter CTA: Task 10
- ✅ Interaction 7 — Smooth scroll inertia: Task 1
- ✅ Interaction 8 — 3D card tilt: Task 6
- ✅ Interaction 9 — Scroll progress bar: Task 1
- ✅ Interaction 10 — Stagger scroll reveals: Tasks 1 + 5,6,8,9,10
- ✅ Section: Hero — Task 3
- ✅ Section: Manifesto — Task 4
- ✅ Section: What I Build — Task 5
- ✅ Section: Projects — Task 6
- ✅ Section: Stats strip — Task 7
- ✅ Section: About — Task 8
- ✅ Section: Stack — Task 9
- ✅ Section: Contact + Footer — Task 10
- ✅ Mobile — Task 11
- ✅ Asset drop-in — Task 12
