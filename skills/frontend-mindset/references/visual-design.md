# Visual Design — Color, Typography, Layout

## Color

### Use OKLCH, not HSL

`oklch(lightness chroma hue)` — lightness 0-100%, chroma ~0-0.4, hue 0-360. Perceptually uniform: equal lightness steps *look* equal, unlike HSL where 50% lightness in yellow looks bright but 50% in blue looks dark.

To build a primary color's variants, hold chroma+hue constant and vary lightness, but reduce chroma as lightness approaches white/black (high chroma at extremes looks garish).

The hue is a brand decision. Don't reach for blue (hue ~250) or warm orange (hue ~60) by reflex — those are the dominant AI-design defaults.

### Palette structure

| Role | Purpose | Example |
|------|---------|---------|
| Primary | Brand, CTAs, key actions | 1 color, 3-5 shades |
| Neutral | Text, backgrounds, borders | 9-11 shade scale |
| Semantic | Success/error/warning/info | 4 colors, 2-3 shades each |
| Surface | Cards, modals, overlays | 2-3 elevation levels |

Skip secondary/tertiary accents unless genuinely needed — most UIs work with one accent color.

**Tinted neutrals**: pure gray (chroma 0) feels lifeless next to a colored brand. Add chroma 0.005–0.015, hued toward the brand's own color — not a generic "warm = friendly, cool = tech" default. The warm-neutral band (lightness 0.84–0.97, chroma <0.06, hue 40-100 — i.e. cream/sand/parchment) is the saturated AI default of 2026; avoid it unless it's genuinely the brand's own neutral.

**60-30-10 by visual weight, not pixel count**: 60% neutral backgrounds/whitespace, 30% secondary (text, borders, inactive states), 10% accent (CTAs, highlights, focus). Accent colors work *because* they're rare — overusing "the brand color" everywhere kills its power.

### Contrast (WCAG)

| Content | AA min | AAA target |
|---|---|---|
| Body text | 4.5:1 | 7:1 |
| Large text (≥18px or bold ≥14px) | 3:1 | 4.5:1 |
| UI components / icons | 3:1 | 4.5:1 |

