---
name: overlays-and-notices
description: The rules for overlays and notices in a he/ar/en back-office dashboard (Next.js + Tailwind 4) - when a question is a sheet (slide-over) and when it is a dialog, how the buttons are ordered, coloured and named, and how notices (toasts) look, how long they stay, and how they are announced. Use when adding or reviewing a modal, dialog, confirm, "are you sure", delete confirmation, slide-over, drawer, sheet, popup, toast, snackbar, sonner notice, undo notice, or a button row in an overlay.
---

# Overlays and notices

One rule for every overlay, so a question looks the same wherever it is asked. A
pattern changes everywhere or not at all: no screen departs from the kit's overlay,
button order or colour on its own to solve a local problem.

## Sheet or dialog

| The person… | Use | Kit entry point |
|---|---|---|
| fills in or chooses something: a field, radios, chips, a dropdown, a list to pick from, a date, a second step | a **sheet** | `SlideOver` (or the kit `Dialog` for a short task of one to three fields) |
| confirms a plain action, or reads a notice: a title, one or two sentences, at most one checkbox as an option on the action, one or two buttons | a **dialog** | `ConfirmDialog` |

Never:

- a field or a choice inside a confirm dialog;
- a yes/no inside a sheet;
- a hand-rolled overlay (`fixed inset-0 …` in a screen). A source-scan test keeps
  `fixed inset-0` out of everything but the kit's overlay files.

The one checkbox a dialog may hold is an option on its action ("Also notify the people
on the waiting list"), not a form.

## The sheet

- Slides in from the **end edge** (left in Hebrew and Arabic), on the spring, over the
  `scrim`, at `e3`, `bg-surface-1`, `rounded-sheet` on its open side. Under reduced
  motion it appears and leaves at once.
- Header: the title in `text-headline`, an optional `text-body text-text-2`
  description, optional facts (chips) under it, a close button (44 px target, labelled).
- Body: laid out at full size from the first frame, scrolls in the room it has.
- Footer: the actions in a sticky bar **below** the content, never floating over it.
  A consequence, when there is one, is the bar's warning line (`text-meta text-warning`
  with an icon) above the buttons.
- **Unsaved changes:** with `isDirty`, closing asks first ("Discard the changes?", a
  dialog), and only then closes.
- `role="dialog"`, `aria-modal`, named by its title (`aria-labelledby`), described by its
  description. Focus moves into it and returns to the opener. Escape closes only the
  innermost overlay (keep a small overlay stack).

## The dialog

- Centred, `max-w-[400px]`, `bg-surface-1`, `rounded-sheet`, `e3`, on the scrim; scales
  in from 0.96 on the spring (a fade under reduced motion).
- An icon on its wash heads the dialog: `bg-danger-soft text-danger` for a destructive
  question, `bg-warning-soft text-warning` for a warning, `bg-brand-soft text-brand-ink`
  otherwise.
- Title in `text-title`, body in `text-body text-text-2`. The sheet title is
  `text-headline`; the dialog title is one step smaller.

## Buttons in an overlay

- **Secondary at the start, primary at the end.** Write the secondary first in the DOM
  and the primary second, in a flex row: Hebrew and Arabic mirror it with the page.
- **One primary** per overlay.
- **Cancel is `ghost`.** A secondary that does something other than nothing says what:
  "Don't send", "Keep editing".
- **A destructive primary is `destructive` and named by its verb:** "Delete", "Remove the
  item", "Cancel the booking". Never "OK", never "Yes", never "Confirm" for a
  destructive action.
- Labels come from the product's own strings in he/ar/en, not the browser's or a
  library's defaults.
- **A primary that does not fit.** The primary label stays on one line. When it does not
  fit beside the secondary (a long Arabic label on a phone, or large text), the two
  **stack** instead of wrapping: the secondary first, full width, still `ghost`; the
  primary under it, full width, in its own variant. Order and colour do not change.

```tsx
<ConfirmDialog
  isOpen={open}
  onClose={() => setOpen(false)}
  onConfirm={remove}
  variant="destructive"
  title={t.items.deleteTitle}               // "Delete this item?"
  description={<Sentence template={t.items.deleteBody} values={{ name: item.name }} />}
  confirmText={t.items.delete}              // "Delete"
  cancelText={t.common.cancel}
/>
```

```tsx
{/* footer: row when the labels fit, column (secondary first) when they don't */}
<div className={cn('flex gap-3 border-t border-border bg-surface-2 px-6 py-4', stacked ? 'flex-col' : 'flex-row justify-center')}>
  <Button variant="ghost" className={cn(stacked && 'w-full')} onClick={onClose}>{cancelText}</Button>
  <Button variant={destructive ? 'destructive' : 'default'} className={cn('whitespace-nowrap', stacked && 'w-full')} onClick={onConfirm}>{confirmText}</Button>
</div>
```

Measure "fits" in one place in the kit (the label's width at the current text size inside
the button's padding), not per screen.

## Notices (toasts)

A notice reports the result of something the person did. Settled rulings:

| Kind | Stays | Countdown bar | Announced |
|---|---|---|---|
| success | 4 s | no | polite |
| info | 4 s | no | polite |
| warning | 5 s | yes | polite |
| failure (error) | 6 s | yes | **assertive** |
| undo | 5 s | yes | polite |

- **The countdown bar** runs along the bottom from the start edge, only on warning,
  failure and undo. It pauses while the pointer is over the notices. A call that names
  its own longer duration gets a bar that counts that duration.
- **The close button** is a 44 px target with a translated label. Every notice has one,
  so a notice whose timer paused (the window lost focus) never blocks clicks.
- **Look:** the kind's wash over `surface-1` (`success-soft`, `info-soft`,
  `warning-soft`, `danger-soft`) with the `e3` shadow; in dark, `e3`'s `border-strong`
  hairline. No border in the kind's colour, no scrim. An icon disc in the kind's ink with
  its glyph (✓, ✕, i, !) drawn in `on-semantic`. Title `text-body-strong`, description
  `text-meta`.
- **Place:** top-centre in the panel, clear of sheet footers. Width about 420 px.
- **Motion:** comes in 12 px from its side with a 0.96 scale and a fade over `enter`; only
  the success check may overshoot; a failure shakes once (±4 px twice in 120 ms).
  **Reduced motion:** a short fade only. No rise, no pop, no shake, no bar (the timer
  still runs).
- **Words:** say what happened in the past tense, from the person's side ("Item saved",
  "The message wasn't sent. Try again."). All from translations. A failure's message is
  also written into a hidden `aria-live="assertive"` region, because most toast
  libraries have one polite region for all.

Implementation with `sonner` (unstyled, drawn by `globals.css`) is in
[references/notices-implementation.md](references/notices-implementation.md).

## Checklist

- [ ] Input or choice → sheet; plain confirm or notice → dialog. No `fixed inset-0` in screens.
- [ ] Secondary first (start), primary last (end); one primary; Cancel is `ghost`.
- [ ] Destructive primary is `destructive` and named by its verb; no "OK"/"Yes".
- [ ] Labels fit on one line or the row stacks (secondary first); nothing wraps inside a button.
- [ ] Sheet: sticky footer under the content, dirty guard, Escape closes the innermost overlay, focus returns.
- [ ] Notices: 4/4/5/6 s (undo 5 s); bar only on warning, failure, undo; 44 px close; failures assertive.
- [ ] Reduced motion: overlays and notices fade only.
- [ ] Any change to an overlay pattern is applied to every screen in the same batch.
