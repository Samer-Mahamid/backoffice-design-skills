---
name: design-tokens-and-theme
description: Colour, radius, shadow and motion tokens for a he/ar/en back-office dashboard on Next.js + Tailwind 4, named by role, designed for light and dark, with the brand as slots the project fills. Use when setting up globals.css or the Tailwind @theme block, adding dark mode, theming a brand colour, choosing a text, background, border or status colour, or reviewing code for hex values, Tailwind palette classes (bg-gray-100, text-red-600, bg-white), raw durations, or light grey text that carries information.
---

# Design tokens and theme

Every colour, radius, shadow and duration in a screen is a **role**, never a value. The
brand is the only colour that changes between products; surfaces, text, status colours,
spacing, type and motion are fixed platform design. Two brands should look like the same
product in two liveries.

## Principles

1. **Roles, not values.** Code names `surface-2`, `text-2`, `danger-soft`. A hex or a
   palette step in a component is a bug, and a ratchet test counts it.
2. **The brand never means anything.** Status is `success`, `warning`, `danger`, `info`.
   Brand is for identity and the primary action.
3. **State has a shape, not just a colour.** A pill carries a wash, an ink, an icon or dot,
   and a word. Colour alone never says a status.
4. **Dark is designed, not inverted.** Each role has its own dark value. Dark surfaces are
   separated by tone and borders, not shadows.

## The roles

| Group | Roles | Tailwind classes |
|---|---|---|
| Surfaces | `surface-0` page, `surface-1` cards, sheets, rows, `surface-2` insets, inputs at rest in a card, chips, table header rows | `bg-surface-0/1/2` |
| Text | `text-1` names, titles, values · `text-2` supporting detail, labels, hints, axis text · `text-3` placeholders and disabled only | `text-text-1/2/3` |
| Lines | `border` hairline (decorative) · `border-strong` input and checkbox outlines at rest (3:1) · `focus-ring` | `border-border`, `border-border-strong`, `outline-focus-ring` |
| Brand | `brand` fill · `on-brand` ink on it · `brand-hover` hover/pressed fill · `brand-ink` text and icons in the brand hue · `brand-soft` selected wash · `brand-strong` deep surfaces only · `accent`, `accent-soft` | `bg-brand text-on-brand hover:bg-brand-hover`, `text-brand-ink`, `bg-brand-soft` |
| Status | `success`, `warning`, `danger`, `info` (ink) and `*-soft` (wash), `on-semantic`, `danger-hover`, `success-hover` | `bg-success-soft text-success`, `bg-danger text-on-semantic` |
| Overlay | `scrim` | `bg-scrim` |
| Charts | `chart-1..6` categorical, `seq-100..700` sequential, `div-neg/mid/pos` diverging, `grid`, `axis` | `fill-chart-1`, `stroke-grid` |
| Radii | `badge` 8, `control` 12, `card` 16, `sheet` 22, `pill` 999 | `rounded-badge/control/card/sheet/pill` |
| Elevation | `e1` resting, `e2` raised, `e3` overlay | `shadow-e1/e2/e3` |
| Motion | `fast` 120, `base` 200, `enter` 300, `page` 350, `slow` 600, `count` 900 ms; `standard`, `decel`, `accel`, `overshoot` curves | `duration-(--duration-base) ease-standard` |

Starter values for every role, light and dark, with their contrast ratios, are in
[references/token-values.md](references/token-values.md).

## Fill your brand

The project supplies a few slots. Everything else is derived or fixed.

| Slot | You supply | Derived from it |
|---|---|---|
| `brand` (light) | your primary colour | `on-brand` (near-black or white, whichever contrasts more), `brand-hover` (12 % away from the ink), `brand-soft` (12 % wash), `brand-strong` (about 28 % toward black), `brand-ink` (lifted, hue kept, to 4.5:1 on the page, a card and its own wash) |
| `brand` (dark) | nothing, or a hand-tuned value | the primary lightened until 4.5:1 on dark `surface-1`; wash at 18 %; `brand-strong` is the light primary |
| `accent` | optional second colour | if unset, the brand rotated +30° on the hue circle |
| `surface-0/1/2` | optional faint tint toward your brand | must keep text-1 and text-2 at 4.5:1; light insets darker than cards, dark insets a step lighter |
| `chart-1` | your brand, stepped to sit with slots 2–6 | slots 2–6 stay fixed; re-validate the palette against your surfaces |
| Logo, font faces | yours | the font chain must cover Hebrew and Arabic glyphs (see `layout-spacing-and-type`) |

Rules for the brand:

- A brand that fails 4.5:1 against the page, a card or its own wash is **never used as
  ink**. Text and icons in the brand hue are always `text-brand-ink`. Fills keep the true
  hue and take `on-brand` ink.
