# Portfolio Enhancements Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use compose:subagent (recommended) or compose:execute to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Enhance the retro portfolio with interactivity (typing effect, XP bar, skill meters, expandable cards, particles, contact form, easter egg) and fix 2 AI-sounding copy lines.

**Architecture:** All changes are in `index.html` (inline `<script>`) and `portfolio.css`. No build tools, no framework, no dependencies. Each feature is self-contained JS + CSS.

**Tech Stack:** Vanilla HTML/CSS/JS, IntersectionObserver API, CSS animations, requestAnimationFrame

---

## File Structure

| File | Changes |
|------|---------|
| `site/index.html` | Copy fixes, new HTML elements (XP bar, skill bars, contact form, particles canvas, expanded project content), new JS blocks |
| `site/portfolio.css` | New styles for XP bar, skill bars, expanded cards, particles, contact form, easter egg |

---

### Task 1: Fix 2 AI-sounding copy lines

**Files:**
- Modify: `site/index.html:181` (security section)
- Modify: `site/index.html:381` (contact section)

- [ ] **Step 1: Fix security copy**

Change line 181 from:
```
When your back needs to be bulletproof, I'll provide the support code. If your product handles sensitive data, private communications, or requires genuine user anonymity, that's a problem I know how to solve.
```
to:
```
When your back needs to be bulletproof, I'll provide the support code. If your product handles sensitive data, private communications, or requires genuine user anonymity — I've built systems for that, and I can do it again.
```

- [ ] **Step 2: Fix contact copy**

Change line 381 from:
```
Need a bulletproof architecture, a full-stack build, or a solid technical strategy? Send me the mission briefing and let's ship the endgame.
```
to:
```
Need a bulletproof architecture, a full-stack build, or a solid technical strategy? Drop me a message and let's figure out what you need.
```

- [ ] **Step 3: Verify**

Open `index.html` in browser. Scroll to Security and Contact sections. Confirm text reads naturally.

---

### Task 2: Typing effect on hero headline

**Files:**
- Modify: `site/index.html:40` (hero headline h1)
- Modify: `site/index.html` (script section, add typing logic)

- [ ] **Step 1: Update the HTML**

Replace:
```html
<h1 class="hero-headline">Code. Code never changes. But the frameworks do</h1>
```
with:
```html
<h1 class="hero-headline" id="hero-headline" data-text="Code. Code never changes. But the frameworks do"><span class="typing-text"></span><span class="typing-cursor">█</span></h1>
```

- [ ] **Step 2: Add CSS for cursor blink**

Add to `portfolio.css` after the `.hero-headline::after` rule (around line 280):
```css
.hero-headline .typing-cursor {
  display: inline-block;
  color: var(--hot);
  animation: blink 0.7s steps(2, start) infinite;
  margin-left: 2px;
  font-size: inherit;
}

.hero-headline .typing-text {
  /* inherits parent styles */
}

/* Hide the static ::after cursor when typing is active */
.hero-headline[data-text]::after {
  display: none;
}
```

- [ ] **Step 3: Add JS typing logic**

Add to the `<script>` section, before the achievement toast code:
```javascript
/* ── Typing effect on hero headline ──────────────────── */
(function() {
  const headline = document.getElementById('hero-headline');
  if (!headline) return;
  const text = headline.dataset.text;
  const textEl = headline.querySelector('.typing-text');
  const cursorEl = headline.querySelector('.typing-cursor');
  let i = 0;
  const speed = 55;

  function type() {
    if (i < text.length) {
      textEl.textContent += text.charAt(i);
      i++;
      setTimeout(type, speed + (Math.random() * 30 - 15));
    } else {
      cursorEl.style.animation = 'blink 1s steps(2,start) infinite';
    }
  }

  const obs = new IntersectionObserver((entries) => {
    if (entries[0].isIntersecting) {
      obs.disconnect();
      setTimeout(type, 400);
    }
  }, { threshold: 0.5 });
  obs.observe(headline);
})();
```

- [ ] **Step 4: Verify**

Open in browser. The headline should type out character by character with a blinking cursor. The static `::after` square should be hidden.

---

### Task 3: Scroll progress XP bar

**Files:**
- Modify: `site/index.html` (add XP bar element after `<body>`)
- Modify: `site/portfolio.css` (add XP bar styles)
- Modify: `site/index.html` (add scroll listener JS)

- [ ] **Step 1: Add HTML element**

