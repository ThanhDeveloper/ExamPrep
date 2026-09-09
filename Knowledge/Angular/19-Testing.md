# Testing Modern Angular

## Current recommendation

New Angular CLI projects use **Vitest** with **jsdom** by default; `ng test` builds and runs Vitest in watch mode interactively. Karma remains supported for existing projects. Migrate deliberately because browser fidelity, fake timers, spies and Zone-based helpers differ.

Test observable behavior and public contracts. Use the smallest real Angular integration that catches the risk: pure unit test, TestBed component/service, HTTP/router harness, or browser E2E.

## Component test

```ts
describe('Counter', () => {
  it('increments from a user click', async () => {
    const fixture = TestBed.createComponent(Counter);
    await fixture.whenStable();

    fixture.nativeElement.querySelector('button').click();
    await fixture.whenStable();

    expect(fixture.nativeElement.textContent).toContain('1');
  });
});
```

In zoneless-style tests, prefer framework notifications/user interactions plus `await fixture.whenStable()` over forcing `detectChanges()` after arbitrary plain-field mutation. Existing suites can keep `detectChanges`; the important requirement is production-compatible notifications.

`ComponentFixture` gives the instance, native element, debug element, change detector and stability helpers. Set inputs with `fixture.componentRef.setInput(...)` to exercise Angular input semantics.

## Service/DI test

```ts
TestBed.configureTestingModule({
  providers: [OrderService, {provide: CLOCK, useValue: fixedClock}],
});
const service = TestBed.inject(OrderService);
expect(service.deadline()).toEqual(expected);
```

Pure services without Angular DI can be constructed/tested directly. Use TestBed when provider resolution/injection context is part of the behavior.

## HTTP test

```ts
TestBed.configureTestingModule({
  providers: [provideHttpClient(), provideHttpClientTesting(), UsersApi],
});

const api = TestBed.inject(UsersApi);
const http = TestBed.inject(HttpTestingController);
let result: User[] | undefined;
api.list('ada').subscribe(v => result = [...v]);

http.expectOne(r => r.url === '/api/users' && r.params.get('term') === 'ada')
  .flush([{id: '1', name: 'Ada'}]);
http.verify();
expect(result?.[0].name).toBe('Ada');
```

Ordering matters: configure `provideHttpClient(...)` before `provideHttpClientTesting()` so the testing backend overrides it while preserving requested features.

## Router test

```ts
TestBed.configureTestingModule({
  providers: [provideRouter([{path: 'users/:id', component: UserDetail}])],
});
const harness = await RouterTestingHarness.create();
const component = await harness.navigateByUrl('/users/42', UserDetail);
expect(component.id()).toBe('42');
```

`RouterTestingHarness` exercises real recognition/activation more reliably than mocking `ActivatedRoute` internals for integration behavior.

## Async tests

- Prefer native `async`/`await`, stable fixtures and Vitest fake timers for new tests.
- `fakeAsync`, `tick`, `flush`, and `waitForAsync` are common supported Angular/Zone-based utilities; Vitest migration needs `zone.js/plugins/vitest-patch` for compatibility.
- Do not mix fake timers and uncontrolled real time.
- Flush HTTP explicitly; await router/resource/render stability explicitly.
- Test cancellation/races with controllable sources, not arbitrary sleeps.

## Signals and effects

Read computed signals after writes; they are lazy. Component effects/rendering are scheduled, so await stability when the assertion depends on the view/effect. Test the public state/result rather than effect run counts unless scheduling itself is the contract.

## Defer blocks

TestBed can set `DeferBlockBehavior.Manual`; retrieve block fixtures and render placeholder/loading/complete/error states deterministically.

## Test doubles

- Fake pure external boundary, not your own business rules.
- Prefer typed fakes/providers over enormous partial object mocks.
- At component level, HTTP testing often gives a more realistic boundary than mocking a data service's implementation details.
- Component harnesses provide a stable user-facing interaction API for reusable components.

## Common mistakes

- `NO_ERRORS_SCHEMA` hides real missing imports/binding errors.
- Testing class methods only while template bindings are broken.
- Calling remote services from unit tests.
- Forgetting `http.verify()`.
- Overusing `detectChanges()` so tests pass without production notification.
- Brittle assertions on private fields, CSS implementation or lifecycle counts.
- Keeping Karma/Jasmine assumptions while claiming a new default setup.

## Official sources

- [Testing overview and Vitest default](https://angular.dev/guide/testing)
- [Component scenarios](https://angular.dev/guide/testing/components-scenarios)
- [HTTP testing](https://angular.dev/guide/http/testing)
- [Router testing](https://angular.dev/guide/routing/testing)
- [Karma-to-Vitest migration](https://angular.dev/guide/testing/migrating-to-vitest)

