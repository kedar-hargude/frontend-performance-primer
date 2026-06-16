# TRACK 9 — CAPSTONES: DIAGNOSE ANYTHING

*The finale. Everything you've built converges here into the exact skill you set out to acquire: open DevTools on **any** site — one you've never seen — read the network waterfall and the flame chart cold, pinpoint which request, which function, which animation, or which third party is the bottleneck, predict how it behaves on weak hardware, and prescribe the fix. These capstones are deliberately multi-cause and realistic: a slow real-world page is never slow for one reason, and here you must use all three buckets, the React knowledge, the perceived-performance lens, and the at-scale discipline together, under your own power.*

*Prerequisites: all of Tracks 0–8. These are the hardest modules in the primer by design. The Exercise 2 of each is run with minimal hints — this is your proving ground.*

*No single spine course — these revisit everything. Before each capstone, skim whichever Frontend Masters course covers the area you felt weakest on.*

---

## Module 9.1 — Diagnose a Slow-Loading Page (Network-Dominant)

**Difficulty:** ●●●●● · **Format:** Investigation + broken project · **Stack:** Full stack + Chrome/WPT

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- Skim **Web Performance Fundamentals, v2** (Todd Gardner) — the whole loading half. This capstone is the integration of Tracks 1, 2, and 5.

**Primary sources:**
- **web.dev — Optimize LCP (end to end)** — https://web.dev/articles/optimize-lcp — *Focus on: combining TTFB, resource load, and render delay into one diagnosis.*
- **Chrome DevTools — Network reference (revisit)** — https://developer.chrome.com/docs/devtools/network/reference — *Focus on: the full diagnostic toolkit.*

**Search prompts:**
- `"systematic slow page load diagnosis waterfall methodology"`
- Ask an AI: *"Give me a complete, repeatable methodology for diagnosing a slow-LOADING page from scratch: baseline the metrics + waterfall + filmstrip, walk the critical path, classify every slow/blocking request by cause (TTFB/server, size/compression, blocking, late discovery, caching, third party), and prescribe the matching fix — then prioritize by impact."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 9, Module 9.1, Exercise 1 — diagnose a slow-loading page,
capstone. Completed: Tracks 0–8. Make this a realistic, multi-cause LOADING disaster.

Build me a realistic page (local full-stack) that loads slowly for MANY loading-bucket reasons at once,
drawn from Tracks 1/2/5: bad TTFB, no compression, a render-blocking critical-path chain, oversized/
mis-formatted images, a lazy-loaded LCP image, broken caching, missing resource hints, and a heavy
third-party script. Do NOT tell me what or how many.

This is a full cold diagnosis. Have me: baseline (CWV + waterfall + WPT filmstrip), then methodically
walk every request, classify each problem by cause, prescribe + apply the matching fix ONE at a time,
re-measuring cumulatively, and produce a prioritized before/after report. Hold the measurement loop
ruthlessly. Long exam tying every fix to its mechanism. Treat this exactly like a real unfamiliar site.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 9, Module 9.1, Exercise 2 (transfer) — diagnose a DIFFERENT
slow-loading page solo. Apply the Exercise 2 rules: NEAR-ZERO hints; full verdict.

