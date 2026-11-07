# The mockup method

Mockups are plain HTML files, drawn with the same token values as the product, so what
the owner approves is what the code can build.

## Layout of the folder

```
design/
  tokens.css            # the token roles as CSS variables + a few helper classes
  build.py              # checks tokens, inlines tokens.css into each mockup
  mockups/
    src/mock-dashboard.html   # sources, each with /*__TOKENS__*/ inside its <style>
    mock-dashboard.html       # built output (commit it; it is what gets shared)
  screens/<change>/            # G3 before/after PNGs
```

## tokens.css

- Every colour role as a variable, light under `:root`, dark under
  `@media (prefers-color-scheme: dark)` and `:root[data-theme="dark"]`.
- The type roles as classes (`.t-headline`, `.t-body`, `.t-meta`, `.t-stat`), radii
  (`--r-sm`, `--r-md`, `--r-lg`, `--r-xl`, `--r-pill`), elevation (`--e1`, `--e2`,
  `--e3`), durations and easings.
- Helpers the mockups share: `.pill.success/warning/danger/info/brand/neutral`,
  `.skeleton` with its sweep, `.num, .money, .phone, .time { direction: ltr;
  unicode-bidi: isolate; display: inline-block; }`, `.tab { font-variant-numeric:
  tabular-nums; }`, and the reduced-motion block.

A helper class named like a frame (`.phone` for a phone number vs a phone-shaped frame)
will force `direction: ltr` on a whole right-to-left frame. Name frames distinctly, or add
`.frame[dir="rtl"] { direction: rtl; }`.

## build.py

1. Read the token source in code (for example `lib/tokens.ts`, or the generated block in
   `globals.css`) and `tokens.css`; compare every colour role, light and dark; on any
   difference print it and exit non-zero.
2. For each `mockups/src/*.html`, assert it contains `/*__TOKENS__*/`, replace the marker
   with the contents of `tokens.css`, and write the result beside `src/`.
3. Print each output's size.

Run it after every token change. A token change that alters another mockup is visible in
the diff.

## What a mockup shows

- Every screen in the change, in he, ar and en; light and dark; desktop (1280, with the
  sidebar) and phone (390).
- The states, not only the happy path: loading skeleton, empty with its action, error with
  Retry, a long Hebrew name, a long Arabic name, a large number.
- Overlays open over their screen; notices of each kind.
- A small control bar at the top of the file (language, theme, width) is fine; it is not
  part of the design.
- Real-looking but invented data. No real people, phones or businesses.

## Approval

Publish the built file (an artifact link, a static host, or the PR's preview) for the
owner. Wait for an explicit approval on the PR before finishing visual code. Record the
approval (date, link, any notes) in the PR and in the project's design log. After
approval, the build follows the frames; a change to an approved frame goes back to the
owner.

## G3 frames

For each changed screen, capture **before** (the base branch) and **after** (the PR)
at 1280 and 390, in he, ar and en, light and dark. Save them under
`design/screens/<change>/` with names like `items-list_he_dark_390_after.png`, and list
them in the PR. Capture from the running app or from the showcase route. Only screens
whose layout changed need frames.
