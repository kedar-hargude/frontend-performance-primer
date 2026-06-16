# TRACK 2 — THE NETWORK & LOADING PIPELINE

*The first of the three buckets, and usually the biggest source of "the page takes forever to appear." This track makes you fluent in everything between the user pressing Enter and the first meaningful pixel: the waterfall, the server's first byte, the protocols, compression, caching, CDNs, resource hints, and the critical path. By the end you can look at a network waterfall and name exactly which request, which phase, and which dependency is costing the user their first impression.*

*Prerequisites: Track 0 (you must be able to read a waterfall) and Track 1 (you must measure before/after). A small local Node server is used in several modules — that's expected and fine.*

*The spine course for this whole track is **Web Performance Fundamentals, v2** (Todd Gardner) — specifically its TTFB, FCP, compression, protocols, caching, and CDN material. Watch the relevant section before each module; it's the best practical walkthrough of this exact pipeline.*

---

## Module 2.1 — The Critical Path: What Has to Happen Before First Paint

**Difficulty:** ●●○○○ · **Format:** Investigation + broken project · **Stack:** Vanilla HTML/CSS/JS + local server

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Improving First Contentful Paint"** section (*Removing Sequence Chains*, *Preloading Resources*, *Lazy Loading Resources*) and the earlier **"Waterfall Charts"** lesson. This is the exact mental model of the critical path and how to shorten it.

**Primary sources:**

- **web.dev — Critical rendering path** — https://web.dev/articles/critical-rendering-path — *Focus on: the path from HTML bytes to first paint, and what blocks it.*
- **web.dev — Render-blocking resources** — https://web.dev/articles/render-blocking-resources — *Focus on: which resources hold up the first paint and how to unblock them.*
- **MDN — `async` vs `defer`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#attr-async — *Focus on: how each changes when a script downloads and executes relative to parsing.*

**Search prompts:**

- `"critical rendering path render blocking first paint"`
- `"async vs defer script parsing blocking"`
- `"request chain collapse dependencies fcp"`
- Ask an AI: *"Explain the critical path to first paint: what the browser must do (build DOM + CSSOM) before it can paint, why CSS and synchronous JS are render-blocking, and what `async`/`defer` change. What is a 'request chain' (sequence chain) and why does collapsing it speed up First Contentful Paint?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 2, Module 2.1, Exercise 1 — the critical path.
Completed: Tracks 0–1.

Build me a small site (served locally, production-style) whose First Contentful Paint is slow for
PURELY structural reasons — not heavy computation: a render-blocking synchronous script in the
<head>, a CSS→font request chain (a stylesheet that references a font, so the font can't even start
until the CSS parses), and a stylesheet positioned to delay first paint. Keep the content tiny so
the delay is all about HOW resources load. Do NOT tell me which resource is the problem.

Tell me how to run it and measure FCP (baseline first). Make me read the waterfall, find the render-
blockers and the request chain, and fix them ONE at a time (defer the script, collapse/preload the
chain), re-measuring FCP after each. Hold the measure-before-and-after loop. Exam me on: what blocks
first paint and why, async vs defer, and why a request chain costs a full round-trip per link.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 2, Module 2.1, Exercise 2 (transfer) — critical path,
new shape. Apply the Exercise 2 rules: solo first; verdict at the end.

Build me a DIFFERENT slow-FCP site where the blocking pattern is different — e.g. a deep request
chain (HTML → JS → JS → fetch → render) and a render-blocking CSS that's larger than it needs to be.
Don't tell me the structure. Have me, solo: baseline FCP, read the waterfall, diagnose every reason
the first paint is delayed, fix each one variable-at-a-time, and confirm the FCP delta. Exam me on
the critical-path model and on why deferring the wrong script (one the first paint actually needs)
would backfire. Verdict: can I diagnose and shorten a critical path cold (proceed), or redo
Exercise 1?
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- A synchronous `<script>` in the `<head>` (no `async`/`defer`) that blocks HTML parsing until it downloads and runs — delaying everything after it. Fix: `defer` (runs after parse, in order) or move to end of body.
- A request chain: HTML references a CSS file, which `@import`s or references a font/another CSS, so each link can only start after the previous one is fetched and parsed — each hop is a full round-trip. Fix: collapse the chain (bundle, inline critical CSS, `preload` the font so it starts immediately).
- A render-blocking stylesheet that holds first paint (CSS is render-blocking by nature) — mitigate by inlining critical CSS and/or splitting.
- The measurement story: baseline FCP is bad; each fix moves it; the waterfall visibly shortens.