Build me a DIFFERENT realistic loading disaster with a different combination. Have me diagnose and fix
the ENTIRE thing solo — baseline, per-request classification, one-at-a-time fixes, cumulative
re-measurement, and a client-ready before/after report. Essentially no hints unless I'm truly stuck
after a real attempt. Hard exam. Verdict: am I ready to walk up to ANY unfamiliar slow-loading site and
diagnose it cold (proceed), or do I need to revisit specific loading tracks?
```

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

This integrates the loading half of the primer. The methodology you must now run unprompted on any page:
1. **Baseline** — CWV, the network waterfall, and a WebPageTest filmstrip (so you see the load as the user does).
2. **Walk the critical path and every request.** For each slow/blocking one, classify by cause: **server** (TTFB — 2.2), **size** (compression/format — 2.3/5.1), **blocking** (critical path — 2.1), **late discovery** (needs preload — 2.7), **caching** (re-downloaded — 2.5), **priority** (LCP image deprioritized — 5.3), or **third party** (8.1).
3. **Prescribe the matching fix per problem, apply one at a time, re-measure** cumulatively (so you know what each was worth, accounting for variance — 1.8).
4. **Prioritize and report** in before/after terms.

The principle: **diagnosing a slow-loading page cold is a disciplined walk of the waterfall — classify each request by failure mode, apply the matching fix, measure each — and it works on any site because the failure modes are finite and you now know them all.** This is half of your stated end goal, executed under your own power.
</details>

---

## Module 9.2 — Diagnose a Janky/Frozen Page (Main-Thread & Rendering Dominant)

**Difficulty:** ●●●●● · **Format:** Investigation + broken project · **Stack:** Full stack + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- Skim **JavaScript Performance** (Steve Kinney) and the **Bare Metal / Blazingly Fast** material you found hardest. Integration of Tracks 3 and 4.

**Primary sources:**
- **Chrome DevTools — Performance (revisit)** — https://developer.chrome.com/docs/devtools/performance — *Focus on: long tasks, Bottom-Up/Call Tree, and the Frames track together.*
- **web.dev — Optimize INP / long tasks (revisit)** — https://web.dev/articles/optimize-inp — *Focus on: attributing interaction slowness.*

**Search prompts:**
- `"diagnose janky frozen page main thread rendering methodology"`
- Ask an AI: *"Give me a methodology to diagnose a page that's janky/frozen/unresponsive (not a loading problem): record a trace under throttle, find long tasks and dropped frames, attribute each to a cause — algorithmic/allocation/deopt JS (main thread) or layout/paint/composite (rendering) — and apply the matching fix. How do I tell a main-thread problem from a rendering problem from the trace?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 9, Module 9.2, Exercise 1 — diagnose a janky/frozen page,
capstone. Completed: Tracks 0–8. Realistic, multi-cause RUNTIME disaster (not loading).

Build me a page that appears fine but is janky/frozen/unresponsive for MANY runtime reasons drawn from
Tracks 3/4: an accidental O(n^2) hot path, heavy allocation causing GC jank, a long synchronous task
that should yield/worker, a layout-triggering animation, layout thrashing, and a non-passive scroll
handler. Do NOT tell me what or how many.

Full cold runtime diagnosis. Have me: record traces under CPU throttle, find the long tasks and dropped
frames, and for EACH attribute the cause — is it main-thread JS (algorithm? allocation? deopt? too much
work?) or rendering (layout? paint? composite? scroll?) — then apply the matching fix one at a time and
re-measure. Hold the loop. Long exam tying each to its mechanism. Treat it like a real unfamiliar
sluggish app.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 9, Module 9.2, Exercise 2 (transfer) — diagnose a DIFFERENT
janky page solo. Apply the Exercise 2 rules: NEAR-ZERO hints; full verdict.

Build me a DIFFERENT runtime disaster with a different mix of main-thread and rendering problems. Have
me diagnose and fix the WHOLE thing solo — trace, classify each long task / dropped frame by cause, fix
in leverage order, re-measure, report. Essentially no hints. Hard exam. Verdict: can I diagnose ANY
unfamiliar janky/frozen page cold — separating main-thread from rendering causes and fixing each
(proceed), or do I need to revisit Tracks 3/4?
```

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

This integrates the runtime half (Tracks 3 + 4) — the other half of your stated goal. The methodology:
1. **Record a trace under CPU throttle** (so weak-hardware behavior shows); baseline.
2. **Find long tasks and dropped frames.** For each, determine the **bucket first**: is the time in **scripting** (yellow — main-thread JS) or **rendering** (purple Layout / green Paint / composite)?
3. **Within main-thread JS** (Track 3), classify: **algorithmic** (O(n²) — 3.7), **allocation/GC** (3.6), **deopt** (3.5), or just **too much work** (yield/worker — 3.2/3.3). Use Bottom-Up/Call Tree to attribute to functions.
4. **Within rendering** (Track 4), classify: **layout** (4.2), **layout thrashing** (4.3), **paint** (4.1), **composite/layers** (4.4), or **scroll** (4.7). Use Paint flashing + Layer borders + the Frames track.
5. **Fix in leverage order** (algorithm first; composite-only animations; passive scroll), one at a time, re-measure.

