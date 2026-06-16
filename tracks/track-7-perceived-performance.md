# TRACK 7 — PERCEIVED PERFORMANCE & THE UX OF SPEED

*The track that separates "technically fast" from "feels fast." Two pages with identical load times can feel completely different depending on what the user sees while waiting and how quickly the app responds to their actions. This is where performance meets UX — skeleton screens, optimistic UI, the human-perception thresholds, and the art of making an unavoidable wait feel shorter. It's also where you learn that the most important number is sometimes the one the user feels, not the one the profiler reads.*

*Prerequisites: Tracks 0–1 (measure honestly, know the metrics) and ideally Track 6 (you'll apply these in React). Mostly React + Tailwind.*

*Spine course: **Web Performance Fundamentals, v2** (Todd Gardner) — its perceived-performance and "How Fast Should Your Site Be?" material grounds this track in the human thresholds. The user-expectation lessons are the conceptual backbone.*

---

## Module 7.1 — Human Perception Thresholds: 100ms, 1s, 10s

**Difficulty:** ●●○○○ · **Format:** Investigation · **Stack:** React + Tailwind + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"User Expectations of Performance"** material and the **"Setting Performance Goals"** section ("How Fast Should Your Site Be?"). Todd frames the human thresholds that everything in this track serves.

**Primary sources:**
- **Nielsen Norman Group — Response Times: The 3 Important Limits** — https://www.nngroup.com/articles/response-times-3-important-limits/ — *Read fully. The canonical 0.1s / 1s / 10s thresholds and what each means for UX.*
- **web.dev — Measure performance with the RAIL model** — https://web.dev/articles/rail — *Focus on: the RAIL targets (Response <100ms, Animation 60fps, Idle, Load) tied to perception.*

**Search prompts:**
- `"nielsen response time limits 100ms 1 second 10 seconds"`
- `"rail model response animation idle load performance"`
- Ask an AI: *"Explain the three human-perception response-time thresholds (≈100ms feels instant, ≈1s keeps flow of thought unbroken, ≈10s is the limit of attention) and the RAIL model's targets. For each threshold, what UX response is appropriate — instant feedback, a simple loader, or a progress indicator with the ability to do something else?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 7, Module 7.1, Exercise 1 — perception thresholds.
Completed: Tracks 0–6. Investigation + small build.

Build me a React + Tailwind demo with several interactions of DIFFERENT latencies: one ~instant
(<100ms), one ~1s, one ~3–5s, one ~10s+ — each WITHOUT appropriate feedback (no loaders, no progress).
Let me feel how bad each unfeedback'd wait is. Do NOT tell me which threshold each crosses.

Have me categorize each interaction by the perception threshold it falls into, then prescribe the
RIGHT feedback for each (instant: nothing/inline; ~1s: a subtle loader; longer: a progress indicator
and/or let the user keep working). Implement the feedback and feel the difference — with NO change to
the actual latency. Exam me on: the three thresholds, what feedback each demands, and why feedback
changes the FELT experience without changing the real time.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 7, Module 7.1, Exercise 2 (transfer) — thresholds applied
to a real flow. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT multi-step flow (e.g. a checkout or a search-then-load) with mixed latencies and
no feedback. Have me, solo: classify each step by threshold, design and implement the appropriate
feedback for each, and articulate why the flow now FEELS faster despite identical real timings. Don't
tell me the thresholds. Exam me on the threshold→feedback mapping and the perceived-vs-actual point.
Verdict: do I understand the human thresholds and their UX implications (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

No bug — this is the conceptual foundation of the track. What you should now hold:

- **~100ms — feels instant.** Below this, the user perceives the system as reacting immediately. Direct manipulation (button presses, hovers, typing) must respond within this window or it feels laggy. Appropriate feedback: the result itself, or instant visual acknowledgment.
- **~1s — keeps the flow of thought unbroken.** Up to about a second, the user stays in flow though they notice the delay. Appropriate feedback: usually a simple loader is enough; no elaborate progress needed.
- **~10s — the limit of attention.** Beyond a second and toward ten, you risk losing them; past ten, they'll mentally switch away. Appropriate feedback: a **progress indicator** (ideally showing actual progress), and ideally let them do something else meanwhile.
- The central insight: **appropriate feedback changes the felt experience without changing the real latency.** An unfeedback'd 3-second wait feels broken; a 3-second wait with a good progress indicator feels reasonable. Same milliseconds, very different UX.

The principle: **human perception has distinct thresholds (~100ms instant, ~1s flow, ~10s attention limit), and matching the right feedback to each makes a wait feel acceptable — perceived performance is a real, designable property, separate from measured latency.** Everything else in this track is technique in service of these thresholds.
</details>

---

## Module 7.2 — Skeleton Screens & Progressive Rendering

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** React + Tailwind + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the perceived-performance material on showing structure early. (The streaming/Suspense work from Track 6 Module 6.10 is the technical sibling of this.)

**Primary sources:**
- **web.dev — Skeleton screens / perceived performance** — search "skeleton screen perceived performance web.dev" — *Focus on: why showing layout structure early beats a spinner.*
- **NN/g — Skeleton screens** — https://www.nngroup.com/articles/skeleton-screens/ — *Focus on: when skeletons help vs when they don't.*

**Search prompts:**
- `"skeleton screen vs spinner perceived performance"`
- `"progressive rendering show content as it loads"`
- Ask an AI: *"Explain skeleton screens and progressive rendering: why a skeleton (gray placeholders matching the final layout) often feels faster than a spinner, and why rendering content progressively (header first, then content, then secondary) feels faster than waiting for everything. When does a skeleton NOT help (very short waits, or when it causes layout shift)?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 7, Module 7.2, Exercise 1 — skeleton screens & progressive
rendering. Completed: Tracks 0–6, Module 7.1.

Build me a React + Tailwind page that, on load, shows a blank screen then a centered spinner for a
couple of seconds (throttled data), then pops in all content at once — a jarring, slow-feeling load.
Do NOT name the problem.

Have me replace the spinner with a SKELETON matching the final layout (no CLS when real content
arrives), and render content PROGRESSIVELY (structure/header first, then primary content, then
secondary) instead of all-at-once. Measure that actual load time is unchanged but the experience feels
faster, and check there's no layout shift. Exam me on: skeleton vs spinner, progressive vs all-at-once,
the CLS pitfall, and why this is perceived not actual improvement.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 7, Module 7.2, Exercise 2 (transfer) — skeletons on a new
layout. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT data-driven view (a feed, a dashboard, a profile page). Have me, solo: design a
skeleton that matches the real layout exactly (so the swap causes no shift), render progressively in a
sensible priority order, and justify why it feels faster. The trap: a skeleton that doesn't match the
final layout causes CLS and feels worse. Don't tell me the design. Exam me on skeleton fidelity and
progressive priority. Verdict: can I build perceived-performance loading states correctly (proceed),
or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problem: blank → spinner → everything-at-once. A centered spinner gives no sense of progress or structure, and the all-at-once pop-in feels abrupt and slow.
- The fixes: a **skeleton screen** — gray placeholders shaped like the final content — so the user immediately sees the page's structure forming (feels like progress); and **progressive rendering** — show the shell/header first, then primary content, then secondary — so something useful appears as soon as possible rather than waiting for the slowest piece. Crucially, the skeleton must **match the final layout** so that when real content replaces it, there's **no layout shift** (a mismatched skeleton causes CLS and feels worse).
- Measure: actual load time is unchanged; the experience feels faster because the user perceives continuous progress and early structure.
- When skeletons don't help: very short waits (the skeleton flashes annoyingly) or when they don't match the layout.

The principle: **showing structure early (skeleton) and content progressively makes a load feel faster than a spinner-then-pop-in, because the user perceives continuous progress — provided the skeleton matches the final layout so there's no shift.** This is the streaming idea (Track 6.10) expressed as a deliberate UX pattern.
</details>

---

## Module 7.3 — Optimistic UI: Responding Before the Server Does

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** React + Tailwind + small Node server

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- No single dedicated FM lesson; the React data/mutation knowledge plus the perception thresholds (7.1) are the prerequisites. Lean on the primary sources.

**Primary sources:**
- **React docs — `useOptimistic`** — https://react.dev/reference/react/useOptimistic — *Focus on: showing the expected result immediately, reconciling when the server responds.*
- **TanStack Query — Optimistic Updates** — https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates — *Focus on: the update-rollback-on-error pattern.*
- **web.dev / UX articles — optimistic UI** — search — *Focus on: when optimism is safe vs risky.*

**Search prompts:**
- `"optimistic ui update useOptimistic rollback error"`
- `"react optimistic mutation instant feedback"`
- Ask an AI: *"Explain optimistic UI: updating the interface immediately as if the server succeeded (instead of waiting), then reconciling — keeping the change if it succeeds, rolling back + showing an error if it fails. How does React's `useOptimistic` help? When is optimism appropriate (likely-to-succeed, reversible actions like a like/toggle) vs risky (payments, irreversible actions)?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 7, Module 7.3, Exercise 1 — optimistic UI, build
challenge. Completed: Tracks 0–6, Modules 7.1–7.2. Small Node server for the latency.

Give me a spec (no solution code): an app with an action that hits a slow server (e.g. like/favorite,
add-to-list, send-message) where currently the UI waits for the response before updating — so it feels
laggy. My job: make it OPTIMISTIC — update immediately, reconcile on success, and ROLL BACK + show an
error on failure. The server should sometimes fail so I must handle rollback. Use `useOptimistic` or a
manual pattern.

Review me like a senior. If I update optimistically but don't handle the failure case, simulate a
server error and ask what the user now sees (a lie). If I make an IRREVERSIBLE/risky action optimistic,
ask whether optimism is appropriate there. Exam me on: the update-reconcile-rollback pattern, when
optimism is safe vs dangerous, and why it improves perceived (not actual) speed.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 7, Module 7.3, Exercise 2 (transfer) — optimistic UI, new
action + judgment. Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT set of actions, SOME suitable for optimism and some NOT (e.g. a reversible toggle
vs a payment vs a destructive delete). Have me, solo: implement optimism only where appropriate (with
correct rollback + error handling), and deliberately NOT for the risky ones — justifying each call.
Don't tell me which is which. Exam me on the safe-vs-risky judgment and the rollback mechanics.
Verdict: can I apply optimistic UI correctly and know when not to (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- The pattern: on the action, **immediately** update the UI to the expected result (so it feels instant — meeting the ~100ms threshold), fire the request, and on the response **reconcile**: keep the optimistic state if it succeeded, or **roll back** to the previous state and **show an error** if it failed. `useOptimistic` manages the temporary optimistic state that auto-reverts when the real state resolves.
- The critical failure case: optimism WITHOUT rollback shows the user a success that didn't happen — a lie. A rigorous reviewer forces a server error and checks you revert cleanly and inform the user.
- The judgment (Exercise 2): optimism suits **likely-to-succeed, reversible, low-stakes** actions (likes, toggles, reordering, adding a comment). It's **wrong** for **irreversible or high-stakes** actions (payments, destructive deletes, anything where a false success is harmful) — there you show real pending state and wait.
- It improves **perceived** speed (instant feedback) while the real request takes the same time.

The principle: **optimistic UI makes an action feel instant by updating immediately and reconciling on the server's response — with mandatory rollback + error handling on failure — and it's appropriate only for reversible, likely-to-succeed actions, never for irreversible or high-stakes ones.** Knowing when NOT to be optimistic is as important as the technique itself.
</details>

---

## Module 7.4 — Prioritizing Above-the-Fold & Critical Content

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** React/HTML + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the FCP/LCP and critical-content material (ties Track 2's critical path to the *perception* of a fast first impression).

**Primary sources:**
- **web.dev — Optimize LCP / prioritize above-the-fold** — https://web.dev/articles/optimize-lcp — *Focus on: getting the visible, important content rendered first.*
- **web.dev — Inline critical CSS** — https://web.dev/articles/extract-critical-css — *Focus on: inlining the CSS for above-the-fold so it renders without a round-trip.*

**Search prompts:**
- `"prioritize above the fold content first paint"`
- `"critical css inline above the fold"`
- Ask an AI: *"Explain prioritizing above-the-fold content for perceived performance: rendering and styling the visible-on-load content first (inlining critical CSS, loading the hero/LCP first, deferring below-the-fold), so the user sees a complete-looking first screen quickly even if the rest is still loading. Why does a fast, complete first screen feel fast regardless of total load time?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 7, Module 7.4, Exercise 1 — prioritize above-the-fold.
Completed: Tracks 0–6, Modules 7.1–7.3.

Build me a page where the above-the-fold content is NOT prioritized: critical CSS arrives late (FOUC
or late styling of the first screen), the hero loads after below-fold junk, and the first screen
looks incomplete for a while. Do NOT name the problems.

Have me measure FCP/LCP and watch the filmstrip of the first screen forming. Make me prioritize the
above-the-fold: inline/critical CSS for the first screen, load the hero/LCP first, defer below-fold
work — so the first screen looks complete fast. Re-measure + re-watch the filmstrip. Exam me on: why a
complete-looking first screen drives perceived speed, what "critical content" means, and how inlining
critical CSS helps.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 7, Module 7.4, Exercise 2 (transfer) — critical content on
a new page. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT page (an article, a product page) where the first screen forms slowly/incompletely.
Have me, solo: identify the critical above-the-fold content, prioritize its styling and assets, defer
the rest, and prove via filmstrip that the first screen now completes fast. Don't tell me what's
critical. Exam me on identifying critical content and the prioritization techniques. Verdict: can I
make a fast-feeling first impression (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problems: critical CSS for the first screen arrives late (so the first screen is unstyled/incomplete for a while — FOUC or late layout), the hero/LCP loads after non-critical below-the-fold content, and the visible first screen looks broken while the user waits.
- The fixes: **inline the critical CSS** for above-the-fold content so the first screen styles without waiting for an external stylesheet round-trip; **load the hero/LCP image first** (eager, high priority — ties to Track 5); **defer below-the-fold** CSS/JS/images so they don't compete with the first screen. The filmstrip shows the first screen now completing quickly.
- The perception principle: users judge speed largely by **how fast the first screen looks complete** — a page whose visible area is done fast feels fast even if off-screen content is still loading. Conversely, a page that shows a broken/unstyled first screen feels slow regardless of total time.

The principle: **prioritizing above-the-fold content — inlining critical CSS, loading the hero first, deferring the rest — makes the first impression fast and complete, which drives perceived speed because users judge by the visible first screen, not total load.** This connects the technical critical path (Track 2) to the felt experience.
</details>

---

## Module 7.5 — Instant Navigation: Prefetching & Speculative Loading

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** Next.js / React Router + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- No dedicated FM lesson; builds on Track 2's resource hints (prefetch) and Next.js knowledge. Lean on primary sources.

**Primary sources:**
- **Next.js — Linking and Prefetching** — https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating — *Focus on: automatic prefetch of in-viewport links and instant client navigation.*
- **web.dev — Speculation Rules API / prerender** — https://developer.chrome.com/docs/web-platform/prerender-pages — *Focus on: prefetch/prerender the likely next page for near-instant navigation.*
- **web.dev — Prefetch on hover/intent** — search "prefetch on hover instant.page" — *Focus on: predicting the next navigation.*

**Search prompts:**
- `"next.js link prefetch instant navigation"`
- `"speculation rules api prerender next page"`
- Ask an AI: *"Explain making navigation feel instant: prefetching the data/code for likely next pages (e.g. Next.js prefetching in-viewport links, or prefetch-on-hover), and the Speculation Rules API to prerender a likely next page so the navigation is near-instant. What's the tradeoff — prefetching wastes bandwidth/work if the user never visits, so how do you predict intent?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 7, Module 7.5, Exercise 1 — instant navigation. Completed:
Tracks 0–6, Modules 7.1–7.4.

Build me a multi-page app (Next.js or React Router) where navigating between pages shows a visible
delay each time — nothing is prefetched, so each navigation fetches code+data from scratch. Do NOT
name the cause.

Have me measure the navigation delay, then make navigations feel instant: prefetch the likely next
page's code/data (in-viewport link prefetch and/or prefetch-on-hover/intent), and explore prerendering
a high-probability next page. Re-measure the perceived navigation speed. Then have me OVER-prefetch
everything and observe the wasted bandwidth/work — and reason about predicting intent. Exam me on: how
prefetch/prerender make navigation feel instant and the bandwidth tradeoff.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 7, Module 7.5, Exercise 2 (transfer) — speculative loading,
new app. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT app with a clear "likely next step" (e.g. a list→detail flow, or a wizard). Have
me, solo: implement intent-based prefetching (hover/viewport) and consider prerendering for the
highest-probability next page, balancing instant-feel against wasted work for users who don't proceed.
Don't tell me the strategy. Exam me on intent prediction and the prefetch tradeoff. Verdict: can I make
navigation feel instant without wasteful over-prefetching (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problem: every navigation fetches its code and data from scratch on click, so there's a visible delay each time — navigation feels sluggish even if each page is individually fine.
- The fixes: **prefetch** the likely next page's code/data *before* the user clicks — Next.js automatically prefetches in-viewport `<Link>`s; you can also prefetch **on hover/focus** (intent signals that the user is about to click). For the highest-probability next page, **prerender** it (Speculation Rules API) so the navigation is near-instant. Re-measure: navigations feel instant because the work was already done.
- The tradeoff: prefetching/prerendering **costs bandwidth and work that's wasted** if the user never visits that page — so you predict **intent** (viewport, hover, known flows) rather than prefetching everything. Over-prefetching on a slow connection or a data-capped device is harmful.
- This is perceived performance: the real work still happens, but it's moved *before* the click so the user never waits.

The principle: **navigation feels instant when the next page's code and data are already prefetched (on viewport/hover/intent) or prerendered — moving the work before the click — but prefetching wastes resources for users who don't proceed, so you predict intent rather than prefetch everything.** This is how modern apps achieve the "no loading between pages" feel.
</details>

---

## Module 7.6 — Scroll Restoration & Maintaining Context

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** React/Next.js + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- No dedicated FM lesson; this is a UX-of-speed detail. Lean on primary sources.

**Primary sources:**
- **MDN — `history.scrollRestoration`** — https://developer.mozilla.org/en-US/docs/Web/API/History/scrollRestoration — *Focus on: manual vs auto scroll restoration.*
- **web.dev / framework docs — scroll restoration in SPAs** — search "spa scroll restoration back button" — *Focus on: restoring scroll position on back-navigation.*

**Search prompts:**
- `"spa scroll restoration back button position"`
- `"history scrollRestoration manual react"`
- Ask an AI: *"Explain scroll restoration: why single-page apps often lose the user's scroll position on back-navigation (the browser's automatic restoration doesn't work when content loads async), making the app feel broken/slow. How do I restore scroll position correctly, and why does preserving context (scroll, state) make an app feel fast and trustworthy?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 7, Module 7.6, Exercise 1 — scroll restoration. Completed:
Tracks 0–6, Modules 7.1–7.5.

Build me an SPA (a long list → detail → back) where pressing Back dumps the user at the TOP of the
list instead of where they were — so they lose their place and the app feels broken/slow. The list
loads async, so the browser's auto-restoration fails. Do NOT name the cause.

Have me reproduce the lost-scroll, then fix it: restore the scroll position on back-navigation
(accounting for async content load), and preserve relevant state. Confirm the experience now feels
seamless. Exam me on: why SPAs lose scroll on back, the async-content timing problem, and why
preserving context makes an app feel fast.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 7, Module 7.6, Exercise 2 (transfer) — context
preservation, new flow. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT flow where navigation loses context (scroll, filter state, form input, or
expanded items) on back/forward. Have me, solo: preserve and restore the relevant context so the flow
feels seamless, handling the async timing. Don't tell me the fix. Exam me on scroll restoration and
context preservation. Verdict: can I make navigation preserve context so the app feels fast and
trustworthy (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problem: in an SPA, navigating list → detail → Back lands the user at the top of the list, not where they were. The browser's **automatic** scroll restoration relies on the content being present at restore time, but in an SPA the list re-renders/re-fetches **async**, so by the time the data loads the browser has already given up restoring — the user loses their place. This feels broken and slow (they have to scroll back down and find where they were).
- The fix: take **manual control** (`history.scrollRestoration = 'manual'`), **save the scroll position** (and relevant list state) when leaving, and **restore it after the async content has loaded** (not before — timing is the crux). Frameworks (Next.js) handle common cases, but custom async lists often need explicit handling. Confirm: Back now returns the user exactly where they were.
- The broader lesson: preserving **context** (scroll, filters, form input, expanded state) across navigation makes an app feel fast and trustworthy — losing it makes even a technically-fast app feel sluggish and frustrating.

The principle: **preserving and correctly restoring context — especially scroll position on back-navigation, timed to after async content loads — makes navigation feel seamless, while losing context makes a fast app feel broken.** Perceived performance includes continuity, not just speed.
</details>

---

## Module 7.7 — The Perceived-Performance Audit (UX-of-Speed Capstone)

**Difficulty:** ●●●●● · **Format:** Investigation + broken project · **Stack:** React/Next.js + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- Revisit **Web Performance Fundamentals, v2** → the user-expectations / "How Fast Should Your Site Be?" material. This capstone integrates the whole track.

**Primary sources:**
- **NN/g — Response time limits (revisit)** — https://www.nngroup.com/articles/response-times-3-important-limits/ — *Focus on: mapping every wait in a flow to a threshold and a feedback strategy.*
- **web.dev — RAIL (revisit)** — https://web.dev/articles/rail — *Focus on: response/animation/load targets.*

**Search prompts:**
- `"perceived performance audit feedback loading states flow"`
- Ask an AI: *"Give me a methodology to audit a user FLOW for perceived performance: identify every wait and interaction, classify each by the perception threshold, and prescribe the right technique (instant feedback, skeleton, optimistic UI, progressive rendering, prefetch, preserved context) — so the whole flow FEELS fast regardless of unavoidable real latencies."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 7, Module 7.7, Exercise 1 — perceived-performance audit,
capstone. Completed: Tracks 0–6, Modules 7.1–7.6. Make it realistic.

Build me a realistic multi-step app/flow (e.g. browse → search → detail → action → confirm) that is
technically not-too-slow but FEELS sluggish everywhere: no instant feedback on taps, spinners instead
of skeletons, blocking waits on actions, top-of-page on back, no prefetch. Do NOT enumerate the
problems.

Full perceived-performance audit. Have me: walk the entire flow, classify every wait/interaction by
threshold, and apply the right technique to each (instant feedback, skeleton, progressive render,
optimistic UI, prefetch, scroll restoration) — WITHOUT necessarily reducing real latency — then judge
whether the flow now feels fast. Hold me to "perceived vs actual." Long exam tying each technique to
its threshold and mechanism.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 7, Module 7.7, Exercise 2 (transfer) — audit a DIFFERENT
flow solo. Apply the Exercise 2 rules: minimal hints; full verdict.

Build me a DIFFERENT realistic flow that feels sluggish for different reasons. Have me audit and fix
the WHOLE thing solo — classify each wait, apply the right perceived-performance technique, justify
each, and produce a before/after of how the flow FEELS (and confirm actual latency where relevant).
Near-zero hints. Hard exam. Then the verdict: can I take any real flow and make it feel fast through
perceived-performance technique (proceed to Track 8), or do I need to redo parts of Track 7?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

The flow combines the track:
- **No instant feedback** on taps (≥100ms threshold violated) → instant visual acknowledgment (7.1).
- **Spinners instead of skeletons** → skeletons + progressive rendering (7.2).
- **Blocking waits on actions** → optimistic UI where appropriate (7.3).
- **Incomplete first screen** → prioritize above-the-fold (7.4).
- **Slow navigation** → prefetch/speculative load (7.5).
- **Lost scroll/context on back** → scroll restoration (7.6).

The methodology you should now run unprompted:
1. **Walk the entire flow** as a user, noting every wait and interaction.
2. **Classify each by perception threshold** (instant / flow / attention-limit) and note whether it has appropriate feedback.
3. **Apply the matching technique** to each — feedback, skeleton, optimism, progressive render, prefetch, context preservation.
4. **Judge the felt result**, holding the perceived-vs-actual distinction: you often improve the *experience* without changing the *latency*.

The principle: **auditing perceived performance is walking a flow, classifying every wait by its human threshold, and applying the technique that makes each feel fast — feedback, skeletons, optimism, prefetch, context preservation — so the whole flow feels fast regardless of unavoidable real latency.** This is the skill that makes a product feel premium, and it's a differentiator most engineers never develop. If you can do Exercise 2 cold, you can make anything feel fast.
</details>

---

*End of Track 7 — Perceived Performance & The UX of Speed. 7 modules. You can now make slow things feel fast — matching feedback to human thresholds, using skeletons, optimistic UI, progressive rendering, instant navigation, and preserved context.*

*Next: Track 8 — Third-Parties, Monitoring & Performance at Scale, where performance becomes an ongoing engineering discipline, not a one-time fix.*
