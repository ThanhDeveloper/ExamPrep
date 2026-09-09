# Templates

Angular templates are HTML plus compiler-recognized binding, control-flow, variable, projection and defer syntax. They are compiled, type-checked executable application code—not strings to build dynamically from user input.

## Expression context and restrictions

Expressions can read component members, template locals and supported globals. JavaScript globals are deliberately limited; assignments are allowed in event statements but not ordinary interpolation/property expressions. Declarations, `new`, most bitwise operators and arbitrary global access are not template expression features. Angular adds optional chaining whose `null`/`undefined` result is `null`, pipes, non-null assertion, `$any`, and assignment operators in event statements.

Keep expressions cheap and side-effect free. Angular can evaluate them repeatedly during synchronization.

```html
@let subtotal = price() * quantity();
<p>{{ subtotal | currency }}</p>
```

`@let` is reactive but cannot be reassigned. It is scoped to the current view and descendants and is not hoisted.

## Built-in control flow — ✅ recommended

### `@if`

```html
@if (user(); as current) {
  <h2>Hello {{ current.name }}</h2>
} @else {
  <a routerLink="/login">Sign in</a>
}
```

**Input:** `user()` returns a user or null. **Output:** one branch's view exists. **Why:** the alias avoids repeated long expressions and narrows the truthy value.

### `@for`, `track`, and `@empty`

```html
<ul>
  @for (user of users(); track user.id;
        let i = $index, first = $first, odd = $odd) {
    <li [class.first]="first" [class.odd]="odd">{{ i + 1 }}. {{ user.name }}</li>
  } @empty {
    <li>No users</li>
  }
</ul>
```

Context locals are `$count`, `$index`, `$first`, `$last`, `$even`, and `$odd`. `track` associates items with views/DOM nodes. Use a stable unique key such as `id`; `$index` is acceptable for truly static collections; object identity is a last resort. Duplicate/unstable keys cause incorrect reuse or excess DOM work. `@for` has no `break` or `continue`.

### `@switch`

```html
@switch (permission()) {
  @case ('admin') { <admin-dashboard /> }
  @case ('editor') { <editor-dashboard /> }
  @default { <viewer-dashboard /> }
}
```

Comparison uses `===`; there is no fallthrough. Current template type checking can diagnose non-exhaustive union switches.

### Legacy control flow — ❌ deprecated

`NgIf`, `NgFor`/`NgForOf`, and `NgSwitch` directives are deprecated since v20 in favor of compiler-native blocks. They remain present in v22, so classify actual projects as deprecated-but-available, not removed. Migrate with:

```text
ng generate @angular/core:control-flow
```

## Template variables

```html
<input #emailInput type="email" />
<button (click)="submit(emailInput.value)">Submit</button>
```

`#emailInput` refers to the element by default, or to a directive/component named by `exportAs`. A reference is scoped to its template view. Do not treat it as persistent application state.

`$event` is the event/output payload:

```html
<input (input)="query.set($any($event.target).value)" />
<user-row (selected)="openUser($event)" />
```

Prefer a typed component handler over `$any` when logic grows.

## Views, templates and grouping

- `<ng-template>` stores an unrendered `TemplateRef`; a structural directive or `NgTemplateOutlet` can instantiate embedded views.
- `<ng-container>` groups template behavior without adding a normal DOM element.
- Control-flow blocks create/destroy embedded views.
- A component boundary creates a distinct view and DI boundary.

## Content projection

```html
<!-- panel template -->
<header><ng-content select="[panel-title]" /></header>
<section><ng-content /></section>
```

Projection slots are determined at build time. Do not conditionally include `<ng-content>`; Angular creates projected content even if the placeholder is hidden. Use template fragments/programmatic rendering when content must be lazy/conditional.

## Deferred templates

`@defer` is covered deeply in Performance. Only standalone dependencies used exclusively inside eligible defer blocks are split; placeholder/loading/error dependencies are eager.

## Senior traps

- A method call in a template is not memoized. Use `computed` for expensive derivation.
- `track $index` breaks identity when rows reorder/insert.
- Hiding an element is different from destroying its view.
- `@if` aliases are view-scoped, not component members.
- A template reference is not a signal and should not be used before its view exists.
- Angular sanitizes bound values, but templates themselves are trusted code.

## Official sources

- [Template overview](https://angular.dev/guide/templates)
- [Expression syntax](https://angular.dev/guide/templates/expression-syntax)
- [Variables](https://angular.dev/guide/templates/variables)
- [Control flow](https://angular.dev/guide/templates/control-flow)
- [Content projection](https://angular.dev/guide/components/content-projection)
- [Control-flow migration](https://angular.dev/reference/migrations/control-flow)