The principle: **diagnosing a janky/frozen page cold is determining the bucket per long-task/dropped-frame (scripting vs rendering), then classifying within it and applying the matching fix — and combined with 9.1 you can now diagnose any page across all three buckets.** This completes the core skill you set out to build.
</details>

---

## Module 9.3 — Diagnose a Slow React App (Framework-Dominant)

**Difficulty:** ●●●●● · **Format:** Investigation + broken project · **Stack:** React/Next.js + Chrome + React DevTools

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- Skim **React Performance, v2** (Steve Kinney). Integration of Track 6 with everything else.

**Primary sources:**
- **React docs — Profiler (revisit)** — https://react.dev/reference/react/Profiler
- **Next.js — performance docs (revisit)** — https://nextjs.org/docs/app/building-your-application/optimizing — *Focus on: the framework-level levers.*

**Search prompts:**
- `"diagnose slow react app methodology profiler renders bundle"`
- Ask an AI: *"Give me a methodology to diagnose a slow React/Next.js app end to end: combine the React Profiler (wasted/expensive renders, context cascades, list cost) with the browser Performance trace and Network (bundle size, hydration, server/client split) — classify each problem and fix in leverage order (colocation, memo, virtualize, code-split, server components, streaming)."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 9, Module 9.3, Exercise 1 — diagnose a slow React app,
capstone. Completed: Tracks 0–8. Realistic multi-cause React/Next.js disaster.

Build me a React/Next.js app slow for MANY reasons spanning Track 6 AND the lower tracks: a re-render
cascade from over-lifted state, a context cascade, memo defeated by unstable props, a huge
un-virtualized list, a bloated bundle, over-hydration/too much client JS, PLUS a lower-level issue
(an oversized image or a layout-thrashing effect). Do NOT tell me what or how many.

Full cold React diagnosis. Have me: use the React Profiler AND a Performance trace AND the Network
panel together, classify each problem by cause (render scope? props? context? list? bundle? hydration?
+ asset/rendering?), and fix in leverage order one at a time, re-measuring. Hold the loop. Long exam.
Treat it like inheriting an unfamiliar slow production React app.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 9, Module 9.3, Exercise 2 (transfer) — diagnose a DIFFERENT
slow React app solo. Apply the Exercise 2 rules: NEAR-ZERO hints; full verdict.