- `brand-soft` is the only tinted background a screen may make from the brand. No screen
  mixes its own alpha (`bg-brand/10` is a smell).
- `brand-strong` is never a fill under `on-brand`. Hover and pressed use `brand-hover`.
- The brand is set on `<html>` as CSS variables at runtime when it comes from data (an
  account's settings), for light and dark both. The derivation code is in the reference.

## Tailwind 4 setup

Tokens live in one generated block in `app/globals.css`, from a TypeScript source
(`lib/tokens.ts`) that tests can import. Use `@theme static` so every variable is emitted.

```css
@import "tailwindcss";

/* dark: follows <html data-theme> when set, the system preference when not */
@custom-variant dark {
  @media (prefers-color-scheme: dark) {
    &:where(:root:not([data-theme="light"]), :root:not([data-theme="light"]) *) { @slot; }
  }
  &:where(:root[data-theme="dark"], :root[data-theme="dark"] *) { @slot; }
}

/* @generated tokens: start (edit lib/tokens.ts, not this) */
@theme static {
  --color-surface-0: #F8F8FA;   /* slot: may tint toward the brand */
  --color-text-2: #5C6170;
  --color-brand: #2F5BEA;       /* slot */
  --radius-card: 16px;
  --shadow-e1: 0 1px 2px rgba(26, 26, 46, 0.06);
  --duration-base: 200ms;
  --ease-standard: cubic-bezier(0.2, 0, 0, 1);
  /* …every role */
}
/* @generated tokens: end */

@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) { --color-surface-0: #121218; /* …every role */ color-scheme: dark; }
}
:root[data-theme="dark"] { --color-surface-0: #121218; /* the same block */ color-scheme: dark; }
```

Apply a saved theme with a tiny inline script in `app/layout.tsx` before first paint, so
the page never flashes the wrong theme.

## Do and don't

| Do | Don't |
|---|---|
| `bg-surface-1 text-text-1 border-border` | `bg-white text-gray-900 border-gray-200` |
| `text-text-2` for a hint, caption, label, empty-state line | `text-text-3` or `text-gray-400` for anything a person must read |
| `text-text-3` for `placeholder:` and disabled text only | a "quiet" status or date in `text-text-3` |
| `bg-danger-soft text-danger` pill with an icon and a word | a red dot as the only sign of a problem |
| `bg-success text-on-semantic` for a solid success button | `bg-emerald-500 text-white` (2.5:1, fails) |
| `text-brand-ink` for a selected chip's label | `text-brand` as text on a light page |
| `shadow-e1` on a resting card, `shadow-e2` on hover | `shadow-md`, `shadow-[0_4px_…]` |
| `duration-(--duration-base) ease-standard` | `duration-300`, `transition-all`, `animate-bounce` |
| `rounded-card` on cards, `rounded-control` on inputs and buttons | `rounded-lg`, `rounded-xl` in a shared component |
| an inset `bg-surface-2` block inside a card | a card inside a card |

Contrast floors the values must keep (WCAG 2.1 AA): text 4.5:1 (`text-1`, `text-2`,
every semantic ink, also on its own wash); large text and meaningful graphics 3:1;
`border-strong` 3:1; the hairline `border` is decorative only; `text-3` never carries
information (1.4.3).

## Motion tokens in one line each

Exits are faster than enters (`base` vs `enter`). Nothing user-facing runs longer than
`page` except a value filling (`slow`) or counting (`count`). Under
`prefers-reduced-motion`, movement and scale stop and only short fades (≤ 150 ms) stay.
Durations and easings live in one module (`lib/motion.ts`); components read them from
there or from the CSS variables, never as literals.

## Guard it

A ratchet test counts what code must stop doing and only lets the count fall:
hex literals outside the token block, palette classes, English fallbacks, physical
directions, motion literals, Tailwind radius steps in `components/ui/`. The pattern, the
regexes and the lowering script are in [references/ratchet.md](references/ratchet.md).
A token test should also check every contrast floor above for the default brand and for
a light and a dark sample brand.

## Checklist

- [ ] No hex, `rgb(`, palette class, `bg-white`, `text-black` in a component.
- [ ] Every text colour is `text-1`, `text-2`, a semantic ink or `brand-ink`; `text-3` only on placeholders and disabled.
- [ ] Status shown with tone + icon/dot + word, never colour alone; brand never used as a status.
- [ ] Brand text uses `brand-ink`; brand fills hover to `brand-hover`.
- [ ] Every new role has a light and a dark value, and the dark one was looked at.
- [ ] Radii, shadows and durations by role; reduced motion respected.
- [ ] The ratchet count did not rise; if it fell, the baseline was lowered in the same change.
