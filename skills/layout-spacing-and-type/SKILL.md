---
name: layout-spacing-and-type
description: Page structure, spacing scale, radii, elevation and type scale for a he/ar/en back-office dashboard on Next.js + Tailwind 4, and how a page holds up at 1280 px and 390 px. Use when laying out a page, the app shell or sidebar, a card grid, a form or a settings section, choosing padding, gap, margin, font size, weight or line height, deciding card versus plain section, or making a table, header or action row work on a phone.
---

# Layout, spacing and type

A back-office page is one shell, one header, and a column of sections. Every gap comes
from one scale, every text size from one set of roles, and every page is checked at 1280
and at 390 px wide, in Hebrew first.

## Spacing scale

| px | Tailwind | Name | Use |
|---|---|---|---|
| 2 | `0.5` | hairline | gaps inside a pill |
| 4 | `1` | xs | label to value |
| 8 | `2` | sm | items in a group, icon to text |
| 12 | `3` | md | inside a card, between cards in a list |
| 16 | `4` | lg | card padding on a phone, card to screen edge |
| 24 | `6` | xl | between sections, page padding on a phone |
| 32 | `8` | xxl | above a section heading, page padding on desktop |
| 48 | `12` | hero | only for a page's opening block |

Use `gap-*` for spacing between siblings. Every padding and margin that has a side is
logical (`ps-`, `pe-`, `ms-`, `me-`); see the `rtl-and-bidi` skill.

## Radii and elevation

- `rounded-badge` (8) small badges and chips inside a pill · `rounded-control` (12)
  inputs, buttons, rows, menus · `rounded-card` (16) cards · `rounded-sheet` (22)
  sheets and dialogs · `rounded-pill` avatars and status pills.
- Nested radii subtract the padding: a 12 image inside a 16 card with 4 padding.
- `shadow-e1` resting card · `shadow-e2` hovered card, menus, sticky bars ·
  `shadow-e3` overlays. In dark the levels are tone and border, not shadow.
- A card never sits on a card. An inset (`bg-surface-2 rounded-control p-3`) inside a
  card is the only nesting.

## Type roles

Define them as Tailwind `--text-*` tokens so `text-body` sets size, weight and line
height together.

| Role | Class | Size | Weight | he/en line height | ar line height | Use |
|---|---|---|---|---|---|---|
| display | `text-display` | 28 | 800 | 1.20 | 1.40 | one per page at most: a report title, a greeting |
| headline | `text-headline` | 22 | 700 | 1.25 | 1.45 | page titles, sheet titles |
| title | `text-title` | 17 | 700 | 1.30 | 1.50 | card titles, dialog titles, section titles with an action |
| section | `text-section` | 14 | 700 | 1.35 | 1.55 | a group label above a list |
| body | `text-body` | 14 | 500 | 1.45 | 1.65 | rows, paragraphs, table cells |
| body-strong | `text-body-strong` | 14 | 600 | 1.45 | 1.65 | the identified thing in a row: a name, an item |
| meta | `text-meta` | 12 | 400 | 1.40 | 1.60 | times, counts, captions, table headers |
| label | `text-label` | 11 | 600 | 1.30 | 1.50 | pills, chips, tab labels: never smaller |
| stat | `text-stat` | 32 | 800 | 1.10 | 1.25 | the one number a card leads with |
| stat-sm | `text-stat-sm` | 24 | 800 | 1.10 | 1.25 | the same in a dense row of tiles |

Rules:

- **Arabic gets taller lines.** Set the Arabic line heights under `:lang(ar)` by
  overriding the `--text-*--line-height` variables. Hebrew/Latin line heights clip
  Arabic diacritics.
- **Never track Hebrew or Arabic.** Letter-spacing breaks Arabic joins. A slight negative
  tracking on display and headline is for `:lang(en)` only.
- **No uppercase.** Hebrew and Arabic have no case. A section label gets weight and
  `text-text-2`, not `uppercase tracking-wide`.
- **Numbers:** `tabular-nums` in columns and while a figure counts; a lone stat keeps
  proportional figures.
- **Text grows; boxes follow.** No fixed height around text. Rows, chips and buttons
  must survive 130 % text, reading views 200 %.
- **One font chain, no language switch.** One `font-family` list that covers Hebrew,
  Arabic and Latin, the Hebrew-and-Latin face first, then the Arabic face, then system
  fallbacks. The first face with the glyph wins, so an Arabic name in a Hebrew row renders
  in the Arabic face without any per-span code. Do not switch the font on `html[lang]`:
  that renders a Hebrew name in the Arabic face on an Arabic page. (The reference panel
  uses Heebo then Cairo, self-hosted with `next/font`.)