Insert after `<body>` and before the CRT overlay:
```html
<!-- SCROLL XP BAR -->
<div class="xp-bar" id="xp-bar">
  <div class="xp-bar-fill" id="xp-bar-fill"></div>
  <span class="xp-bar-label" id="xp-bar-label">0%</span>
</div>
```

- [ ] **Step 2: Add CSS**

Add to `portfolio.css` after the CRT section:
```css
/* ─── SCROLL XP BAR ──────────────────────────────────────── */
.xp-bar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 6px;
  background: var(--bg-dark);
  z-index: 300;
  border-bottom: 2px solid var(--ink);
}

.xp-bar-fill {
  height: 100%;
  width: 0%;
  background: var(--accent);
  transition: width 0.1s steps(10, end);
}

.xp-bar-label {
  position: absolute;
  right: 8px;
  top: 10px;
  font-family: var(--font-display);
  font-size: 7px;
  color: var(--text-dim);
  opacity: 0;
  transition: opacity 0.2s;
}

.xp-bar:hover .xp-bar-label {
  opacity: 1;
}
```

- [ ] **Step 3: Add JS**

Add to the `<script>` section:
```javascript
/* ── Scroll XP bar ──────────────────────────────────── */
const xpFill = document.getElementById('xp-bar-fill');
const xpLabel = document.getElementById('xp-bar-label');
window.addEventListener('scroll', () => {
  const scrollable = document.documentElement.scrollHeight - window.innerHeight;
  const pct = scrollable > 0 ? Math.round((window.scrollY / scrollable) * 100) : 0;
  xpFill.style.width = pct + '%';
  if (xpLabel) xpLabel.textContent = pct + '%';
}, { passive: true });
```

- [ ] **Step 4: Verify**

Open in browser. A thin green bar at the top should fill as you scroll. Hovering shows percentage.

---

### Task 4: Skill bars with scroll animation

**Files:**
- Modify: `site/index.html:123-166` (replace flat tags with skill bars)
- Modify: `site/portfolio.css` (add skill bar styles)

- [ ] **Step 1: Replace skills section HTML**

