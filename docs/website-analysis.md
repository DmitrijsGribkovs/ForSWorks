# ForS Works — Website Analysis

**Scope:** `index.html` (325 lines), `about.html` (190 lines), `prices.html` (213 lines) and the
14 assets in the project root, all read from this workspace. No network fetch was required; the
live URL `https://forsworks.vercel.app/index.html` was used for cross-checking only.
**Date:** 2026-09-27

Every item below cites a `file:line` from the real source in this workspace, or a command
reproducing the measurement. Priorities are ordered by user impact, not by effort.

---

## Evidence gathered

| Check | Command | Result |
| --- | --- | --- |
| Line counts | `index.html` / `about.html` / `prices.html` | 325 / 190 / 213 |
| Duplicate media | `Get-FileHash -Algorithm MD5 c*.mp4 f*.png` | `c3`=`c4`=`c5`; `f3`=`f4`=`f5` |
| Image payloads | `System.Drawing.Image.FromFile` | `favicon.png` 512x512 / 563 KB; `icon.png` 1024x1024 / 1,920 KB; `bg.png` 1672x940 / 1,768 KB |
| Unreferenced assets | `Select-String -Path *.html -Pattern 'bg2\.png|icon\.png'` | 0 hits for either |
| Video payload | `Get-ChildItem c*.mp4 \| Measure-Object Length -Sum` | 44.3 MB across 5 files; 51.6 MB total repo |
| Dead CSS / a11y / SEO tags | `Select-String` for `.bg-slider {`, `meta name="description"`, `og:`, `aria-current`, `prefers-reduced-motion`, `poster=`, `role=`, `tabindex`, `<main`, `<header`, `<footer`, `loading=`, `\.css` | **0 matches for every pattern** |

---

## P0 — Breaks the product for real visitors

### 1. The media query hides the site's only tagline and its only call to action

`index.html:199-204`

```css
@media (max-width: 2168px) {
    .content-section p,
    .cta-button {
        display: none !important;
    }
}
```

The comment on `index.html:198` says "Mobile: hide paragraph and button", but `2168px` is a
desktop-width threshold, not a mobile one. The rule therefore matches on **every** viewport up to
2168 CSS pixels — every phone, every laptop, every 1440p display, and most 4K displays once the OS
applies any scaling. The tagline `3D models painted with love.` (`index.html:223`) and the
`View Prices` button (`index.html:224`) render only on viewports *wider* than 2168px, i.e. almost
never. The home page currently presents a bare `<h1>` and a thumbnail strip.

**Fix:** use a real mobile breakpoint (`max-width: 768px`) and, if the intent really was to declutter
large screens, add a separate `min-width` rule for the oversized case instead of one inverted query.

### 2. `overflow: hidden` plus `position: absolute` makes page content unreachable

`index.html:18`, `about.html:18`, `prices.html:18` all set `overflow: hidden` on `html, body`, and
all three layout their content with `position: absolute` (`index.html:146-156`,
`about.html:68-79`, `prices.html:68-79`).

There is no scroll container anywhere, so anything extending past the viewport is permanently
clipped. `prices.html` is the worst case: `.price-grid` is `flex-wrap: wrap`
(`prices.html:94-99`) with no breakpoint, so on a narrow screen the three `.price-card` blocks stack
to roughly 1000px of content under a `top: 90px` offset, and the bottom two cards are simply
unreachable. `prices.html` and `about.html` have **no media queries at all** — the entire responsive
strategy for the site is the one broken rule in item 1.

**Fix:** use `min-height: 100vh` with normal document flow (`position: static`, scrollable
overflow) plus an `overflow: hidden` wrapper on `index.html` only, which genuinely is a fixed
full-bleed canvas.

### 3. The three pages describe three different businesses

- `index.html:223` — "3D models painted with love."
- `about.html:175` — "calm, memorable outdoor experiences … from first light on the peaks to quiet evenings in the valley"
- `about.html:178-179` — "make it easy to explore striking landscapes", "a local host, and a gallery of the day"
- `prices.html:178` — "Choose a plan that fits your next experience. Every package includes gallery access and personal support."
- `prices.html:184`, `195`, `204-206` — "Half-day guided visit", "Full-day experience", "Private custom itinerary", "Dedicated host"

The home page and the 21 background videos (`index.html:234-254`) are a 3D-model-paint shop; the
About and Prices pages are copy-pasted guided-hiking-tour content. A visitor who clicks `Prices`
lands on a different company. Nothing on the site names a 3D model, a medium, a portfolio, or a
licence.

**Fix:** decide what ForS Works sells, then rewrite `about.html:175-179` and the three
`.price-card` blocks at `prices.html:180-209` to match. This is a content decision, not a code
decision — it needs the owner.

