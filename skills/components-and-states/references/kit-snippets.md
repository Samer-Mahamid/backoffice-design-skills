# Kit snippets

Working shapes of the core kit pieces, on the token classes from
`design-tokens-and-theme`. Adapt names to the project; keep the behaviour.

`cn` is `clsx` + `tailwind-merge`. `useI18n()` returns `{ t, language, dir }`.
`useMotion()` returns the motion tokens and `reduced`.

## Button

```tsx
import { cva, type VariantProps } from 'class-variance-authority';
import { Loader2 } from 'lucide-react';
import * as React from 'react';
import { Icon } from '@/components/ui/icon';
import { cn } from '@/lib/utils';

export const buttonVariants = cva(
  'inline-flex items-center justify-center gap-2 rounded-control text-body-strong font-semibold ' +
  'transition-[color,background-color,border-color,box-shadow,opacity,scale] duration-(--duration-base) ease-standard ' +
  'focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-focus-ring ' +
  'disabled:pointer-events-none disabled:opacity-50 active:scale-[0.98] motion-reduce:active:scale-100',
  {
    variants: {
      variant: {
        default: 'bg-brand text-on-brand shadow-e1 hover:bg-brand-hover',
        destructive: 'bg-danger text-on-semantic shadow-e1 hover:bg-danger-hover',
        outline: 'border border-border bg-surface-1 text-text-1 shadow-e1 hover:bg-surface-2',
        secondary: 'bg-surface-2 text-text-1 hover:bg-border',
        ghost: 'text-text-2 hover:bg-surface-2 hover:text-text-1',
        link: 'text-brand-ink underline-offset-4 hover:underline p-0 h-auto',
        success: 'bg-success text-on-semantic shadow-e1 hover:bg-success-hover',
      },
      size: { default: 'h-10 px-4 py-2', sm: 'h-8 px-3 text-meta', lg: 'h-12 px-6', icon: 'size-10', 'icon-sm': 'size-8' },
    },
    defaultVariants: { variant: 'default', size: 'default' },
  },
);

export interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement>, VariantProps<typeof buttonVariants> {
  loading?: boolean;
  icon?: React.ReactNode;
}

export const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, loading, icon, children, disabled, ...props }, ref) => (
    <button ref={ref} className={cn(buttonVariants({ variant, size, className }))} disabled={disabled || loading} {...props}>
      {/* the one turning mark a button may show; held still under reduced motion in globals.css */}
      {loading ? <Icon icon={Loader2} size={16} className="ring-indeterminate" /> : icon ? <span className="shrink-0">{icon}</span> : null}
      {children}
    </button>
  ),
);
Button.displayName = 'Button';
```

## Input

```tsx
export const Input = React.forwardRef<HTMLInputElement, React.InputHTMLAttributes<HTMLInputElement> & { error?: boolean; icon?: React.ReactNode }>(
  ({ className, error, icon, ...props }, ref) => (
    <div className="relative">
      {icon && <span className="pointer-events-none absolute start-3 top-1/2 -translate-y-1/2 text-text-3">{icon}</span>}
      <input
        ref={ref}
        aria-invalid={error || undefined}
        className={cn(
          'flex h-10 w-full rounded-control border bg-surface-1 px-3 py-2 text-body text-text-1 placeholder:text-text-3',
          'transition-colors duration-(--duration-fast) ease-standard focus-visible:outline-2 focus-visible:outline-offset-2',
          'disabled:cursor-not-allowed disabled:opacity-50 disabled:bg-surface-2',
          error ? 'border-danger focus-visible:outline-danger' : 'border-border-strong focus:border-brand focus-visible:outline-focus-ring',
          icon && 'ps-10',
          className,
        )}
        {...props}
      />
    </div>
  ),
);
```

A field row:

```tsx
<div className="space-y-1.5">
  <label htmlFor={id} className="text-body-strong text-text-1">{t.form.email}</label>
  <Input id={id} type="email" dir="ltr" error={!!err} aria-describedby={`${id}-note`} />
  <p id={`${id}-note`} className={cn('text-meta', err ? 'text-danger' : 'text-text-2')}>{err ?? t.form.emailHint}</p>
</div>
```

## StatusPill

