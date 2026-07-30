# Motion & Interaction

## Motion timing

The 100/300/500 rule — duration matters more than easing curve for "feels right":

| Duration | Use case |
|---|---|
| 100-150ms | Instant feedback: button press, toggle, color change |
| 200-300ms | State changes: menu open, tooltip, hover |
| 300-500ms | Layout changes: accordion, modal, drawer |
| 500-800ms | Entrance animations: page load, hero reveal |

Exit animations run faster than entrances — ~75% of enter duration.

```css
--ease-out-quart: cubic-bezier(0.25, 1, 0.5, 1);
--ease-out-quint: cubic-bezier(0.22, 1, 0.36, 1);
--ease-out-expo:  cubic-bezier(0.16, 1, 0.3, 1);
/* never: bounce/elastic curves — feel dated, draw attention to the animation itself */
```

## What to animate

- **Reliable defaults**: `transform`, `opacity` — movement, press feedback, list choreography
- **Atmospheric, when it earns it**: blur/filter/backdrop-filter (focus pulls, glass/lens), clip-path/masks (wipes, editorial reveals), shadow/glow (energy, focus, active state)
- **Never animate casually**: layout-driving properties (`width`, `height`, `top`, `left`, margin) — use FLIP-style transforms or `grid-template-rows` to fake layout animation instead

List staggering is legitimate for cards-in-a-grid or list-items-appearing — cap total stagger time (10 items × 50ms = 500ms; reduce per-item delay past that). Whole-section fade-on-scroll for every section is not staggering, it's the saturated AI default — reserve scroll-triggered reveals for moments that earn it. A reveal must enhance an already-visible default; never gate content visibility on a class-triggered transition (it never fires on hidden tabs/headless renders, so the section ships blank).

## Advanced techniques — reach for these when polish alone won't close the gap

- **View Transitions API**: shared-element morphing between states (a list item expanding into a detail page, a button morphing into a dialog) — closest thing to native FLIP animation, no library needed. Same-document works in all major browsers; cross-document has no Firefox support yet.
- **`@starting-style`**: animate an element in from `display: none` with CSS only, including entry keyframes — no JS mount-animation hack needed.
- **`animation-timeline: scroll()`**: CSS-only scroll-driven animation (parallax, progress bars, reveal sequences). Chrome/Edge/Safari; Firefox is flag-only — always ship a static fallback behind `@supports`.
- **Virtual scrolling**: required, not optional, past a few hundred rows/items — render only visible rows (TanStack Virtual, or roll your own for simple cases). "Large datasets" in production-hardening.md and "what to animate" above both assume this is already in place.
- **Web Workers / OffscreenCanvas**: move heavy computation or rendering off the main thread so the UI never blocks — appropriate once a feature is provably janky, not a default.

Progressive enhancement is non-negotiable for all of the above — every technique needs a fallback that still looks good without it:

```css
@supports (animation-timeline: scroll()) {
  .hero { animation-timeline: scroll(); }
}
```

Before shipping an ambitious effect, run: **the wow test** (show it to someone cold — do they react?), **the removal test** (take it away — is anything actually lost?), **the device test** (mid-range phone, still 60fps?), **the reduced-motion test** (still good with motion off?). An effect that fails the removal test is decoration, not design — cut it rather than ship it as default scaffolding.

## Performance

- `will-change` only for known-expensive animations, scoped to `:hover`/`.animating` — never preemptively page-wide
- Intersection Observer over scroll listeners; unobserve after firing once
- Bound expensive blur/filter/shadow areas — small or isolated, use `contain`
- Target 60fps (16ms/frame); `requestAnimationFrame` for JS-driven animation

## Perceived performance

- Anything under ~80ms feels instant (the brain buffers sensory input for that long)
- Start transitions preemptively while loading; show content progressively rather than waiting for everything
- Optimistic UI for low-stakes actions (likes, follows) — never for payments or destructive operations
- Ease-out feels satisfying for entrances; ease-in (accelerating toward completion) makes tasks feel shorter via the peak-end effect

## Accessibility

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

Non-negotiable on every animation — typically a crossfade or instant transition as the alternative.

---

## The 8 interactive states

Every interactive element needs: **default, hover, focus, active, disabled, loading, error, success**. The common miss is designing hover without focus or vice versa — they're different; keyboard users never see hover.

## Focus rings

Never `outline: none` without a replacement — that's an accessibility violation. Use `:focus-visible` to show the ring only for keyboard users:

```css
button:focus { outline: none; }
button:focus-visible {
  outline: 2px solid var(--color-accent);
  outline-offset: 2px;
}
```

Ring: ≥3:1 contrast against adjacent colors, 2-3px thick, offset from the element, consistent across all interactive elements.

## Forms

Placeholders aren't labels — they disappear on input, always use a visible `<label>`. Validate on blur, not every keystroke (exception: password strength). Place errors below the field, connected via `aria-describedby`.

## Loading states

Skeleton screens beat spinners — they preview content shape and feel faster. Optimistic updates (show success immediately, rollback on failure) for low-stakes actions only.

## Modals

Use the native `<dialog>` element or the `inert` attribute on background content — both replace hand-rolled focus-trap JS:

```html
<main inert><!-- unfocusable while modal is open --></main>
<dialog open><h2>Title</h2></dialog>
```

```js
dialog.showModal(); // focus trap, closes on Escape, for free
```

## Dropdowns and overlays — the clipping bug

`position: absolute` inside a container with `overflow: hidden`/`auto` gets clipped. This is the single most common dropdown bug in generated code. Fixes, in order of preference:

1. **Popover API** — places the element in the top layer, above everything regardless of z-index/overflow, with light-dismiss and accessibility built in:
```html
<button popovertarget="menu">Open</button>
<div id="menu" popover>...</div>
```
2. **CSS anchor positioning** (Chrome/Edge 125+, no Firefox/Safari yet) — tethers an overlay to its trigger via `position: fixed` + `position-anchor`, escaping ancestor `overflow` clipping; pair with `@position-try` for viewport-edge flipping.
3. **Portal/teleport** (React `createPortal`, Vue `<Teleport>`) to `document.body`, positioned via `getBoundingClientRect()` + `position: fixed`, recalculated on scroll/resize.
4. **Fixed positioning fallback** for unsupported browsers — manual coordinates, flip near viewport edges.

## Destructive actions: undo beats confirm

Users click through confirmation dialogs mindlessly. Remove from UI immediately, show an undo toast, delete for real after the toast expires. Reserve confirmation dialogs for truly irreversible actions (account deletion) or batch/high-cost operations — and when you do confirm, name the action specifically ("Delete project" not "Yes").

## Keyboard navigation

**Roving tabindex** for component groups (tabs, menu, radio group) — one item tabbable, arrow keys move within:

```html
<div role="tablist">
  <button role="tab" tabindex="0">Tab 1</button>
  <button role="tab" tabindex="-1">Tab 2</button>
</div>
```

**Skip links** (`<a href="#main-content">Skip to main content</a>`) — hidden off-screen, visible on focus.

**Gestures aren't discoverable** — swipe-to-delete needs a visible fallback (peeking action, coach mark on first use, or a menu item); never the only way to perform an action.

**Never**: remove focus indicators without an alternative, placeholder-as-label, touch targets <44×44px, generic error messages, custom controls without ARIA/keyboard support.
