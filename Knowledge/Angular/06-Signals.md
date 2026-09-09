# Signals and Asynchronous Resources

Signals are Angular's fine-grained primitive for representing and deriving current state. A signal is a callable getter; invoking it in a reactive context records a producer→consumer edge.

## `signal`, `set`, `update`, readonly

**Input:** count state and an increment action.

```ts
const count = signal(1);
const publicCount = count.asReadonly();

count.set(2);                 // replace with an exact value
count.update(value => value + 1); // calculate from current value
console.log(publicCount());
```

**Output:** `3`.  
**Why:** both APIs update the same writable signal; `asReadonly` hides mutation methods but is not a deep immutability boundary.

Signals use referential equality (`Object.is`) by default. Mutating an object without setting a new value neither communicates ownership nor necessarily notifies dependents:

```ts
users.update(xs => xs.map(x => x.id === id ? {...x, active: true} : x));
```

Custom equality can suppress notifications, but deep comparison can cost more than the work it avoids and can hide changes.

## `computed`: lazy, memoized derived state

```ts
const price = signal(100);
const quantity = signal(2);
const total = computed(() => price() * quantity());

console.log(total()); // 200
quantity.set(3);
console.log(total()); // 300
```

The derivation does not eagerly recalculate on every write. A dependency change invalidates the cached value; the next read recomputes, then caches it. Computed signals are read-only. Use them for derivation, filtering, aggregation and view models.

## Dynamic dependency tracking

```ts
const showCount = signal(false);
const count = signal(0);
const message = computed(() => showCount() ? `Count: ${count()}` : 'Hidden');
```

Initially only `showCount` is read. Changing `count` does not invalidate `message`. After `showCount.set(true)` and a read, `count` becomes a dependency. If the branch later stops reading it, the edge is removed. Dependencies reflect the most recent synchronous execution, not all possible branches.

## Reactive context and `untracked`

`computed`, `linkedSignal`, effects, resource parameters/loaders and template/host evaluation establish reactive contexts. Tracking is synchronous; reads after `await` are not tracked.

```ts
effect(() => {
  const user = currentUser();
  console.log(user.name, untracked(counter));
});
```

Only `currentUser` should retrigger the effect. Use `untracked` sparingly; it changes the dependency graph and can create stale behavior.

## `effect`: side effects, not derivation

```ts
constructor() {
  effect(onCleanup => {
    const id = this.userId();
    const timer = setTimeout(() => this.audit.log(id), 500);
    onCleanup(() => clearTimeout(timer));
  });
}
```

Effects run at least once and track dependencies dynamically. Component effects participate in synchronization; root effects run as microtasks. They require an injection context unless passed an injector and are normally destroyed with that context. Use for imperative non-reactive boundaries: logging, storage, canvas or third-party widgets. Prefer `computed` for derived values and `linkedSignal` for writable dependent values. State propagation through effects risks cycles, extra passes and `ExpressionChangedAfterItHasBeenCheckedError`.

## `linkedSignal` — ✅ stable since v20

It is writable state whose default/reset value is derived reactively.

```ts
const options = signal([{id: 1}, {id: 2}]);
const selected = linkedSignal({
  source: options,
  computation: (next, previous) =>
    next.find(x => x.id === previous?.value.id) ?? next[0],
});

selected.set(options()[1]);
options.set([{id: 2}, {id: 3}]);
console.log(selected().id); // 2
```

Use it when a user-overridable selection must remain valid as its option set changes. A `computed` cannot be written; a plain signal does not reset with dependencies.

## Signals in components

- Reading a signal in a template registers that view as a consumer.
- Updating it notifies Angular to schedule/check the necessary view.
- `input`, `model`, signal queries and resource state are signals.
- Expose `Signal<T>`/`asReadonly()` from services; keep `WritableSignal<T>` private.

```ts
@Service()
export class CartStore {
  private readonly _items = signal<readonly Item[]>([]);
  readonly items = this._items.asReadonly();
  readonly total = computed(() => this._items().reduce((n, x) => n + x.price, 0));
  add(item: Item) { this._items.update(xs => [...xs, item]); }
}
```

## Resource APIs — ✅ stable since v22

