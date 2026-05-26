# Plan: Make Exported PNG Match the Design Preview Exactly

**Status:** Open — current export still does not match preview
**Last attempted:** Commit `6ca8c63` (capture original canvas in-place after un-zoom)
**Owner:** Claude + User

---

## 1. Problem Statement

When a user clicks "Export PNG", the saved file should be a 1:1 pixel-accurate copy of the design they see in the preview area (the `#sign-canvas` element). It is not.

**Symptoms observed across iterations:**

| Symptom | Seen in commit |
|---|---|
| Output blank / all transparent | Off-screen stage with `opacity: 0` |
| Layout collapsed to mobile/responsive form | `windowWidth: 1080` html2canvas option |
| Sponsor logos clipped or missing entirely | Off-screen stage with `position: static` |
| Header text/ornament cropped at top | All clone-based attempts |
| Watermark + border missing | All clone-based attempts |
| Custom event background color not applied | Some cloned attempts |
| Fonts fall back to system serif | When `document.fonts.ready` is skipped |

**User-facing goal:** "What I see in the preview is exactly what comes out of Export."

---

## 2. Root-Cause Analysis

### 2.1 Why html2canvas keeps failing for us

html2canvas (v1.4.1) is not a true screenshot — it re-implements a CSS engine by reading `getComputedStyle()` and re-drawing into a 2D canvas. Several CSS features we rely on are poorly supported or have edge cases:

| Feature we use | html2canvas support |
|---|---|
| `display: grid` with `grid-template-rows: auto 1fr auto` | Partial — row sizing can drift |
| `object-fit: contain` on `<img>` | Works in 1.4+ but inconsistent across browsers |
| CSS custom properties (`--logo-size`, `--serif-font`, …) | Works only if the element being captured owns the property |
| `position: absolute` children (`.sign-border`, `.sign-watermark`) | Works only if positioning ancestor is preserved exactly |
| `transform: scale()` (our zoom) | Reset before capture or capture is shrunk |
| `transition` on properties being changed at capture time | Can race the capture and yield partial frames |
| Web fonts (Playfair Display, Phetsarath OT) | Must be fully loaded before capture, else fallback fonts render |
| Multi-line Lao text with `\n` + `white-space: pre-line` | Works |
| Inline `<svg>` (our preset logos) | Generally OK |

### 2.2 Why cloning broke things

Each clone-based attempt failed because:

- `cloneNode(true)` copies the DOM tree and inline styles, but **detaches the clone from the original layout context**. Any `position: absolute` child positioned relative to the original `#sign-canvas` re-resolves against the cloned ancestor, which can differ.
- CSS variables set via `element.style.setProperty('--x', …)` are inline on the *captured root* and inherit downward, but if the clone's root has `position: static` (or its computed style differs), inherited styles can break.
- Children whose width depends on flex/grid distribution (event cards filling the workspace) re-layout when the clone is appended off-screen — and that re-layout can be subtly different from the preview if any ancestor's width is implicit.

### 2.3 What probably still goes wrong in the current "in-place capture"

The current approach captures the real canvas after temporarily un-zooming it. This avoids clone bugs, but html2canvas still has to interpret our CSS, so any of the engine limitations above can still bite:

- Grid layout precision
- Custom font timing on first export
- `object-fit` rendering of uploaded logos
- Edge cases in transform reset (we set `transform: none` inline, but the previous value was `scale(0.3)` — some browsers keep a stale render layer briefly)

---

## 3. Solutions, Ranked by Recommendation

### ⭐ Option A — Switch to `html-to-image` (RECOMMENDED, fast)

