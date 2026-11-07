---
name: rtl-and-bidi
description: Right-to-left by construction for a Hebrew/Arabic/English back-office dashboard on Next.js + Tailwind 4 + lucide - logical classes, which icons mirror, bidi isolation of phones, emails, URLs, codes, money, percentages and time ranges, mixed-direction sentences, and the guard tests. Use when writing any className with a side (left/right, pl/pr, ml/mr, text-left, border-l, rounded-r), placing an arrow, chevron, send, undo, external-link or trend icon, showing a value or a user-typed name inside Hebrew or Arabic text, building tabs, steppers, sliders, charts or a slide-over, or fixing bugs like "+1 555…" showing as "…555 1+", "-12%" as "12%-", or a time range reversed.
---

# RTL and bidi

Hebrew and Arabic are the primary languages; English is the exception. Every screen is
built right-to-left first and checked left-to-right. Nothing is mirrored afterwards by
hand: the page's `dir` does it, through logical classes, the `<Icon>` wrapper and bidi
isolates.

## The page

The i18n provider sets `document.documentElement.dir` (`rtl` for he and ar, `ltr` for en)
and `lang` whenever the language changes. Components read `dir` from the context only
when they must compute something (a slide direction, a key mapping); styling uses
logical classes and the `rtl:`/`ltr:` variants.

## Logical classes only

| Physical (never) | Logical (always) |
|---|---|
| `pl-4 pr-4` | `ps-4 pe-4` |
| `ml-2 mr-auto` | `ms-2 me-auto` |
| `left-0 right-3` | `start-0 end-3` |
| `text-left text-right` | `text-start text-end` |
| `border-l border-r-2` | `border-s border-e-2` |
| `rounded-l-lg rounded-tr-md` | `rounded-s-…` `rounded-se-…` |
| `space-x-2` | `gap-2` |
| CSS `margin-left`, `left:`, `text-align: left` | `margin-inline-start`, `inset-inline-start`, `text-align: start` |

- The only exempt physical classes are the centring pair `left-1/2 -translate-x-1/2`.
- Use `ltr:`/`rtl:` pairs only when a physical side is really meant, and write both.
- A transform that moves along the line flips its sign: a sheet from the end edge comes
  from `x: '100%'` in LTR and `x: '-100%'` in RTL.
- The ratchet's `physical-direction` counter keeps the count at zero (see
  `design-tokens-and-theme`).

## Things that run in reading direction

Day strips and week rows (the first weekday at the start edge), progress bars, sliders,
steppers, breadcrumbs, pagination, carousels, skeleton shimmer (start → end), list
entrances, a notice's countdown bar, chart time axes. Tabs: in RTL **ArrowLeft is the
next tab**. Rings are direction-neutral and start at 12 o'clock.

## Icons

Draw every lucide icon through the kit's `<Icon>`, never raw (`<ChevronLeft />`).

```tsx
export const DIRECTIONAL_ICONS: ReadonlySet<string> = new Set([
  'ChevronLeft', 'ChevronRight', 'ChevronsLeft', 'ChevronsRight', 'ChevronFirst', 'ChevronLast',
  'ArrowLeft', 'ArrowRight', 'ArrowUpLeft', 'ArrowUpRight', 'ArrowDownLeft', 'ArrowDownRight',
  'ArrowBigLeft', 'ArrowBigRight', 'ArrowLeftToLine', 'ArrowRightToLine', 'MoveLeft', 'MoveRight',
  'CornerDownLeft', 'CornerDownRight', 'CornerUpLeft', 'CornerUpRight',
  'Send', 'SendHorizontal', 'Undo', 'Undo2', 'Redo', 'Redo2', 'Reply', 'ReplyAll', 'Forward',
  'LogIn', 'LogOut', 'ListIndentIncrease', 'ListIndentDecrease',
  'PanelLeft', 'PanelRight', 'PanelLeftClose', 'PanelLeftOpen', 'PanelRightClose', 'PanelRightOpen',
  'ExternalLink', 'Quote',
]);

export function Icon({ icon: Glyph, label, mirror, size = 20, strokeWidth = 1.75, className, ...rest }: IconProps) {
  const flips = mirror ?? DIRECTIONAL_ICONS.has(Glyph.displayName ?? '');
  return (
    <Glyph size={size} strokeWidth={strokeWidth}
      className={cn('shrink-0', flips && 'rtl:-scale-x-100', className)}
      aria-hidden={label ? undefined : true} aria-label={label} role={label ? 'img' : undefined}
      focusable="false" {...rest} />
  );
}
```

| Mirrors in RTL | Never mirrors |
|---|---|
| arrows, chevrons, send, undo/redo, reply/forward, log in/out, indents, side panels, external-link, quote | **trend lines** (`TrendingUp`, `TrendingDown`: turned round, a rising line reads as falling), clocks, search, checks, media controls, a QR frame, logos |

- An icon that means a physical side (a "dock to the left" control) passes
  `mirror={false}`, so the decision is written where it is made.
