# Rendering and Platform Boundaries

## Rendering modes

| Mode | Where initial HTML is produced | Per-request compute | Personalization | SEO/startup |
|---|---|---:|---|---|
| CSR | browser after JS | low server | easy after API calls | weakest initial content, simplest browser model |
| SSR | server for each request | highest | possible with careful isolation | strong initial HTML/SEO |
| SSG/prerender | build time | near zero | not request-specific | fastest/cacheable, build-time data only |

Angular can choose modes per server route via `RenderMode.Client`, `Server`, and `Prerender`.

```mermaid
sequenceDiagram
  participant B as Browser
  participant S as Angular server/CDN
  participant A as API
  B->>S: GET /product/42
  S->>A: fetch render data (SSR) or read built HTML (SSG)
  S-->>B: populated HTML + serialized state + JS links
  B->>B: reuse DOM during hydration
  B->>B: register listeners; replay queued events
  B-->>B: application interactive
```

## Browser rendering mechanics

The compiler emits creation/update instructions. Creation allocates hosts/nodes/directives/views and resolves providers. Update evaluates bindings and writes changed values. Embedded views represent control-flow/deferred/template instances. Angular uses a renderer abstraction but browser applications ultimately interact with DOM.

Avoid direct DOM access because it can bypass sanitization, fail under SSR, make testing difficult and force layout. Use templates/host bindings. When necessary, inject `ElementRef`, use render callbacks and clean up observers/listeners.

## Programmatic component rendering

Use `NgComponentOutlet` for template-declared dynamic types or `ViewContainerRef.createComponent` for imperative placement. Top-level overlays may use `createComponent` plus `ApplicationRef.attachView`. Always define provider ownership, input/output bindings, attachment and destruction.

```html
<ng-container *ngComponentOutlet="selectedType(); inputs: selectedInputs()" />
```

Do not reach for dynamic compilation. Choose from trusted compiled component types; user input must never become Angular template code.

## Hydration

Hydration reuses server-rendered DOM rather than destroying/recreating it. Server and client initial structure must match. Invalid HTML, direct DOM mutation before hydration, browser-only branches and non-deterministic content cause mismatches.

Event replay captures supported user events before hydration and replays them afterward. Incremental hydration leaves eligible `@defer` regions dehydrated until `hydrate` triggers fire; Angular 22 enables it by default with `provideClientHydration()`, and it enables event replay automatically. Opt out with `withNoIncrementalHydration()` only for a measured compatibility reason.

## Server-compatible code

- `window`, `document`, `navigator`, `location` may not exist.
- Use platform-specific providers; Angular recommends them over scattered runtime checks.
- `afterNextRender`/`afterEveryRender` are browser-only for DOM work.
- Keep first server and client render deterministic.
- Use per-request factories/tokens for request state; top-level mutable values can leak between requests.
- Register unknown async work with `PendingTasks` so SSR stability waits.

## Transfer cache

During SSR, eligible `HttpClient` GET/HEAD results can be serialized and reused during initial client rendering, then the cache stops after stability. Credential/auth/cookie/no-cache responses are excluded by default. Never broaden transfer caching of personalized data casually: serialized data becomes part of HTML and intermediary caches may share it.

## Official sources

- [Server and hybrid rendering](https://angular.dev/guide/ssr)
- [Hydration](https://angular.dev/guide/hydration)
- [Incremental hydration](https://angular.dev/guide/incremental-hydration)
- [Programmatic rendering](https://angular.dev/guide/components/programmatic-rendering)

