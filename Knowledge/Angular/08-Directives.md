# Directives

A directive attaches Angular behavior to a host. A component is a directive with a template; an attribute directive changes behavior/appearance of an existing host; a structural directive manages embedded views.

## Attribute directive

```ts
@Directive({
  selector: '[appHighlight]',
  host: {
    '[style.backgroundColor]': 'color()',
    '(mouseenter)': 'hovered.emit(true)',
    '(mouseleave)': 'hovered.emit(false)',
  },
})
export class Highlight {
  readonly color = input('gold', {alias: 'appHighlight'});
  readonly hovered = output<boolean>();
}
```

```html
<p [appHighlight]="warningColor()" (hovered)="setPreview($event)">Attention</p>
```

**Input:** color and pointer events. **Output:** host style plus typed hover intent. **Why:** host metadata keeps host behavior with the directive. Current style guidance prefers `host` over decorator host bindings/listeners.

## Structural directive mental expansion

```html
<section *appPermission="'admin'">Secret</section>
```

Conceptually becomes:

```html
<ng-template [appPermission]="'admin'"><section>Secret</section></ng-template>
```

A custom structural directive injects `TemplateRef` and `ViewContainerRef`, decides when to create an embedded view and owns its cleanup. Only one `*` shorthand can occupy an element; use `<ng-container>` to nest behaviors.

```ts
@Directive({selector: '[appPermission]'})
export class PermissionDirective {
  private readonly template = inject(TemplateRef<unknown>);
  private readonly container = inject(ViewContainerRef);
  private readonly auth = inject(AuthService);
  readonly role = input.required<string>({alias: 'appPermission'});

  constructor() {
    effect(() => {
      this.container.clear();
      if (this.auth.hasRole(this.role())) {
        this.container.createEmbeddedView(this.template);
      }
    });
  }
}
```

This illustrates mechanics, but production code should avoid repeatedly rebuilding a view when it can reuse/detach it. For ordinary conditions/loops/switches, Angular explicitly recommends `@if`, `@for`, `@switch`; custom structural directives are for reusable rendering semantics those blocks do not cover.

## Directive composition

`hostDirectives` can apply directives to a component/directive host and selectively expose inputs/outputs. It is compile-time composition; consumers do not add the composed directive themselves. Watch for host binding collisions and hidden API complexity.

## Common mistakes

- Using direct DOM APIs without considering SSR/security; prefer host/template binding or renderer abstractions.
- Applying business policy globally through a directive with hidden services.
- Reimplementing `@if`/`@for`.
- Forgetting view cleanup or duplicating embedded views.
- Assuming directives introduce DOM elements.

## Official sources

- [Directive overview](https://angular.dev/guide/directives)
- [Attribute directives](https://angular.dev/guide/directives/attribute-directives)
- [Structural directives](https://angular.dev/guide/directives/structural-directives)
- [Directive composition](https://angular.dev/guide/directives/directive-composition-api)

