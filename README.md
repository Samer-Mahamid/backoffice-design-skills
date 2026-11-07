# Back-office design skills

Claude Code skills that teach AI agents to design and build a back-office dashboard the
way a production he/ar/en studio panel was designed: Next.js (App Router) + Tailwind 4 +
lucide icons, Hebrew, Arabic and English, right-to-left by design.

The skills carry the method, the structure, spacing, type scale, states, overlays,
notices, RTL and copy rules, and the guard tests that keep them true. Colour is a set of
**slots**: each product fills in its own brand, and the rest is derived or fixed.

## The skills

| Skill | Covers |
|---|---|
| [`design-tokens-and-theme`](skills/design-tokens-and-theme/SKILL.md) | Colour roles (surfaces 0–2, text 1–3, borders, brand and its derived roles, status inks and washes, charts), light and dark, radii, elevation, motion tokens, the "fill your brand" slots, the Tailwind 4 `@theme` setup, and the ratchet test that keeps raw values out. |
| [`layout-spacing-and-type`](skills/layout-spacing-and-type/SKILL.md) | The spacing scale, radii and elevation in use, the type roles with Arabic line heights, the shell and sidebar, page anatomy, card versus plain section, and making every page work at 1280 and 390 px (tables become rows on a phone). |
| [`components-and-states`](skills/components-and-states/SKILL.md) | The kit (buttons, fields, chips, segmented tabs, status pills, stat tiles, data tables with `rowOpens`, list rows, cards) and the four states every screen has: loading skeleton, empty with its next action, error with honest words and Retry, data. One shared rule per concept. |
| [`overlays-and-notices`](skills/overlays-and-notices/SKILL.md) | Sheet versus dialog, button order (secondary at the start, one primary at the end), destructive actions named by their verb, stacking when a label does not fit, and notices: 4/4/5/6 s by kind, a countdown bar on warning, failure and undo, a 44 px close, failures announced assertively. |
| [`rtl-and-bidi`](skills/rtl-and-bidi/SKILL.md) | RTL by construction: logical classes only, the `<Icon>` wrapper and its directional set (trend lines, clocks, checks and search never mirror), bidi isolation of phones, emails, codes, money, percentages and time ranges, mixed-direction sentences, LTR fields, charts, and the guard tests. |
| [`i18n-and-copy`](skills/i18n-and-copy/SKILL.md) | One translations file with three equal languages, no silent English fallback, ICU plurals and gender, impersonal Hebrew and Arabic, a gender-free construction before a user-named thing, Western digits everywhere, dates and times through one module, sentence case, words from the user's side. |
| [`design-workflow-and-review`](skills/design-workflow-and-review/SKILL.md) | The process: foundation decisions, a token-built HTML mockup approved by the owner, before/after frames, building to the approved frames, screens reviewed before code, the test gate. "A pattern changes everywhere, never one screen", pure moves before visual change, a guard test for every rule (mutation-checked), and the review checklist. |
| [`dashboard-patterns`](skills/dashboard-patterns/SKILL.md) | KPI tiles, deltas and unmirrored trend icons, period and filter rows, the card grid, per-card states, charts in RTL with table twins, record cards and history lists, and a first-run dashboard that teaches the next step instead of showing zeros. |

Each skill is a `SKILL.md` (the rules) and a `references/` folder with longer material:
token values, working component code, guard-test shapes.

## Install

### Copy

```bash
mkdir -p .claude/skills
cp -R /path/to/backoffice-design-skills/skills/* .claude/skills/
```

Commit `.claude/skills/` with the project so every agent session has them.

### Submodule and symlinks (stays up to date)

```bash
git submodule add git@github.com:Samer-Mahamid/backoffice-design-skills.git vendor/backoffice-design-skills
mkdir -p .claude/skills
for s in vendor/backoffice-design-skills/skills/*/; do
  name=$(basename "$s")
  ln -s "../../vendor/backoffice-design-skills/skills/$name" ".claude/skills/$name"
done
git add .gitmodules vendor/backoffice-design-skills .claude/skills
```

Update later with `git submodule update --remote vendor/backoffice-design-skills`, and
commit the moved pointer.

Claude Code loads skills from `.claude/skills/<name>/SKILL.md` in the project (or
`~/.claude/skills/` for every project on a machine). An agent picks a skill when the task
matches its `description`, and reads the `references/` files when the skill points to
them.

## Point your agents at them

Paste this into the project's `CLAUDE.md`:

```markdown
## Design

This dashboard follows the back-office design skills in `.claude/skills/`. Before any UI
work, load the matching skill:

- tokens, colours, dark mode, brand → `design-tokens-and-theme`
- page layout, spacing, type, phone widths → `layout-spacing-and-type`
- buttons, fields, tables, pills, loading/empty/error states → `components-and-states`
- dialogs, sheets, confirms, toasts → `overlays-and-notices`
- anything with a side, an icon, or a value inside Hebrew/Arabic text → `rtl-and-bidi`
- any user-facing string, number, date or time → `i18n-and-copy`
- planning a screen, mockups, reviewing a UI PR → `design-workflow-and-review`
- the dashboard and report pages → `dashboard-patterns`

Our brand slots: brand `#______`, accent `#______` (or none), logo `public/logo.svg`,
font faces: ______ (must cover Hebrew and Arabic). Default language: Hebrew.
Never add a hex value or a Tailwind palette class to a component; never use a physical
side class; never fall back to English in code. The guard tests enforce this.
```

Fill in the brand line once; everything else stays as written.

## What is not here

- No product's brand values, names, data, endpoints or business rules. The examples use
  neutral words ("item", "record", "member").
- The motion catalogue beyond the tokens and the reduced-motion rule.
- Mobile-app (Flutter) specifics. The source product shared these rules with a Flutter
  app, but these skills target the web dashboard.
