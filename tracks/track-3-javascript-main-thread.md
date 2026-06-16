# TRACK 3 — JAVASCRIPT & THE MAIN THREAD

*The second bucket: "the page appeared, but it's frozen / janky / sluggish." This track is about the cost of your own code — how the browser's single main thread runs it, why long tasks freeze the UI, and (the part you asked about) what the JavaScript engine actually does to your code under the hood. This is where the deep-JS-engine courses finally pay off, and where you'll understand **why they exist**: you cannot write "blazingly fast JS" without knowing what makes V8 fast or slow.*

*Prerequisites: Tracks 0–1 (read a flame chart, measure honestly). A local Node server appears in a few modules. This track leans on three Frontend Masters courses, and here's what each is FOR:*

- ***JavaScript Performance** (Steve Kinney) — the bridge: the three buckets, the render pipeline, and a real intro to the V8 optimizing compiler and deoptimization.*
- ***Bare Metal JavaScript: The JavaScript Virtual Machine** (Miško Hevery) — **this is the answer to "why do deep-JS courses matter for performance?"** It teaches how high-level JS becomes low-level CPU instructions: memory operations, hidden classes, inline caching, and deoptimization. Understanding this is the difference between guessing and knowing why one function is 50x slower than another.*
- ***Blazingly Fast JavaScript** (ThePrimeagen) — applied hot-path optimization: garbage collection, memory profiling with the V8 profiler, data-structure choices, and (crucially) honest real-world benchmarking instead of deceptive micro-benchmarks.*

---

## Module 3.1 — The Main Thread: Why One Slow Function Freezes Everything

**Difficulty:** ●●○○○ · **Format:** Broken project · **Stack:** Vanilla JS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **JavaScript Performance** (Steve Kinney) → the intro and *"Thinking About Performance"* (the three buckets again, now with focus on the JS/main-thread bucket).
- **Web Performance Fundamentals, v2** → *"Improving Interaction to Next Paint"* and *"Yielding the Main Thread"* — the practical view of why a busy main thread blocks input and paint.

**Primary sources:**

- **MDN — The event loop** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Event_loop — *Focus on: run-to-completion, and that JS, layout, paint, and input all share one thread.*
- **web.dev — Optimize long tasks** — https://web.dev/articles/optimize-long-tasks — *Focus on: what a long task is (>50ms), how it blocks input and rendering, and the idea of yielding.*
- **MDN — Long Tasks API** — https://developer.mozilla.org/en-US/docs/Web/API/PerformanceLongTaskTiming — *Focus on: measuring long tasks programmatically.*

**Search prompts:**

- `"main thread single threaded javascript blocks ui"`
- `"long task 50ms blocks input rendering"`
- `"javascript run to completion event loop"`
- Ask an AI: *"Explain why the browser's main thread is the bottleneck: it runs JavaScript, layout, paint, AND input handling, one task at a time, run-to-completion. Why does a single synchronous function that takes 500ms freeze the entire page — no clicks, no scroll, no paint? What is a 'long task' and why is 50ms the threshold?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 3, Module 3.1, Exercise 1 — the main thread.
Completed: Tracks 0–2.

Build me a page with a button that runs a heavy SYNCHRONOUS computation (real work over a sizable
dataset) on the main thread, plus a spinner that's supposed to spin and a text input I can type in.
When I click the button, the page should visibly FREEZE — spinner stops, typing lags, clicks
ignored. The work is correct; it's just monolithic. Do NOT tell me why it freezes.

