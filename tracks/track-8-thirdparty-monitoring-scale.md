# TRACK 8 — THIRD-PARTIES, MONITORING & PERFORMANCE AT SCALE

*The track that turns performance from a one-time cleanup into an ongoing engineering discipline — the part that actually keeps a real product fast as a team ships to it every day. You'll tame the third-party tax (the analytics, ads, chat widgets, and tags that quietly wreck real-world performance), stand up monitoring that catches regressions before users do, enforce budgets in CI, learn to test on genuinely weak hardware (including your parents' phones), and connect performance to the business numbers that get your work funded and your salary raised.*

*Prerequisites: Tracks 0–6 (you must be able to measure, diagnose, and fix across all three buckets). Several modules use a small Node server and your real low-end test devices.*

*Spine courses: **Web Performance Fundamentals, v2** (Todd Gardner) for the RUM/monitoring and third-party material, and the **Front-End System Design** material (FM) for the at-scale/budgets thinking. Primary sources carry the modern CI and device-testing pieces.*

---

## Module 8.1 — The Third-Party Tax: Tags, Analytics, Ads & Widgets

**Difficulty:** ●●●●○ · **Format:** Investigation + broken project · **Stack:** HTML/React + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the third-party / external-scripts material in the loading and main-thread sections. Todd shows how external scripts you don't control can dominate your performance.

**Primary sources:**
- **web.dev — Identify slow third-party JavaScript** — https://web.dev/articles/identify-slow-third-party-javascript — *Focus on: attributing main-thread time and bytes to third-party origins.*
- **web.dev — Loading third-party JavaScript** — https://web.dev/articles/efficiently-load-third-party-javascript — *Focus on: async/defer, lazy-loading, and not letting third parties block.*
- **Chrome DevTools — Third-party attribution** — search "devtools third party usage" — *Focus on: seeing per-origin cost.*

**Search prompts:**
- `"third party javascript performance cost analytics ads tags"`
- `"attribute main thread time to third party script"`
- Ask an AI: *"Explain the 'third-party tax': how analytics, ad tags, A/B testing scripts, chat widgets, and tag managers — code you don't control, loaded from other origins — add bytes, block the main thread, and degrade performance. How do I attribute cost to each third party in DevTools, and what's the danger of a third party that loads MORE scripts dynamically?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.1, Exercise 1 — the third-party tax.
Completed: Tracks 0–6. Investigation + build.

Build me a page that's reasonably fast on its OWN, then loaded down with simulated third parties: an
analytics snippet, a tag-manager-like loader that pulls in more scripts, a chat widget, and an
ad-like script — several synchronous or main-thread-heavy, from "other origins" (separate local
endpoints with delay). Do NOT tell me which third party costs what.

