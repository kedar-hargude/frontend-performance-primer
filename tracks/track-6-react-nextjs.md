# TRACK 6 — REACT & NEXT.JS PERFORMANCE AT SCALE

*Your actual stack, at the depth that commands senior salaries. You write React for a living; this track makes you the person who understands its performance from the rendering model up — why components re-render, what reconciliation costs, when memoization helps and when it's noise, how to render 100k rows smoothly, and how hydration and server components change the whole performance equation. By the end you can profile any React app and name exactly why it's slow.*

*Prerequisites: Tracks 0–4 (measure honestly, read a flame chart, understand the main thread and the pixel pipeline). React + Tailwind, with Next.js for the framework half.*

*Spine course: **React Performance, v2** (Steve Kinney) — Fiber and reconciliation, the causes of re-renders, `memo`/`useMemo`/`useCallback`, virtualization, code-splitting, Suspense and hydration, transitions, and the React 19 compiler. THE course for this track; watch the relevant section before each module. **JavaScript Performance** (Steve Kinney) and the React DevTools Profiler underpin it.*

---

## Module 6.1 — The React Render Model: Why Components Re-render

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** React + Tailwind (Vite) + React DevTools

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **React Performance, v2** (Steve Kinney) → the opening sections on **how React renders** and **what causes a re-render** (state change, parent re-render, context change). The foundation of the entire track.

**Primary sources:**

- **React docs — Render and Commit** — https://react.dev/learn/render-and-commit — *Focus on: trigger → render → commit, and that "render" is React calling your component, not touching the DOM.*
- **Josh Comeau — Why React Re-renders** — https://www.joshwcomeau.com/react/why-react-re-renders/ — *Read fully. The clearest explanation of the real re-render triggers (and the myths).*
- **React docs — State as a snapshot** — https://react.dev/learn/state-as-a-snapshot — *Focus on: how a state change schedules a re-render.*

**Search prompts:**

- `"why react component re-renders parent state props"`
- `"react render vs commit phase"`
- Ask an AI: *"Explain precisely what causes a React component to re-render: its own state changing, its parent re-rendering, or a context it consumes changing. Bust the myth that 'a component only re-renders when its props change' — by default a child re-renders whenever its parent does, regardless of props. What's the difference between the render phase and the commit phase?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.1, Exercise 1 — the React render model.
Completed: Tracks 0–5. I write React professionally; probe my mental model of WHY things re-render,
don't over-explain JSX.

Build me a React + Tailwind app where far more re-rendering happens than necessary: a state change
high in the tree re-renders large unrelated subtrees, and components re-render every time their
parent does even though "nothing changed." Add visible re-render indicators (or rely on the Profiler).
Do NOT tell me the cause.