Build me a DIFFERENT slow React/Next.js app with a different mix. Have me diagnose and fix the WHOLE
thing solo — profile across all three tools, classify, fix in leverage order, re-measure, produce a
before/after report. Essentially no hints. Hard exam. Verdict: can I inherit ANY unfamiliar slow React
app and diagnose it cold across React-level and lower-level causes (proceed), or revisit Track 6?
```

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

This integrates your actual stack with everything beneath it. The methodology:
1. **Three tools together:** the **React Profiler** (renders, context cascades, list cost), a **Performance trace** (main-thread/commit cost, hydration), and the **Network panel** (bundle size, assets).
2. **Classify each problem:** React-level — **render scope** (colocation — 6.4), **unstable props defeating memo** (6.5), **context cascade** (6.6), **un-virtualized list** (6.7), **bundle** (6.8), **hydration/client-JS** (6.9/6.11) — and lower-level — **assets** (Track 5), **rendering** (Track 4), **main-thread JS** (Track 3).
3. **Fix in leverage order:** structural React fixes (colocation/composition) first, then memo/context, then virtualize/code-split/server-components, plus the lower-level fixes where they bite. One at a time, re-measure.

The principle: **diagnosing a slow React app cold means using the React Profiler, a Performance trace, and the Network panel together — because a slow React app is slow at the React level AND the asset/rendering/main-thread level — and fixing in leverage order.** This is the single most directly marketable diagnosis for your career, since you'll inherit React apps for a living.
</details>

---

## Module 9.4 — The Blind Audit: A Site You've Never Seen

**Difficulty:** ●●●●● · **Format:** Investigation (real public sites) · **Stack:** Real websites + Chrome/WPT/Lighthouse

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- None specific — this is pure application. Optionally skim Mastering Chrome Developer Tools, v4 for tool fluency.

**Primary sources:**
- **web.dev — Lighthouse + field data together** — https://pagespeed.web.dev — *Focus on: combining lab + field for a real site.*
- **WebPageTest (revisit)** — https://docs.webpagetest.org — *Focus on: filmstrip + waterfall on a real site.*

**Search prompts:**
- `"how to audit any website performance from scratch methodology"`
- Ask an AI: *"Give me a complete blind-audit checklist for any public website's performance: lab (Lighthouse) + field (CrUX/PSI) + WebPageTest filmstrip + a DevTools trace; identify the dominant bucket, the LCP element and its blockers, the worst third parties, the main-thread offenders, and the rendering issues; then produce a prioritized, evidence-backed report. How do I do this WITHOUT access to the source code?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 9, Module 9.4, Exercise 1 — blind audit of a REAL site.
Completed: Tracks 0–8. This is the literal rehearsal of my end goal, on real public sites I don't
control.

Act as my audit coach (you can't see the sites, so coach my METHOD, don't give answers). Have me pick
3 real public websites (you can suggest categories: a news site, an e-commerce site, a heavy web app).
For EACH, guide me through a full blind audit with only public tools: PageSpeed Insights (lab + field),
WebPageTest (filmstrip + waterfall), a DevTools Performance trace (throttled), and the Network panel —
identifying the dominant bucket, the LCP element + its blockers, the worst third parties, main-thread
offenders, and rendering issues. Make me WRITE a prioritized, evidence-backed report for each (top 3
fixes with expected impact), reasoning about the cause even without the source.

Exam me on my methodology and on whether my conclusions are grounded in evidence vs guesses. This is
exactly the skill I want: open a site I've never seen and diagnose it cold.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 9, Module 9.4, Exercise 2 (transfer) — blind audit, solo,
harder sites. Apply the Exercise 2 rules: minimal coaching; full verdict.

Have me, solo, blind-audit 2–3 DIFFERENT real sites of my own choosing (ideally including my company's
site or a competitor) — full public-tool audit, dominant bucket, LCP/blockers, third parties,
main-thread + rendering issues — and produce a client-ready prioritized report for each, entirely on my
own. Minimal coaching. Then exam me hard on evidence and prioritization. Verdict: can I walk up to ANY
real website I've never seen and produce a credible, evidence-backed performance audit cold — the exact
skill I set out to build (proceed), or do I need more reps?
```

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

This is the literal rehearsal of your stated goal, on real sites you don't control. The blind-audit method (no source access needed):
1. **Lab + field together** (PageSpeed Insights): the CrUX field data tells you the real-user truth (and which metric is failing at P75); Lighthouse gives the lab diagnosis. Disagreements are informative (1.3).
2. **WebPageTest filmstrip + waterfall:** *see* the load; find when meaningful content appears, the largest/blocking requests, and the third parties.
3. **DevTools Performance trace (throttled):** find long tasks and dropped frames; attribute main-thread and rendering cost — even on a site you don't own, the trace shows you *what* runs.
4. **Identify:** the **dominant bucket** (network/main-thread/rendering), the **LCP element** and what blocks it, the **worst third parties** (per-origin attribution — 8.1), the **main-thread offenders**, and **rendering issues** (Paint flashing).
5. **Produce a prioritized, evidence-backed report:** top 3 fixes with expected impact — reasoning about cause from observable behavior, not source.

The principle: **you can audit any public site's performance cold using only public tools — lab+field, WebPageTest, and a DevTools trace — because the symptoms are observable and the failure modes are the finite set you've mastered, so you reason from evidence to cause without ever seeing the code.** This is exactly the capability you wanted: open any site, read it cold, name what's wrong. It's also a literal consulting/interview deliverable.
</details>

---

## Module 9.5 — Predicting Weak-Hardware Behavior

