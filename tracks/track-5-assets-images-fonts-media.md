# TRACK 5 — ASSETS: IMAGES, FONTS & MEDIA

*The unglamorous track that produces the biggest real-world wins. On most real sites, images and fonts are the heaviest bytes and the most common causes of slow LCP and layout shift — yet they're the easiest to fix once you know how. This track makes you the person who can look at a page and immediately spot the 2MB hero image, the render-blocking font, and the unsized image that's shifting the layout.*

*Prerequisites: Tracks 0–2 (read a waterfall, measure CWV, understand the critical path). Mostly HTML/CSS with a little server work.*

*Spine course: **Web Performance Fundamentals, v2** (Todd Gardner) — its **"Improving Largest Contentful Paint"** section is heavily about images, and it covers font and asset optimization directly. This is the primary FM course for the whole track.*

---

## Module 5.1 — Image Formats: JPEG, PNG, WebP, AVIF

**Difficulty:** ●●○○○ · **Format:** Investigation + broken project · **Stack:** HTML + local server + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Improving Largest Contentful Paint"** section, especially the image-optimization and modern-format lessons. Todd shows the dramatic byte savings from choosing the right format.

**Primary sources:**
- **web.dev — Choose the right image format** — https://web.dev/articles/choose-the-right-image-format — *Focus on: when to use JPEG vs PNG vs WebP vs AVIF, and lossy vs lossless.*
- **web.dev — Use WebP / AVIF images** — https://web.dev/articles/serve-images-webp — *Focus on: the byte savings of modern formats and fallbacks.*
- **MDN — Image file type guide** — https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Image_types — *Focus on: format characteristics and browser support.*

**Search prompts:**
- `"jpeg vs png vs webp vs avif when to use"`
- `"avif webp file size savings vs jpeg"`
- `"lossy vs lossless image compression web"`
- Ask an AI: *"Compare JPEG, PNG, WebP, and AVIF for the web: which suits photos vs graphics-with-transparency vs simple graphics, the lossy/lossless distinction, and the typical byte savings of WebP and AVIF over JPEG/PNG. How do I serve a modern format with a fallback for older browsers using `<picture>`?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 5, Module 5.1, Exercise 1 — image formats. Completed:
Tracks 0–4.

Build me a page using the WRONG image formats for the job: a large photo saved as PNG (huge), a
simple flat-color graphic as a heavy JPEG (artifacts + bigger than needed), and everything in legacy
formats when WebP/AVIF would be far smaller. Serve it locally. Do NOT tell me which image is wrong.

