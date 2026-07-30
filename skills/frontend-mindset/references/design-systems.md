# Design Systems & Register

## Pick a register first

Every design decision below (color, type, motion, density) depends on which register the surface is in. Decide before touching color or type.

**Brand** — marketing, landing pages, campaigns, portfolios, long-form content. Design IS the product; the slop test is "would someone say AI made this."

**Product** — app UI, admin, dashboards, tools, authenticated surfaces. Design SERVES the product; the slop test is "would a user fluent in Linear/Stripe/Notion trust this without pausing." Familiarity is a feature here, not a failure — the actual failure mode is strangeness without purpose (over-decorated buttons, invented affordances for standard tasks, display fonts in labels).

| | Brand | Product |
|---|---|---|
| Typography scale | Fluid `clamp()`, ratio ≥1.25 | Fixed `rem`, ratio 1.125-1.2 |
| Color | Pick a strategy below, can exceed Restrained | Restrained is the floor; accent = action/state only |
| Motion | One rehearsed hero moment, scroll choreography earns its place | 150-250ms, state-only, no page-load sequence |
| Layout | Asymmetric, fluid spacing, intentional grid-breaking | Predictable grids, structural responsiveness (not fluid type) |
| Fonts | Genuine display/body pairing | One well-tuned family often carries everything |

Product permissions (things brand surfaces can't get away with): system font stacks, standard nav patterns (top bar + side nav, breadcrumbs, command palettes), density (many rows, many labels), consistency over surprise screen-to-screen.

Product bans (beyond the shared absolute bans in SKILL.md): decorative motion that doesn't convey state, inconsistent component vocabulary across screens (the "save" button must look the same everywhere), display fonts in UI labels/data, reinvented standard affordances (custom scrollbars, weird form controls), modal-as-first-thought (exhaust inline/progressive alternatives first).

## Color strategy (brand register)

Pick the strategy before picking colors:

| Strategy | Description | When |
|---|---|---|
| Restrained | Tinted neutrals + one accent ≤10% | Product default; brand minimalism |
| Committed | One saturated color carries 30-60% of the surface | Identity-driven brand pages |
| Full palette | 3-4 named roles, each deliberate | Brand campaigns, product data viz |
| Drenched | The surface IS the color | Brand heroes, campaign pages |

Dark vs. light is never a default — "tools look cool dark" and "light to be safe" are both reflexes, not decisions. Write one concrete sentence about who uses this, where, under what light, in what mood; if the sentence doesn't force an answer, it's not specific enough yet.

## Font selection (brand register) — run every time, never skip

1. Write three concrete brand-voice words from the brief — physical-object words, not "modern"/"elegant". E.g. "warm and mechanical and opinionated", not "clean and professional".
2. List the three fonts you'd reach for by reflex. If any appear in the reflex-reject list below, reject them — they're training-data defaults that create monoculture across every AI-assisted project.
3. Browse a real catalog (Google Fonts, Pangram Pangram, Future Fonts, Adobe Fonts, ABC Dinamo, Klim, Velvetyne) holding the three words in mind. Find the font as a *physical object* (a museum caption, a 1970s terminal manual, a concert poster, a receipt from a diner) — reject the first thing that "looks designy".
4. Cross-check: "elegant" isn't necessarily serif, "technical" isn't necessarily sans, "warm" isn't Fraunces. If the final pick matches the original reflex, restart.

**Reflex-reject list** (training-data defaults, look further): Fraunces, Newsreader, Lora, Crimson/Crimson Pro/Crimson Text, Playfair Display, Cormorant/Cormorant Garamond, Syne, IBM Plex Mono/Sans/Serif, Space Mono, Space Grotesk, Inter, DM Sans/Serif Display/Serif Text, Outfit, Plus Jakarta Sans, Instrument Sans/Serif.

**Reflex-reject aesthetic lanes** — currently-saturated families one tier deeper than font choice. A brief lands here without a register reason that genuinely requires it (a literal magazine, a literal terminal) and it's still the AI default: **editorial-typographic** — italic display serif (Fraunces/Recoleta/Newsreader) + small mono labels + ruled separators + monochrome restraint. By 2026 every Stripe-adjacent/Notion-adjacent brand has landed here.

These lists apply to **new design choices** only — if a brand already committed to a font or lane as its identity, identity-preservation wins.

Two families minimum is the rule only when the voice needs it — one well-chosen family with committed weight/size contrast is stronger than a timid display+body pair.

## Imagery is not optional when the brief implies it

A restaurant, hotel, travel, fashion, or photography brief without real imagery reads as incomplete, not restrained — a solid-color rectangle where a hero photo belongs is worse than a representative stock photo. "Imagery" includes product screenshots, data visualizations, generated SVG/canvas/WebGL scenes, not just photography.

- For greenfield work without local assets, use stock imagery (Unsplash is the default: `https://images.unsplash.com/photo-{id}?auto=format&fit=crop&w=1600&q=80`) — verify URLs resolve before referencing them; a guessed photo ID that looks plausible often 404s and ships as a broken image. Prefer fewer photos you've confirmed exist over more you guessed.
- Search for the brand's physical object, not the generic category: "handmade pasta on a scratched wooden table" beats "Italian food".
- One decisive photo beats five mediocre ones — committing to a mood matters more than padding with stock.
- Alt text carries voice too: "Coastal fettuccine, hand-cut, served on the terrace" beats "pasta dish".

## Building a token system

Two layers: primitive tokens (`--blue-500`) and semantic tokens (`--color-primary: var(--blue-500)`). Dark mode only redefines the semantic layer — primitives stay constant.

Palette roles needed for a complete system: primary (1 color, 3-5 shades), neutral (9-11 shade scale), semantic (success/error/warning/info, 2-3 shades each), surface (2-3 elevation levels). Skip secondary/tertiary accent colors unless there's a real need — decision fatigue and visual noise scale with palette size.

Name tokens semantically, not by value: `--text-body` / `--space-md`, never `--font-16` / `--spacing-8`.

## Extracting a design system from existing code

Only extract what's used 3+ times with the same intent — premature abstraction is worse than duplication, and a component generic enough to fit every case is usually useless for all of them. Two components that look similar but serve different purposes should stay separate.

When extracting a component, give it: a clear props API with sensible defaults, the variants actually needed (not speculative ones), accessibility built in (ARIA, keyboard nav, focus management), and a short usage note. When extracting a token, document when to use it, not just its value.

After extraction: find every existing instance of the pattern, migrate it to the shared version, verify visual/functional parity, then delete the old implementation — don't leave both versions live.