### 4. There is no conversion path anywhere on the site

Every call to action is a link to another static page:

- `index.html:224` → `prices.html`
- `prices.html:188` "Get Started" → `about.html`
- `prices.html:198` "Choose Premium" → `about.html`
- `prices.html:208` "Book Private" → `about.html`

There is no contact form, no email address, no checkout, no cart, no enquiry target, and no terms
or privacy page. A visitor who wants to buy reaches `about.html`, reads two paragraphs, and finds
one unverifiable Instagram link (`about.html:182`). The pricing page names three paid tiers and no
way to pay for any of them.

**Fix:** point the `.cta-button` hrefs at a real destination — a mailto, a form, or a checkout
route — and add legal/footer content.

---

## P1 — Performance: 51.6 MB for three static pages

### 5. The slide list is 21 entries long and 16 of them are the same file

`index.html:233-255`

```js
{ video: "c5.mp4", thumb: "f5.png" },   // lines 239-253, repeated 15 more times
```

`index.html:238-253` declares `c5.mp4`/`f5.png` sixteen times, and the array is closed with a repeat
of the *first* entry at `index.html:254`. Hashing the assets confirms this is not just a repeated
reference but duplicated payload:

```
c3.mp4 == c4.mp4 == c5.mp4   (11,351,730 bytes each, MD5 E0E674585C501F0FF277239772225AF4)
f3.png == f4.png == f5.png   (228,211 bytes each,  MD5 C79ACECC3F54DE377441EC628598B05E)
```

The `slides.forEach` at `index.html:261-289` turns all 21 entries into 21 `<video>` elements and 21
`<img>` elements for what is really 5 unique pairs. The browser will cache-share the bytes, but it
instantiates 21 independent media players, 21 decoders, and a 21-thumbnail strip ~2835px wide inside
an 800px container (`index.html:63`).

**Fix:** reduce the array to the 5 unique pairs, delete `c4.mp4` and `c5.mp4` and `f4.png`/`f5.png`
(21.7 MB + 446 KB), and re-point the array. If a longer rotation is wanted, rotate a 5-element list
instead of duplicating it.

### 6. `autoplay` on every slide defeats the `preload` optimisation on the line above it

`index.html:269-271`

```js
slide.autoplay = true;
// Only preload/play the first video eagerly; others load on demand
slide.preload = index === 0 ? 'auto' : 'none';
```

The comment states the intent, but `autoplay = true` instructs the browser to fetch and begin
decoding each of the 21 video elements regardless of `preload = 'none'`. Setting `preload` after
`autoplay` does not undo it. Net effect: all 44.3 MB of video starts downloading on page load, and
the "load on demand" path never engages.

**Fix:** set `autoplay` only for the active slide (or drop `autoplay` entirely and call `.play()`
on the single active element), and keep `preload="none"` on the rest.

### 7. 3.6 MB of the deploy is never referenced by anything

`Select-String -Path *.html -Pattern 'bg2\.png|icon\.png'` returns **0 matches**. `bg2.png`
(1,801 KB) and `icon.png` (1,920 KB) are dead weight shipped on every request path. Combined with
item 5, roughly 25 MB of the 51.6 MB repo is removable with no code change beyond the array.

### 8. The favicon is a 563 KB, 512x512 PNG on all three pages

`index.html:7`, `about.html:7`, `prices.html:7` all reference `favicon.png`, which is 512x512 and
563 KB. A favicon renders at 16-32px; this file is fetched in full on every page view to be scaled
down. There is also no `apple-touch-icon` and no `<link rel="manifest">`.

**Fix:** export a 32x32 and a 180x180 PNG (a few KB each) and reference those.

### 9. Thumbnails and the background carry no dimensions and no lazy loading

`index.html:278-281` creates each `<img>` with only `src` and `alt`. No `width`/`height` (so the
strip reflows as images decode) and no `loading="lazy"` for the 16 off-screen thumbs. `bg.png`
(`index.html:20`) is 1,768 KB served as the page background for every visitor, and `bg2.png` shows
it is not even a downscaled variant.

**Fix:** set intrinsic `width`/`height` on the thumb elements, add `loading="lazy"` +
`decoding="async"` to all but the first, and ship a WebP/AVIF background at the largest breakpoint
you actually need.

---

## P2 — Accessibility

### 10. The primary navigation is 21 unlabelled, click-only images

`index.html:277-288`

```js
const thumb = document.createElement('img');
thumb.src = item.thumb;
thumb.alt = '';
...
thumb.addEventListener('click', () => changeVideo(index));
```