- A sort arrow pointing up or down does not mirror (it is not on the reading axis).
- An icon alone is never a control's only name: the button has an `aria-label`.

## Bidi isolation

In a right-to-left line, neutral characters (`+`, `-`, `%`, `:`, `/`, the dash of a
range) take the line's direction and move. Isolate every value that must read left to
right, and every user-typed name inside a sentence.

| Value | Markup | Reads |
|---|---|---|
| phone number | `<span dir="ltr">+1 555-010-0199</span>` | not `…0199-010 555 1+` |
| email, URL, code, ID | `<span dir="ltr">` | |
| a figure with a sign or unit | `<bdi dir="ltr">-12%</bdi>`, `<bdi dir="ltr">₪1,200</bdi>` | not `12%-`, not `1,200₪` |
| time range | `<span dir="ltr" className="whitespace-nowrap tabular-nums">08:15–09:15</span>` | not `09:15–08:15` |
| a name the user typed, inside a sentence | `<bdi>{name}</bdi>` (direction from its own text) | the sentence around it unchanged |
| a paragraph the user typed | `<p dir="auto">` | |

- **Money:** the currency sign and the minus sit inside the isolate, the minus first:
  `-₪1,200`, never `₪-1,200`, never `(₪1,200)`. Thousands with a comma.
- **Ranges** use the kit dash, an en dash with no spaces: `08:15–09:15`. Put word joiners
  (U+2060) around it in plain text so it never breaks.
- **Numbers never break from their units:** join them with a no-break space (U+00A0):
  `5 min`, `12 items`.
- **Plain strings** (an `aria-label`, a `title`, a notice, `document.title`) cannot hold
  elements: wrap the value in LRI U+2066 … PDI U+2069.
- **Strip bidi marks from `Intl` output** before isolating it: Arabic dates carry U+200F
  between their parts, and inside an LTR isolate they reorder `25/10/2026` into
  `252026/10/`.
- **Never splice a value into a translated string.** Once it is part of the string,
  nothing can isolate it. Render the template with each placeholder in its own `<bdi>`:

```tsx
export function Sentence({ template, values }: { template: string; values: Record<string, React.ReactNode> }) {
  return (
    <span>{/* one span, so a flex parent lays the sentence out as one item */}
      {template.split(/(\{\w+\})/).map((part, n) => {
        const key = /^\{(\w+)\}$/.exec(part)?.[1];
        return key !== undefined && key in values ? <bdi key={n}>{values[key]}</bdi> : <Fragment key={n}>{part}</Fragment>;
      })}
    </span>
  );
}
// <Sentence template={t.items.sentTo} values={{ phone: <span dir="ltr">{phone}</span> }} />
```

## Fields

- Phone, email, URL, tax-id and ID-number fields are `dir="ltr"` in every language (a
  leading `+` otherwise lands after the digits).
- Every other field follows the page. Never force `dir="ltr"` on text that can be Hebrew
  or Arabic.
- Digits typed in Arabic-Indic become 0-9 (see `i18n-and-copy`).

## Mixed-script rows

A Hebrew row that shows an Arabic name (or the reverse) needs nothing special: one font
chain covers both scripts, and the bidi algorithm orders the run. Do not switch fonts per
span, and do not force the name's direction.

## Charts

Time runs from the start edge: in RTL reverse the x-axis and put the y-axis at the start
edge (Recharts: `reversed` on `XAxis`, `orientation="right"` on `YAxis` when
`dir === 'rtl'`). Bars and heatmap columns follow. Tooltips and legends align to start.

## Guard tests

Five guards keep this true. Their shapes are in
[references/rtl-guards.md](references/rtl-guards.md); worked cases are in
[references/bidi-cookbook.md](references/bidi-cookbook.md).

1. `physical-direction` ratchet counter.
2. `ui/icon.test.tsx`: the wrapper mirrors the directional set and leaves trend, clock,
   search and check icons alone; every name in the set exists in lucide.
3. `directional-icons.test.tsx`: a TypeScript-parser scan for directional lucide icons
   drawn raw (any import alias) outside `<Icon>`.
4. `ltr-fields.test.tsx`: every input typed `email`/`tel`/`url`, or bound to a phone,
   email, URL or ID value, carries `dir="ltr"`.
5. Render tests that assert order by isolation: the value sits in a `dir="ltr"` element,
   and the text content reads in the right order in he, ar and en.

## Checklist

- [ ] No physical side classes or CSS; `gap` instead of `space-x`.
- [ ] Every lucide icon goes through `<Icon>`; trend lines, clocks, checks, search unmirrored.
- [ ] Phones, emails, URLs, codes, money, percentages, times and ranges isolated LTR.
- [ ] User-typed names inside sentences are `<bdi>`; templates rendered with `Sentence`, never spliced.
- [ ] LTR fields carry `dir="ltr"`; free-text fields follow the page.
- [ ] Strips, steppers, sliders, tabs keys, sheets and shimmer move in reading direction.
- [ ] Charts reversed in RTL.
- [ ] Looked at in Hebrew first, then Arabic, then English.
