# Token values and brand derivation

Starter values for every role. Text, border, status and dark surface values are platform
values that passed the contrast floors in the reference product. Light surfaces and the
brand are slots. After you change any surface, re-run the contrast test: a lighter dark
card or a tinted page can sink a platform ink under AA.

## Surfaces

| Token | Light | Dark | Use |
|---|---|---|---|
| `surface-0` | slot: near-white, may tint toward the brand (reference `#FAF7F8`) | `#121218` | the page |
| `surface-1` | `#FFFFFF` | `#1E1E2E` | cards, sheets, list rows, dialogs |
| `surface-2` | slot: a step darker than the card (reference `#F5EEF1`) | `#2A2A3C` | insets, inputs in a card, chips, table header rows |
| `scrim` | `rgba(26, 26, 46, 0.40)` | `rgba(0, 0, 0, 0.55)` | behind sheets and dialogs |

In light, the inset is darker than the card. In dark, the inset is a step lighter than
the card. If a variant's dark inset is less than 0.04 lighter (HSL) than its card, lift it
to the card plus 0.06, hue and saturation kept.

## Text

| Token | Light | on `surface-1` | Dark | on dark `surface-1` | Use |
|---|---|---|---|---|---|
| `text-1` | `#1A1A2E` | 17.1:1 | `#F1F1F4` | 14.6:1 | names, titles, values |
| `text-2` | `#5C6170` | 6.2:1 (5.8:1 on `surface-0`) | `#A1A5B3` | 6.7:1 | supporting detail, labels, hints, axis text |
| `text-3` | `#9CA3AF` | 2.5:1 | `#6B7280` | 3.4:1 | placeholders and disabled only |

`text-2` is one step darker than the common `#6B7280` so 12 px text passes 4.5:1 on the
page too.

## Lines

| Token | Light | Dark |
|---|---|---|
| `border` | slot near `#E8E2E5` (hairline, decorative) | `#2D2D3D` |
| `border-strong` | `#8A8590` (3.6:1, input and checkbox outlines at rest) | `#6F6F86` (3.5:1) |
| `focus-ring` | `var(--color-brand)`, 2 px, offset 2 px | the same |

## Status

Each has an **ink** (text, icon, solid fill) and a **soft** wash (pills, banners) with the
ink on top. The light inks are the 700 steps, not the 500s: `#10B981` under white text is
2.5:1.

| Token | Light ink | on white | Light soft | Dark ink | on `#1E1E2E` | Dark soft |
|---|---|---|---|---|---|---|
| `success` | `#047857` | 5.5:1 | `rgba(16,185,129,.14)` | `#34D399` | 8.5:1 | `rgba(52,211,153,.18)` |
| `warning` | `#92400E` | 7.1:1 | `rgba(245,158,11,.16)` | `#FBBF24` | 9.8:1 | `rgba(251,191,36,.18)` |
| `danger` | `#B91C1C` | 6.5:1 | `rgba(239,68,68,.12)` | `#F87171` | 5.9:1 | `rgba(248,113,113,.16)` |
| `info` | `#0369A1` | 5.9:1 | `rgba(14,165,233,.14)` | `#38BDF8` | 7.7:1 | `rgba(56,189,248,.18)` |
| `on-semantic` | `#FFFFFF` | | | `#121218` | | |
| `danger-hover` | `#A31919` | | | `#F98282` | | |
| `success-hover` | `#046A4D` | | | `#4CD8A5` | | |

`info` is a real hue of its own. Do not reuse the accent for it: on a red-ish brand the
accent reads as danger.

## Brand slots

| Token | Light | Dark |
|---|---|---|
| `brand` | your primary | primary lightened until ≥ 4.5:1 on dark `surface-1` |
| `on-brand` | `inkOn(brand)`: `#FFFFFF` or near-black, whichever contrasts more | the same rule |
| `brand-hover` | brand mixed 12 % toward black under white ink, toward white under dark ink | the same, from the dark brand |
| `brand-ink` | brand lifted, hue kept, until ≥ 4.5:1 on `surface-0`, `surface-1` and `brand-soft` over either | the same, from the dark brand |
| `brand-soft` | brand at 12 % alpha | brand at 18 % |
| `brand-strong` | primary mixed 28.5 % toward black | the light primary |
| `accent` | yours, or brand rotated +30° | lightened like `brand` |
| `accent-soft` | 12 % | 18 % |

## Elevation

| Level | Light | Dark | Use |
|---|---|---|---|
| `e1` | `0 1px 2px rgba(26,26,46,.06)` + 1 px `border` | no shadow; tone does it | cards, rows |
| `e2` | `0 4px 12px -2px rgba(26,26,46,.10), 0 1px 2px rgba(26,26,46,.06)` | `0 0 0 1px var(--color-border)` | hovered card, menus, sticky action bars |
| `e3` | `0 16px 40px -8px rgba(26,26,46,.18)` | `0 0 0 1px var(--color-border-strong), 0 24px 48px -12px rgba(0,0,0,.6)` | sheets, dialogs, notices, tooltips |

