---
name: design-workflow-and-review
description: The process for designing, building and reviewing back-office dashboard UI in a he/ar/en product - foundation decisions, a token-built HTML mockup approved by the owner, before/after screenshots, building to the approved frames, design review then code review, and a test gate with a guard for every rule. Use when planning a redesign or a new screen or pattern, preparing mockups or screenshots for approval, splitting a large file before a visual change, deciding whether a local fix may depart from the kit, adding a design guard test, or reviewing a UI pull request.
---

# Design workflow and review

Design changes land in a fixed order, with the owner's approval at two gates, and every
rule they rely on is held by a test. The aim: one product that looks the same on every
screen, in every language, after many hands have touched it.

## The gates

| Gate | What | Who approves |
|---|---|---|
| **G1 foundation** | tokens, fonts, type scale, motion, kit inventory, open decisions listed with a recommendation each | owner |
| **G2 mockup** | each screen or pattern as a static HTML mockup in he, ar, en, light and dark, at 1280 and 390 | owner, before any visual code is finished |
| **build** | code to the approved frames, one PR per screen group | — |
| **G3 before/after** | screenshots of the old and new screen in he, ar, en × light, dark at 1280 and 390, in the PR | design review |
| **review** | the screens first (design review), then the code (code review) | reviewers |
| **G4 walk** | a walk in Hebrew first, then Arabic, then English; reduced motion on; text at 130 % and 200 % | owner, for what only a real device shows |
| **gate** | typecheck, build, the full test suite, the ratchet | CI |

Do not finish visual work before G2. While waiting, do the work that needs no visual
decision: pure moves, fixtures, test scaffolding, the states.

## G2: mockups built from the tokens

- One static HTML file per area (`design/mockups/src/mock-<area>.html`), using only
  classes defined in the shared `tokens.css`.
- A build script inlines `tokens.css` at a `/*__TOKENS__*/` marker into each mockup, and
  **fails if the tokens in code and in `tokens.css` differ**. Building one mockup must
  leave every other mockup byte-identical.
- Each frame is labelled with language, theme and width. Hebrew and Arabic frames carry
  `dir="rtl"`; watch for a helper class that forces `direction: ltr` on a frame by name.
- The mockup PR is docs-only. Its description says what each screen shows and which
  current behaviour it keeps.

Details in [references/mockup-method.md](references/mockup-method.md).

## Building

- **Behaviour does not change in a design change.** A restyle does not alter a flow, a
  rule, an API call or a permission. If a screen is wrong, list it in the PR under
  "Gaps". A tiny, safe fix goes in its own commit with its own test, named in the PR.
- **Pure moves first.** A file over about 400 lines is split into smaller components
  before any visual change, in its own commit: only file creation and imports change,
  tests unchanged and green before and after. The visual commit comes later, so each
  reverts on its own.
- **Kit first.** Build from `components/ui/*`. If the screen needs something the kit
  lacks, add it to the kit (with its states in the showcase), then use it.
- **Fix batches and feature batches in separate commits**, so they revert independently.

## A pattern changes everywhere, never one screen

A fix that touches a pattern (overlays, notices, buttons, fields, chips, tables) uses the
kit's component and the rule for that pattern. If the fix needs the pattern itself to
change:

1. get a rule for the pattern (from the design owner);
2. find every place the rule applies, in every app that shares the design;
3. change them all in one batch, or list the ones left with a reason.

Never just the one screen. When a fix lands on a screen a planned redesign will replace,
build it in the kit's language now, and add it to that redesign's **kept fixes** list
(fix, screen, behaviour, words) so the redesign keeps it.

## One shared rule per concept

When the same question is answered in two places (is this record active? which actions
does this state allow? how is a time range written?), answer it in one module and import
it everywhere: `lib/record-state.ts`, `lib/clock.ts`, `lib/digits.ts`, the `STATUSES`
registry, the `DIRECTIONAL_ICONS` set. Prefer the server's answer over a client
re-derivation.

## A guard test for every rule

A rule nobody tests erodes. For each rule:

- write the test that fails when the rule is broken (a source scan, a render assertion,
  a ratchet counter);
- **mutation-check it once:** break the rule on purpose in one place, watch the test
  fail with a message that says what to do, restore;
- make it print `file:line` and the offending text;
- for scans: parse code with the TypeScript compiler API where syntax matters, strip
  comments (a guard's own comment names what it forbids), never treat `"image/*"` as a
  comment opener, and check any allow-list for stale entries.

The guards a he/ar/en dashboard needs are listed in
[references/review-checklist.md](references/review-checklist.md).

## The showcase

A route that renders every kit component in every state, in {light, dark} × {he, ar, en},
built only with a flag (404 otherwise, so it never ships). It is the G3 surface for kit
changes and the target for snapshot or screenshot tests.

## Reviewing a UI pull request

Review the **screens first**, then the code.

1. Open the before/after frames. Compare with the approved G2 mockup, frame by frame.
2. Check consistency with the rest of the product, not only the screen itself.
3. Then the code: kit use, tokens, logical classes, isolation, copy keys, tests.
4. One adversarial pass by one reviewer who gets the code, not the author's conclusions.
5. Every confirmed finding is fixed, and each fix has a test that fails without it.

The PR ends with an evidence line: the tested commit and the totals, for example
`tested tree <sha>: N passed, 0 failed, 0 skipped`. Nothing may fail; an unexpected skip
counts as a failure.

## Checklist for a design PR

- [ ] G2 approved before the visual work; the build matches the approved frames.
- [ ] Pure moves in their own commit; fix and feature commits separate.
- [ ] No behaviour change; gaps listed in the PR.
- [ ] A changed pattern is applied everywhere, or the remaining places are listed.
- [ ] Before/after frames: he, ar, en × light, dark × 1280, 390.
- [ ] All four states built and tested; long Hebrew and Arabic text tested.
- [ ] New rules have guard tests, each mutation-checked once.
- [ ] Ratchet did not rise; lowered if it fell.
- [ ] Typecheck, build and the full suite pass; evidence line in the PR.
