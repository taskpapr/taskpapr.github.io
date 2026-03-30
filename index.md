---
layout: home
title: taskpapr
nav_exclude: true
search_exclude: true
description: "A minimal, paper-inspired task board. No noise, no friction. Self-hosted, spatial, and calm."
---

<div class="hp-hero">
  <div class="hp-hero__inner">
    <h1 class="hp-hero__headline">The task board that works<br>like a sheet of paper.</h1>
    <p class="hp-hero__sub">
      Spatial, not hierarchical. Calm, not cluttered.<br>
      Open it, use it, close it — no maintenance required.
    </p>
    <div class="hp-hero__ctas">
      <a href="{{ '/docs/getting-started' | relative_url }}" class="hp-btn hp-btn--primary">Get started</a>
      <a href="{{ '/docs/features/' | relative_url }}" class="hp-btn hp-btn--secondary">See the features</a>
    </div>
  </div>
</div>

---

<section class="hp-section">
  <h2 class="hp-section__title">Why taskpapr?</h2>
  <div class="hp-why-grid">
    <div class="hp-why-item">
      <h3>No friction to open</h3>
      <p>Your board is always one click away. No loading screens, no onboarding prompts, no "what would you like to do today?" Just the board.</p>
    </div>
    <div class="hp-why-item">
      <h3>Spatial, not hierarchical</h3>
      <p>Tasks live on a canvas you can arrange freely. Position things the way your brain already organises them — not the way a database wants to store them.</p>
    </div>
    <div class="hp-why-item">
      <h3>Done stays visible</h3>
      <p>Completed tasks don't vanish. They stay on the board, faded, until you deliberately clear them — like sweeping a real desk. You decide when the slate is clean.</p>
    </div>
  </div>
</section>

---

<section class="hp-section">
  <h2 class="hp-section__title">What it does</h2>
  <p class="hp-section__intro">Six things taskpapr does that most task tools don't.</p>
  <div class="hp-features-grid">

    <div class="hp-feature-card">
      <div class="hp-feature-card__label">Canvas</div>
      <h3 class="hp-feature-card__title">Infinite canvas, freely draggable tiles</h3>
      <p class="hp-feature-card__body">Tiles live on an open canvas. Drag them anywhere. Group related work by proximity, not by folder. The layout is yours.</p>
    </div>

    <div class="hp-feature-card">
      <div class="hp-feature-card__label">Done</div>
      <h3 class="hp-feature-card__title">Completed tasks stay visible</h3>
      <p class="hp-feature-card__body">Finishing a task marks it, it doesn't erase it. You see what you've done. Clear the board when you're ready — not when the app decides.</p>
    </div>

    <div class="hp-feature-card hp-feature-card--wip">
      <div class="hp-feature-card__label">WIP</div>
      <h3 class="hp-feature-card__title">In-motion state</h3>
      <p class="hp-feature-card__body">An amber left stripe marks tasks that are actively in progress — not just "to do", not yet done. Things in motion look different from things waiting.</p>
    </div>

    <div class="hp-feature-card hp-feature-card--plates">
      <div class="hp-feature-card__label">Plates</div>
      <h3 class="hp-feature-card__title">Spinning plates</h3>
      <p class="hp-feature-card__body">Recurring tasks that need regular attention. The longer since you last touched one, the more urgent it looks. Visual heat, not calendar alerts.</p>
    </div>

    <div class="hp-feature-card hp-feature-card--rot">
      <div class="hp-feature-card__label">Rot</div>
      <h3 class="hp-feature-card__title">Task rot</h3>
      <p class="hp-feature-card__body">Tasks you haven't touched in a while quietly fade — like paper left in a drawer. A gentle signal that something has been forgotten, not a red badge screaming at you.</p>
    </div>

    <div class="hp-feature-card">
      <div class="hp-feature-card__label">Goals</div>
      <h3 class="hp-feature-card__title">Goals view</h3>
      <p class="hp-feature-card__body">Step back and see the big picture without leaving the board. Goals sit above the noise — a quiet reminder of what all this work is actually for.</p>
    </div>

  </div>
</section>

---

<section class="hp-section hp-section--who">
  <h2 class="hp-section__title">Who it's for</h2>
  <div class="hp-who">
    <p>taskpapr was built for people who have tried every task manager and quietly abandoned them all. People who open a new app with hope, spend an hour setting it up, and then never open it again because maintaining the system became its own job.</p>
    <p>It's for people whose brains work spatially — who think in clusters and proximity, not in nested folders. Who need to <em>see</em> what's done to feel like they've made progress. Who get overwhelmed by dashboards full of badges and counts and overdue warnings.</p>
    <p>The paper metaphor isn't decoration. A sheet of paper doesn't nag you. It doesn't reorganise itself while you're not looking. It holds what you put on it, shows you what you've crossed off, and waits patiently until you're ready to deal with the rest. That's what taskpapr tries to be.</p>
  </div>
</section>

---

<section class="hp-section hp-section--selfhost">
  <h2 class="hp-section__title">Self-hosted, by design</h2>
  <div class="hp-selfhost">
    <div class="hp-selfhost__text">
      <p>taskpapr runs on your own machine or server. Your tasks stay yours — no accounts, no cloud sync, no subscription.</p>
      <ul class="hp-selfhost__list">
        <li><strong>Node.js</strong> — runs anywhere Node runs</li>
        <li><strong>SQLite</strong> — a single file, no database server needed</li>
        <li><strong>Docker</strong> — one command to get started</li>
        <li><strong>Mac or Linux</strong> — runs happily on a laptop or a home server</li>
      </ul>
      <a href="/docs/getting-started" class="hp-btn hp-btn--primary">Get started →</a>
    </div>
    <div class="hp-selfhost__aside">
      <div class="hp-callout">
        <p class="hp-callout__label">Quick start</p>
        <pre class="hp-callout__code"><code>docker run -p 3000:3000 \
  -v taskpapr_data:/data \
  ghcr.io/taskpapr/taskpapr</code></pre>
        <p class="hp-callout__note">Then open <strong>localhost:3000</strong> in your browser.</p>
      </div>
    </div>
  </div>
</section>

---

<footer class="hp-footer">
  <div class="hp-footer__inner">
    <span class="hp-footer__name">taskpapr</span>
    <span class="hp-footer__sep">&middot;</span>
    <a href="https://github.com/taskpapr" class="hp-footer__link">GitHub</a>
    <span class="hp-footer__sep">&middot;</span>
    <span class="hp-footer__version">v0.35.1</span>
    <span class="hp-footer__sep">&middot;</span>
    <span class="hp-footer__license">License TBD</span>
  </div>
</footer>