Replace the entire `<div class="skills-grid">` (lines 123-166) with:
```html
<div class="skills-grid">
  <div class="skills-col">
    <div class="skills-col-header">
      <span class="skills-col-dot skills-col-dot--love"></span>
      What I reach for first
    </div>
    <div class="skill-bars">
      <div class="skill-bar" data-level="95">
        <div class="skill-bar-label">Angular</div>
        <div class="skill-bar-track"><div class="skill-bar-fill"></div></div>
      </div>
      <div class="skill-bar" data-level="90">
        <div class="skill-bar-label">Ionic</div>
        <div class="skill-bar-track"><div class="skill-bar-fill"></div></div>
      </div>
      <div class="skill-bar" data-level="92">
        <div class="skill-bar-label">.NET / C#</div>
        <div class="skill-bar-track"><div class="skill-bar-fill"></div></div>
      </div>
      <div class="skill-bar" data-level="93">
        <div class="skill-bar-label">TypeScript</div>
        <div class="skill-bar-track"><div class="skill-bar-fill"></div></div>
      </div>
      <div class="skill-bar" data-level="85">
        <div class="skill-bar-label">WebRTC</div>
        <div class="skill-bar-track"><div class="skill-bar-fill"></div></div>
      </div>
      <div class="skill-bar" data-level="88">
        <div class="skill-bar-label">SignalR</div>
        <div class="skill-bar-track"><div class="skill-bar-fill"></div></div>
      </div>
      <div class="skill-bar" data-level="90">
        <div class="skill-bar-label">IIS / Windows Server</div>
        <div class="skill-bar-track"><div class="skill-bar-fill"></div></div>
      </div>
      <div class="skill-bar" data-level="87">
        <div class="skill-bar-label">PostgreSQL</div>
        <div class="skill-bar-track"><div class="skill-bar-fill"></div></div>
      </div>
      <div class="skill-bar" data-level="82">
        <div class="skill-bar-label">Signal Protocol</div>
        <div class="skill-bar-track"><div class="skill-bar-fill"></div></div>
      </div>
      <div class="skill-bar" data-level="88">
        <div class="skill-bar-label">CLI tooling</div>
        <div class="skill-bar-track"><div class="skill-bar-fill"></div></div>
      </div>
      <div class="skill-bar" data-level="80">
        <div class="skill-bar-label">AI engineering</div>
        <div class="skill-bar-track"><div class="skill-bar-fill"></div></div>
      </div>
      <div class="skill-bar" data-level="78">
        <div class="skill-bar-label">LiveKit SFU</div>
        <div class="skill-bar-track"><div class="skill-bar-fill"></div></div>
      </div>
      <div class="skill-bar" data-level="85">
        <div class="skill-bar-label">Redis</div>
        <div class="skill-bar-track"><div class="skill-bar-fill"></div></div>
      </div>
      <div class="skill-bar" data-level="82">
        <div class="skill-bar-label">MinIO</div>
        <div class="skill-bar-track"><div class="skill-bar-fill"></div></div>
      </div>
    </div>
  </div>
  <div class="skills-divider"></div>
  <div class="skills-col">
    <div class="skills-col-header">
      <span class="skills-col-dot skills-col-dot--also"></span>
      What I also deliver
    </div>
    <div class="skill-bars">
      <div class="skill-bar" data-level="75">
        <div class="skill-bar-label">React</div>
        <div class="skill-bar-track"><div class="skill-bar-fill skill-bar-fill--also"></div></div>
      </div>
      <div class="skill-bar" data-level="72">
        <div class="skill-bar-label">Node.js / NestJS</div>
        <div class="skill-bar-track"><div class="skill-bar-fill skill-bar-fill--also"></div></div>
      </div>
      <div class="skill-bar" data-level="70">
        <div class="skill-bar-label">Next.js</div>
        <div class="skill-bar-track"><div class="skill-bar-fill skill-bar-fill--also"></div></div>
      </div>
      <div class="skill-bar" data-level="68">
        <div class="skill-bar-label">Ubuntu servers</div>
        <div class="skill-bar-track"><div class="skill-bar-fill skill-bar-fill--also"></div></div>
      </div>
      <div class="skill-bar" data-level="65">
        <div class="skill-bar-label">WordPress</div>
        <div class="skill-bar-track"><div class="skill-bar-fill skill-bar-fill--also"></div></div>
      </div>
      <div class="skill-bar" data-level="60">
        <div class="skill-bar-label">OpenCart</div>
        <div class="skill-bar-track"><div class="skill-bar-fill skill-bar-fill--also"></div></div>
      </div>
      <div class="skill-bar" data-level="70">
        <div class="skill-bar-label">SEO</div>
        <div class="skill-bar-track"><div class="skill-bar-fill skill-bar-fill--also"></div></div>
      </div>
      <div class="skill-bar" data-level="72">
        <div class="skill-bar-label">MySQL</div>
        <div class="skill-bar-track"><div class="skill-bar-fill skill-bar-fill--also"></div></div>
      </div>
      <div class="skill-bar" data-level="68">
        <div class="skill-bar-label">Redux</div>
        <div class="skill-bar-track"><div class="skill-bar-fill skill-bar-fill--also"></div></div>
      </div>
    </div>
    <p class="skills-note">Gets the job done. Not my default choice.</p>
  </div>
</div>
```

- [ ] **Step 2: Add CSS for skill bars**

Add to `portfolio.css` after the `.skills-note` rule:
```css
/* ─── SKILL BARS ──────────────────────────────────────────── */
.skill-bars {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.skill-bar {
  display: flex;
  align-items: center;
  gap: 12px;
}

.skill-bar-label {
  font-family: var(--font-body);
  font-size: 1.05rem;
  color: var(--text);
  min-width: 140px;
  text-align: right;
  flex-shrink: 0;
}

.skill-bar-track {
  flex: 1;
  height: 14px;
  background: var(--bg);
  border: 2px solid var(--ink);
  position: relative;
  overflow: hidden;
}

.skill-bar-fill {
  height: 100%;
  width: 0%;
  background: var(--accent);
  transition: width 1s steps(20, end);
}

.skill-bar-fill--also {
  background: var(--sky);
}

.skill-bar.is-animated .skill-bar-fill {
  width: var(--bar-width, 0%);
}

@media (max-width: 520px) {
  .skill-bar { flex-direction: column; align-items: flex-start; gap: 4px; }
  .skill-bar-label { min-width: auto; text-align: left; }
}
```

- [ ] **Step 3: Add JS for skill bar animation**

Add to the `<script>` section:
```javascript
/* ── Skill bars: animate on scroll ──────────────────── */
document.querySelectorAll('.skill-bar').forEach(bar => {
  const level = bar.dataset.level;
  bar.style.setProperty('--bar-width', level + '%');
});
const skillObs = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('is-animated');
      skillObs.unobserve(entry.target);
    }
  });
}, { threshold: 0.3 });
document.querySelectorAll('.skill-bar').forEach(bar => skillObs.observe(bar));
```