The thumbnail strip is the only way to move between works on the site, and it is built as `<img>`
elements with an empty `alt`, a click handler, no `tabindex`, no `role`, and no key handler. It is
invisible to keyboard and screen-reader users: a screen reader announces 21 unlabelled images, and
the site is effectively unusable without a mouse. The thumbnails are the content, so `alt = ''` is
also wrong on its own terms — each needs a real description of the piece.

**Fix:** render `<button type="button">` wrapping the `<img>`, give each a descriptive `alt`, and
add `ArrowLeft`/`ArrowRight` handling on the strip. Also call
`activeThumb.scrollIntoView({ inline: 'nearest' })` at the end of `changeVideo`
(`index.html:315-321`) so the activated thumb is not left scrolled out of view.

### 11. No `aria-current` on the active nav link

`index.html:215`, `about.html:170`, `prices.html:172` mark the current page with a CSS `.active`
class only. `aria-current` appears **0 times** across all three files, so assistive tech has no way
to know which page you are on.

### 12. Five autoplaying looping videos with no reduced-motion escape hatch

`index.html:36` sets a `0.8s` cross-fade transition and `index.html:266-269` sets `muted`,
`loop`, `playsInline` and `autoplay` on every slide. `prefers-reduced-motion` appears **0 times** in
the codebase. For a visitor with vestibular sensitivity there is no way to stop the motion short of
leaving the page.

**Fix:** wrap the transitions and the autoplay start in a
`@media (prefers-reduced-motion: no-preference)` guard, and show a static frame when motion is
reduced.

### 13. Price-card lists lose their list semantics and their bullets

`prices.html:135-145`

```css
.price-card ul { list-style: none; text-align: left; margin-bottom: 22px; }
.price-card li { font-size: 15px; line-height: 1.8; padding-left: 4px; }
```

`list-style: none` without a replacement marker and without `role="list"` causes VoiceOver to stop
announcing the three features as a list, and sighted visitors get three unmarked lines with no
visual grouping. `role=` appears **0 times** in the codebase.

### 14. No semantic landmarks on any page

`<main>`, `<header>` and `<footer>` appear **0 times** across all three files. Layout is entirely
`<div>`, `<nav>` and `<section>`. There is also no heading hierarchy beyond a single `<h1>` per
page — `prices.html` jumps from `<h1>` (line 177) to three sibling `<h2>`s inside cards
(`prices.html:181`, `191`, `201`), which is acceptable, but there is no `<main>` to scope them.

### 15. Background videos are neither hidden from assistive tech nor given a poster frame

`index.html:263-269` creates the `<video>` elements with no `aria-hidden`, no `poster`, and no
`<track>`. They are decorative — the `.overlay` at `index.html:44-53` and `pointer-events: none`
confirm that — so they should be `aria-hidden="true"` to keep 21 unlabelled media elements out of
the accessibility tree. The missing `poster` also means a black frame flashes on every slide change
while the video buffers.

### 16. Autoplay rejection is swallowed silently

`index.html:294` and `index.html:313`

```js
firstSlide.play().catch(() => {});
```

If autoplay is blocked (low-power mode, data saver, some mobile browsers) the failure is discarded
and the user gets a flat `#111` background (`index.html:23`) with a fully functional-looking
thumbnail strip that switches nothing. There is no `poster` fallback, so the failure is
indistinguishable from a broken page.

---

## P3 — SEO and social sharing

### 17. No meta description, no canonical, no Open Graph, no theme colour

`Select-String` for `meta name="description"`, `og:`, and `\.css` returns **0 matches** in all three
files. Each page has only `<title>` (`index.html:6`, `about.html:6`, `prices.html:6`). Every shared
link renders as a bare title with no description and no preview image, and there is no canonical URL
to prevent `index.html` and `/` being indexed as duplicates.

### 18. Brand name is spelled two different ways

`index.html:6` uses `ForSWorks` in the `<title>` and the `<link rel="icon">` path, while
`index.html:222` renders `Welcome to ForS Works` and `about.html:182` links to
`instagram.com/ForSWorks`. Pick one spelling and apply it everywhere, including the file naming.

---

## P4 — Maintainability and convention drift

### 19. ~60 lines of CSS are duplicated verbatim across all three files

The `.nav-menu` block at `index.html:114-143` is byte-identical to `about.html:38-66` and
`prices.html:38-66`. The same is true of the `*` reset (`index.html:9-13` = `about.html:9-13` =
`prices.html:9-13`), `.overlay` (`index.html:44-53` = `about.html:30-36` = `prices.html:30-36`), and
`.cta-button` (`index.html:171-187` = `about.html:112-129` = `prices.html:147-163`). `Select-String`
for `\.css` returns **0 matches** — there is no shared stylesheet, so every fix to the nav or the
button must be made three times and will drift.