Have me measure: record a Performance trace (4x CPU throttle), find the long task, and confirm NO
frames painted and input was queued during it. Make me explain, before any fix, exactly why the
spinner froze. Then — without yet fixing it properly (that's the next modules) — have me prove the
diagnosis by shrinking the work and watching the freeze shrink. Exam me on: the single-threaded
model, run-to-completion, why one function blocks paint AND input, and what a long task is.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 3, Module 3.1, Exercise 2 (transfer) — main-thread
blocking, new scenario. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT page where the freeze is subtler — not one obvious button, but something that
blocks during what should be a smooth interaction (e.g. heavy synchronous work fired on every
keystroke or on scroll, so the UI stutters rather than fully locking). Don't tell me where the work
is. Have me, solo: reproduce the jank, record a trace, find the long tasks, and explain precisely
why the interaction stutters in terms of the main thread and the event loop. Exam me on why "it's
async-looking" (an event handler) doesn't make it non-blocking if the work inside is synchronous.
Verdict: do I deeply understand the main-thread bottleneck (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- Exercise 1: a single long synchronous loop on the main thread. While it runs, the event loop can't process anything else — so input events queue up unhandled, and the rendering steps (including the spinner's animation frames) never run. The page is frozen until the function returns. The trace shows one long task (often hundreds of ms) with no paint events inside it.
- Exercise 2: heavy synchronous work inside a frequent handler (keystroke/scroll), so instead of one big freeze you get repeated stutters — each handler invocation is a mini long-task that drops frames.
- The core model: the browser's **main thread is single-threaded and run-to-completion** — it runs one task fully before starting the next, and JS execution, style/layout, paint, and input handling all compete for it. A function that runs for 500ms is 500ms during which nothing else — not a click, not a scroll, not a single repaint — can happen.
- "Long task" = any task over 50ms; beyond that, input responsiveness visibly degrades (this is why INP and TBT care about them).
- Being inside an event handler doesn't make work async — synchronous work is synchronous wherever it runs.

The principle: **the main thread runs your JS, your layout, your paint, and your input handling one-at-a-time to completion — so any long synchronous task freezes the entire UI, because there's literally no other thread to handle the spinner, the scroll, or the click while your code runs.** Every fix in this track is ultimately about getting work *off* the main thread or *broken up* so the thread stays free. Recognizing a main-thread freeze in a trace is half of solving the "frozen page" bucket.

</details>

---

## Module 3.2 — Breaking Up Long Tasks: Yielding to the Event Loop

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla JS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **Web Performance Fundamentals, v2** → *"Yielding the Main Thread"* (with `setTimeout` and `requestAnimationFrame`) — Todd's concrete demonstration of returning control to the browser between chunks.

**Primary sources:**

- **web.dev — Optimize long tasks (yielding)** — https://web.dev/articles/optimize-long-tasks — *Focus on: chunking work and yielding with `setTimeout`, `scheduler.yield()`, and `isInputPending()`.*
- **MDN — `scheduler.yield()`** — https://developer.mozilla.org/en-US/docs/Web/API/Scheduler/yield — *Focus on: yielding to the event loop while keeping your place in the queue.*
- **web.dev — `isInputPending()`** — search the term — *Focus on: yielding only when input is actually waiting.*

**Search prompts:**

- `"break up long task chunk yield setTimeout scheduler.yield"`
- `"isInputPending yield to main thread"`
- `"javascript chunk heavy work keep ui responsive"`
- Ask an AI: *"How do I break a long synchronous task into chunks that keep the UI responsive? Explain yielding with `setTimeout(0)`, `await scheduler.yield()`, and `isInputPending()`. Crucially: does yielding make the work FASTER, or just more responsive? Why does the total work take the same (or slightly more) time but the page no longer freezes?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 3, Module 3.2, Exercise 1 — yielding to the event loop.
Completed: Tracks 0–2, Module 3.1.

Reuse the freezing pattern: build me a page that processes a large dataset synchronously on a click
and freezes the UI. My job is to keep the UI responsive by CHUNKING the work and yielding between
chunks — but do NOT show me the yielding code or tell me which technique to use.

Have me baseline (trace shows one long task, frozen spinner), then break the work into chunks that
yield (setTimeout, then scheduler.yield where available), re-trace, and confirm: the spinner now
spins, input is handled between chunks, and the long task is replaced by many short tasks. CRUCIAL:
make me measure that total work time did NOT meaningfully decrease — and explain why that's fine.
Exam me on: what yielding actually does, why responsiveness ≠ speed here, and where the work should
really go if it's truly heavy (foreshadow: a worker).
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 3, Module 3.2, Exercise 2 (transfer) — yielding, new
shape + a judgment call. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT heavy operation (e.g. rendering/processing a big list in batches) that freezes.
Have me, solo, make it responsive via chunked yielding, measuring before/after for BOTH
responsiveness (frames painted, input handled) and total time. Then the judgment call: have me
decide whether chunking is even the right fix here or whether this work belongs on a Web Worker
(next module) — and justify it. Don't tell me the answer. Exam me on the tradeoff: chunking keeps
the UI alive but still uses the main thread and can be slower overall; a worker frees the main
thread entirely but adds messaging cost. Verdict: do I understand yielding AND when it's the wrong
tool (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The fix is to split the loop into chunks and **yield** between them so the event loop can run rendering and input handling: `setTimeout(processNextChunk, 0)` (classic), or `await scheduler.yield()` (modern, keeps your priority in the queue), optionally guided by `isInputPending()` (yield only when input is waiting). The long task becomes many short tasks, and the spinner/input come back to life.
- The crucial measurement: **total work time does not meaningfully drop** — you did the same computation, just interleaved with other work; it may even be slightly *longer* due to scheduling overhead. What changed is **responsiveness**, not speed. This distinction trips up many developers and is exactly what an interviewer probes.
- The judgment call (Exercise 2): chunking is right when the work must touch the DOM or is moderate; but if the work is genuinely heavy and CPU-bound and doesn't need the DOM, it belongs on a **Web Worker** (next module), which frees the main thread entirely instead of just timesharing it.

The principle: **yielding breaks one long task into many short ones so the event loop can interleave rendering and input — trading a frozen page for a responsive one without making the work itself faster.** It's the right tool for moderate or DOM-touching work; for heavy pure computation, the better answer is to leave the main thread behind. Knowing *which* is the senior call.

</details>

---

## Module 3.3 — Web Workers: True Parallelism Off the Main Thread

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** Vanilla JS (Web Worker) + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **Blazingly Fast JavaScript** (ThePrimeagen) — the course's mindset on moving heavy work off the hot path and measuring it honestly applies directly here, even though workers are a browser primitive. (No single dedicated lesson — the relevant idea is "do less on the thread that matters.")

**Primary sources:**

- **MDN — Using Web Workers** — https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers — *Focus on: the worker model, `postMessage`, no DOM access, and that it's a real separate thread.*
- **MDN — Structured clone algorithm & Transferable objects** — https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm — *Focus on: data is COPIED to the worker; Transferables MOVE large buffers without copying.*
- **MDN — Comlink (pattern)** — https://github.com/GoogleChromeLabs/comlink — *Focus on: ergonomic worker messaging.*

**Search prompts:**

- `"web worker postMessage offload heavy computation"`
- `"structured clone vs transferable arraybuffer worker"`
- `"web worker keep ui responsive cpu bound"`
- Ask an AI: *"Explain Web Workers for performance: why moving CPU-bound work to a worker keeps the main thread completely free (real parallelism), unlike chunking which only timeshares the one thread. How does `postMessage` transfer data — structured clone (copy) vs Transferables (move) — and what are the constraints (no DOM, message-passing only)? When is the messaging cost worth it?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 3, Module 3.3, Exercise 1 — Web Workers, build
challenge. Completed: Tracks 0–2, Modules 3.1–3.2.

Take the heavy main-thread computation from earlier and give me a spec (no solution code) to move it
ONTO a Web Worker: the UI (spinner, input, a button) must stay PERFECTLY responsive while a genuinely
heavy computation runs in the worker and posts its result back. Include passing a large dataset in
and a large result out.

Review me like a senior. Measure first: prove (trace) the main thread is now idle during the
computation, unlike the chunking version. If I try to touch the DOM from the worker, don't just say
it's banned — ask me what a worker can and can't access and why. If I copy a huge buffer where I
could TRANSFER it, ask about structured clone vs Transferables and make me measure the messaging
cost. Exam me on: why a worker keeps the UI fully responsive when chunking only mitigates, and the
copy-vs-transfer tradeoff.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 3, Module 3.3, Exercise 2 (transfer) — worker, applied
+ measured. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT CPU-heavy feature (e.g. parsing/transforming a large file, or an image filter
over pixel data). Have me, solo, move it to a worker, keep the UI responsive, and MEASURE three
things: main-thread idle time during the work, the messaging/serialization cost (and whether
Transferables cut it), and end-to-end time vs the main-thread version. Then have me decide honestly
whether the worker was WORTH it for this case (sometimes the messaging cost eats the benefit for
small work). Don't tell me the answer. Exam me on the full tradeoff. Verdict: can I correctly decide
when to reach for a worker and prove it (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- A worker script that receives the dataset via `onmessage`, does the heavy work on its OWN thread, and `postMessage`s the result back — leaving the main thread free to render and handle input the whole time. The trace proves it: main thread idle during the computation (vs busy-but-chunked in Module 3.2).
- Data transfer: structured clone **copies** data to the worker (the worker gets its own copy); for large `ArrayBuffer`s, **Transferables** move ownership with no copy (near-zero cost) — measurable as a big drop in the messaging time. Know that after transfer, the sender can no longer use the buffer.
- Constraints: no DOM, no `window` — workers do computation and message back results that the main thread applies to the DOM.
- The honest tradeoff (Exercise 2): for genuinely heavy CPU work, a worker is a clear win (full main-thread freedom). For *small* work, the messaging + serialization overhead can exceed the computation, making it net slower — which is why you MEASURE rather than cargo-cult "always use a worker."

The principle: **a Web Worker runs on a real separate thread, so CPU-bound work there never blocks the main thread's rendering and input — true parallelism, unlike chunking's timesharing — at the cost of a message-passing boundary you must measure (copy vs transfer).** Workers are the correct home for heavy pure computation; the senior skill is recognizing when the work is heavy enough to justify the messaging cost.

</details>

---

## Module 3.4 — The JavaScript Engine: How V8 Runs Your Code

**Difficulty:** ●●●●○ · **Format:** Investigation · **Stack:** Node (with V8 flags) + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **Bare Metal JavaScript: The JavaScript Virtual Machine** (Miško Hevery) — **this is the course this whole module exists to motivate.** Watch the early sections on how high-level JS becomes CPU instructions: parsing, bytecode, and the journey through the VM. This is *why deep-JS courses matter for performance* — you're learning what the engine does to your code.
- **JavaScript Performance** (Steve Kinney) → the sections where Steve uses V8's internal flags to trace the optimizing compiler turning a function fast (and later, deoptimizing it).

**Primary sources:**

- **V8 blog — Ignition & TurboFan / "Celebrating 10 years"** — https://v8.dev/blog — *Focus on: the pipeline — parse → bytecode (Ignition interpreter) → optimized machine code (TurboFan) for hot functions.*
- **Mathias Bynens — "JavaScript engine fundamentals: Shapes and Inline Caches"** — https://mathiasbynens.be/notes/shapes-ics — *Read fully. The clearest explanation of hidden classes (shapes) and inline caches.*
- **V8 — Understanding the pipeline** — https://v8.dev/docs/ignition — *Focus on: bytecode and the tiering-up to optimized code.*

**Search prompts:**

- `"v8 ignition turbofan bytecode optimizing compiler"`
- `"v8 hidden classes shapes inline caches"`
- `"how javascript engine runs code jit compilation"`
- Ask an AI: *"Explain the V8 pipeline: how source becomes an AST, then bytecode run by the Ignition interpreter, then how 'hot' functions get compiled to optimized machine code by TurboFan (JIT). What are hidden classes (shapes) and inline caches, and why do they make repeated property access on consistently-shaped objects fast?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 3, Module 3.4, Exercise 1 — how V8 runs your code,
investigation. Completed: Tracks 0–2, Modules 3.1–3.3.

Guide me through SEEING the engine work, using Node with V8 trace flags (you tell me exactly which
flags and how to run, e.g. --trace-opt / --trace-deopt, and how to read the output). Give me a small
hot function called many times. Have me observe it get optimized (tier up to optimized code) and see
that optimized code is much faster. Do NOT explain the output for me — make me read the trace and
narrate what V8 did.

Also have me create two objects with the SAME shape vs DIFFERENT shapes and reason about hidden
classes / inline caches (you can point me to a way to observe or at least reason about the
difference). Exam me on: the parse→bytecode→optimized-code pipeline, what makes a function "hot,"
what a hidden class is, and what an inline cache caches. Demand layman + technical.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 3, Module 3.4, Exercise 2 (transfer) — engine model,
applied prediction. Apply the Exercise 2 rules: solo first; verdict.

Give me a few small functions/objects and have me, solo, PREDICT (before running) which will be
engine-friendly (stable shapes, monomorphic, optimizable) and which will fight the engine — then
verify with V8 trace flags or benchmarks. You design the cases so prediction is possible if I
understand shapes and inline caches. Don't tell me which is which. Exam me on: why consistent object
shapes matter, what "monomorphic vs polymorphic" inline caches mean, and what tiering-up requires.
Verdict: do I have a working mental model of the engine (proceed to deopts), or redo Exercise 1?
```

### 🔒 SPOILER — What This Is Teaching (the "why deep-JS courses matter" payoff)

<details>
<summary>Click only after you finish or give up</summary>

This is the module that answers your question about why deep-JS-engine knowledge matters for performance. What you should now hold:

- **The pipeline:** V8 parses your source to an AST, generates **bytecode** run by the **Ignition** interpreter (fast to start). Functions that run a lot ("hot") get compiled by **TurboFan** to optimized **machine code** (JIT — just-in-time compilation). This "tiering up" is why a function gets faster the more it runs — the engine invested in optimizing it.
- **Hidden classes (shapes):** objects with the same set of properties added in the same order share a hidden class. The engine uses this to access properties by a fixed memory offset (fast) instead of a dictionary lookup (slow). Create objects inconsistently and you spawn many shapes, defeating this.
- **Inline caches (ICs):** at a property-access site, V8 caches "for objects of this shape, the property is at this offset." If the site always sees ONE shape (**monomorphic**), it's blazing fast. If it sees many shapes (**polymorphic/megamorphic**), the cache thrashes and access slows down.
- **The payoff:** "blazingly fast JS" isn't magic — it's writing code the engine can optimize: stable object shapes, monomorphic call sites, types that don't change. You can't aim for that without knowing the engine exists and what it rewards.

The principle: **V8 runs your code through an interpreter and then JIT-compiles hot functions to machine code, relying on stable object shapes and monomorphic inline caches to go fast — so understanding the engine is what lets you write code that stays on its fast path instead of accidentally forcing slow lookups.** This is the entire reason the deep-JS-VM courses exist: performance at the language level is a conversation with the engine, and you have to know what it's listening for.

</details>

---

## Module 3.5 — Deoptimization: How to Make V8 Give Up on Your Code

**Difficulty:** ●●●●○ · **Format:** Broken project + investigation · **Stack:** Node (V8 flags) + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **Bare Metal JavaScript: The JavaScript Virtual Machine** (Miško Hevery) → the sections on **inline caching and deoptimization** specifically. This is the heart of the course and of this module.
- **JavaScript Performance** (Steve Kinney) → the lesson where Steve traces a **deoptimization** (e.g. the effect of deleting properties or changing types) using V8 flags.

**Primary sources:**

- **Mathias Bynens — Shapes and ICs (revisit)** — https://mathiasbynens.be/notes/shapes-ics — *Focus on: how polymorphism and shape changes degrade ICs.*
- **V8 blog — deoptimization / "optimizing for ...="** — https://v8.dev/blog — *Focus on: what causes a deopt and the cost of bailing back to bytecode.*
- **"What's up with monomorphism?"** (Vyacheslav Egorov) — https://mrale.ph/blog/2015/01/11/whats-up-with-monomorphism.html — *Deep but excellent on ICs and deopts.*

**Search prompts:**

- `"v8 deoptimization causes trace-deopt"`
- `"javascript polymorphic monomorphic megamorphic inline cache"`
- `"deleting object properties hurts performance v8"`
- Ask an AI: *"What causes V8 to DEOPTIMIZE a function — bail out of optimized machine code back to slower bytecode? Cover: changing an object's shape after creation (adding/deleting properties), mixing types at a call site (polymorphism), `arguments` misuse, and `try/catch` in older engines. Why is a deopt expensive, and how do I keep a hot function monomorphic and shape-stable?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 3, Module 3.5, Exercise 1 — deoptimization. Completed:
Tracks 0–2, Modules 3.1–3.4.

Build me a Node script with a hot function that LOOKS fine but is secretly deoptimized: it processes
objects that don't share a stable shape (built inconsistently, or mutated after creation, or a
property deleted), and/or a call site that receives mixed types — so V8 optimizes then bails out
repeatedly. Do NOT tell me what's causing the deopt.

Give me the V8 flags to trace optimization/deoptimization and a benchmark harness. Have me: confirm
the function is slower than it should be, use --trace-deopt to SEE the bailout, and trace it back to
the shape/type instability. Fix it (stabilize shapes, keep the call site monomorphic) and re-measure
the speedup. Hold the measure loop. Exam me on: what triggers a deopt, why it's costly, and
monomorphic vs polymorphic vs megamorphic.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 3, Module 3.5, Exercise 2 (transfer) — deopt hunting,
new cause. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT hot path with a different deopt trigger (e.g. a function that's monomorphic
until it suddenly receives a different type, or an object pattern that changes shape midway through
a loop). Have me, solo: benchmark, trace the deopt with V8 flags, identify the exact trigger, fix
it, and prove the speedup — AND state honestly whether the speedup matters for a real app or is a
micro-optimization (ThePrimeagen's warning: micro-benchmarks deceive; the real bottleneck is usually
"doing something too many times," not a single dot-access). Don't tell me the trigger. Exam me on
when engine-level optimization is worth chasing vs when it's a distraction. Verdict: can I hunt and
judge deopts maturely (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The deopt triggers (one per exercise): objects built with inconsistent shapes or **mutated after creation** (adding/deleting properties changes the hidden class); a hot call site that receives **mixed types** (becomes polymorphic/megamorphic, so the inline cache thrashes); deleting a property (forces a shape transition and often a dictionary-mode object). `--trace-deopt` shows V8 bailing the optimized function back to bytecode, then re-optimizing, then bailing again — the churn is the cost.
- The fix: keep hot-path objects **shape-stable** (initialize all properties up front, in the same order; never `delete`; don't add properties later) and keep call sites **monomorphic** (consistent types). Re-measure: the optimized, stable version is meaningfully faster, and the deopt trace goes quiet.
- The maturity lesson (built into Exercise 2, straight from ThePrimeagen): micro-benchmarks lie, and engine-level deopts are usually NOT your real bottleneck — "you're doing something too many times in too big a scope, or allocating memory you don't need" matters far more than a single deopt. Chase deopts when profiling actually points at a hot function; don't pre-optimize object shapes everywhere out of superstition.

The principle: **V8 optimizes hot functions assuming stable shapes and consistent types, and deoptimizes — expensively — when those assumptions break (shape changes, type mixing, property deletion); keeping hot paths monomorphic and shape-stable keeps them on the fast path.** But the senior judgment is knowing this matters only for genuinely hot code that profiling flagged — most performance wins are algorithmic and "do less," not engine micro-tuning.

</details>

---

## Module 3.6 — Memory & Garbage Collection: Allocation as a Cost

**Difficulty:** ●●●●○ · **Format:** Investigation + broken project · **Stack:** Node/Chrome (memory tools)

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **Blazingly Fast JavaScript** (ThePrimeagen) — the **garbage collection and memory profiling** material is the spine of this module: GC pauses, memory pressure, profiling allocations, and reducing allocation in hot paths.
- **JavaScript Performance** (Steve Kinney) → the memory/GC discussion.

**Primary sources:**

- **MDN — Memory management** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_management — *Focus on: reachability, mark-and-sweep, and that allocation/collection cost CPU time.*
- **V8 blog — Trash talk / Orinoco (the GC)** — https://v8.dev/blog/trash-talk — *Focus on: generational GC, minor (scavenge) vs major (mark-compact) collections, and GC pauses.*
- **Chrome DevTools — Memory & allocation profiling** — https://developer.chrome.com/docs/devtools/memory-problems — *Focus on: allocation timelines and finding what allocates the most.*

**Search prompts:**

- `"javascript garbage collection generational minor major pause"`
- `"reduce allocation hot path performance gc pressure"`
- `"chrome devtools allocation profiler memory timeline"`
- Ask an AI: *"Explain JS garbage collection's performance cost: generational GC (young generation scavenges vs old generation mark-compact), why GC runs on the main thread and causes pauses/jank, and why allocating lots of short-lived objects in a hot loop creates 'GC pressure.' How do I profile allocations and reduce them (reuse objects, avoid allocation in hot loops)?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 3, Module 3.6, Exercise 1 — memory & GC. Completed:
Tracks 0–2, Modules 3.1–3.5.

Build me a hot path (an animation loop or a data-processing loop) that ALLOCATES heavily — creating
many short-lived objects/arrays every iteration — causing GC pressure and periodic jank (GC pauses
showing up as dropped frames). Do NOT tell me allocation is the problem.

Have me: record a Performance trace and a memory/allocation profile, SEE the sawtooth memory pattern
and the GC pauses correlated with the jank, and trace the jank to the per-iteration allocations. Fix
it (reuse objects/buffers, hoist allocations out of the loop, avoid creating garbage) and re-measure
both memory churn and frame consistency. Exam me on: generational GC, why GC pauses cause jank, what
"GC pressure" is, and how reducing allocation smooths it.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 3, Module 3.6, Exercise 2 (transfer) — allocation in a
new hot path. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT allocation-heavy scenario (e.g. processing a stream of data, or a particle
system) that janks from GC. Have me, solo: profile allocations, find what's generating garbage,
reduce it (object pooling / reuse / choosing the right data structure), and prove smoother frames +
less GC. Then the judgment: have me state when allocation-reduction is worth the added code
complexity vs when the clarity is worth more than the micro-win. Don't tell me what's allocating.
Exam me on GC mechanics and the readability/perf tradeoff. Verdict: can I diagnose and reduce GC
pressure responsibly (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problem: a hot loop allocates many short-lived objects/arrays per iteration (e.g. `{x, y}` per particle per frame, or a new array each pass). These become garbage immediately, filling the **young generation**, which triggers frequent **minor GC** (scavenges); heavy churn eventually causes **major GC** (mark-compact) pauses. Because GC runs on the **main thread**, those pauses drop frames — visible as periodic jank correlated with memory sawteeth in the trace.
- The fix: **reduce allocation in the hot path** — reuse objects/arrays across iterations (object pooling), hoist allocations out of the loop, mutate in place instead of creating new objects, and pick data structures that don't churn. Re-measure: memory churn flattens, GC pauses shrink, frames smooth out.
- The mechanics: generational GC assumes most objects die young (the "generational hypothesis"), so it collects the young generation cheaply and often; but if you allocate garbage fast enough, even cheap collections add up and the occasional major GC stalls the thread.
- The maturity note (Exercise 2): allocation reduction trades readability for speed — worth it in genuinely hot paths (animation loops, high-frequency processing) that profiling flagged, but premature object-pooling everywhere is the wrong instinct.

The principle: **every allocation is future GC work, GC runs on the main thread and pauses it, so allocating heavily in a hot loop creates GC pressure that shows up as jank — and reducing allocation (reuse, pooling, in-place mutation) smooths it.** Memory performance is about creating less garbage where it matters; the skill is profiling to find those spots rather than micro-managing allocation everywhere.

</details>

---

## Module 3.7 — Data Structures & Algorithmic Cost in the Hot Path

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla JS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **Blazingly Fast JavaScript** (ThePrimeagen) — the **data structures** material (arrays vs sets vs maps vs linked lists) and the recurring theme: the real bottleneck is usually "doing something too many times," i.e. algorithmic, not micro.

**Primary sources:**

- **MDN — Map and Set** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map — *Focus on: O(1) lookup with Map/Set vs O(n) `Array.includes`/`find`.*
- **Big-O primer** — search "big o notation javascript array operations" — *Focus on: the cost of common operations and why O(n²) in a render path is deadly.*

**Search prompts:**

- `"array includes O(n) vs set has O(1) performance"`
- `"accidental O(n^2) nested loop javascript"`
- `"choosing data structure map set array performance"`
- Ask an AI: *"Show me how the wrong data structure creates quadratic blowups: `Array.includes`/`find` inside a loop (O(n²)) vs a `Set`/`Map` lookup (O(1) per check). Walk me through spotting accidental O(n²) in real code (a nested loop, or a `.find` inside a `.map`) and why it's invisible on small data but catastrophic at scale."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 3, Module 3.7, Exercise 1 — data structures & algorithmic
cost. Completed: Tracks 0–2, Modules 3.1–3.6.

Build me a feature that's fine on small data but grinds to a halt on large data because of an
ACCIDENTAL O(n²) — e.g. an `Array.includes`/`find` inside a loop, or a nested loop doing a lookup
that a Map/Set would make O(1). Provide a small dataset (feels fine) AND a large one (janks/freezes).
Do NOT tell me the complexity is the problem.

Have me: reproduce on the large dataset, profile to find the hot function, recognize the quadratic
pattern, and fix it with the right data structure (Map/Set for O(1) lookup), re-measuring across
dataset sizes to show the curve flatten. Exam me on: why O(n²) hides on small data, the cost of
common array ops, and when a Map/Set is the right call.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 3, Module 3.7, Exercise 2 (transfer) — algorithmic cost,
new instance. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT scenario with a different algorithmic trap (e.g. repeated sorting inside a loop,
or rebuilding a lookup structure every iteration instead of once, or deduping with includes). Have
me, solo: find the hot path by profiling (not by guessing — ThePrimeagen's point), identify the
algorithmic cost, fix it, and prove the improvement scales. Make me explain why this is almost always
a bigger win than any engine-level micro-optimization. Don't tell me the trap. Exam me on
complexity reasoning and on "profile, don't guess." Verdict: can I spot and fix algorithmic
bottlenecks (proceed to the track capstone), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The trap: an O(n²) pattern — `array.includes(x)` or `array.find(...)` inside a loop over the same array (each lookup is O(n), done n times), or a nested loop, or rebuilding a derived structure every iteration. On 100 items it's invisible; on 50,000 it's seconds of frozen main thread.
- The fix: replace linear lookups with a **`Set`** (membership) or **`Map`** (key→value), making each lookup O(1) and the whole operation O(n). Build the lookup structure ONCE, outside the loop. Re-measuring across sizes shows the quadratic curve become linear — the defining proof.
- The methodology (ThePrimeagen's core message): **profile to find the hot path; don't guess.** The bottleneck is almost always algorithmic — "doing something too many times" — not a clever micro-optimization. This is why this module comes after the engine modules: engine tuning is a rounding error next to fixing an O(n²).

The principle: **the largest JavaScript performance wins are algorithmic — replacing accidental O(n²) lookups with O(1) Map/Set access and doing work once instead of n times — and they dwarf engine-level micro-optimizations.** The expert reflex is to profile, find where time actually goes, and check the complexity of the hot path first. This is the highest-leverage skill in the whole track.

</details>

---

## Module 3.8 — The Main-Thread Diagnosis (JavaScript Capstone)

**Difficulty:** ●●●●● · **Format:** Investigation + broken project · **Stack:** Vanilla JS / Node + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- Revisit whichever of **JavaScript Performance**, **Blazingly Fast JavaScript**, and **Bare Metal JavaScript** covered the specific issues you found hardest in this track. This capstone integrates the whole track.

**Primary sources:**

- **Chrome DevTools — Performance: find the bottleneck (revisit)** — https://developer.chrome.com/docs/devtools/performance — *Focus on: Bottom-Up / Call Tree to attribute main-thread time to specific functions.*
- **web.dev — Optimize long tasks (revisit)** — https://web.dev/articles/optimize-long-tasks — *Focus on: the full toolkit (yield, worker, do less).*

**Search prompts:**

- `"diagnose main thread bottleneck flame chart bottom-up"`
- `"reduce javascript main thread work methodology"`
- Ask an AI: *"Give me a systematic method to diagnose a sluggish/janky page that's bottlenecked on JavaScript: record a trace, find the long tasks, use Bottom-Up/Call Tree to attribute time to functions, then classify each hot function (algorithmic? allocation/GC? deopt? just too much main-thread work?) and pick the right fix (better algorithm, less allocation, chunk/yield, or move to a worker)."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 3, Module 3.8, Exercise 1 — main-thread diagnosis,
JavaScript capstone. Completed: Tracks 0–2, Modules 3.1–3.7. Make it genuinely hard.

Build me an app that's janky/sluggish for SEVERAL JS reasons at once: an accidental O(n²) hot path,
heavy allocation causing GC jank, a long synchronous task that should yield or move to a worker, and
maybe a deoptimized hot function. Do NOT tell me how many problems exist or what they are.

This is a full main-thread diagnosis. Have me: record a trace, find the long tasks, use Bottom-Up/
Call Tree to attribute time to specific functions, and for EACH hot function classify the cause
(algorithm? allocation? deopt? too-much-work?) and apply the matching fix — one at a time,
re-measuring. Hold the loop ruthlessly. Long exam tying each fix to its mechanism. This rehearses
diagnosing a real frozen/janky page cold.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 3, Module 3.8, Exercise 2 (transfer) — diagnose a
DIFFERENT janky app solo. Apply the Exercise 2 rules: minimal hints; full verdict.

Build me a DIFFERENT app with a different mix of main-thread problems. Have me diagnose and fix the
WHOLE thing solo: trace, attribute time function-by-function, classify each bottleneck, fix with the
right tool (algorithm / allocation / yield / worker), and produce a before/after report. Near-zero
hints unless I'm genuinely stuck after a real attempt. Hard exam. Then the verdict I need: can I walk
up to an unfamiliar janky/frozen page and diagnose its main-thread bottlenecks cold (proceed to
Track 4 — rendering), or do I need to redo parts of Track 3?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

The app combines the track's lessons, e.g.:

- An **accidental O(n²)** hot path (Module 3.7) — usually the biggest single win.
- **Allocation-heavy** code causing GC jank (Module 3.6).
- A **long synchronous task** that should yield or move to a worker (Modules 3.2/3.3).
- Possibly a **deoptimized** hot function (Module 3.5).

The methodology you should now run unprompted:

1. **Record** a trace under throttling; baseline the numbers.
2. **Find the long tasks**, then use **Bottom-Up / Call Tree** to attribute main-thread time to specific functions (don't guess — ThePrimeagen's rule).
3. **Classify each hot function:** Is it slow because of *algorithm* (fix the complexity / data structure), *allocation* (reduce garbage), *deopt* (stabilize shapes/types), or just *too much work on the main thread* (chunk/yield, or move to a worker)?
4. **Apply the matching fix, one at a time, re-measure.** Order matters: fix the O(n²) first (biggest lever), then allocation, then yielding/worker, then engine-level tuning only if profiling still points there.

The principle: **diagnosing a JS-bottlenecked page is a systematic attribution: find the long tasks, attribute time to functions, classify each by cause, and apply the matching fix in order of leverage — algorithm and "do less" first, engine micro-tuning last.** This module is your end goal for the main-thread bucket: open a trace on a janky page and name what's eating the thread and how to fix it. If you can do Exercise 2 cold, you own this bucket.

</details>

---

*End of Track 3 — JavaScript & The Main Thread. 8 modules. You now understand why a page freezes, how to keep the main thread free (yield, workers), what the V8 engine does to your code (and why the deep-JS courses exist), and how to diagnose a janky page function-by-function.*

*Next: Track 4 — Rendering Performance & The Pixel Pipeline, where "janky scroll and animation" and your "which animation is killing this page on weak hardware" goal get solved.*
