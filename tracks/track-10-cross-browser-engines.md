# TRACK 10 — CROSS-BROWSER & MULTI-ENGINE PERFORMANCE

*An advanced specialization, meant to be taken AFTER the core tracks (0–8) and ideally after the Track 9 capstones. Everything you've done so far has been, quietly, Chrome-shaped: DevTools, the V8 engine, Blink's rendering. That's the right place to start — it's the majority engine and the best tooling — but "system-level performance thinking" (the exact phrase from Yangshun Tay's "Frontend isn't dead" piece) includes a reality most engineers never confront: the same code performs differently on different browser engines, and the worst surprises — especially on iOS Safari — are invisible from Chrome. This track makes you the rare frontend engineer who can profile and reason across Blink, WebKit, and Gecko, and fix a page that's fast on Chrome but janky on Safari.*

*This is genuinely frontier knowledge and a real moat: it's where "works on my machine" goes to die, and almost nobody can diagnose it. The article's "profiling reflows caused by a specific CSS property on a specific browser" lives entirely here.*

**Practical note on devices.** Some of this needs Apple hardware. Safari only runs on macOS/iOS, and the Safari Web Inspector needs a Mac (and, for device profiling, an iPhone/iPad). If you don't have Apple devices: a cloud service like **BrowserStack** or **LambdaTest** gives you real Safari/iOS sessions (some with profiling), and the conceptual modules (10.1, 10.4, 10.5, 10.8) can be done by reasoning + reading without any Apple device at all. Firefox runs everywhere, so the Gecko modules need no special hardware. Don't let "I don't own a Mac" stop you — do the conceptual modules now and the device modules when you can borrow or rent access.

*Spine viewing/reading: there isn't a single Frontend Masters course dedicated to cross-engine performance (which is part of why this knowledge is scarce), so this track leans on primary sources — the engine teams' own blogs and docs (WebKit blog, Mozilla Hacks, Chrome/web.dev), MDN, and engineer conference talks. Your FM course **Bare Metal JavaScript: The JS Virtual Machine** is the one direct tie-in, for the JS-engine module (10.4).*

*Format unchanged: Prep → Exercise 1 (guided) → Exercise 2 (transfer, verified, verdict) → 🔒 SPOILER. Follow CLAUDE.md and PERF-CONTRACT.md.*

---

## Module 10.1 — The Three Engines and Why They Diverge

**Difficulty:** ●●●○○ · **Format:** Investigation (concept + measurement) · **Stack:** Chrome + Firefox (+ Safari if available)

### Prep — Read & Watch Before You Start

**Watch / primary viewing:**
- **"How browsers work" / rendering-engine talks** — search `"how browser rendering engine works talk"` — *Focus on: the shared pipeline (parse → DOM/CSSOM → layout → paint → composite) that all engines implement differently.*

**Primary sources:**
- **MDN — Web browser engines** — search `"mdn browser engine blink webkit gecko"` — *Focus on: which browsers use which engine, and that engine = rendering engine + JS engine.*
- **web.dev / "Inside look at modern web browser"** — https://developer.chrome.com/blog/inside-browser-part1 — *Focus on: the multi-process architecture and the rendering pipeline (Blink's view; the concepts generalize).*
- **WebKit blog** — https://webkit.org/blog/ — *Skim a few recent posts. Focus on: that WebKit has its own priorities, optimizations, and constraints distinct from Blink.*
- **Mozilla Hacks — Inside the engine** — search `"mozilla hacks gecko servo stylo quantum"` — *Focus on: Gecko's parallel style engine (Stylo) and how Gecko differs.*

**Search prompts:**
- `"blink vs webkit vs gecko differences"`
- `"v8 javascriptcore spidermonkey which browser"`
- `"why does same code perform differently in safari"`
- Ask an AI: *"Map the three major browser engines: Blink (Chrome, Edge, Opera, most Android) with the V8 JS engine; WebKit (Safari, all iOS browsers historically) with JavaScriptCore/Nitro; Gecko (Firefox) with SpiderMonkey. Blink forked from WebKit in 2013, then diverged. Explain WHY the same HTML/CSS/JS can perform differently across them — different layout/paint/compositing implementations, different JS JIT strategies, different memory and process models — and why Chrome-only profiling gives an incomplete performance picture."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.1, Exercise 1 — the three engines and why
they diverge. Completed: Tracks 0–9.

Teach me the engine landscape as a clear mental model first: the three rendering engines (Blink,
WebKit, Gecko), their paired JS engines (V8, JavaScriptCore, SpiderMonkey), which browsers ship each,
and the key history (Blink forked from WebKit in 2013, iOS historically WebKit-only — note the EU DMA
is changing that, so flag it as "verify current state"). Then explain the AXES on which engines
diverge for performance: JS execution (JIT tiers/strategies), layout, paint, compositing, memory/
process model, and feature support. Use concrete examples of each divergence.

Then make it real: give me a small test page exercising something likely to differ (e.g. a moderately
heavy layout + a JS hot loop + a CSS effect), and have me run the SAME page in Chrome and Firefox
(and Safari if I have it), capturing a simple metric (a timing, or just the profiler's headline
numbers) in each. Make me OBSERVE that the numbers differ across engines, and reason about why.
Exam me on: the engine→browser map, the divergence axes, and why Chrome-only profiling is incomplete.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.1, Exercise 2 (transfer) — engine reasoning.
Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT small page emphasizing a different potential divergence (e.g. a CSS-heavy effect,
or a different JS pattern). Have me, solo: predict WHICH engine is likely to handle it better or worse
and WHY (from the divergence axes), then run it across the engines I have access to and check my
prediction against reality, reasoning about any surprise. Don't tell me the answer. Exam me on
explaining engine divergence from first principles. Verdict: can I reason about why engines diverge
and predict/verify it (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Concept & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- A browser **engine** is really two engines: a **rendering engine** (parses HTML/CSS, builds the DOM/CSSOM, does layout/paint/composite) and a **JavaScript engine** (parses and JIT-compiles JS). The three pairs: **Blink + V8** (Chrome, Edge, Opera, most Android browsers), **WebKit + JavaScriptCore/Nitro** (Safari; historically *all* iOS browsers, including iOS Chrome/Firefox — though the EU DMA is now changing the iOS-WebKit-only rule, so verify current state), and **Gecko + SpiderMonkey** (Firefox).
- History that matters: Blink **forked from WebKit in 2013** and the two have diverged substantially since; they are no longer interchangeable. Gecko is independent and has pursued its own innovations (parallel CSS styling via Stylo/Quantum).
- The divergence axes (where the same code performs differently): **JS execution** (each JS engine has different JIT tiers and optimization heuristics, so a pattern fast on V8 can be slower on JavaScriptCore or SpiderMonkey, and deopt triggers differ — 10.4); **layout** (different algorithms/costs for reflow); **paint & compositing** (which properties get GPU-accelerated, raster costs, layer management differ — 10.5); **memory & process model** (especially iOS's tight limits — 10.7); **feature support** (an API present on one engine, absent on another, forcing different code paths — 10.8).
- Why Chrome-only profiling is incomplete: your DevTools numbers describe Blink+V8. They do not tell you how WebKit lays out your page, how JavaScriptCore runs your hot loop, or that iOS will kill your tab under memory pressure. The majority of "works on my machine, broken for users" performance bugs hide in this gap.

The principle: **a browser engine is a rendering engine plus a JS engine, and the three major pairs (Blink+V8, WebKit+JSC, Gecko+SpiderMonkey) implement the same web standards with different algorithms, optimizations, and limits — so identical code legitimately performs differently across them, and single-engine (Chrome) profiling is a partial view.** This is the foundation for everything else in the track: you can't diagnose cross-engine performance without knowing the engines diverge and why.
</details>

---

## Module 10.2 — Profiling Outside Chrome: Safari Web Inspector

**Difficulty:** ●●●●○ · **Format:** Tool mastery + investigation · **Stack:** Safari (macOS) — or BrowserStack

### Prep — Read & Watch Before You Start

**Primary sources:**
- **WebKit — Web Inspector** — https://webkit.org/web-inspector/ — *Focus on: the Timelines tab (the Safari equivalent of Chrome's Performance panel), the JavaScript & Events timeline, Layout & Rendering, and Memory.*
- **Apple — Safari Developer Help: Timelines** — search `"safari web inspector timelines tab profiling"` — *Focus on: recording a timeline and reading where time goes.*
- **WebKit — Enabling Develop menu / Web Inspector** — search `"enable safari develop menu web inspector"` — *Focus on: turning on the developer tools in Safari.*

**Search prompts:**
- `"safari web inspector timelines record performance"`
- `"safari profiling javascript layout rendering"`
- `"chrome devtools vs safari web inspector equivalents"`
- Ask an AI: *"I know Chrome DevTools' Performance panel well. Teach me the Safari Web Inspector equivalent: how to record a Timeline, the JavaScript & Events / Layout & Rendering / Memory timelines, how to read where main-thread time goes, and how to find layout/paint costs. Map each Chrome DevTools feature I rely on to its Safari Web Inspector counterpart (and note what's missing or different)."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.2, Exercise 1 — Safari Web Inspector.
Completed: Tracks 0–9, Module 10.1. (Needs Safari via a Mac, or a BrowserStack Safari session with
inspector access. If I truly can't access Safari, tell me and pivot to a reading+mapping exercise.)

Walk me through profiling in Safari Web Inspector as a Chrome-DevTools user. First, map the tools:
for each Chrome Performance-panel capability I use (record a trace, find long tasks, see layout/paint,
flame chart, memory), show me the Safari Web Inspector equivalent and how it differs. Then have me
take a page with a known main-thread problem (reuse one from an earlier track), record a Timeline in
Safari, and locate the cost — narrating the Safari-specific UI. Make me find the same problem I'd
find in Chrome, but using Safari's tools. Exam me on the tool mapping and on reading a Safari timeline.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.2, Exercise 2 (transfer) — Safari profiling
solo. Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT page with a different cost (a layout/paint issue, or a JS hot spot). Have me, solo:
profile it in Safari Web Inspector, locate the dominant cost, and state what I'd do about it — without
me falling back to Chrome. Don't tell me where the cost is. Exam me on independent Safari profiling.
Verdict: can I profile a page in Safari Web Inspector and find the bottleneck (proceed), or redo
Exercise 1?
```

### 🔒 SPOILER — What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- You cannot diagnose a Safari-only performance problem with Chrome DevTools — different engine, different tool. **Safari Web Inspector** is the WebKit equivalent, and its **Timelines** tab is the analog of Chrome's Performance panel: record a timeline, then read the JavaScript & Events, Layout & Rendering, and Memory timelines to see where main-thread time goes.
- The Chrome→Safari tool mapping (approximate): Performance panel → Timelines; flame chart → the JavaScript & Events timeline's call tree; layout/paint events → Layout & Rendering timeline; Memory panel → the Memory timeline / heap snapshots. Some Chrome features have no exact equivalent and vice versa — the point is fluency in both, not assuming parity.
- The skill is being able to walk into the *other* toolset and still find the bottleneck. Engineers who can only use Chrome DevTools are blind to half the web's users.

The principle: **Safari Web Inspector's Timelines is the WebKit counterpart to Chrome's Performance panel, and profiling a WebKit page requires it — because a Safari-specific cost is invisible in Chrome.** Tool fluency across engines is the practical half of cross-browser performance; the conceptual half (why the cost exists) comes from the engine-divergence modules.
</details>

---

## Module 10.3 — Profiling Outside Chrome: The Firefox Profiler

**Difficulty:** ●●●○○ · **Format:** Tool mastery + investigation · **Stack:** Firefox (any OS)

### Prep — Read & Watch Before You Start

**Primary sources:**
- **Firefox Profiler docs** — https://profiler.firefox.com/docs/ — *Focus on: recording a profile, the Call Tree, the Stack Chart (flame graph), the Marker Chart, and the Flame Graph view.*
- **Mozilla — Profiling with the Firefox Profiler** — search `"firefox profiler call tree stack chart guide"` — *Focus on: reading where time is spent and the per-category breakdown.*
- **Firefox DevTools — Performance** — https://firefox-source-docs.mozilla.org/devtools-user/performance/ — *Focus on: the DevTools performance tooling and how it differs from Chrome's.*

**Search prompts:**
- `"firefox profiler how to record read call tree"`
- `"firefox profiler stack chart flame graph"`
- `"firefox profiler layout reflow analysis"`
- Ask an AI: *"Teach me the Firefox Profiler as a Chrome DevTools user: recording a profile, the Call Tree vs Stack Chart vs Flame Graph vs Marker Chart, the category coloring, and how it surfaces layout/reflow and JS costs. What does the Firefox Profiler show or do well that Chrome's Performance panel doesn't (e.g. its markers, its handling of multi-threaded work), and how do I map my Chrome habits onto it?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.3, Exercise 1 — the Firefox Profiler.
Completed: Tracks 0–9, Modules 10.1–10.2. (Firefox runs on any OS — no special hardware.)

Walk me through the Firefox Profiler as a Chrome-DevTools user. Map the tools: Chrome's flame chart →
Firefox's Stack Chart / Flame Graph; Chrome's bottom-up/call-tree → Firefox's Call Tree; Chrome's
performance markers → Firefox's Marker Chart. Then have me profile a page with a known cost (reuse an
earlier-track page — ideally one with a layout/reflow issue, which the Firefox Profiler presents
well), record in Firefox, and locate the cost using Firefox's views. Note anything Firefox surfaces
that Chrome didn't (or shows differently). Exam me on the tool mapping and reading a Firefox profile.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.3, Exercise 2 (transfer) — Firefox profiling
solo. Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT page with a different dominant cost. Have me, solo: profile it in the Firefox
Profiler, find the bottleneck, and compare — does Firefox agree with what Chrome would say, or does
the cost differ across engines (tie back to 10.1)? Don't tell me the answer. Exam me on independent
Firefox profiling and on noticing cross-engine differences. Verdict: can I profile in Firefox and
find the bottleneck (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The **Firefox Profiler** is Gecko's profiling tool, and it runs on any OS, so it's the easiest non-Chrome profiler to adopt. Its views: the **Call Tree** (aggregated where-time-went), the **Stack Chart** and **Flame Graph** (timeline and aggregated flame views), and the **Marker Chart** (layout, reflow, paint, GC, and custom markers over time).
- The Chrome→Firefox mapping: flame chart → Stack Chart/Flame Graph; call tree/bottom-up → Call Tree; performance markers/timings → Marker Chart. The Firefox Profiler is particularly good at showing reflow/layout markers and at multi-threaded breakdowns.
- Because it's the same page on a *different* engine, comparing the Firefox profile to the Chrome one is a direct way to *see* engine divergence (10.1): the dominant cost may shift, confirming that "fast in Chrome" ≠ "fast everywhere."

The principle: **the Firefox Profiler is Gecko's analog to Chrome's Performance panel — runs everywhere, with Call Tree, Stack Chart/Flame Graph, and a Marker Chart — and profiling the same page in it both builds cross-engine tool fluency and lets you directly observe how the cost differs on a different engine.** It's the lowest-friction way to stop being a single-engine profiler.
</details>

---

## Module 10.4 — JavaScript Engine Divergence: V8 vs JavaScriptCore vs SpiderMonkey

**Difficulty:** ●●●●● · **Format:** Investigation + measurement · **Stack:** Chrome + Firefox (+ Safari)

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — direct tie-in):**
- **Bare Metal JavaScript: The JS Virtual Machine** → the sections on parsing, bytecode, the optimizing compiler, hidden classes/shapes, inline caches, and **deoptimization**. You learned this V8-centrically in Track 3; this module generalizes it across engines.

**Primary sources:**
- **V8 blog** — https://v8.dev/blog — *Focus on: how V8 optimizes (Sparkplug/Maglev/TurboFan tiers, hidden classes, inline caches).*
- **WebKit — JavaScriptCore / "Speculation in JavaScriptCore"** — https://webkit.org/blog/10308/speculation-in-javascriptcore/ — *Focus on: JSC's multi-tier JIT (LLInt → Baseline → DFG → FTL) and how its speculation/deopt model compares to V8's.*
- **Mozilla — SpiderMonkey / "Warp" posts on Mozilla Hacks** — search `"spidermonkey warp ion jit mozilla hacks"` — *Focus on: SpiderMonkey's JIT pipeline and optimization strategy.*

**Search prompts:**
- `"v8 javascriptcore spidermonkey jit tiers comparison"`
- `"javascript fast in chrome slow in safari why"`
- `"engine specific deoptimization differences"`
- Ask an AI: *"Compare the three JS engines' optimization: V8 (Ignition + Sparkplug/Maglev/TurboFan), JavaScriptCore (LLInt → Baseline → DFG → FTL), SpiderMonkey (interpreter → Baseline → Warp/Ion). All use multi-tier JITs with speculation and deoptimization, but their heuristics differ — so a hot loop or a polymorphic call site can be optimized on one engine and deoptimized on another. Give me examples of patterns that perform differently across engines and explain why, building on hidden classes/shapes, inline caches, and megamorphism."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.4, Exercise 1 — JS engine divergence.
Completed: Tracks 0–9, Modules 10.1–10.3. This builds on Track 3's V8 deopt work.

First, teach me the three JITs side by side: V8, JavaScriptCore, and SpiderMonkey each use a tiered
interpreter→JIT pipeline with speculation and deoptimization, but with different heuristics — so the
SAME JS can be optimized on one engine and deoptimized on another. Connect to what I learned in Track
3 (hidden classes/shapes, inline caches, megamorphism, deopt triggers) and generalize it.

Then make it measurable: give me a few JS micro-benchmarks designed to stress optimization (a
monomorphic vs polymorphic vs megamorphic call site; a function that changes object shape; a hot
numeric loop; something that triggers a deopt on V8). Have me run each in Chrome and Firefox (and
Safari if available), record the timings, and OBSERVE that the relative performance differs across
engines. Make me reason about which engine handled what better and why. (Caveat me on micro-benchmark
pitfalls — warmup, dead-code elimination — so I measure honestly.) Exam me on the engine JIT
comparison and on explaining a measured cross-engine difference.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.4, Exercise 2 (transfer) — JS engine
divergence solo. Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT JS pattern (e.g. heavy string work, array method chains, a different
polymorphism scenario). Have me, solo: predict how the three engines will differ and why (from their
JIT/speculation models), then measure across the engines I have and reconcile with my prediction,
handling micro-benchmark hygiene myself. Don't tell me the answer. Exam me on reasoning about
engine-specific JS performance. Verdict: can I reason about and measure JS engine divergence
(proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- All three JS engines use a **tiered pipeline**: a fast interpreter for cold code, then one or more **JIT** tiers that **speculatively optimize** hot code based on observed types, with **deoptimization** back to a lower tier when assumptions break. V8: Ignition (interpreter) → Sparkplug → Maglev → TurboFan. JavaScriptCore: LLInt → Baseline → DFG → FTL. SpiderMonkey: interpreter → Baseline → Warp/Ion. Same idea, different heuristics and thresholds.
- Because the heuristics differ, **the same JS can be optimized on one engine and deoptimized on another** — a call site that stays monomorphic and fast on V8 might be handled differently by JSC; a shape change that triggers a V8 deopt might cost differently on SpiderMonkey. The Track 3 concepts (hidden classes/shapes, inline caches, monomorphic→polymorphic→megamorphic degradation, deopt triggers) apply to all three, but the exact costs and cliffs are engine-specific.
- Practical upshot: a "blazing fast" hot path tuned only against V8 may not be fast on JavaScriptCore (i.e. on every iPhone). Truly engine-robust hot code avoids patterns that any major engine punishes (keep call sites monomorphic, keep object shapes stable, avoid megamorphism) rather than micro-tuning for one engine.
- Honest measurement: micro-benchmarks lie if you don't warm up the JIT, prevent dead-code elimination, and run enough iterations — so cross-engine timing comparisons need hygiene.

The principle: **V8, JavaScriptCore, and SpiderMonkey all JIT-compile hot JS with speculation and deoptimization but use different heuristics, so identical code can hit different optimization/deopt outcomes per engine — meaning engine-robust hot code follows the shared rules (monomorphism, stable shapes, no megamorphism) rather than tuning for V8 alone.** Your Track 3 V8 knowledge generalizes; the cliffs just move per engine, and on iOS the engine is always JavaScriptCore.
</details>

---

## Module 10.5 — Rendering & Compositing Divergence: When a CSS Property Is Cheap on Blink but Expensive on WebKit

**Difficulty:** ●●●●● · **Format:** Investigation + measurement · **Stack:** Chrome + Safari (+ Firefox)

### Prep — Read & Watch Before You Start

**Primary sources:**
- **WebKit blog — compositing / "CSS will-change", filters, backdrop-filter posts** — https://webkit.org/blog/ — *Search the blog for compositing, `backdrop-filter`, and `will-change`. Focus on: WebKit's compositing and raster costs and how they differ from Blink's.*
- **web.dev — Stick to compositor-only properties / "Animations guide"** — https://web.dev/articles/animations-guide — *Focus on: which properties are compositor-friendly (Blink's view) — then question whether it holds on WebKit.*
- **MDN — will-change** — https://developer.mozilla.org/en-US/docs/Web/CSS/will-change — *Focus on: the warning that over-promoting layers has costs — costs that differ per engine.*
- **csstriggers (archived reference)** — search `"csstriggers layout paint composite reference"` — *Focus on: the layout/paint/composite map (Blink-era) — and that engines disagree.*

**Search prompts:**
- `"backdrop-filter performance safari vs chrome"`
- `"will-change layer promotion cost different browsers"`
- `"css filter blur expensive safari ios"`
- Ask an AI: *"Explain why a CSS property can be cheap to render on Blink but expensive on WebKit: differences in which properties get GPU-composited, layer/raster costs, and how each engine handles effects like `backdrop-filter`, `filter: blur()`, large composited layers, blend modes, and `will-change` over-promotion. iOS Safari especially can choke on effects that are smooth in Chrome. How do I identify a rendering cost that only appears on WebKit?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.5, Exercise 1 — rendering/compositing
divergence. Completed: Tracks 0–9, Modules 10.1–10.4. This extends Track 4 (the pixel pipeline) across
engines.

Teach me why the Track-4 pixel pipeline (style/layout/paint/composite) has engine-specific COSTS:
which properties get GPU-composited differs, raster and layer-management costs differ, and effects
like `backdrop-filter`, `filter: blur()`, large/many composited layers, blend modes, and `will-change`
over-promotion can be smooth on Blink but expensive on WebKit (and especially janky on iOS Safari).

Then make it concrete: give me a page with a few such effects (a backdrop-filtered sticky header, a
blurred overlay, a heavily layer-promoted animation). Have me profile its rendering in Chrome (Track-4
style) AND in Safari Web Inspector (10.2), and compare the paint/composite cost across engines —
finding an effect that's cheap on Blink but expensive on WebKit. Exam me on the per-engine rendering
cost differences and on how I'd identify a WebKit-only rendering problem.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.5, Exercise 2 (transfer) — rendering
divergence solo. Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT page with different effects (different filters/blends/compositing patterns). Have
me, solo: predict which effects are likely to be more expensive on WebKit than Blink and why, then
profile across engines to confirm, and propose an engine-robust alternative for any effect that's a
WebKit cliff (e.g. a cheaper effect, a static fallback, or reduced layer count). Don't tell me the
answer. Exam me on identifying and fixing engine-specific rendering costs. Verdict: can I diagnose a
rendering cost that differs across engines and make it engine-robust (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The Track-4 pixel pipeline is universal in concept, but its **costs are engine-specific**: engines differ in which properties get GPU-composited, how they rasterize and manage layers, and how expensive specific effects are. The "compositor-only properties are cheap" guidance is Blink-shaped and doesn't fully transfer.
- Known cross-engine cliffs: **`backdrop-filter`** and **`filter: blur()`** can be smooth in Chrome but expensive in Safari/WebKit (and especially on iOS GPUs); large or numerous **composited layers** cost differently; **blend modes** and **`will-change`** over-promotion can backfire harder on WebKit; an animation that's silky on a desktop Blink GPU can melt a real iOS GPU (tie back to Track 8.8's real-device reality).
- Diagnosing a WebKit-only rendering cost requires profiling rendering in **Safari Web Inspector** (10.2), not Chrome — the cost is invisible in Blink's profiler. The fix is usually an **engine-robust alternative**: a cheaper effect, fewer composited layers, a static fallback on the offending engine, or `@supports`/feature-detection to swap effects.
- This is the literal scenario in Yangshun's article — "profiling reflows [and paint] caused by a specific CSS property on a specific browser" — and it's where most "beautiful on my Mac Chrome, janky on the user's iPhone" bugs come from.

The principle: **the rendering pipeline's costs are engine-specific — which properties composite, raster/layer costs, and effect expense all differ — so an effect cheap on Blink (e.g. `backdrop-filter`, heavy blur, many layers) can be a cliff on WebKit/iOS, diagnosable only by profiling in Safari and fixable with engine-robust alternatives.** Track 4 taught the pipeline; this taught that the bill differs per engine, which is the "specific CSS property on a specific browser" skill exactly.
</details>

---
## Module 10.6 — Scrolling, Momentum & Input Differences Across Engines

**Difficulty:** ●●●●○ · **Format:** Investigation + broken project · **Stack:** Chrome + Safari/iOS (+ Firefox)

### Prep — Read & Watch Before You Start

**Primary sources:**
- **MDN — Passive event listeners** — https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#improving_scroll_performance_using_passive_listeners — *Focus on: why non-passive touch/wheel listeners block scrolling, and engine differences in defaults.*
- **WebKit — scrolling / `-webkit-overflow-scrolling` / momentum** — search `"webkit momentum scrolling ios overflow"` — *Focus on: iOS momentum scrolling behavior and its quirks.*
- **web.dev — Debounce your input handlers / scroll-linked effects** — search `"scroll linked effects jank requestAnimationFrame"` — *Focus on: why scroll-linked effects jank, and how it differs across engines.*
- **MDN — overscroll-behavior** — https://developer.mozilla.org/en-US/docs/Web/CSS/overscroll-behavior — *Focus on: controlling scroll chaining/bounce, which behaves differently per engine.*

**Search prompts:**
- `"ios safari scroll jank passive listeners"`
- `"scroll linked animation janky safari vs chrome"`
- `"120hz promotion safari animation frame rate"`
- Ask an AI: *"Explain cross-engine scrolling and input differences: passive vs non-passive touch/wheel listeners and their engine-default differences, iOS momentum scrolling and `-webkit-overflow-scrolling`, `overscroll-behavior` differences, why scroll-linked effects (parallax, scroll-driven animations) jank differently across engines, and how high-refresh displays (ProMotion 120Hz) interact with animation frame budgets. Why is scrolling smooth in Chrome but janky on iOS Safari, and how do I fix it engine-robustly?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.6, Exercise 1 — scrolling/momentum/input
differences. Completed: Tracks 0–9, Modules 10.1–10.5.

Teach me how scrolling and input performance diverge across engines: non-passive touch/wheel listeners
blocking the scroll thread (and engine default differences), iOS momentum scrolling quirks,
`overscroll-behavior` differences, scroll-linked effects (parallax / scroll-driven animation) janking
differently per engine, and high-refresh (120Hz ProMotion) tightening the frame budget.

Then give me a page that scrolls smoothly in Chrome desktop but janks on iOS Safari: a non-passive
touch/scroll handler doing layout-reading work, plus a scroll-linked effect. Have me reproduce the
jank (ideally on a real iPhone or BrowserStack iOS; otherwise reason from a Safari profile), diagnose
why it's worse on WebKit/iOS, and fix it engine-robustly (passive listeners, move work off the scroll
handler / into rAF, simplify the scroll-linked effect, respect the frame budget). Exam me on the
cross-engine scroll/input differences and the fix.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.6, Exercise 2 (transfer) — scroll/input
divergence solo. Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT scroll/input scenario (a different scroll-linked effect, a drag interaction, or a
sticky/parallax pattern). Have me, solo: predict where it'll be worse on iOS Safari and why, verify,
and fix it so it's smooth across engines. Don't tell me the answer. Exam me on diagnosing and fixing
cross-engine scroll/input jank. Verdict: can I make scrolling/input smooth across engines (proceed),
or redo Exercise 1?
```

### 🔒 SPOILER — What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- Scrolling and input performance diverge across engines in ways that bite hardest on iOS Safari. **Non-passive** touch/wheel listeners block the scroll thread (the engine must wait to see if you'll `preventDefault`); passive listeners let scrolling stay on the compositor — and engine defaults/behaviors differ. **iOS momentum scrolling** has its own quirks. **`overscroll-behavior`** (scroll chaining, bounce) behaves differently per engine. **Scroll-linked effects** (parallax, scroll-driven animations) jank differently because each engine schedules scroll and rendering differently. **High-refresh displays** (ProMotion 120Hz) cut the per-frame budget roughly in half (≈8ms vs ≈16ms), so an effect that hits 60fps on a 60Hz screen can drop frames on a 120Hz iPhone.
- The fix is engine-robust: mark scroll/touch listeners **passive**, keep work **off** the scroll handler (read once, write via `requestAnimationFrame`, never layout-thrash in a scroll callback — Track 4.3), simplify or drop expensive scroll-linked effects on weaker engines, and respect the tighter frame budget on high-refresh displays.
- Diagnosis often needs a **real iOS device** (10.9) or a Safari profile (10.2) — desktop Chrome won't show the iOS scroll jank.

The principle: **scrolling and input diverge across engines — passive-listener handling, momentum, overscroll, scroll-linked-effect scheduling, and high-refresh frame budgets all differ — so "smooth in Chrome" can be "janky on iOS Safari," and the fix is keeping scroll handlers passive and off the main thread plus simplifying effects to stay within the (sometimes 8ms) frame budget.** This is a top source of real-world iOS jank that Chrome profiling never reveals.
</details>

---

## Module 10.7 — The iOS Safari Reality: Memory Ceilings, Tab Reloads & Platform Limits

**Difficulty:** ●●●●● · **Format:** Investigation + broken project · **Stack:** iOS Safari (real device or BrowserStack)

### Prep — Read & Watch Before You Start

**Primary sources:**
- **WebKit — memory / "Web Inspector Memory" + Jetsam** — search `"ios safari memory limit tab reload jetsam webkit"` — *Focus on: iOS's aggressive memory management and why a tab reloads under pressure.*
- **web.dev — Memory and the JS heap** — search `"javascript heap memory limit mobile browser"` — *Focus on: heap limits being far smaller on mobile, especially iOS.*
- **WKWebView / iOS web constraints** — search `"wkwebview memory constraints ios web app"` — *Focus on: the platform limits an in-app or Safari web view runs under.*
- **DMA / iOS browser engine choice (current state)** — search `"ios third party browser engines eu dma webkit"` — *Focus on: that iOS was historically WebKit-only and this is changing under regulation — verify current state.*

**Search prompts:**
- `"why does safari reload my tab a problem occurred"`
- `"ios safari javascript memory limit crash"`
- `"reduce memory usage spa ios safari"`
- Ask an AI: *"Explain the iOS Safari memory reality: iOS imposes tight per-tab memory limits and will RELOAD a tab ('A problem repeatedly occurred') or kill it under memory pressure (Jetsam) — far more aggressively than desktop. The JS heap ceiling is much lower on iOS. Why do memory-heavy SPAs (large caches, leaks, huge DOM, big images decoded in memory) that run fine on desktop Chrome get reloaded/killed on iPhones, and how do I reduce memory footprint to stay under the limit? Also: note that iOS was historically WebKit-only for all browsers (changing under the EU DMA — verify current state)."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.7, Exercise 1 — the iOS Safari memory
reality. Completed: Tracks 0–9, Modules 10.1–10.6. (Best on a real iPhone or BrowserStack iOS; if
unavailable, do this as a memory-profiling + reasoning exercise and I'll flag what needs a device.)

Teach me the iOS platform reality: tight per-tab memory limits, tab RELOADS / kills under memory
pressure (Jetsam), a much lower JS-heap ceiling than desktop, and why this is invisible on desktop
Chrome. Note that iOS historically required WebKit for ALL browsers, so on iPhone "use Chrome" didn't
change the engine — and flag that the EU DMA is changing this (verify current state).

Then give me a memory-heavy SPA that's fine on desktop but gets reloaded/killed on iOS: an unbounded
in-memory cache, a leak (listeners/closures/detached DOM retained — tie to Track 8.9), a huge DOM,
and large images decoded in memory. Have me measure memory (Safari/Chrome memory tools + heap
snapshots), find the footprint drivers, and reduce them (bound caches, fix leaks, virtualize the DOM,
right-size/lazy images) until it stays under a realistic ceiling. Exam me on the iOS memory model and
the footprint reductions.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.7, Exercise 2 (transfer) — iOS memory solo.
Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT memory-heavy app (a different leak/cache/asset profile). Have me, solo: profile
its memory, identify why it would be reloaded/killed on iOS, and reduce the footprint enough to
survive — proving the reduction. Don't tell me the drivers. Exam me on diagnosing and fixing iOS
memory pressure. Verdict: can I diagnose and reduce a memory footprint to survive iOS limits
(proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- iOS is a fundamentally more constrained platform, and this is invisible from desktop. iOS imposes **tight per-tab memory limits** and will **reload the tab** ("A problem repeatedly occurred") or **kill it** under memory pressure (the system's **Jetsam** mechanism), far more aggressively than desktop browsers. The **JS heap ceiling is much lower** on iOS than on desktop.
- So memory-heavy SPAs that run fine on desktop Chrome — **unbounded in-memory caches**, **leaks** (retained listeners/closures/detached DOM — Track 8.9), a **huge DOM**, **large images decoded in memory** — get reloaded or killed on iPhones, which users experience as the page flashing and losing state.
- Historically, **all iOS browsers were required to use WebKit** (iOS Chrome/Firefox were WebKit under the hood), so "switch browsers" didn't change the engine or the limits on iPhone. The **EU DMA** is changing the engine-choice rule — verify the current state — but WebKit/iOS constraints remain the practical reality.
- The fix is footprint discipline: **bound caches**, **fix leaks**, **virtualize large lists/DOM** (Track 6.7), **right-size and lazy-load images** (Track 5) so they aren't all decoded in memory at once, and generally treat memory as a hard budget — because on iOS it is.

The principle: **iOS enforces tight memory limits and reloads/kills tabs under pressure, with a much lower JS-heap ceiling than desktop — so memory-heavy apps that are fine on desktop Chrome get reloaded on iPhones, and surviving iOS means treating memory as a hard budget (bounded caches, no leaks, virtualized DOM, right-sized images).** This is the single most surprising "works on my machine" cliff, because desktop gives no warning of it.
</details>

---

## Module 10.8 — Feature Availability & Fallbacks That Become Performance Cliffs

**Difficulty:** ●●●●○ · **Format:** Investigation + broken project · **Stack:** Chrome + Safari + Firefox

### Prep — Read & Watch Before You Start

**Primary sources:**
- **MDN — `@supports` / feature queries** — https://developer.mozilla.org/en-US/docs/Web/CSS/@supports — *Focus on: detecting feature support and branching, instead of assuming.*
- **caniuse.com** — https://caniuse.com — *Focus on: checking real cross-engine support, including the "known issues" notes per feature.*
- **web.dev — Progressive enhancement & feature detection** — search `"feature detection progressive enhancement performance"` — *Focus on: enhancing where supported, falling back where not — without the fallback becoming a cliff.*
- **MDN — `content-visibility`, native lazy loading, IntersectionObserver (support tables)** — *Focus on: examples where support and behavior differ across engines.*

**Search prompts:**
- `"content-visibility browser support fallback"`
- `"feature detection fallback javascript polyfill performance cost"`
- `"safari missing api polyfill performance"`
- Ask an AI: *"When a performance-relevant feature is missing on one engine (e.g. `content-visibility`, native `loading=lazy`, a newer image format, an IntersectionObserver option, a CSS feature), the fallback I ship can itself be a performance cliff — a heavy JS polyfill, eager loading, a larger asset, or a more expensive layout path. How do I feature-detect (`@supports`, JS checks) and provide fallbacks that degrade gracefully WITHOUT the fallback tanking performance on the engine that needed it?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.8, Exercise 1 — feature availability &
fallback cliffs. Completed: Tracks 0–9, Modules 10.1–10.7.

Teach me the pattern: a performance feature available on one engine but not another (e.g.
`content-visibility`, native lazy-loading, a modern image format, a CSS feature) means I ship a
FALLBACK for the engine that lacks it — and a careless fallback (a heavy JS polyfill, eager loading,
a much larger asset, a costlier layout path) can be WORSE on that engine than no feature at all. So
the engine that most needs help gets the slowest path.

Then give me a page that uses a perf feature with a naive fallback that's a cliff on the engine
lacking support (e.g. a big polyfill that runs on Safari, or a giant fallback image where a modern
format isn't supported). Have me detect support (`@supports` / JS feature checks / `<picture>` with
type fallbacks), measure the fallback's cost on the engine that hits it, and replace it with a
graceful fallback that doesn't tank performance. Exam me on feature detection and on designing
fallbacks that degrade without a perf cliff.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.8, Exercise 2 (transfer) — fallback cliffs
solo. Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT feature-support gap with a different naive fallback. Have me, solo: identify which
engine takes the fallback path, measure its cost there, and design a graceful, performant fallback
(or decide the feature isn't worth the cliff). Don't tell me the answer. Exam me on cross-engine
feature detection and graceful, performant degradation. Verdict: can I ship features with fallbacks
that don't become perf cliffs on the engine that needs them (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- Performance-relevant features (e.g. **`content-visibility`**, native **`loading="lazy"`**, modern **image formats** like AVIF, certain **CSS** features, some **IntersectionObserver** options) aren't supported uniformly across engines. When a feature is missing, you ship a **fallback** — and a careless fallback can be a **performance cliff** on the very engine that lacked the feature: a heavy JS **polyfill** that runs only on Safari, **eager loading** where native lazy-loading was unavailable, a **much larger asset** where a modern format wasn't supported, or a **costlier layout path**.
- The cruel irony: the engine that most needs help (the one missing the optimization) gets the *slowest* path if you're careless — so your "fallback" inverts the intent.
- The fix is **feature detection + graceful degradation**: `@supports` for CSS, JS feature checks, `<picture>`/`type` for image-format fallbacks, and choosing fallbacks that degrade *gracefully* (a slightly less optimal but still cheap path) rather than a cliff. Sometimes the right call is to skip the enhancement entirely if its fallback is too costly.
- This connects to the whole track: you must *know* the engine support differs (10.1), and ideally *measure* the fallback's cost on the engine that hits it (10.2/10.3).

The principle: **performance features have uneven cross-engine support, so you ship fallbacks — and a careless fallback (heavy polyfill, eager load, larger asset, costlier path) becomes a performance cliff on the engine that lacked the feature, inverting the intent — so you feature-detect and design fallbacks that degrade gracefully without tanking performance.** Progressive enhancement done with performance awareness, not just correctness.
</details>

---

## Module 10.9 — Real Device Profiling: Profiling an Actual iPhone via Safari Web Inspector

**Difficulty:** ●●●●○ · **Format:** Investigation + real device · **Stack:** Real iPhone/iPad + Mac (Safari Web Inspector)

### Prep — Read & Watch Before You Start

**Primary sources:**
- **Apple — Inspecting iOS Safari with Web Inspector** — search `"safari web inspector inspect iphone device usb"` — *Focus on: enabling Web Inspector on the iOS device (Settings → Safari → Advanced) and connecting it to the Mac's Safari Develop menu over USB.*
- **WebKit — Remote Web Inspector** — https://webkit.org/web-inspector/ — *Focus on: profiling the page running on the actual device, not a desktop simulation.*
- **Revisit Track 8.8 (Android remote debugging)** — *the same idea — profile the real low-end device — but for iOS, the platform where most "works on Android, broken on iPhone" surprises live.*

**Search prompts:**
- `"inspect iphone safari from mac web inspector"`
- `"profile ios safari real device timeline"`
- `"safari develop menu device web inspector usb"`
- Ask an AI: *"Walk me through profiling a real iPhone: enable Web Inspector on the device (Settings → Safari → Advanced → Web Inspector), connect via USB, open the page on the iPhone, and inspect/profile it from the Mac's Safari Develop menu → [device]. How do I record a Timeline of the real-device page, and what will the real iPhone reveal that desktop Safari simulation and Chrome both miss (real GPU/CPU/memory limits, real iOS behavior)?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.9, Exercise 1 — real iPhone profiling.
Completed: Tracks 0–9, Modules 10.1–10.8. (Needs a real iPhone/iPad + a Mac. If I lack Apple devices,
tell me and substitute a BrowserStack real-device session or a reasoning exercise, flagging the gap.)

This is the iOS counterpart to Track 8.8's Android remote debugging. Walk me through: enabling Web
Inspector on the iPhone (Settings → Safari → Advanced), connecting via USB, opening my page on the
device, and profiling it from the Mac's Safari Develop menu → [my iPhone]. Have me take a page from an
earlier module (ideally the rendering/scroll/memory ones — 10.5/10.6/10.7) and profile it on the REAL
iPhone, comparing what the real device shows vs desktop Chrome and vs Safari-on-Mac simulation. Make
me find a cost that only the real iPhone reveals (real GPU limits on an effect, real memory pressure,
real thermal/CPU limits). Exam me on the real-device workflow and on what the device reveals that
simulation hides.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.9, Exercise 2 (transfer) — real iPhone
profiling solo. Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT page. Have me, solo: profile it on a real iPhone via Web Inspector, find the
dominant real-device cost, and state the fix — distinguishing what's worse on the real device than in
simulation. Don't tell me the cost. Exam me on independent real-iOS profiling. Verdict: can I profile
a real iPhone and find the real-device bottleneck (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- This is the iOS twin of Track 8.8's Android remote debugging: profile the page on the **real device**, because simulation underestimates real costs. Enable **Web Inspector on the iPhone** (Settings → Safari → Advanced → Web Inspector), connect via **USB**, open the page on the device, and profile it from the **Mac's Safari Develop menu → [device]** — recording a real-device Timeline.
- The real iPhone reveals what both desktop Chrome and desktop-Safari simulation hide: **real GPU limits** (the `backdrop-filter`/blur/heavy-compositing effects from 10.5 that melt a real mobile GPU), **real memory pressure** and tab reloads (10.7), and **real thermal/CPU throttling** under sustained work. iOS is where "works on my machine" most often dies, and only the real device gives you the truth.
- Combined with the Android real-device work (8.8), you can now ground-truth performance on *both* mobile platforms — the two that matter most and that desktop development systematically lies about.

The principle: **profiling a real iPhone via Safari Web Inspector (device Web Inspector + USB + the Mac's Develop menu) is the iOS counterpart to Android remote debugging, revealing real GPU/memory/thermal limits that desktop Chrome and even desktop-Safari simulation hide — making it the ground truth for the platform where most cross-engine performance surprises live.** Desktop is the lab; the real iPhone is the verdict.
</details>

---

## Module 10.10 — Cross-Engine Capstone: Fast on Chrome, Janky on Safari/iOS

**Difficulty:** ●●●●● · **Format:** Investigation + broken project (multi-cause) · **Stack:** Chrome + Safari/iOS (+ Firefox) + all tools

### Prep — Read & Watch Before You Start

**Primary sources:**
- **Revisit your notes from 10.1–10.9** — *this capstone composes the whole track.*
- **Revisit Track 9's diagnosis methodology** — *triage by dominant cost, fix in impact order, re-baseline — now applied across engines.*

**Search prompts:**
- `"page fast chrome slow safari debugging methodology"`
- `"cross browser performance regression diagnosis"`
- Ask an AI: *"Give me a methodology for a page that's fast on Chrome but janky on Safari/iOS: confirm the gap by profiling on both engines (and a real iPhone), triage whether the dominant cross-engine cost is JS-engine, rendering/compositing, scrolling/input, memory, or a fallback cliff, fix the biggest engine-specific cost first, re-baseline across engines, and repeat — making the page fast on EVERY engine, not just Chrome."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.10, Exercise 1 — cross-engine capstone.
Completed: Tracks 0–9, Modules 10.1–10.9. This is the whole track at once. (Best with Safari/iOS access
— real device ideal, BrowserStack acceptable; tell me if I lack it and adapt.)

Build me a page that is genuinely fast on Chrome but JANKY on Safari/iOS for SEVERAL distinct,
engine-specific reasons at once — e.g.: a hot JS path that deopts on JavaScriptCore but not V8 (10.4);
a `backdrop-filter`/heavy-blur effect cheap on Blink but expensive on WebKit (10.5); a non-passive
scroll handler that janks on iOS (10.6); a memory footprint that pressures iOS (10.7); and a
feature-fallback that's a cliff on Safari (10.8). Do NOT tell me the causes or which engine each hits.

Have me work the Track-9 way, but cross-engine: confirm the Chrome-vs-Safari gap by profiling BOTH
(and a real iPhone if I have one), triage the dominant engine-specific cost, fix it, RE-BASELINE
across engines (the bottleneck will move), and repeat until the page is fast on every engine — not
just Chrome. Make me justify each diagnosis with engine reasoning and prove each fix on the engine it
mattered. Exam me hard: which cost was on which engine, why, and how I confirmed it.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 10, Module 10.10, Exercise 2 (transfer) — cross-engine
capstone, cold. Apply the Exercise 2 rules: solo first; verdict. This is the proof I've got the track.

Build me a DIFFERENT page that's fast on Chrome but janky on Safari/iOS for a different mix of
engine-specific causes. Hand it to me cold. Have me, solo and start to finish: confirm the gap across
engines, triage and fix in impact order, re-baseline across engines after each fix, and get it fast on
every engine — diagnosing each cause's engine and reason myself. Don't tell me anything. Exam me on
the full cross-engine diagnosis. Verdict: can I take an unfamiliar page that's fast on Chrome but
janky on another engine, diagnose WHY per engine, and fix it cold? If yes, I'm the rare engineer who
does cross-engine performance — the exact "specific CSS property on a specific browser" skill from the
article. If not, which module to revisit.
```

### 🔒 SPOILER — What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The capstone composes the whole track into the canonical real-world scenario: **fast on Chrome, janky on Safari/iOS**, for *several* engine-specific reasons at once. You apply Track 9's methodology — triage by dominant cost, fix in impact order, **re-baseline after each fix** because the bottleneck moves — but **across engines**, profiling on both (and a real iPhone) rather than Chrome alone.
- The multi-cause mix exercises everything: a hot path that **deopts on JavaScriptCore** but not V8 (10.4); a **`backdrop-filter`/blur** effect cheap on Blink, expensive on WebKit (10.5); a **non-passive scroll handler** janking on iOS (10.6); a **memory footprint** pressuring iOS (10.7); a **feature-fallback cliff** on Safari (10.8) — each diagnosed with engine reasoning (10.1) using the right engine's profiler (10.2/10.3/10.9).
- The deliverable skill is exactly Yangshun's "optimization that requires system-level thinking" and "profiling reflows caused by a specific CSS property on a specific browser": not "Chrome says it's fine," but "it's fast on every engine, and I can prove why each engine-specific cost existed and that I fixed it."

The principle: **a page fast on Chrome but janky on Safari/iOS is conquered by the Track-9 method applied cross-engine — confirm the gap on both engines, triage the dominant engine-specific cost (JS-engine deopt, WebKit rendering, iOS scroll/memory, fallback cliff), fix it, re-baseline across engines, repeat — until it's fast everywhere.** Being able to do this cold makes you the rare engineer who owns performance across the whole browser landscape, not just the one engine with the nicest DevTools.
</details>

---

*End of Track 10 — Cross-Browser & Multi-Engine Performance. 10 modules. You can now reason about why Blink, WebKit, and Gecko diverge; profile in Safari Web Inspector and the Firefox Profiler, not just Chrome; diagnose JS-engine, rendering, scrolling, memory, and fallback differences across engines; profile a real iPhone; and fix a page that's fast on Chrome but janky on Safari/iOS — the exact "specific CSS property on a specific browser" system-level skill, and a genuine moat almost no frontend engineer has.*

*This completes the performance primer's advanced specialization. Combined with Tracks 0–9, you can diagnose any page, on any major engine, on real devices — which is the whole job, done at a level the market rarely sees.*
