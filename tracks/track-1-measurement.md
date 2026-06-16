# TRACK 1 — MEASUREMENT, METRICS & METHODOLOGY

*You cannot optimize what you cannot measure, and you cannot measure what you cannot define. This track makes you fluent in the numbers that matter — Core Web Vitals and the metrics around them — and, crucially, in the methodology of measuring honestly: lab vs field, percentiles vs averages, and the discipline that separates real performance work from cargo-cult tweaking. By the end you can put a number on any user experience and defend it.*

---

## Module 1.1 — The Core Web Vitals: LCP, INP, CLS

**Difficulty:** ●●○○○ · **Format:** Investigation + broken project · **Stack:** Vanilla HTML/CSS/JS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Core Web Vitals & Other Performance Metrics"** section. This is the definitive walkthrough — Todd builds each metric (LCP, CLS, INP, plus FID/TTFB/FCP) up from what the user actually feels and shows the math. Watch the whole section before Exercise 1; it's the single best explanation of these metrics anywhere.

**Primary sources:**
- **web.dev — Web Vitals** — https://web.dev/articles/vitals — *Focus on: the three Core Web Vitals, their "good/needs-improvement/poor" thresholds, and that they're measured at the 75th percentile.*
- **web.dev — Largest Contentful Paint (LCP)** — https://web.dev/articles/lcp — *Focus on: what element qualifies as the LCP, and the four sub-parts (TTFB, resource load delay, resource load time, render delay).*
- **web.dev — Interaction to Next Paint (INP)** — https://web.dev/articles/inp — *Focus on: the three parts of an interaction (input delay, processing time, presentation delay) and why INP replaced FID.*
- **web.dev — Cumulative Layout Shift (CLS)** — https://web.dev/articles/cls — *Focus on: impact fraction × distance fraction, and the session-window definition.*

**Search prompts:**
- `"core web vitals lcp inp cls thresholds 75th percentile"`
- `"lcp sub-parts ttfb load delay render delay"`
- `"inp input delay processing presentation"`
- Ask an AI: *"Explain each Core Web Vital precisely: LCP (and its four sub-parts), INP (and its three parts, and why it replaced FID), and CLS (and the impact × distance math). For each, give the good/poor thresholds, what a user feels when it's bad, and the single most common cause."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.1, Exercise 1 — measuring the three Core
Web Vitals. Completed: Track 0.

Build me a single realistic landing page (HTML/CSS/JS, served locally) that scores measurably
BADLY on all three Core Web Vitals — a slow LCP, a janky high INP on its main button, and visible
CLS as things load — but do NOT tell me which code causes which metric, or even which metrics are
bad. Make the symptoms real and reproducible under throttling.

Tell me only how to run it and how to measure CWV three ways: the Web Vitals extension, a Lighthouse
run (with throttling), and reading them in the Performance panel. Do NOT interpret the numbers.
Make me, for each metric: get a baseline NUMBER, identify which element/interaction is responsible
(e.g. WHICH element is the LCP, WHICH interaction blew INP, WHICH shift caused CLS), and only then
hypothesize the cause. Hold me to the measure→hypothesize→fix→re-measure loop, one metric at a
time. Exam me on what each metric measures, its parts, and what a user feels when it's bad.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.1, Exercise 2 (transfer) — Core Web Vitals,
inverted. Apply the Exercise 2 rules: I work solo and report before any hint; end with a verdict.

