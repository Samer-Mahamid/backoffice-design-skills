# Bidi cookbook

Each case: what goes wrong in a right-to-left line, and the fix. Examples are written in
logical order; "shows" is what a reader sees.

## A phone number

- Wrong: `<p>{t.codeSentTo} {phone}</p>` → the `+` is neutral and lands at the end:
  shows `…0199-010 555 1+`.
- Right: `<Sentence template={t.codeSentTo} values={{ phone: <span dir="ltr">{phone}</span> }} />`
  with the key written `"נשלח קוד אל {phone}"`.
- Standing alone in a cell: `<span dir="ltr">{phone}</span>`.
- Format it one way everywhere, grouped, with the country code: `+1 555-010-0199`.

## A percentage change

- Wrong: `<span>{delta > 0 ? '+' : ''}{delta}%</span>` → shows `12%-` or `%12+`.
- Right: `<bdi dir="ltr">{delta > 0 ? '+' : ''}{delta}%</bdi>`.

## Money

- Wrong: `₪{-1200}` → `₪-1,200`; or a bare `1,200 ₪` that reorders.
- Right: the minus leads the sign, inside an isolate:

```ts
const NUMBER = new Intl.NumberFormat('en-US');   // Western digits, thousands commas, every language
export const figure = (value: number, prefix = '', suffix = '') => {
  const digits = NUMBER.format(Math.abs(value));
  return `${value < 0 && digits !== '0' ? '-' : ''}${prefix}${digits}${suffix}`;
};
// <bdi dir="ltr">{figure(-1200, '₪')}</bdi>  → -₪1,200
```

No decimals unless the amount has them. Never parentheses for a negative.

## A time range

- Wrong: `` `${start} - ${end}` `` in a Hebrew cell → shows `11:00 - 10:00`; with an
  English browser clock, `AM - 11:00 AM 10:00`.
- Right: `formatClockRange(start, end, language)` from `lib/clock.ts` (24-hour, the kit
  dash U+2013, no spaces) inside `<span dir="ltr" className="whitespace-nowrap tabular-nums">`.
- A date and a time on one line: the date in the page's direction, then ` · `, then the
  isolated range: `יום ראשון, 8/11/2026 · 08:30–09:30`.
- As plain text (a notice, an aria-label): `${day} · \u2066${start}\u2060\u2013\u2060${end}\u2069`.

## A user-typed name in a sentence

- Wrong: `` t.deleteConfirm.replace('{name}', name) `` → a Latin name at the end of a
  Hebrew question drags the `?` to the wrong side.
- Right: `<Sentence template={t.deleteConfirm} values={{ name }} />`: the `<bdi>` takes
  the name's own direction and leaves the sentence alone.
- A person with no name: show their phone (isolated LTR), else a translated "Unknown",
  never an empty hole in the sentence.

## An Intl date in Arabic

- Wrong: `<span dir="ltr">{d.toLocaleDateString('ar-u-nu-latn')}</span>` → Intl puts
  U+200F between the parts; inside the isolate `25/10/2026` shows `252026/10/`.
- Right: strip `[\u200E\u200F\u061C\u202A-\u202E\u2066-\u2069]` from every Intl output
  (do it inside `lib/clock.ts`, once).

## A count and its noun

- Wrong: `` `${n} ${t.items}` `` → Arabic needs a different noun form for 1, 2, 3–10,
  11–99 and 100+; Hebrew has its own for 2 of some words.
- Right: an ICU plural with `{count}` inside each branch (see `i18n-and-copy`), and a
  no-break space between the number and the word.

## An email or URL field

- Wrong: `<Input type="email" />` on an Arabic page → the caret starts at the right edge
  and `@` jumps around while typing.
- Right: `<Input type="email" dir="ltr" />`. The label and hint stay in the page's
  direction.

## A breadcrumb or "Back" link

- Wrong: `<ChevronRight />` hard-coded as "forward" → points backwards in Hebrew.
- Right: `<Icon icon={ChevronRight} />` (mirrors), and write it for LTR: forward is
  `ChevronRight`, back is `ChevronLeft`.

## A trend icon

- Wrong: mirroring `TrendingUp` in RTL → a rise looks like a fall.
- Right: `<Icon icon={TrendingUp} />`: the trend icons are not in the directional set.
  The colour (`success`/`danger`) and the signed figure carry the meaning together.
