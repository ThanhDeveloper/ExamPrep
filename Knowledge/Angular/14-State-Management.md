# State Management and Async Interoperability

State management is ownership plus transitions, lifetime, derivation, effects, persistence and synchronization—not a library choice.

## State taxonomy

| State | Preferred owner |
|---|---|
| ephemeral UI (`expanded`, selected tab) | component signal |
| derived UI | `computed` |
| child-editable value | component `model` when two-way API is appropriate |
| feature workflow | feature/route-scoped service with private writable signals |
| filters/paging that must survive/share | router URL |
| server entity/query | backend as authority; resource/RxJS client cache with invalidation |
| event/time/concurrency stream | RxJS |
| cross-application durable state | root service only when truly app-wide |

## Minimal signal store

```ts
interface UsersState {
  readonly users: readonly User[];
  readonly loading: boolean;
  readonly error: string | null;
}

@Service({autoProvided: false})
export class UsersStore {
  private readonly api = inject(UsersApi);
  private readonly state = signal<UsersState>({users: [], loading: false, error: null});
  readonly users = computed(() => this.state().users);
  readonly loading = computed(() => this.state().loading);

  load() {
    this.state.update(s => ({...s, loading: true, error: null}));
    this.api.list('').pipe(take(1)).subscribe({
      next: users => this.state.set({users, loading: false, error: null}),
      error: e => this.state.update(s => ({...s, loading: false, error: String(e)})),
    });
  }
}
```

Provide it at the users route for a feature lifetime. For parameter-driven reads, a `resource` or RxJS pipeline avoids manual concurrency; commands still need explicit outcomes.

## Encapsulation

Expose read-only state and named commands. A public writable signal lets every consumer mutate without invariant, audit or ownership. `asReadonly()` prevents `.set/.update` through the public type but not deep object mutation, so use immutable values/conventions too.

## State versus server cache

Client workflow state and server state have different problems. Server state needs freshness, request deduplication, invalidation after mutations, optimistic update/rollback, retry and offline policy. A normalized client cache is valuable when many views edit/reference the same entities; it adds mapping and invalidation complexity when screens are independent.

Normalized example:

```ts
interface EntityState<T> {
  ids: readonly string[];
  entities: Readonly<Record<string, T>>;
}
```

Selectors derive views; actions describe events/commands; reducers make deterministic transitions; effects connect external I/O. Those terms are general/NgRx-style architecture, not requirements of core Angular.

## When an external store helps

Use one when many teams/features share complex state transitions, audit/replay/devtools conventions matter, effects/concurrency need standardization, or normalized entity relationships are extensive. Do not add it merely because the application is large. Core signals + DI services often suffice.

## Promise versus Observable versus Signal

| Primitive | Best model | Cancellation/time | Angular use |
|---|---|---|---|
| Promise | one eventual outcome | no native unsubscribe; `AbortSignal` by convention | `async` imperative operation, router/bootstrap APIs |
| Observable | 0..N notifications | subscription teardown; operators | HTTP, events, form/router streams |
| Signal | current synchronously readable value | not a stream | UI/current/derived state |

An HTTP call can be an Observable; its current loading/value/error representation can be a resource or signals. These are complementary layers.

## Interop rules

- `toSignal(source$, {initialValue})` subscribes immediately; create once and reuse.
- `toSignal(..., {requireSync: true})` is appropriate only when the source must emit synchronously.
- errors from the Observable are thrown on signal read unless handled upstream; completion leaves the last value.
- `toObservable(signal)` uses an effect and may coalesce multiple writes before synchronization.
- both normally need an injection context or explicit injector.
- `outputFromObservable` forwards next notifications; handle source errors yourself.
- `outputToObservable` turns `OutputRef` events into a stream.

## Avoid effect-based stores

Bad:

```ts
effect(() => this.total.set(this.price() * this.quantity()));
```

Good:

```ts
readonly total = computed(() => this.price() * this.quantity());
```

Effects are appropriate at imperative edges, not as a default reducer/derivation mechanism.

## Official sources

- [Signals](https://angular.dev/guide/signals)
- [Resources](https://angular.dev/guide/signals/resource)
- [RxJS interop](https://angular.dev/ecosystem/rxjs-interop)
- [Router state](https://angular.dev/guide/routing/read-route-state)

