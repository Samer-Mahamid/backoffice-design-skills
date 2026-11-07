# The ratchet: counts that may only fall

A migration to tokens never finishes in one change. The ratchet makes progress one-way: a
test counts every offender, compares the count with a committed baseline, and fails when
the count **rises** and also when it **falls without the baseline being lowered** (slack
in the baseline is room for a regression to hide in).

## Files

- `test/ratchet.test.ts`: the counters and the comparison.
- `test/ratchet-baseline.json`: `{ "about": "...", "<counter>": { "total": N, "files": { "<path>": n } } }`.
- `test/support/ratchet.ts`: the source reader and the counters.
- `scripts/ratchet-lower.sh`: rewrites the baseline to the current counts, refusing to raise any.

## Counters

| Key | Counts | Advice printed on failure |
|---|---|---|
| `english-fallbacks` | `t.x \|\| 'Words'`, `?? "Words"`, `\|\|= 'Words'`, a fallback through a ternary or a join | add the key to all three languages |
| `hex-literals` | `#abc`, `#a1b2c3`, `#a1b2c3d4` in `.tsx`/`.ts`/`.css`, outside the generated token block; not `#123` in a sentence, a link fragment or an HTML entity | use a role |
| `palette-classes` | `(bg\|text\|border\|ring\|from\|via\|to\|fill\|stroke\|outline\|divide\|shadow\|inset-ring\|drop-shadow\|text-shadow\|…)-(slate\|gray\|zinc\|…\|rose)-\d+`, `-white`, `-black`, in class strings, `@apply`, `var(--color-slate-500)`, `theme(--color-red-500)` | use a role |
| `physical-direction` | `pl- pr- ml- mr- left- right- text-left text-right border-l border-r rounded-l rounded-r rounded-tl…` not under `ltr:`/`rtl:`; `margin-left`, `padding-right`, `left:`, `text-align: left` in CSS. Exempt: `left-1/2 -translate-x-1/2` centring | use `ps- pe- ms- me- start- end- text-start border-s rounded-s` |
| `motion-literals` | `animate-*`, `transition-all`, `duration-300`, `duration-[250ms]`, framer `duration: 0.4` outside `lib/motion.ts`; `transition: all .3s` in CSS. Not a form's `duration: 60` minutes | use the motion tokens |
| `ui-tailwind-radii` | `rounded`, `rounded-sm…4xl`, `rounded-t-xl` inside `components/ui/` only | use `rounded-badge/control/card/sheet/pill` |

## How the counters read code

These details are where source scans go wrong. Each one has broken a real scan.

- **Strip comments first, keep offsets.** A guard's own explanatory comment names the
  thing it forbids. Blank comments to spaces so line numbers stay true.
- **A quoted value is not a comment.** `accept="image/*"` opens a `/*` for a naive regex
  and hides the next hundred lines. Skip quoted strings while stripping.
- **Read every string shape:** single, double, template literals with nested `${…}` and
  nested templates, JSX attribute values over several lines.
- **Count symbols, not lines.** Two offenders on one line are two.
- **Print what moved.** On failure, list each file whose count changed, with
  `file:line  text` for every hit, so the fix is obvious.
- **Prove each counter sees each shape.** Keep a `describe('what each counter sees')`
  block with positive and negative examples (see below).

## Test shape

```ts
import { readFileSync, writeFileSync } from 'node:fs';
import path from 'node:path';
import { describe, expect, it } from 'vitest';
import { COUNTERS, countAll, readSource } from './support/ratchet';

const root = path.resolve(__dirname, '..');
const baselineFile = path.join(root, 'test/ratchet-baseline.json');

describe('the ratchets', () => {
  const baseline = JSON.parse(readFileSync(baselineFile, 'utf8'));
  const counts = countAll(root);

  it.each(COUNTERS.map((c) => [c.key, c] as const))('%s does not rise, and the baseline is tight', (key, counter) => {
    const allowed = baseline[key]?.total;
    expect(allowed, `${key} has no baseline; run scripts/ratchet-lower.sh`).toBeTypeOf('number');
    if (process.env.RATCHET_LOWER === '1') return;
    expect(counts[key].total, `${key} rose. ${counter.advice}`).toBeLessThanOrEqual(allowed);
    expect(counts[key].total, `${key} fell: run scripts/ratchet-lower.sh`).toBe(allowed);
  });

  it('lowers the baseline when asked, never raising it', () => {
    if (process.env.RATCHET_LOWER !== '1') return;
    const next: Record<string, unknown> = { about: baseline.about };
    for (const c of COUNTERS) {
      const now = counts[c.key];
      if (now.total > (baseline[c.key]?.total ?? Infinity)) throw new Error(`${c.key} rose; the baseline only goes down`);
      next[c.key] = { total: now.total, files: now.files };
    }
    writeFileSync(baselineFile, JSON.stringify(next, null, 2) + '\n');
  });
});

describe('what each counter sees', () => {
  const find = (key: string, src: string, file = 'components/x.tsx') =>
    COUNTERS.find((c) => c.key === key)!.find(readSource(file, src)).map((m) => m.text);

  it('palette classes in any string shape, not the roles', () => {
    expect(find('palette-classes', `const a = "text-slate-500 hover:bg-indigo-50 bg-white !bg-red-500";`))
      .toEqual(['text-slate-500', 'bg-indigo-50', 'bg-white', '!bg-red-500']);
    expect(find('palette-classes', `const a = "bg-surface-1 text-text-2 border-border bg-brand";`)).toEqual([]);
  });

  it('a comment-like run inside a string is not a comment', () => {
    const src = `const a = <input accept="image/*" />;\nconst b = 'bg-slate-100';`;
    expect(find('palette-classes', src)).toEqual(['bg-slate-100']);
  });

  it('physical sides, but not logical utilities or centring', () => {
    expect(find('physical-direction', `const a = "pl-4 ml-auto text-left border-l md:pr-2";`))
      .toEqual(['pl-4', 'ml-auto', 'text-left', 'border-l', 'md:pr-2']);
    expect(find('physical-direction', `const a = "ps-4 me-2 start-0 text-start border-s absolute left-1/2 -translate-x-1/2";`)).toEqual([]);
  });
});
```

## Lowering script

```bash
#!/usr/bin/env bash
# Lowers test/ratchet-baseline.json to the current counts after offenders were removed.
# Refuses to raise any count: a rise is a regression for the change to fix.
set -euo pipefail
cd "$(dirname "$0")/.."
RATCHET_LOWER=1 npx vitest run test/ratchet.test.ts
git --no-pager diff --stat -- test/ratchet-baseline.json
```

## Rules for people and agents

- Never raise the baseline by hand. A rise is fixed in the change that caused it.
- After removing offenders, run the script and commit the lowered baseline in the same
  change.
- New code and rewritten lines carry zero offenders. Old screens fall as they are
  redesigned.
- Starting a new project, the baseline is zero for every counter: keep it there.
