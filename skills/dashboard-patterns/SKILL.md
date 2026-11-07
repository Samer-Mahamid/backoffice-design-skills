---
name: dashboard-patterns
description: Patterns for the home dashboard and report pages of a he/ar/en back-office (Next.js + Tailwind 4 + lucide) - KPI stat tiles, delta and trend indicators that do not mirror, period and filter rows, the card grid, record cards and history lists, charts in RTL, per-card states, and an empty first-run dashboard that teaches the next step. Use when building a dashboard, home page, KPI or stat card, trend or "vs last month" delta, sparkline, report page, filter bar, period picker, drill-down table, member or record card, activity history, or a dashboard for a brand-new account.
---

# Dashboard patterns

A dashboard answers "how are things, and what needs me?" in one screen. Each card leads
with one number, says what it is compared with, and links to the report behind it. A
card that fails fails alone. A new account's dashboard teaches the first steps instead
of showing zeros.

## Page structure

1. **Page header:** the title, a short greeting or the period's name, actions at the end
   ("Customize", "Export").
2. **Period row:** preset segments (Today, This week, This month, Last month, Last 30
   days, More…) and a custom range. The default is the current month, labelled as running:
   "This month · so far".
3. **Filter row** (reports): chips per dimension with a count badge, a "Clear filters"
   action; the state lives in the URL so a view is linkable.
4. **Card grid** (dashboard) or **KPI row → chart card → drill-down table** (report).

Everything below the period and filter rows recomputes against the same slice. Changing
the period or a filter **holds the current frame at 60 % opacity** until the new answer
arrives; figures then move to their new values. No skeleton flash, no layout jump.

## The card grid

- 12 columns from about 1100 px. Wide cards (today, money, owed) span 6; single-figure
  cards span 3; the rest span 4. Between about 700 and 1100 px, cards span 6 or 12.
  Below 700 px, one column in the same order.
- Use a container query, so the grid follows the room beside the sidebar.
- Card anatomy: the title in `text-section text-text-2`, a drill link at the end
  ("Report" with a chevron that mirrors), the headline in `text-stat`, secondary facts in
  `text-meta text-text-2`.
- Customize mode (optional): lifts cards to `e2`, adds a drag handle, up/down buttons
  and Hide; hidden cards fold into a tray under the grid; "Save layout" stores order and
  hidden set per user (the same on every device). Cancel restores.

## The stat tile

| Part | Rule |
|---|---|
| label | `text-body text-text-2` |
| figure | `text-stat` (or `text-stat-sm` in a dense row), counted up from the number on screen over `count` (900 ms), `tabular-nums` while counting, proportional when settled; jumps under reduced motion |
| unit | inside the figure's LTR isolate: `₪1,200`, `85%`, `-₪300` |
| no figure | "—" with a screen-reader note, **never 0**; hidden by permissions says so to a screen reader |
| loading | a skeleton where the figure goes, never 0 |
| error | a compact error state inside the tile, with Retry |
| delta | percent with its sign, in an LTR isolate; a trend icon; the comparison named ("vs August", "vs the same days last month") |
| subtitle, footnote | `text-meta text-text-2`: what the number counts, and what part of it is something |
| icon well | optional, `rounded-control p-2.5` on the tone's wash |

### Delta and trend

- Colour says whether the move is **good**, not whether it is up: a rise is good unless
  the metric is one where down is good (refunds, debt, no-shows, cancellations). Pass
  `goodWhen: 'down'` for those.
- Good is `text-success`, bad is `text-danger`. The sign and the word carry the meaning
  too: `+12%`, `-4%`.
- The icon is `TrendingUp` or `TrendingDown` and **does not mirror in RTL**: turned
  round, a rising line reads as falling.
- 0 % or no comparison: hide the delta. The delta fades in after the figure lands.

```tsx
<StatTile
  label={t.dash.revenue}
  value={data?.revenue ?? null}
  prefix="₪"
  loading={isLoading}
  error={error}
  onRetry={mutate}
  delta={{ value: data?.revenueDelta ?? null, period: t.dash.vsSameDaysLastMonth }}
  subtitle={sayCount(t.dash.payments, data?.payments ?? 0, t.locale)}
  icon={Wallet}
  tone="brand"
/>
```

