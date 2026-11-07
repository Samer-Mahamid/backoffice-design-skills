# i18n guard tests

Each rule in the skill has a test. Add them on day one; they are cheap when the
translations file is small.

## Parity: the same keys in all three languages

```ts
import { describe, expect, it } from 'vitest';
import { translations } from '@/lib/translations';

const locales = ['he', 'ar', 'en'] as const;
const paths = (v: unknown, prefix = ''): string[] =>
  v === null || typeof v !== 'object' || Array.isArray(v)
    ? [prefix]
    : Object.entries(v as Record<string, unknown>).flatMap(([k, x]) => paths(x, prefix ? `${prefix}.${k}` : k));
const byLocale = Object.fromEntries(locales.map((l) => [l, new Set(paths(translations[l]))])) as Record<(typeof locales)[number], Set<string>>;
const get = (o: unknown, p: string) => p.split('.').reduce<unknown>((x, k) => (x as Record<string, unknown> | undefined)?.[k], o);

describe('translation parity', () => {
  it.each(['he', 'ar'] as const)('%s has every key English has', (l) => {
    const missing = [...byLocale.en].filter((p) => !byLocale[l].has(p));
    expect(missing, `${l} is missing:\n${missing.join('\n')}`).toEqual([]);
  });
  it.each(['he', 'ar'] as const)('English has every key %s has', (l) => {
    const extra = [...byLocale[l]].filter((p) => !byLocale.en.has(p));
    expect(extra, `not in English:\n${extra.join('\n')}`).toEqual([]);
  });
  it('no Hebrew or Arabic string is left as its English original', () => {
    const allowed = /^(English|עברית|العربية|WhatsApp|SMS|ID|QR|URL|Email)$/;   // names that stay as they are
    const suspicious: string[] = [];
    for (const l of ['he', 'ar'] as const) for (const p of byLocale[l]) {
      const v = get(translations[l], p), en = get(translations.en, p);
      if (typeof v === 'string' && v === en && /[A-Za-z]/.test(v) && !allowed.test(v.trim())) suspicious.push(`${l}.${p} = ${JSON.stringify(v)}`);
    }
    expect(suspicious, `Untranslated:\n${suspicious.join('\n')}`).toEqual([]);
  });
});
```

## Keys exist: what the code reads is real

Parity is not enough: three files can agree perfectly on a key none of them has, and the
page shows its English fallback forever. Scan `app/` and `components/` for `t.a.b.c`
chains (and aliases like `const ti = t.items` then `ti.title`; keep one alias per name per
file so the scan can follow it), and assert each path exists in at least one language.
Parity does the rest.

## English fallbacks: the ratchet

The `english-fallbacks` counter (see `design-tokens-and-theme/references/ratchet.md`)
counts `|| 'Words'`, `?? "Words"`, `||= 'Words'`, and fallbacks through a ternary or a
join. In a new project its baseline is zero.

## Reader-addressing words (Hebrew)

```ts
const BOUND = '[\\s"?!.,()]';
// Masculine second person and imperatives. Leave out forms that are also ordinary nouns
// ("המשך" continuation, "שנה" year, "הזמן" the time, "רשום" registered).
const HEBREW_ADDRESSING = ['אתה', 'בטוח', 'תרצה', 'תוכל', 'מסכים', 'רשאי', 'תאשר', 'תבחר'];
const HEBREW_IMPERATIVE = [
  'בחר', 'הזן', 'השלם', 'התחבר', 'שלח', 'דלג', 'ערוך', 'הוסף', 'מחק', 'שמור', 'עקוב', 'נסה', 'צפה',
  'הצג', 'סגור', 'נהל', 'התנתק', 'הפעל', 'העלה', 'הורד', 'שתף', 'חפש', 'סנן', 'עדכן', 'התחל', 'הגדר',
  'צור', 'לחץ', 'הפק', 'חזור', 'הסר',
];
const hit = (word: string, value: string) => new RegExp(`(^|${BOUND})ו?${word}($|${BOUND})`).test(value);

// 1. every Hebrew string resolved for 'female' contains no HEBREW_ADDRESSING word;
// 2. every Hebrew string resolved for 'other' contains no HEBREW_IMPERATIVE word;
// 3. at least one string differs between the 'female' and 'male' resolutions;
// 4. end to end: render the provider signed in as female, male, and unknown, and read one
//    gendered key: feminine, masculine, neutral (and neutral with nobody signed in).
```

Write an Arabic list the same way for the common masculine imperatives (`اختر`, `أدخل`,
`احفظ`, `احذف`, `اضغط`) if your Arabic copy is written by more than one hand.

## Digits

```ts
export function westernDigits(text: string): string {
  return text.replace(/[\u0660-\u0669\u06F0-\u06F9\u066B]/g, (c) => {
    const u = c.charCodeAt(0);
    if (u === 0x066b) return '.';
    return String(u - (u >= 0x06f0 ? 0x06f0 : 0x0660));
  });
}
```

- Unit: both digit sets and the decimal key convert; neighbouring characters and
  Hebrew/Arabic letters are untouched; one UTF-16 unit in, one out, so the caret stays.
- Field: typing `٤٢` into a text field and a number field yields `42` in the value and
  reaches `onChange`; a password field is left alone.
- Output: every formatter in `lib/clock.ts` and the number formatter, called for `ar`,
  produces no U+0660–0669 or U+06F0–06F9.

## Times only through lib/clock.ts

Parse each source file with the TypeScript compiler API and flag:

- `x.toLocaleTimeString(…)`;
- `x.toLocaleString(…)` / `x.toLocaleDateString(…)` whose options ask for `hour`,
  `minute`, `timeStyle` or `dateStyle` with time;
- `new Date(…).toLocaleString()` with no options (it writes a date and a time);
- `new Intl.DateTimeFormat(…)` asking for an hour;
- the string literal `'ar-u-nu-latn'` (the locale rule spelled out by hand).

Not flagged: a number's `toLocaleString` (money, counts), a date's options without an
hour. Every file except `lib/clock.ts` must have none. Prove the scan with inline sources
in both directions, and check that it does find the calls inside `lib/clock.ts` itself.

Unit tests of `lib/clock.ts`, per language: `13:05` and `00:05` (not `24:05`); a range
start first with the en dash and no spaces; no `AM`/`PM`; Latin digits in Arabic.
