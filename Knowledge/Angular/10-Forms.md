# Forms

Angular 22 has three first-party form systems. Choose by state model and project context, not ideology.

| System | Source of truth | Best fit | Status |
|---|---|---|---|
| Signal Forms | writable signal + field tree | new signal-oriented apps, schema validation, typed field state | ✅ stable since v22 |
| Reactive Forms | explicit `AbstractControl` tree | mature complex/dynamic forms, Observable workflows, existing code | ✅ stable/current |
| Template-driven | template directives and mutable model | simple forms/prototypes | 🟡 supported/context dependent |

Official comparison guidance still calls Reactive Forms a solid choice when mature production stability, fine control, or an existing reactive codebase matters. Stable does not mean every ecosystem integration has equal maturity.

## Signal Forms

```ts
import {form, FormField, required, email} from '@angular/forms/signals';

@Component({
  imports: [FormField],
  template: `
    <input type="email" [formField]="loginForm.email" />
    <input type="password" [formField]="loginForm.password" />
    @if (loginForm.email().touched() && loginForm.email().invalid()) {
      <p>{{ loginForm.email().errors()[0].message }}</p>
    }
    <button [disabled]="!loginForm().valid()">Sign in</button>
  `,
})
export class Login {
  readonly model = signal({email: '', password: ''});
  readonly loginForm = form(this.model, path => {
    required(path.email, {message: 'Email is required'});
    email(path.email, {message: 'Use a valid email'});
    required(path.password, {message: 'Password is required'});
  });
}
```

**Input:** user input through `[formField]`. **Output:** the writable model signal, field-state signals and validation update automatically. **Why:** `form()` builds a navigable/callable field tree that mirrors a plain-object/array model; the original signal remains the source of truth.

Field state includes `value`, `valid`, `invalid`, `errors`, `pending`, `touched`, `dirty`, `disabled`, `hidden`, and `readonly`. During async validation, both `valid()` and `invalid()` can be false; check `pending()` explicitly.

Structural model objects/arrays should be plain. Class instances, `Map` and `Set` are not supported structural layers. Translate domain objects at the boundary.

Signal Forms async validators use `validateHttp`/`validateAsync`; sync validation runs first and stale async validation is canceled on value change. Explicitly choose submission policy when validators are pending.

## Reactive Forms

```ts
@Component({
  imports: [ReactiveFormsModule],
  template: `
    <form [formGroup]="profile" (ngSubmit)="save()">
      <input formControlName="name" />
      <div formArrayName="aliases">
        @for (control of aliases.controls; track control) {
          <input [formControl]="control" />
        }
      </div>
      <button [disabled]="profile.invalid || profile.pending">Save</button>
    </form>`,
})
export class ProfileEditor {
  private readonly fb = inject(NonNullableFormBuilder);
  readonly profile = this.fb.group({
    name: ['', [Validators.required, Validators.minLength(2)]],
    aliases: this.fb.array<string>([]),
  });
  get aliases() { return this.profile.controls.aliases; }
  save() { if (this.profile.valid) console.log(this.profile.getRawValue()); }
}
```

`FormControl` owns one value/status, `FormGroup` fixed keyed controls, `FormArray` ordered homogeneous controls, and `FormRecord` open-ended homogeneous keyed controls. Typed Reactive Forms have been default since v14.

### `setValue` versus `patchValue`

| API | Contract |
|---|---|
| `setValue` | provide the complete expected structure; catches missing/extra structure |
| `patchValue` | update only supplied members; useful for partial DTOs but can hide misspelled/ignored fields |

```ts
profile.setValue({name: 'Ada', aliases: ['Enchantress']});
profile.patchValue({name: 'Grace'});
```

Disabled controls are omitted from a group's `.value`; `getRawValue()` includes them. A control is nullable by default because `.reset()` can set `null`; `NonNullableFormBuilder` changes both type and reset behavior.

### State model

| State | Meaning |
|---|---|
| valid/invalid | validator result |
| pending | async validation in progress |
| disabled | excluded from validation/ancestor aggregate value |
| touched/untouched | blur/visit interaction |
| dirty/pristine | UI changed value / has not |

Do not confuse dirty with “different from initial value” or touched with focus.

### Async validator

```ts
const username = new FormControl('', {
  nonNullable: true,
  validators: [Validators.required],
  asyncValidators: [uniqueUsernameValidator],
  updateOn: 'blur',
});
```

Angular runs async validators only if sync validators pass. Returned Observables must complete. `updateOn: 'blur'|'submit'` can reduce request traffic.

`valueChanges` and `statusChanges` are multicasting Observables. A child control emits before its parent aggregate has necessarily updated; subscribe at the ownership level you need. Use RxJS operators for derived async work, and manage imperative subscriptions.

## Template-driven Forms

```html
<form #form="ngForm" (ngSubmit)="save()">
  <input name="email" [(ngModel)]="model.email" required email #email="ngModel" />
  @if (email.invalid && (email.touched || form.submitted)) {
    <p>Enter a valid email.</p>
  }
  <button [disabled]="form.invalid">Save</button>
</form>
```

Import `FormsModule`. `NgForm` aggregates, `NgModel` bridges a control and model, and `NgModelGroup` nests. This approach is concise but validation/control structure is distributed through template directives and scales less predictably.

## Custom controls

Reactive/template forms communicate with native/custom controls through `ControlValueAccessor`. Signal Forms custom controls can bind field/model contracts directly. A reusable control must propagate value, disabled state, touched state and validation correctly; implementing only value writes is incomplete.

## Mistakes

- Mixing form systems on the same controls without an explicit integration design.
- Subscribing to `valueChanges` merely to copy one control to another; use form rules/derived state and prevent loops.
- Calling `setValue` with a partial server DTO.
- Using `patchValue` everywhere and silently ignoring contract drift.
- Enabling a disabled control only to read it; use `getRawValue` if appropriate.
- Running HTTP async validation on every keystroke without cancellation/debounce/update policy.
- Showing all errors before interaction/submission.
- Trusting client validation as server authorization/business enforcement.

## Official sources

- [Forms overview](https://angular.dev/guide/forms)
- [Signal Forms](https://angular.dev/guide/forms/signals/overview)
- [Signal Forms comparison](https://angular.dev/guide/forms/signals/comparison)
- [Reactive Forms](https://angular.dev/guide/forms/reactive-forms)
- [Typed Forms](https://angular.dev/guide/forms/typed-forms)
- [Template-driven Forms](https://angular.dev/guide/forms/template-driven-forms)
- [Validation](https://angular.dev/guide/forms/form-validation)