**What:** Replace html2canvas with [`html-to-image`](https://github.com/bubkoo/html-to-image). It serializes the DOM to an SVG `foreignObject`, then rasterizes via the browser's native renderer.

**Pros:**
- Uses the browser's actual rendering engine — Grid, Flexbox, `object-fit`, custom fonts, filters, transforms all render exactly as they do on screen
- Drop-in API: `htmlToImage.toPng(node).then(dataUrl => …)`
- Actively maintained (last release 2024)
- ~30 KB minified

**Cons:**
- Same-origin restriction on images: every `<img>` whose `src` is an external URL must respond with proper CORS headers. Our setup uses data URLs (`base64`) for uploaded logos and inline SVG for presets, so this is **not a problem for us**.
- Safari has historically been finicky with `foreignObject` — works in 17+.

**Effort:** ~30 minutes.

**Implementation outline:**
```html
<script src="https://cdn.jsdelivr.net/npm/html-to-image@1.11.13/dist/html-to-image.min.js"></script>
```
```javascript
async function exportAsPNG() {
    if (document.fonts?.ready) await document.fonts.ready;
    const canvas = document.getElementById('sign-canvas');
    const prevTransform = canvas.style.transform;
    canvas.style.transform = 'none';
    await new Promise(r => requestAnimationFrame(r));
    try {
        const dataUrl = await htmlToImage.toPng(canvas, {
            pixelRatio: 1,
            width: outW,
            height: outH,
            cacheBust: true,
        });
        // …trigger download
    } finally {
        canvas.style.transform = prevTransform;
    }
}
```

---

### Option B — `modern-screenshot` (more accurate but heavier)

**What:** [`modern-screenshot`](https://github.com/qq15725/modern-screenshot) — a fork of `html-to-image` with extra fixes for fonts, shadows, and SVG.

**Pros:** Highest fidelity client-side renderer available today.
**Cons:** ~70 KB. Slightly slower. Same `foreignObject` Safari caveat.
**When to choose this:** If Option A still misses edge cases (uncommon).

---

### Option C — Refactor canvas CSS to be html2canvas-friendly

Stay on html2canvas but remove the features it struggles with:

1. **Replace grid with absolute positioning**
   ```css
   .sign-canvas { position: relative; }
   .sign-header   { position: absolute; top: 24px;    left: 50%; transform: translateX(-50%); }
   .sign-workspace{ position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); }
   .sign-footer   { position: absolute; bottom: 24px; left: 50%; transform: translateX(-50%); }
   ```
   Predictable layout, exact pixel positions, no flex/grid ambiguity.

2. **Replace `object-fit: contain` with explicit dimensions**
   On image load, store `naturalW` / `naturalH`, then compute and write inline `width` / `height` that already fit inside the artwork-render box at the correct aspect ratio. No `object-fit` needed.

3. **Inline all CSS variables on the capture root**
   Right before capture, walk the DOM under `#sign-canvas` and replace every `var(--x, …)` reference with the resolved value as an inline style. Eliminates inheritance ambiguity.

4. **Pre-warm fonts**
   On page load, render every text style with `visibility: hidden` once so the browser fetches the WOFF files before the user ever clicks Export.

**Pros:** No new dependency.
**Cons:** Loses the cleaner grid CSS. More code to maintain. Doesn't fix every html2canvas quirk.
**Effort:** ~2 hours.

---

### Option D — Server-side render with Puppeteer / Playwright (highest fidelity)

Move export to a serverless function (Vercel, Cloudflare Workers, or AWS Lambda):

1. Client POSTs the full state JSON to `/api/render`
2. Server spins up headless Chromium, navigates to a rendering URL with the state, calls `page.screenshot()`
3. Server returns the PNG bytes

**Pros:**
- Pixel-perfect — it's a real browser screenshot
- All fonts/features supported
- Output dimension guaranteed

**Cons:**
- Requires backend (we're on GitHub Pages, static-only)
- Cold-start latency 1–3 s
- Chromium binaries are heavy; Vercel free tier may need `@sparticuz/chromium`

**When to choose this:** If client-side renders never reach acceptable fidelity, or we want the same export to work on Mac/Windows/iPad/Android identically.
**Effort:** ~4 hours, plus deploy setup.

---

### Option E — Hand-roll on the 2D Canvas API

Draw every element directly with `ctx.fillText`, `ctx.drawImage`, etc.

**Pros:** Total control, smallest output sizes, no dependencies.
**Cons:** Re-implementing layout (grid, flex, text wrapping, ordinals, dividers) is weeks of work. Hard to keep in sync with the live design as new features are added.
**Verdict:** Not worth it for our scope.

---

## 4. Recommendation

**Do Option A first.** It's the lowest-effort change with the highest probability of fixing every symptom we've seen, because it uses the browser's real renderer instead of a CSS interpreter. We keep our entire current CSS as-is.

If Option A still has gaps (very unlikely for our markup), escalate to Option B.

Reserve Option D for a future "Pro" tier if pixel-perfect cross-platform consistency becomes a paid feature.

---

## 5. Implementation Plan for Option A

### 5.1 Code changes

1. **Replace CDN script** in `<head>`:
   - Remove: `https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js`
   - Add: `https://cdn.jsdelivr.net/npm/html-to-image@1.11.13/dist/html-to-image.min.js`

2. **Rewrite `exportAsPNG()`** (in `index.html`):
   ```javascript
   async function exportAsPNG() {
       if (typeof htmlToImage === 'undefined') {
           showToast('ไลบรารี html-to-image ไม่พร้อม', 'error');
           return;
       }
       showToast('กำลังเตรียม PNG... กรุณารอ');

       if (document.fonts?.ready) {
           try { await document.fonts.ready; } catch (e) {}
       }

       const canvas = document.getElementById('sign-canvas');
       const wrap   = document.getElementById('canvas-wrap');
       const outW = state.orientation === 'portrait' ? 1080 : 1920;
       const outH = state.orientation === 'portrait' ? 1920 : 1080;

       const prev = {
           transform: canvas.style.transform,
           wrapOverflow: wrap.style.overflow,
           wrapPadding: wrap.style.padding,
       };
       canvas.style.transform = 'none';
       wrap.style.overflow = 'visible';
       wrap.style.padding = '0';

       await new Promise(r => requestAnimationFrame(r));
       await new Promise(r => requestAnimationFrame(r));

       try {
           const dataUrl = await htmlToImage.toPng(canvas, {
               pixelRatio: 1,
               width: outW,
               height: outH,
               cacheBust: true,
               style: { transform: 'none' },
           });
           const link = document.createElement('a');
           link.download = `Souphattra_Sign_${outW}x${outH}_${Date.now()}.png`;
           link.href = dataUrl;
           link.click();
           showToast(`บันทึก PNG ${outW}×${outH}px สำเร็จ!`, 'success');
       } catch (err) {
           console.error(err);
           showToast('เกิดข้อผิดพลาด: ' + err.message, 'error');
       } finally {
           canvas.style.transform = prev.transform;
           wrap.style.overflow = prev.wrapOverflow;
           wrap.style.padding = prev.wrapPadding;
           updateZoom();
       }
   }
   ```

3. **Keep all current CSS** — grid layout, CSS variables, `object-fit: contain`, transitions. `html-to-image` handles them.

### 5.2 Test matrix

After implementing, verify all of these match preview ↔ PNG:

- [ ] Default state (Dual mode, 2 events, 4 preset logos in event 1, 2 in event 2)
- [ ] Single mode with uploaded hotel logo
- [ ] Triple mode with custom backgrounds from PPTX colors
- [ ] Dark theme + dark canvas
- [ ] Custom event background (e.g., bright red `#E4002B`)
- [ ] Long Lao multi-line title that wraps
- [ ] Bilingual title (Lao + English)
- [ ] After dragging an artwork to a non-default position
- [ ] After rotating an artwork
- [ ] Banner-mode artwork (full-width)
- [ ] Hotel name text toggled ON
- [ ] Footer ❖ ❖ ❖ toggled OFF
- [ ] Border toggled OFF
- [ ] Watermark toggled OFF
- [ ] Largest Floor size (200px)
- [ ] Largest canvas padding (400px)
- [ ] Box Y-offset at +100 and at −100
- [ ] Every font in the font dropdown (Cinzel, EB Garamond, etc.)

### 5.3 Acceptance criteria

PNG is considered correct when, for the same state:
1. Output dimensions are exactly 1080×1920 (portrait) or 1920×1080 (landscape)
2. All event cards visible, sized identically to preview
3. All artwork (preset or uploaded) at the same position, scale, rotation as preview
4. Hotel logo / ornament at top with the same size
5. Border line, watermark visibility, and footer match preview toggles
6. Selected font is used for every serif element (visually distinguishable from default Playfair)
7. Custom background colors render with correct hex value (verifiable in image editor)
8. Lao text renders in Phetsarath, not a fallback

---

## 6. Fallback Plan

If Option A fails any acceptance test:

1. Capture a screenshot of the failing case + record state JSON
2. Try Option B (`modern-screenshot`) — same API shape, swap library only
3. If still failing, schedule Option C refactor (1–2 days) or Option D server-side (~half day to set up)

---

## 7. Out of Scope (parked for later)

These are nice-to-have but not part of fixing export fidelity:

- Watermark pattern improvement (multiple scattered ornaments vs single centered)
- Better default Gov Lao preset SVG (closer to real emblem)
- Per-event font selection
- Export at 2× DPI for print
- JPEG/WEBP export option
- Batch export of all 3 events as separate PNGs
