# Data Binding

Bindings connect template expressions and events to DOM/component APIs. Angular evaluates source expressions during synchronization and writes only changed binding values.

## Binding matrix

| Syntax | Direction | Target |
|---|---|---|
| `{{value}}` | state → text | text node/string context |
| `[disabled]="flag"` | state → property/input | DOM property or directive/component input |
| `[attr.aria-label]="label"` | state → attribute | HTML/ARIA attribute (`null` removes it) |
| `[class.active]="active"` | state → class | one CSS class |
| `[class]="classes"` | state → classes | string/array/object class set |
| `[style.width.px]="width"` | state → style | CSS property with optional unit |
| `(click)="save()"` | event → code | DOM event or Angular output |
| `[(value)]="selection"` | state ↔ event | `value` input/model plus `valueChange` output |

## Property versus attribute

HTML initializes DOM; thereafter properties usually represent live state. Bind a property for behavior (`disabled`, `value`, component input). Bind an attribute when no corresponding property exists or ARIA/SVG semantics require it.

```html
<button [disabled]="saving()" [attr.aria-label]="buttonLabel()">Save</button>
```

**Input:** two signals. **Output:** the DOM `disabled` property and `aria-label` attribute update. **Why:** disabled is live element state; ARIA is expressed as an attribute.

## Interpolation is string-context binding

```html
<img alt="Profile photo of {{ user().name }}" />
```

Angular escapes interpolated text. Interpolation is not a safe mechanism for constructing templates or scripts.

## Event binding

```ts
onKeydown(event: KeyboardEvent) {
  if (event.key === 'Enter') this.submit();
}
```

```html
<input (keydown)="onKeydown($event)" />
```

Returning `false` from an event handler in a template causes Angular to call `preventDefault`; explicit event handling is clearer for nontrivial behavior. Event handling in a component subtree is also an Angular render notification.

## Two-way binding

### Native control with Forms

```html
<input [(ngModel)]="name" name="name" />
```

This requires `FormsModule` and expands conceptually to `[ngModel]` plus `(ngModelChange)`.

### Component model

```ts
// child
value = model(0);
```

```html
<!-- parent; quantity can be a writable signal -->
<app-stepper [(value)]="quantity" />
```

Avoid two-way binding for command-like interactions (`delete`, `submit`, `approve`). Emit explicit intent.

## Class/style binding performance

Angular compares string values by value. For array/object class/style bindings, keep stable references if content has not changed. Prefer clear single-class/style bindings when only one property changes. CSS classes usually encode design intent better than many inline style expressions.

## Component binding resolution

If an element matches a directive/component input name, `[name]` binds to that input; otherwise Angular validates a DOM property. Attribute binding is always explicit with `attr.`. Custom events do not bubble unless separately dispatched as DOM events.

## Security contexts

Angular sanitizes values in HTML and URL contexts when bound through templates. Resource URLs cannot generally be sanitized because they load executable code. `bypassSecurityTrust*` disables a protection and must never wrap untrusted values. See Security.

## Interview questions

**Why does `[attr.disabled]="false"` still disable some HTML controls?** The string-valued attribute still exists; remove with `null`, or bind the boolean DOM property `[disabled]`.

**What does banana-in-a-box require?** An input/model named `x` and corresponding output `xChange`, or a framework directive such as `ngModel` implementing the pair.

**Does property binding always write an HTML attribute?** No. It normally writes the live DOM property or Angular input.

## Official sources

- [Binding text, properties and attributes](https://angular.dev/guide/templates/binding)
- [Event listeners](https://angular.dev/guide/templates/event-listeners)
- [Two-way binding](https://angular.dev/guide/templates/two-way-binding)

