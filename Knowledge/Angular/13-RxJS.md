# RxJS for Angular

RxJS models notifications over time and supplies composition, concurrency, cancellation, error and completion semantics. Angular 22 supports RxJS `^6.5.3 || ^7.4.0`; use the project-compatible version and current pipeable APIs.

## Core types

- **Observable:** lazy description of an execution; a subscription usually starts it.
- **Observer:** `next`, `error`, `complete` consumer.
- **Subscription:** teardown handle; `unsubscribe` cancels/cleans the execution.
- **Subject:** both Observable and Observer; multicasts pushed notifications.

| Type | New subscriber receives | Requires seed | Typical Angular use |
|---|---|---:|---|
| `Subject<T>` | future values only | no | imperative event boundary |
| `BehaviorSubject<T>` | current/latest then future | yes | legacy RxJS-held current state |
| `ReplaySubject<T>` | configured buffered values then future | no | replay with explicit buffer/time |
| `AsyncSubject<T>` | final value only on completion | no | rare; one-result completion cache |

Prefer a private subject and expose `.asObservable()`. Do not expose `.next()` to arbitrary consumers.

## Cold versus hot

Cold sources create independent executions per subscription (`HttpClient` normally sends again). Hot sources exist independently and multicast (DOM events, Subjects). `share`/`shareReplay` can multicast a source but require lifecycle/reset/cache decisions.

## Creation and everyday operators

| Category | Operators | Mental model |
|---|---|---|
| creation | `of`, `from`, `interval`, `timer` | values/iterables/promises/time → Observable |
| transform | `map` | each value → another value |
| filter/rate | `filter`, `debounceTime`, `distinctUntilChanged`, `take`, `takeUntil` | select, delay bursts, stop |
| combine | `combineLatest`, `forkJoin`, `merge`, `concat`, `zip` | latest tuple, final tuple, interleave, sequence, index-pair |
| errors | `catchError`, `retry` | recover/replace/rethrow, resubscribe |
| utility | `tap`, `finalize`, `share`, `shareReplay` | observe side effect, teardown, multicast/cache |

`combineLatest` waits until every source emits, then emits on any source with latest values. `forkJoin` waits for all sources to complete and emits their last values; an infinite source prevents it from emitting. `zip` pairs nth values. `merge` interleaves. `concat` subscribes to the next only after prior completion.

## Higher-order Observables

Mapping a value to an Observable creates `Observable<Observable<T>>`. A flattening operator decides subscription concurrency and cancellation.

```ts
searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => api.search(term)),
);
```

### Flattening comparison

| Operator | Concurrent inners | New outer value while active | Best fit | Risk |
|---|---:|---|---|---|
| `switchMap` | 1 | unsubscribe old, switch to new | search, route-driven reads | cancels work that must finish |
| `mergeMap` | many (configurable) | run concurrently | independent writes/parallel reads | out-of-order results, load |
| `concatMap` | 1 | queue | ordered writes | backlog/stale queued work |
| `exhaustMap` | 1 | ignore new | prevent double-submit/login | drops legitimate latest intent |

For inputs `a → an → ang → angu → angular` while every request is slow:

- `switchMap`: prior four subscriptions are canceled; only `angular` should survive.
- `mergeMap`: all five run and may complete out of order.
- `concatMap`: all five run sequentially, preserving order but serving stale searches.
- `exhaustMap`: first `a` runs; later values during it are ignored (a later value only runs if emitted after completion).

Unsubscription requests cancellation; whether a non-Angular backend truly stops its external side effect depends on that source. Angular HTTP aborts in-flight transport.

## Error placement

```ts
clicks$.pipe(
  switchMap(() => api.refresh().pipe(
    catchError(error => of({kind: 'failed' as const, error})),
  )),
).subscribe();
```

Inner `catchError` lets future clicks continue. An outer `catchError` usually ends/replaces the whole click-driven stream after one error. `retry` resubscribes upstream; it can duplicate side effects. `finalize` runs on completion, error, or unsubscribe—good for resource cleanup, but global spinners need concurrency counting rather than a boolean.

## `shareReplay` nuance

```ts
config$ = defer(() => http.get<Config>('/api/config')).pipe(
  shareReplay({bufferSize: 1, refCount: true}),
);
```

This shares an active execution and replays one value. Decide: should the cache survive zero subscribers? reset on error? refresh when? With a completing HTTP source, a replayed success can remain cached. If freshness matters, implement invalidation/reload rather than assuming `refCount` means “never cached.”

## Subscription management

Preferred order:

1. `AsyncPipe` for template output.
2. `toSignal` once for synchronous state consumption.
3. finite streams (`HttpClient`, `take(1)`) where completion is intrinsic.
4. `takeUntilDestroyed()` for imperative long-lived subscriptions.
5. manual aggregate subscription only when ownership is explicit.

Bad:

```ts
ngOnInit() { this.service.data$.subscribe(v => this.data = v); }
```

Good for imperative effect:

```ts
private readonly destroyRef = inject(DestroyRef);
start() {
  this.service.data$.pipe(takeUntilDestroyed(this.destroyRef)).subscribe(v => this.chart.draw(v));
}
```

`takeUntilDestroyed()` without an explicit `DestroyRef` must be called in an injection context; a normal method is not one.

## Nested subscription anti-pattern

```ts
// Bad: nested lifetime and stale race
route.params.subscribe(p => api.get(p['id']).subscribe(user => this.user = user));

// Good
user$ = route.paramMap.pipe(
  map(p => p.get('id')),
  filter((id): id is string => id !== null),
  distinctUntilChanged(),
  switchMap(id => api.get(id)),
);
```

## Senior prediction

```ts
of(1, 2).pipe(
  tap(x => console.log('tap', x)),
  map(x => x * 10),
).subscribe(x => console.log('next', x));
```

Output is `tap 1`, `next 10`, `tap 2`, `next 20`; synchronous sources execute operator pipeline per emission, not stage-by-stage over the whole collection.

## Official sources

- [RxJS Observable guide](https://rxjs.dev/guide/observable)
- [RxJS Subject guide](https://rxjs.dev/guide/subject)
- [RxJS operator API](https://rxjs.dev/api/operators)
- [Angular RxJS interop](https://angular.dev/ecosystem/rxjs-interop)
- [`takeUntilDestroyed`](https://angular.dev/ecosystem/rxjs-interop/take-until-destroyed)