This time invert the task to prove I understand what DRIVES each metric. Scaffold me a CLEAN, fast
page that scores well on all three CWV. My job, solo: predict — before measuring — exactly how each
metric will move if I (a) swap the hero for a giant unoptimized image, (b) add a heavy synchronous
handler to the main button, (c) inject a late banner with no reserved space. Then make each change
ONE at a time, measure before/after, and confirm whether my prediction was right and by roughly how
much. Do not tell me the answers; check my predictions against my measurements with me. Exam me on
which sub-part of each metric each change hit (e.g. which part of LCP the image hurt). Verdict: do I
understand CWV drivers (proceed), or were my predictions off (redo Exercise 1)?
```

The inversion (predict → break → measure the damage) is a harder, cleaner test than fixing — if you can predict each metric's movement in advance and the numbers confirm it, you genuinely understand the vitals.

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

The page has one cause per metric:

- **Bad LCP:** the largest element (a hero image or headline) renders late — typically a big unoptimized image that loads slowly, possibly behind render-blocking resources. You should identify the LCP *element* in DevTools (it's labelled), get its time, and see which of the four sub-parts dominates (here, resource load time).
- **Bad INP:** the main button's click handler does heavy synchronous work, so the next paint after the click is delayed. You should reproduce the interaction, measure INP, and see the long task in the flame chart blocking the paint.
- **Bad CLS:** content loads without reserved space (an image with no width/height, or a banner injected above existing content), so the page visibly jumps. You should identify *which* element shifted and what it pushed.

The expert skills: (1) **attribution** — not just "LCP is bad" but "*this specific element* is the LCP and *this sub-part* is why"; (2) measuring the **same** metric three different ways and understanding why the numbers differ slightly (extension vs Lighthouse lab vs Performance panel); (3) the discipline of one-metric-at-a-time.

The principle: **Core Web Vitals quantify what users actually feel — how fast the main thing appears (LCP), how fast the page answers a tap (INP), and how much it jumps around (CLS) — and each decomposes into parts that point straight at the cause.** Naming the responsible element/interaction is the move that turns a score into a fix.
</details>

---

## Module 1.2 — The Performance API: Measuring From Code

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** Vanilla JS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** → the **"Capturing Performance Metrics"** section: *"Performance API"*, *"Performance Observer"*, *"Browser Support for Performance Metrics"*. Todd shows exactly how to read these metrics programmatically and the "observer effect" gotcha.

**Primary sources:**
- **MDN — Performance API** — https://developer.mozilla.org/en-US/docs/Web/API/Performance_API — *Focus on: `performance.now()`, the timeline, and `mark()`/`measure()` for custom timings.*
- **MDN — PerformanceObserver** — https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver — *Focus on: subscribing to entry types (paint, LCP, layout-shift, event, longtask) and the `buffered` flag.*
- **MDN — Navigation & Resource Timing** — https://developer.mozilla.org/en-US/docs/Web/API/PerformanceNavigationTiming — *Focus on: the timing phases available per navigation and per resource.*
- **web.dev — The `web-vitals` library** — https://github.com/GoogleChrome/web-vitals — *Focus on: how it wraps PerformanceObserver to report real CWV.*

**Search prompts:**
- `"performanceobserver lcp cls inp entry types"`
- `"performance mark measure custom metrics"`
- `"web-vitals library onLCP onINP onCLS"`
- Ask an AI: *"Teach me to measure performance from JavaScript: `performance.mark`/`measure` for custom timings, and `PerformanceObserver` to capture paint, LCP, layout-shift, longtask, and event entries. Why is the `buffered: true` flag important, and what's the 'observer effect'? How does the web-vitals library use these to report field CWV?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.2, Exercise 1 — measuring from code with
the Performance API, build challenge. Completed: Track 0, Module 1.1.

Give me a spec (no solution code) to instrument a small existing app so it MEASURES itself: capture
LCP, CLS, and INP via PerformanceObserver and log them; add custom `performance.mark`/`measure`
timings around a specific expensive operation (e.g. "time from click to data rendered"); and read
Navigation Timing to report TTFB and FCP. Provide the uninstrumented app; I write all the
measurement code.

Review me like a senior. If I forget `buffered: true` and miss the LCP that fired before my observer
attached, do not tell me — ask me why my LCP is null when the page clearly painted. If my custom
measure spans the wrong two points, ask me what exactly I'm timing. Make me explain what each
observer entry type reports and why measuring from code (field-capable) differs from a one-off
Lighthouse run (lab).
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.2, Exercise 2 (transfer) — instrumenting
from code, new app. Apply the Exercise 2 rules: I attempt solo and report before any hint; end with
a verdict.

Give me a DIFFERENT uninstrumented app and have me, solo, instrument it to report all three CWV via
PerformanceObserver plus one custom `mark`/`measure` around a meaningful operation, logging results
on unload. Then have me install the real `web-vitals` library and compare my hand-rolled numbers to
its output. Here's the verification hook: if my numbers DON'T match, do not tell me why — ask me
which entry type or which flag could cause the gap, and make me find it (it's almost always missing
`buffered: true` or the wrong entry type). Exam me on what each observer reports and why code-based
measurement is what enables field/RUM data. Verdict: do my observers match the library and do I
understand why (proceed), or did I need significant help (redo Exercise 1)?
```