The principle: **the browser cannot paint until it has the DOM and the CSSOM, so render-blocking CSS, parser-blocking synchronous JS, and multi-hop request chains directly delay First Contentful Paint — and you shorten the path by deferring non-critical scripts, collapsing chains, and preloading what the first paint truly needs.** The waterfall is where you *see* the critical path; FCP is the number that proves you shortened it.

</details>

---

## Module 2.2 — Time to First Byte: The Server's Head Start

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Node server + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Improving Time to First Byte"** section (*Baseline Time to First Byte*, *Host Capacity & Proximity*) plus the *"Time to First Byte (TTFB)"* metric lesson. Todd breaks TTFB into its sub-steps and shows what's in your control.

**Primary sources:**

- **web.dev — Time to First Byte (TTFB)** — https://web.dev/articles/ttfb — *Focus on: the sub-parts (redirects, DNS, connect, TLS, request, server processing) and good targets.*
- **MDN — Server Timing** — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Server-Timing — *Focus on: exposing server-side phase timings to DevTools so you can see WHERE the server's time went.*
- **web.dev — Reduce server response times** — search the term — *Focus on: caching, doing less per request, proximity.*

**Search prompts:**

- `"ttfb breakdown dns connect tls server processing"`
- `"server-timing header devtools"`
- `"reduce time to first byte server response"`
- Ask an AI: *"Break down Time to First Byte into its phases: redirects, DNS, TCP connect, TLS handshake, request send, and server processing time. Which are network/proximity costs and which are server-work costs? How does the Server-Timing header let me see server-side breakdown in DevTools, and what are the main levers to reduce TTFB?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 2, Module 2.2, Exercise 1 — Time to First Byte.
Completed: Tracks 0–1, Module 2.1. Uses a local Node server.

Build me a tiny Node server + page where TTFB is BAD for a server-work reason: the route does
something slow per request that it doesn't need to do every time (e.g. an artificial slow
computation or a synchronous "DB" call with no caching) before sending the first byte. Add a
Server-Timing header that breaks down the server's internal phases — but do NOT tell me which phase
is the culprit.

