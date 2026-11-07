---
name: i18n-and-copy
description: Hebrew, Arabic and English translations and the copy rules of a back-office dashboard on Next.js - one translations file with three equal languages, no silent English fallback, ICU plurals and gender, impersonal Hebrew and Arabic, Western digits everywhere, dates and times through one module, sentence case, and words named from the user's side. Use when adding or changing any user-facing string, translation key, label, button text, error or empty message, a count with a noun, gendered Hebrew or Arabic, a date, time, number or digit format, or reviewing for `t.x || 'English'` fallbacks, toLocaleString/Intl calls, Arabic-Indic digits, or masculine imperatives.
---

# i18n and copy

Three languages, equal: Hebrew (default), Arabic, English. Every string a person reads
comes from the translations file, exists in all three, and was written for each
language, not machine-shaped from English.

## Structure

```ts
// lib/translations.ts
export type Language = 'he' | 'ar' | 'en';
export const translations = {
  he: { locale: 'he', common: { save: 'שמירה', cancel: 'ביטול' }, items: { /* … */ } },
  ar: { locale: 'ar', common: { save: 'حفظ', cancel: 'إلغاء' }, items: { /* … */ } },
  en: { locale: 'en', common: { save: 'Save', cancel: 'Cancel' }, items: { /* … */ } },
};
```

