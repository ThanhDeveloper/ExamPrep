# Performance and `@defer`

Performance work starts with measurement: Angular DevTools/Chrome DevTools identify synchronization cycles, component costs, long tasks, layout and network waterfalls. Optimize the actual bottleneck.

## Performance layers

| Layer | High-value techniques |
|---|---|
| startup/network | route lazy loading, `@defer`, tree shaking, ESM, bundle budgets, image optimization |
| rendering | stable `@for track`, limit DOM, virtualize/paginate, avoid layout thrashing |
| computation | OnPush default, zoneless default, `computed`, pure pipes, cheap templates |
| async/data | cancellation, dedupe with explicit cache policy, batching/backpressure |
| server/hybrid | choose CSR/SSR/SSG per route, hydration, safe transfer cache |

## `@for track`

```html
@for (user of users(); track user.id) {
  <app-user [user]="user" />
}
```

The key maps logical items to existing views/DOM. Without a stable meaningful key, refetched objects can look entirely new, causing node/component recreation, lost focus/input state and excess work. `$index` is incorrect for reorderable/insertable lists. A changing/duplicate key is not identity.

For 10,000 rows, tracking alone does not reduce DOM size. Add viewport virtualization, server paging/infinite loading, incremental rendering, simplified row templates and measured aggregation.

## Expensive template work

Bad:

```html
<p>{{ calculatePortfolioTotal(positions()) }}</p>
```

Good:

```ts
readonly total = computed(() => calculatePortfolioTotal(this.positions()));
```

```html
<p>{{ total() }}</p>
```

Angular evaluates bindings synchronously. `computed` caches until dependencies invalidate. Pure pipes are another option for reusable stateless transformation.

## `@defer`

```html
@defer (on viewport; prefetch on idle) {
  <analytics-chart />
} @placeholder (minimum 300ms) {
  <div class="chart-skeleton" aria-label="Chart loading"></div>
} @loading (after 100ms; minimum 300ms) {
  <p>Loading chart…</p>
} @error {
  <p>Chart failed to load.</p>
}
```

**Input:** idle time can prefetch code; viewport entry triggers display. **Output:** chart dependencies are a lazy chunk and placeholders cover states. **Why:** prefetch separates network timing from render timing.

### Triggers

| Trigger | Fires |
|---|---|
| `idle` | browser idle; default; optional timeout |
| `viewport` | observed placeholder/reference enters viewport |
| `interaction` | click/keydown on placeholder/reference |
| `hover` | mouseover/focusin |
| `immediate` | after non-deferred content renders |
| `timer(x)` | after duration |
| `when expression` | first time expression becomes truthy; does not revert |

Multiple triggers are OR conditions. `prefetch on/when` loads code without rendering main content.

### What really splits

The dependency must be standalone and only referenced inside defer blocks in that file. A query, outside reference or barrel/static import relationship can make it eager. Placeholder/loading/error dependencies are eager. Transitive dependencies may come through NgModules.

Default SSR/SSG renders placeholder/nothing and activates triggers in the client. With incremental hydration and `hydrate` triggers, main content can render on the server and stay dehydrated until its trigger. Avoid deferring above-the-fold/LCP content and reserve dimensions to protect CLS. Nested blocks with identical triggers cause cascading request bursts.

## Images

Use `NgOptimizedImage`, provide width/height or a correctly positioned `fill` container, mark only actual LCP images `priority`, use responsive sizing/CDN loaders, and lazy load non-critical images. It helps generate `srcset`, priority/preconnect/preload behavior and warns about distortion/layout problems.

## Large dashboards and WebSockets

- Coalesce/buffer high-frequency updates before rendering.
- Partition state by widget; avoid rebuilding whole object graphs for one metric without reason.
- Use stable keys and incremental aggregation.
- Render at a human-visible cadence (for example animation frame), not every server message.
- Pause hidden/offscreen feeds if semantics permit.
- Profile memory and subscription lifetimes.

## Polling and repeated HTTP

Use `timer(...).pipe(switchMap(...))` when a new poll should replace stale work; use exhaust semantics when overlapping must be suppressed. Pause on hidden/offline if appropriate; add jitter/backoff; share only within an explicit cache scope; stop on destroy.

## Tree shaking and code splitting

Tree shaking removes statically unreachable code; lazy loading moves reachable code into later chunks. A root provider that is tree-shakable can still be in the initial chunk if injected eagerly. Barrel imports, side effects and CommonJS dependencies reduce optimization. Inspect build output/budgets rather than assuming.

## Performance anti-checklist

- Do not call `detectChanges` in loops.
- Do not make every pipe impure.
- Do not memoize without bounded cache/invalidations.
- Do not add `shareReplay` everywhere.
- Do not lazy-load tiny code into a network waterfall.
- Do not optimize change detection while 10,000 DOM nodes remain the bottleneck.
- Do not equate SSR with universally faster interaction; measure server latency, JS and hydration cost.

## Official sources

- [Performance overview](https://angular.dev/best-practices/performance)
- [Chrome profiling](https://angular.dev/best-practices/profiling-with-chrome-devtools)
- [Slow computations](https://angular.dev/best-practices/slow-computations)
- [`@defer`](https://angular.dev/guide/templates/defer)
- [Image optimization](https://angular.dev/best-practices/performance/image-optimization)