`resource` maps reactive parameters to an abortable Promise loader and exposes `value`, `status`, `error`, `isLoading`, `snapshot`, `hasValue`, `reload` and writable local value behavior. It is intended for reads, not mutations; a parameter change/destroy can abort a pending load.

```ts
userId = input.required<string>();
user = resource({
  params: () => ({id: this.userId()}),
  loader: ({params, abortSignal}) =>
    fetch(`/api/users/${params.id}`, {signal: abortSignal}).then(r => {
      if (!r.ok) throw new Error(`HTTP ${r.status}`);
      return r.json() as Promise<User>;
    }),
});
```

**Input:** `userId`. **Output:** reactive loading/value/error state. **Why:** parameter changes invalidate the asynchronous dependency and stale loads are canceled. For SSR caching, a stable `id` may reuse a server result; never serialize user-specific data into shared/cached HTML carelessly.

`httpResource` is the HttpClient-backed version: it supports interceptors/testing and returns response metadata. Use for reactive reads; use `HttpClient` commands for POST/PUT/PATCH/DELETE mutations.

`rxResource` accepts an Observable-producing stream/loader and is stable since v22. Its stream must emit or error before completing.

## Signal versus Observable

| Dimension | Signal | Observable |
|---|---|---|
| Model | current value/state | notifications over time |
| Read | synchronous pull getter | values pushed to subscribers |
| Initial/current value | always has a readable state (possibly `undefined` by type) | may emit zero values or much later |
| Composition | `computed` dependency graph | rich time/concurrency operators |
| Execution | reads do not subscribe to external work | often lazy; each subscription can start work |
| Multiplicity | naturally shared value | cold/unicast or hot/multicast depending on source |
| Errors/completion | not normal signal channels; resources expose state | first-class error and completion notifications |
| Cancellation | not a base signal concept | subscription teardown; flattening can cancel inner streams |
| Best use | UI/local/derived state | events, HTTP, WebSocket, timing and async workflows |

Do not say “signals replace RxJS.” Signals improve state modeling; RxJS remains better for temporal/concurrent streams. Resources bridge a common asynchronous-state use case.

## RxJS interoperability

```ts
query = signal('');
results$ = toObservable(this.query).pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(q => this.http.get<Result[]>('/api/search', {params: {q}})),
);
results = toSignal(this.results$, {initialValue: []});
```

`toSignal` subscribes immediately and normally cleans up with its injection context. Reuse the result; calling it repeatedly creates subscriptions. `requireSync` asserts synchronous first emission. Observable errors are thrown when reading the signal unless handled upstream.

`toObservable` uses an effect. Because effects are scheduled, several signal writes before stabilization can collapse to the final emitted value. It needs an injection context or explicit injector.

`outputFromObservable` and `outputToObservable` are stable since v19. `takeUntilDestroyed` is the standard bridge for imperative subscriptions.

## Modern API status

| API | Introduced/stabilized | Status/recommendation |
|---|---|---|
| Signals | introduced v16; core APIs later stabilized | ✅ current state primitive |
| `input`/`output`/`model` | production-ready/stable v19 | ✅ component authoring |
| Signal queries | production-ready/stable v19 | ✅ prefer for new queries |
| `linkedSignal` | stable v20 | ✅ writable dependent state |
| `resource`/`httpResource`/`rxResource` | stable v22 | ✅ async reads with explicit state; not mutations |
| `debounced` | v22-era API | 🧪 experimental; RxJS debouncing remains stable |

## Prediction: memoization

```ts
const count = signal(1);
const doubled = computed(() => { console.log('computed'); return count() * 2; });
console.log(doubled());
console.log(doubled());
count.set(2);
console.log(doubled());
```

Output: `computed, 2, 2, computed, 4`. The second read uses cache; `set` invalidates but recomputation waits until read.

## Official sources

- [Signals overview](https://angular.dev/guide/signals)
- [Effects](https://angular.dev/guide/signals/effect)
- [`linkedSignal`](https://angular.dev/guide/signals/linked-signal)
- [Resources](https://angular.dev/guide/signals/resource)
- [`resource` API and mutation warning](https://angular.dev/api/core/resource)
- [RxJS interop](https://angular.dev/ecosystem/rxjs-interop)

