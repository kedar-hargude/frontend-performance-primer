# TRACK 0 — ORIENTATION & YOUR INSTRUMENTS

*Before you can fix slow, you must be able to see slow. This track is short and mostly hands-on: install your instruments, learn what the three kinds of performance even are, and take your first measurement. Skip nothing here — every later track assumes you can drive DevTools.*

---

## Module 0.1 — What "Performance" Even Means (The Three Buckets)

**Difficulty:** ●○○○○ · **Format:** Investigation · **Stack:** Any website + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **JavaScript Performance** (Steve Kinney) → *"Thinking About Performance"* — the single clearest framing of the three kinds of performance (network load, JavaScript/compute, rendering). ~14 min. This is the mental map the entire primer hangs on.
- **Web Performance Fundamentals, v2** (Todd Gardner) → *"Introduction"* and *"User Expectations of Performance"* — why speed is a feature, and the human thresholds (100ms feels instant, 1s holds attention, 10s loses them).

**Primary sources:**
- **web.dev — Why does speed matter?** — https://web.dev/why-speed-matters/ — *Focus on: the link between speed, bounce rate, and conversions — performance is a business metric, not vanity.*
- **MDN — Performance (overview)** — https://developer.mozilla.org/en-US/docs/Web/Performance — *Focus on: the high-level map of what affects perceived and actual speed.*

**Search prompts:**
- `"three types of web performance network rendering javascript"`
- `"human perception 100ms 1s 10s response time"`
- Ask an AI: *"Explain the three buckets of web performance — network/loading, JavaScript/main-thread, and rendering — with one concrete symptom a user would feel from each. Then explain the 100ms / 1s / 10s human-perception thresholds and why they set our targets."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow the teaching contract in CLAUDE.md and the performance rules in PERF-CONTRACT.md.
Track 0, Module 0.1, Exercise 1 — the three buckets of performance. I am a near-beginner at
performance. This is an investigation, not a coding task — do not write code for me.

Give me a structured field exercise: pick 3 well-known websites (you name them — one news/media
site, one e-commerce site, one web app). For EACH, walk me through opening Chrome DevTools and
forming a first impression of which of the three performance buckets — network/loading,
JavaScript/main-thread, or rendering — seems to be its biggest cost. Tell me only which panels to
open and what to look at; do NOT tell me what I'll find or what the answer is.

Make me write down, for each site, a one-line guess per bucket and WHY. Then run the Socratic
exam: probe whether I can actually tell a network problem from a main-thread problem from a
rendering problem by their symptoms. Do not accept "it felt slow" — make me tie each symptom to a
bucket and explain the mechanism in both layman's and technical terms.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 0, Module 0.1, Exercise 2 (transfer) — the three
buckets again, new scenario. Apply the Exercise 2 rules: make me attempt the full diagnosis solo
and report before you give any hint, and end with an explicit verdict on whether this concept landed.