Have me measure baseline TTFB in DevTools (Timing tab) and read the Server-Timing breakdown. Make me
locate the slow server phase, then fix it (cache the expensive work, do less per request) and
re-measure. Keep the loop. Exam me on: the phases of TTFB, which are proximity/network vs server-
work, why TTFB gates FCP and LCP downstream, and what Server-Timing exposes.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 2, Module 2.2, Exercise 2 (transfer) — TTFB, new
cause. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT server/page where TTFB is bad for a different reason in the chain — e.g. an
unnecessary redirect (or chain of redirects) before the real response, plus a per-request cost that
should be memoized. Don't tell me which. Have me, solo: baseline TTFB, decompose it (redirect? DNS?
server processing?), fix each, and prove the delta. Exam me on why a redirect adds a full round-trip,
why server-processing TTFB and network-proximity TTFB need totally different fixes, and how a CDN
(next module) would change the proximity part. Verdict: can I decompose and attack a bad TTFB
(proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- Exercise 1: the route does expensive work on every request before responding (a slow synchronous computation or uncached "DB" lookup), inflating the **server-processing** portion of TTFB. The Server-Timing header reveals which internal phase ate the time. Fix: cache/memoize the expensive work so most requests skip it.
- Exercise 2: an unnecessary **redirect** (each redirect is a full extra round-trip before the real response even starts) plus a repeated per-request cost. Fix: remove the redirect (serve the final URL directly) and cache the work.
- TTFB decomposes into: redirects → DNS → TCP connect → TLS → request → **server processing** → first byte. Redirects/DNS/connect/TLS are largely **proximity and connection** costs (helped by CDNs, connection reuse, fewer origins); server processing is **your code's** cost (helped by caching and doing less).
- Why it matters downstream: TTFB is the floor for FCP and LCP — the browser can't paint content it hasn't started receiving, so a 1.5s TTFB caps how fast everything after it can be.

The principle: **TTFB is the sum of getting to the server, the handshake, and the server thinking — and you must decompose it to fix it, because a proximity problem (use a CDN, reuse connections) and a server-work problem (cache, do less) have completely different solutions.** Server-Timing turns "the server is slow" into "*this specific phase* is slow."

</details>

---

## Module 2.3 — Compression: Gzip, Brotli, and Sending Fewer Bytes

**Difficulty:** ●●○○○ · **Format:** Broken project · **Stack:** Node server + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Enabling Gzip & Brotli Compression"** lesson. Todd enables compression locally and shows the byte savings on text assets, plus why it doesn't help already-compressed formats.

**Primary sources:**

- **MDN — HTTP compression** — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Compression — *Focus on: `Content-Encoding`, `Accept-Encoding`, gzip vs Brotli, and which content types benefit.*
- **web.dev — Reduce network payloads with compression** — search the term — *Focus on: text assets (HTML/CSS/JS/SVG/JSON) compress well; images/video do not (already compressed).*
- **Brotli vs gzip** — search — *Focus on: Brotli's better ratios at higher compression levels, and the level/CPU tradeoff.*

**Search prompts:**

- `"gzip vs brotli compression ratio level"`
- `"content-encoding accept-encoding compression http"`
- `"why compress text not images already compressed"`
- Ask an AI: *"Explain HTTP compression: how `Accept-Encoding`/`Content-Encoding` negotiate gzip or Brotli, why text assets (HTML/CSS/JS/JSON/SVG) shrink dramatically but images/video barely change (already compressed), and the tradeoff between Brotli's higher compression levels and the CPU cost to compress."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 2, Module 2.3, Exercise 1 — compression. Completed:
Tracks 0–1, Modules 2.1–2.2. Local Node server.

Build me a Node server serving a page with chunky TEXT assets (a largish CSS and JS bundle, some
JSON) and a couple of images — but with compression DISABLED. Do NOT tell me compression is off or
which assets would benefit.

Have me measure baseline transfer sizes and load time in the Network panel (note transferred vs
resource size). Make me notice the text assets are sent uncompressed, enable gzip then Brotli on the
server, and re-measure the transfer sizes and timing. Also make me try compressing the images and
observe it barely helps — and explain why. Keep the loop. Exam me on: the negotiation handshake,
gzip vs Brotli, why text compresses and images don't, and the level/CPU tradeoff.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 2, Module 2.3, Exercise 2 (transfer) — compression,
applied judgment. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT server serving a mix of asset types (HTML, JS, CSS, SVG, JSON, PNG, JPEG, and
a font). Have me, solo: measure each asset's transfer size, decide WHICH assets to compress and with
WHAT (gzip vs Brotli, and which level), apply it, and prove the byte savings per asset. The catch:
some assets shouldn't be compressed (already-compressed images) and over-compressing tiny files can
cost more than it saves. Don't tell me which is which — make me reason it out. Exam me on the
decision logic. Verdict: can I make correct, measured compression decisions per asset type
(proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- Exercise 1: compression is off, so text assets transfer at full size. Enabling gzip shrinks them a lot; Brotli (especially at higher levels) shrinks them more. The Network panel shows "transferred" dropping far below "resource size." Compressing the images (already-compressed JPEG/PNG) yields almost nothing — sometimes slightly larger — which is the lesson.
- Exercise 2: the correct decisions are: compress the text formats (HTML/CSS/JS/SVG/JSON) with Brotli where supported (gzip fallback); do NOT bother compressing already-compressed images (JPEG/PNG/WebP/AVIF); be cautious compressing fonts (WOFF2 is already compressed); and recognize that for *tiny* files the overhead can exceed the benefit.
- The negotiation: the browser sends `Accept-Encoding: gzip, br`; the server picks one and responds with `Content-Encoding`. No negotiation, no compression.
- The tradeoff: Brotli at max level compresses best but costs more CPU to compress — fine for static assets compressed once at build time, a consideration for dynamically-compressed responses.

The principle: **compression shrinks text-based assets dramatically by encoding redundancy, but does nothing for already-compressed media — so the win is enabling Brotli/gzip on your HTML/CSS/JS/JSON/SVG, ideally precompressed at build time, while leaving images and fonts alone.** It's one of the highest-ratio, lowest-effort loading wins, and the skill is knowing exactly which assets it helps.

</details>

---

## Module 2.4 — HTTP/1.1 vs HTTP/2 vs HTTP/3: Why the Protocol Matters

**Difficulty:** ●●●○○ · **Format:** Investigation · **Stack:** Public sites + Chrome (+ optional local h2 server)

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Efficient Protocols"** and **"Using HTTP/2 & HTTP/3"** lessons. Todd shows how multiplexing over one connection (h2) and QUIC/UDP (h3) change the loading picture versus h1's connection limit.

**Primary sources:**

- **web.dev — Introduction to HTTP/2** — search the term — *Focus on: multiplexing many requests over one connection, eliminating h1's head-of-line blocking and 6-connection limit.*
- **web.dev — HTTP/3 / QUIC** — search the term — *Focus on: running over UDP/QUIC to remove TCP head-of-line blocking and speed up connection setup.*
- **MDN — Connection management in HTTP/1.x** — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Connection_management_in_HTTP_1.x — *Focus on: the per-origin connection limit and why h1 forced domain sharding hacks.*

**Search prompts:**

- `"http/1.1 vs http/2 multiplexing connection limit"`
- `"http/3 quic head of line blocking"`
- `"http2 domain sharding antipattern"`
- Ask an AI: *"Compare HTTP/1.1, HTTP/2, and HTTP/3 for performance. Why did h1's 6-connections-per-origin limit and head-of-line blocking force hacks like domain sharding and sprite sheets? How does h2 multiplexing fix that, and what does h3/QUIC over UDP add? Why are old h1-era optimizations sometimes harmful under h2/h3?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 2, Module 2.4, Exercise 1 — HTTP protocols.
Completed: Tracks 0–1, Modules 2.1–2.3. Mostly investigation.

Guide me through observing the protocol's effect: have me enable the "Protocol" column in the
Network panel and inspect 3–4 real sites, noting which use h2/h3 and how their many requests share
connections (look at the Connection ID / the waterfall's parallelism). Then give me a small local
scenario (a page with many small assets) and have me reason about how it would load differently
under h1 (6-connection limit, requests queuing) vs h2 (multiplexed). Do NOT explain the waterfalls
for me — make me read the parallelism.

Exam me on: h1's connection limit and head-of-line blocking, what h2 multiplexing changes, what
h3/QUIC adds, and why h1-era tricks (domain sharding, concatenating everything into one giant file)
can HURT under h2/h3. Demand layman + technical.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 2, Module 2.4, Exercise 2 (transfer) — protocol-aware
decisions. Apply the Exercise 2 rules: solo first; verdict.

Scenario: I inherit a site built for HTTP/1.1 — it shards assets across many subdomains and
concatenates all JS into one massive bundle "to reduce requests." The site now runs on h2. Have me,
solo, explain what's now SUBOPTIMAL about those h1-era choices under h2/h3, what I'd change (and
what I'd measure to prove it), and the cases where some bundling still helps. You play the skeptical
senior who built the original h1 setup. Exam me on the reasoning. Verdict: can I make protocol-aware
architecture decisions and defend them (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

No bug — protocol literacy that changes how you architect loading. What you should now hold:

- **HTTP/1.1** opens a limited number of connections per origin (≈6 in browsers) and suffers head-of-line blocking — a slow response holds up the connection. This forced era-specific hacks: **domain sharding** (spread assets across subdomains to get more parallel connections), giant concatenated bundles and CSS sprites (fewer requests to dodge the limit).
- **HTTP/2** **multiplexes** many requests/responses over a *single* connection — no 6-request ceiling, no per-request connection setup. This makes the h1 hacks counterproductive: domain sharding now adds redundant connection/TLS setups, and one massive bundle hurts caching and delays parsing when many smaller, separately-cacheable files would multiplex fine.
- **HTTP/3 / QUIC** runs over UDP, removing TCP's head-of-line blocking (a lost packet no longer stalls all streams) and speeding up connection establishment — especially helpful on lossy mobile networks.
- The architectural lesson: **optimizations are protocol-dependent.** "Reduce requests at all costs" was an h1 truth; under h2/h3 the calculus shifts toward more, smaller, independently-cacheable resources — though *some* bundling still helps (reducing per-file overhead, enabling better compression, avoiding thousands of tiny requests).

The principle: **the transport protocol determines what "fast loading" even means — h1's connection limits forced bundling and sharding, while h2/h3 multiplexing makes those same tricks harmful — so an expert checks the protocol before applying loading advice.** Recognizing an h1-era setup running on h2 and knowing what to undo is a senior-level diagnosis.

</details>

---

## Module 2.5 — HTTP Caching: Cache-Control, ETags, and the 304

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** Node server + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Caching"** and **"Enabling Caching Headers"** lessons. Todd shows server vs browser caching, `Cache-Control`/`Expires`, and the 304 (ETag/Last-Modified) revalidation flow.

**Primary sources:**

- **MDN — HTTP caching** — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Caching — *Read fully. Focus on: `Cache-Control` (max-age, no-cache, no-store, immutable, public/private), `ETag`/`If-None-Match`, and the 304 Not Modified flow.*
- **web.dev — Love your cache** — https://web.dev/articles/love-your-cache — *Focus on: the fingerprinted-asset + long-immutable-cache strategy.*
- **MDN — `ETag`** — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/ETag — *Focus on: conditional requests and revalidation without re-downloading.*

**Search prompts:**

- `"cache-control max-age immutable no-cache no-store difference"`
- `"etag if-none-match 304 not modified revalidation"`
- `"fingerprinted assets long cache strategy"`
- Ask an AI: *"Explain HTTP caching precisely: the difference between `no-store`, `no-cache`, and `max-age`; what `immutable` does; and the ETag/`If-None-Match` → 304 Not Modified revalidation flow (which avoids re-downloading unchanged content). What's the standard strategy: short/no cache for HTML, long `immutable` cache for fingerprinted (hashed-filename) assets?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 2, Module 2.5, Exercise 1 — HTTP caching. Completed:
Tracks 0–1, Modules 2.1–2.4. Local Node server.

Build me a Node server + multi-asset page with BROKEN caching: static assets (JS/CSS/images) sent
with no useful cache headers (or `no-store`), so a returning visitor re-downloads everything; and an
HTML file cached too aggressively, so users get a stale page after a deploy. Do NOT tell me which
asset has which caching mistake.

Have me measure the RETURN-visit experience (reload with cache enabled; watch which requests are
re-fetched vs served from cache vs 304). Make me diagnose the bad headers, then fix them: long
`immutable` cache for fingerprinted static assets, ETag revalidation where appropriate, and
short/no-cache for HTML. Re-measure the return visit. Exam me on: max-age vs no-cache vs no-store vs
immutable, the 304 flow, and why HTML and hashed assets get opposite policies.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 2, Module 2.5, Exercise 2 (transfer) — caching strategy.
Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT app with a deploy problem: fingerprinted assets (e.g. app.a1b2c3.js) AND a
classic "users see the old app after we ship" bug, plus an API response that's either cached when it
shouldn't be or not revalidated when it could be. Have me, solo, design the full caching policy for
every resource type (HTML, hashed JS/CSS, images, fonts, API JSON), implement it, and PROVE return-
visit behavior with the Network panel (from-cache vs 304 vs re-download). Don't tell me the policy —
make me derive it and defend each choice. Exam me on the deploy/staleness tradeoff. Verdict: can I
design and verify a complete caching strategy (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- Exercise 1: static assets with `no-store`/no cache headers, so a returning visitor re-downloads JS/CSS/images that never changed — wasted bytes and time. Fix: `Cache-Control: public, max-age=31536000, immutable` for **fingerprinted** assets (filename includes a content hash, so a change = a new URL, making infinite caching safe). The HTML cached too long, so deploys don't reach users → fix: short max-age or `no-cache` (revalidate) for HTML, since its URL is stable.
- The 304 flow: with an `ETag`, the browser sends `If-None-Match`; if unchanged, the server replies **304 Not Modified** with no body — saving the download while confirming freshness. You should see 304s in the Network panel for revalidated assets.
- The header meanings: `no-store` = never cache; `no-cache` = cache but revalidate every time before use; `max-age=N` = fresh for N seconds; `immutable` = don't even revalidate during max-age (the content at this URL will never change).
- The opposite-policies insight: **HTML gets short/revalidated caching** (stable URL, must reflect deploys) while **fingerprinted assets get long immutable caching** (URL changes on every content change, so it's always safe to cache forever).

The principle: **HTTP caching lets returning visitors skip downloads entirely (from cache) or skip re-downloads of unchanged files (304) — and the standard strategy is long-immutable caching for content-hashed assets plus short/revalidated caching for HTML, which gives instant repeat visits without ever serving a stale app.** Getting this right is one of the largest return-visit wins available, and the fingerprint+immutable pattern is something every senior is expected to know cold.

</details>

---

## Module 2.6 — CDNs & Geographic Latency

**Difficulty:** ●●●○○ · **Format:** Investigation · **Stack:** Public sites + WebPageTest + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Host Capacity & Proximity"** lesson (in Improving TTFB). Todd explains how a CDN reduces the latency between the user and the bytes by serving from an edge near them.

**Primary sources:**

- **web.dev — Content delivery networks (CDNs)** — https://web.dev/articles/content-delivery-networks — *Focus on: edge caching, proximity, offloading the origin, and cache-hit ratio.*
- **Cloudflare/Fastly learning centers — "What is a CDN"** — search — *Focus on: points of presence, edge vs origin, and what's cacheable at the edge.*
- **MDN — latency vs bandwidth** — search "latency vs bandwidth web performance" — *Focus on: why distance (latency/RTT) often dominates over raw bandwidth for small requests.*

**Search prompts:**

- `"how cdn improves performance edge caching proximity"`
- `"cdn cache hit ratio origin offload"`
- `"latency vs bandwidth round trip distance"`
- Ask an AI: *"Explain how a CDN improves performance: serving cached assets from an edge location physically near the user reduces round-trip latency, and offloads the origin. What is cache-hit ratio and why does it matter? Why does latency (governed by distance/round-trips) often matter more than bandwidth for web loading, especially for many small requests?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 2, Module 2.6, Exercise 1 — CDNs and geographic
latency. Completed: Tracks 0–1, Modules 2.1–2.5. Investigation.

Guide me through demonstrating proximity's effect using WebPageTest (from Module 1.6): have me test
the SAME site from a location NEAR its origin and from a location FAR from it, and compare TTFB and
load. Then test a site that's clearly on a global CDN from multiple far-apart locations and observe
how consistent its TTFB stays. Do NOT explain the numbers — make me read them and infer what the
CDN is doing. Also have me inspect response headers for CDN fingerprints (cache HIT/MISS, edge
server headers).

Exam me on: why distance creates latency (round-trips), how a CDN edge cache cuts it, what cache-hit
ratio means, what's safely cacheable at the edge (static assets) vs not (personalized HTML), and
why latency often beats bandwidth as the bottleneck. Demand layman + technical.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 2, Module 2.6, Exercise 2 (transfer) — CDN strategy for
a real audience. Apply the Exercise 2 rules: solo first; verdict.

Scenario tied to my situation: an app whose origin server is in one region (say US-east) but whose
users are largely in India on mobile. Have me, solo, lay out: what's currently slow and why
(proximity/latency), exactly what I'd put on a CDN and what I can't (static vs personalized),
roughly how much TTFB improvement I'd expect for the Indian users and how I'd MEASURE it (WebPageTest
from India before/after), and what cache-hit ratio I'd aim for. You play a skeptical lead who thinks
"the server is fast, it's fine." Exam me on latency vs bandwidth and edge-cacheability. Verdict: can
I make and defend a CDN decision for a real audience (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — What This Is Teaching

<details>
<summary>Click only after you finish or give up</summary>

No bug — this is proximity reasoning, hugely relevant to you specifically (origin often far from Indian users). What you should now hold:

- **Distance is latency.** Every request is at least one round-trip; the farther the server, the longer each round-trip (light speed + routing), and many small requests multiply that. A user in India hitting a US origin pays that tax on *every* round-trip — handshake, TTFB, and each resource.
- **A CDN serves from the edge** — a point of presence physically near the user — so cached assets travel a short distance instead of across the world. This slashes TTFB and resource-fetch latency and offloads the origin.
- **Cache-hit ratio** = the fraction of requests the edge serves without going back to origin. High hit ratio = most users get edge speed; low hit ratio = the CDN keeps falling back to the slow origin, so you're paying for a CDN without the benefit.
- **What's edge-cacheable:** static, non-personalized assets (JS/CSS/images/fonts) cache beautifully at the edge. Personalized or per-request HTML/API responses are harder (need edge logic, or stay dynamic) — though edge compute and stale-while-revalidate blur this.
- **Latency vs bandwidth:** for typical web loading (many smallish requests), the round-trip *latency* usually dominates more than raw *bandwidth* — which is why a CDN (cutting distance) often helps more than a fatter pipe.

The principle: **a CDN attacks the proximity component of every request by serving bytes from an edge near the user — cutting latency that distance imposes — and for a globally-distributed or far-from-origin audience (like Indian users on a US-hosted app) it's often the single biggest loading win.** Reasoning about latency-vs-bandwidth and edge-cacheability for a specific audience is exactly the senior judgment that justifies a big salary.

</details>

---

## Module 2.7 — Resource Hints: preconnect, preload, prefetch, dns-prefetch

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** Vanilla HTML + local server + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Preloading Resources"** lesson (and *Removing Sequence Chains*). Todd uses `preconnect`/`preload` on fonts to start critical fetches earlier and break dependency chains.

**Primary sources:**

- **web.dev — Resource hints overview / preload critical assets** — https://web.dev/articles/preload-critical-assets — *Focus on: `preload` for late-discovered critical resources.*
- **web.dev — preconnect & dns-prefetch** — https://web.dev/articles/preconnect-and-dns-prefetch — *Focus on: warming up a connection to a third-party origin early.*
- **MDN — `<link rel>` (preload/prefetch/preconnect/dns-prefetch/modulepreload)** — https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/rel — *Focus on: what each hint does and when it helps vs hurts.*
- **web.dev — fetchpriority / Priority Hints** — https://web.dev/articles/fetch-priority — *Focus on: nudging the browser's resource priority.*

**Search prompts:**

- `"preload vs prefetch vs preconnect vs dns-prefetch"`
- `"resource hints overuse hurt performance"`
- `"fetchpriority high lcp image"`
- Ask an AI: *"Explain each resource hint precisely and when to use it: `preconnect` and `dns-prefetch` (warm a connection to an origin early), `preload` (fetch a late-discovered critical resource now), `prefetch` (fetch something for a FUTURE navigation), `modulepreload` (for ES modules), and `fetchpriority`. What goes wrong if I overuse `preload` or preconnect to too many origins?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 2, Module 2.7, Exercise 1 — resource hints. Completed:
Tracks 0–1, Modules 2.1–2.6. Local server.

Build me a page with critical resources discovered LATE: a critical font referenced deep in a CSS
file (so it starts very late), a third-party origin (e.g. a CDN/font host) connected to late, and
the LCP image loaded at default priority while less important things compete. Do NOT tell me which
resource needs which hint.

Have me baseline (FCP/LCP + the waterfall's late-discovery staircase), then fix with the RIGHT hints
— `preconnect` the third-party origin, `preload` the critical late-discovered resource, `fetchpriority`
the LCP image — re-measuring each time. Then have me ADD too many preloads/preconnects and watch it
get WORSE, and explain why. Keep the loop. Exam me on each hint, when it helps, and the overuse
failure mode.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 2, Module 2.7, Exercise 2 (transfer) — hints, correct
selection. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT page with a different late-discovery pattern AND a next-page navigation that
could be prefetched. Have me, solo, choose the RIGHT hint for each situation — preconnect vs preload
vs prefetch vs dns-prefetch vs fetchpriority — apply only what's justified, and prove each one helped
(or, if it didn't, remove it). The trap: it's easy to sprinkle hints everywhere; the skill is using
the minimum that helps and knowing prefetch is for the NEXT navigation, not this one. Don't tell me
which hint goes where. Exam me on the distinctions and the overuse cost. Verdict: can I select
resource hints surgically (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- A critical resource (font/image) discovered late because it's referenced deep in a CSS/JS chain — the browser can't fetch what it hasn't parsed yet. Fix: `preload` it in the HTML head so it starts immediately.
- A third-party origin (font host, CDN, API) where DNS+TCP+TLS happens late, delaying the first request to it. Fix: `preconnect` (full handshake warm-up) or `dns-prefetch` (just DNS) early.
- The LCP image at default/low priority while less important resources compete. Fix: `fetchpriority="high"` (and possibly `preload`).
- The overuse failure: preloading too many things makes them all compete and can DELAY the genuinely critical one (you've told the browser "everything is top priority," which means nothing is); preconnecting to many origins wastes connections and CPU. Hints are a scalpel, not a sprinkle.
- `prefetch` is for a *future* navigation (low priority, for the next page) — using it for the current page is wrong; using `preload` for a resource the current page doesn't need is wasteful.

The principle: **resource hints let you correct the browser's discovery and priority decisions — preconnect to warm a connection, preload a late-discovered critical resource, prefetch for the next navigation, fetchpriority to reorder — but only the *minimum justified set* helps; overusing them creates contention that slows the very resources you meant to speed up.** The expert skill is surgical selection: the right hint, for the right resource, and nothing more.

</details>

---

## Module 2.8 — The Full Waterfall Diagnosis (Network Capstone)

**Difficulty:** ●●●●● · **Format:** Investigation + broken project · **Stack:** Node server + multi-asset app + Chrome/WPT

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**

- **Web Performance Fundamentals, v2** (Todd Gardner) → re-watch the **"Waterfall Charts"** lesson and skim the whole **"Improving Time to First Byte"** and **"Improving First Contentful Paint"** sections. This capstone combines everything in Track 2.

**Primary sources:**

- **Chrome DevTools — Network reference (revisit)** — https://developer.chrome.com/docs/devtools/network/reference — *Focus on: priority column, initiator chains, and the timing breakdown together.*
- **web.dev — Optimize resource loading (overview)** — search — *Focus on: combining hints, compression, caching, and protocol into one coherent loading strategy.*

**Search prompts:**

- `"network waterfall diagnosis methodology bottleneck"`
- `"initiator chain devtools request dependency"`
- Ask an AI: *"Give me a systematic methodology for diagnosing a slow-loading page from its network waterfall: how to find the critical path, spot render-blockers, identify late-discovered resources via the initiator chain, read the timing breakdown to classify each slow request (server? size? connection?), and decide which fix (compression, caching, hints, protocol, CDN) addresses each."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 2, Module 2.8, Exercise 1 — full waterfall diagnosis,
network capstone. Completed: Tracks 0–1, Modules 2.1–2.7. Make this genuinely hard.

Build me a realistic multi-asset app (local Node server) that is slow to load for SEVERAL reasons
at once — drawn from this whole track: a bad TTFB, missing compression, a render-blocking chain, a
late-discovered critical resource, broken caching, and a missing hint or two. Do NOT tell me how many
problems there are or what they are.

This is a full diagnosis. Have me: baseline the load (FCP/LCP/waterfall/WPT filmstrip), then work
methodically — for EACH slow or blocking request, classify it (which phase? render-blocking? late-
discovered? uncacheable?) and prescribe the specific fix, applying them one at a time and re-measuring
the cumulative improvement. Hold me to the loop ruthlessly. Run a long exam tying each fix back to its
mechanism. This is the rehearsal for diagnosing a real site cold.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 2, Module 2.8, Exercise 2 (transfer) — diagnose a
DIFFERENT messy load. Apply the Exercise 2 rules: solo, minimal hints; full verdict.

Build me a DIFFERENT multi-problem loading scenario with a different combination of issues than
Exercise 1. Have me diagnose and fix the ENTIRE thing solo — baseline, methodical per-request
classification, one-at-a-time fixes, cumulative re-measurement, and a final before/after report I
could show a client. Give essentially no hints unless I'm truly stuck after a real attempt. Exam me
hard. Then give the verdict I most need: am I ready to walk up to an unfamiliar real site's waterfall
and diagnose its loading bottlenecks cold (proceed to Track 3), or do I need to redo parts of Track 2?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

The app deliberately combines the whole track, e.g.:

- **Bad TTFB** from uncached per-request server work (Module 2.2).
- **No compression** on text assets (Module 2.3).
- **A render-blocking critical-path chain** — sync script and/or CSS→font dependency (Module 2.1).
- **A late-discovered critical resource** that needs a `preload` (Module 2.7).
- **Broken caching** so return visits re-download everything (Module 2.5).
- Possibly a **missing preconnect** to a third-party origin and a **mis-prioritized LCP image** (Modules 2.6/2.7).

The methodology you should now execute without being told:

1. **Baseline** with numbers and a waterfall/filmstrip.
2. **Walk the waterfall request by request.** For each slow or blocking one, classify via the timing breakdown and initiator: is it a *server* problem (TTFB), a *size* problem (compression/optimization), a *blocking* problem (critical path), a *discovery* problem (late, needs preload), a *connection* problem (needs preconnect/CDN), or a *caching* problem (re-downloaded)?
3. **Prescribe the matching fix** for each — and apply ONE at a time, re-measuring, so you know what each was worth.
4. **Report** the cumulative before/after.

The principle: **a real slow page is never slow for one reason — diagnosis is a systematic walk of the waterfall where you classify each request by its failure mode and apply the matching fix, measuring each step.** This module IS your stated end goal for loading performance: open a waterfall, and name what's wrong and how to fix it, request by request. If you can do Exercise 2 cold, you can do it on a real client site.

</details>

---

*End of Track 2 — The Network & Loading Pipeline. 8 modules. You can now diagnose and fix why a page is slow to appear — TTFB, compression, protocols, caching, CDNs, the critical path, and resource hints — by reading the waterfall.*

*Next: Track 3 — JavaScript & The Main Thread, where "the page appeared but it's frozen" gets solved, and where the deep-JS-engine courses finally pay off.*