```tsx
export type Tone = 'success' | 'warning' | 'danger' | 'info' | 'brand' | 'neutral';
const TONES: Record<Tone, string> = {
  success: 'bg-success-soft text-success',
  warning: 'bg-warning-soft text-warning',
  danger: 'bg-danger-soft text-danger',
  info: 'bg-info-soft text-info',
  brand: 'bg-brand-soft text-brand-ink',
  neutral: 'bg-surface-2 text-text-2',
};

/** Known statuses: each one's tone and icon, in one place. Labels come from translations. */
export const STATUSES = {
  confirmed: { tone: 'success', icon: CircleCheck },
  pending: { tone: 'warning', icon: Clock },
  waiting: { tone: 'info', icon: Hourglass },
  cancelled: { tone: 'neutral', icon: CircleX },
  overdue: { tone: 'danger', icon: CircleAlert },
} as const satisfies Record<string, { tone: Tone; icon: LucideIcon }>;

export function StatusPill({ tone = 'neutral', icon, dot, label, className }: { tone?: Tone; icon?: LucideIcon; dot?: boolean; label: React.ReactNode; className?: string }) {
  return (
    <span className={cn('inline-flex items-center gap-1 whitespace-nowrap rounded-pill px-2 py-0.5 text-label', TONES[tone], className)}>
      {icon ? <Icon icon={icon} size={12} /> : dot ? <span aria-hidden className="size-1.5 rounded-pill bg-current" /> : null}
      {label}
    </span>
  );
}
```

Pick tones by meaning: a live, good state speaks (`success`, `info`); finished or
inactive states are quiet (`neutral`); only a problem the reader must act on is `danger`.

## Skeleton

```tsx
export function Skeleton({ variant = 'line', count = 3, className }: { variant?: 'line' | 'circle' | 'card' | 'list'; count?: number; className?: string }) {
  if (variant === 'list') {
    return (
      <div aria-hidden className={cn('flex flex-col gap-3', className)}>
        {Array.from({ length: count }, (_, i) => (
          <div key={i} className="flex items-center gap-3">
            <span className="skeleton block size-9 shrink-0 rounded-pill" />
            <div className="flex flex-1 flex-col gap-2">
              <span className="skeleton block h-3 w-2/3 rounded-badge" />
              <span className="skeleton block h-3 w-1/3 rounded-badge" />
            </div>
          </div>
        ))}
      </div>
    );
  }
  const shape = { line: 'h-3 w-full rounded-badge', circle: 'size-10 rounded-pill', card: 'h-28 w-full rounded-card' }[variant];
  return <span aria-hidden className={cn('skeleton block', shape, className)} />;
}
```

```css
/* globals.css: surface-2 blocks with a sweep from the start edge to the end */
.skeleton { position: relative; overflow: hidden; background: var(--color-surface-2); }
.skeleton::after {
  content: ""; position: absolute; inset: 0;
  background: linear-gradient(90deg, transparent, color-mix(in srgb, var(--color-surface-1) 60%, transparent), transparent);
  animation: sweep var(--duration-shimmer) linear infinite;
}
@keyframes sweep { from { transform: translateX(-100%); } to { transform: translateX(100%); } }
[dir="rtl"] .skeleton::after { animation-direction: reverse; }
@media (prefers-reduced-motion: reduce) { .skeleton::after { display: none; } }
```

## ErrorState and errorKind

```tsx
export type ErrorKind = 'offline' | 'refused' | 'notFound' | 'failed';

export function errorKind(error: unknown): ErrorKind {
  const e = error as { response?: { status?: number }; status?: number; request?: unknown; code?: string; message?: string } | null;
  const status = e?.response?.status ?? e?.status ?? Number(/status code (\d{3})/.exec(e?.message ?? '')?.[1] ?? NaN);
  if (status === 403) return 'refused';
  if (status === 404) return 'notFound';
  if (e?.code === 'ECONNABORTED' || e?.code === 'ETIMEDOUT') return 'failed';      // a slow server, not offline
  if (Number.isNaN(status) && (e?.code === 'ERR_NETWORK' || (e?.request !== undefined && !e?.response)
      || (typeof navigator !== 'undefined' && navigator.onLine === false))) return 'offline';
  return 'failed';
}

export function ErrorState({ kind = 'failed', title, detail, onRetry, compact }: { kind?: ErrorKind; title?: string; detail?: string; onRetry?: () => void; compact?: boolean }) {
  const { t } = useI18n();
  const copy = t.kit.errors[kind];                       // { title, detail } in he/ar/en
  const look = LOOK[kind];                               // icon + well classes, see SKILL.md table
  const retry = onRetry && (kind === 'offline' || kind === 'failed');
  return (
    <div role="alert" className={cn('flex flex-col items-center text-center', compact ? 'gap-2 px-3 py-4' : 'gap-3 px-6 py-12')}>
      <span className={cn('rounded-card', compact ? 'p-2.5' : 'p-4', look.well)}><Icon icon={look.icon} size={compact ? 20 : 28} /></span>
      <h3 className={cn(compact ? 'text-body-strong' : 'text-title', 'text-text-1')}>{title ?? copy.title}</h3>
      <p className={cn('max-w-sm text-text-2', compact ? 'text-meta' : 'text-body')}>{detail ?? copy.detail}</p>
      {retry && (
        <button type="button" onClick={onRetry}
          className="mt-1 inline-flex min-h-11 items-center gap-2 rounded-control border border-border-strong bg-surface-1 px-4 text-body-strong text-text-1 hover:bg-surface-2 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-focus-ring">
          <Icon icon={RotateCw} size={16} />{t.kit.retry}
        </button>
      )}
    </div>
  );
}
```