**Difficulty:** ●●●●● · **Format:** Investigation + real device · **Stack:** Real low-end device + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- Revisit the throttling/device material in **Web Performance Fundamentals, v2**. Integration of Tracks 4 and 8.8.

**Primary sources:**
- **Chrome — Remote debugging (revisit)** — https://developer.chrome.com/docs/devtools/remote-debugging — *Focus on: profiling the real device.*
- **web.dev — CPU throttling and real devices** — search — *Focus on: what simulation captures vs misses.*

**Search prompts:**
- `"predict low end device performance from desktop trace"`
- Ask an AI: *"How do I predict how a page will behave on weak hardware from a desktop trace? What scales linearly with CPU throttling (most main-thread JS), what's WORSE on real devices than simulation (thermal throttling, memory pressure, real GPU limits on animations), and how do I form a confident prediction — then verify on a real low-end phone via remote debugging?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 9, Module 9.5, Exercise 1 — predicting weak-hardware
behavior. Completed: Tracks 0–8 (incl. 8.8 real-device setup). Uses a real low-end phone.

Build me (or reuse) a page with a mix of costs: heavy main-thread JS, a GPU-bound animation, and
notable memory use. Have me FIRST predict, from a desktop trace + progressive CPU throttling (4x, 6x),
how each part will behave on a real low-end phone — writing down specific predictions (which part
degrades most, what will be worse than the simulation suggests, what will thermally throttle). THEN
verify on the real phone via remote debugging (8.8) and compare prediction to reality.

Do NOT pre-explain. Make me confront where my prediction was right and where the real device surprised
me (thermal/memory/GPU). Exam me on: what scales with CPU throttle, what simulation under-predicts, and
how to form a confident weak-hardware prediction.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 9, Module 9.5, Exercise 2 (transfer) — predict then verify,
new page. Apply the Exercise 2 rules: solo first; verdict.

