# Review checklist and guard inventory

Use the checklist on every UI pull request. Use the inventory when setting up a project,
and add a guard whenever a review finds a rule broken twice.

## Screens (design review)

**Look**
- [ ] Matches the approved mockup frame by frame; any departure is explained.
- [ ] Looks like the rest of the product: same header, same card, same table, same pill.
- [ ] One primary action per view and per overlay.
- [ ] Status said by tone + icon/dot + word; the brand colour means nothing.
- [ ] Nothing a person must read is in the faint grey (`text-3`).

**Direction**
- [ ] Hebrew and Arabic frames read right to left: sidebar on the right, actions at the
      left end, chevrons pointing the right way, trend arrows not flipped.
- [ ] Phones, emails, codes, money, percentages, times and ranges read correctly inside
      Hebrew and Arabic text.
- [ ] Day strips, steppers, progress and tabs move in reading direction.

**Fit**
- [ ] 390 px: one column, no horizontal page scroll, tables as rows, action rows wrap,
      nothing cut off.
- [ ] 1280 px: grids use the room beside the sidebar; nothing stretched to absurd widths.
- [ ] Long Hebrew and Arabic strings wrap; nothing truncates without a way to read it.
- [ ] 130 % text: rows and buttons grow, nothing overlaps.

**States**
- [ ] Loading skeleton in the shape of the content; no spinner; no `0` before the figure.
- [ ] Empty state says what to do and offers it.
- [ ] Error state honest, with Retry where it helps.
- [ ] Dark mode looked at, not assumed.

**Words**
- [ ] Every string translated in he, ar, en; no English on a Hebrew screen.
- [ ] Hebrew and Arabic impersonal; no masculine imperative; addressing via `select`.
- [ ] Counts agree with their nouns; Western digits; 24-hour times; the kit dash in ranges.
- [ ] Sentence case in English; the glossary term; destructive buttons named by their verb.

**Overlays and notices**
- [ ] Input → sheet; plain confirm → dialog; secondary at the start, primary at the end.
- [ ] Notices: right kind, right duration, failures announced.

**Motion and access**
- [ ] Reduced motion: no movement, short fades only.
- [ ] Focus ring visible on every control; tab order sensible; Escape closes the top overlay.
- [ ] Icon-only buttons labelled; targets 44 px.
- [ ] Charts have a table twin and a text summary.

## Code (code review)

- [ ] Built from `components/ui/*`; no local restyle of a kit component; no hand-rolled overlay.
- [ ] Token classes only: no hex, palette class, `bg-white`, Tailwind radius step in the kit, raw duration.
- [ ] Logical classes only; `gap` not `space-x`; every lucide icon through `<Icon>`.
- [ ] Values isolated with `dir="ltr"` / `<bdi>`; templates rendered with `Sentence`, never spliced.
- [ ] Keys added to all three languages as one block at the end; no `||` fallback.
- [ ] Dates and times only through `lib/clock.ts`; numbers through the shared formatter.
- [ ] One module per concept; no second derivation of a state the server sends.
- [ ] Tests for every state, in he → ar → en; a test per new rule, mutation-checked.
- [ ] Fixtures shaped like what the API really sends, not like what the screen wants.
- [ ] Ratchet not raised; lowered if it fell.

## Guard inventory

| Guard | Holds | Lives in |
|---|---|---|
| `ratchet.test.ts` + baseline + `ratchet-lower.sh` | no hex, palette classes, English fallbacks, physical directions, motion literals, Tailwind radii in the kit; counts only fall | `design-tokens-and-theme/references/ratchet.md` |
| token contrast test | every text and ink role meets its floor on its grounds, light and dark, for sample brands | `design-tokens-and-theme/references/token-values.md` |
| `ui/icon.test.tsx` | the wrapper mirrors exactly the directional set; trend, clock, search, check never | `rtl-and-bidi/references/rtl-guards.md` |
| `directional-icons.test.tsx` | no directional lucide icon drawn raw, under any alias | same |
| `ltr-fields.test.tsx` | phone, email, URL and ID fields are `dir="ltr"` | same |
| isolation render tests | ranges, phones, signed figures render inside an LTR isolate | same |
| `translation-parity.test.ts` | the three languages have the same keys; nothing left untranslated | `i18n-and-copy/references/i18n-guards.md` |
| `translation-keys-exist.test.ts` | every key the code reads exists | same |
| gender copy test | no masculine imperatives or addressing words in Hebrew; branches differ; wiring end to end | same |
| `western-digits.test.tsx` | typed digits convert; formatted output has no Arabic-Indic digits | same |
| `clock.test.tsx` | 24-hour, kit dash, Latin digits; no time formatting outside `lib/clock.ts` | same |
| overlay source scan | no `fixed inset-0` outside the kit's overlay files | `overlays-and-notices` |
| notices test | durations per kind, bar only on warning/failure/undo, failures assertive, reduced motion | `overlays-and-notices/references/notices-implementation.md` |
| state tests per screen | loading, empty, error with Retry, data, long text | `components-and-states` |
| showcase test | every kit component renders in every state without errors | `components-and-states` |
