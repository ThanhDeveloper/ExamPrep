# Pipes

Pipes are declarative template value transformations. Built-ins include `DatePipe`, `CurrencyPipe`, `DecimalPipe`, `PercentPipe`, `UpperCasePipe`, `LowerCasePipe`, `TitleCasePipe`, `JsonPipe`, `KeyValuePipe`, `SlicePipe`, i18n pipes and `AsyncPipe`.

## Custom pure pipe

```ts
@Pipe({name: 'initials'})
export class InitialsPipe implements PipeTransform {
  transform(name: string): string {
    return name.trim().split(/\s+/).map(part => part[0]?.toUpperCase()).join('');
  }
}
```

```html
<span>{{ 'Ada Lovelace' | initials }}</span>
```

**Input:** a name. **Output:** `AL`. **Why:** Angular invokes a pure pipe when primitive arguments change or object references change, then reuses the result while inputs remain identical.

## Pure versus impure

| Pure (default) | Impure (`pure: false`) |
|---|---|
| runs for changed input references/primitive values | can run every check of its view |
| safe for deterministic transformations | needed only when observing internal mutation/external state |
| enables reuse/memoization-like behavior | can dominate render cost |

An impure pipe is dangerous because even a cheap operation multiplied by many bindings and checks becomes expensive. Fix ownership/immutability or move temporal state into a signal/Observable before choosing impurity.

## `AsyncPipe`

```html
@if (user$ | async; as user) {
  <h2>{{ user.name }}</h2>
}
```

`AsyncPipe` subscribes to an Observable/Promise, returns the latest value, marks its view for checking when a new value arrives, switches subscription when the bound reference changes, and unsubscribes on view destruction. It is preferable to a manual subscription used only to display state.

### Manual subscription is valid when

- performing an imperative side effect;
- coordinating APIs outside templates/signals;
- a command needs explicit success/error handling.

Use `takeUntilDestroyed` for long-lived subscriptions. Do not subscribe in a getter or template-invoked method.

## Pipe arguments and chaining

```html
{{ total() | currency:'USD':'symbol':'1.2-2' }}
{{ createdAt() | date:'medium' | uppercase }}
```

Pipes execute left to right. Avoid hiding business logic or large array filtering/sorting in a pipe; a `computed` gives explicit ownership and reuse.

## Senior questions

**Why did a pure pipe not rerun after `array.push`?** The array reference did not change. Replace the array or deliberately use another reactive design.

**Does AsyncPipe prevent every leak?** It owns only its subscription. A hot source/service may still retain resources and shared-stream caching may still be misconfigured.

## Official sources

- [Pipes guide](https://angular.dev/guide/templates/pipes)
- [`AsyncPipe` API](https://angular.dev/api/common/AsyncPipe)