**Fix:** extract `styles.css` and link it from all three pages. The three `.bg` background URLs
(`about.html:26`, `prices.html:26`) are the only intentional per-page difference worth keeping via
a modifier class.

### 20. `changeVideo` shadows the `slides` array and re-queries the DOM it already has

`index.html:233` declares `const slides = [...]` (the configuration array). `index.html:300` declares
`const slides = document.querySelectorAll('.bg-slide')` inside `changeVideo`, shadowing it. There is
no bug today only because the function never needs the array — but any future line in that function
that reaches for `slides` expecting config gets a `NodeList` instead, and fails silently. The
shadowed name is also why the function cannot find `item.video` to re-`src` a recycled slide.

**Fix:** rename the local to `videoEls` and keep references to the elements created in the
`forEach` at `index.html:261-289`, rather than re-querying on every click.

### 21. `.bg-slider` has no CSS rule at all

`index.html:210` is `<div class="bg-slider" id="bgSlider">`, and
`Select-String -Pattern 'bg-slider\s*\{'` returns **0 matches** — only `.bg-slide` is styled
(`index.html:26-37`). The container is `position: static` with auto height, so its children
(`position: absolute`, `index.html:27-29`) resolve against the initial containing block and it works
by accident. Any future `position: relative` or `overflow` on this container silently breaks the
entire background layer.

**Fix:** give `.bg-slider` an explicit `position: absolute; inset: 0;` rule so the layout intent is
declared rather than emergent.

### 22. Mixed indentation on the favicon line

`index.html:7`, `about.html:7` and `prices.html:7` each begin with a literal TAB character while
every other line in all three files uses four spaces. Trivial to fix, but it is the kind of drift
that signals the files were generated rather than maintained.

### 23. Unverifiable social link and no legal or contact footer

`about.html:182` links to `https://www.instagram.com/ForSWorks`. I cannot verify this handle
exists from the workspace, and the brand is rendered as "ForS Works" at `index.html:222`, so the
handle is worth confirming. It is also the **only** social/contact link on the site
(`.social-links` at `about.html:181-187` contains a single `<a>`), and no page carries a contact
email, copyright line, terms link, or privacy notice.

---

## GitHub review — unscoped, cannot be started

No GitHub organisation or repository was ever named for this task, in this issue or any parent, and
the project's own codebase record has no remote either — the ForS Works project
(`projectId e62aa8bf-16eb-48ca-a609-712ab4961433`) reports `repoUrl: null`, `repoRef: null`, and
`repoName: null` on both its codebase entry and its primary workspace
(`workspaceId 6dbbe29d-9a45-4fb9-9fcf-b678fa4ffe29`), sourced from a local folder rather than a
clone. The project description does say "We have access for github for website", so access is
claimed but never pointed at a specific org or repo. I am not going to guess one.

Once an org/repo is supplied, this is what I would review:

1. **Deploy path and build** — whether the three HTML files plus 14 assets at the repo root match
   what is actually served on Vercel, or whether a build step rewrites them.
2. **Media pipeline** — how `c1.mp4`–`c5.mp4` and `f1.png`–`f5.png` are produced and re-encoded.
   The byte-identical `c3`/`c4`/`c5` and `f3`/`f4`/`f5` sets (items 5 and 6) suggest a broken or
   partially-completed export step; I would find the script and fix it at the source rather than
   hand-deleting the duplicates.
3. **Where `bg2.png` and `icon.png` are used** — they are unreferenced by any HTML in this
   workspace (item 7), so either they are consumed by something outside these three pages, or they
   are genuinely abandoned. The repo history would settle that.
4. **Asset budget enforcement** — 51.6 MB in a static site with no compression step. I would look
   for a missing image/video optimisation pass and a size ceiling in CI.
5. **Whether a shared stylesheet was ever planned** — the triplicated CSS (item 19) will be
   painful to fix by hand three times; if there is a build step, the fix belongs in it.

---

## Suggested order of work

1. Items 1 and 2 — one-line CSS changes that unblock the home page and make the price page usable
   on mobile. Do these first and re-test on a 390x844 viewport and a 1280x600 viewport.
2. Item 5 and 6 — reduce the array to 5 entries and fix autoplay. Expect the largest single
   performance win; re-measure the transferred bytes on load before and after.
3. Items 3 and 4 — resolve with the owner what the site is selling, then fix copy and add a real
   conversion path. These need a human decision, not just a patch.
4. Item 8, 7 and 9 — asset weight cleanup. Mechanical once 5 is done.
5. Items 10-16 — accessibility pass, best done as one sweep since they touch the same elements.
6. Item 19 — extract `styles.css` before any of the above, so the rest of the fixes land in one
   place instead of three.