Matching the battle-tested `web-vitals` library is an objective pass/fail signal — either your observers are correct or they aren't, and Claude can verify it with you.

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- A `PerformanceObserver` per relevant entry type — `largest-contentful-paint`, `layout-shift` (accumulated into CLS with the session-window logic), `event`/`first-input` (for INP-style latency), and `longtask` — each created with `{ buffered: true }` so entries that fired *before* the observer attached are still delivered (the #1 bug: a null/missing LCP because the observer was too late and didn't buffer).
- Custom timing with `performance.mark('start')` / `performance.mark('end')` / `performance.measure('op', 'start', 'end')`, spanning exactly the operation of interest (a common error is bracketing the wrong region, so the number is meaningless).
- Navigation Timing for TTFB (`responseStart - requestStart`) and FCP via the paint observer.
- Understanding the "observer effect": measuring costs a little, and heavy logging can itself affect what you measure.
- Why this matters: code-based measurement is what powers **field/RUM** data (real users), as opposed to a single lab run. The web-vitals library is just a robust wrapper over exactly these observers.

The principle: **the Performance API and PerformanceObserver let your code measure the real experience as it happens — which is the foundation of field measurement and custom metrics — but only if you capture buffered entries and bracket the right spans.** This is how you measure things Lighthouse can't see: *your* app's specific key moments.
</details>

---

## Module 1.3 — Lab Data vs Field Data (and the CrUX Report)

**Difficulty:** ●●●○○ · **Format:** Investigation · **Stack:** Any site + Chrome + PageSpeed Insights

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Testing & Tools"** section, especially the parts on lab vs synthetic vs field data, the **Chrome User Experience Report (CrUX)**, and PageSpeed Insights. Todd hammers the single most important methodology point in the whole course: *your lab number and your users' real number are different, and the field number is the one that counts.*

**Primary sources:**
- **web.dev — Lab and field data** — https://web.dev/articles/lab-and-field-data-differences — *Focus on: why a fast lab score can hide a slow real-user experience, and vice versa.*
- **CrUX overview** — https://developer.chrome.com/docs/crux — *Focus on: what CrUX is (real Chrome users' field data), its 28-day rolling 75th-percentile window, and that it's what Google ranks on.*
- **PageSpeed Insights** — https://pagespeed.web.dev — *Run your own site through it. Focus on: the split between "Discover what your real users are experiencing" (field/CrUX) and "Diagnose performance issues" (lab/Lighthouse).*

**Search prompts:**
- `"lab data vs field data web performance difference"`
- `"chrome ux report crux 75th percentile 28 days"`
- `"why lighthouse score differs from real users"`
- Ask an AI: *"Explain the difference between lab data (a controlled synthetic test like Lighthouse) and field data (real users, like CrUX). Why can they disagree wildly? What is the Chrome User Experience Report, what window and percentile does it use, and why is field data the source of truth for SEO and real UX?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.3, Exercise 1 — lab vs field data.
Completed: Track 0, Modules 1.1–1.2. This is an investigation; do not write app code.

Guide me through a structured comparison on 3–4 REAL sites (you pick a mix: one big popular site
with lots of traffic, one smaller/niche site, and ideally one I can run myself). For each: have me
run PageSpeed Insights and record BOTH the field data (CrUX, if available) and the lab data
(Lighthouse) for the three CWV side by side. Do NOT interpret the gaps for me. Make me find a site
where lab and field DISAGREE and form a hypothesis about why (e.g. lab uses one throttled cold
load; field is millions of real warm/cold loads across devices).

Run the exam: make me explain what CrUX actually measures, why a site might have "no field data,"
why lab and field diverge, and which one I'd trust for a "is my site fast for real users?"
decision. Demand the layman + technical explanation of each.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.3, Exercise 2 (transfer) — lab vs field,
new investigation. Apply the Exercise 2 rules: I work solo and report before any hint; end with a
verdict.

Give me a scenario: a teammate says "Lighthouse gives us 98, we're fast, ship it." Have me, solo,
argue back using lab-vs-field reasoning — what could be true that makes 98-in-lab compatible with a
slow real-user experience? Then have me actually find a real site whose lab score and CrUX field
data tell different stories and present the case. Exam me on: the 75th-percentile choice (why P75,
not average or median?), why a brand-new page has no CrUX data, and when a GOOD lab score still
warrants worry. Verdict: do I understand lab vs field well enough to push back on a "Lighthouse says
98" argument in an interview (proceed), or did this wobble (redo Exercise 1)?
```

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

No bug — this is the methodology backbone of credible performance work. What you should now hold:

- **Lab data** = one controlled, synthetic load (Lighthouse), same conditions every time. Great for *debugging* and reproducibility, because you can change one thing and re-measure. But it's ONE simulated device/network, not your real audience.
- **Field data** = real users' real loads, aggregated (CrUX). It's the truth about experienced performance, captured at the **75th percentile over a 28-day rolling window**, and it's what Google's ranking uses. But you can't "debug" it directly — it's a lagging aggregate.
- **Why they disagree:** lab is one cold load on a simulated mid-tier phone; field spans flagships and ₹6,000 Androids, warm and cold caches, fast and patchy networks, all percentiles collapsed to P75. A site can ace lab and still fail field (or the reverse) because they measure different things.
- **Why P75:** the average hides your worst-served users; P75 means "the experience that 75% of users got *at least* this good" — it deliberately weights toward the slower quarter, because those are the users you're losing.
- **"No field data":** low-traffic pages don't have enough real samples to report in CrUX.

The expert move, and a killer interview line: **use lab data to diagnose and iterate, but judge success on field data — because you are optimizing for real users at P75, not for your laptop in a quiet room.**
</details>

---

## Module 1.4 — Percentiles vs Averages: Why the Mean Lies

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** Vanilla JS (small data exercise)

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Statistics"** lesson in the Testing & Tools section. Todd shows exactly why averaging performance data is misleading and why percentiles (P75, P95, P99) tell the real story. Short and essential.

**Primary sources:**
- **web.dev — Percentiles in performance** (concept) — search the term; *Focus on: why a single slow tail can be invisible in the mean.*
- **MDN — statistics primer / any percentile explainer** — *Focus on: what "P95 = 4s" actually means (95% of samples were ≤ 4s; the worst 5% were worse).*
- **Catchpoint / WebPageTest blogs on percentiles** — *Focus on: long-tail latency and why averages hide it.*

**Search prompts:**
- `"why averages mislead web performance percentiles p95 p99"`
- `"long tail latency performance percentile"`
- `"median vs mean vs p95 latency explained"`
- Ask an AI: *"Explain why the AVERAGE (mean) is a bad way to summarize performance data, using a concrete distribution. What do P50/P75/P95/P99 mean, why does a bimodal distribution (fast cache hits + slow cache misses) make the mean meaningless, and why do we optimize the tail?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.4, Exercise 1 — percentiles vs averages,
build challenge. Completed: Track 0, Modules 1.1–1.3.

Give me a spec (no solution code) to build a tiny tool: feed it an array of "page load times"
(you provide a few realistic datasets — one tight, one with a long slow tail, one bimodal like
fast-cache-hits-plus-slow-misses), and have me compute and display the mean, median (P50), P75,
P95, and P99, plus a simple histogram (text or basic bars). I write all the stats code.

Review me like a senior. Before I code, make me predict, for the bimodal dataset, how the mean and
the P95 will differ and which one "lies." If my percentile calculation is off (off-by-one in the
index, not sorting, wrong interpolation), do not fix it — ask me what P95 means in plain words and
make me verify by hand on a tiny array. Exam me on: why the mean hid the slow users, what P75/P95/
P99 each tell me, and which I'd report to a stakeholder and why.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.4, Exercise 2 (transfer) — percentiles,
applied to a real decision. Apply the Exercise 2 rules: solo attempt first; verdict at the end.

Give me a NEW dataset and a scenario: "Our average load time is 1.2s — great, right?" Have me,
solo, compute the full percentile spread, discover the hidden slow tail (you design the data so the
mean looks fine but P95/P99 are alarming), and write the one-paragraph argument I'd give the team
about why the average is hiding a real problem and what to actually target. Exam me on: why we
usually optimize toward a percentile target (e.g. "P75 LCP under 2.5s") rather than an average, and
what a gap between P75 and P95 implies about who's being underserved. Verdict: can I reason in
percentiles like a performance lead (proceed), or did the stats trip me up (redo Exercise 1)?
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Correct percentile computation: sort the array, then index correctly (and know whether you're using nearest-rank or interpolation — be consistent and able to explain it). The classic bugs are forgetting to sort and off-by-one indexing.
- A histogram that makes the *shape* visible — especially the bimodal case where two humps (cache hits near 200ms, misses near 3s) average out to a middle value that *no actual user experienced*.
- The core realization: in the bimodal/long-tail data, the **mean is a number nobody got**. The median might look fine while P95 is terrible. The slow tail is real users — often on bad networks or cheap devices — and they're the ones who bounce.
- Why we target percentiles: "P75 LCP < 2.5s" is a promise about the *experience most users get*, deliberately accounting for the slower portion. An average target lets you "fix" the number by making fast users faster while ignoring the suffering tail.
- The P75↔P95 gap as a diagnostic: a big gap means a meaningful chunk of users have a much worse experience than your "typical" user — a signal to investigate device/network segments.

The principle: **performance data is a distribution, not a number, and the mean systematically hides the slow tail where your real losses are — so experts think and target in percentiles (P75/P95/P99), because that's where the underserved users live.**
</details>

---

## Module 1.5 — Lighthouse Deep-Dive: How the Score Actually Works

**Difficulty:** ●●●○○ · **Format:** Investigation + broken project · **Stack:** Any built site + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Google Lighthouse"** and **"Analyzing Results in Performance Tab"** lessons. Todd shows how to run Lighthouse, simulate real conditions (screen size, network throttling, CPU throttling), and read the report and its waterfall/flame output.
- **Mastering Chrome Developer Tools, v4** → the Lighthouse panel coverage, for the DevTools-integrated workflow.

**Primary sources:**
- **Lighthouse — Performance scoring** — https://developer.chrome.com/docs/lighthouse/performance/performance-scoring — *Focus on: the weighted metrics (LCP, TBT, CLS, FCP, Speed Index) and how the 0–100 score is computed.*
- **Lighthouse — Throttling** — https://developer.chrome.com/docs/lighthouse/overview/#throttling — *Focus on: simulated vs applied throttling, and why the default simulates a mid-tier mobile on slow 4G.*
- **web.dev — Total Blocking Time (TBT)** — https://web.dev/articles/tbt — *Focus on: TBT as the lab proxy for INP (since INP needs real interaction).*

**Search prompts:**
- `"lighthouse performance score weights lcp tbt cls"`
- `"lighthouse simulated throttling vs applied"`
- `"total blocking time tbt lab proxy for inp"`
- Ask an AI: *"How does Lighthouse compute its 0–100 performance score? What metrics does it weight and by how much? What is Total Blocking Time and why does Lighthouse use it as a lab stand-in for INP? Explain simulated vs applied throttling, and why two runs of the same page can score differently."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.5, Exercise 1 — Lighthouse internals.
Completed: Track 0, Modules 1.1–1.4.

Build me a small site (served locally, PRODUCTION build — remind me why dev builds don't count)
with a deliberately mixed performance profile: decent on some metrics, poor on others, so the
Lighthouse report has a spread to interpret. Do NOT tell me which metrics are bad or why.

Tell me only how to run Lighthouse correctly (mobile preset, throttling on, production build,
incognito/no extensions). Then guide me with QUESTIONS: make me read which metrics are dragging the
score, look up their weights, and figure out which ONE improvement would move the score the most
(the highest-weighted failing metric). Have me also run Lighthouse 3 times and note the variance.
Exam me on: the metric weights, what TBT is and why it's the INP proxy in lab, simulated vs applied
throttling, and why I should never trust a single run.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.5, Exercise 2 (transfer) — Lighthouse,
score-driven prioritization. Apply the Exercise 2 rules: solo first; verdict at the end.

Give me a DIFFERENT site with a low Lighthouse score and a scenario: "We have time to fix exactly
ONE thing this sprint to raise the score the most." Have me, solo, run Lighthouse correctly,
identify from the weights + the failing metrics which single fix has the highest score leverage,
make that one change, and re-measure to confirm the score moved as predicted. Don't tell me which
metric to chase — make me derive it from the weighting. Exam me on: why optimizing the highest-
weighted failing metric is the rational move, why the score is NOT the real goal (field UX is), and
the run-to-run variance trap. Verdict: can I use Lighthouse as a prioritization tool rather than a
vanity number (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The Lighthouse performance score is a **weighted blend**, not a single metric. The heavily-weighted contributors are **LCP**, **Total Blocking Time (TBT)**, and **CLS**, with **FCP** and **Speed Index** contributing less. (Exact weights shift between Lighthouse versions — the skill is looking them up, not memorizing.)
- This means the rational way to raise a score is to **fix the highest-weighted FAILING metric first** — improving an already-good low-weight metric barely moves the needle. The planted site is built so one specific metric (e.g. a bad LCP or a high TBT from a long task) is both failing and high-weight; that's the leverage point.
- **TBT is the lab proxy for INP.** INP needs a real user interaction, which a synthetic lab run can't perform reliably, so Lighthouse measures TBT (total main-thread blocking time over 50ms during load) as a stand-in for "will this page respond slowly to input."
- **Throttling:** the default mobile run *simulates* a mid-tier phone on slow 4G (simulated throttling estimates timings; applied throttling actually slows the connection). Either way: the score reflects a deliberately handicapped device, which is the point.
- **Variance:** the same page scores differently run to run (system load, network simulation noise). One run is noise; run several and look at the spread (ties back to Module 1.4).

The principle: **the Lighthouse score is a weighted lab estimate, useful as a diagnostic and a prioritization tool — fix the highest-weighted failing metric for maximum leverage — but it is not the goal; real-user field data is.** Treating the number as the objective is how people "optimize" their way to a great score and a still-slow site.
</details>

---

## Module 1.6 — WebPageTest & Filmstrips: Seeing the Load Frame by Frame

**Difficulty:** ●●●○○ · **Format:** Investigation · **Stack:** Any public site + WebPageTest.org

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Using WebPageTest.org"** lesson. Todd shows how WPT gives you control lab tools don't: test from specific locations, real devices, specific connection types, and the filmstrip/video of the actual load.

**Primary sources:**
- **WebPageTest — documentation** — https://docs.webpagetest.org — *Focus on: running a test, the waterfall, the filmstrip view, Speed Index, and the "first/repeat view" distinction.*
- **web.dev — Speed Index** — search the term — *Focus on: what Speed Index measures (how quickly content is visually populated) and why the filmstrip makes it intuitive.*
- **WebPageTest — Connection/location settings** — *Focus on: testing as a real user in another region on a real slow connection — something Lighthouse can't do.*

**Search prompts:**
- `"webpagetest filmstrip speed index read"`
- `"webpagetest first view vs repeat view caching"`
- `"webpagetest test from location connection real device"`
- Ask an AI: *"Teach me to use WebPageTest.org: how to run a test, read the waterfall and the FILMSTRIP (frame-by-frame load), what Speed Index measures, and the difference between first view and repeat view (cache). What can WebPageTest show me that Lighthouse and the local DevTools can't?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.6, Exercise 1 — WebPageTest and filmstrips.
Completed: Track 0, Modules 1.1–1.5. Investigation; no app code.

Guide me through running WebPageTest on 2–3 REAL public sites (you pick a mix; include one likely
to be slow on a throttled mobile connection). For each, have me run a test from a specific location
on a slow connection, then READ: the filmstrip (when does meaningful content actually appear?), the
waterfall, the Speed Index, and the first-view vs repeat-view difference. Do NOT interpret the
results for me — ask me questions that make me narrate the load story from the filmstrip frames.

Exam me on: what Speed Index captures that LCP alone doesn't, what the first-view/repeat-view gap
tells me about caching, and why testing from a far-away location on a real slow connection reveals
things my fast local machine hides. Demand the layman + technical explanation.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.6, Exercise 2 (transfer) — WebPageTest,
comparative. Apply the Exercise 2 rules: solo first; verdict.

Give me a task: pick a site and a direct competitor (you can suggest a pair), run BOTH through
WebPageTest on the same connection/location, and produce — solo — a side-by-side verdict from the
filmstrips and Speed Index about which one *feels* faster to load and exactly why (which frames
show content sooner, which has the worse repeat-view caching, etc.). This is the literal skill of
"look at a few things and diagnose." Exam me on reading a filmstrip and on why Speed Index can rank
two pages differently than a single milestone metric would. Verdict: can I run and read a WPT
comparison cold (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

No bug — this is a tool you'll use for the rest of your career, and it directly trains your end goal (look at a load and diagnose it). What you should now be able to do:

- **Read a filmstrip:** WebPageTest captures the actual rendering frame by frame, so you can *see* exactly when the page goes from blank → first content → meaningful content → fully loaded. This makes "it feels slow" concrete: you point at the frame where the user is still staring at white.
- **Read Speed Index:** roughly, "how quickly the visible area gets populated" — a lower number means content appeared progressively and early. It captures *visual completeness over time*, which a single milestone like LCP can miss (two pages with the same LCP can feel very different if one painted progressively and the other flashed everything at once late).
- **First view vs repeat view:** first view = cold cache (new visitor); repeat view = warm cache (returning visitor). A big gap means caching is doing a lot of work (good for returns) — or that your first-visit experience is the one to fix.
- **Test from real conditions:** WPT runs on real devices from real locations on real (or accurately throttled) connections — so you can test "what does a user in another region on 4G actually experience," which your fast local machine and even Lighthouse's simulation approximate less faithfully.

The principle: **WebPageTest turns a page load into a frame-by-frame film you can watch and compare, from conditions that match real users — making "which one feels faster and why" an observation, not a guess.** This is the closest tool to your stated goal of glancing at a load and diagnosing it.
</details>

---

## Module 1.7 — Build a Tiny RUM Beacon: Measuring Real Users

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** Vanilla JS + a small Node server

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Real User Monitoring"** and **"Request Metrics Monitoring Tool"** lessons. Todd explains why RUM (collecting metrics from actual users) beats lab data for knowing the truth, and what a RUM pipeline looks like.

**Primary sources:**
- **web.dev — The `web-vitals` library (attribution build)** — https://github.com/GoogleChrome/web-vitals#send-the-results-to-an-analytics-endpoint — *Focus on: collecting CWV in the field and beaconing them to a server.*
- **MDN — `navigator.sendBeacon`** — https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon — *Focus on: reliably sending data on page unload without blocking navigation.*
- **MDN — `visibilitychange` / page lifecycle** — https://developer.mozilla.org/en-US/docs/Web/API/Document/visibilitychange_event — *Focus on: the correct moment to flush metrics (don't rely on `unload`).*

**Search prompts:**
- `"navigator.sendBeacon report metrics page unload"`
- `"web-vitals send to analytics endpoint rum"`
- `"visibilitychange flush metrics reliable"`
- Ask an AI: *"How do I build a minimal Real User Monitoring pipeline: collect the three CWV with the web-vitals library, and reliably send them to a server endpoint when the user leaves, using `navigator.sendBeacon` and the `visibilitychange` event (not `unload`). Why is sendBeacon the right tool, and why is `unload` unreliable, especially on mobile?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.7, Exercise 1 — build a tiny RUM beacon,
build challenge. Completed: Track 0, Modules 1.1–1.6. A small local Node server is allowed here.

Give me a spec (no solution code): a small page that collects the three CWV (use the web-vitals
library), batches them, and beacons them to a minimal Node endpoint when the user leaves — using
`navigator.sendBeacon` triggered on `visibilitychange`→hidden, NOT on `unload`. The server logs
each payload. I write all of it (client collection + the tiny server).

Review me like a senior. If I flush on `unload`, do not tell me it's wrong — ask me what happens on
mobile when the user switches apps and the page is frozen/killed without firing unload. If I send
a metric before it's finalized (e.g. CLS keeps accumulating), ask me when each metric is actually
"done." If sendBeacon silently fails, ask me about payload size and the beacon's constraints. Exam
me on why RUM beats lab for ground truth, and why the unload/visibility distinction matters.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.7, Exercise 2 (transfer) — extend the RUM
beacon. Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT starter and have me, solo, build a slightly richer RUM beacon: collect the CWV
PLUS one custom timing and a couple of context fields (connection type via the Network Information
API, and whether it was a first or repeat visit), beacon them reliably, and have the server compute
and print the P75 of each metric across all received samples (tying back to Module 1.4 percentiles).
Don't hand me the lifecycle code — make me get the reliable-flush right myself. Exam me on: why P75
on the server mirrors what CrUX does, why context fields (connection, device) matter for segmenting,
and the sendBeacon/visibilitychange correctness points. Verdict: could I stand up a real basic RUM
pipeline at work (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Client: use `web-vitals` (`onLCP`, `onINP`, `onCLS`) to collect finalized metrics; **flush on `visibilitychange` → `hidden`** (and/or `pagehide`), **not** `unload`. On mobile, when a user switches apps, the page is often frozen then discarded *without* ever firing `unload`, so unload-based reporting silently loses a huge fraction of real (often the worst) sessions. `visibilitychange→hidden` fires reliably.
- Use **`navigator.sendBeacon`** (or `fetch` with `keepalive`) so the request survives the page going away without blocking navigation. Know the constraints: beacons are small-payload, POST, fire-and-forget (no response handling), and there are size limits — so batch compactly.
- Timing of finalization: CLS accumulates over the page's life and INP isn't known until interactions happen — so you report them at the end (on hidden), not eagerly. Reporting a metric "too early" gives a wrong value.
- Server: a minimal endpoint that accepts the beacon and (Exercise 2) computes P75 across samples — which is exactly the shape of what CrUX/commercial RUM do, just smaller.
- Context fields (connection type, first/repeat visit, device) are what let you *segment* — "our P75 LCP is fine overall but terrible for users on slow connections" is the kind of insight that only field data with context can give.

The principle: **RUM measures the truth — what real users on real devices actually experienced — by collecting finalized metrics in the browser and reliably beaconing them out at page-hide, then aggregating at a percentile.** Lab data tells you what *might* happen; RUM tells you what *did*, and the reliability details (visibilitychange, sendBeacon, finalization timing) are what separate a real pipeline from one that quietly drops your most important sessions.
</details>

---

## Module 1.8 — Run-to-Run Variance & Statistical Rigor

**Difficulty:** ●●●○○ · **Format:** Investigation · **Stack:** Any site + Chrome/Lighthouse

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → revisit **"Statistics"** and **"Testing Performance"** with variance in mind — the lab-vs-field and percentile lessons underpin this. (No single dedicated lesson; this module operationalizes those into a discipline.)

**Primary sources:**
- **web.dev — Variability in performance measurement** — search "lighthouse variability" → https://developer.chrome.com/docs/lighthouse/overview/#variability — *Focus on: the sources of run-to-run noise (CPU contention, network, A/B content, client variance) and how to reduce it.*
- **web.dev — How to measure reliably** — *Focus on: multiple runs, median of runs, controlling the environment.*

**Search prompts:**
- `"lighthouse run to run variability reduce"`
- `"how many runs performance measurement reliable median"`
- `"sources of noise performance benchmarking"`
- Ask an AI: *"Why does measuring the same page twice give different performance numbers? List the sources of run-to-run variance (CPU contention, network noise, extensions, A/B/personalized content, thermal throttling). How do I measure reliably — how many runs, median vs mean of runs, and how to control the environment — so my 'before vs after' comparison is trustworthy?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.8, Exercise 1 — run-to-run variance.
Completed: Track 0, Modules 1.1–1.7. Investigation.

Guide me through an experiment on ONE page: run Lighthouse (or a Performance trace) the WRONG way
first — once, with extensions on, other tabs/apps running, on a dev build — and record the number.
Then have me run it the RIGHT way — production build, incognito, quiet machine, throttled,
MULTIPLE runs — and record the spread. Do NOT tell me the answer; make me observe how much the
number jumps and form a rule for how many runs I need and which summary (median of runs) to trust.

Exam me on: the sources of variance, why a single run can't validate a fix (the change might be
smaller than the noise), why median-of-runs beats one run, and how I'd design a trustworthy
before/after comparison. Demand the layman + technical explanation.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.8, Exercise 2 (transfer) — variance in a
real before/after. Apply the Exercise 2 rules: solo first; verdict.

Give me a page with a small genuine optimization I can apply. Have me, solo, design and run a
trustworthy A/B measurement: baseline with N runs (I decide N and justify it), apply the one change,
re-measure with N runs, and decide whether the improvement is REAL or within the noise band. You
provide the page but do NOT tell me whether the change actually helps — make me prove it
statistically (is the median delta bigger than the run-to-run spread?). Exam me on: why "it went
from 1.4s to 1.3s on one run" is not evidence, and how I'd report a result honestly. Verdict: can I
run a noise-aware before/after that I'd defend to a skeptical senior (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

No bug — this is the discipline that makes every other measurement in the course trustworthy. What you should now hold:

- **Every measurement is noisy.** The same page, measured twice, differs — because of CPU contention from other processes, network variability, browser extensions, thermal throttling, and (on real sites) A/B tests and personalized content. The noise is often **larger than the improvement you're trying to detect**.
- **One run proves nothing.** "It went from 1.4s to 1.3s" after your change might be pure noise — the next run might be 1.5s. A single-run before/after is the most common way developers fool themselves into shipping placebo fixes (or discarding real ones).
- **Reduce noise:** production build (Rule 1), incognito/no extensions, quiet machine (close other apps/tabs), consistent throttling, same network conditions.
- **Then run multiple times and take the median** of the runs (the median is robust to the occasional outlier run). Compare the *median delta* against the *spread* — if your improvement is bigger than the run-to-run variance, it's likely real; if it's inside the noise band, you haven't shown anything.
- **How many runs:** enough that the median stabilizes — often 5–10 for Lighthouse; more if the page is noisy.

The principle: **a performance measurement is a noisy sample, not a fact, so a credible before/after requires a controlled environment, multiple runs, a robust summary (median of runs), and a delta that exceeds the noise.** This is what lets you say "this fix works" and mean it — and it's exactly the rigor an interviewer probes when they ask "how did you know your optimization actually helped?"
</details>

---

## Module 1.9 — Setting Performance Goals & Budgets

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** Any built site + Lighthouse/bundle tooling

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Setting Performance Goals"** section: *"How Fast Should Your Site Be?"*, *"Determining Performance Goals"*, *"Understanding Your Users"*. Todd's framework: goals come from your users, your competitors, and SEO — not from arbitrary numbers.

**Primary sources:**
- **web.dev — Performance budgets 101** — https://web.dev/articles/performance-budgets-101 — *Focus on: budget types (milestone metrics, quantity/size budgets, rule-based) and choosing thresholds.*
- **web.dev — Your first performance budget** — https://web.dev/articles/your-first-performance-budget — *Focus on: starting from current numbers and ratcheting.*
- **web.dev — Core Web Vitals thresholds** — https://web.dev/articles/defining-core-web-vitals-thresholds — *Focus on: where the "good/poor" lines come from.*

**Search prompts:**
- `"performance budget metric size quantity thresholds"`
- `"how to set performance goals competitors users seo"`
- `"performance budget ratchet baseline"`
- Ask an AI: *"How do I set realistic performance goals and budgets? Explain the three inputs (user needs, competitor benchmarks, SEO/CWV thresholds), the types of budget (metric budgets like 'LCP < 2.5s', quantity budgets like 'JS < 200KB', and rule budgets), and the 'ratchet from your current baseline' strategy so the budget is achievable and only gets stricter."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.9, Exercise 1 — performance goals and
budgets, build challenge. Completed: Track 0, Modules 1.1–1.8. This closes Track 1.

Give me a real-ish site (served locally) and a scenario where I must SET a performance budget for
it. My job, no solution code from you: measure the current baseline (CWV + total JS/image bytes),
research the right CWV thresholds and a reasonable competitor benchmark, and write a concrete
budget — metric budgets (e.g. P75 LCP target) AND quantity budgets (e.g. JS KB ceiling) — justified
by users, competitors, and SEO, and ratcheted from the current baseline so it's achievable.

Review me like a senior. If I pull budget numbers from thin air, do not accept them — ask me what
my baseline is, what my users' devices/networks look like, and what the CWV "good" line is. If I set
a budget so loose it's already passing trivially or so tight it's hopeless, push back. Exam me on:
the three inputs to a goal, the budget types, and why a budget must be enforced (foreshadow: in CI)
to matter. Demand layman + technical justification for each number.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 1, Module 1.9, Exercise 2 (transfer) — budget for a
real product. Apply the Exercise 2 rules: solo first; verdict.

Scenario: I'm the new performance owner for an app (you describe one — e.g. an Indian e-commerce
site whose users are largely on mid/low-end Android over 4G). Have me, solo, define a full
performance budget: the target CWV at P75, the byte budgets for JS/CSS/images/fonts, and a one-
paragraph rationale tying every number to THESE users, a plausible competitor, and SEO — plus how
I'd ratchet it over two quarters. Don't give me the numbers; make me derive and defend them. Exam
me on why a budget for low-end-Android users differs from one for a desktop SaaS app, and why an
unenforced budget is theater. Verdict: could I walk into a senior interview and lay out a defensible
performance budget for a given audience (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- A budget grounded in **three inputs**: (1) **user needs** — who they are, what devices/networks they're on (a budget for low-end Android on 4G is far stricter on JS bytes than one for desktop SaaS); (2) **competitors** — users notice a difference of roughly 20%, so "as fast or faster than the main alternative" is a real target; (3) **SEO/CWV thresholds** — the "good" lines (LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1) at P75.
- **Budget types:** *metric budgets* ("P75 LCP < 2.5s"), *quantity/size budgets* ("JS ≤ 170KB gzipped, images ≤ X"), and *rule budgets* (Lighthouse audits that must pass). The good answer uses more than one type.
- **Ratchet from the baseline:** you measure where you are now and set the budget slightly tighter than current, then tighten over time — a budget you're already failing by 5x is ignored; a budget you ratchet quarter by quarter actually drives behavior.
- **Per-audience reasoning:** the same app for different users gets different budgets. The Exercise 2 low-end-Android case should produce a noticeably stricter JS-byte budget than a desktop app, because parse/execute cost on a weak CPU dominates.
- **Enforcement is the point:** a budget that isn't checked (ideally in CI — Track 8) is decoration. The number only changes behavior if crossing it has a consequence.

The principle: **a performance budget is a concrete, enforceable commitment derived from your real users, your competitors, and the CWV thresholds — ratcheted from your current baseline so it's achievable and tightening.** Setting one well is a senior skill: it turns "make it fast" into specific numbers the whole team can build against, and explaining your reasoning for a given audience is exactly the kind of answer that makes an interviewer take a 4-year dev seriously.
</details>

---

*End of Track 1 — Measurement, Metrics & Methodology. 9 modules. You can now define, measure, and defend any performance number — lab and field, at the right percentile, with the noise accounted for, against a real budget.*

*Next: Track 2 — The Network & Loading Pipeline, where you start making things actually faster.*
