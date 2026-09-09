# Components

## Mental model

A component is a directive with a template. `@Component` supplies compile-time metadata; Angular creates an instance, a host element, an element injector and a view. The class holds behavior/state, the template describes DOM, and styles are scoped according to encapsulation.

## Metadata reference

| Field | Meaning | Senior caution |
|---|---|---|
| `selector` | CSS selector matching a host | Prefer a project prefix for reusable app components |
| `template` / `templateUrl` | inline/external template | Templates are trusted executable code; never build them from user input |
| `styles` / `styleUrl` / `styleUrls` | inline/external styles | Style choice does not change component behavior |
| `imports` | template dependencies | Explicit for standalone declarations; unused imports can be diagnosed |
| `providers` | providers on the host element injector | Creates an instance per component occurrence/subtree lifetime |
| `viewProviders` | providers visible to the view, not projected content | Use only when that visibility distinction is intentional |
| `host` | host properties/attributes/listeners | Preferred over `@HostBinding`/`@HostListener` in current style guidance |
| `encapsulation` | `Emulated`, `ShadowDom`, or `None` | `None` is global CSS; Emulated is selector rewriting, not Shadow DOM |
| `changeDetection` | `OnPush` or `Eager`/`Default` | OnPush is default since v22 |
| `standalone` | set `false` for an NgModule declaration | Standalone is default/recommended for new code |

```ts
@Component({
  selector: 'app-price-card',
  imports: [CurrencyPipe],
  template: `<button type="button" (click)="chosen.emit(id())">
    {{ label() }} — {{ price() | currency }}
  </button>`,
  styleUrl: './price-card.css',
  host: {'[class.selected]': 'selected()', 'role': 'group'},
})
export class PriceCard {
  readonly id = input.required<string>();
  readonly price = input(0, {transform: numberAttribute});
  readonly label = input('Plan');
  readonly selected = input(false, {transform: booleanAttribute});
  readonly chosen = output<string>();
}
```

**Input:** ID, price, label and selection state. **Output:** rendered price and a typed chosen-ID event. **Why:** inputs are read-only signals controlled by the parent; the output publishes intent without coupling the card to navigation/business policy.

## Input APIs

### `input()` and `input.required()` — ✅ recommended

`input(defaultValue)` creates an `InputSignal<T>`. Angular writes the binding; the child reads by calling it. `input.required<T>()` removes `undefined` from the read type and makes omission a compile-time template error. Options include aliases and transforms. A transform must be pure and should only coerce representation, not encode business logic.

```ts
quantity = input(1, {transform: numberAttribute});
product = input.required<Product>();
```

### `@Input()` — 🟡 supported

Decorator inputs remain useful in legacy code and certain patterns. They are ordinary fields/setters, so derived state often needs `ngOnChanges` or a setter; signal inputs compose directly with `computed`.

| `@Input` | `input()` |
|---|---|
| field/setter updated by Angular | read-only signal updated by Angular |
| required via metadata/type tooling | `input.required<T>()` |
| derive via setter/hook/getter | derive via `computed()` |
| supported | current signal-oriented style |

## Output APIs

### `output()` — ✅ recommended

Returns `OutputEmitterRef<T>`. Call `.emit(value)`. Angular cleans up programmatic output subscriptions when it destroys involved components. Outputs do not bubble through the DOM.

```ts
saved = output<Order>();
save(order: Order) { this.saved.emit(order); }
```

### `@Output() EventEmitter` — 🟡 supported

`EventEmitter` is stable, not deprecated, but `output()` is simpler and compiler-integrated. Do not use `EventEmitter` as a general service event bus; RxJS or signals express those contracts better.

## `model()` for component two-way binding

`model(initial)` creates a writable `ModelSignal<T>` plus an implicit `nameChange` output.

```ts
@Component({selector: 'app-stepper', template: `
  <button (click)="value.update(v => v - 1)">−</button>
  {{ value() }}
  <button (click)="value.update(v => v + 1)">+</button>
`})
export class Stepper { readonly value = model(0); }
```

Parent:

```html
<app-stepper [(value)]="quantity" />
```

**Input:** parent's property or writable signal. **Output:** child writes propagate through `valueChange`. **Why:** model inputs are intended for values the child itself edits, especially custom controls; use input+output when commands should be explicit.

## Composition

- Inputs are data/configuration; outputs are user/domain intent.
- Content projection (`ng-content`) lets a parent supply markup while the receiving component controls placement.
- Prefer composition over inheritance; Angular metadata and DI make component inheritance surprising.
- Smart/page components orchestrate; UI components render and emit. Treat this as a design tool, not dogma.
- Avoid hidden coupling through root services in every leaf component.

## Host and encapsulation

```ts
@Directive({
  selector: '[appPressable]',
  host: {
    'role': 'button',
    '[attr.aria-disabled]': 'disabled()',
    '(keydown.enter)': 'activate.emit()',
  },
})
export class Pressable {
  disabled = input(false);
  activate = output<void>();
}
```

Host bindings belong to the component/directive's own host. If a consumer and directive both bind the same host property, collision precedence depends on static/dynamic forms; avoid designing conflicting APIs.

## Common mistakes

- Mutating an input object hides ownership and can leave consumers stale; emit intent or replace state immutably.
- Mirroring every input into another signal with `effect()` creates synchronization bugs; use `computed` or `linkedSignal`.
- Adding a provider to a component accidentally creates one service instance per component instance.
- `ViewEncapsulation.Emulated` does not stop global CSS entering the component.
- Output names that look like native DOM events confuse consumers.
- A required input can still be unavailable if read too early in construction; let the template/lifecycle establish it.

## Senior interview prompts

**Are standalone components actually standalone?** They still depend on imports and DI; “standalone” means no declaring NgModule is required.

**When use `model()`?** When the component semantically edits a value and two-way binding is desirable. Do not use it to hide unrelated state transitions.

**Are NgModules deprecated?** No. Official guidance recommends standalone for new code; NgModules remain supported and relevant to existing code/libraries.

## Official sources

- [`@Component` API](https://angular.dev/api/core/Component)
- [Accepting inputs](https://angular.dev/guide/components/inputs)
- [Custom outputs](https://angular.dev/guide/components/outputs)
- [Host elements](https://angular.dev/guide/components/host-elements)
- [Styling/encapsulation](https://angular.dev/guide/components/styling)
- [Advanced configuration](https://angular.dev/guide/components/advanced-configuration)