- [ ] **Step 4: Verify**

Open in browser. Scroll to Skills section. Bars should animate from 0% to their target width when they enter the viewport.

---

### Task 5: Project card expand/collapse

**Files:**
- Modify: `site/index.html:201-261` (add hidden detail blocks to project cards)
- Modify: `site/portfolio.css` (add expand styles)
- Modify: `site/index.html` (add expand JS)

- [ ] **Step 1: Add hidden detail content to each project card**

For each `.project-card`, add a `<div class="project-detail">` after the `<div class="project-tags">`. Example for the first card (lines 202-216):

After:
```html
<div class="project-tags">
  ...
</div>
```
Add:
```html
<div class="project-detail">
  <p>Consumer app, business app, admin panel, and .NET backend — deployed on IIS with full service worker lifecycle management and zero-downtime update propagation.</p>
</div>
```

Repeat for each card with appropriate detail text:

**Card 2 (Secure Chat Messenger):**
```html
<div class="project-detail">
  <p>Key challenges: implementing Signal Protocol key exchange in the browser, managing WebRTC peer connections through restrictive firewalls, and building a message queue that works offline. Result: a messenger where even we can't read the messages.</p>
</div>
```

**Card 3 (Done Qpon / SkillDeal):**
```html
<div class="project-detail">
  <p>Started as a weekend experiment, grew into a working marketplace. Did a full security audit, refactored the auth flow, hardened the API, and cleaned up the data model. Still running as a hobby project.</p>
</div>
```

**Card 4 (Creavi):**
```html
<div class="project-detail">
  <p>Worked across multiple products: video conferencing for internal teams, entertainment portal with live ticket sales, scheduling systems with complex recurring patterns. Also shipped Android and iOS builds via Ionic.</p>
</div>
```

- [ ] **Step 2: Add CSS for expand/collapse**

Add to `portfolio.css` after the project card styles:
```css
/* ─── PROJECT CARD EXPAND ─────────────────────────────────── */
.project-detail {
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.3s steps(10, end), padding 0.3s;
  padding: 0 0;
}

.project-card.is-expanded .project-detail {
  max-height: 200px;
  padding: 1rem 0 0;
}

.project-card.is-expanded .project-detail p {
  font-size: 1rem;
  color: var(--text-muted);
  line-height: 1.45;
  border-top: 2px dashed var(--border-lt);
  padding-top: 1rem;
}

.project-card.is-expanded {
  border-color: var(--accent-dk);
  box-shadow: 4px 4px 0 var(--accent-dk);
}
```

- [ ] **Step 3: Add JS for expand/collapse**

Add to the `<script>` section:
```javascript
/* ── Project cards: expand/collapse ─────────────────── */
document.querySelectorAll('.project-card').forEach(card => {
  card.addEventListener('click', (e) => {
    if (e.target.closest('a')) return;
    card.classList.toggle('is-expanded');
  });
});
```

- [ ] **Step 4: Verify**

Open in browser. Click a project card. It should expand to show detail text. Click again to collapse.

---

### Task 6: Floating pixel particles in hero

**Files:**
- Modify: `site/index.html` (add canvas element + JS)
- Modify: `site/portfolio.css` (canvas positioning)

- [ ] **Step 1: Add canvas element**

Insert inside `.hero`, after `<div class="container">` closing tag and before `</section>`:
```html
<canvas class="hero-particles" id="hero-particles"></canvas>
```

- [ ] **Step 2: Add CSS**

Add to `portfolio.css` in the hero section:
```css
.hero-particles {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  pointer-events: none;
}
```

- [ ] **Step 3: Add JS particle system**