## The shell

```tsx
<div className="flex h-screen overflow-hidden">
  {/* the sidebar sits at the start edge: right in Hebrew and Arabic */}
  <aside className="hidden md:flex w-62 shrink-0 flex-col gap-1 border-e border-border bg-surface-1 p-3">
    {/* logo, nav groups; active row: bg-brand-soft text-brand-ink; others text-text-2 hover:bg-surface-2 */}
  </aside>
  <main className="flex-1 overflow-y-auto bg-surface-0">
    <div className="sticky top-0 z-20 border-b border-border bg-surface-1/85 px-6 py-2.5 backdrop-blur md:px-8">{/* top bar */}</div>
    <div className="p-6 pb-24 md:p-8 md:pb-24">{children}</div>
  </main>
</div>
```

- The sidebar is light (`surface-1`) with a `brand-soft` active row and the brand's logo.
  It is not a fixed dark colour that ignores the brand.
- On a phone the sidebar becomes a top bar (`h-14`, logo and name, a menu button at the
  end edge) and a drawer.
- Leave bottom padding (`pb-24`) when a floating action can cover the last row.

## Page anatomy

1. **Page header:** `h1` in `text-headline text-text-1`, an optional `text-body
   text-text-2` description, actions at the end edge. It is a flex row from `sm:` up and
   a column below; the title block is `min-w-0` and the actions `flex-wrap`, so neither
   pushes the other off the screen. `mb-8` below it.
2. **Controls row** when the page has a period, filters or tabs: one row above
   everything it scopes.
3. **Sections**, `gap-6` apart, `mt-8` above a section heading. Each section is a card or
   a plain block.

**Card or plain section?** A card holds one unit the reader treats as a thing: a tile, a
list, a form section, a table. Page-level prose, a header, an empty page's message are
plain. Two cards of the same kind sit in a grid, not stacked inside a third.

```tsx
<section className="rounded-card border border-border bg-surface-1 p-6 shadow-e1">
  <div className="mb-4 flex items-center justify-between gap-3">
    <h2 className="text-title text-text-1">{t.items.title}</h2>
    {action}
  </div>
  …
</section>
```

## Widths: 1280 and 390

Check every page at **1280 px** (desktop with the sidebar) and **390 px** (a phone), in
Hebrew, Arabic and English.

- **Grids follow their container, not the window.** A grid beside a sidebar has less
  room than the window says. Use container queries:
  `<div className="@container"><div className="grid grid-cols-1 gap-5 @lg:grid-cols-2 @4xl:grid-cols-4">`.
- **Phones get one column.** Dashboard cards span 12 below about 700 px.
- **Tables become rows on a phone.** Below `md`, render each record as a list row with
  its two most important facts and a status; the rest is behind a tap. Horizontal scroll
  inside the table card is a fallback for a wide grid that must stay a grid, never the
  plan for a primary list.

```tsx
<div className="hidden md:block"><DataTable … /></div>
<ListRows className="md:hidden">{rows.map((r) => <ListRow key={r.id} title={r.name} meta={[r.when, r.count]} trailing={<StatusPill status={r.status} />} />)}</ListRows>
```

- **Action rows wrap.** A row of buttons beside an identity block: the identity block is
  `min-w-0 flex-1`, the actions `flex flex-wrap gap-2`. The fifth button is never cut off.
- **Nothing has a fixed width** that holds text. Paragraphs cap at `max-w-[70ch]`
  (`max-w-[36ch]` for a short hint under a control).
- **Truncate only with a way to read the rest** (a title or tooltip). Prefer wrapping:
  Hebrew and Arabic strings run longer than English in many places.
- **Targets are 44 px** in the panel (`min-h-11`), with 8 px between adjacent targets.
  Pad an icon button; do not shrink the icon.

## Checklist

- [ ] Gaps and paddings from the scale; logical sides only.
- [ ] Text uses the role classes; no `text-sm`, `text-[13px]`, `font-bold` added to fix one spot.
- [ ] No uppercase or letter-spacing on Hebrew or Arabic; Arabic line heights under `:lang(ar)`.
- [ ] No card inside a card; insets use `surface-2`.
- [ ] Grids use container queries; one column on a phone.
- [ ] Every table has a phone form (rows) or a stated reason it scrolls.
- [ ] Action rows wrap; title blocks are `min-w-0`.
- [ ] Checked at 1280 and 390 in he, ar, en, light and dark, and at 130 % text.