## EmptyState

```tsx
export function EmptyState({ icon = Inbox, title, description, action }: { icon?: LucideIcon; title: string; description?: string; action?: React.ReactNode }) {
  return (
    <div className="flex flex-col items-center justify-center px-6 py-16 text-center">
      <div className="mb-5 rounded-card bg-surface-2 p-5 text-text-3"><Icon icon={icon} size={40} /></div>
      <h3 className="mb-1 text-title text-text-1">{title}</h3>
      {description && <p className="mb-6 max-w-sm text-body text-text-2">{description}</p>}
      {action}
    </div>
  );
}
```

The decorative icon may be `text-3`; the words are `text-1` and `text-2`.

## AsyncView

```tsx
export interface AsyncState<T> { data?: T; error?: unknown; isLoading?: boolean; mutate?: () => unknown }

export function AsyncView<T>({ state, skeleton, children, isEmpty, empty, onRetry, errorTitle, errorDetail, className }: {
  state: AsyncState<T>; skeleton: React.ReactNode; children: (data: T) => React.ReactNode;
  isEmpty?: (data: T) => boolean; empty?: React.ReactNode; onRetry?: () => void;
  errorTitle?: string; errorDetail?: string; className?: string;
}) {
  const { t } = useI18n();
  const last = React.useRef<T | undefined>(undefined);
  if (state.data !== undefined) last.current = state.data;

  if (state.error !== undefined && state.data === undefined) {
    const kind = errorKind(state.error);
    const own = kind === 'failed';
    return <ErrorState kind={kind} title={own ? errorTitle : undefined} detail={own ? errorDetail : undefined}
      onRetry={onRetry ?? (state.mutate ? () => void state.mutate?.() : undefined)} />;
  }

  const held = state.data === undefined && state.isLoading === true;   // a new request, an old answer on screen
  const shown = state.data ?? (held ? last.current : undefined);
  if (shown === undefined) {
    return (
      <div className={className}>
        <div aria-busy="true">{skeleton}</div>
        <span role="status" className="sr-only">{t.kit.loading}</span>
      </div>
    );
  }
  const body = isEmpty?.(shown) ? (empty ?? <EmptyState title={t.kit.empty} />) : children(shown);
  return (
    <div aria-busy={held || undefined} className={cn('transition-opacity duration-(--duration-base) ease-standard', held && 'opacity-60', className)}>
      {body}
    </div>
  );
}
```

With SWR, a changed key with `keepPreviousData` off gives `data: undefined,
isLoading: true`; the ref keeps the last answer on screen meanwhile.

## DataTable row behaviour

```tsx
{rows.map((row, i) => {
  const opens = !!onRowClick && (rowOpens ? rowOpens(row) : true);
  return (
    <tr key={keyOf(row)}
        className={cn('transition-colors duration-(--duration-base) ease-standard hover:bg-surface-2', opens && 'group/row cursor-pointer')}
        onClick={opens ? () => onRowClick?.(row) : undefined}>
      {columns.map((col) => (
        <td key={col.key} className={cn('px-4 py-3.5 text-body text-text-1', col.className)}>
          {col.render ? col.render(row, i) : row[col.key]}
        </td>
      ))}
    </tr>
  );
})}

// a cell that names the row, underlined on row hover
render: (row) => <span className="text-body-strong group-hover/row:underline">{row.name}</span>
// a control in a row
render: (row) => <Button size="sm" variant="ghost" onClick={(e) => { e.stopPropagation(); act(row); }}>{t.items.act}</Button>
```

## Tabs keyboard

```ts
const rtl = (e.currentTarget.closest('[dir]') as HTMLElement | null)?.dir === 'rtl';
const forward = rtl ? 'ArrowLeft' : 'ArrowRight';
const back = rtl ? 'ArrowRight' : 'ArrowLeft';
// forward → next tab, back → previous, Home → first, End → last; the tab moved to is chosen.
// Only the chosen tab is in the tab order (tabIndex 0); the rest are -1.
// Each tab: role="tab" aria-selected aria-controls; each panel: role="tabpanel" aria-labelledby.
```