Take a DIFFERENT page (or a real site) and have me, solo: from a desktop trace + throttling, write a
confident prediction of its real low-end-device behavior (which bucket dominates on weak hardware, what
will be unusable, what will thermally degrade), THEN verify on the real phone and score my own
prediction. Don't pre-explain. Exam me on prediction accuracy and on the simulation-vs-reality gaps.
Verdict: can I confidently predict weak-hardware behavior and verify it — completing the goal of
predicting how any site behaves on the devices my users actually have (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What This Is Teaching (your stated prediction goal)

<details>
<summary>Click only after you finish or give up</summary>

This trains the specific prediction skill in your goal: "predict how it'll behave on weak hardware." What you should now be able to do:

- **What scales predictably with CPU throttling:** most **main-thread JavaScript** cost scales roughly linearly — a function that takes 50ms at 1x takes ~200–300ms at 4x–6x, so desktop throttling predicts it reasonably well. This is the part simulation handles.
- **What simulation UNDER-predicts (verify on real hardware):** **thermal throttling** (a real phone slows as it heats during sustained work — invisible in a short simulated trace), **memory pressure** (real GC stalls and even crashes on low-RAM devices that a fast laptop never hits), and **real GPU limits** (the composite-heavy or layer-exploding animations from Track 4 can be smooth under CPU throttle but melt a real cheap GPU, because CPU throttling doesn't throttle the GPU).
- **Forming a confident prediction:** use progressive throttling (4x, 6x) to bracket main-thread behavior, then flag the GPU/memory/thermal risks as "worse on real hardware," and **verify on the real phone** (8.8) to calibrate your intuition over time.

The principle: **you predict weak-hardware behavior by scaling main-thread cost with CPU throttling (which is roughly linear and predictable) while explicitly flagging what simulation under-predicts — thermal throttling, memory pressure, and real GPU limits — then verifying on a real low-end device to calibrate.** This completes the prediction half of your goal: not just diagnosing the current device, but confidently forecasting the experience of the users you can't see.
</details>

---

## Module 9.6 — The Multi-Bucket Disaster: Everything At Once

**Difficulty:** ●●●●● · **Format:** Investigation + broken project · **Stack:** Full stack + real device + all tools

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- None — this is the grand integration. Skim any course area you still feel shaky on.

**Primary sources:**
- Revisit your own notes/spoilers from the track capstones (2.8, 3.8, 4.9, 5.8, 6.14). This module assumes all of them.

**Search prompts:**
- `"holistic web performance audit all buckets prioritize"`
- Ask an AI: *"Give me a methodology for a page that's slow in ALL ways at once — loading, main-thread, rendering, React, third parties, and on weak hardware. How do I avoid getting lost: triage to find the dominant cost first, fix in order of user-impact-per-effort, and re-baseline after each major fix since fixing one bucket can change which bucket now dominates?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 9, Module 9.6, Exercise 1 — the multi-bucket disaster,
grand capstone. Completed: Tracks 0–8 + capstones 9.1–9.5. Make this the hardest module in the primer.

Build me a single realistic app that is slow in EVERY way at once: loading problems (TTFB, compression,
critical path, assets, caching), main-thread problems (O(n^2), allocation, long tasks), rendering
problems (layout/paint/scroll), React problems (re-render cascades, no virtualization, bundle,
hydration), third-party tax, AND it's brutal on weak hardware. Do NOT tell me anything about what's
wrong.

This is everything. Have me run a DISCIPLINED holistic audit: triage to find the dominant cost first
(don't get lost), fix in order of user-impact-per-effort, and CRUCIALLY re-baseline after each major
fix because the dominant bucket SHIFTS as you fix things. Verify the hardest parts on a real low-end
device. Produce a full prioritized before/after report. Hold the loop ruthlessly. The longest, hardest
exam in the primer, tying everything together.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 9, Module 9.6, Exercise 2 (transfer) — a DIFFERENT
multi-bucket disaster, solo. Apply the Exercise 2 rules: ESSENTIALLY NO hints; the final verdict.

Build me a DIFFERENT everything-at-once disaster with a different composition. Have me diagnose and fix
the ENTIRE thing solo, holistically — triage, fix in impact order, re-baseline as buckets shift, verify
on real hardware, and produce a complete client-ready report. No hints unless I'm genuinely stuck after
a serious attempt. The hardest exam. Then the FINAL verdict of the whole primer: am I now a frontend
performance expert who can open any site, diagnose anything across every bucket, predict weak-hardware
behavior, and prescribe + prove the fixes — or are there specific areas to revisit?
```

### 🔒 SPOILER — What This Is Teaching (the grand integration)

<details>
<summary>Click only after you finish or give up</summary>

This is the whole primer in one module. The holistic methodology, beyond any single bucket:
1. **Triage first — find the dominant cost.** Don't dive into the first problem you see. A quick pass (Lighthouse + a trace + the waterfall) tells you whether loading, main-thread, rendering, React, or third parties dominates *right now*. Fix the biggest lever first.
2. **Fix in order of user-impact-per-effort.** Not hardest-first, not easiest-first — biggest felt improvement per unit work. A 2MB image (one-line fix, huge LCP win) beats a clever memo refactor (much work, small win).
3. **Re-baseline after each major fix — because the dominant bucket SHIFTS.** Once you fix the network bottleneck, the main thread might become the limiter; once you fix that, rendering might dominate. The single most important holistic insight: **the bottleneck moves**, so you re-measure and re-triage rather than blindly working a pre-made list.
4. **Verify the hard parts on real hardware** (9.5/8.8) — especially animations and memory.
5. **Report** the cumulative before/after across all buckets.

The principle: **a page slow in every way is conquered by triage, not heroics — find the dominant cost, fix in impact-per-effort order, and re-baseline after each major fix because the bottleneck shifts as you remove it — verifying the hard parts on real hardware.** This is the complete expert skill: not knowing a hundred fixes, but knowing how to find the *right* one *next*, repeatedly, until the page is fast for real users on real devices. If you can do Exercise 2 cold, you are the engineer you set out to become.
</details>

---

## Module 9.7 — The Live Diagnosis: Think Aloud Under Pressure (Interview Rehearsal)

**Difficulty:** ●●●●● · **Format:** Simulated interview · **Stack:** Any site + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- None. This rehearses presenting the skill, not learning new content.

**Primary sources:**
- Your own track spoilers — this module is about *articulating* what you now know under pressure.

**Search prompts:**
- `"frontend performance interview questions senior diagnose"`
- Ask an AI: *"Act as a senior interviewer for a frontend performance role. Ask me to diagnose a described slow page out loud, probe my methodology, push back on hand-wavy answers, and ask the deep follow-ups (why does animating transform avoid layout? what does hydration cost? how do you know your fix worked?). Score my answers like a real interview."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 9, Module 9.7, Exercise 1 — live diagnosis interview
rehearsal. Completed: Tracks 0–8 + capstones 9.1–9.6.

Act as a tough senior/staff interviewer for a frontend performance role. Run a realistic interview: (1)
ask me conceptual deep-dives (the render pipeline, why transform is composite-only, hydration cost,
percentiles vs averages, what causes a deopt, lab vs field); (2) describe a slow page and make me
diagnose it OUT LOUD, narrating my methodology; (3) push back hard on any hand-wavy or unmeasured claim
— demand the mechanism and how I'd prove the fix worked. Don't let me off easy.

Score me honestly at the end: where I sounded like an expert, where I was vague, and exactly what to
tighten. This is rehearsal for the interviews that get me to senior + the salary I want.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 9, Module 9.7, Exercise 2 (transfer) — full mock loop.
Apply the Exercise 2 rules: I drive; you verify and score.

Run a DIFFERENT, harder mock interview loop: a rapid-fire concept round, a live "here's a janky page,
talk me through your diagnosis" round, and a system-design-flavored "how would you keep this product
fast as the team scales" round (Track 8). Grade each round, name my weakest area, and give me a final
readiness verdict: am I interview-ready for a senior frontend performance role, or which track's
concepts should I sharpen first? Be honest — a soft verdict helps no one.
```

### 🔒 SPOILER — What This Is Teaching (the career payoff)

<details>
<summary>Click only after you finish or give up</summary>

This module converts capability into the thing that changes your salary: **the ability to demonstrate the skill, out loud, under pressure.** What it rehearses:

- **Conceptual fluency:** explaining mechanisms crisply (the pixel pipeline, why `transform` is composite-only, hydration cost, percentiles vs averages, deopts, lab vs field) — in both layman and technical terms, exactly as the primer's exams trained.
- **Live diagnosis narration:** talking through a methodology clearly — baseline, find the dominant cost, classify, fix, measure — so an interviewer sees a systematic expert, not someone guessing.
- **Defending claims with evidence:** never saying "this is faster" without "here's the baseline, here's the result, here's the mechanism, here's how I'd verify in the field." The interviewer pushing back is the test of whether your knowledge is real.
- **System-level thinking** (Track 8): answering "how do you keep a product fast at scale" with budgets, CI, RUM/SLOs, and ownership — the staff-level signal.

The principle: **expertise that you can't articulate under pressure doesn't get you hired or promoted — so the final skill is narrating a systematic, evidence-backed diagnosis out loud and defending it against tough follow-ups.** You built the capability across eight tracks; this rehearses *presenting* it. That presentation is what converts the skill into the senior role and the ₹4–5L+ compensation you set out to reach.

---

## 🎓 PERFORMANCE PRIMER COMPLETE

If you can do the Exercise 2 of every Track 9 capstone cold — diagnose a loading disaster, a janky page, a slow React app, a blind real-site audit, predict weak-hardware behavior, conquer a multi-bucket disaster, and articulate it all in an interview — then you have become exactly what you set out to be: a frontend performance expert who can open DevTools on any site, read the waterfall and the flame chart cold, pinpoint the bottleneck across every bucket, predict how it behaves on the cheap phones your users actually hold, and prescribe and prove the fix.

That is a rare, durable, AI-resistant skill, and it is precisely the kind of expertise that commands a senior/staff title and the compensation you're aiming for. Go open a slow site and enjoy seeing straight through it.
</details>

---

*End of Track 9 — Capstones: Diagnose Anything. 7 modules. You can now diagnose any page across every bucket, on any hardware, and prove it under pressure.*

*End of the Opus Frontend Performance Primer.*