Have me use the React DevTools Profiler + "Highlight updates" to SEE which components re-render and
WHEN. Make me explain, before any fix, the actual re-render trigger for each (own state? parent
render? context?). Then have me predict which components will re-render on a given interaction. Exam
me on: the three re-render triggers, the parent-re-renders-child default, and render vs commit.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.1, Exercise 2 (transfer) — predict the
re-renders. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT component tree with state, props, and context in play. Have me, solo, BEFORE
profiling, predict exactly which components will re-render on each of several interactions — then
verify with the Profiler. You design it so prediction is possible if I understand the model. Don't
tell me the answers. Exam me on the triggers and on why "props didn't change" doesn't stop a re-render
by default. Verdict: do I deeply understand the React render model (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The core thing most developers get wrong: **a component re-renders when (1) its own state changes, (2) its parent re-renders, or (3) a context it consumes changes** — and crucially, **a child re-renders whenever its parent does, by default, regardless of whether its props changed.** "It only re-renders if props change" is a myth (true only with `React.memo`).
- The app shows: a state change near the top re-renders the whole subtree; children re-render with their parent even though their own props/state are unchanged. Profiler + "Highlight updates" makes it visible.
- Render vs commit: "render" = React calls your function to compute the new tree (CPU cost, even if output is identical); "commit" = React applies minimal DOM changes. A re-render isn't automatically a DOM update — but it IS wasted CPU if output is unchanged.

The principle: **a React component re-renders due to its own state, its parent rendering, or a context change — and the parent-renders-child default means re-renders cascade down regardless of props.** Understanding this precisely is the prerequisite for every React optimization; without it, memoization is cargo-culting.

</details>

---

## Module 6.2 — Reconciliation, the Virtual DOM & Keys

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + Tailwind + React DevTools

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **React Performance, v2** (Steve Kinney) → the **Fiber / reconciliation** sections — how React diffs the tree, what the virtual DOM does, and how keys drive list reconciliation.

**Primary sources:**

- **React docs — Preserving and resetting state** — https://react.dev/learn/preserving-and-resetting-state — *Focus on: same-component-same-position preserves state; keys control identity.*
- **React docs — Rendering lists (keys)** — https://react.dev/learn/rendering-lists#keeping-list-items-in-order-with-key — *Focus on: why index keys break on reorder/insert.*
- **React (legacy) — Reconciliation** — https://legacy.reactjs.org/docs/reconciliation.html — *Focus on: the diffing heuristics.*

**Search prompts:**

- `"react reconciliation fiber virtual dom diffing"`
- `"react key index anti-pattern reorder performance"`
- Ask an AI: *"Explain React reconciliation: what the virtual DOM is, how React diffs old and new trees to compute minimal DOM updates, and how keys identify list items across renders. Why does array-index-as-key cause both correctness bugs AND extra DOM work when a list reorders or items are inserted? What is Fiber?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.2, Exercise 1 — reconciliation & keys.
Completed: Tracks 0–5, Module 6.1.

Build me a React + Tailwind app with a large, frequently-reordered/filtered list that uses array
INDEX as the key — so reconciliation does excessive DOM work (and state bleeds between items) on
reorder/insert. Do NOT name the cause.

Have me profile the list operations (Profiler + a Performance trace), observe the excessive commit
work and state bleeding, and trace it to the keying. Fix it with stable unique keys, re-measure the
commit cost on reorder. Exam me on: what reconciliation does, how keys let React match elements across
renders, why index keys force unnecessary unmount/remount + DOM work on reorder, and what Fiber is.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.2, Exercise 2 (transfer) — reconciliation,
new scenario. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT reconciliation problem (e.g. items prepended, or a conditional that swaps
component types at a position causing remounts, or a key collision). Have me, solo: profile, identify
the unnecessary reconciliation work, fix it, and prove the reduced commit cost. Don't tell me the
cause. Exam me on the diffing heuristics and key identity. Verdict: do I understand reconciliation and
keys for performance (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problem: a large list keyed by **array index**. On reorder/filter/insert the index→item mapping changes, so React (which matches by key+position) thinks different items changed than actually did — forcing extra DOM mutations, re-mounting components that didn't need it, and state bleeding to the wrong rows.
- The fix: a **stable, unique key** (item id) so React matches each element to the same logical item across renders, minimizing DOM work and preserving per-item state. Re-measure: commit cost on reorder drops.
- Reconciliation: React diffs a virtual tree using heuristics (different type → replace subtree; same type → update in place; keyed children → match by key) and commits the **minimal** real DOM changes. **Fiber** is the reconciler architecture that makes the work interruptible (enabling concurrent features, Module 6.12).

The principle: **reconciliation diffs the virtual tree to compute minimal DOM updates, and keys are how React identifies elements across renders — stable unique keys let it reuse work, index keys force DOM churn and state bugs on reorder.** Understanding the diff lets you reason about commit cost, not just render cost.

</details>

---

## Module 6.3 — The React DevTools Profiler: Finding Wasted Renders

**Difficulty:** ●●●○○ · **Format:** Investigation · **Stack:** React + Tailwind + React DevTools

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **React Performance, v2** (Steve Kinney) → the **React DevTools Profiler** sections — recording, the flame graph and ranked chart, and finding components that rendered unnecessarily.

**Primary sources:**

- **React docs — React Developer Tools / Profiler** — https://react.dev/learn/react-developer-tools — *Focus on: recording a profile and reading commits.*
- **React docs — `<Profiler>`** — https://react.dev/reference/react/Profiler — *Focus on: `actualDuration` vs `baseDuration`.*

**Search prompts:**

- `"react devtools profiler flame graph ranked wasted renders"`
- `"react profiler why did this render"`
- Ask an AI: *"Teach me the React DevTools Profiler: recording an interaction, reading the flame graph (which components rendered and how long) and the ranked chart, using 'Highlight updates' to see re-renders live, and enabling 'Record why each component rendered.' How do I distinguish a NECESSARY render from a WASTED one?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.3, Exercise 1 — the React Profiler.
Completed: Tracks 0–5, Modules 6.1–6.2.

Build me a React + Tailwind app that FEELS sluggish on interaction because of wasted renders: typing
in one input re-renders a large unrelated part of the tree; an expensive component re-renders on
unrelated state changes. Make the waste real (perceptible or under CPU throttle). Do NOT tell me
what's wasteful.

Pure Profiler practice. Have me record interactions, read the flame graph + ranked chart, enable "why
did this render," and IDENTIFY which renders are wasted and why — without fixing yet. Make me
distinguish necessary from wasted renders. Exam me on: reading the flame graph, actualDuration vs
baseDuration, and how to tell a wasted render from a needed one.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.3, Exercise 2 (transfer) — profile a new app.
Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT sluggish app. Have me, solo: profile it, find the top wasted-render offenders,
explain WHY each rendered (using "why did this render"), and rank them by cost — a prioritized fix
list (fixes come later). Don't tell me the offenders. Exam me on Profiler literacy and prioritizing by
measured cost, not guesses. Verdict: can I profile a React app and find the wasted renders cold
(proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

Instrument training — the React equivalent of Track 0's flame-chart bootcamp. You should now be able to:

- **Record** an interaction and read the **flame graph** (each bar = a component that rendered; width = time) and the **ranked chart** (sorted by render time).
- Enable **"Record why each component rendered"** so the Profiler names the trigger (props/state/parent/context) — turning "it re-rendered" into "it re-rendered *because*."
- Use **"Highlight updates"** to see re-render cascades live.
- Distinguish **`actualDuration`** (this render's time) from **`baseDuration`** (estimate without memo), and a **necessary** render from a **wasted** one (rendered but identical output).
- **Prioritize by measured cost** — fix components that render often AND take long.

The principle: **the React Profiler turns re-rendering from invisible to measurable — which components rendered, how long, and why — so you fix wasted renders on evidence, not superstition.** Always profile before optimizing.

</details>

---

## Module 6.4 — State Colocation: The Best Optimization

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** React + Tailwind + React DevTools

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **React Performance, v2** (Steve Kinney) → the **state placement** material — reducing re-render scope by moving state down (colocation/lifting content as the first-line fix before memoization).

**Primary sources:**

- **Kent C. Dodds — State Colocation will make your React app faster** — https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster — *Read fully.*
- **Dan Abramov — Before You memo()** — https://overreacted.io/before-you-memo/ — *Focus on: moving state down and lifting content up (composition).*
- **React docs — Sharing state** — https://react.dev/learn/sharing-state-between-components — *Focus on: closest-common-ancestor placement.*

**Search prompts:**

- `"react state colocation move state down performance"`
- `"react lift content up children prop avoid re-render"`
- Ask an AI: *"Explain state colocation: why moving state DOWN to the smallest component that needs it shrinks the re-render scope. Then the 'lift content up / pass children' pattern — how passing an expensive subtree as `children` keeps it from re-rendering when the stateful parent updates. Why are these better first moves than memoization?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.4, Exercise 1 — state colocation. Completed:
Tracks 0–5, Modules 6.1–6.3.

Build me a React + Tailwind app where state lives too HIGH: an input's value (or a hover/open toggle)
held in a top-level component, so every keystroke re-renders a large expensive subtree that doesn't
care about it. Do NOT name the cause.

Have me profile (every keystroke re-renders the world), then fix it WITHOUT memoization first: move
state DOWN into the small component that needs it (colocation), and/or lift the expensive subtree UP
and pass it as `children`. Re-measure the re-render scope. Exam me on: why colocation shrinks scope,
how passing `children` avoids re-rendering, and why these beat memo as first moves.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.4, Exercise 2 (transfer) — colocation, new
case. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT app with over-lifted state causing a broad re-render cascade (a form, a filter
bar, a hover state held too high). Have me, solo: profile, decide whether to move state down or lift
content up (or both), apply with NO memoization, prove the scope shrank. Don't tell me the fix. Exam
me on choosing colocation vs composition and why structural fixes beat memo. Verdict: can I fix
re-render scope structurally (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problem: state held too high re-renders the **entire subtree below** on every change, including expensive components that don't use it.
- The fixes (no memo):
  - **Colocation — move state down:** put state in the smallest component that needs it; a change now re-renders only that tiny subtree.
  - **Lift content up / pass `children`:** pass an expensive sibling as `props.children`; because `children` is created in the grandparent and passed as a stable prop, the stateful parent re-rendering doesn't re-render it.
- Re-measure: the scope collapses from the whole tree to a small piece. These are free (no comparison cost, no deps, no staleness) and often eliminate the problem outright.

The principle: **the cheapest re-render is the one that never happens — moving state down and lifting content up shrink the re-render scope structurally, and they're the correct FIRST optimization, before memoization.** Many "memo everything" situations dissolve once state lives in the right place.

</details>

---

## Module 6.5 — Memoization Done Right: memo, useMemo, useCallback

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + Tailwind + React DevTools

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **React Performance, v2** (Steve Kinney) → the **`memo` / `useMemo` / `useCallback`** sections — what each does, referential equality, and why a new inline function/object prop defeats `memo`.

**Primary sources:**

- **React docs — `memo`** — https://react.dev/reference/react/memo — *Focus on: skipping re-renders on referentially-equal props, and what defeats it.*
- **React docs — `useMemo` / `useCallback`** — https://react.dev/reference/react/useMemo — *Focus on: caching a value / function identity, and "should you add it everywhere?"*
- **Dan Abramov — Before You memo()** — https://overreacted.io/before-you-memo/ — *Focus on: when memo is wasted overhead.*

**Search prompts:**

- `"react.memo not working new function object prop"`
- `"usememo usecallback when actually needed referential equality"`
- Ask an AI: *"Explain when `React.memo`, `useMemo`, and `useCallback` actually help vs are wasted overhead. Why does a new inline function/object prop defeat a `memo`'d child? How do `useCallback`/`useMemo` stabilize references? Why is memoization NOT free, so sprinkling it everywhere can hurt?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.5, Exercise 1 — memoization done right.
Completed: Tracks 0–5, Modules 6.1–6.4.

Build me a React + Tailwind app with memoization BROKEN and MISUSED: a `React.memo` child that still
re-renders because it gets a new inline function/object prop each render; an expensive computation
re-run every render that SHOULD be useMemo'd; and useMemo/useCallback sprinkled pointlessly with
wrong/empty deps (some stale, some pure overhead). Do NOT name the issues.

Have me profile, then: fix the broken memo by stabilizing props, add memoization ONLY where
measurement justifies it, remove the pointless ones, confirm with the Profiler. Re-measure. Exam me
on: referential equality, why a new function/object defeats memo, what useMemo/useCallback cache, and
the cost of over-memoizing.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.5, Exercise 2 (transfer) — memo judgment.
Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT app mixing real memoization needs with memoization noise. Have me, solo: profile
to decide WHERE memo/useMemo/useCallback genuinely help (measured), apply only those, fix any broken
memo (unstable props), remove the wasteful ones, prove the net improvement. The trap: memoizing
everything is as wrong as memoizing nothing. Don't tell me where. Exam me on the decision logic and
referential equality. Verdict: can I apply memoization surgically from measurement (proceed), or redo
Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- A `React.memo` child re-rendering anyway because the parent passes a **new inline function** (`onClick={() => ...}`) or **object/array** every render — a new reference fails the shallow prop comparison. Fix: `useCallback`/`useMemo` to stabilize.
- An **expensive computation** re-run every render → `useMemo` with correct deps.
- **Pointless memoization** on trivial values (overhead > benefit) and wrong/empty deps (stale or no-op). Memoization isn't free — each adds a comparison and memory.
- The method: **profile first**, memoize only where a measured expensive/wasted render justifies it, and ensure the memo isn't defeated by an unstable prop upstream.

The principle: **`React.memo` only skips a re-render when props are referentially equal, so unstable function/object props silently defeat it; `useMemo`/`useCallback` provide that stability or cache real work — but memoization has its own cost, so it's a measured, surgical tool, applied after colocation/composition.**

</details>

---

## Module 6.6 — Context Performance: The Re-render Cascade

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + Tailwind + React DevTools

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **React Performance, v2** (Steve Kinney) → the **Context** performance section — why every consumer re-renders on a context value change, and how to split/stabilize.

**Primary sources:**

- **React docs — Passing data deeply with context** — https://react.dev/learn/passing-data-deeply-with-context — *Focus on: every consumer re-renders when the provider value changes by reference.*
- **React docs — Scaling up with reducer + context** — https://react.dev/learn/scaling-up-with-reducer-and-context — *Focus on: separating state and dispatch contexts.*

**Search prompts:**

- `"react context re-render all consumers performance"`
- `"split context value memo provider re-render"`
- Ask an AI: *"Why does every consumer of a React Context re-render when the provider's `value` changes — and why does passing a NEW object as `value` each render re-render ALL consumers every time? Fixes: memoize the value, split one big context into multiple by concern, separate state from dispatch."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.6, Exercise 1 — context performance.
Completed: Tracks 0–5, Modules 6.1–6.5.

Build me a React + Tailwind app where ONE context holds several unrelated values in a single object
passed as `value` (recreated each render), so EVERY consumer re-renders on ANY change — even consumers
using one unchanging slice. Deep tree, several consumers. Do NOT name the cause.

Have me profile (the cascade is obvious), then fix it: memoize/stabilize the value, and SPLIT the
context by concern so a consumer only re-renders for its slice (and/or separate state from dispatch).
Re-measure consumer re-renders. Exam me on: why context propagates by reference identity, why one big
value object is the trap, and how splitting/memoizing fixes it.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.6, Exercise 2 (transfer) — context, new
shape. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT context re-render problem (theme + auth + cart in one context, or frequent
updates churning all consumers). Have me, solo: profile, fix by splitting/memoizing/separating
state-and-dispatch, prove the cascade is gone. Consider whether some of this even belongs in context.
Don't tell me the fix. Exam me on context's re-render semantics. Verdict: can I diagnose and fix
context cascades (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problem: one context whose `value={{ user, theme, cart, setCart, ... }}` is a **new object every render**, so all consumers re-render every render; and all concerns bundled, so a `theme`-only consumer re-renders when `cart` changes. Profiler shows the whole consumer set lighting up on any update.
- The fixes: **memoize the value** (`useMemo`); **split the context by concern** (separate Theme/Auth/Cart contexts) so each consumer subscribes only to its slice; **separate state from dispatch** so dispatch-only components don't re-render on state changes.
- Re-measure: each consumer re-renders only for relevant changes.

The principle: **context delivers an update to every consumer whenever the provider's `value` changes by reference — so a fresh value object or one over-broad context re-renders the entire consumer set, and the fixes are memoizing the value and splitting by concern.** Context is a transport, not an optimized store.

</details>

---

## Module 6.7 — List Virtualization: Rendering 100k Rows

**Difficulty:** ●●●●● · **Format:** Build challenge · **Stack:** React + Tailwind + React DevTools

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **React Performance, v2** (Steve Kinney) → the **virtualization / windowing** section — rendering only visible rows to keep huge lists fast.

**Primary sources:**

- **TanStack Virtual — Introduction** — https://tanstack.com/virtual/latest/docs/introduction — *Focus on: windowing, `estimateSize`, `overscan`.*
- **web.dev — Virtualize large lists** — https://web.dev/articles/virtualize-long-lists-react-window — *Focus on: the windowing math and constant DOM node count.*

**Search prompts:**

- `"react list virtualization windowing tanstack virtual"`
- `"render only visible rows constant dom nodes"`
- Ask an AI: *"Explain list virtualization (windowing): given scroll position, row height, and container height, which row indices are visible? How does rendering only the visible window (plus overscan) while faking full scroll height keep a 100k-row list at a constant ~30 DOM nodes? What makes variable-height rows harder, and what does `overscan` do?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.7, Exercise 1 — list virtualization, build
challenge. Completed: Tracks 0–5, Modules 6.1–6.6. Make it hard.

Give me a React + Tailwind app rendering a 50k–100k row list ALL AT ONCE (every row a real DOM node +
component), so initial render and scroll are catastrophic. Profile the disaster first (thousands of
nodes, huge commit, janky scroll). Then have me implement virtualization — FIRST from scratch (compute
visible indices from scrollTop/rowHeight/containerHeight, render only the window + overscan, fake
total height with a spacer), THEN optionally with TanStack Virtual. No solution code for the
from-scratch math.

Review me like a senior. Before coding, make me compute by hand: at 40px rows × 100k rows, total
height? Given scrollTop 8000 and a 600px container, which indices render? If my DOM node count grows
while scrolling, ask what should stay constant. Re-measure: constant nodes, smooth scroll, tiny
commit. Exam me on the windowing math and why it works.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.7, Exercise 2 (transfer) — virtualization,
harder variant. Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT large-list scenario with a twist: VARIABLE row heights (or a grid), plus it must
combine with another concern from this track (memoized rows, stable keys). Have me, solo: virtualize
correctly (handling variable heights/measurement), keep DOM nodes constant, keep scroll smooth, prove
it with the Profiler + a trace. Near-zero hints on the math. Exam me on the windowing math, overscan,
and the variable-height challenge. Verdict: can I virtualize a real large list (proceed), or redo
Exercise 1?
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- The disaster: rendering all 100k rows = 100k+ DOM nodes/components — enormous initial render/commit, huge memory, janky scroll.
- From-scratch virtualization: an outer scroll container of fixed height; an inner spacer sized `totalRows * rowHeight` (correct scrollbar); compute `startIndex = floor(scrollTop / rowHeight)`, `endIndex = ceil((scrollTop + containerHeight) / rowHeight)`, render only `[start - overscan, end + overscan]`, positioned at `index * rowHeight`. DOM node count stays ~constant regardless of total rows.
- `overscan`: a few extra rows above/below so fast scroll doesn't flash blank.
- Variable heights (Exercise 2): can't derive index from a constant rowHeight — needs measured/estimated heights + a cumulative-offset structure (what TanStack Virtual manages). The genuinely hard case.
- Re-measure: constant nodes, tiny commit, smooth scroll at 100k rows.

The principle: **virtualization renders only the visible window (plus overscan) while faking full scroll height, so a 100k-row list costs the same as a 30-row one — the windowing math is the core, variable-height rows the hard extension.** The canonical "render at scale" technique and a frequent senior interview topic.

</details>

---

## Module 6.8 — Code Splitting & Lazy Loading Components

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + Vite + bundle visualizer

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **React Performance, v2** (Steve Kinney) → the **code-splitting / `React.lazy` / Suspense** section — splitting by route/component to shrink the initial bundle.
- **Web Performance with Webpack** (Sean Larkin) — optional deeper dive on splitting and bundle analysis if you do that course.

**Primary sources:**

- **React docs — `lazy` and Suspense** — https://react.dev/reference/react/lazy — *Focus on: lazy-loading a component and its chunk.*
- **web.dev — Code splitting with dynamic `import()`** — https://web.dev/articles/reduce-javascript-payloads-with-code-splitting — *Focus on: route/feature split points.*
- **vite-bundle-visualizer / rollup-plugin-visualizer** — https://github.com/btd/rollup-plugin-visualizer — *Focus on: reading the treemap to find what to split.*

**Search prompts:**

- `"react lazy suspense route code splitting"`
- `"bundle analyzer treemap reduce initial bundle"`
- Ask an AI: *"Explain code splitting in React: how `React.lazy` + `Suspense` and dynamic `import()` create separate chunks loaded on demand, splitting by route or heavy feature so the initial bundle stays small. How do I read a bundle treemap to find the biggest offenders, and what's the cost of OVER-splitting into many tiny chunks?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.8, Exercise 1 — code splitting. Completed:
Tracks 0–5, Modules 6.1–6.7.

Build me a React + Vite multi-route app with a bloated initial bundle: a heavy library (charts/date/
markdown/editor) imported eagerly at the top but used on one rare route; all routes in the main
chunk; a big component loaded upfront but shown only behind a tab/modal. Include a bundle visualizer.
Do NOT tell me what's heavy.

Have me build, open the treemap, measure the initial chunk. Find the offenders, then split: React.lazy
+ Suspense for routes, dynamic import() for the heavy library at its point of use, defer the
behind-a-tab component — re-measuring each time. Then have me OVER-split and observe the downside.
Exam me on: what moved to which chunk, the load-time win, and the over-splitting tradeoff.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.8, Exercise 2 (transfer) — splitting a new
app. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT app with a different bundle-bloat profile. Have me, solo: read the treemap,
choose split points (routes? heavy features? vendor?), implement with lazy/Suspense/dynamic import,
handle loading states, prove the initial-bundle reduction + faster start. Don't tell me what to split.
Exam me on choosing boundaries and avoiding over-splitting/waterfalls. Verdict: can I shrink an
initial bundle via splitting (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The bloat: a heavy library imported eagerly but used on one rare route; no route-level splitting; a big component loaded upfront but shown only behind a tab/modal. The treemap shows the initial chunk dominated by deferrable code.
- The fixes: **`React.lazy(() => import('./Route'))` + `<Suspense>`** per route (loaded on navigation); **dynamic `import()`** for the heavy library at its point of use; lazy-load the behind-a-tab component when the tab opens. Re-measure: smaller initial chunk + lazy chunks; faster start.
- Over-splitting: many tiny chunks → many requests and possible waterfalls. Split at **meaningful boundaries** (routes, heavy features), not everything. Each lazy boundary needs a Suspense fallback.

The principle: **code splitting keeps the initial bundle small by deferring non-critical code into on-demand chunks, and the treemap shows exactly what to defer — the goal being the smallest initial payload that renders the first view, without fragmenting into a waterfall.** One of the biggest start-up levers for a real app.

</details>

---

## Module 6.9 — Hydration: The Cost of Making Server HTML Interactive

**Difficulty:** ●●●●○ · **Format:** Investigation + broken project · **Stack:** Next.js (App Router) + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **React Performance, v2** (Steve Kinney) → the **server-side rendering & hydration** section — what hydration costs and why too much client JS makes a server-rendered page feel slow to become interactive.

**Primary sources:**

- **web.dev — Rendering on the web (hydration)** — https://web.dev/articles/rendering-on-the-web — *Focus on: SSR + hydration, and the gap where HTML is visible but not interactive.*
- **Next.js — Server and Client Components** — https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns — *Focus on: `'use client'` boundaries and shipping less client JS.*
- **React docs — `hydrateRoot`** — https://react.dev/reference/react-dom/client/hydrateRoot — *Focus on: attaching interactivity to server HTML.*

**Search prompts:**

- `"react hydration cost ssr interactive delay"`
- `"next.js use client boundary reduce client javascript"`
- Ask an AI: *"Explain hydration: a server-rendered page shows HTML fast, but React must download the JS and 'hydrate' (attach listeners, rebuild state) before it's interactive — so during that gap the page looks ready but ignores clicks (bad INP/TBT). Why does shipping LESS client JS (more server components, fewer `'use client'` boundaries) reduce hydration cost?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.9, Exercise 1 — hydration cost. Completed:
Tracks 0–5, Modules 6.1–6.8.

Scaffold me a Next.js (App Router) app that server-renders fast but is slow to become INTERACTIVE
because too much is client: a `'use client'` near the top pulling a large subtree (and heavy deps) to
the client to hydrate, so there's a long "visible but dead" gap and poor INP/TBT. Do NOT name the
cause.

Have me measure: the gap between content appearing and interactivity (trace, TBT, try clicking during
hydration), and the client JS size. Find the over-broad client boundary, push `'use client'` DOWN to
the small interactive leaves, keep the rest as server components, re-measure. Exam me on: what
hydration does, why it blocks interactivity, and why less client JS = cheaper hydration.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.9, Exercise 2 (transfer) — hydration, new
app. Apply the Exercise 2 rules: solo first; verdict.

Scaffold me a DIFFERENT Next.js app with a different over-hydration pattern (a mostly-static page
shipped as one big client island, or heavy client-only libs that could be server-side). Have me,
solo: measure time-to-interactive and client JS, minimize the client boundary, move work to the
server, prove hydration cost dropped. Don't tell me the fix. Exam me on the server/client boundary's
perf implications. Verdict: can I diagnose and reduce hydration cost (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problem: a `'use client'` high in the tree makes a large subtree (and deps) ship to the client and **hydrate**. HTML appears fast (SSR) but isn't interactive until React downloads that JS and hydrates — attaching listeners and rebuilding state. During the gap the page looks ready but ignores input (bad INP/TBT). Bigger client JS → longer gap.
- The fix: push **`'use client'` down to the smallest interactive leaves**, keeping the rest as **server components** (HTML, zero JS for themselves). Less client JS → less to download/parse/hydrate → faster interactivity. Re-measure: smaller client bundle, shorter dead gap, better TBT.
- Mental model: server components are free of client JS cost; every `'use client'` boundary is a hydration bill — keep interactive islands small, static majority on the server.

The principle: **hydration is the cost of making server-rendered HTML interactive — downloading and running client JS to attach behavior — so a page can look ready while ignoring input, and the fix is shipping less client JS by keeping `'use client'` boundaries small.** Essential for modern Next.js performance.

</details>

---

## Module 6.10 — Streaming SSR & Suspense for Performance

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** Next.js (App Router) + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **React Performance, v2** (Steve Kinney) → the **Suspense / streaming** material — sending the fast shell first and streaming slower parts in.

**Primary sources:**

- **Next.js — Loading UI and Streaming** — https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming — *Focus on: `loading.js`, Suspense boundaries, streaming fast parts before slow.*
- **React docs — `<Suspense>`** — https://react.dev/reference/react/Suspense — *Focus on: showing a fallback for a slow part without blocking the rest.*

**Search prompts:**

- `"next.js streaming suspense loading.js fast shell"`
- `"streaming ssr ttfb stream slow components"`
- Ask an AI: *"Explain streaming SSR with Suspense in Next.js: instead of waiting for ALL data before sending any HTML, stream the fast shell immediately and stream slow components in as their data resolves (fallback meanwhile). How does wrapping a slow data-dependent component in `<Suspense>` improve TTFB/FCP and perceived speed?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.10, Exercise 1 — streaming SSR & Suspense.
Completed: Tracks 0–5, Modules 6.1–6.9.

Scaffold me a Next.js (App Router) page where ONE slow data dependency blocks the WHOLE page — the
user stares at nothing until the slowest query resolves. Do NOT name the cause.

Have me measure (TTFB/FCP — the whole page waits). Wrap the slow component in `<Suspense>` with a
fallback (or use loading.js) so the fast shell streams immediately and the slow part streams in when
ready, re-measure the improved TTFB/FCP and perceived load. Exam me on: how streaming works, why one
slow query shouldn't block the page, and what Suspense boundaries control.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.10, Exercise 2 (transfer) — streaming, new
page. Apply the Exercise 2 rules: solo first; verdict.

Scaffold me a DIFFERENT page with multiple data sources of varying speed. Have me, solo: place
Suspense boundaries so fast content appears immediately and each slow section streams in independently
with its own fallback (not one big spinner), prove the TTFB/FCP/perceived improvement. The trap:
boundaries too coarse = one slow part still blocks a lot. Don't tell me where. Exam me on boundary
placement. Verdict: can I use streaming/Suspense to improve real load (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problem: a page where the server waits for **everything** (including one slow query) before sending any HTML — TTFB/FCP gated by the slowest data source; the user sees nothing until then.
- The fix: wrap the slow component in **`<Suspense fallback={...}>`** (or route-level `loading.js`). The server **streams the fast shell immediately** (instant FCP), shows the fallback for the slow region, and **streams that region in** when its data resolves. Re-measure: TTFB/FCP improve, perceived load much better.
- Boundary placement (Exercise 2): granular boundaries let each slow section stream independently; one coarse boundary lets a slow part block a large region. Isolate each independently-slow piece.

The principle: **streaming SSR with Suspense sends the fast shell immediately and streams slow data-dependent parts in as they resolve — so one slow query no longer blocks the whole page, improving TTFB, FCP, and perceived speed.** Correct boundary placement is the skill.

</details>

---

## Module 6.11 — Server Components & Reducing Client JS

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** Next.js (App Router) + bundle visualizer

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **React Performance, v2** (Steve Kinney) → the **React Server Components** material — moving work and code to the server to ship less client JS (pairs with hydration).

**Primary sources:**

- **Next.js — Server Components** — https://nextjs.org/docs/app/building-your-application/rendering/server-components — *Focus on: zero client JS for server components, async data fetching on the server.*
- **React docs — Server Components** — https://react.dev/reference/rsc/server-components — *Focus on: what stays off the client.*

**Search prompts:**

- `"react server components reduce client bundle javascript"`
- `"move heavy library to server component next.js"`
- Ask an AI: *"Explain how React Server Components reduce client JavaScript: a server component runs only on the server and ships ZERO JS to the client (just rendered output), so moving data fetching, heavy formatting/parsing libraries, and non-interactive UI to server components shrinks the client bundle and hydration cost. What CAN'T be a server component (anything interactive/stateful/browser-API)?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.11, Exercise 1 — server components & client
JS. Completed: Tracks 0–5, Modules 6.1–6.10.

Scaffold me a Next.js (App Router) app shipping unnecessary client JS: components marked `'use client'`
that don't need to be (no interactivity), and heavy libraries (markdown parser, date/format lib)
bundled to the client when they could run on the server. Include a bundle visualizer. Do NOT name the
issues.

Have me measure the client bundle, find the needless client components + client-bundled libs, convert
the non-interactive ones to server components and move the heavy libs server-side, re-measure the
client JS drop. Exam me on: what server components ship (nothing), what must stay client, and why this
cuts both bundle size and hydration cost.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.11, Exercise 2 (transfer) — server/client
split, new app. Apply the Exercise 2 rules: solo first; verdict.

Scaffold me a DIFFERENT app with a suboptimal server/client split. Have me, solo: audit which
components truly need to be client (interactive/stateful/browser API) vs which can be server, move the
boundary correctly, keep heavy non-interactive work on the server, prove the client-JS reduction. The
trap: pushing the boundary wrong breaks interactivity or leaves heavy libs on the client. Don't tell
me the split. Exam me on the server/client decision and its perf impact. Verdict: can I optimize the
boundary for minimal client JS (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problem: components marked `'use client'` with **no interactivity** (no state/effects/handlers/browser APIs) shipping needless JS; and **heavy libraries** (markdown parser, date/format, syntax highlighter) bundled to the client when they could run once on the server. The visualizer shows the bloat.
- The fix: convert non-interactive components to **server components** (run on the server, ship **zero JS**, only rendered HTML), and move heavy non-interactive library work server-side so those libs never reach the client bundle. Keep `'use client'` only for genuinely interactive leaves. Re-measure: client JS drops, which also reduces hydration cost (6.9).
- What must stay client: anything using state/effects/handlers/browser APIs.

The principle: **server components ship zero client JavaScript — only rendered output — so moving data fetching, heavy non-interactive libraries, and static UI to the server shrinks both the client bundle and hydration cost, while only genuinely interactive pieces stay client.** One of the most impactful modern React performance skills.

</details>

---

## Module 6.12 — Concurrent React: useTransition & useDeferredValue

**Difficulty:** ●●●●● · **Format:** Build challenge · **Stack:** React + Tailwind + React DevTools

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **React Performance, v2** (Steve Kinney) → the **concurrent features / transitions** section — `useTransition` and `useDeferredValue` for keeping the UI responsive during expensive updates.

**Primary sources:**

- **React docs — `useTransition`** — https://react.dev/reference/react/useTransition — *Focus on: marking a non-urgent update so urgent input stays responsive; `isPending`.*
- **React docs — `useDeferredValue`** — https://react.dev/reference/react/useDeferredValue — *Focus on: deferring a value so expensive renders driven by it don't block urgent updates.*
- **React docs — startTransition** — https://react.dev/reference/react/startTransition — *Focus on: how concurrent React interrupts a non-urgent render.*

**Search prompts:**

- `"react usetransition ispending non-urgent update responsive input"`
- `"react usedeferredvalue expensive list filter"`
- Ask an AI: *"Explain `useTransition` and `useDeferredValue`. What's an urgent vs non-urgent update, and how does marking an expensive update as a transition keep the input responsive (concurrent React interrupts the non-urgent render for urgent input)? Difference: transition wraps the update you control, deferredValue wraps a value you receive. Crucially: do they make the work faster, or just keep the UI responsive?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.12, Exercise 1 — concurrent React, build
challenge. Completed: Tracks 0–5, Modules 6.1–6.11. Make it hard.

Give me a spec (no solution code) for a feature where urgent input (typing in a search/filter) fights
an expensive update (filtering + rendering a very large list, or a heavy viz): naively, every
keystroke triggers the expensive render synchronously, so typing lags. My job: keep the input
perfectly responsive using `useTransition` and/or `useDeferredValue`, with a pending indicator.

Review me like a senior. Measure the lag first. If typing still lags, ask which update is urgent vs
non-urgent and whether I marked the right one. If I use both hooks redundantly, ask what each does
differently. CRUCIAL: make me confirm the expensive work isn't FASTER — the UI is just responsive
because the urgent update interrupts the non-urgent one. Exam me on the mechanism and the
transition-vs-deferredValue distinction.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.12, Exercise 2 (transfer) — concurrent, new
case. Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT urgent-vs-expensive scenario (a tab switch rendering a heavy view, or a slider
driving an expensive computation). Have me, solo: keep the urgent interaction responsive with the
RIGHT concurrent tool, add a proper pending state, prove (Profiler/trace) the input stays responsive
while the expensive view catches up — without claiming the work got faster. Don't tell me which hook.
Exam me on urgent/non-urgent, transition vs deferredValue, and "responsive not faster." Verdict: do I
understand concurrent React for responsiveness (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- The input's value update stays **urgent** (responsive); the **expensive** derived render is made **non-urgent** — either by wrapping the driving state update in `startTransition` (`useTransition`, with `isPending` for an indicator), or by feeding the expensive component a `useDeferredValue` of the input.
- The distinction: **`useTransition`** wraps the **state update you control** (mark the transition at the source); **`useDeferredValue`** wraps a **value you receive** (defer downstream, when you don't control the update). Not both on the same path.
- **Crucial:** concurrent React does NOT make the work faster. It keeps the UI **responsive** by letting urgent input interrupt the non-urgent render, so the input never feels stuck while the list catches up. "Faster" is the misunderstanding to catch.
- A pending UI (dim the stale list, a spinner) via `isPending` or comparing deferred vs current value.
- Common mistakes: marking the input value itself as a transition (laggy typing); using both hooks redundantly; expecting a speedup; no pending feedback.

The principle: **concurrent React can interrupt a low-priority render to handle urgent input, so marking the expensive update as a transition — or deferring the value that drives it — keeps typing responsive while the heavy view catches up; it improves responsiveness, not raw speed.** Transitions act at the source, deferred values at consumption.

</details>

---

## Module 6.13 — The React Compiler & The Future of Memoization

**Difficulty:** ●●●○○ · **Format:** Investigation · **Stack:** React 19 + React DevTools

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **React Performance, v2** (Steve Kinney) → the **React Compiler** section (React 19) — automatic memoization and what it changes about your `memo`/`useMemo`/`useCallback` discipline.

**Primary sources:**

- **React docs — React Compiler** — https://react.dev/learn/react-compiler — *Focus on: what the compiler auto-memoizes, its rules-of-React requirements, and what it does/doesn't replace.*
- **React blog — React Compiler announcement** — https://react.dev/blog — *Focus on: the motivation (eliminate manual memoization) and current status.*

**Search prompts:**

- `"react compiler automatic memoization react 19"`
- `"react compiler replace usememo usecallback"`
- Ask an AI: *"Explain the React Compiler: how it automatically memoizes components and values at build time so you (mostly) don't need manual `memo`/`useMemo`/`useCallback`. What 'rules of React' must your code follow for it to work, what does it NOT solve (re-render scope, algorithmic cost, bundle size), and should I still understand manual memoization?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.13, Exercise 1 — the React Compiler.
Completed: Tracks 0–5, Modules 6.1–6.12. Investigation.

Give me a small React 19 app with wasted renders (like Module 6.5) but WITHOUT manual memoization.
Have me: profile the wasted renders, then enable the React Compiler and re-profile to SEE the
auto-memoization eliminate them. Do NOT explain the result for me. Then have me find a case the
compiler does NOT fix (e.g. a re-render-scope problem better solved by colocation, or an algorithmic
hot path, or bundle size) and recognize the compiler's limits. Exam me on: what the compiler
automates, the rules-of-React it relies on, and what it does NOT solve.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.13, Exercise 2 (transfer) — compiler limits.
Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT app and have me, solo: determine which of its performance problems the React
Compiler handles automatically vs which still need ME (re-render scope/colocation, virtualization,
code-splitting, algorithmic cost, hydration/client-JS). Apply the compiler where it helps, fix the
rest manually, and prove the result. Don't tell me which is which. Exam me on the compiler's scope and
its limits. Verdict: do I understand where automatic memoization ends and engineering judgment begins
(proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

- The React Compiler (stable in the React 19 era) **automatically memoizes** components and values at build time, so most manual `memo`/`useMemo`/`useCallback` becomes unnecessary — it analyzes your code and inserts the equivalent caching for you. Enabling it on the wasted-render app eliminates those wasted renders without you writing memo.
- Its requirement: your code must follow the **rules of React** (pure render, no mutation during render, proper hooks usage) — the compiler relies on that to memoize safely. Code that breaks the rules opts out.
- **What it does NOT solve** (the key lesson): it doesn't fix **re-render scope** (state still belongs colocated — Module 6.4), **algorithmic cost** in your render/handlers, **virtualization** for huge lists, **code-splitting/bundle size**, or **hydration/client-JS** decisions. It removes the manual-memoization chore; it doesn't remove the need to understand the render model and architecture.
- Why you still learned manual memoization (Module 6.5): you need to understand *what* is being automated to reason about performance, debug when the compiler opts out, and work in codebases not yet using it.

The principle: **the React Compiler automates memoization so you rarely hand-write `memo`/`useMemo`/`useCallback`, but it doesn't replace performance engineering — re-render scope, algorithms, virtualization, bundle size, and hydration are still yours.** Understanding the render model matters more than ever, because the compiler is a tool you direct, not a substitute for judgment.

</details>

---

## Module 6.14 — The React Performance Diagnosis (React Capstone)

**Difficulty:** ●●●●● · **Format:** Investigation + broken project · **Stack:** React/Next.js + React DevTools

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- Revisit whichever **React Performance, v2** sections covered the issues you found hardest. This capstone integrates the whole track.

**Primary sources:**

- **React docs — Profiler (revisit)** — https://react.dev/reference/react/Profiler — *Focus on: attributing render cost.*
- **web.dev — Optimize INP (React angle)** — https://web.dev/articles/optimize-inp — *Focus on: interaction responsiveness in a React app.*

**Search prompts:**

- `"diagnose react app performance methodology profiler"`
- Ask an AI: *"Give me a systematic method to diagnose a slow React app: profile to find wasted/expensive renders, classify each (re-render scope? unstable props defeating memo? context cascade? huge list? big bundle? hydration? algorithmic cost in render?), and apply the matching fix (colocation, memo, split context, virtualize, code-split, server components, better algorithm) in order of leverage."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.14, Exercise 1 — React performance diagnosis,
capstone. Completed: Tracks 0–5, Modules 6.1–6.13. Make it genuinely hard.

Build me a React/Next.js app that's slow for SEVERAL reasons at once: over-lifted state causing a
re-render cascade, a context cascade, a memo defeated by unstable props, a large un-virtualized list,
a bloated initial bundle, and maybe over-hydration. Do NOT tell me how many problems or what they are.

Full React diagnosis. Have me: profile (React Profiler + a Performance trace), for each problem
classify the cause (scope? props? context? list size? bundle? hydration? algorithm?), and apply the
matching fix in order of leverage (colocation first, then context/memo, virtualize, code-split, server
components) — one at a time, re-measuring. Hold the loop. Long exam tying each fix to its mechanism.
Rehearses diagnosing a real slow React app cold.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 6, Module 6.14, Exercise 2 (transfer) — diagnose a
DIFFERENT slow React app solo. Apply the Exercise 2 rules: minimal hints; full verdict.

Build me a DIFFERENT slow React/Next.js app with a different mix of problems. Have me diagnose and fix
the WHOLE thing solo — profile, classify each bottleneck, fix in leverage order, re-measure, produce a
before/after report. Near-zero hints. Hard exam. Then the verdict I need: can I open an unfamiliar slow
React app and diagnose its performance cold — naming why it's slow and the fix for each (proceed to
Track 7), or do I need to redo parts of Track 6?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

The app combines the track:

- **Over-lifted state** → colocation/composition (6.4).
- **Context cascade** → split/memoize context (6.6).
- **Memo defeated by unstable props** → stabilize or restructure (6.5).
- **Large un-virtualized list** → virtualize (6.7).
- **Bloated initial bundle** → code-split (6.8).
- Possibly **over-hydration / too much client JS** → server components, smaller `'use client'` (6.9/6.11).

The methodology you should now run unprompted:

1. **Profile** (React Profiler for renders + a Performance trace for main-thread/commit cost); baseline.
2. **For each problem, classify the cause:** re-render scope, unstable props, context, list size, bundle size, hydration, or algorithmic cost in render.
3. **Apply the matching fix in leverage order** — colocation/composition first (free, structural), then context/memo, then virtualize/code-split/server-components — one at a time, re-measuring.
4. **Report** the cumulative improvement.

The principle: **diagnosing a slow React app is systematic attribution — profile, classify each bottleneck by cause, and apply the matching fix in order of leverage (structural before memoization before architectural).** This capstone is your React end goal: open a slow React app and name why it's slow and how to fix each part. If you can do Exercise 2 cold, you own your own stack's performance — the single most marketable skill for your career.

</details>

---

*End of Track 6 — React & Next.js Performance at Scale. 14 modules. You can now diagnose any React app's performance from the render model up — re-renders, reconciliation, memoization, context, virtualization, code-splitting, hydration, server components, concurrent features, and the compiler.*

*Next: Track 7 — Perceived Performance & The UX of Speed, where you make slow things FEEL fast.*