## Charts

Categorical, in this fixed order, never cycled. Slot 1 is your brand stepped into the
band; slots 2–6 stay. Validate the order against your real surfaces (worst adjacent
colour-blind ΔE, normal-vision ΔE, 3:1 against the surface where marks carry meaning).

| Slot | Light | Dark |
|---|---|---|
| 1 | slot (reference `#A8305A`) | slot (reference `#C95C82`) |
| 2 | `#5B5BD6` | `#7878E4` |
| 3 | `#2E9E5B` | `#34A165` |
| 4 | `#9A62C4` | `#A072C4` |
| 5 | `#B87F00` | `#B98A14` |
| 6 | `#0F8FAF` | `#2396B2` |

- Slot 5 sits just above 3:1 on white: a series in it carries a direct label or a table.
- More than six series fold into "Other". Scatter and small multiples cap at three.
- Sequential (magnitude) is one hue, the brand's, seven steps light→dark; an ordinal ramp
  (funnel stages, tiers) starts at a step that clears 2:1 against the surface.
- Diverging (above/below a target) is brand ↔ slot 6 around a neutral midpoint.
- Status in charts uses the status roles, never a categorical slot.
- Grid and axis lines `border`, axis text `text-2`, tooltip on `surface-1` with `e3`.
  Text never wears a series colour.

## Motion

| Token | ms | For |
|---|---|---|
| `fast` | 120 | press feedback, toggles |
| `base` | 200 | colour and state changes, crossfades, every exit |
| `enter` | 300 | an element entering, a sheet opening, a notice arriving |
| `page` | 350 | route transitions |
| `slow` | 600 | a ring or bar filling |
| `count` | 900 | a number counting to its value |
| `shimmer` | 1600 loop | skeleton sweep |

Curves: `standard` `cubic-bezier(0.2,0,0,1)`, `decel` `(0.05,0.7,0.1,1)` for entering,
`accel` `(0.3,0,0.8,0.15)` for leaving, `overshoot` `(0.34,1.56,0.64,1)` only for a success
check. Enters travel 12 px (rows), 16 px (cards), never more than 24 px. Stagger 40 ms per
item, at most 8 items.

## Brand derivation (sketch)

Run this where the brand arrives (an account's settings, a profile page) and set the result on
`document.documentElement.style`. With no brand, set nothing: the stylesheet defaults are
the tokens.

```ts
type RGB = [number, number, number];
const hex = (h: string): RGB => [1, 3, 5].map((i) => parseInt(h.slice(i, i + 2), 16)) as RGB;
const toHex = (c: RGB) => '#' + c.map((v) => Math.round(v).toString(16).padStart(2, '0')).join('').toUpperCase();
const lum = (h: string) => {
  const [r, g, b] = hex(h).map((v) => { v /= 255; return v <= 0.03928 ? v / 12.92 : ((v + 0.055) / 1.055) ** 2.4; });
  return 0.2126 * r + 0.7152 * g + 0.0722 * b;
};
export const contrast = (a: string, b: string) => {
  const [x, y] = [lum(a), lum(b)].sort((p, q) => q - p);
  return (x + 0.05) / (y + 0.05);
};
export const mix = (a: string, b: string, t: number) => toHex(hex(a).map((v, i) => v + (hex(b)[i] - v) * t) as RGB);
export const inkOn = (fill: string) => (contrast(fill, '#FFFFFF') >= contrast(fill, '#141019') ? '#FFFFFF' : '#141019');

/** Move [ink] toward white or black, whichever the ground needs, until it reads [ratio] on it. */
export function legibleOn(ink: string, ground: string, ratio = 4.5): string {
  const target = lum(ground) > 0.18 ? '#000000' : '#FFFFFF';
  for (let t = 0; t <= 1; t += 0.02) {
    const c = mix(ink, target, t);
    if (contrast(c, ground) >= ratio) return c;
  }
  return target;
}

/** Text in the brand hue: lifted only as far as the page, a card and the brand wash over each need. */
export function brandInk(brand: string, grounds: string[]): string {
  let ink = brand;
  for (const g of grounds) if (contrast(ink, g) < 4.5) ink = legibleOn(ink, g);
  return ink;
}

export function brandHover(brand: string, onBrand: string) {
  return mix(brand, lum(onBrand) > 0.5 ? '#000000' : '#FFFFFF', 0.12);
}
```

Mixing toward black or white shifts saturation a little; a hue-preserving lift (HSL
lightness steps) is closer to "hue kept". Either way, test the result against the floors,
not by eye. Pin the derivation with tests: a light brand (gold), a dark brand (navy), and a
mid brand, in light and dark, each checked for `on-brand` ≥ 4.5:1 on `brand` and on
`brand-hover`, and `brand-ink` ≥ 4.5:1 on all four grounds.
