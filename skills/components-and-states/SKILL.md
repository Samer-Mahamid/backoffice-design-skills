---
name: components-and-states
description: The shared component kit of a he/ar/en back-office dashboard (Next.js + Tailwind 4 + lucide) and the four states every screen has. Use when building or reviewing a button, input, select, chip, segmented tabs, status pill or badge, stat tile, data table, list row or card, or when a page or card loads data and needs its loading skeleton, empty state with a next action, error state with Retry, and data state (spinner, "No data", "Something went wrong", useSWR, AsyncView).
---

# Components and states

Screens are built from one kit in `components/ui/`. A screen never restyles a kit
component's look to solve a local problem; if the kit is wrong, the kit changes, for
every screen at once.

## The kit

| Component | Look | Rules |
|---|---|---|
| `Button` | `rounded-control text-body-strong h-10 px-4`, focus ring | variants below; one primary per view and per overlay; `loading` keeps the label and shows an inline spinner in place of the icon |
| `Input`, `Textarea` | `h-10 rounded-control border border-border-strong bg-surface-1 text-body`, `placeholder:text-text-3` | `error` turns the outline and the focus ring `danger`; leading icon at `start-3` with `ps-10` |
| `Select`, `DateField`, `NumberField` | as `Input` | number fields convert typed Arabic-Indic digits to 0-9 |
| Chips / `FilterChipRow` | `rounded-pill` | selected: `bg-brand-soft text-brand-ink`; a count badge; "Clear" ends the row |
| `Tabs` (segmented) | list `inline-flex gap-1 rounded-control bg-surface-2 p-1`; selected tab `bg-surface-1 shadow-e1 text-text-1`, others `text-text-2` | WAI-ARIA tabs; arrow keys move in reading direction |
| `StatusPill` | `inline-flex gap-1 rounded-pill px-2 py-0.5 text-label`, wash + ink | tone + icon (or dot) + word, never colour alone; never interactive |
| `StatTile` | card, label `text-body text-text-2`, figure `text-stat` | see the `dashboard-patterns` skill |
| `DataTable` | card frame; header `bg-surface-2 text-meta font-semibold text-text-2 text-start`; rows `text-body`, `hover:bg-surface-2`, `divide-y divide-border` | skeleton rows; `rowOpens`; paging chevrons through `<Icon>` |
| `ListRows` / `ListRow` | a card of rows, hairline between rows inset from the start edge | leading, title over meta facts joined by a quiet `·`, trailing |
| `Card` | `rounded-card border border-border bg-surface-1 shadow-e1`; `hover` lifts to `shadow-e2` | no card on a card |
| `EmptyState`, `ErrorState`, `Skeleton`, `AsyncView` | see "The four states" | |
| `Icon` | lucide at stroke 1.75, 20 px | mirrors directional icons in RTL; see `rtl-and-bidi` |

Full code for the core pieces is in [references/kit-snippets.md](references/kit-snippets.md).

### Buttons

| Variant | Classes | For |
|---|---|---|
| `default` (brand) | `bg-brand text-on-brand shadow-e1 hover:bg-brand-hover` | the one primary action |
| `destructive` | `bg-danger text-on-semantic shadow-e1 hover:bg-danger-hover` | a destructive primary, named by its verb |
| `outline` | `border border-border bg-surface-1 text-text-1 shadow-e1 hover:bg-surface-2` | secondary actions on a page |
| `secondary` | `bg-surface-2 text-text-1 hover:bg-border` | quiet secondary |
| `ghost` | `text-text-2 hover:bg-surface-2 hover:text-text-1` | Cancel, toolbar actions |
| `link` | `text-brand-ink underline-offset-4 hover:underline` | inline navigation |

All share: `focus-visible:outline-2 focus-visible:outline-offset-2
focus-visible:outline-focus-ring`, `disabled:opacity-50 disabled:pointer-events-none`,
`active:scale-[0.98] motion-reduce:active:scale-100`. Sizes: `default h-10`, `sm h-8`,
`lg h-12`, `icon size-10`. An icon-only button has an `aria-label`.

### Fields

- The label sits above the field (`text-body-strong text-text-1`), the hint under it
  in `text-meta text-text-2`. Placeholder text is never the only label and never the
  hint: `text-3` cannot carry information.
- An error replaces the hint: `text-meta text-danger` with an icon, tied by
  `aria-describedby`, and the field gets `error`. On submit, focus moves to the first
  invalid field.
- Phone, email, URL, tax-id and ID fields are `dir="ltr"` in every language.
- Digits typed in Arabic-Indic (٠–٩) or extended (۰–۹) become 0-9 as they arrive.

### Data tables