Have me measure the baseline (just first-party), then the loaded version (CWV, main-thread time,
bytes), and ATTRIBUTE the damage to each third party (DevTools, per-origin). Make me find the worst
offender and the one that loads more scripts. Exam me on: how third parties hurt (bytes + main thread +
blocking), how to attribute per-origin cost, and the special danger of script-injecting third parties.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.1, Exercise 2 (transfer) — third-party audit,
real-ish page. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT page with a different mix of third parties (including one that's heavy but
"business-critical" and one that's pure bloat). Have me, solo: attribute each one's cost, rank them by
damage, and produce a prioritized verdict — which to remove, which to defer/lazy-load, which to keep
but mitigate. Don't tell me the costs. Exam me on attribution and on the remove/defer/keep decision
framework. Verdict: can I audit and triage third-party cost (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The damage: third-party scripts (analytics, tag managers, ads, chat, A/B tools) add **bytes** (download), **main-thread work** (parse + execute, often blocking interactivity → bad INP/TBT), and sometimes **render-blocking** behavior. A **tag manager** is especially dangerous because it loads *more* scripts dynamically at runtime — so the cost is unbounded and invisible at build time.
- Attribution: DevTools (and the "third-party usage" view) let you attribute bytes and main-thread time **per origin**, so you can name which third party costs what — turning "the page is slow" into "the chat widget is eating 400ms of main thread."
- The triage framework (Exercise 2): for each third party — **remove** (is it even used? pure bloat?), **defer/lazy-load** (load on interaction or after the page is interactive — next module's facades), or **keep but mitigate** (it's business-critical, so isolate/optimize it). Business value vs performance cost is a real negotiation.

The principle: **third parties are code you don't control that taxes your bytes, main thread, and sometimes rendering — and a tag manager can load unbounded further scripts — so you must attribute cost per-origin and triage each as remove / defer / keep-but-mitigate.** The third-party tax is often the single biggest gap between a clean first-party page and a slow real-world site, and quantifying it is a high-value, frequently-overlooked skill.
</details>

---

## Module 8.2 — Facades: Loading Heavy Embeds on Interaction

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** React/HTML + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- No dedicated FM lesson; builds directly on 8.1. Lean on primary sources.

**Primary sources:**
- **web.dev — Lazy-load third-party resources with facades** — https://web.dev/articles/third-party-facades — *Focus on: showing a lightweight placeholder (facade) that loads the real heavy embed only on interaction.*
- **web.dev — lite-youtube-embed (example)** — https://github.com/paulirish/lite-youtube-embed — *Focus on: a real facade implementation.*

**Search prompts:**
- `"third party facade load on interaction youtube embed"`
- `"lite youtube embed facade performance"`
- Ask an AI: *"Explain the facade pattern for heavy third-party embeds (YouTube, maps, chat, social widgets): show a lightweight static placeholder (an image + play button) and only load the real heavy iframe/script when the user clicks/interacts. Why does this dramatically cut initial load cost, and what's the UX tradeoff (one extra click, slight delay on first interaction)?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.2, Exercise 1 — facades, build challenge.
Completed: Tracks 0–6, Module 8.1.

Give me a spec (no solution code): a page with heavy embeds loaded eagerly on page load — a video
embed (YouTube-style heavy iframe + scripts), a map, and a chat widget — tanking initial CWV. My job:
replace each with a FACADE — a lightweight placeholder (image/button) that loads the real embed only
on click/interaction. I build it.

Review me like a senior. Measure the eager baseline first (the embeds' cost on initial load). If my
facade still loads the heavy thing eagerly, ask what "on interaction" means. If my facade causes
layout shift when the real embed swaps in, ask what space I reserved. Re-measure initial load with
facades. Exam me on: why facades cut initial cost, the UX tradeoff, and the CLS pitfall.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.2, Exercise 2 (transfer) — facades, new
embeds. Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT page with different heavy embeds (e.g. a social-media embed, an analytics-heavy
widget, a comments system). Have me, solo: facade the ones that should be deferred, keep correct space
reservation (no CLS), make the first interaction smooth, and prove the initial-load improvement. The
judgment: not everything should be a facade (something truly above-the-fold and central might need to
load). Don't tell me which. Exam me on the facade pattern and when it's appropriate. Verdict: can I
apply facades correctly (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- The pattern: instead of loading a heavy embed (a YouTube iframe pulls hundreds of KB of script; a map widget, a chat SDK) on page load, render a **facade** — a lightweight static placeholder (a thumbnail image + a play/open button) — and only load the **real** embed when the user **interacts** with it. Most users never interact with most embeds, so this saves the full cost for them entirely.
- Measure: the eager version pays the embed's full byte + main-thread cost on initial load (bad LCP/INP/TBT); the facade version pays almost nothing until interaction.
- The pitfalls a reviewer checks: the facade must actually defer the heavy load (not load it hidden); it must **reserve the embed's space** so swapping in the real embed causes **no CLS**; and the first interaction should feel smooth (a brief load is acceptable — the UX tradeoff is one extra click + slight first-interaction delay in exchange for a much faster page).
- The judgment (Exercise 2): facades suit embeds that aren't immediately essential; something genuinely central and above-the-fold might warrant eager loading.

The principle: **a facade replaces a heavy third-party embed with a lightweight placeholder that loads the real thing only on interaction — saving the full cost for the majority who never interact — at the cost of one extra click, and only if space is reserved to avoid CLS.** It's the highest-leverage fix for embed-heavy pages and a direct application of the third-party triage from 8.1.
</details>

---

## Module 8.3 — Partytown & Offloading Third-Parties to a Worker

**Difficulty:** ●●●●○ · **Format:** Investigation + build · **Stack:** React/HTML + Partytown + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- No dedicated FM lesson; advanced third-party mitigation. Lean on primary sources.

**Primary sources:**
- **Partytown — Introduction** — https://partytown.builder.io/ — *Focus on: running third-party scripts in a Web Worker so they don't block the main thread, and the constraints.*
- **web.dev — Offload third-party scripts (concept)** — search "partytown web worker third party" — *Focus on: which scripts can be offloaded and which can't.*

**Search prompts:**
- `"partytown run third party scripts web worker"`
- `"offload analytics to web worker main thread"`
- Ask an AI: *"Explain Partytown: how it runs third-party scripts (analytics, tag managers) inside a Web Worker so their execution doesn't block the main thread, using a proxy to give them sandboxed access to DOM/APIs. What kinds of scripts work well with it, what are the limitations/overhead, and when is it worth the added complexity vs just removing or facading the script?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.3, Exercise 1 — offloading third parties.
Completed: Tracks 0–6, Modules 8.1–8.2.

Build me a page where main-thread-heavy third-party-style scripts (analytics/tag-style work) block
interactivity (bad INP/TBT) — proven in a trace. Then have me offload them to a worker with Partytown
(you tell me setup), and re-measure: the main thread should free up, INP/TBT improve. Do NOT pre-explain
which scripts benefit. Have me also find a script that does NOT work well offloaded (needs synchronous
DOM/timing) and recognize the limitation. Exam me on: how worker-offloading frees the main thread, the
proxy/constraints, and when offloading is the right tool vs remove/facade.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.3, Exercise 2 (transfer) — offload decision,
new mix. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT set of third-party scripts. Have me, solo: decide for EACH whether the right move
is remove, facade, offload-to-worker, or keep-and-mitigate — apply the offloading where it fits, prove
the main-thread improvement, and justify why offloading wasn't right for the others. Don't tell me the
decisions. Exam me on the full third-party mitigation toolkit and choosing among its options. Verdict:
can I choose the right third-party mitigation per script (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

- The technique: **Partytown** runs third-party scripts inside a **Web Worker** instead of the main thread, so their parse/execute work doesn't block your UI (improving INP/TBT). Because workers can't directly touch the DOM, Partytown uses a **proxy** to forward the scripts' DOM/API calls back to the main thread in a controlled way.
- Measure: heavy analytics/tag work that was blocking the main thread (visible in the trace) moves off it; the main thread frees up, interactivity improves.
- The limitations (the lesson): not every script offloads cleanly — scripts that need **synchronous DOM access**, precise timing, or that manipulate the page heavily can break or behave oddly through the proxy; there's also setup complexity and proxy overhead. So it's not a universal fix.
- The full toolkit (Exercise 2): for each third party, choose **remove** (8.1), **facade** (8.2), **offload to worker** (this module), or **keep-and-mitigate** (async/defer, load late). Offloading is one option, not the default.

The principle: **Partytown moves third-party scripts to a Web Worker so they stop blocking the main thread, via a proxy for DOM access — a powerful mitigation for main-thread-heavy analytics/tags, but limited to scripts that don't need synchronous DOM/timing, so it's one tool in a toolkit alongside remove, facade, and defer.** Choosing the right mitigation per script is the senior skill this whole third-party arc builds toward.
</details>

---

## Module 8.4 — RUM Dashboards: SLOs, Sampling & Segmentation

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** JS + Node server + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the **Real User Monitoring** material (revisit from Track 1.7) — now extended to dashboards, SLOs, and segmentation.

**Primary sources:**
- **web.dev — Monitor performance with RUM** — search "web.dev real user monitoring" — *Focus on: tracking field CWV over time, by segment.*
- **Google SRE — Service Level Objectives** — https://sre.google/sre-book/service-level-objectives/ — *Focus on: SLI/SLO/error-budget thinking applied to a performance target.*
- **web.dev — CrUX / field data segmentation** — https://developer.chrome.com/docs/crux — *Focus on: segmenting by device, connection, country.*

**Search prompts:**
- `"rum dashboard p75 cwv over time segment device connection"`
- `"performance slo error budget web vitals"`
- `"rum sampling rate cost"`
- Ask an AI: *"Explain a production RUM setup beyond a single beacon: tracking P75 CWV over TIME (trends), SEGMENTING by device class / connection / country / page type, setting a performance SLO (e.g. 'P75 LCP < 2.5s for 90% of page-type-X views') with an error budget, and SAMPLING (why you don't send every session, and the cost/accuracy tradeoff). How do segments reveal problems the aggregate hides?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.4, Exercise 1 — RUM dashboards, build
challenge. Completed: Tracks 0–6, Modules 8.1–8.3. Node server + the RUM beacon from Track 1.7.

Extend my RUM beacon into a small DASHBOARD spec (I build it): collect CWV + context (device class,
connection, page type) from simulated sessions, store them, and compute/display P75 per metric OVER
TIME and BY SEGMENT, plus an SLO line (e.g. P75 LCP < 2.5s) showing pass/fail. Seed it with data where
the AGGREGATE looks fine but one SEGMENT (e.g. low-end Android / slow connection) is failing.

Review me like a senior. If I only show the aggregate, ask what it's hiding (make me segment and find
the failing cohort). If I send every session, ask about sampling cost at scale. If my SLO has no
error budget concept, ask how I'd know I'm "off track." Exam me on: trends vs snapshots, segmentation,
SLOs/error budgets, and sampling.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.4, Exercise 2 (transfer) — diagnose from RUM,
new data. Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT seeded RUM dataset hiding a real problem in a segment (e.g. one page type
regressed after a date, or one country/device is far worse). Have me, solo: explore the dashboard,
SEGMENT to find the hidden problem, state which cohort is affected and form a hypothesis about the
cause, and define an appropriate SLO. Don't tell me where the problem is. Exam me on reading RUM data
and on SLO/segmentation thinking. Verdict: can I run RUM as an ongoing monitoring discipline (proceed),
or redo Exercise 1?
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- A dashboard that tracks **P75 CWV over time** (trends — so you see regressions and improvements, not just a snapshot) and **by segment** (device class, connection, country, page type). The seeded data's lesson: the **aggregate looks fine** while a **specific segment** (low-end Android on a slow connection) is failing — exactly your target users. Only segmentation reveals it.
- **SLOs:** a concrete target like "P75 LCP < 2.5s for page type X," ideally with an **error budget** (you allow some fraction of bad sessions; when you exceed it, you stop shipping features and fix performance). This is borrowed from SRE and is how mature teams keep performance from rotting.
- **Sampling:** at real scale you don't beacon every session (cost, volume) — you sample a fraction, accepting some statistical noise for affordability. Know the tradeoff.
- Why segmentation matters most: "our P75 is fine" can hide that your growth market (e.g. India, low-end Android) has a terrible experience — the segment view is where real, business-relevant problems surface.

The principle: **production RUM is trends + segmentation + SLOs + sampling, not a single number — because the aggregate hides the segments (low-end devices, slow connections, specific pages) where real users suffer, and an SLO with an error budget is how you keep performance from regressing over time.** This is performance as an ongoing operational discipline, and it's exactly the kind of system-level thinking that marks a senior/staff engineer.
</details>

---

## Module 8.5 — Performance Budgets Enforced in CI

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** Node + Lighthouse CI + GitHub Actions

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the budgets/goals material (revisit from Track 1.9) — now enforced automatically.

**Primary sources:**
- **web.dev — Using Lighthouse CI** — https://web.dev/articles/lighthouse-ci — *Focus on: running Lighthouse in CI and asserting against thresholds.*
- **Lighthouse CI — Getting started** — https://github.com/GoogleChrome/lighthouse-ci/blob/main/docs/getting-started.md — *Focus on: `assert` config and failing the build on regression.*
- **web.dev — Performance budgets with bundlesize / size-limit** — search — *Focus on: failing CI when a bundle grows past its budget.*

**Search prompts:**
- `"lighthouse ci assert budget fail build"`
- `"size-limit bundlesize ci fail performance budget"`
- Ask an AI: *"How do I enforce a performance budget in CI so regressions are caught before merge? Explain Lighthouse CI (run Lighthouse on a PR, assert CWV/score thresholds, fail the build if exceeded) and bundle-size budgets (size-limit/bundlesize failing CI when JS grows past a limit). Why is an ENFORCED budget the only kind that actually works?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.5, Exercise 1 — budgets in CI, build
challenge. Completed: Tracks 0–6, Modules 8.1–8.4.

Give me a small app + a CI setup (GitHub Actions or a local CI script) and have me, no solution code:
configure Lighthouse CI to run on the app and ASSERT performance thresholds (CWV/score), plus a
bundle-size budget (size-limit) — and wire it so a build FAILS when a budget is exceeded. Then have me
introduce a regression (a heavy import, an oversized image) and watch CI FAIL, then fix it and watch it
pass.

Review me like a senior. If my thresholds are so loose nothing ever fails, ask what the budget is for.
If I assert only the Lighthouse score (not specific metrics/bytes), ask what that misses. Exam me on:
why enforced budgets work and advisory ones don't, what to assert (metrics + bytes), and how this
prevents slow regressions from merging.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.5, Exercise 2 (transfer) — CI budget for a
real-ish repo. Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT app and have me, solo: design and wire a complete CI performance gate — Lighthouse
CI assertions at sensible thresholds (justified, ratcheted from baseline per Track 1.9) AND bundle-size
budgets per entry/chunk — then prove it by introducing and catching a regression. Don't tell me the
thresholds. Exam me on threshold-setting, what to gate on, and the developer-experience tradeoff (too
strict = noise, too loose = useless). Verdict: can I stand up a CI performance gate at work (proceed),
or redo Exercise 1?
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- The setup: **Lighthouse CI** runs Lighthouse on each PR/commit and **asserts** against configured thresholds (CWV, specific audits, score); **size-limit/bundlesize** asserts JS bundle sizes against per-entry budgets. Wired into CI (GitHub Actions), exceeding a budget **fails the build**, blocking the merge.
- The proof: introduce a regression (a heavy eager import, an unoptimized image, a bundle that grew) → CI goes red; fix it → green. This is the mechanism that keeps a fast app fast as a whole team ships to it.
- What to assert (the lesson): not just the overall Lighthouse score (too coarse, noisy) — assert **specific metrics** (LCP, TBT/INP proxy, CLS) and **byte budgets** per bundle, since those map to concrete causes. Account for Lighthouse's run-to-run variance (Track 1.8) by using medians/multiple runs so CI isn't flaky.
- The threshold judgment: too loose and nothing ever fails (theater); too strict and CI is constant noise that developers learn to ignore or bypass. Ratchet from the current baseline (Track 1.9) so it's achievable and tightening.

The principle: **a performance budget only works when it's ENFORCED — Lighthouse CI asserting metrics and size-limit asserting bytes, failing the build on regression — because that's what stops a slow change from merging; an advisory budget nobody enforces is decoration.** Automating the gate is what makes performance durable across a team, and building one is a clear staff-level contribution.
</details>

---

## Module 8.6 — Regression Detection & Bundle-Size Bots

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** Node + CI + bundle tooling

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- No dedicated FM lesson; extends 8.5. Lean on primary sources.

**Primary sources:**
- **size-limit (with CI comment action)** — https://github.com/ai/size-limit — *Focus on: reporting bundle-size DELTAS on a PR.*
- **web.dev — Monitoring bundle size** — search "track bundle size over time ci" — *Focus on: catching gradual growth.*
- **Statoscope / source-map-explorer** — search — *Focus on: explaining WHAT grew when a bundle regresses.*

**Search prompts:**
- `"size-limit github action pr bundle size delta comment"`
- `"detect bundle size regression what grew source-map-explorer"`
- Ask an AI: *"Explain bundle-size regression detection on PRs: a bot that comments the size delta of each bundle on a pull request (e.g. '+18KB to main chunk'), so reviewers see the cost of a change before merging. How do tools like source-map-explorer/Statoscope explain WHAT caused the growth (a new dependency, a duplicate)? Why is catching the +5KB-per-PR creep important — the 'death by a thousand cuts' problem?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.6, Exercise 1 — regression detection.
Completed: Tracks 0–6, Modules 8.1–8.5.

Set me up (I do the config) with a bundle-size bot on PRs that reports the size DELTA per chunk, and a
way to explain WHAT grew (source-map-explorer or similar). Have me make a PR that adds a heavy
dependency, see the bot report the delta, then use the explainer to identify exactly what caused the
growth and decide whether it's justified or fixable (a lighter alternative, tree-shaking, dynamic
import). Do NOT pre-explain the cause.

Review me like a senior. If I only see "bundle got bigger" without knowing WHY, push me to the
explainer. If the growth is a duplicate dependency, ask how that happened. Exam me on: per-PR delta
reporting, explaining growth, the thousand-cuts problem, and deciding justified vs fixable growth.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.6, Exercise 2 (transfer) — investigate a
regression solo. Apply the Exercise 2 rules: solo first; verdict.

Give me a repo with a bundle that grew for a NON-OBVIOUS reason (e.g. a tree-shaking failure, a
duplicated dependency at two versions, a barrel-file import pulling in everything, or an accidental
moment.js-style heavy lib). Have me, solo: detect the regression, use the tooling to find the real
cause, and fix it (lighter lib / proper imports / dedupe). Don't tell me the cause. Exam me on the
investigation method. Verdict: can I detect and root-cause a bundle regression (proceed), or redo
Exercise 1?
```

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

- The setup: a **bundle-size bot** (size-limit's GitHub action or similar) comments the **size delta per chunk** on each PR — so a reviewer sees "this PR adds 18KB to the main bundle" *before* merging, making the performance cost of a change visible at review time.
- Explaining growth: when a bundle grows, **source-map-explorer / Statoscope** show *what's inside* — letting you find that the +18KB was a heavy date library, or a **duplicated dependency** at two versions, or a **barrel-file (`index.ts`) import** that pulled in an entire library instead of one function, or a **tree-shaking failure**.
- The decision: is the growth **justified** (a genuinely needed feature) or **fixable** (a lighter alternative, a dynamic import, proper named imports, deduping)? Often it's fixable.
- The thousand-cuts problem (the core lesson): no single PR makes a bundle "too big," but **+3KB per PR over months** silently bloats it. Per-PR delta reporting is what catches the creep that a one-time audit misses — it makes bundle size a continuous, visible concern.

The principle: **regression detection makes the performance cost of each change visible at review time (per-PR size deltas) and explains what caused any growth (source-map-explorer) — catching the death-by-a-thousand-cuts creep that no one-time audit can, and letting you root-cause non-obvious bloat like duplicate deps or barrel imports.** This is how a codebase stays lean over years, and it's a high-leverage system to introduce on any team.
</details>

---

## Module 8.7 — Proving Business Impact: Performance & Conversion

**Difficulty:** ●●●○○ · **Format:** Investigation · **Stack:** Analysis + reasoning

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Why Performance Matters"** / business-case material — speed as a revenue/conversion lever, not a vanity metric.

**Primary sources:**
- **web.dev — Why speed matters / business impact** — https://web.dev/why-speed-matters/ — *Focus on: the documented links between speed, bounce, engagement, and conversion.*
- **web.dev — Case studies** — https://web.dev/case-studies — *Focus on: real companies' before/after revenue and conversion numbers.*
- **WPO Stats** — https://wpostats.com/ — *Focus on: a catalog of performance→business-metric correlations.*

**Search prompts:**
- `"performance conversion rate case study revenue impact"`
- `"page speed bounce rate business metric wpostats"`
- Ask an AI: *"How do I make the BUSINESS case for performance work? Walk me through the documented relationships between speed and business metrics (bounce rate, conversion, engagement, revenue), how to estimate the impact of an improvement for a specific site (using its traffic + conversion + the elasticity from case studies), and how to frame a performance project to leadership in money terms, not milliseconds."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.7, Exercise 1 — proving business impact.
Completed: Tracks 0–6, Modules 8.1–8.6. Investigation/analysis; no app code.

Give me a realistic scenario: a site with given monthly traffic, conversion rate, average order value,
and a known performance problem (e.g. P75 LCP of 4.5s). Have me, using documented speed→conversion
relationships, ESTIMATE the business impact of fixing it to 2.5s — in revenue, not milliseconds — and
write the one-paragraph pitch I'd give leadership to fund the work. Do NOT hand me the numbers; make me
reason from the case-study elasticities and the site's funnel.

Push me like a skeptical exec: "prove this is worth an engineer-month." Make me show the estimate's
assumptions and how I'd MEASURE the actual impact after shipping (a before/after on conversion, ideally
controlled). Exam me on: the speed→business links, estimating impact for a specific funnel, and framing
in money.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.7, Exercise 2 (transfer) — business case, new
scenario. Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT business (different model — e.g. ad-supported media, SaaS signup, marketplace) with
its own metrics. Have me, solo: identify which performance metric matters most for THIS business model,
estimate the impact of a specific improvement in that model's money terms, and write the funding pitch
+ the post-ship measurement plan. Don't tell me the framing. Exam me on tailoring the business case to
the model and on honest measurement of impact. Verdict: can I make a credible, model-appropriate
business case for performance work (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What This Is Teaching (the career module)

<details>
<summary>Click only after you finish or give up</summary>

This is the module that turns performance skill into salary and influence. What you should now be able to do:

- **Connect speed to money.** There's a well-documented relationship between speed and business metrics: slower pages → higher bounce, lower engagement, lower conversion; faster pages → measurable lifts. The exact elasticity varies, but the direction and rough magnitude are established across many case studies (WPO Stats, web.dev case studies).
- **Estimate impact for a specific funnel.** Given a site's traffic, conversion rate, and order value (or its model's equivalent), apply a plausible conversion lift from the case-study evidence to estimate revenue impact of a specific improvement — framed in **money, not milliseconds**.
- **Tailor to the business model** (Exercise 2): for e-commerce it's conversion/revenue; for ad-supported media it's pageviews/engagement/ad revenue; for SaaS it's signup/activation; for a marketplace it's both sides' engagement. The metric that matters depends on how the business makes money.
- **Frame the pitch and the measurement.** Leadership funds money, not LCP. Make the estimate with explicit assumptions, then commit to **measuring the actual impact** after shipping (a controlled before/after on the business metric) so you can prove it worked — which funds the next project and builds your reputation.

The principle: **performance is a business lever, and the highest-value skill is translating a technical improvement into an estimated and then measured impact on the metric a specific business model cares about — framing the work in money, not milliseconds.** This is what gets performance work prioritized, gets you credited for revenue, and directly supports the senior/staff trajectory and the ₹4–5L+ goal: an engineer who can connect their work to business outcomes is paid as one.
</details>

---

## Module 8.8 — Real Low-End Device Testing (Your Parents' Phones)

**Difficulty:** ●●●●○ · **Format:** Investigation + hands-on · **Stack:** Real Android device + Chrome remote debugging

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the device/throttling material — and recall the "you are not your user" rule that's run through the whole primer. This module makes it literal.

**Primary sources:**
- **Chrome — Remote debug Android devices** — https://developer.chrome.com/docs/devtools/remote-debugging — *Read fully. Focus on: enabling USB debugging and inspecting the real device's pages from your laptop's DevTools.*
- **Chrome — Remote debugging: get started** — https://developer.chrome.com/docs/devtools/remote-debugging/local-server — *Focus on: port forwarding so the phone can reach your local dev server.*
- **web.dev — Why you should test on real low-end devices** — search "test on real low-end android performance" — *Focus on: how simulated throttling underestimates real cheap-phone pain (thermal, memory, real CPU).*

**Search prompts:**
- `"chrome remote debugging android usb devtools real device"`
- `"port forwarding chrome inspect localhost on phone"`
- `"real low-end android testing vs cpu throttling difference"`
- Ask an AI: *"Walk me step by step through remote-debugging a real Android phone from my laptop: enabling Developer Options + USB debugging on the phone, connecting via USB, opening chrome://inspect, inspecting the phone's tabs with full DevTools (Performance, Network), and setting up PORT FORWARDING so the phone can load my laptop's localhost dev server. Then: why does a REAL cheap phone reveal problems (thermal throttling, limited memory, real slow CPU/GPU) that DevTools' simulated 4x throttle underestimates?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.8, Exercise 1 — real low-end device testing.
Completed: Tracks 0–6, Modules 8.1–8.7. HANDS-ON with a real low-end Android phone (e.g. a parent's).

This is the literal "you are not your user" module. Walk me through, step by step: enabling Developer
Options + USB debugging on the Android phone, connecting it to my laptop, opening chrome://inspect, and
inspecting a real page running ON THE PHONE with my laptop's DevTools (Performance trace + Network).
Then set up PORT FORWARDING so the phone can load one of my earlier modules' apps from my laptop's
localhost.

Have me run a Performance trace of the SAME page three ways and compare: (a) my laptop unthrottled,
(b) my laptop with 4x–6x CPU + Slow 4G simulated throttle, (c) the REAL low-end phone. Make me observe
where the real device is WORSE than the simulation (thermal throttling over time, memory limits, real
GPU limits on the animations from Track 4). Do NOT pre-explain the gaps — make me find them. Exam me
on: the remote-debug setup, port forwarding, and specifically WHERE and WHY real-device behavior
diverges from simulated throttling.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.8, Exercise 2 (transfer) — real-device
diagnosis. Apply the Exercise 2 rules: solo first; verdict.

Have me, solo, take a DIFFERENT app (ideally one of my janky-animation or heavy-JS modules from Tracks
3–4, or a real site) and profile it ON THE REAL PHONE via remote debugging — diagnosing a problem that
is WORSE or only-visible on the real device than in simulation (e.g. an animation that's fine throttled
but melts the real GPU, or memory pressure that only bites on the real device). Produce a verdict on
the real-device experience and the fix. Near-zero hints. Exam me on real-device profiling and on what
simulation misses. Verdict: can I test and diagnose on genuine low-end hardware — the ultimate
"you are not your user" skill (proceed to the capstone track), or redo Exercise 1?
```

### 🔒 SPOILER — What This Is Teaching (the skill you specifically wanted)

<details>
<summary>Click only after you finish or give up</summary>

This module teaches the exact skill you asked for — testing on the real low-end phones you have access to. What you should now be able to do:

- **Set up remote debugging:** on the Android phone, enable **Developer Options** (tap Build Number 7×) → **USB debugging**; connect via USB; on your laptop open **`chrome://inspect`**; the phone's open tabs appear and you can **inspect** them with your laptop's full DevTools — run a real **Performance trace** and **Network** capture of pages *as the phone actually runs them*.
- **Port forwarding:** configure it in chrome://inspect so the phone can load **`localhost:<port>`** from your laptop's dev server — letting you test your in-development apps (and all your earlier modules) on the real device.
- **The core comparison:** profile the same page (a) unthrottled laptop, (b) simulated 4x–6x CPU + Slow 4G, (c) the real low-end phone — and observe that the **real device is often worse** than the simulation, because simulation doesn't fully capture **thermal throttling** (the phone slows as it heats during sustained work), **real memory limits** (GC pressure and crashes the simulation won't show), and **real GPU limits** (the Track-4 animations that were "fine throttled" can melt a real cheap GPU).
- **The payoff:** this makes "you are not your user" literal and completes your stated goal of predicting weak-hardware behavior — not by guessing from a desktop throttle, but by *actually running on the hardware your users have*.

The principle: **real low-end-device testing via Chrome remote debugging (USB debugging + chrome://inspect + port forwarding) reveals what simulated throttling underestimates — thermal throttling, real memory and GPU limits — making it the ground truth for the "you are not your user" rule.** Simulated throttling is your everyday tool; the real device is the final verdict, and now you can get it whenever you need it.
</details>

---

## Module 8.9 — Memory Leaks in Long-Lived SPAs

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + Chrome memory tools

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Blazingly Fast JavaScript** (ThePrimeagen) → the memory/GC material (revisit from Track 3.6) — now applied to leaks that accumulate over a long session rather than per-frame churn.

**Primary sources:**
- **Chrome DevTools — Fix memory problems** — https://developer.chrome.com/docs/devtools/memory-problems — *Focus on: heap snapshots, comparing snapshots, and the allocation timeline to find growth.*
- **MDN — Memory management / leaks** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_management — *Focus on: common leak sources (detached DOM, lingering listeners/timers, closures).*
- **web.dev — Detached DOM / memory leaks** — search — *Focus on: detached nodes retained by JS references.*

**Search prompts:**
- `"chrome devtools heap snapshot find memory leak detached dom"`
- `"react memory leak event listener timer cleanup useEffect"`
- Ask an AI: *"Explain memory leaks in long-lived SPAs: common causes (event listeners/timers/subscriptions not cleaned up, detached DOM nodes retained by JS, growing caches, closures capturing large objects, useEffect missing cleanup) and how to find them with Chrome heap snapshots (take snapshot → interact → take another → compare for objects that should have been freed but grew). Why do leaks matter more in SPAs that stay open for hours?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.9, Exercise 1 — memory leaks in SPAs.
Completed: Tracks 0–6, Modules 8.1–8.8.

Build me a React SPA that LEAKS as you navigate/interact repeatedly: a component that adds an event
listener or interval (or a subscription) without cleaning it up on unmount, and/or detached DOM
retained by a lingering reference, and/or an ever-growing cache. Over many navigations, memory climbs
and it eventually janks/slows. Do NOT name the leak.

Have me reproduce the growth: take a heap snapshot, interact/navigate repeatedly, take another, and
COMPARE to find what's growing that shouldn't (detached nodes, listener counts). Trace it to the
missing cleanup, fix it (useEffect cleanup, removeEventListener, clearInterval, unsubscribe, bounded
cache), and prove memory stabilizes. Exam me on: common SPA leak sources, reading a snapshot
comparison, and why leaks bite long-lived sessions specifically.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.9, Exercise 2 (transfer) — leak hunt, new
cause. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT leaking SPA with a different leak source (e.g. a closure capturing a large object,
a global registry that never evicts, an observer not disconnected). Have me, solo: detect the growth
via snapshots/allocation timeline, root-cause it, fix it, and prove stabilization. Don't tell me the
cause. Exam me on the leak-hunting method and prevention patterns. Verdict: can I find and fix a real
SPA memory leak (proceed to Track 9), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The leaks (one per exercise): an **event listener / interval / subscription not cleaned up** on unmount (every mount adds another, and the old ones — plus everything they close over — can't be collected); **detached DOM nodes** still referenced by JS (removed from the document but kept alive by a variable/closure); an **ever-growing cache/registry** that never evicts; a **closure capturing a large object** that outlives its usefulness. In React, the usual root cause is a **`useEffect` missing its cleanup function**.
- Finding it: **heap snapshots** — take one, interact/navigate repeatedly, take another, and **compare**; objects (detached DOM nodes, listener counts, your data) that **grew and weren't freed** are the leak. The allocation timeline shows retained growth that never drops.
- The fix: **clean up** — `useEffect` return cleanup, `removeEventListener`, `clearInterval`, `unsubscribe`, `observer.disconnect()`, and bound/evict caches. Re-measure: memory stabilizes across repeated interactions instead of climbing.
- Why SPAs specifically: a traditional page reload clears everything; an SPA that stays open for hours **accumulates** leaks until it slows, janks (GC pressure from Track 3.6), or crashes the tab — especially painful on the low-memory devices from 8.8.

The principle: **long-lived SPAs leak when listeners, timers, subscriptions, detached DOM, or unbounded caches aren't cleaned up — accumulating over a session until performance degrades — and you find them by comparing heap snapshots for what grew and shouldn't have, then adding the missing cleanup.** Leak-hunting is a senior diagnostic skill, and it's where Track 3's memory knowledge meets real, long-running production apps.
</details>

---

## Module 8.10 — The Performance Culture Playbook (Scale Capstone)

**Difficulty:** ●●●●● · **Format:** Investigation + synthesis · **Stack:** Reasoning + light setup

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- Revisit the **Front-End System Design** material and **Web Performance Fundamentals, v2**'s goals/monitoring sections. This capstone synthesizes the whole track into how you'd run performance for a team.

**Primary sources:**
- **web.dev — Making performance stick / performance culture** — search "web.dev performance culture" — *Focus on: process, ownership, and keeping performance from regressing.*
- **web.dev — Establishing performance budgets & governance** — https://web.dev/articles/performance-budgets-101 — *Focus on: budgets + monitoring + CI as a system.*

**Search prompts:**
- `"performance culture team process ownership regression"`
- `"frontend performance governance budgets monitoring ci"`
- Ask an AI: *"How does a team keep a product fast over years (not just fix it once)? Synthesize a 'performance culture playbook': budgets (set + ratcheted), CI enforcement, RUM monitoring with SLOs and segmentation, regression detection on PRs, third-party governance, real-device testing, and clear ownership — plus how to get buy-in via business-impact framing. What's the minimum viable version for a small team vs the full version?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.10, Exercise 1 — performance culture playbook,
scale capstone. Completed: Tracks 0–6, Modules 8.1–8.9. Synthesis; minimal code.

Give me a realistic company scenario (e.g. my own startup, or a mid-size product with a perf problem
and no process). Have me design the full PERFORMANCE PROGRAM that would keep it fast: budgets (Track
1.9) + CI enforcement (8.5) + RUM with SLOs and segmentation (8.4) + PR regression detection (8.6) +
third-party governance (8.1–8.3) + real-device testing (8.8) + leak vigilance (8.9) + the business-case
framing (8.7) + clear ownership. Make me also define the MINIMUM VIABLE version for a small team that
can't do all of it at once, and the sequence to roll it out.

Push me like a skeptical eng lead: "we have no time for this — what are the two things that matter
most?" Make me prioritize ruthlessly and justify. Exam me on how the pieces fit into a system and the
MVP-vs-full tradeoff.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 8, Module 8.10, Exercise 2 (transfer) — playbook for a
different org. Apply the Exercise 2 rules: solo first; verdict.

Give me a DIFFERENT organization with different constraints (e.g. a large legacy app with entrenched
third parties, or a fast-growing startup shipping daily with no guardrails). Have me, solo: design the
performance program and rollout sequence tailored to ITS constraints, naming what to do first, what to
automate, who owns it, and how to prove value to keep it funded. Don't tell me the plan. Exam me on
tailoring the program to the org and on sequencing. Verdict: could I credibly OWN performance for a
team/org (the staff-level signal) — proceed to Track 9, or redo Exercise 1?
```

### 🔒 SPOILER — What This Is Teaching (the staff-level synthesis)

<details>
<summary>Click only after you finish or give up</summary>

This capstone synthesizes the whole track into the thing that distinguishes a senior who fixes pages from a **staff engineer who owns performance for an organization**. What you should now be able to articulate:

- **The full system:** performance stays good only as an ongoing program — **budgets** (set and ratcheted, 1.9) + **CI enforcement** (8.5) so regressions can't merge + **RUM with SLOs and segmentation** (8.4) so you know the real-user truth and catch field regressions + **PR regression detection** (8.6) for the thousand-cuts creep + **third-party governance** (8.1–8.3) so external scripts don't silently wreck things + **real-device testing** (8.8) for ground truth + **leak vigilance** (8.9) + the **business-case framing** (8.7) to keep it funded + **clear ownership** (someone is accountable).
- **MVP vs full:** a small team can't do all of it at once. The highest-leverage minimum is usually: a **CI budget on bundle size + key metrics** (stops regressions cheaply) and **basic RUM** (tells you the real-user truth). Everything else layers on as the team grows.
- **Sequencing and prioritization:** roll out in order of leverage and effort; tailor to the org's constraints (a legacy app with entrenched third parties has a different first move than a greenfield startup). Be able to answer "what are the two things that matter most?" decisively.
- **Ownership and influence:** the staff-level move is making performance a *property of the system and the process*, not a heroic one-time cleanup — and proving its business value so it stays resourced.

The principle: **keeping a product fast over years is a system — budgets, CI enforcement, RUM/SLOs, regression detection, third-party governance, real-device testing, leak vigilance, business framing, and ownership — and the staff-level skill is designing that program, sequencing it for the org's real constraints, and proving its value.** This is the capstone of performance as a discipline, and being able to own it is precisely what justifies a senior/staff title and the compensation that comes with it.
</details>

---

*End of Track 8 — Third-Parties, Monitoring & Performance at Scale. 10 modules. You can now tame third parties, monitor real users with SLOs, enforce budgets in CI, catch regressions, test on genuine low-end hardware, hunt memory leaks, prove business impact, and design a performance program a whole team runs on.*

*Next: Track 9 — Capstones: Diagnose Anything, where everything converges into the exact skill you set out to build.*
