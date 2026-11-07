# Changelog

## v1 — 2026-11-07

First set of skills, distilled from a production he/ar/en studio panel's design system,
kit and guard tests.

- `design-tokens-and-theme`: colour roles in light and dark, brand slots and their
  derivation, radii, elevation, motion tokens, the Tailwind 4 `@theme` setup, the ratchet.
- `layout-spacing-and-type`: spacing scale, type roles (Arabic line heights, no tracking
  or uppercase on Hebrew and Arabic), shell, page anatomy, 1280/390 widths.
- `components-and-states`: the kit and the four states (skeleton, empty with action,
  error with Retry, data), error kinds, one shared rule per concept, the showcase route.
- `overlays-and-notices`: sheet versus dialog, button order and danger, stacking, notices
  at 4/4/5/6 s with a countdown on warning, failure and undo, and a 44 px close.
- `rtl-and-bidi`: logical classes, the `<Icon>` directional set (trend lines do not
  mirror), bidi isolation, `Sentence` templates, LTR fields, charts, guard tests.
- `i18n-and-copy`: three-language translations without fallbacks, ICU plural and gender
  resolvers, impersonal Hebrew and Arabic, Western digits, one clock module, word rules.
- `design-workflow-and-review`: gates G1–G4, the mockup method, pure moves, "a pattern
  changes everywhere", guard tests with mutation checks, the review checklist.
- `dashboard-patterns`: stat tiles and deltas, period and filter rows, the card grid,
  per-card states, charts in RTL, record cards, history lists, first-run dashboards.
