# ICU resolvers for a plain translations object

The translations stay plain strings (no message-format library), so `t.a.b` keeps
working everywhere. Two small resolvers read the ICU groups. The syntax is standard ICU,
so a string can move to an ARB or a `.po` file unchanged.

## Gender: resolve the whole tree once

```ts
export type GenderBranch = 'female' | 'male' | 'other';

/** Tolerant of what the server stores: "Female", "female", "F", the Hebrew or Arabic word. */
export function genderBranch(stored?: string | null): GenderBranch {
  const g = (stored ?? '').trim().toLowerCase();
  if (g.startsWith('f') || g === 'נקבה' || g === 'أنثى') return 'female';
  if (g.startsWith('m') || g === 'זכר' || g === 'ذكر') return 'male';
  return 'other';
}

/** Replace each `{gender, select, female{…} male{…} other{…}}` group with one branch. */
export function resolveGender(value: string, branch: GenderBranch): string {
  const open = value.indexOf('{gender,');
  if (open < 0) return value;
  // Walk to the matching brace: branches hold `{name}` placeholders of their own.
  let depth = 0, end = -1;
  for (let i = open; i < value.length; i++) {
    if (value[i] === '{') depth++;
    else if (value[i] === '}' && --depth === 0) { end = i; break; }
  }
  if (end < 0) return value;
  const body = value.slice(open, end + 1);
  const inner = body.slice(body.indexOf(',', body.indexOf(',') + 1) + 1, -1);
  const branches: Record<string, string> = {};
  for (let i = 0; i < inner.length;) {
    const brace = inner.indexOf('{', i);
    if (brace < 0) break;
    const name = inner.slice(i, brace).trim();
    let d = 0, close = brace;
    for (let j = brace; j < inner.length; j++) {
      if (inner[j] === '{') d++;
      else if (inner[j] === '}' && --d === 0) { close = j; break; }
    }
    branches[name] = inner.slice(brace + 1, close);
    i = close + 1;
  }
  const chosen = branches[branch] ?? branches.other ?? '';
  return resolveGender(value.slice(0, open) + chosen + value.slice(end + 1), branch);   // a string may hold two groups
}

/** A copy of the translations with every gender group resolved, functions included. */
export function resolveTree<T>(tree: T, branch: GenderBranch): T {
  if (typeof tree === 'string') return resolveGender(tree, branch) as T;
  if (typeof tree === 'function') {
    const fn = tree as unknown as (...a: unknown[]) => unknown;
    return ((...a: unknown[]) => { const out = fn(...a); return typeof out === 'string' ? resolveGender(out, branch) : out; }) as T;
  }
  if (Array.isArray(tree)) return tree.map((v) => resolveTree(v, branch)) as T;
  if (tree && typeof tree === 'object') {
    return Object.fromEntries(Object.entries(tree).map(([k, v]) => [k, resolveTree(v, branch)])) as T;
  }
  return tree;
}

// In the provider: the signed-in person's OWN gender (optional; 'other' when unknown or signed out).
const t = useMemo(() => resolveTree(translations[language], genderBranch(user?.gender)), [language, user?.gender]);
```

Take the reader's gender from their own user record, not from a setting about the
audience of the business: the person running a women-only venue may be a man.

## Plural: resolve with the numbers a sentence counts

```ts
const OPEN = /\{(\w+),\s*plural,/;

/** A whole number, or undefined. An empty string is not zero. */
export function wholeNumber(value: unknown): number | undefined {
  if (typeof value === 'number') return Number.isInteger(value) ? value : undefined;
  if (typeof value !== 'string' || !/^[+-]?\d+$/.test(value.trim())) return undefined;
  return Number(value.trim());
}

function branchesOf(inner: string): Record<string, string> {
  const out: Record<string, string> = {};
  for (let i = 0; i < inner.length;) {
    const brace = inner.indexOf('{', i);
    if (brace < 0) break;
    const name = inner.slice(i, brace).trim();
    let d = 0, close = -1;
    for (let j = brace; j < inner.length; j++) {
      if (inner[j] === '{') d++;
      else if (inner[j] === '}' && --d === 0) { close = j; break; }
    }
    if (close < 0) break;
    out[name] = inner.slice(brace + 1, close);
    i = close + 1;
  }
  return out;
}

function pick(b: Record<string, string>, n: number, rules: Intl.PluralRules) {
  return b[`=${n}`] ?? b[rules.select(n)] ?? (rules.select(n) === 'two' ? b.few : undefined) ?? b.other ?? '';
}

/** `value` with each plural group replaced; undefined when a group's number is missing. */
export function resolvePlural(value: string, count: (name: string) => number | undefined, locale: string): string | undefined {
  const m = OPEN.exec(value);
  if (!m) return value;
  let depth = 0, end = -1;
  for (let i = m.index; i < value.length; i++) {
    if (value[i] === '{') depth++;
    else if (value[i] === '}' && --depth === 0) { end = i; break; }
  }
  if (end < 0) return value;
  const n = count(m[1]);
  if (n === undefined) return undefined;
  const chosen = pick(branchesOf(value.slice(m.index + m[0].length, end)), n, new Intl.PluralRules(locale));
  return resolvePlural(value.slice(0, m.index) + chosen + value.slice(end + 1), count, locale);
}

/** A label that is a number and its noun, and nothing else. */
export function sayCount(template: string, n: number, locale: string): string {
  return (resolvePlural(template, () => n, locale) ?? template).replace(/\{count\}/g, String(n));
}
```

Put a `locale` key at the top of each language block (`'he'`, `'ar'`, `'en'`) and pass it
to `Intl.PluralRules`.

## Placeholders

`{name}` placeholders stay in the resolved strings. Render them with the `Sentence`
component (see `rtl-and-bidi`) so each value is isolated. `.replace('{name}', name)` is
acceptable only for a string that never reaches a right-to-left line with a value of the
other direction, which in a he/ar/en product is almost never.