Give me 3 NEW sites to investigate (different kinds than Exercise 1 — pick a media-heavy site, a
single-page web app, and a documentation/text site). Have me open DevTools and, for each, classify
which performance bucket dominates and justify it from symptoms — with no guidance on what I'll
find. Then exam me harder than Exercise 1: give me a described symptom ("the page is visible but my
typing lags 2 seconds behind") and make me name the bucket and the mechanism instantly, several
times. At the end, tell me plainly: did this land (proceed), or did I lean on hints / miss exam
questions (redo Exercise 1)?
```

If you want a more personal version, tell Claude to use your own company's app or a client's site as one of the three — diagnosing something you actually own is the best possible transfer test.

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

There is no "bug" here — the lesson is the *taxonomy itself*. Almost every performance problem you'll ever meet is one of three things:

1. **Network/loading** — bytes take too long to arrive (big files, too many requests, slow server, far-away server, no caching). Symptom: a long blank/white period before *anything* shows; a long network waterfall.
2. **JavaScript/main-thread** — the browser is busy *running code* and can't respond. Symptom: the page appears but is frozen — clicks do nothing, typing lags, scroll stutters; long bars in the Performance flame chart.
3. **Rendering** — the browser is busy *drawing* (layout, paint, composite). Symptom: janky animations and scrolling, things visibly re-laying-out; lots of purple "Layout"/green "Paint" in the flame chart.

The expert skill you're starting to build: **map a felt symptom to a bucket in seconds**, because the bucket tells you which tool to reach for next. A white screen → network waterfall. A frozen page → main-thread flame chart. Janky scroll → rendering/paint. Everything in this primer is depth within these three buckets.

The principle: **performance is not one thing; it's three distinct cost centers with three distinct symptoms and three distinct toolsets — and diagnosis starts by naming the bucket.**
</details>

---

## Module 0.2 — DevTools Bootcamp: The Performance Panel

**Difficulty:** ●●○○○ · **Format:** Investigation · **Stack:** Vanilla HTML/JS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Mastering Chrome Developer Tools, v4** (Jon Kuperman) → the **Performance panel** sections — recording a profile, reading the main-thread track, and the flame chart. This is your instrument-training; you'll use this panel in nearly every module.
- **Web Performance Fundamentals, v2** → *"Flame Charts"* and *"Analyzing Results in Performance Tab"* — Todd's walkthrough of reading a flame chart and the waterfall together.

**Primary sources:**
- **Chrome DevTools — Performance panel reference** — https://developer.chrome.com/docs/devtools/performance — *Focus on: recording, the main thread track, and reading the flame chart top-to-bottom (caller → callee).*
- **Chrome DevTools — Performance features reference** — https://developer.chrome.com/docs/devtools/performance/reference — *Focus on: the activity tabs (Summary, Bottom-Up, Call Tree, Event Log).*
- **Chrome DevTools — Analyze runtime performance (tutorial)** — https://developer.chrome.com/docs/devtools/performance/nav — *Do the hands-on walkthrough.*

**Search prompts:**
- `"chrome devtools performance panel flame chart read"`
- `"chrome devtools record performance profile main thread"`
- `"cpu throttling network throttling devtools"`
- Ask an AI: *"Teach me to read a Chrome DevTools Performance recording for the first time: what the main-thread track shows, how to read a flame chart (what width means, what stacking means, caller vs callee), and what 'long task' red triangles mean. How do I enable CPU and network throttling, and why must I?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 0, Module 0.2, Exercise 1 — DevTools Performance
panel bootcamp. I'm new to this panel.

Build me a SINGLE self-contained HTML file with a button that, when clicked, runs a mix of work:
some moderately heavy synchronous JavaScript (a loop doing real computation), a little DOM
updating, and a short animation. The goal is NOT to fix anything yet — it's to give me something
real to RECORD and READ. Make the work substantial enough to show up clearly in a profile.

Tell me only how to run it and the exact steps to record a Performance profile (including turning
on 4x or 6x CPU throttling). Do NOT interpret the recording for me. Then guide me with QUESTIONS to
read my own recording: make me find the long task, identify the widest bar in the flame chart, say
whether time went to scripting/rendering/painting, and read the call tree to find which function
ate the time. Run the exam on whether I can actually navigate a flame chart: width, stacking,
caller/callee direction, and what throttling changed.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 0, Module 0.2, Exercise 2 (transfer) — reading a flame
chart, new scenario. Apply the Exercise 2 rules: I attempt the diagnosis solo and report before you
give any hint; end with an explicit verdict.

Build me a DIFFERENT single HTML file whose button triggers a different mix of work than Exercise 1
— this time make the dominant cost ambiguous between scripting and rendering, so I have to actually
read the chart to tell which it is (don't tell me which it is). Have me record a profile under 4x
CPU throttling and diagnose, solo, where the time goes and which function/activity is responsible.
Then exam me on flame-chart literacy: width = ?, vertical stacking = ?, caller vs callee direction,
what a long task is, and how to tell a scripting bottleneck from a rendering one by color. Verdict
at the end: did flame-chart reading land (proceed), or did I need hints / miss exam questions (redo
Exercise 1)?
```

Bonus once you pass: record the same project throttled and unthrottled and explain to Claude why the two pictures differ — that contrast is the whole point of Rule 2 (you are not your user).

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

Again, no bug — this is instrument training, the most important hour in Track 0. What you should now be able to do:

- **Record** a profile and know that the **main thread** track is where JS execution, layout, and paint all show up serially (because they share one thread).
- **Read a flame chart:** horizontal width = time spent (wider = slower); vertical stacking = call depth (the function on top was *called by* the one beneath it — caller is below, callee is above in Chrome's layout). The widest bar at the bottom is your biggest cost; follow it upward to find the specific function responsible.
- **Spot a long task:** the red-cornered/striped blocks marking tasks over 50ms — these are what block input and cause "frozen page" feelings.
- **Read colors:** yellow = scripting (JS), purple = rendering (style/layout), green = painting/compositing. The dominant color tells you the bucket.
- **Throttle:** 4x–6x CPU and "Slow 4G" network simulate a real cheap phone. The same site can look fine unthrottled and catastrophic throttled — and the throttled view is the truthful one.

The principle: **the Performance panel is your microscope, and the flame chart is how time-on-the-main-thread becomes visible — width is time, stacking is the call path, color is the bucket.** Fluency here is the foundation of every diagnosis you'll ever make; an expert reads a flame chart the way a doctor reads an X-ray.
</details>

---

## Module 0.3 — DevTools Bootcamp: The Network Panel & Waterfall

**Difficulty:** ●●○○○ · **Format:** Investigation · **Stack:** Vanilla HTML + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** → *"Waterfall Charts"* — Todd explains exactly how to read the loading waterfall (what each row and bar segment means). This is the single most important diagnostic skill for loading performance.
- **Mastering Chrome Developer Tools, v4** → the **Network panel** sections — filtering, the timing breakdown, and the waterfall.

**Primary sources:**
- **Chrome DevTools — Network features reference** — https://developer.chrome.com/docs/devtools/network/reference — *Focus on: the Timing tab breakdown (Queued, Stalled, DNS, Initial connection, SSL, TTFB, Content Download) and the waterfall column.*
- **Chrome DevTools — Inspect network activity** — https://developer.chrome.com/docs/devtools/network — *Do the walkthrough. Focus on: reading request order, sizes, and the dependency "staircase."*
- **MDN — Resource Timing** — https://developer.mozilla.org/en-US/docs/Web/API/Performance_API/Resource_timing — *Focus on: the phases of a single request (the same ones the waterfall visualizes).*

**Search prompts:**
- `"chrome devtools network waterfall read timing breakdown"`
- `"network request stalled ttfb content download meaning"`
- `"network waterfall request chain staircase"`
- Ask an AI: *"Teach me to read the Chrome Network panel waterfall: what each phase of a request bar means (queueing, stalled, DNS, connect, SSL, TTFB/waiting, content download), what a 'staircase' request chain indicates, and how to spot render-blocking resources and the largest/slowest requests."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 0, Module 0.3, Exercise 1 — Network panel and the
waterfall. New to this panel.

Build me a small static site (an HTML file plus a few linked CSS/JS files and a couple of images)
served locally, structured so the waterfall is INTERESTING to read: a few resources that load in
sequence, at least one larger asset, and a render-blocking resource — but do NOT fix or optimize
anything, and do NOT tell me which is which. This is reading practice.

Tell me only how to serve it and how to open the Network panel with "Disable cache" on and "Slow
4G" throttling. Then guide me with QUESTIONS to read the waterfall myself: which resource is on the
critical path, which is largest, which blocked rendering, where the "staircase" dependency chain is,
and what each segment of a request's timing bar means. Run the exam on whether I can read a
waterfall: order, size, blocking, and the timing breakdown of a single request.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 0, Module 0.3, Exercise 2 (transfer) — reading a
waterfall, new scenario. Apply the Exercise 2 rules: I attempt solo and report before any hint;
end with a verdict.

Build me a DIFFERENT small static site whose waterfall has a different shape than Exercise 1 — this
time bury the worst problem in a single request's TIMING breakdown (e.g. one resource whose pain is
in TTFB, or in Stalled, rather than in raw size) so I can't diagnose it by size alone and must open
the Timing tab. Don't tell me which request or which phase. Have me serve it, throttle (Slow 4G,
cache disabled), and diagnose solo: largest requests, render-blockers, the staircase, and the one
request whose timing phase is the real problem — naming WHICH phase and what that implies about the
fix. Exam me on the timing breakdown (what each phase means and which fix each points to). Verdict:
did waterfall-reading land, or redo Exercise 1?
```

Bonus once you pass: run the same diagnosis on a content-heavy real site (a news or e-commerce homepage) and narrate the load story to Claude for a sanity check.

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

No bug — waterfall-reading fluency, the counterpart to flame-chart fluency. What you should now be able to do:

- **Read the waterfall as a timeline:** each row is one request; time flows left to right; the bar's position shows when it started and how long each phase took.
- **Read one request's timing breakdown:** *Queued/Stalled* (waiting for a connection or priority), *DNS* (looking up the server), *Initial connection + SSL* (the handshake), *Waiting/TTFB* (server thinking), *Content Download* (bytes arriving). A request that's slow because of TTFB is a *server* problem; one slow because of Content Download is a *too-big* problem; one slow because of Stalled is a *too-many-connections/priority* problem. Same symptom, totally different fixes — and the breakdown tells you which.
- **Spot the critical path:** the chain of requests that must complete before the user sees meaningful content.
- **Spot render-blocking resources:** CSS and synchronous JS in the `<head>` that hold up the first paint.
- **Spot request chains (the "staircase"):** when resource B can't start until resource A is parsed (e.g. a font referenced by a CSS file referenced by the HTML) — each step adds a full round-trip of delay.

The principle: **the network waterfall turns "the page loaded slowly" into a precise, phase-by-phase account of *why* — which request, which phase, and what depended on what.** Combined with the flame chart from 0.2, you now have eyes on two of the three buckets; an expert glances at a waterfall and immediately knows whether the problem is the server, the byte size, the connection count, or a dependency chain.
</details>

---
---


---

*End of Track 0 — Orientation & Your Instruments. 3 modules. You can now drive the Performance panel and the Network panel — every later track assumes this.*

*Next: Track 1 — Measurement, Metrics & Methodology.*