Code for the tile, the count-up and the figure formatter is in
[references/stat-tile.md](references/stat-tile.md).

## Per-card states

| State | Card shows |
|---|---|
| first load | its own skeleton, card shape kept |
| error | its own error and Retry; the other cards stay live (**one card failing never fails the page**: the API returns a status per card) |
| empty | an `EmptyState` in place of the chart, with the next action |
| partial | the figures, a `warning` pill in the header with the reason, Retry |
| not allowed | **absent.** What the reader may not see is not rendered: no lock card, no greyed tile; the grid closes the gap. A late 403 shows the error state with "You no longer have access to this report" |
| refetch | held at 60 %, then the values move |

## Charts

- Form first, then colour by job: categorical slots in fixed order (slot 1 the brand),
  sequential for magnitude, diverging around a target, status roles for status (never a
  categorical slot). See `design-tokens-and-theme/references/token-values.md`.
- A comparison period is a dashed line behind the current one.
- **RTL:** time runs from the start edge. Reverse the x-axis and put the y-axis at the
  start edge when `dir === 'rtl'`; legends and tooltips align to start.
- Grid and axis lines `border`, axis text `text-2`, tooltip on `surface-1` at `e3`. Text
  never takes a series colour.
- **Every chart has a table twin** (the drill-down table, or a "Show as table" toggle)
  and a text summary for screen readers ("Revenue this month ₪48,250, up 12 % on
  August"). Keyboard focus shows the same tooltip as hover.
- Small charts on cards: a sparkline (current solid, previous dashed), a stacked bar
  with a legend, a mini heatmap. On a phone a heatmap card becomes text ("Busiest:
  Sunday 18:00") with the heatmap behind a tap.

## Record cards and history lists

- **A record card** (a member, a customer, an item): identity block (avatar with
  initials on `brand-soft` in `brand-ink`, name in `text-body-strong`, facts in
  `text-meta` joined by `·`) and a state pill; its actions wrap beside it.
- **The state drives the card:** one module maps the record's state to the pill's tone,
  a start-edge bar (`border-s-4`, the tone's colour), the history line and which actions
  are offered. Live states speak (`success`, `info`); finished ones are quiet (`neutral`).
- **History lists:** a `ListRows` card, newest first, each row "what happened" in
  `text-body`, "when · who" in `text-meta`, dates through `lib/clock.ts`. Finished records
  read as history: what was used of what it held ("8 of 10 used"), not as active.
- **Upcoming lists:** time (isolated, `tabular-nums`, fixed width) at the start, a colour
  bar, the name, then count/capacity at the end.

## The empty dashboard teaches

A brand-new account has no data. Do not show a wall of zeros and empty charts.

- Show a **first-steps card** at the top: the handful of setup steps in order, a progress
  bar ("2 of 7"), the next step with a one-line hint and its action button, done steps
  with a check and the figure that ticked them. Steps are ticked from real data by the
  server, not by clicks.
- It shows only to the person who can act on it, stays quiet while it loads or if it
  fails, and offers to hide itself only when every step is done.
- Each empty card is an `EmptyState` whose action is that card's first step ("Add the
  first item").
- Charts with no data show the empty state, not empty axes.

## Checklist

- [ ] One period row and one filter row scope everything below them; filters in the URL.
- [ ] Each tile: label, figure (never 0 for unknown), unit isolated, named comparison.
- [ ] Delta colour by good/bad; trend icons not mirrored; sign in the isolate.
- [ ] Each card loads, fails and empties on its own; what the reader may not see is absent.
- [ ] Refetch holds the frame at 60 %; no skeleton flash.
- [ ] Charts reversed in RTL, with a table twin and a text summary.
- [ ] Record cards keyed on one state module; history reads as history.
- [ ] A new account sees first steps and actions, not zeros.
- [ ] Checked at 1280 and 390, he/ar/en, light/dark.