- `useI18n()` returns `{ t, language, setLanguage, dir }`, with `t` typed as
  `typeof translations.he`. The provider sets `<html dir lang>`; the first render is the
  default language (the server's), then the saved choice applies. Only the person's own
  choice is saved.
- Add new keys as **one block at the end of each language**, the same keys in the same
  order in all three. That keeps merges clean.
- **No fallback in code.** `t.items.title || 'Items'` shows English on a Hebrew page the
  day the key goes missing, and nothing says so. A missing key must fail a test instead.
- Kit components take their default words (Cancel, Retry, Loading, page labels) from the
  translations too, never English literals.
- Translate every label a library renders (toast close button, region names, date picker
  months).

Guards (shapes in [references/i18n-guards.md](references/i18n-guards.md)):
**parity** (the three languages have the same keys, both ways), **keys exist** (every
`t.a.b` in the code is a real key), **untranslated** (no he/ar value equal to its English
one except an allow-list of names), the **english-fallbacks** ratchet counter.

## Plurals

Hebrew and Arabic make a counted noun agree with its number. Arabic has forms for 1, 2,
3–10, 11–99 and 100+ (by the last two digits, so 103 is plural again). Never splice a
number in front of a word chosen by an `if`.

```ts
// he: '{count, plural, =1{פריט אחד} =2{שני פריטים} other{{count} פריטים}}'
// ar: '{count, plural, =0{لا عناصر} =1{عنصر واحد} =2{عنصران} few{{count} عناصر} many{{count} عنصرًا} other{{count} عنصر}}'
// en: '{count, plural, =1{1 item} other{{count} items}}'
sayCount(t.items.count, n, t.locale);
```

The resolver uses `Intl.PluralRules`, picks an exact `=n` first, then the category, then
`other`. A count that did not arrive is not zero: the resolver returns `undefined` and the
caller shows "—". Code in [references/icu-resolvers.md](references/icu-resolvers.md).

## Gender: impersonal first

Hebrew and Arabic conjugate the person addressed. The panel's voice avoids it:

1. **Labels and buttons are verbal nouns**, which have no gender: he `שמירה`, `מחיקה`,
   `הוספת פריט`; ar `حفظ`, `حذف`, `إضافة عنصر`. Never the masculine imperative (`שמור`,
   `מחק`, `בחר`, `הזן`, `לחץ`).
2. **Sentences are impersonal:** he `יש לבחור תאריך`, `אפשר לנסות שוב`; ar `يجب اختيار
   تاريخ`, `يمكن المحاولة مرة أخرى`; or the neutral plural.
3. **Only when a sentence must address the reader** (a confirm question, a greeting),
   write it as an ICU `select` on the reader's **own** gender, with `other` in the neutral
   plural, and `other` is the default when the gender is unknown:
   `{gender, select, female{…} male{…} other{…}}`. A sentence about a third person
   selects on **that** person's gender.
4. **Never a bare masculine form**, and never a sentence built by concatenation: the
   whole sentence is one message, so the translator can move the verb.
5. English keeps one form, but the key exists in all three files with the same shape.

**Before a user-named thing**, write a construction that agrees with nothing. Users name
items freely, in either gender, often with their own kind-word inside the name. A template `כבר קיים {item}` reads wrong with a feminine name, and
`כבר קיים פריט {item}` doubles the noun. Use the existential: he `כבר יש {item} בתאריך
הזה`, ar `هناك بالفعل {item} في هذا التاريخ`. No generic noun and no agreeing verb or
adjective directly before the placeholder. Test every such string with one masculine and
one feminine name before it ships.

Guard: a test walks every Hebrew string (gender groups resolved) and fails on listed
masculine imperatives and addressing words, matched as whole words, and checks that the
female and male resolutions differ somewhere (so a "conversion" that gave all branches the
masculine text fails). Word lists match words, not grammar: an adjective can collide with
an imperative; reword it.

## Digits: Western everywhere

The same 0-9 in Hebrew, Arabic and English, shown and typed. Never Arabic-Indic (٠–٩).

- **Shown:** format numbers with `new Intl.NumberFormat('en-US')`, and dates and times
  with the locale `ar-u-nu-latn` for Arabic (`he`, `en` otherwise).
- **Typed:** every text, search, tel, number, url and email field converts Arabic-Indic
  (U+0660–0669) and extended (U+06F0–06F9) digits to 0-9 as they arrive, and the Arabic
  decimal separator (U+066B) to a point; a password is left as typed. Install it once
  for the whole app (a `beforeinput`/`input` listener), not per field.
- Guard: render Arabic screens and formatters and assert no U+0660–0669/U+06F0–06F9 in
  the output.

## Dates and times: one module

All date and time formatting lives in `lib/clock.ts`. A test parses the code and fails
on any `toLocaleTimeString`, `toLocaleString`/`toLocaleDateString` with an hour or no
options on a date, or `new Intl.DateTimeFormat` with an hour, outside that file.

- The locale comes from the app's language, never the browser's (`toLocaleTimeString([])`
  is the browser's: an English browser puts "01:30 PM" in the Hebrew page).
- One clock: 24-hour, `hourCycle: 'h23'` (not `hour12: false`, which some engines answer
  with "24:05").
- Ranges with the kit dash, no spaces: `RANGE_DASH = '\u2013'` → `08:15–09:15`.
- A calendar day sent as `YYYY-MM-DD` is built from its parts, not parsed:
  `new Date('2026-10-04')` is midnight UTC, the day before west of Greenwich.
- Strip bidi marks from every `Intl` output; use the Arabic comma `،` (U+060C) in Arabic
  lists of date parts.
- One date format per job (a heading, a cell, a short label), the same in every place.
- One week start for the product (the reference uses Sunday), and day strips run from it
  at the start edge.

## Words

- **Sentence case in English:** "Add item", "Item types", never "Add Item".
- **One term per concept**, agreed with the owner in all three languages and written into
  every string. Code identifiers, routes and API fields may keep older names; only the
  words change. Keep a short glossary.
- **Named from the user's side:** call things what the person sees and does, not what
  the database calls them. A status reads as the person would say it ("Ended", "Used up",
  "Starts on 8/11"), an action as its outcome ("Send reminder", not "Trigger job").
- **Buttons say the verb.** Destructive ones are named by their verb ("Delete", "Remove
  the item"); no "OK"/"Yes" for an action.
- **Errors are honest:** what happened, and what can be done. "The items couldn't load.
  Check the connection and try again." Not "Error", not a code, not a stack. When the
  server refuses with a reason, show that reason in the reader's language (map its error
  code to a key).
- **Empty states** say there is nothing yet and what to do about it, with the action.
- **No uppercase** for emphasis: Hebrew and Arabic have no case.

## Checklist

- [ ] Every new string is a key in he, ar and en, added as one block at the end of each.
- [ ] No `|| 'English'` or `?? "English"` fallback; library labels translated.
- [ ] Counts use ICU plurals; no number spliced before a word.
- [ ] Hebrew and Arabic labels are verbal nouns; sentences impersonal; addressing goes through `select` with a neutral `other`.
- [ ] No noun or agreeing verb directly before a user-named placeholder; tested with masculine and feminine names.
- [ ] Western digits shown and typed.
- [ ] Every date and time through `lib/clock.ts`; 24-hour; kit dash in ranges.
- [ ] Sentence case in English; the glossary term used; errors honest with a next step.
