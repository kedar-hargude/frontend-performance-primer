# TRACK 4 — RENDERING PERFORMANCE & THE PIXEL PIPELINE

*The third bucket: "janky scroll, stuttering animations, the pretty page that melts a cheap phone." This is the track that most directly serves your stated goal — being able to look at a fancy site, find exactly which animation or effect is browser-heavy, and predict how it'll behave on weak hardware. You'll learn the full pixel pipeline (style → layout → paint → composite), why some changes are cheap and others catastrophic, and how to keep animations on the GPU's fast path.*

*Prerequisites: Tracks 0–1 (read a flame chart, measure honestly), and Track 3 helps (the main thread runs rendering too). A few modules touch React later, but this track is mostly vanilla so the mechanics are exposed.*

*Spine courses: **JavaScript Performance** (Steve Kinney) covers the render pipeline and reflow/repaint directly; **Web Performance Fundamentals, v2** touches rendering under INP/animations; and **Mastering Chrome Developer Tools, v4** for the Rendering tab and paint/layer tools you'll live in here.*

---

## Module 4.1 — The Pixel Pipeline: Style, Layout, Paint, Composite

**Difficulty:** ●●○○○ · **Format:** Investigation · **Stack:** Vanilla HTML/CSS/JS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **JavaScript Performance** (Steve Kinney) → the **rendering pipeline** sections — Steve walks the browser's path from a style change to pixels on screen (recalculate style → layout → paint → composite), which is the backbone of this whole track.

**Primary sources:**
- **web.dev — Inside look at modern web browser (rendering)** — https://web.dev/articles/rendering-performance — *Focus on: the five steps (JS → style → layout → paint → composite) and which steps a given change triggers.*
- **web.dev — The pixel pipeline** — https://web.dev/articles/rendering-performance#the_pixel_pipeline — *Focus on: the three paths (full pipeline, paint+composite, composite-only) and why composite-only is cheapest.*
- **CSS Triggers (concept)** — https://web.dev/articles/animations-guide — *Focus on: which properties trigger layout vs paint vs composite.*

**Search prompts:**
- `"browser rendering pipeline style layout paint composite"`
- `"which css properties trigger layout vs paint vs composite"`
- `"pixel pipeline composite only cheapest"`
- Ask an AI: *"Walk me through the browser's pixel pipeline: JavaScript → recalculate style → layout (reflow) → paint → composite. Explain the three possible paths (a change can trigger the whole pipeline, just paint+composite, or just composite) and why composite-only changes (transform/opacity) are the cheapest — they skip layout and paint entirely."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.1, Exercise 1 — the pixel pipeline.
Completed: Tracks 0–3.

Build me a page with three buttons, each animating an element a DIFFERENT way: (A) by changing a
layout property (e.g. `top`/`left`/`width`), (B) by changing a paint property (e.g.
`background-color`/`box-shadow`), and (C) by changing a composite-only property (`transform`/
`opacity`). Do NOT tell me which path each triggers or which is cheapest.

Have me record a Performance trace for each (with 4x–6x CPU throttle) and open the DevTools Rendering
tab tools (Paint flashing, FPS meter). Make me OBSERVE, for each animation, which pipeline stages
fire (look for "Recalculate Style", "Layout", "Paint", "Composite Layers" in the trace) and which
animation is smoothest. Do NOT interpret for me. Exam me on: the five pipeline stages, the three
paths, and why composite-only is cheapest.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.1, Exercise 2 (transfer) — classify the
pipeline cost of arbitrary changes. Apply the Exercise 2 rules: solo first; verdict.

