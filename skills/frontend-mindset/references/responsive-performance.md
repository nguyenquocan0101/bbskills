# Responsive Design & Performance

## Mobile-first, content-driven breakpoints

Write base styles for mobile, layer complexity with `min-width` queries — desktop-first (`max-width`) loads unnecessary styles first. Don't chase device sizes; start narrow, stretch until the design breaks, add a breakpoint there. Three breakpoints usually suffice (640/768/1024px); use `clamp()` for fluid values where a breakpoint isn't needed.

## Detect input method, not just screen size

A laptop can have a touchscreen; a tablet can have a keyboard. Screen size doesn't tell you input method:

```css
@media (pointer: coarse) { .button { padding: 12px 20px; } } /* touch: bigger target */
@media (hover: none) { .card { /* no hover-dependent functionality */ } }
```

Never rely on hover for functionality — touch users can't hover.

## Touch targets and safe areas

44×44px minimum, even when the visual element is smaller (pad with a pseudo-element). Handle notches/home indicators with `env()`:

```css
.footer { padding-bottom: max(1rem, env(safe-area-inset-bottom)); }
```

```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
```

## Responsive images

```html
<img src="hero-800.jpg"
  srcset="hero-400.jpg 400w, hero-800.jpg 800w, hero-1200.jpg 1200w"
  sizes="(max-width: 768px) 100vw, 50vw"
  loading="lazy" alt="...">
```

Use `<picture>` + `<source media>` when art direction differs across breakpoints (different crop, not just resolution), not just `srcset`.

## Adaptation is not scaling

Each target context needs rethought structure, not pixel scaling:

| Context | Layout | Interaction |
|---|---|---|
| Mobile | Single column, bottom nav | Touch targets, swipe, thumb-zone controls |
| Tablet | Two-column, master-detail | Touch + pointer both |
| Desktop | Multi-column, persistent side nav | Hover, keyboard shortcuts, right-click, multi-select |
| Print | Page breaks, no nav/interactive elements | — |

Never hide core functionality on mobile, never assume desktop = powerful device, always test landscape orientation.

## Core Web Vitals

| Metric | Target | Fix |
|---|---|---|
| LCP (Largest Contentful Paint) | <2.5s | Optimize hero image, inline critical CSS, preload key resources, SSR |
| INP/FID | <200ms / <100ms | Break up long tasks, defer non-critical JS, web workers for heavy compute |
| CLS (Cumulative Layout Shift) | <0.1 | `aspect-ratio` on images/video, never inject content above existing content |

```css
.image-container { aspect-ratio: 16 / 9; } /* reserve space before load */
```

## Loading performance

- Images: WebP/AVIF, correctly sized (not 3000px for a 300px slot), lazy-load below-fold, 80-85% quality is usually imperceptible
- JS: route/component-based code splitting, tree-shake, dynamic `import()` for heavy components
- Fonts: `font-display: swap`/`optional`, subset to needed characters, preload only the critical weight
- Critical CSS inline, the rest async

## Rendering performance

Batch DOM reads and writes — don't alternate (forces layout thrashing):

```js
// bad: read-write-read-write forces reflow each iteration
elements.forEach(el => { el.style.height = el.offsetHeight * 2; });

// good: batch all reads, then all writes
const heights = elements.map(el => el.offsetHeight);
elements.forEach((el, i) => { el.style.height = heights[i] * 2; });
```

- `content-visibility: auto` for long off-screen content; virtualize very long lists
- `transform`/`opacity` for animation (GPU-accelerated); avoid animating `left`/`width` (CPU-bound layout)
- `will-change` sparingly, scoped to known-expensive elements — it allocates a new compositor layer

## Measure before optimizing

Lighthouse, WebPageTest, Chrome DevTools Performance panel, bundle analyzers. Test on a real low-end Android, not just a flagship iPhone on fast wifi — desktop Chrome with fast connection isn't representative.

**Never**: optimize without measuring first, sacrifice accessibility for performance, lazy-load above-fold content, `will-change` everywhere, micro-optimize while the actual bottleneck sits untouched.