Have me measure baseline transfer sizes (Network panel) and LCP. Make me identify the oversized/
mis-formatted images, convert each to the right format (you tell me tools/commands to convert), serve
modern formats with `<picture>` fallbacks, and re-measure the byte savings and LCP. Exam me on: which
format suits which content, lossy vs lossless, the savings of WebP/AVIF, and the `<picture>` fallback
pattern.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 5, Module 5.1, Exercise 2 (transfer) — format decisions
across a gallery. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT page with a MIX of image types (photos, logos with transparency, icons,
screenshots, a gradient). Have me, solo, decide the optimal format for EACH (and justify lossy vs
lossless and a quality level), convert them, serve modern-with-fallback, and prove the total byte
savings + LCP improvement. The trap: there's no single "best format" — it depends per image. Don't
tell me which goes where. Exam me on the decision logic per content type. Verdict: can I make correct
per-image format decisions (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The mistakes: a **photo as PNG** (PNG is lossless — great for graphics/transparency, terrible for photos; the file is many times larger than a JPEG/WebP would be); a **flat graphic as JPEG** (JPEG's lossy compression adds artifacts to sharp edges and may be larger than a PNG/WebP for simple graphics); everything in **legacy formats** when **WebP** (~25–35% smaller than JPEG) or **AVIF** (often ~50% smaller) would cut bytes dramatically.
- The fixes: photos → JPEG or, better, WebP/AVIF (lossy); graphics with transparency or sharp edges → PNG or WebP/AVIF (lossless mode); serve modern formats with a `<picture>` element offering AVIF → WebP → JPEG fallbacks so older browsers still work. Re-measure: large transfer-size drops and an LCP improvement if the LCP element was an image.
- The per-image truth (Exercise 2): there is no universal best format — it depends on content (photo vs graphic vs transparency) and the lossy/lossless tradeoff, plus an appropriate quality level (you rarely need maximum quality).

The principle: **choosing the right image format per content type — and serving modern formats (WebP/AVIF) with fallbacks — is one of the largest, easiest byte savings available, because images are usually the heaviest assets on a page.** Spotting a photo-as-PNG or a legacy-format hero is an instant, high-impact diagnosis.
</details>

---

## Module 5.2 — Responsive Images: srcset, sizes & the Right Pixels

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** HTML + local server + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Improving Largest Contentful Paint"** responsive-image material — serving appropriately-sized images instead of one giant image for all screens.

**Primary sources:**
- **MDN — Responsive images** — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Responsive_images — *Read fully. Focus on: `srcset` with width descriptors, the `sizes` attribute, and `<picture>` for art direction.*
- **web.dev — Serve responsive images** — https://web.dev/articles/serve-responsive-images — *Focus on: why sending desktop-sized images to phones wastes huge bytes.*
- **web.dev — `sizes` attribute** — search — *Focus on: how `sizes` tells the browser the display size so it picks the right `srcset` candidate.*

**Search prompts:**
- `"srcset sizes responsive images width descriptor"`
- `"picture element art direction vs srcset"`
- `"serving oversized images to mobile waste"`
- Ask an AI: *"Explain responsive images precisely: `srcset` with width descriptors (e.g. `img-400.jpg 400w`), how the `sizes` attribute tells the browser the rendered width so it picks the smallest sufficient candidate, and when to use `<picture>` for art direction (different crops) vs `srcset` (same image, different sizes). Why does serving one big image to all devices waste so much on mobile?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 5, Module 5.2, Exercise 1 — responsive images.
Completed: Tracks 0–4, Module 5.1.

Build me a page that serves ONE giant high-resolution image to every device — so a phone downloads a
2000px+ image to display it at 360px wide, wasting most of the bytes. Serve it locally. Do NOT tell
me about srcset.

Have me measure (emulate a mobile viewport + Slow 4G; note the wasted transferred bytes vs displayed
size). Make me implement `srcset` with width descriptors + a correct `sizes` attribute (and generate
the resized variants — you tell me the tooling), then verify in the Network panel that the mobile
viewport now downloads a SMALL candidate. Re-measure bytes + LCP. Exam me on: how srcset/sizes let
the browser choose, what the `w` descriptors mean, how `sizes` works, and picture-vs-srcset.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 5, Module 5.2, Exercise 2 (transfer) — responsive
images, including art direction. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT layout with images at different display sizes across breakpoints, INCLUDING one
case that needs art direction (a different crop on mobile vs desktop, via `<picture>`). Have me,
solo: set up correct srcset/sizes for the resolution-switching images AND `<picture>` for the
art-directed one, generate variants, and prove each viewport downloads the right candidate. The trap:
a wrong `sizes` value makes the browser pick a too-big (or too-small) image. Don't tell me the
values. Exam me on resolution-switching vs art-direction and on getting `sizes` right. Verdict: can I
implement responsive images correctly (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problem: one large image (`<img src="huge.jpg">`) served to all devices. A phone displaying it at 360px CSS pixels still downloads the full 2000px+ file — wasting the vast majority of those bytes, slowing LCP especially on the slow mobile networks where it hurts most.
- The fix: **`srcset`** with width descriptors (`huge-400.jpg 400w, huge-800.jpg 800w, huge-1600.jpg 1600w`) gives the browser candidates; the **`sizes`** attribute tells it the rendered width at each breakpoint (e.g. `sizes="(max-width: 600px) 100vw, 50vw"`), so it picks the smallest candidate that's still sharp (accounting for device pixel ratio). Generate the resized variants with an image tool. Verify the mobile viewport now pulls a small candidate.
- **`<picture>`** is for **art direction** — serving a genuinely different image/crop at different breakpoints (e.g. a tight crop on mobile, wide on desktop) — and for format fallbacks; **`srcset`/`sizes`** is for **resolution switching** (same image, different sizes).
- The `sizes` trap: get it wrong and the browser picks a too-large image (no savings) or too-small (blurry).

The principle: **responsive images let the browser download the smallest image that still looks sharp for the actual display size and screen density — via `srcset` candidates and a correct `sizes` hint — which can cut image bytes by an order of magnitude on mobile.** Serving one giant image to everyone is one of the most common and wasteful real-world mistakes, and spotting it is an instant win.
</details>

---

## Module 5.3 — Lazy Loading & Prioritizing the LCP Image

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** HTML + local server + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the **"Lazy Loading Resources"** lesson and the LCP-image material in the LCP section.

**Primary sources:**
- **web.dev — Browser-level lazy loading (`loading="lazy"`)** — https://web.dev/articles/browser-level-image-lazy-loading — *Focus on: lazy-loading below-the-fold images, and the critical rule NOT to lazy-load the LCP image.*
- **web.dev — Optimize LCP / preload the LCP image** — https://web.dev/articles/optimize-lcp — *Focus on: `fetchpriority="high"` and preloading the LCP image so it loads first.*
- **web.dev — `fetchpriority`** — https://web.dev/articles/fetch-priority — *Focus on: raising the LCP image's priority, lowering non-critical images'.*

**Search prompts:**
- `"loading lazy image below the fold"`
- `"do not lazy load lcp image fetchpriority high"`
- `"preload lcp image priority"`
- Ask an AI: *"Explain image loading priority: use `loading='lazy'` for below-the-fold images to defer them, but why must I NOT lazy-load the LCP (hero) image — and instead `fetchpriority='high'` and/or preload it? How does lazy-loading the LCP image actually make LCP worse, and how do I get the priority right?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 5, Module 5.3, Exercise 1 — lazy loading & LCP priority.
Completed: Tracks 0–4, Modules 5.1–5.2.

Build me a long page with many images where the priority is BACKWARDS: the hero/LCP image is
lazy-loaded (or low priority) so it loads late, while below-the-fold images load eagerly and compete.
Do NOT tell me the priority is wrong.

Have me measure baseline LCP and watch the Network panel/priorities. Make me discover the LCP image
is being deprioritized, then fix it: NOT lazy on the LCP image, `fetchpriority="high"` (and/or
preload) on it, `loading="lazy"` on the below-the-fold images. Re-measure LCP. Exam me on: why
lazy-loading the LCP image hurts LCP, when lazy loading helps, and how fetchpriority/preload reorder
the queue.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 5, Module 5.3, Exercise 2 (transfer) — priority on a new
page. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT image-heavy page (e.g. a product page or article with a hero, inline images, and
a long gallery). Have me, solo: identify the LCP image, get its loading priority right (eager + high
priority + maybe preload), lazy-load everything below the fold, and prove the LCP improved and the
below-fold images no longer compete early. Don't tell me which image is the LCP — make me find it.
Exam me on the priority reasoning. Verdict: can I get image loading priority right (proceed), or redo
Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The backwards priority: the **LCP (hero) image is lazy-loaded** or left at default/low priority, so the browser delays it — directly inflating LCP, because the LCP metric is literally "when did the largest element render." Meanwhile **below-the-fold images load eagerly**, competing for bandwidth with the thing the user actually needs first.
- The fixes: **never lazy-load the LCP image** (lazy-loading defers it, which is the opposite of what you want for the most important element); instead set **`fetchpriority="high"`** on it and/or **preload** it so it starts immediately; and set **`loading="lazy"`** on genuinely below-the-fold images so they don't compete. Re-measure: LCP drops.
- Why lazy-loading the LCP image is a classic self-inflicted wound: developers apply `loading="lazy"` to *all* images "for performance," not realizing it sabotages the single most important image.
- Lazy loading is great for off-screen images (saves bytes for content the user may never scroll to) — the skill is applying it to the right images only.

The principle: **the LCP image must be loaded as early and high-priority as possible (eager, `fetchpriority='high'`, preloaded), while below-the-fold images should be lazy-loaded — so blanket-lazy-loading every image backfires by deferring the hero.** Getting image priority right is one of the most direct LCP levers, and spotting a lazy-loaded hero is a frequent, easy diagnosis.
</details>

---

## Module 5.4 — Image Dimensions & Cumulative Layout Shift

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** HTML/CSS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the **CLS** lesson (and its Q&A) — unsized images/media are the textbook CLS cause.

**Primary sources:**
- **web.dev — Optimize CLS / setting dimensions** — https://web.dev/articles/optimize-cls — *Focus on: width/height attributes and `aspect-ratio` reserving space so images don't shift content.*
- **web.dev — `aspect-ratio`** — https://web.dev/articles/aspect-ratio — *Focus on: reserving space for responsive images.*
- **MDN — `<img>` width/height** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#attr-width — *Focus on: how dimensions let the browser reserve space before the image loads.*

**Search prompts:**
- `"image width height aspect-ratio prevent layout shift cls"`
- `"reserve space image responsive aspect ratio"`
- Ask an AI: *"Explain how unsized images cause Cumulative Layout Shift: the browser doesn't know the image's size until it loads, so content jumps when it arrives. How do `width`/`height` attributes and `aspect-ratio` reserve the space ahead of time (even for responsive images), eliminating the shift?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 5, Module 5.4, Exercise 1 — image dimensions & CLS.
Completed: Tracks 0–4, Modules 5.1–5.3.

Build me a page with several images (and maybe an embed/ad slot) that have NO dimensions, so as they
load, the content below them JUMPS — a bad CLS. Do NOT tell me the cause.

Have me measure CLS (Web Vitals extension / Lighthouse) and visually observe the shifts (DevTools can
highlight layout shifts). Make me identify the unsized media, then fix it: set `width`/`height`
attributes (or `aspect-ratio`) to reserve space, including for the responsive images. Re-measure CLS
to ~0. Exam me on: why unsized images shift content, how dimensions/aspect-ratio reserve space, and
how this works even when the image is fluid-width.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 5, Module 5.4, Exercise 2 (transfer) — CLS from media &
dynamic content. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT page with CLS from a mix of sources: unsized images, a late-loading banner/ad
inserted above content, and a web-font swap that reflows text (foreshadow next module). Have me,
solo: measure CLS, attribute each shift to its source, fix each (reserve space for media and the ad
slot; handle the font), and prove CLS dropped. Don't tell me the sources. Exam me on reserving space
and on the different CLS causes. Verdict: can I diagnose and fix layout shift from media (proceed),
or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problem: images (and embeds/ad slots) with no declared dimensions. The browser doesn't know how tall they are until they load, so it lays out the page as if they're zero-height, then **shifts everything below them down** when the image arrives — producing visible jumps and a bad CLS.
- The fix: set **`width` and `height` attributes** on `<img>` (the browser computes the aspect ratio and reserves the correct space before the image loads, even when CSS makes the image fluid-width), or use the CSS **`aspect-ratio`** property. For ad/embed slots, reserve a min-height. Re-measure: CLS drops toward 0.
- The subtlety: modern browsers use the width/height *attributes* to derive an aspect ratio and reserve space even when the image is styled `width: 100%; height: auto` — so always include the intrinsic dimensions, even for responsive images.
- Other CLS sources (Exercise 2): content injected above existing content (banners/ads) without reserved space, and font swaps reflowing text (next module).

The principle: **layout shift from media comes from the browser not knowing an element's size until it loads, so reserving space ahead of time — via width/height attributes or `aspect-ratio` — eliminates the jump.** Unsized images are the most common CLS cause on the web, and adding dimensions is a trivial, high-impact fix that spots instantly in an audit.
</details>

---

## Module 5.5 — Web Font Loading: FOIT, FOUT & font-display

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** HTML/CSS + local server + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- **Web Performance Fundamentals, v2** (Todd Gardner) → the font-related material in the FCP/LCP sections (web fonts as render-blocking, and how to load them without blocking text).

**Primary sources:**
- **web.dev — Best practices for fonts** — https://web.dev/articles/font-best-practices — *Focus on: `font-display`, preloading, and self-hosting vs third-party.*
- **MDN — `font-display`** — https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display — *Focus on: `swap`/`optional`/`fallback`/`block` and the FOIT vs FOUT tradeoff.*
- **web.dev — Prevent layout shifts from web fonts** — https://web.dev/articles/css-size-adjust — *Focus on: `size-adjust` / metric overrides to match the fallback font and kill font-swap CLS.*

**Search prompts:**
- `"font-display swap optional fallback foit fout"`
- `"preload web font reduce render delay"`
- `"font swap layout shift size-adjust metric override"`
- Ask an AI: *"Explain web font loading problems: FOIT (invisible text while the font loads) vs FOUT (fallback text that swaps to the web font). How do the `font-display` values (`block`/`swap`/`fallback`/`optional`) choose between them? Why does the swap cause layout shift, and how do `size-adjust`/`ascent-override` (metric overrides) on the fallback font reduce or eliminate it? When should I preload a font?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 5, Module 5.5, Exercise 1 — web font loading. Completed:
Tracks 0–4, Modules 5.1–5.4.

Build me a page with web-font problems: a custom font that causes invisible text on load (FOIT,
default font-display) and/or a big layout shift when it swaps in, plus the font discovered late
(referenced deep in CSS) so it loads slowly. Serve it locally. Do NOT name the issues.

Have me observe the invisible-text / text-swap behavior and measure its effect on FCP/LCP/CLS. Make
me fix it: choose an appropriate `font-display` (and explain the tradeoff), `preload` the critical
font so it starts early, and reduce the swap's layout shift with a metric-matched fallback
(`size-adjust`/ascent overrides). Re-measure. Exam me on: FOIT vs FOUT, the font-display values, why
preloading helps, and how metric overrides kill the swap shift.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 5, Module 5.5, Exercise 2 (transfer) — fonts, new setup
+ subsetting. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT page using multiple weights of a third-party-hosted font (e.g. several Google
Font weights) loaded sub-optimally. Have me, solo: self-host and subset the font (reduce to needed
characters/weights — you tell me the tooling), set the right font-display, preload the critical one,
preconnect if any third-party origin remains, and minimize swap shift. Prove the byte savings + CWV
improvement. Don't tell me the steps. Exam me on subsetting, self-hosting tradeoffs, and the font-
display choice. Verdict: can I do production-grade font loading (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problems: the default `font-display` behavior causes **FOIT** (text invisible for up to a few seconds while the font loads — bad for perceived performance and LCP if text is the LCP); when the font swaps in, the different metrics cause a **layout shift** (FOUT's downside — bad CLS); and the font is **discovered late** (referenced deep in CSS), so it starts loading slowly.
- The fixes: pick a **`font-display`** value for the right tradeoff — `swap` (show fallback immediately, swap when ready — no invisible text, but a swap shift) is the common default; `optional` (use the font only if it loads almost instantly — best for avoiding shift, may not use the font on first visit); `fallback` (a middle ground). **`preload`** the critical font so it starts immediately instead of after CSS parses. Reduce the swap's layout shift with a **metric-matched fallback** using `size-adjust`/`ascent-override`/`descent-override` so the fallback occupies the same space as the web font. Re-measure FCP/LCP/CLS.
- Exercise 2 additions: **subset** the font (strip unused glyphs/weights → big byte savings), **self-host** (avoids a third-party origin round-trip; if you must use a third party, `preconnect`), and load only the weights you actually use.

The principle: **web fonts block or shift text, so good font loading is about choosing the FOIT/FOUT tradeoff (`font-display`), starting the critical font early (preload), shrinking it (subset/self-host), and matching the fallback's metrics to kill the swap shift.** Fonts are a top cause of both slow text rendering and CLS, and handling them well is a clear senior signal.
</details>

---

## Module 5.6 — Image Placeholders & Perceived Load (LQIP, Blur-up)

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** HTML/CSS/JS + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- No dedicated FM lesson; this builds on the LCP/perceived-performance ideas in **Web Performance Fundamentals, v2**. Lean on primary sources.

**Primary sources:**
- **web.dev — Placeholders / blur-up technique** — https://web.dev/articles/image-component (and search "LQIP blur-up") — *Focus on: showing a tiny blurred preview while the full image loads.*
- **BlurHash** — https://blurha.sh — *Focus on: encoding a compact blurred placeholder string.*
- **MDN — `<img>` decoding / `decoding="async"`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#attr-decoding — *Focus on: not blocking on image decode.*

**Search prompts:**
- `"lqip blur-up low quality image placeholder"`
- `"blurhash placeholder image loading"`
- `"image decoding async perceived performance"`
- Ask an AI: *"Explain image placeholder techniques for perceived performance: LQIP (low-quality image placeholder) and blur-up (a tiny blurred version shown immediately, then replaced by the full image), and BlurHash (a compact encoded placeholder). Why do these improve perceived speed and reduce the jarring 'pop-in' even though they don't make the real image load faster?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 5, Module 5.6, Exercise 1 — image placeholders, build
challenge. Completed: Tracks 0–4, Modules 5.1–5.5.

Give me a spec (no solution code) to build a gallery where images currently "pop in" jarringly on
slow connections, and have me implement a blur-up / LQIP placeholder: show a tiny blurred preview
immediately, then cross-fade to the full image when it loads, with space reserved (no CLS). I write
it. Test on Slow 4G.

Review me like a senior. Make me MEASURE that this doesn't change actual LCP/load time but improves
PERCEIVED experience (and reason about whether the placeholder could hurt LCP if done wrong). If my
placeholder causes a layout shift, ask me what space I reserved. Exam me on: why placeholders help
perceived performance without speeding real load, and the CLS/LCP pitfalls.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 5, Module 5.6, Exercise 2 (transfer) — placeholders, new
technique. Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT image-heavy view and have me, solo, implement a different placeholder approach
(e.g. BlurHash or a dominant-color placeholder) with proper space reservation and a clean transition,
on a throttled connection. Prove no CLS, no LCP regression, and a smoother perceived load. Don't tell
me the implementation. Exam me on the perceived-vs-actual distinction and the pitfalls. Verdict: can
I implement placeholders that improve perceived load without hurting metrics (proceed), or redo
Exercise 1?
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- A placeholder strategy: show something *immediately* in the image's reserved space — a tiny blurred low-res version (LQIP/blur-up), a BlurHash-decoded gradient, or the dominant color — then cross-fade to the full image on load. Space is reserved (width/height/aspect-ratio) so there's **no CLS**.
- The key measurement insight: placeholders do **not** make the real image load faster — actual LCP/load time is unchanged. They improve **perceived** performance: the user sees structure and a preview instead of blank boxes, so the wait feels shorter and the "pop-in" is smooth.
- The pitfalls: if the placeholder itself is the LCP element or delays the real image, you can *hurt* LCP; if space isn't reserved, the swap causes CLS; an over-heavy placeholder (large inline data) adds bytes. Done right, the placeholder is tiny (a few hundred bytes inline) and the transition is GPU-friendly (opacity).
- This connects to Track 7 (perceived performance) — it's a UX-of-speed technique grounded in real asset handling.

The principle: **placeholders (blur-up/LQIP/BlurHash/dominant-color) improve how fast a page *feels* by filling reserved space with an instant preview and smoothing the image's arrival — without changing actual load time — so they're a perceived-performance tool that must be implemented without introducing CLS or LCP regressions.** Knowing the difference between perceived and actual improvement is itself a senior distinction.
</details>

---

## Module 5.7 — Video & Heavy Media

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** HTML + local server + Chrome

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- No dedicated FM lesson; build on the loading/critical-path knowledge from **Web Performance Fundamentals, v2**. Lean on primary sources.

**Primary sources:**
- **web.dev — Optimize video** — search "web.dev video performance" / https://web.dev/articles/fast-playback-with-preload — *Focus on: `preload` settings, `poster`, lazy video, and not autoplaying heavy video.*
- **MDN — `<video>` attributes** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video — *Focus on: `preload`, `poster`, `muted`/`autoplay` constraints.*
- **web.dev — Replace GIFs with video** — https://web.dev/articles/replace-gifs-with-videos — *Focus on: animated GIFs being enormous; MP4/WebM are far smaller.*

**Search prompts:**
- `"video preload poster lazy load performance"`
- `"replace animated gif with video file size"`
- `"autoplay background video performance cost"`
- Ask an AI: *"Explain video performance: the `preload` attribute (`none`/`metadata`/`auto`) and when each is right, using a `poster` so the video area isn't blank, lazy-loading offscreen videos, and why animated GIFs are huge compared to an MP4/WebM. What's the cost of an autoplaying background video, especially on mobile/weak hardware?"*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 5, Module 5.7, Exercise 1 — video & heavy media.
Completed: Tracks 0–4, Modules 5.1–5.6.

Build me a page with media problems: a large animated GIF used for a short loop (huge bytes), a video
with `preload="auto"` that downloads megabytes before the user ever plays it, no poster (blank area +
CLS), and an autoplaying background video that hammers a throttled device. Serve locally. Do NOT name
the issues.

Have me measure (transfer bytes, the wasted video preload, CPU during autoplay under throttle). Make
me fix each: replace the GIF with a muted looping MP4/WebM, set `preload="none"`/`metadata`, add a
`poster` with reserved space, lazy-load the offscreen video, and reconsider the autoplay cost.
Re-measure. Exam me on: GIF-vs-video bytes, the preload settings, poster/CLS, and autoplay cost on
weak hardware.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 5, Module 5.7, Exercise 2 (transfer) — media decisions.
Apply the Exercise 2 rules: solo first; verdict.

Build me a DIFFERENT media-heavy page (e.g. a landing page with a hero video and several inline
clips). Have me, solo: make the right call for each piece of media — format, preload strategy,
poster, lazy-loading, and whether autoplay is even justified given weak-device cost — and prove the
byte + CPU savings. Don't tell me the answers. Exam me on the tradeoffs, especially the weak-hardware
autoplay cost. Verdict: can I make sound heavy-media decisions (proceed), or redo Exercise 1?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

- The problems: an **animated GIF** (GIFs are enormous — often 5–10x the size of an equivalent video) used for a loop; a `<video preload="auto">` that **downloads the whole video** before the user clicks play (wasted megabytes); **no `poster`**, so the video area is blank then shifts (CLS); an **autoplaying background video** that consumes CPU/GPU/battery and bandwidth, brutal on weak devices and metered connections.
- The fixes: replace the GIF with a **muted, looping, `playsinline` MP4/WebM** (far smaller); set **`preload="none"`** or **`"metadata"`** so the browser doesn't download the whole file upfront; add a **`poster`** image and reserve space (no CLS); **lazy-load** offscreen videos; and question whether autoplay is worth its cost — if kept, it must be muted, and you should consider disabling it on slow connections / reduced-data / weak devices.
- Re-measure: large byte savings, no wasted preload, lower CPU under throttle.

The principle: **heavy media is often the single largest payload on a page, so the wins are format (video over GIF), conservative preloading, posters with reserved space, lazy-loading, and honest scrutiny of autoplay's cost on weak hardware.** Spotting a multi-megabyte GIF or an auto-preloading hero video is an immediate, high-impact diagnosis.
</details>

---

## Module 5.8 — The Asset Audit (Asset Capstone)

**Difficulty:** ●●●●● · **Format:** Investigation + broken project · **Stack:** HTML/CSS + local server + Chrome/WPT

### Prep — Read & Watch Before You Start

**Watch (Frontend Masters — prerequisite):**
- Revisit **Web Performance Fundamentals, v2** → the whole **"Improving Largest Contentful Paint"** section. This capstone integrates every asset lesson.

**Primary sources:**
- **web.dev — Optimize LCP (revisit)** — https://web.dev/articles/optimize-lcp — *Focus on: the asset side of LCP end to end.*
- **web.dev — Lighthouse image/font audits** — search — *Focus on: the specific audits that flag oversized images, missing dimensions, render-blocking fonts.*

**Search prompts:**
- `"image audit oversized unsized lighthouse"`
- `"asset optimization checklist images fonts video"`
- Ask an AI: *"Give me a complete asset-audit methodology for a page: find every oversized/mis-formatted image, every unsized image (CLS risk), the LCP image and whether it's prioritized correctly, render-blocking or shifting fonts, and heavy media — then prioritize the fixes by impact on bytes and CWV."*

### Exercise 1 — Guided (paste into Claude Code)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 5, Module 5.8, Exercise 1 — asset audit, capstone.
Completed: Tracks 0–4, Modules 5.1–5.7. Make it genuinely hard.

Build me a realistic content-heavy page (a landing/article/product page) loaded with asset problems
at once: mis-formatted + oversized images, one giant non-responsive image, a lazy-loaded LCP image,
unsized images causing CLS, a render-blocking/shifting web font, and a heavy GIF or auto-preloading
video. Do NOT tell me how many problems or what they are.

Full asset audit. Have me: baseline (total bytes, LCP, CLS, the waterfall/WPT filmstrip), then go
asset by asset — classify each problem (format? size? priority? dimensions? font? media?) and apply
the matching fix, one at a time, re-measuring bytes + CWV cumulatively. Hold the loop. Long exam
tying each fix to its mechanism + metric. This rehearses auditing a real page's assets cold.
```

### Exercise 2 — Transfer (paste into Claude Code; Claude verifies)

```text
Follow CLAUDE.md and PERF-CONTRACT.md. Track 5, Module 5.8, Exercise 2 (transfer) — audit a DIFFERENT
asset-heavy page solo. Apply the Exercise 2 rules: minimal hints; full verdict.

Build me a DIFFERENT asset-heavy page with a different mix of problems. Have me audit and fix the
WHOLE thing solo — per-asset classification, one-at-a-time fixes, cumulative re-measurement of bytes
and CWV, and a before/after report I could hand a client. Near-zero hints. Hard exam. Then the
verdict: can I open a real content-heavy site cold and audit its images/fonts/media for byte and CWV
wins (proceed to Track 6), or do I need to redo parts of Track 5?
```

### 🔒 SPOILER — The Planted Issues & What It Teaches

<details>
<summary>Click only after you finish or give up</summary>

The page combines the whole track:
- **Mis-formatted/oversized images** (5.1) → right format + compression.
- **A non-responsive giant image** (5.2) → srcset/sizes.
- **A lazy-loaded LCP image** (5.3) → eager + fetchpriority/preload.
- **Unsized images causing CLS** (5.4) → width/height/aspect-ratio.
- **A render-blocking/shifting font** (5.5) → font-display + preload + metric-matched fallback.
- **A heavy GIF or auto-preloading video** (5.7) → video format + preload="none" + poster.

The methodology you should now run unprompted:
1. **Baseline** total bytes, LCP, CLS, and a filmstrip.
2. **Walk every asset.** For each, classify the problem: wrong *format*, wrong *size*, wrong *priority*, missing *dimensions*, *font* handling, or *media* weight.
3. **Apply the matching fix per asset, one at a time, re-measure** bytes and the relevant CWV.
4. **Report** the cumulative byte and CWV improvement.

The principle: **assets are usually the biggest, easiest performance wins, and an asset audit is a systematic per-asset classification — format, size, priority, dimensions, fonts, media — each mapped to a specific fix and a specific metric.** This capstone is your asset-side end goal: open a content-heavy page and immediately spot the oversized hero, the unsized images, the blocking font, and the bloated media. If you can do Exercise 2 cold, you own the asset bucket.
</details>

---

*End of Track 5 — Assets: Images, Fonts & Media. 8 modules. You can now audit and fix the heaviest, most common real-world performance problems — images, fonts, and media — for big byte savings and CWV wins.*

*Next: Track 6 — React & Next.js Performance at Scale, where your actual stack's performance gets the deep treatment.*