- `rowOpens(row)` decides which rows open something. Only those rows take
  `cursor-pointer` and the click, and carry `group/row` so a cell can underline its name
  on row hover (`group-hover/row:underline`). A control inside a row stops propagation.
- Header cells: no uppercase, no tracking, `text-start`. Sort icons are arrows inside the
  header (`ArrowUp`, `ArrowDown`, a faded `ArrowUpDown`).
- Loading: five skeleton rows in the real columns, never a spinner.
- Empty: a line of words, or a whole `EmptyState` / `ErrorState` (a failed load with
  Retry) in the body.
- The footer counts with a plural ("1 item", "2 items", Arabic's six forms) and pages
  with chevrons that mirror in RTL.
- Below `md`, a primary list renders as `ListRow`s (see `layout-spacing-and-type`).

## The four states

Every screen and every card that loads data has four states, designed and tested:

| State | Shows | Never |
|---|---|---|
| **Loading** | a `Skeleton` in the shape of what is coming (lines, a list, a card, table rows); "loading" said once to screen readers | a spinner where content will be; a `0` where a figure will be |
| **Empty** | `EmptyState`: an icon, a title saying there is nothing yet, a `text-2` line saying what to do, and the **action** that does it | a bare "No data"; an empty table with headers and no explanation |
| **Error** | `ErrorState`: what happened in honest words, and **Retry** where retrying could help | raw error text, a stack, "Error 500", a red line with no way out |
| **Data** | the content | |

When the request changes (another period, another filter), keep the last answer on
screen at 60 % opacity until the new one arrives: no skeleton flash, no layout jump.

### Error kinds

| Kind | When | Icon well | Retry |
|---|---|---|---|
| `offline` | no response reached the browser | `bg-warning-soft text-warning`, `WifiOff` | yes |
| `failed` | a 5xx, a timeout, anything else | `bg-danger-soft text-danger`, `TriangleAlert` | yes |
| `refused` | 403 | `bg-surface-2 text-text-2`, `Lock` | no: it will not change on retry |
| `notFound` | 404 | `bg-surface-2 text-text-2`, `SearchX` | no |

A 401 is an expired session that the sign-in flow renews, not an error state. A failed
load may carry its own title ("The items couldn't load"); offline, refused and not
found keep the kit's words. When the server sends a refusal reason, show that reason in
the reader's language (map its error code to a translation key).

### One component for it

```tsx
<AsyncView
  state={useSWR(key, fetcher)}
  skeleton={<Skeleton variant="list" count={5} />}
  isEmpty={(rows) => rows.length === 0}
  empty={<EmptyState icon={Inbox} title={t.items.emptyTitle} description={t.items.emptyHint}
           action={<Button onClick={openAdd}>{t.items.add}</Button>} />}
  errorTitle={t.items.loadFailed}
>
  {(rows) => <ItemsTable rows={rows} />}
</AsyncView>
```

An unknown figure shows "—" (with a screen-reader note), never 0. A figure hidden by the
reader's permissions says so to a screen reader too.

## One shared rule per concept

When a record has a state that drives several things on screen (the pill's tone, the
card's edge, the history line, which actions are offered), compute the state **once**,
preferably on the server, and key everything off it in one module:

```ts
// lib/record-state.ts
export type RecordState = 'active' | 'paused' | 'scheduled' | 'ended' | 'cancelled';
export const RECORD_STATE_TONE: Record<RecordState, Tone> = {
  active: 'success', paused: 'info', scheduled: 'info', ended: 'neutral', cancelled: 'neutral',
};
export const recordStateEdge = (s: RecordState) =>
  s === 'active' ? 'border-s-success' : s === 'paused' || s === 'scheduled' ? 'border-s-info' : 'border-s-border-strong';
export const RECORD_STATE_ACTIONS: Record<RecordState, { edit: boolean; pause: boolean; cancel: boolean }> = { /* the owner's table */ };
```

The card never re-derives the state from a stored status that can outlive it. The table
of actions per state is written down once and tested.

## A showcase route

Keep a page (for example `/dashboard/design`, built only when a flag is set, answering
404 otherwise) that renders every kit component in every state in {light, dark} × {he,
ar, en}. It is the review surface for before/after screenshots and the target of
snapshot tests.

## Checklist

- [ ] Built from `components/ui/*`; no local restyle of a kit component.
- [ ] One primary button per view and per overlay.
- [ ] Every loading place has a skeleton of the right shape; no spinner except inside a busy button.
- [ ] Empty state names the next action and offers it.
- [ ] Error state has honest words and Retry only where retrying helps.
- [ ] Unknown figures show "—", never 0.
- [ ] Status pills have tone, icon or dot, and a word.
- [ ] Table rows that open something are the only clickable rows; row controls stop propagation.
- [ ] States tested in he → ar → en, with long Hebrew and Arabic text.
