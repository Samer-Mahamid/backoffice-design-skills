# RTL guard tests

Vitest + Testing Library + the TypeScript compiler API. Each guard is a rule written as
a test. Mutation-check each once when you add it: break the rule in one file, see the
test fail with a clear message, restore.

## 1. The Icon wrapper

```tsx
import * as lucide from 'lucide-react';
import { ArrowRight, ChevronLeft, Clock, Search, TrendingDown, TrendingUp } from 'lucide-react';
import { render } from '@testing-library/react';
import { describe, expect, it } from 'vitest';
import { DIRECTIONAL_ICONS, Icon } from '@/components/ui/icon';

describe('Icon', () => {
  it.each([ChevronLeft, ArrowRight])('turns %o round in right-to-left', (glyph) => {
    const svg = render(<Icon icon={glyph} />).container.querySelector('svg')!;
    expect(svg.getAttribute('class')).toContain('rtl:-scale-x-100');
  });

  // A trend line keeps its slant: turned round, a rising line reads as a falling one.
  it.each([Clock, Search, TrendingUp, TrendingDown])('leaves %o alone', (glyph) => {
    const svg = render(<Icon icon={glyph} />).container.querySelector('svg')!;
    expect(svg.getAttribute('class')).not.toContain('scale-x');
  });

  it('lets a caller decide either way', () => {
    expect(render(<Icon icon={Clock} mirror />).container.querySelector('svg')!.getAttribute('class')).toContain('rtl:-scale-x-100');
    expect(render(<Icon icon={ChevronLeft} mirror={false} />).container.querySelector('svg')!.getAttribute('class')).not.toContain('scale-x');
  });

  it('is decorative unless labelled', () => {
    expect(render(<Icon icon={Search} />).container.querySelector('svg')!.getAttribute('aria-hidden')).toBe('true');
    const named = render(<Icon icon={Search} label="Search" />).container.querySelector('svg')!;
    expect(named.getAttribute('role')).toBe('img');
    expect(named.getAttribute('aria-label')).toBe('Search');
  });

  it('names only icons lucide has, so a typo cannot quietly stop one mirroring', () => {
    const known = lucide as unknown as Record<string, { displayName?: string }>;
    for (const name of DIRECTIONAL_ICONS) expect(known[name]?.displayName, name).toBe(name);
  });
});
```

## 2. No raw directional icons

Parse, do not grep: a comment, a string or `accept="image/*"` must not hide or invent an
icon. Follow every way a name can arrive: `ArrowLeft`, `ArrowLeftIcon`, `LucideArrowLeft`,
`import { ArrowLeft as Back }`, `import * as L` then `<L.ArrowLeft />`, `const Back = ArrowLeft`.

```ts
import ts from 'typescript';
import { DIRECTIONAL_ICONS } from '@/components/ui/icon';

const lucideName = (exported: string) => {
  if (DIRECTIONAL_ICONS.has(exported)) return exported;
  const bare = exported.replace(/^Lucide/, '').replace(/Icon$/, '');
  return DIRECTIONAL_ICONS.has(bare) ? bare : exported;
};

export function rawDirectionalIcons(source: string, file = 'x.tsx') {
  const sf = ts.createSourceFile(file, source, ts.ScriptTarget.Latest, true, ts.ScriptKind.TSX);
  const named = new Map<string, string>();      // local name → lucide name
  const namespaces = new Set<string>();
  for (const st of sf.statements) {
    if (!ts.isImportDeclaration(st) || !ts.isStringLiteral(st.moduleSpecifier)) continue;
    if (!/^lucide-react(\/|$)/.test(st.moduleSpecifier.text)) continue;
    const b = st.importClause?.namedBindings;
    if (b && ts.isNamespaceImport(b)) namespaces.add(b.name.text);
    if (b && ts.isNamedImports(b)) for (const el of b.elements) named.set(el.name.text, lucideName((el.propertyName ?? el.name).text));
  }
  // aliases: const Back = ArrowLeft
  const visitAlias = (n: ts.Node) => {
    if (ts.isVariableDeclaration(n) && ts.isIdentifier(n.name) && n.initializer && ts.isIdentifier(n.initializer) && named.has(n.initializer.text)) {
      named.set(n.name.text, named.get(n.initializer.text)!);
    }
    ts.forEachChild(n, visitAlias);
  };
  visitAlias(sf);

  const out: { line: number; name: string }[] = [];
  const visit = (n: ts.Node) => {
    if (ts.isJsxOpeningElement(n) || ts.isJsxSelfClosingElement(n)) {
      const tag = n.tagName.getText(sf);
      const [ns, member] = tag.split('.');
      const name = member && namespaces.has(ns) ? lucideName(member) : named.get(tag);
      if (name && DIRECTIONAL_ICONS.has(name)) out.push({ line: sf.getLineAndCharacterOfPosition(n.getStart()).line + 1, name });
    }
    ts.forEachChild(n, visit);
  };
  visit(sf);
  return out;
}

// the test: walk app/ and components/ (skip components/ui/icon.tsx), collect, expect [].
// plus a 'what the scan sees' block: aliases, namespaces, and a comment or string that must NOT count.
```

Add a render test for one real screen in Arabic: its Next button's chevron carries
`rtl:-scale-x-100`, so a regression in the screen, not only the scan, fails.

## 3. LTR fields

```ts
const HOLDS_LTR_VALUE = [
  /\btype="(email|tel|url)"/,
  /\binputMode="(email|tel|url)"/,
  /\b(value=\{|register\(\s*["'])[\w$?.]*?(phone|Phone|email|Email|url|Url|website|Website|taxId|nationalId|idNumber)\w*/,
];
const LTR = /\bdir=(?:"ltr"|\{\s*['"]ltr['"]\s*\})/;
// For every <Input, <input, <Textarea tag (comments blanked, quoted values kept, the tag read to
// its closing '>' past {…} and quotes): if any HOLDS_LTR_VALUE matches, LTR must match.
// Report file:line and the tag. Keep a 'what the scan sees' block with a multi-line tag,
// a value bound through an object path, and a commented-out field that must not count.
```

## 4. Isolation in rendered output

```tsx
it.each(['he', 'ar', 'en'] as const)('a time range reads start first in %s', (lang) => {
  setLanguage(lang);
  const { container } = render(<ClockRange start="2030-09-19T08:15:00" end="2030-09-19T09:15:00" />);
  const el = container.querySelector('[dir="ltr"]')!;
  expect(el.textContent).toBe('08:15–09:15');          // the kit dash, no spaces
});

it('a phone number inside a Hebrew sentence stays whole', () => {
  render(<Sentence template="נשלח קוד אל {phone}" values={{ phone: <span dir="ltr">+1 555-010-0199</span> }} />);
  const phone = screen.getByText('+1 555-010-0199');
  expect(phone.closest('bdi')).not.toBeNull();
  expect(phone.getAttribute('dir')).toBe('ltr');
});
```

jsdom does no layout, so these tests assert the structure that makes the order right
(the isolate and its content). Look at the real screen too, in Hebrew and Arabic.

## Source-scan hygiene (all guards)

- Strip comments before scanning, keeping line numbers; never treat a quoted value as
  a comment.
- Report `file:line` and the offending text.
- An allowance list of known sites is checked for staleness: if a listed site no longer
  offends, the test fails until the entry is removed.
- Prove the scan sees each shape with small inline sources, positive and negative.