Add to the `<script>` section:
```javascript
/* ── Floating pixel particles ───────────────────────── */
(function() {
  const canvas = document.getElementById('hero-particles');
  if (!canvas) return;
  const ctx = canvas.getContext('2d');
  const hero = canvas.parentElement;
  let particles = [];
  const COLORS = ['#2ec27e','#ffd23f','#ff5d73','#4ea8de','#9b5de5'];
  const COUNT = 25;

  function resize() {
    canvas.width = hero.offsetWidth;
    canvas.height = hero.offsetHeight;
  }

  function spawn() {
    return {
      x: Math.random() * canvas.width,
      y: Math.random() * canvas.height,
      size: 2 + Math.floor(Math.random() * 3) * 2,
      color: COLORS[Math.floor(Math.random() * COLORS.length)],
      vx: (Math.random() - 0.5) * 0.4,
      vy: -0.2 - Math.random() * 0.3,
      life: 1,
      decay: 0.002 + Math.random() * 0.003
    };
  }

  function init() {
    resize();
    particles = [];
    for (let i = 0; i < COUNT; i++) {
      const p = spawn();
      p.life = Math.random();
      particles.push(p);
    }
  }

  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    for (let i = particles.length - 1; i >= 0; i--) {
      const p = particles[i];
      p.x += p.vx;
      p.y += p.vy;
      p.life -= p.decay;
      if (p.life <= 0) { particles[i] = spawn(); continue; }
      ctx.globalAlpha = p.life * 0.6;
      ctx.fillStyle = p.color;
      ctx.fillRect(Math.round(p.x), Math.round(p.y), p.size, p.size);
    }
    ctx.globalAlpha = 1;
    requestAnimationFrame(draw);
  }

  init();
  draw();
  window.addEventListener('resize', resize);
})();
```

- [ ] **Step 4: Verify**

Open in browser. Hero section should have subtle colored pixel squares floating upward.

---

### Task 7: Contact form (retro terminal style)

**Files:**
- Modify: `site/index.html:376-398` (add contact form)
- Modify: `site/portfolio.css` (add form styles)
- Modify: `site/index.html` (add form JS)

- [ ] **Step 1: Add contact form HTML**

After the `.contact-links` div (line 395), add:
```html
<form class="contact-form" id="contact-form">
  <div class="contact-form-header">
    <span class="contact-form-prompt">&gt; _</span>
    <span class="contact-form-title">NEW MESSAGE</span>
  </div>
  <div class="contact-form-field">
    <label for="contact-name">NAME:</label>
    <input type="text" id="contact-name" name="name" autocomplete="name" placeholder="enter your name">
  </div>
  <div class="contact-form-field">
    <label for="contact-email">EMAIL:</label>
    <input type="email" id="contact-email" name="email" autocomplete="email" placeholder="enter your email">
  </div>
  <div class="contact-form-field">
    <label for="contact-msg">MESSAGE:</label>
    <textarea id="contact-msg" name="message" rows="4" placeholder="type your message..."></textarea>
  </div>
  <button type="submit" class="contact-form-send">SEND ▶</button>
</form>
```

- [ ] **Step 2: Add CSS for form**

Add to `portfolio.css` after the contact section styles:
```css
/* ─── CONTACT FORM ───────────────────────────────────────── */
.contact-form {
  margin-top: 2.5rem;
  background: var(--bg-card);
  border: 3px solid var(--ink);
  box-shadow: var(--shadow);
  padding: 0;
  max-width: 520px;
}

.contact-form-header {
  background: var(--ink);
  color: var(--bg-card);
  padding: 0.6rem 1rem;
  display: flex;
  align-items: center;
  gap: 12px;
  font-family: var(--font-display);
  font-size: 9px;
  text-transform: uppercase;
}

.contact-form-prompt {
  color: var(--accent);
}

.contact-form-field {
  padding: 0.75rem 1rem;
  border-bottom: 2px dashed var(--border-lt);
  display: flex;
  align-items: flex-start;
  gap: 10px;
}

.contact-form-field label {
  font-family: var(--font-display);
  font-size: 9px;
  color: var(--text-dim);
  text-transform: uppercase;
  min-width: 70px;
  padding-top: 6px;
  flex-shrink: 0;
}

.contact-form-field input,
.contact-form-field textarea {
  flex: 1;
  font-family: var(--font-body);
  font-size: 1.1rem;
  color: var(--text);
  background: transparent;
  border: none;
  outline: none;
  resize: vertical;
  line-height: 1.4;
}

.contact-form-field input::placeholder,
.contact-form-field textarea::placeholder {
  color: var(--text-dim);
  opacity: 0.6;
}

.contact-form-field input:focus,
.contact-form-field textarea:focus {
  background: rgba(46,194,126,0.05);
}

.contact-form-send {
  font-family: var(--font-display);
  font-size: 11px;
  text-transform: uppercase;
  color: var(--bg-card);
  background: var(--accent);
  border: 3px solid var(--ink);
  box-shadow: var(--shadow);
  padding: 0.85rem 1.5rem;
  margin: 1rem;
  cursor: pointer;
  transition: transform 0.1s, box-shadow 0.1s, background 0.12s;
}

.contact-form-send:hover {
  background: var(--accent-dk);
  transform: translate(-2px,-2px);
  box-shadow: var(--shadow-lg);
}

.contact-form-send:active {
  transform: translate(4px,4px);
  box-shadow: 0 0 0 var(--ink);
}
```

