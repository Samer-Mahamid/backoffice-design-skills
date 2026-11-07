# Notices with sonner

`sonner` places, stacks, times and dismisses the notices; the kit's CSS draws them. Mount
one themed toaster in the shell.

## Durations and the assertive region

```tsx
'use client';
import { useEffect, useState, type CSSProperties } from 'react';
import { Toaster, toast } from 'sonner';

export const NOTICE_MS = { success: 4000, info: 4000, warning: 5000, error: 6000, undo: 5000 } as const;
const NOTICE_ALERT_MS = 1000; // how long a failure's words stay in the assertive region

type Show = typeof toast.error;
const WRAPPED = Symbol.for('app.notice.duration');
const region: { write?: (message: string) => void } = {};

/**
 * [show] with its kind's default duration. A call that names its own duration keeps it, and
 * the notice carries it as --notice-ms so the countdown empties exactly when it goes.
 * With [assertive], a message given as words also goes to the assertive region.
 */
export function withDuration(show: Show, ms: number, { assertive = false } = {}): Show {
  if ((show as Show & { [WRAPPED]?: number })[WRAPPED]) return show;   // wrap once, however often the module loads
  const wrapped = ((message: Parameters<Show>[0], data?: Parameters<Show>[1]) => {
    if (assertive && typeof message === 'string') region.write?.(message);
    const duration = data?.duration ?? ms;
    const style = { '--notice-ms': `${duration}ms`, ...data?.style } as CSSProperties;
    return show(message, { ...data, duration, style });
  }) as Show;
  return Object.assign(wrapped, { [WRAPPED]: ms });
}

toast.error = withDuration(toast.error, NOTICE_MS.error, { assertive: true });
toast.warning = withDuration(toast.warning, NOTICE_MS.warning);

export function ThemedToaster({ theme }: { theme: 'light' | 'dark' }) {
  const { t } = useI18n();
  const [said, setSaid] = useState({ message: '', n: 0 });

  useEffect(() => {
    let clear: ReturnType<typeof setTimeout> | undefined;
    const write = (message: string) => {
      clearTimeout(clear);
      setSaid((s) => ({ message, n: s.n + 1 }));          // a new node each time: the same failure twice is read twice
      clear = setTimeout(() => setSaid((s) => ({ ...s, message: '' })), NOTICE_ALERT_MS);
    };
    region.write = write;
    return () => { clearTimeout(clear); if (region.write === write) region.write = undefined; };
  }, []);

  return (
    <>
      <Toaster
        theme={theme}
        position="top-center"
        closeButton
        duration={NOTICE_MS.success}
        icons={{ success: <NoticeIcon kind="success" />, error: <NoticeIcon kind="error" />, info: <NoticeIcon kind="info" />, warning: <NoticeIcon kind="warning" /> }}
        containerAriaLabel={t.notices.region}
        toastOptions={{ unstyled: true, className: 'notice', closeButtonAriaLabel: t.notices.close }}
        style={{ '--width': '420px' } as CSSProperties}
      />
      <div className="sr-only" aria-live="assertive" aria-atomic="true">
        {said.message && <span key={said.n}>{said.message}</span>}
      </div>
    </>
  );
}
```

## The icon

```tsx
const GLYPH = {
  success: 'M5 12.5l4.5 4.5L19 7.5',
  error: 'M7 7l10 10M17 7L7 17',
  info: 'M12 16.5v-5M12 7.6v.1',
  warning: 'M12 13V8M12 16.6v.1',
} as const;

export function NoticeIcon({ kind }: { kind: keyof typeof GLYPH }) {
  return (
    <span className="notice-icon" data-kind={kind} aria-hidden="true">
      <svg viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" strokeWidth="3" strokeLinecap="round" strokeLinejoin="round">
        {/* pathLength makes every glyph 24 long, so one dash animation draws each one fully */}
        <path className="notice-glyph" d={GLYPH[kind]} pathLength={24} />
      </svg>
    </span>
  );
}
```

## The CSS (sketch)

```css
/* Notices: each kind on its wash over surface-1, e3; a bar on warning and error */
.notice {
  position: relative; display: flex; gap: 12px; align-items: flex-start;
  width: var(--width); padding: 12px 16px; border-radius: var(--radius-card);
  background: var(--color-surface-1); color: var(--color-text-1); box-shadow: var(--shadow-e3);
  overflow: hidden;
}
.notice[data-type="success"] { background-image: linear-gradient(var(--color-success-soft), var(--color-success-soft)); }
.notice[data-type="info"]    { background-image: linear-gradient(var(--color-info-soft), var(--color-info-soft)); }
.notice[data-type="warning"] { background-image: linear-gradient(var(--color-warning-soft), var(--color-warning-soft)); }
.notice[data-type="error"]   { background-image: linear-gradient(var(--color-danger-soft), var(--color-danger-soft)); }

.notice-icon { display: grid; place-items: center; inline-size: 24px; block-size: 24px; border-radius: 999px; color: var(--color-on-semantic); }
.notice-icon[data-kind="success"] { background: var(--color-success); }
.notice-icon[data-kind="error"]   { background: var(--color-danger); }
.notice-icon[data-kind="info"]    { background: var(--color-info); }
.notice-icon[data-kind="warning"] { background: var(--color-warning); }

/* the countdown, from the start edge, only where the kind has one */
.notice[data-type="warning"]::after,
.notice[data-type="error"]::after {
  content: ""; position: absolute; inset-inline: 0; inset-block-end: 0; block-size: 3px;
  background: currentColor; opacity: .35;
  transform-origin: left; animation: notice-countdown var(--notice-ms, 5000ms) linear forwards;
}
/* transform-origin has no logical keyword: anchor the bar at the start edge per direction */
[dir="rtl"] .notice::after { transform-origin: right; }
[data-sonner-toaster]:hover .notice::after { animation-play-state: paused; }
@keyframes notice-countdown { from { transform: scaleX(1); } to { transform: scaleX(0); } }

/* the close button: a 44 px target */
.notice [data-close-button] { min-inline-size: 44px; min-block-size: 44px; }

@media (prefers-reduced-motion: reduce) {
  .notice::after { display: none; }       /* no bar; the timer still runs */
  .notice, .notice-icon, .notice-glyph { animation: none !important; }
  .notice { transition: opacity var(--duration-fast) linear !important; }  /* a short fade of its own */
}
```

The bar colour, glyph drawing and shake keyframes are the project's to tune; durations
come from the motion tokens. An undo notice is `toast(message, { action: { label:
t.common.undo, onClick: undo }, duration: NOTICE_MS.undo })` with a class that also
draws the bar.

## Tests worth having

- Each kind shows its icon and colour class; the toaster is mounted with a translated
  region label and close label.
- A failure stays 6 s and a warning 5 s by default; a call's own duration wins and sets
  `--notice-ms`.
- Only warning, failure and undo draw the bar (read the CSS from `globals.css` in the
  test: jsdom applies no stylesheet).
- A failure's words appear in the assertive region, and the same failure twice is a new
  node.
- Under reduced motion the CSS removes the bar and the movement.