Dangerous combos: light gray on white, red/green together (8% of men can't distinguish), blue on red (vibrates), yellow on white. If contrast is even close, bump the body color toward ink, don't trust the eye — use a checker.

Gray text on a colored background looks washed out — use a darker shade of the background's own hue, or transparency on the text color, not a generic gray.

### Dark mode is not inverted light mode

| Light | Dark |
|---|---|
| Shadows for depth | Lighter surfaces for depth (no shadows) |
| Dark text on light | Light text on dark, reduce weight slightly |
| Vibrant accents | Desaturate accents slightly |
| White background | Pure black, or a brand-tinted near-black (oklch 12-18%) |

Build a 3-step surface elevation scale (e.g. 15%/20%/25% lightness) using the same hue/chroma as the brand, varying only lightness. Light text on dark reads heavier than dark-on-light — compensate by stepping body weight down (400→350) or by raising line-height 0.05-0.1 and adding letter-spacing 0.01-0.02em.

### Token hierarchy

Two layers: primitive (`--blue-500`) and semantic (`--color-primary: var(--blue-500)`). Dark mode only redefines the semantic layer.

Heavy use of `rgba`/`hsla` alpha is usually a design smell — an incomplete palette. Define explicit overlay colors per context instead; alpha is fine for focus rings and interactive states where see-through is structurally needed.

**Never**: rainbow palettes (cap at 2-4 colors beyond neutrals), color as the only state indicator (pair with icon/label), untested red/green pairs, default purple-blue gradients.

---

## Typography

### Hierarchy: fewer sizes, more contrast

Five sizes cover most needs — caption, secondary, body, subheading, heading. Use a consistent ratio (1.25, 1.333, or 1.5) between them; don't pick sizes arbitrarily. Sizes too close together (14/15/16px) create muddy hierarchy — go bigger jumps, not finer ones.

Combine size + weight + color + space for hierarchy; don't rely on size alone. A heading that's larger, bolder, AND has more space above it reads as primary without trying.

Weight contrast needs to be loud to register: pair a 900 weight with a 200 weight, not 600 with 400 — adjacent weights on the same scale read as a rendering inconsistency, not a deliberate hierarchy choice.

### Scale: fluid vs fixed

- **Fluid `clamp(min, preferred, max)`**: headings/display text on marketing pages where text dominates and needs to breathe across viewports. Bound the ratio: `max ≤ ~2.5× min`. Scale container width and font-size together so character measure stays in 45-75ch at every viewport.
- **Fixed `rem`**: app UIs, dashboards, data-dense interfaces — no major design system (Material, Polaris, Primer, Carbon) uses fluid type in product UI. Body text stays fixed even on marketing pages.

Hero/display heading ceiling: `clamp()` max ≤ 6rem (~96px) — above that the page is shouting. Letter-spacing floor on display headings: ≥ -0.04em, tighter and letters touch.

### Pairing

You often don't need a second font — one family in multiple weights creates cleaner hierarchy than two competing typefaces. When pairing, contrast on a real axis: serif+sans, geometric+humanist, condensed-display+wide-body. Never pair two fonts that are similar but not identical (two geometric sans-serifs).

System fonts (`-apple-system, BlinkMacSystemFont, "Segoe UI", system-ui`) are underrated for apps where performance > personality — native look, instant load, highly readable.

### Readability

- `max-width: 65ch` on text containers (45-75ch range)
- Line-height: tighter for headings (1.1-1.2), looser for body (1.5-1.7); bump 0.05-0.1 for light-on-dark
- Body text ≥16px / 1rem, always — never `px`, use `rem` to respect user zoom settings
- `text-wrap: balance` on h1-h3 for even line lengths; `text-wrap: pretty` on long prose
- Pick paragraph spacing OR first-line indent, never both

### Loading without layout shift

```css
@font-face {
  font-family: 'CustomFont';
  src: url('font.woff2') format('woff2');
  font-display: swap; /* or 'optional' if zero shift matters more than branded font on slow networks */
}
```

Match fallback metrics (`size-adjust`, `ascent-override`, `descent-override`, `line-gap-override`) to minimize reflow — tools like Fontaine calculate these automatically. Preload only the critical weight (usually body regular). Use a variable font for 3+ weights/styles; static files are fine for 1-2.

**Never**: more than 2-3 font families, body text below 16px, `user-scalable=no`, decorative fonts for body text, default to Inter/Roboto/Open Sans when personality matters.

---

## Layout & Spacing

### Spacing system

Use a consistent scale (framework scale, rem tokens, or custom) — what matters is values come from a defined set, not arbitrary numbers. Prefer a 4pt base (4, 8, 12, 16, 24, 32, 48, 64, 96) over 8pt; 8pt is too coarse and you'll need 12 between 8 and 16. Use `gap` for sibling spacing over margins — avoids margin-collapse hacks.

### Rhythm through contrast, not uniformity

Tight grouping for related elements (8-12px), generous separation between sections (48-96px). Equal padding everywhere kills rhythm. Vary spacing within sections too — not every row needs the same gap.

### Layout tool choice

- **Flexbox**: 1D — rows, nav bars, button groups, component internals
- **Grid**: 2D — page structure, dashboards, anything needing coordinated rows AND columns
- **Container queries** for components, viewport queries for page layout:

```css
.card-container { container-type: inline-size; }
@container (min-width: 400px) {
  .card { grid-template-columns: 120px 1fr; }
}
```

### Hierarchy

The squint test: blur your eyes — can you still identify primary, secondary, and groupings? Space alone is often enough; add color or size contrast only when space isn't sufficient. The strongest hierarchy combines 2-3 tools at once:

| Tool | Strong | Weak |
|---|---|---|
| Size | 3:1 ratio+ | <2:1 |
| Weight | Bold vs Regular | Medium vs Regular |
| Color | High contrast | Similar tones |
| Space | Surrounded by whitespace | Crowded |

### Card-grid monotony

Cards are the lazy default — use them only when content is genuinely distinct and actionable. Never nest cards inside cards. Vary sizes, span columns, or mix cards with non-card content to break repetition. The hero-metric layout (big number, small label, gradient accent) is a template, not a design decision — fine if it shows real data, wrong as decoration.

### Optical adjustments

- Touch targets ≥44×44px even when the visual element is smaller — expand the hit area with padding or a pseudo-element, don't enlarge the icon itself.
- Geometrically centered glyphs often look off-center (play icons shift right, arrows shift toward direction). Only nudge if you're confident it looks wrong — don't adjust speculatively.

**Never**: arbitrary spacing values outside the scale, identical spacing everywhere, wrapping everything in cards, nesting cards, identical card grids as a default template.
