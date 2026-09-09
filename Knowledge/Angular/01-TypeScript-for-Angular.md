# TypeScript for Angular

This is the Angular subset of TypeScript, not a general language tutorial. Angular 22.0 requires TypeScript `>=6.0.0 <6.1.0`; verify patch/minor compatibility before upgrading TypeScript independently in [Angular's compatibility table](https://angular.dev/reference/versions).

## Null safety and inference

Use `strict` and `strictTemplates`. Values from inputs, queries, forms, resources, DOM and routes may legitimately be absent. Narrow them rather than scattering `!`.

```ts
type LoadState<T> =
  | {kind: 'loading'}
  | {kind: 'loaded'; value: T}
  | {kind: 'failed'; error: Error};

function label(state: LoadState<User>): string {
  switch (state.kind) {
    case 'loading': return 'Loading';
    case 'loaded': return state.value.name;
    case 'failed': return state.error.message;
  }
}
```

**Input:** one discriminated-union state. **Output:** an exhaustively narrowed result without unsafe casts. **Why:** this maps directly to UI control flow and makes impossible state combinations unrepresentable.

## Interfaces, aliases and DTOs

```ts
interface ApiResponse<T> {
  readonly data: T;
  readonly success: boolean;
}

type UserSummary = Pick<User, 'id' | 'displayName'>;
type UsersResponse = ApiResponse<readonly UserSummary[]>;
```

`ApiResponse<User[]>` substitutes `User[]` for `T`, so `data` has type `User[]`. Prefer interfaces for extendable object contracts and aliases for unions, mapped/conditional types, primitives and compositions. Neither exists at runtime, so neither can be an Angular DI token; use a class or `InjectionToken<T>`.

## Angular-relevant type toolbox

| Feature | Angular use |
|---|---|
| Union/literal types | view states, route data, API result discriminators |
| Intersections | compose configuration/capability types; avoid accidental impossible intersections |
| Generics | API clients, reusable components, typed forms/stores |
| `readonly` | communicate ownership; complements immutable updates but does not deep-freeze |
| Optional properties | distinguish omission from `undefined`; important for PATCH DTOs |
| `keyof` | typed column definitions and generic property selectors |
| `typeof` | derive a type from a value/configuration |
| Indexed access | `User['id']`, response payload extraction |
| Mapped types | form/control maps, readonly/partial projections |
| Conditional types | API helpers whose result depends on an option/type |
| Utility types | `Pick`, `Omit`, `Partial`, `Required`, `Readonly`, `Record`, `Awaited` |
| Abstract classes | runtime DI token plus contract; unlike interfaces, they survive emission |
| Decorators | Angular compiler metadata (`@Component`, `@Directive`, `@Pipe`, service decorators) |

## Mapped form types

```ts
type ControlsOf<T> = {
  [K in keyof T]: FormControl<T[K]>;
};

interface Login { email: string; remember: boolean; }
type LoginControls = ControlsOf<Login>;
// {email: FormControl<string>; remember: FormControl<boolean>}
```

Do not build clever generic abstractions that hide disabled-control semantics, nested groups, arrays or nullable reset behavior. Prefer framework-inferred typed forms where possible.

## Narrowing DOM events

```ts
onInput(event: Event) {
  const input = event.target;
  if (!(input instanceof HTMLInputElement)) return;
  this.query.set(input.value);
}
```

`EventTarget` does not promise `.value`. A cast asserts; `instanceof` verifies and narrows. In templates Angular can infer `$event` under strict template checking.

## Access modifiers and templates

- `private`: implementation details not used by a template.
- `protected`: template-visible API that should not be a public component API; the Angular style guide favors this for template-only members.
- `public`: programmatic consumers and public component surface.
- `readonly`: references that Angular/consumers should not replace, such as signal objects and injected dependencies.

```ts
export class UserCard {
  protected readonly user = input.required<User>();
  private readonly audit = inject(AuditService);
}
```

## Enums versus literal unions

Literal unions emit no runtime object and narrow naturally:

```ts
type Role = 'admin' | 'editor' | 'viewer';
```

Use an enum only when its runtime object/iteration or compatibility is valuable. Avoid `const enum` across library boundaries without understanding compilation constraints.

## Common Angular TypeScript mistakes

- `any` disables both TS and downstream template safety; use `unknown` and narrow.
- Non-null assertions hide lifecycle/query/resource states.
- Interfaces cannot be injected because they are erased.
- `readonly User[]` prevents array mutation through that reference, not mutation of each `User`.
- A type assertion does not validate an HTTP response. `http.get<User>()` is a compile-time claim, not runtime parsing.
- Deep domain class instances are poor Signal Forms structural models; use plain objects at the form boundary.
- Subscription callback `this` errors usually come from passing unbound methods; arrow callbacks preserve lexical `this`.

## Senior interview note

Explain the three layers independently: TypeScript checks source, Angular checks template bindings, and runtime data remains untrusted. Good types reduce bugs but do not validate JSON or enforce authorization.

## Official sources

- [TypeScript handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript utility types](https://www.typescriptlang.org/docs/handbook/utility-types.html)
- [Angular template type checking](https://angular.dev/tools/cli/template-typecheck)
- [Angular style guide](https://angular.dev/style-guide)

