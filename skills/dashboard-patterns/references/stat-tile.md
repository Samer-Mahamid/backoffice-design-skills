# Stat tile code

A working shape of the tile on the token classes, framer-motion for the count, and the
kit's `Icon`, `Skeleton`, `ErrorState`.

## Figure formatting

```ts
/** Western digits with thousands commas in every language. */
const NUMBER = new Intl.NumberFormat('en-US');

/** A figure with its unit, the minus leading the unit: "-₪1,200", never "₪-1,200". */
export function tileFigure(value: number, prefix = '', suffix = ''): string {
  const digits = NUMBER.format(Math.abs(value));
  return `${value < 0 && digits !== '0' ? '-' : ''}${prefix}${digits}${suffix}`;
}
```

## Count-up

```ts
import { animate } from 'framer-motion';

/**
 * Counts from the number on screen (0 the first time) to the value over the count duration.
 * A new value mid-count carries on from where it got to. Keeps the value's own decimals.
 * Under reduced motion it is simply the value.
 */
export function useCountUp(value: number | null) {
  const reduced = usePrefersReducedMotion();
  const onScreen = useRef(reduced && value !== null ? value : 0);
  const [shown, setShown] = useState(onScreen.current);
  const [counting, setCounting] = useState(false);
  const decimals = value === null ? 0 : Math.min(2, (String(value).split('.')[1] ?? '').length);

  useEffect(() => {
    if (value === null) return;
    if (reduced) { onScreen.current = value; setShown(value); setCounting(false); return; }
    setCounting(true);
    const controls = animate(onScreen.current, value, {
      duration: duration.count, ease: ease.decel,
      onUpdate: (v) => { const at = Number(v.toFixed(decimals)); onScreen.current = at; setShown(at); },
      onComplete: () => setCounting(false),
    });
    return () => controls.stop();
  }, [value, reduced, decimals]);

  return { shown, counting };
}
```

Keep the counting figure in its own small component, so each frame re-renders a few
lines and not the whole card.

## The tile

```tsx
export type StatTone = 'brand' | 'accent' | 'success' | 'warning' | 'danger' | 'info' | 'neutral';
const WELL: Record<StatTone, string> = {
  brand: 'bg-brand-soft text-brand-ink', accent: 'bg-accent-soft text-accent',
  success: 'bg-success-soft text-success', warning: 'bg-warning-soft text-warning',
  danger: 'bg-danger-soft text-danger', info: 'bg-info-soft text-info',
  neutral: 'bg-surface-2 text-text-2',       // nothing to act on (an "owed" tile at nothing owed)
};

export function StatTile({ label, value, text, hidden = false, prefix, suffix, loading = false, error, onRetry,
  delta, spark, icon, tone = 'brand', subtitle, footnote, compact = false }: StatTileProps) {
  const { t } = useI18n();
  const failed = error != null && !loading;
  const { shown, counting } = useCountUp(loading || failed || text !== undefined ? null : value);
  const good = delta?.value != null && (delta.value > 0) === (delta.goodWhen !== 'down');
  const showDelta = !loading && !failed && text === undefined && value !== null && delta?.value != null && delta.value !== 0;

  return (
    <div className="relative flex flex-col gap-3 overflow-hidden rounded-card border border-border bg-surface-1 p-5 shadow-e1 transition-shadow duration-(--duration-base) ease-standard hover:shadow-e2">
      <div className="flex items-start justify-between gap-3">
        <div className="flex min-w-0 flex-col gap-1">
          <p className="text-body text-text-2">{label}</p>
          <p className={cn(compact ? 'text-stat-sm' : 'text-stat', 'text-text-1')} aria-busy={loading || undefined}>
            {loading ? <Skeleton className={cn('inline-block align-middle', compact ? 'h-7 w-20' : 'h-9 w-24')} />
              : failed ? null
              : text !== undefined ? <span dir="auto">{text}</span>       /* "Never", "3 days ago": words, no count */
              : value === null ? <span>—</span>
              : <bdi dir="ltr" className={cn(counting && 'tabular-nums')}>{tileFigure(shown, prefix, suffix)}</bdi>}
          </p>
          {!loading && !failed && text === undefined && value === null && (
            <span className="sr-only">{hidden ? t.kit.notAvailable : t.kit.noFigure}</span>
          )}
          {!loading && !failed && subtitle && <p className="text-meta text-text-2">{subtitle}</p>}
          {!loading && !failed && footnote && <p className="text-meta text-text-2">{footnote}</p>}
          {showDelta && (
            <div className={cn('flex flex-wrap items-center gap-1.5 text-meta transition-opacity duration-(--duration-base)', counting ? 'opacity-0' : 'opacity-100')}>
              <span className={cn('inline-flex items-center gap-1 font-semibold', good ? 'text-success' : 'text-danger')}>
                <Icon icon={delta!.value! > 0 ? TrendingUp : TrendingDown} size={14} />   {/* not in the directional set */}
                <bdi dir="ltr">{delta!.value! > 0 ? '+' : ''}{delta!.value}%</bdi>
              </span>
              <span className="text-text-2">{delta!.period}</span>
            </div>
          )}
        </div>
        {icon && <span className={cn('shrink-0 rounded-control p-2.5', WELL[tone])}><Icon icon={icon} size={22} /></span>}
      </div>
      {failed && <ErrorState kind={errorKind(error)} onRetry={onRetry} compact />}
      {!loading && !failed && spark}
    </div>
  );
}
```

## Tests worth having

- Loading shows a skeleton and no digit; `null` shows "—" and a screen-reader note.
- `-1200` with `₪` renders `-₪1,200` inside a `dir="ltr"` element, in he, ar and en.
- A rise on a `goodWhen: 'down'` metric is `text-danger`; the trend icon has no
  `rtl:-scale-x-100`.
- A 0 or missing delta renders no delta row.
- An error renders the compact error state with Retry; the other tiles on the page still
  render their figures.
- Arabic output has no Arabic-Indic digits.

## Charts in RTL with Recharts

```tsx
const rtl = dir === 'rtl';
<LineChart data={rows}>
  <CartesianGrid stroke="var(--color-grid)" vertical={false} />
  <XAxis dataKey="day" reversed={rtl} tick={{ fill: 'var(--color-text-2)', fontSize: 12 }} stroke="var(--color-axis)" />
  <YAxis orientation={rtl ? 'right' : 'left'} tick={{ fill: 'var(--color-text-2)', fontSize: 12 }} stroke="var(--color-axis)" />
  <Line dataKey="previous" stroke="var(--color-chart-6)" strokeDasharray="4 4" dot={false} />
  <Line dataKey="current" stroke="var(--color-chart-1)" strokeWidth={2} dot={false} />
  <Tooltip contentStyle={{ background: 'var(--color-surface-1)', boxShadow: 'var(--shadow-e3)', textAlign: 'start' }} />
</LineChart>
```

`orientation="right"` here is a library prop naming a physical side, so it is written per
direction; the ratchet counts class names and CSS, not this.