Give me a list of ~8 different CSS changes/animations (mix of layout-triggering, paint-triggering,
and composite-only — including some non-obvious ones). Have me, solo, PREDICT for each which
pipeline stages it triggers and how expensive it is, BEFORE measuring — then verify each with a
trace + the Rendering tab. You design it so prediction is possible if I understand the pipeline.
Don't tell me the answers. Exam me on why the same visual effect (moving an element) can be cheap or
ruinously expensive depending on which property I animate. Verdict: can I classify any change's
pipeline cost on sight (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

No bug — this is the foundational model for the whole track. What you should now hold:

- The **pixel pipeline** has five stages: **JavaScript** (your code changes something) → **Recalculate Style** (figure out which CSS applies) → **Layout/Reflow** (compute geometry — positions and sizes) → **Paint** (fill in pixels: colors, text, shadows) → **Composite** (assemble painted layers into the final image, often on the GPU).
- A change can trigger **three different paths**:
  - **Full pipeline** (layout → paint → composite): animating geometric properties (`width`, `height`, `top`, `left`, `margin`) forces the browser to recompute layout for potentially the whole page, then repaint, then composite. Most expensive.
  - **Paint + composite** (skips layout): changing `background-color`, `box-shadow`, `color` — no geometry change, but pixels must be repainted.
  - **Composite-only** (skips layout AND paint): changing **`transform`** and **`opacity`** — the element is already painted into a layer; the browser just re-positions/blends that layer, often entirely on the GPU. Cheapest by far.
- This is why "move an element" can be cheap or catastrophic: animating `left` triggers layout every frame; animating `transform: translateX()` triggers neither layout nor paint.

The principle: **every visual change costs differently depending on how far down the pixel pipeline it reaches — layout-triggering changes are expensive, paint changes are moderate, and composite-only changes (transform/opacity) are nearly free because they skip straight to the GPU.** Knowing which path a property takes is the single most important rendering-performance skill, and it's the lens for every remaining module in this track.
</details>

---

## Module 4.2 — Reflow (Layout): The Most Expensive Stage

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla JS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **JavaScript Performance** (Steve Kinney) → the **reflow / layout** lesson — Steve shows what forces the browser to recompute layout and why it's costly.

**Primary sources:**
- **MDN — Reflow** — https://developer.mozilla.org/en-US/docs/Glossary/Reflow — *Focus on: what reflow is and what triggers it.*
- **web.dev — Avoid large, complex layouts** — https://web.dev/articles/avoid-large-complex-layouts-and-layout-thrashing — *Focus on: layout cost scaling with DOM size and complexity.*
- **Paul Irish — What forces layout/reflow (gist)** — https://gist.github.com/paulirish/5d52fb081b3570c81e3a — *Reference: every property/method that forces synchronous layout.*

**Search prompts:**
- `"what triggers reflow layout browser"`
- `"forced synchronous layout offsetHeight getBoundingClientRect"`
- `"layout cost dom size complexity"`
- Ask an AI: *"What is reflow (layout) and why is it the most expensive rendering stage? List the things that trigger it (changing geometry, adding/removing DOM, and READING layout properties like `offsetHeight`/`getBoundingClientRect`/`scrollTop`). Why does reading a layout property force a synchronous layout if there are pending style changes?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.2, Exercise 1 — reflow. Completed:
Tracks 0–3, Module 4.1.

Build me a page where an interaction is sluggish because it triggers excessive LAYOUT: e.g. code
that animates a geometric property (width/top/left) on a sizable element every frame, and/or a
DOM operation that forces a big reflow where a cheaper approach existed. Do NOT name the cause.

Have me baseline (trace shows lots of purple "Layout" work), confirm via the Rendering tab that
layout is firing repeatedly, and trace it to the layout-triggering property/operation. Fix it
(switch the animation to transform, or restructure to avoid the reflow) and re-measure the Layout
time and frame rate. Hold the loop. Exam me on: what reflow computes, what triggers it, why it
scales with DOM size, and why animating `left` is far worse than animating `transform`.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.2, Exercise 2 (transfer) — reflow, new
cause. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT layout-heavy scenario (e.g. a resize/scroll-driven effect that recalculates
geometry, or repeatedly toggling something that changes the document flow). Have me, solo: find the
excessive layout in the trace, identify the trigger, fix it, and prove the Layout time dropped and
frames smoothed. Exam me on why layout cost depends on how much of the page must be re-laid-out and
why a big DOM amplifies it. Verdict: can I spot and eliminate reflow-driven jank (proceed), or redo
Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problem: animating a geometric property (`width`/`height`/`top`/`left`) on a non-trivial element forces **layout every frame** — the browser recomputes the geometry of that element and potentially everything affected by it. On a large/complex DOM this is expensive, and at 60fps it's catastrophic. The trace is full of "Layout" (purple) work.
- The fix: animate **`transform`** instead (composite-only — no layout), or restructure so the reflow doesn't happen (e.g. take the animated element out of flow, or change a cheaper property). Layout time drops to near-zero; frames smooth out.
- What forces layout: changing geometry, adding/removing/resizing DOM, changing font/content that affects size — AND, importantly, *reading* certain layout properties (`offsetHeight`, `offsetWidth`, `getBoundingClientRect`, `scrollTop`, `clientWidth`...) while style changes are pending, which forces a **synchronous** layout right then (next module's topic).
- Why it scales: layout can cascade — changing one element's size can reflow its siblings, ancestors, and descendants, so a big or deeply-nested DOM makes every reflow more expensive.

The principle: **layout (reflow) is the most expensive rendering stage because it recomputes geometry that can cascade across the page, so animating geometric properties or triggering frequent reflows is a primary cause of jank — and the fix is almost always to avoid layout entirely by using composite-only properties or restructuring.** Spotting purple "Layout" bars dominating a trace is an instant diagnosis.
</details>

---

## Module 4.3 — Layout Thrashing: Forced Synchronous Layout

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** Vanilla JS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **JavaScript Performance** (Steve Kinney) → the **layout thrashing / forced synchronous layout** material — the read-then-write-in-a-loop antipattern and how to batch.

**Primary sources:**
- **web.dev — Avoid layout thrashing** — https://web.dev/articles/avoid-large-complex-layouts-and-layout-thrashing#avoid_forced_synchronous_layouts — *Focus on: the read/write batching pattern.*
- **Paul Irish — What forces layout (revisit)** — https://gist.github.com/paulirish/5d52fb081b3570c81e3a — *Reference: the exact reads that force synchronous layout.*
- **MDN — `requestAnimationFrame`** — https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame — *Focus on: scheduling the write phase aligned to the frame.*

**Search prompts:**
- `"layout thrashing read write batch forced synchronous layout"`
- `"interleaved read write offsetHeight loop reflow"`
- Ask an AI: *"Explain layout thrashing (forced synchronous layout): why reading a layout property (like `offsetHeight`) right after writing a style, repeatedly in a loop, forces the browser to recompute layout on every iteration. Show the fix: separate the loop into a READ phase (measure everything) then a WRITE phase (mutate everything). Why does batching collapse N reflows into one?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.3, Exercise 1 — layout thrashing. Completed:
Tracks 0–3, Modules 4.1–4.2. Raise the difficulty.

Build me a feature that loops over MANY elements and, per element, READS a layout property and then
WRITES a style based on it (interleaved) — forcing a synchronous reflow every iteration. Classic
examples: an "equal heights" routine, a masonry layout, or a scroll effect. It should jank badly
under CPU throttle. Do NOT name the antipattern.

Have me baseline (the trace shows a wall of repeated Layout events interleaved with script), confirm
the read-after-write pattern, and fix it by splitting into a single READ pass then a single WRITE
pass (scheduling writes in rAF where apt). Re-measure: the wall of layouts collapses to ~one. Hold
the loop. Exam me on: what forces synchronous layout, why interleaving defeats the browser's
batching, and why read-all-then-write-all fixes it.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.3, Exercise 2 (transfer) — thrashing, new
instance. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT layout-thrashing scenario (a different real-world routine that interleaves
reads and writes over many nodes). Have me, solo: spot the thrash in the trace, identify the exact
read that forces sync layout, refactor into measure-then-mutate phases, and prove the Layout count
collapsed. Exam me on which property reads are "layout-forcing" and why the browser can't batch when
you read mid-loop. Verdict: can I recognize and fix layout thrashing cold (proceed), or redo
Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The antipattern: a loop that, per element, **writes** a style (queuing a layout invalidation) and then **reads** a layout property (`offsetHeight`/`getBoundingClientRect`/etc.) — the read forces the browser to flush the pending layout **synchronously, right now**, because it must return an accurate value. Do this N times and you force N synchronous reflows. The trace shows alternating script/Layout bars — the "thrash" signature.
- The fix: **batch** — one pass that READS all the measurements into an array (no writes yet), then one pass that WRITES all the mutations (no reads). Now the browser does the layout work once. Schedule the write phase in `requestAnimationFrame` to align with the frame. The wall of Layout events collapses to roughly one.
- Why interleaving is pathological: the browser *wants* to batch layout (it defers it until needed), but reading a layout-dependent value while changes are pending forces an immediate flush, defeating the batching every iteration.

The principle: **reading a layout property while style changes are pending forces a synchronous reflow, so interleaving reads and writes in a loop triggers a reflow per iteration — and separating the loop into a read phase then a write phase collapses N reflows into one.** Layout thrashing is one of the most common real-world rendering bugs; the read/write batching fix is a senior-level reflex.
</details>

---

## Module 4.4 — The Compositor: Animating on the GPU

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** Vanilla JS/CSS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **JavaScript Performance** (Steve Kinney) → the **compositing / GPU** discussion — why `transform`/`opacity` animate on the compositor thread and stay smooth even when the main thread is busy.

**Primary sources:**
- **web.dev — Stick to compositor-only properties and manage layer count** — https://web.dev/articles/stick-to-compositor-only-properties-and-manage-layer-count — *Focus on: transform/opacity on the compositor, and `will-change` to promote a layer (and its cost).*
- **web.dev — Animations and performance** — https://web.dev/articles/animations-and-performance — *Focus on: the compositor thread running animations independently of the main thread.*
- **MDN — `will-change`** — https://developer.mozilla.org/en-US/docs/Web/CSS/will-change — *Focus on: promoting a layer deliberately, and why overusing it backfires.*

**Search prompts:**
- `"compositor thread transform opacity gpu animation"`
- `"will-change layer promotion cost memory"`
- `"animation smooth even when main thread busy compositor"`
- Ask an AI: *"Explain the compositor thread: how animating `transform` and `opacity` can run on the GPU/compositor independently of the main thread, so the animation stays smooth even if the main thread is busy. What does promoting an element to its own layer (via `will-change` or `transform`) cost in GPU memory, and why does promoting too many layers ('layer explosion') hurt?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.4, Exercise 1 — the compositor. Completed:
Tracks 0–3, Modules 4.1–4.3.

Build me a page with: (1) an animation using a layout/paint property that JANKS, especially when the
main thread is also busy (give me a button that runs some main-thread work so I can see the animation
stutter); and (2) the SAME visual animation done with `transform`/`opacity` that stays smooth even
during main-thread work. Do NOT tell me which is which or why.

Have me trace both and use the DevTools Layers panel + the Rendering tab (Layer borders, FPS meter).
Make me OBSERVE that the compositor-driven animation stays at 60fps even while the main thread is
blocked, and the other doesn't. Then have me explore `will-change` to promote a layer — AND deliberately
overuse it to see layer explosion / memory cost in the Layers panel. Exam me on: the compositor
thread, why transform/opacity bypass the main thread, layer promotion, and the cost of too many layers.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.4, Exercise 2 (transfer) — compositor,
applied to a real effect. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT janky animation (e.g. a sliding panel, a parallax-ish effect, or a moving card)
implemented with layout-triggering properties. Have me, solo: convert it to a compositor-friendly
implementation (transform/opacity), promote layers only where justified, prove via the Layers panel
and a trace that it now runs on the compositor at 60fps even under main-thread load, AND check the
GPU memory cost of any layers I created. The trap: promoting everything with will-change backfires.
Don't tell me the fix. Exam me on the compositor model and the layer-count tradeoff. Verdict: can I
move an animation to the GPU correctly and judge layer cost (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The contrast: the layout/paint-based animation must go through the main thread every frame, so when the main thread is busy (your button's work), it stutters and drops frames. The **`transform`/`opacity`** animation runs on the **compositor thread** — the element is painted once into its own layer, and the compositor just re-positions/blends that layer on the GPU, independently of the main thread. So it stays at 60fps even while the main thread is blocked. The Layers panel shows the promoted layer; the FPS meter confirms smoothness.
- **Layer promotion:** `will-change: transform` (or an existing `transform`/`opacity` animation) promotes an element to its own compositor layer so it can be animated cheaply. But each layer costs **GPU memory**, and promoting too many ("layer explosion") wastes memory and can *hurt* — especially on weak devices. Overusing `will-change` is a classic mistake.
- The weak-hardware angle (your goal): compositor animations are what stay smooth on cheap phones; layout/paint animations are what melt them. And layer explosion is what runs them out of GPU memory.

The principle: **the compositor thread can animate `transform` and `opacity` on the GPU independently of a busy main thread, which is why those properties stay smooth under load — but each promoted layer costs GPU memory, so you promote deliberately, not everywhere.** This is the core of building animations that survive weak hardware: keep them composite-only, and keep layer count sane.
</details>

---

## Module 4.5 — Diagnosing a Janky Animation (Your Core Goal)

**Difficulty:** ●●●●○ · **Format:** Investigation + broken project · **Stack:** Vanilla JS/CSS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Mastering Chrome Developer Tools, v4** → the **Rendering tab** tools (Paint flashing, Layer borders, FPS/frame rendering stats) and the **Performance panel** frames view. This is the toolset for pinpointing a heavy animation.

**Primary sources:**
- **Chrome DevTools — Analyze runtime performance** — https://developer.chrome.com/docs/devtools/performance — *Focus on: the Frames track, dropped frames, and finding what each frame spent time on.*
- **Chrome DevTools — Rendering tab** — https://developer.chrome.com/docs/devtools/rendering — *Focus on: Paint flashing (what's repainting), Layer borders, and the FPS meter.*
- **web.dev — Diagnose forced reflows / slow frames** — https://web.dev/articles/rendering-performance — *Focus on: reading the per-frame breakdown.*

**Search prompts:**
- `"devtools paint flashing find what repaints"`
- `"devtools frames dropped jank diagnose animation"`
- `"which animation causing jank profiling"`
- Ask an AI: *"Give me a method to find which animation/effect on a page is causing jank: use the Rendering tab's Paint flashing (to see what's repainting), Layer borders (to see compositor layers), and the FPS meter; record a trace and read the Frames track for dropped frames; then attribute each janky frame to its cause (layout? paint? too many layers?). How do I isolate ONE offending effect on a busy page?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.5, Exercise 1 — diagnosing a janky animation.
Completed: Tracks 0–3, Modules 4.1–4.4. This module IS my core goal — take it seriously.

Build me a "pretty" page with SEVERAL animations/effects running, where ONE (or two) of them is the
real performance villain (layout-triggering or paint-heavy or causing layer churn) while the others
are fine. Make it look like a real fancy landing page. Do NOT tell me which effect is the problem.

This is pure diagnosis. Have me: turn on Paint flashing and the FPS meter, record a trace, find the
dropped frames, and ISOLATE which specific animation is causing the jank and WHY (which pipeline
stage it hits). Make me prove it by disabling that one effect and watching frames recover. Then have
me fix that effect (composite-only rewrite) and confirm. Exam me on the isolation method and the
mechanism. This rehearses exactly what I want to do on real sites.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.5, Exercise 2 (transfer) — isolate the
villain on a NEW busy page. Apply the Exercise 2 rules: minimal hints; verdict.

Build me a DIFFERENT fancy page with a different mix of effects and a different hidden villain (maybe
two villains with different causes — one layout, one paint/layers). Have me, solo: use Paint
flashing, Layer borders, the FPS meter, and a trace to isolate each problem effect, name its
mechanism, prove it by toggling, and fix it. Near-zero hints. Exam me hard on the methodology. Then
the verdict I most want: could I open a real fancy site cold and pinpoint which animation is browser-
heavy and how it'd behave on weak hardware (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What This Is Teaching (your stated end goal)

<details>
<summary>Click only after you finish or give up</summary>

This module directly trains the skill you described: look at a pretty page and find which effect is killing it. The methodology you should now run:

- **Paint flashing** (Rendering tab): the browser highlights regions as they repaint. A well-built composite-only animation shows almost no flashing; a paint-heavy one (animated shadows, gradients, large repainting areas) lights up constantly — instantly revealing the culprit.
- **FPS meter / Frames track:** shows dropped frames. Correlate the drops with which effect is active.
- **Layer borders** (Rendering tab): shows compositor layers — too many (layer explosion) or unexpected promotions point at GPU-memory problems.
- **The trace's Frames track:** for a janky frame, see whether time went to Layout (purple), Paint (green), or compositing — telling you the *stage*, hence the cause.
- **Isolation by toggling:** disable one effect at a time and watch whether frames recover — the definitive way to pin the villain on a busy page.
- **The weak-hardware prediction:** layout/paint-heavy effects that are merely "a bit janky" on your laptop will be unusable on a cheap phone (less CPU/GPU). Composite-only effects scale down gracefully. This is how you predict weak-device behavior from a desktop trace + throttling.

The principle: **finding the janky effect on a busy page is a systematic isolation — Paint flashing shows what repaints, Layer borders show compositor cost, the Frames track shows which pipeline stage each dropped frame spent on, and toggling effects one at a time confirms the villain.** Master this and you can do exactly what you set out to: open any fancy site, name the browser-heavy animation, explain why, and predict its behavior on weak hardware.
</details>

---

## Module 4.6 — CSS Containment & content-visibility: Skipping Offscreen Work

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla HTML/CSS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- No dedicated FM lesson; this is modern CSS the courses predate. Lean on the primary sources below — but the **JavaScript Performance** pipeline knowledge (what layout/paint cost) is the prerequisite understanding.

**Primary sources:**
- **MDN — CSS containment (`contain`)** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Using_CSS_containment — *Focus on: telling the browser a subtree is independent so it can skip work outside it.*
- **web.dev — `content-visibility`** — https://web.dev/articles/content-visibility — *Focus on: `content-visibility: auto` to skip rendering work for offscreen content (huge win on long pages).*
- **MDN — `content-visibility` & `contain-intrinsic-size`** — https://developer.mozilla.org/en-US/docs/Web/CSS/content-visibility — *Focus on: avoiding scrollbar jumps with intrinsic size.*

**Search prompts:**
- `"content-visibility auto skip offscreen rendering"`
- `"css contain layout paint style isolate subtree"`
- `"contain-intrinsic-size scrollbar jump content-visibility"`
- Ask an AI: *"Explain CSS containment (`contain`) and `content-visibility: auto`: how they let the browser skip layout/paint work for subtrees that are offscreen or independent, dramatically speeding up long pages. Why is `contain-intrinsic-size` needed alongside `content-visibility` to prevent scrollbar jumps?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.6, Exercise 1 — containment &
content-visibility. Completed: Tracks 0–3, Modules 4.1–4.5.

Build me a very long page (hundreds of complex sections/cards far below the fold) that is slow to
render and slow on interaction because the browser does layout/paint work for ALL of it, including
offscreen content. Do NOT mention containment.

Have me baseline (rendering time, scroll smoothness, the trace's layout/paint cost). Make me discover
that offscreen content is being rendered unnecessarily, then apply `content-visibility: auto` (with
`contain-intrinsic-size` to avoid scroll jumps) and/or `contain` to skip that work, re-measuring the
rendering cost and scroll smoothness. Exam me on: what containment promises the browser, what
`content-visibility: auto` skips, and why intrinsic-size matters.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.6, Exercise 2 (transfer) — containment, new
layout. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT long/complex page (e.g. a feed, a docs page, a dashboard with many widgets).
Have me, solo: measure the rendering cost, apply containment/content-visibility correctly to skip
offscreen and isolate independent subtrees, handle the intrinsic-size to avoid scrollbar jumps, and
prove the rendering/scroll improvement. The trap: wrong intrinsic-size causes layout jumps; over-
containing can break expected layout. Don't tell me where to apply it. Exam me on the tradeoffs.
Verdict: can I use containment to cut offscreen rendering work correctly (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problem: a long page where the browser computes layout and paint for ALL sections, including the hundreds offscreen — wasting work on content the user can't see, slowing initial render and every subsequent layout.
- The fix: **`content-visibility: auto`** on the offscreen sections tells the browser to skip their rendering work until they're near the viewport (it still reserves space). Pair with **`contain-intrinsic-size`** to give each skipped section an estimated size, so the scrollbar doesn't jump as content is rendered/un-rendered. **`contain: layout/paint/style`** more generally promises the browser a subtree is independent, so changes inside it don't trigger layout/paint outside it.
- Re-measure: initial rendering cost drops sharply (the browser skips offscreen work), and scrolling is smoother because only near-viewport content is rendered.
- The traps: a wrong `contain-intrinsic-size` causes scrollbar jumps as estimates are corrected; over-aggressive `contain` can break layouts that actually did depend on cross-boundary effects.

The principle: **`content-visibility` and `contain` let you tell the browser which work it can skip — rendering offscreen content lazily and isolating independent subtrees — which is one of the biggest rendering wins for long, complex pages, at the cost of getting intrinsic sizing right.** This is a modern, underused lever that an expert reaches for on any long feed or document.
</details>

---

## Module 4.7 — Scroll Performance: Passive Listeners & Smooth Scrolling

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla JS/CSS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- No single dedicated FM lesson; the **JavaScript Performance** main-thread/rendering knowledge is the prerequisite. Lean on primary sources.

**Primary sources:**
- **MDN — Passive event listeners** — https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#improving_scrolling_performance_with_passive_listeners — *Focus on: how a non-passive scroll/touch listener blocks scrolling, and `{ passive: true }`.*
- **MDN — `overscroll-behavior`** — https://developer.mozilla.org/en-US/docs/Web/CSS/overscroll-behavior — *Focus on: preventing scroll chaining jank.*
- **web.dev — Debounce your input handlers / scroll** — search — *Focus on: not doing heavy work on every scroll event; using rAF/IntersectionObserver instead.*

**Search prompts:**
- `"passive event listener scroll performance"`
- `"scroll jank heavy handler requestAnimationFrame"`
- `"scroll-linked effects performance intersectionobserver"`
- Ask an AI: *"Explain scroll performance: why a non-passive `scroll`/`touchstart` listener can block the browser from scrolling on the compositor (it must wait to see if you call `preventDefault`), and how `{ passive: true }` fixes it. Why is doing heavy work in a scroll handler janky, and what should I use instead (rAF throttling, IntersectionObserver, CSS scroll-driven animations)?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.7, Exercise 1 — scroll performance.
Completed: Tracks 0–3, Modules 4.1–4.6.

Build me a page with janky scrolling caused by: a non-passive scroll/touch listener that blocks the
compositor, and/or a scroll handler doing heavy work (layout reads, DOM updates) on every scroll
event. Do NOT name the causes.

Have me baseline scroll smoothness (FPS meter, trace during scroll, the "scroll blocking" warnings).
Make me find the non-passive listener and the heavy handler, then fix them: `{ passive: true }`,
throttle work to rAF / move detection to IntersectionObserver, avoid layout reads in the handler.
Re-measure scroll FPS. Exam me on: why a non-passive listener blocks compositor scrolling, why heavy
scroll handlers jank, and the better primitives (rAF, IntersectionObserver).
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.7, Exercise 2 (transfer) — scroll, new
effect. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT scroll-linked effect (e.g. a sticky header that changes on scroll, a progress
bar, or reveal-on-scroll) implemented in a janky way. Have me, solo: diagnose the scroll jank (passive?
heavy handler? layout reads?), fix it with the right primitive, and prove smooth scrolling on
throttled CPU. Don't tell me the cause. Exam me on the compositor-scroll/passive relationship and on
choosing rAF vs IntersectionObserver vs CSS scroll-driven animation. Verdict: can I build and fix
smooth scroll effects (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problems: (1) a **non-passive** `scroll`/`touchstart`/`wheel` listener — the browser must wait for the handler to (possibly) call `preventDefault` before it can scroll, so it can't scroll on the compositor immediately, causing input-delay jank. Fix: `addEventListener('scroll', fn, { passive: true })`. (2) A scroll handler doing **heavy work every event** — layout reads (`scrollY`-driven measurements), DOM updates — firing dozens of times per second. Fix: throttle to one update per frame with `requestAnimationFrame`, move visibility detection to **IntersectionObserver**, or use CSS scroll-driven animations where possible. (3) Layout reads in the handler causing forced sync layout (ties to 4.3).
- Re-measure: scroll FPS recovers; the "scroll blocking" warnings disappear.
- The mechanism: scrolling normally happens on the compositor thread (smooth even if main thread is busy) — but a non-passive listener or heavy main-thread scroll work drags scrolling back onto the main thread.

The principle: **smooth scrolling depends on keeping the scroll path off the main thread — passive listeners so the compositor can scroll immediately, and minimal per-event work (throttled to rAF, or replaced by IntersectionObserver/CSS) so the main thread doesn't bottleneck the scroll.** Janky scroll is almost always a non-passive listener or an over-busy handler, and both are quick, high-impact fixes.
</details>

---

## Module 4.8 — DOM Size & Style Recalculation

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla JS/CSS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **JavaScript Performance** (Steve Kinney) → the **style recalculation** material — how the browser figures out which styles apply and why a huge DOM amplifies every recalc.

**Primary sources:**
- **web.dev — DOM size and interactivity** — https://web.dev/articles/dom-size-and-interactivity — *Focus on: why a large DOM slows style/layout and memory.*
- **web.dev — Reduce the scope and complexity of style calculations** — https://web.dev/articles/reduce-the-scope-and-complexity-of-style-calculations — *Focus on: selector cost and how many elements a recalc touches.*

**Search prompts:**
- `"large dom size performance style recalculation"`
- `"css selector performance style recalc scope"`
- Ask an AI: *"Why does a large DOM hurt performance — for style recalculation, layout, and memory? Is CSS selector complexity usually the bottleneck, or is it the NUMBER of elements a style recalc must process? How do I find excessive style-recalc cost in a trace and reduce it (smaller DOM, simpler structure, scoped changes)?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.8, Exercise 1 — DOM size & style recalc.
Completed: Tracks 0–3, Modules 4.1–4.7.

Build me a page with an enormous/over-nested DOM (tens of thousands of nodes, or deeply nested
structure) where a simple class toggle or interaction triggers expensive Recalculate Style and
Layout over a huge tree, causing lag. Do NOT name the cause.

Have me baseline (the trace shows big "Recalculate Style" + "Layout" bars on a trivial change),
recognize the DOM size as the amplifier, and reduce it (flatten/trim the DOM, scope the change, or
virtualize — foreshadow Track 6). Re-measure the recalc/layout cost. Exam me on: what style recalc
does, why DOM size amplifies it (and layout, and memory), and whether selector complexity or element
count is usually the real cost.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.8, Exercise 2 (transfer) — DOM cost, new
case. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT case where DOM size/complexity is the bottleneck (e.g. a giant table, or a
component that renders far more nodes than necessary). Have me, solo: measure the recalc/layout cost,
identify the DOM bloat, reduce it, and prove the improvement. Make me reason about when the right fix
is "render fewer nodes" (virtualization, Track 6) vs "simplify structure." Don't tell me the fix.
Exam me on DOM-size cost and the element-count-vs-selector-complexity question. Verdict: can I
diagnose DOM-size-driven rendering cost (proceed to the capstone), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problem: a huge or deeply-nested DOM means that even a trivial change (toggling a class) forces the browser to **recalculate styles** for many elements and potentially **re-layout** a large tree — so a "simple" interaction shows big Recalculate Style + Layout bars. A large DOM also costs memory and makes every reflow more expensive.
- The fix: **reduce the number of nodes** — flatten unnecessary nesting, remove wrapper-div soup, render fewer elements, and for very large lists, **virtualize** (render only visible rows — full treatment in Track 6). Scope changes so a recalc touches a small subtree (ties to containment, 4.6). Re-measure: recalc/layout cost drops with the node count.
- The element-count-vs-selector question: in practice the **number of elements** a recalc must process usually dominates over selector complexity (modern engines are fast at matching). So "fewer nodes" beats "cleverer selectors" as a strategy, though pathological selectors can still matter.

The principle: **the cost of style recalculation and layout scales with how many DOM nodes are involved, so an oversized or over-nested DOM makes every style change and reflow expensive — and the highest-leverage fix is rendering fewer nodes (trim structure, virtualize), not micro-optimizing selectors.** DOM size is a quiet, pervasive rendering tax that experts watch closely, especially on data-heavy UIs.
</details>

---

## Module 4.9 — The Rendering Diagnosis (Pixel-Pipeline Capstone)

**Difficulty:** ●●●●● · **Format:** Investigation + broken project · **Stack:** Vanilla JS/CSS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- Revisit **JavaScript Performance** (rendering pipeline) and **Mastering Chrome Developer Tools, v4** (Rendering tab, Layers, Frames) — this capstone integrates the whole track.

**Primary sources:**
- **web.dev — Rendering performance (revisit)** — https://web.dev/articles/rendering-performance — *Focus on: the full diagnostic toolkit together.*
- **Chrome DevTools — Rendering tab + Performance frames (revisit)** — https://developer.chrome.com/docs/devtools/rendering — *Focus on: combining Paint flashing, Layers, and the Frames track for diagnosis.*

**Search prompts:**
- `"diagnose rendering jank methodology layout paint composite"`
- Ask an AI: *"Give me a complete methodology to diagnose a janky page's RENDERING: identify whether each janky frame is layout-bound, paint-bound, or composite/layer-bound; use Paint flashing, Layer borders, the FPS meter, and the Frames track; then map each problem to its fix (composite-only animation, batch reads/writes, containment, passive scroll, smaller DOM)."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.9, Exercise 1 — rendering diagnosis,
pixel-pipeline capstone. Completed: Tracks 0–3, Modules 4.1–4.8. Make it genuinely hard.

Build me a fancy, busy page that's janky for SEVERAL rendering reasons at once: a layout-triggering
animation, a paint-heavy effect, layout thrashing somewhere, a non-passive scroll handler, and a
too-large DOM doing expensive recalcs. Do NOT tell me how many problems or what they are.

Full rendering diagnosis. Have me: use Paint flashing, Layer borders, FPS meter, and a trace; for
each janky frame, classify it (layout? paint? composite/layers? recalc?); isolate each villain;
apply the matching fix (composite-only, batch read/write, containment, passive listener, trim DOM);
and re-measure cumulatively. Hold the loop. Long exam tying each fix to its pipeline stage. This is
the full rehearsal for diagnosing a real fancy site's rendering cold.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 4, Module 4.9, Exercise 2 (transfer) — diagnose a
DIFFERENT janky page solo. Apply the Exercise 2 rules: minimal hints; full verdict.

Build me a DIFFERENT fancy page with a different mix of rendering problems. Have me diagnose and fix
the WHOLE thing solo — classify each janky frame by pipeline stage, isolate and fix each villain,
re-measure, and produce a before/after report I could show a client. Near-zero hints. Hard exam.
Then the verdict I need most: can I open an unfamiliar fancy/animation-heavy site cold, pinpoint
exactly which effects are browser-heavy, explain the pipeline stage each abuses, and predict weak-
hardware behavior (proceed to Track 5), or do I need to redo parts of Track 4?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

The page combines the track, e.g.:
- A **layout-triggering animation** (Module 4.2) — animate `transform` instead.
- A **paint-heavy effect** (animated shadow/gradient) — reduce paint area or rethink.
- **Layout thrashing** (Module 4.3) — batch reads/writes.
- A **non-passive scroll handler** (Module 4.7) — make passive, throttle.
- A **too-large DOM** causing expensive recalcs (Module 4.8) — trim/virtualize, contain (4.6).

The methodology you should now run unprompted:
1. **Baseline** under throttle (this is where weak-hardware behavior shows up).
2. **Paint flashing** → what's repainting unnecessarily. **Layer borders** → layer/composite issues. **FPS meter + Frames track** → where frames drop.
3. **For each janky frame, classify the pipeline stage**: Recalculate Style? Layout? Paint? Composite? That names the cause.
4. **Apply the matching fix** (composite-only animation, batch read/write, containment, passive scroll, smaller DOM), one at a time, re-measuring.
5. **Predict weak hardware:** anything layout/paint-heavy or layer-exploding that's "slightly janky" throttled will be unusable on a real cheap phone; composite-only + contained scales down.

The principle: **rendering diagnosis is classifying each janky frame by pixel-pipeline stage (recalc/layout/paint/composite) and applying the matching fix — and CPU/GPU throttling lets you predict weak-hardware behavior from your own machine.** This capstone is the heart of your stated goal: open a fancy site, find the browser-heavy effects, explain the mechanism, and predict how it degrades on weak devices. If you can do Exercise 2 cold, you own the rendering bucket.
</details>

---

*End of Track 4 — Rendering Performance & The Pixel Pipeline. 9 modules. You can now diagnose janky scroll and animations, name the offending effect and its pipeline stage on a busy page, keep animations on the GPU, and predict weak-hardware behavior — exactly the skill set you set out to build.*

*Next: Track 5 — Assets: Images, Fonts & Media, where the biggest real-world loading wins usually live.*