- [ ] **Step 3: Add form JS**

Add to the `<script>` section:
```javascript
/* ── Contact form: retro terminal ───────────────────── */
const contactForm = document.getElementById('contact-form');
if (contactForm) {
  contactForm.addEventListener('submit', (e) => {
    e.preventDefault();
    const name = document.getElementById('contact-name').value.trim();
    const email = document.getElementById('contact-email').value.trim();
    const msg = document.getElementById('contact-msg').value.trim();
    if (!name || !email || !msg) {
      unlock('ERROR: all fields required');
      return;
    }
    const subject = encodeURIComponent('Portfolio contact from ' + name);
    const body = encodeURIComponent(msg + '\n\n— ' + name + ' (' + email + ')');
    window.location.href = 'mailto:your@email.com?subject=' + subject + '&body=' + body;
    unlock('Message queued: opening mail client');
    contactForm.reset();
  });
}
```

- [ ] **Step 4: Verify**

Open in browser. Scroll to Contact section. Fill in the form and submit. It should open the mail client with prefilled content.

---

### Task 8: "95" watermark easter egg

**Files:**
- Modify: `site/index.html` (make watermark clickable)
- Modify: `site/portfolio.css` (cursor style)
- Modify: `site/index.html` (add easter egg JS)

- [ ] **Step 1: Make watermark interactive**

The "95" watermark is currently a `::before` pseudo-element (not clickable). Replace with a real element.

Add after the hero `<div class="container">` closing tag, before `<canvas>`:
```html
<div class="hero-watermark" id="hero-watermark" title="Click me">95</div>
```

- [ ] **Step 2: Update CSS**

Replace the `.hero::before` rule (lines 234-246) with:
```css
.hero-watermark {
  position: absolute;
  top: 50%;
  right: -2%;
  transform: translateY(-50%);
  font-family: var(--font-display);
  font-size: 32vw;
  line-height: 1;
  color: rgba(43,43,60,0.05);
  z-index: 0;
  pointer-events: auto;
  cursor: pointer;
  transition: color 0.3s, text-shadow 0.3s;
  user-select: none;
}

.hero-watermark:hover {
  color: rgba(43,43,60,0.12);
  text-shadow: 0 0 40px rgba(255,211,63,0.3);
}
```

Remove or comment out the old `.hero::before` block.

- [ ] **Step 3: Add JS easter egg**

Add to the `<script>` section:
```javascript
/* ── "95" watermark easter egg ──────────────────────── */
const watermark = document.getElementById('hero-watermark');
if (watermark) {
  let clicks = 0;
  watermark.addEventListener('click', () => {
    clicks++;
    if (clicks === 1) {
      unlock('Hmm, what\'s this number?');
    } else if (clicks === 3) {
      unlock('1995 — the year it all started');
      watermark.style.color = 'rgba(46,194,126,0.15)';
      watermark.style.textShadow = '0 0 60px rgba(46,194,126,0.2)';
    } else if (clicks === 7) {
      unlock('C++ and AI research. Those were the days.');
      watermark.style.color = 'rgba(255,93,115,0.15)';
      watermark.style.textShadow = '0 0 60px rgba(255,93,115,0.2)';
    } else if (clicks === 15) {
      unlock('LEGENDARY: you found the deep lore!');
      document.body.classList.add('cheat-mode');
      setTimeout(() => document.body.classList.remove('cheat-mode'), 5000);
    }
  });
}
```

- [ ] **Step 4: Verify**

Open in browser. Click the faded "95" in the hero background. Achievement toasts should appear at 1, 3, 7, and 15 clicks.

---

## Verification Checklist

After all tasks are complete:

- [ ] Open in browser, scroll through entire page
- [ ] Hero headline types out with cursor
- [ ] XP bar fills as you scroll
- [ ] Skill bars animate on scroll
- [ ] Project cards expand/collapse on click
- [ ] Pixel particles float in hero
- [ ] Contact form submits to mail client
- [ ] "95" watermark triggers easter eggs
- [ ] Security copy reads naturally
- [ ] Contact copy reads naturally
- [ ] Mobile: test at 375px width — all features work
- [ ] Konami code still works
- [ ] Achievement toasts still work
