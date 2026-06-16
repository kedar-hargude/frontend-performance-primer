# THE OPUS FRONTEND PERFORMANCE PRIMER

*A dedicated, problems-first path to becoming a frontend performance expert — the kind who opens DevTools on any site, reads the network waterfall and the flame chart cold, and names the bottleneck before the page finishes loading. Built to be done **before** the general Frontend Primer, from absolute beginner to brutal. Every concept is taught twice: once guided, then a second transfer exercise — both verified by Claude — that you should be able to do with ease if you actually understood it.*

---

## WHAT THIS IS

This is not a course you watch. It is a curriculum you *fight through*. For each concept you will:

1. **Read the Prep** — primary sources (MDN, web.dev, specs) **and** the relevant Frontend Masters course. For performance specifically, you asked for the courses, so they're first-class prerequisites here. The Prep names the course and what understanding it gives you; it only points you to a *specific section* when just one part of a long course is relevant (so you're not watching six hours for one skill).
2. **Do Exercise 1 (guided)** — paste the prompt into Claude Code. It scaffolds a real, broken or unmeasured project, refuses to tell you what's wrong, and makes you measure, diagnose, and fix it — then proves you understood it with a Socratic exam.
3. **Do Exercise 2 (transfer)** — a *different* project exercising the *same* concept, also pasted into Claude Code. Claude scaffolds and **verifies this one too** — same diagnosis check, same exam — but withholds hints harder (you must attempt it solo and report before it helps) and ends with an explicit verdict: *did this concept actually land, or should you redo Exercise 1?* This verdict is the signal you can't get on your own, which is exactly why Exercise 2 is run with Claude, not as unsupervised homework.

The teaching behavior (never reveal the bug, the `hint` ladder, `I give up`, `exam me`, proof-of-understanding) lives in the root `CLAUDE.md`, loaded automatically by Claude Code every session. The performance-specific rules (always measure before AND after; always state numbers; always test throttled; how the two exercises differ) live in `PERF-CONTRACT.md`, also auto-loaded. You don't paste either — they're just there.

---

## THE ONE NON-NEGOTIABLE HABIT

**Measure first. Measure last. Never guess.** Every single performance module follows the same loop, and you will be held to it:

> **Baseline → Hypothesize → Change ONE thing → Re-measure → Compare → Explain the delta.**

A fix you can't measure is a fix you can't prove, and a number you didn't write down is a number you'll misremember. By the end you'll do this reflexively, which is exactly what makes an expert look like they're "guessing" correctly — they're not guessing, they've internalized the loop.

And the two rules every performance teacher repeats, which you will tape to your monitor:

- **Rule 1: Never measure the development build.** Dev builds are slow on purpose (warnings, no minification, source maps). Measure production builds.
- **Rule 2: You are not your user.** You have a fast laptop on fast internet. Your user has a ₹8,000 Android phone on a patchy 4G connection. **Always throttle CPU and network** to simulate them, or your "fast" site is a lie.

---

## HOW TO SET UP (do this once)

One git repo for the whole performance primer. At the root:

- `CLAUDE.md` — the teaching contract (reuse the one from your Frontend Primer; it's identical).
- `PERF-CONTRACT.md` — the performance addendum (provided alongside this file).
- `.gitignore` — ignore `node_modules`.

Each module gets its own folder (e.g. `track-1-measurement/module-1.3-flame-charts/`), and within it, **two subfolders**: `exercise-1/` and `exercise-2/`. Each is its own standalone project. `cd` into the one you're working on and launch Claude Code from inside it — the root `CLAUDE.md` and `PERF-CONTRACT.md` still load via directory walk-up. Commit each finished exercise.

**Tooling you'll install as you go** (the module Prep tells you when): Chrome (your primary instrument — DevTools is the microscope of this entire course), the **Web Vitals Chrome extension**, **Lighthouse** (built into Chrome), and Node.js (for the local servers in the network/caching/SSR modules). Later modules add `web-vitals`, a bundle visualizer, and `lighthouse-ci`.

---

## ABOUT THE FRONTEND MASTERS COURSES (read this once)

You told me you're a theory-lover who wants to become an experimenter. Good — that's the entire design of this primer. The Frontend Masters performance courses are *excellent* and I've made them prerequisites, but here's the deal that makes you an expert instead of a course-completionist:

**Watch the FM course (or the named section), THEN do the exercise without re-watching.** The video gives you the mental model; the exercise forces you to *use* it under your own power, which is the part that sticks. If you can't do the exercise after watching, you didn't really learn it from the video — and that's useful information, not a failure. The Prep cites the **whole course** by default (and what it gives you); it only points to a **specific section** when one narrow skill is all the module needs.

The courses I lean on most, and what each is *for* (so the names stop being a mystery):

- **Web Performance Fundamentals, v2** (Todd Gardner) — the spine of Tracks 1–5. Loading, Core Web Vitals, measurement tools, network, images, caching. If you do one course first, do this.
- **React Performance, v2** (Steve Kinney) — the spine of the React track. Fiber, reconciliation, re-render causes, memoization, virtualization, code-splitting, Suspense/hydration, React 19 compiler.
- **JavaScript Performance** (Steve Kinney) — the bridge from "the network is slow" to "*my code* is slow": the render pipeline (layout/reflow/paint/composite), main-thread work, and a real intro to the V8 engine and deopts.
- **Bare Metal JavaScript: The JS Virtual Machine** — *this is the answer to "why do deep-JS courses matter for performance?"* It teaches what the engine actually does to your code: parsing, bytecode, the optimizing compiler, hidden classes, inline caches, and **deoptimization**. You cannot write "blazingly fast JS" without knowing what makes the engine fast or slow, and this is where that lives. (More on this when we reach Track 3.)
- **Blazingly Fast JavaScript** — applied hot-path optimization: data structures, allocation, avoiding the patterns that make V8 bail out of its fast path.
- **Mastering Chrome Developer Tools, v4** — your instrument-training course. Threaded through every track, because DevTools *is* the lab.

When a module has a matching FM section, it's listed in Prep as a prerequisite. When it doesn't, primary docs carry it.

---

## TRACK MAP

You will likely do these roughly in order; later tracks deliberately require earlier ones, and the *hardest* modules require several tracks at once (a slow real-world page is never slow for one reason).

- **Track 0 — Orientation & Your Instruments** — what performance is, the mental model, and DevTools basics so you can measure anything. *(Start here even if impatient.)*
- **Track 1 — Measurement, Metrics & Methodology** — Core Web Vitals, the Performance APIs, lab vs field, percentiles, Lighthouse, WebPageTest, RUM. You cannot optimize what you cannot measure.
- **Track 2 — The Network & Loading Pipeline** — the waterfall, TTFB, protocols, compression, caching, CDNs, resource hints, critical path.
- **Track 3 — JavaScript & The Main Thread** — the engine (V8, bytecode, JIT, deopts), long tasks, yielding, workers, the cost of JS.
- **Track 4 — Rendering Performance & The Pixel Pipeline** — style/layout/paint/composite, reflow, layout thrashing, the compositor, containment, animations on weak hardware (your "which animation is killing this page" goal lives here).
- **Track 5 — Assets: Images, Fonts & Media** — formats, responsive images, lazy/eager loading, font loading, the biggest real-world wins.
- **Track 6 — React & Next.js Performance at Scale** — Fiber, re-renders, memoization, virtualization, code-splitting, hydration, Suspense, concurrent features, the compiler.
- **Track 7 — Perceived Performance & UX of Speed** — making slow things feel fast.
- **Track 8 — Third-Parties, Monitoring & Performance at Scale** — the third-party tax, facades, RUM dashboards, budgets in CI, proving business impact.
- **Track 9 — Capstones: Diagnose Anything** — multi-cause real-world pages where you must use everything at once. This is where you become the person who reads a flame chart cold.

**The full primer is complete: 10 tracks, 83 modules, 166 hands-on exercises (two per module).** Each track is its own file:

- `perf-track-0-orientation.md` (3 modules)
- `perf-track-1-measurement.md` (9)
- `perf-track-2-network-loading.md` (8)
- `perf-track-3-javascript-main-thread.md` (8)
- `perf-track-4-rendering-pixel-pipeline.md` (9)
- `perf-track-5-assets-images-fonts-media.md` (8)
- `perf-track-6-react-nextjs.md` (14)
- `perf-track-7-perceived-performance.md` (7)
- `perf-track-8-thirdparty-monitoring-scale.md` (10)
- `perf-track-9-capstones-diagnose-anything.md` (7)

Plus this file and `PERF-CONTRACT.md` at your repo root. Work through them in order; Track 9 is the proving ground where you diagnose unfamiliar pages cold.*

---

## PREREQUISITE NOTE (overlap with the general Frontend Primer)

You're doing this performance primer first, so it's built standalone — it re-teaches what it needs. But three concepts are genuinely deep and have a fuller treatment in the general primer. Where a performance module only needs the *performance angle*, it teaches it inline. If you ever want the complete from-scratch version, the cross-reference is noted. You do **not** need to do any general-primer module before starting here. Start at Track 0.

---

---
