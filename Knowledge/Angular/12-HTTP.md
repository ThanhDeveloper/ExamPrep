# HTTP and Server Data

`HttpClient` is injectable by default in Angular v21+. Angular 22 uses Fetch by default. Call `provideHttpClient(...)` when configuring features such as interceptors/XSRF/JSONP or overriding backend behavior. `HttpClientModule` is deprecated.

## Typed requests

```ts
@Service()
export class UsersApi {
  private readonly http = inject(HttpClient);

  list(term: string) {
    return this.http.get<readonly UserDto[]>('/api/users', {
      params: {term},
    });
  }
  create(command: CreateUser) { return this.http.post<UserDto>('/api/users', command); }
  replace(id: string, dto: UserDto) { return this.http.put(`/api/users/${id}`, dto); }
  patch(id: string, patch: Partial<UserDto>) { return this.http.patch(`/api/users/${id}`, patch); }
  delete(id: string) { return this.http.delete<void>(`/api/users/${id}`); }
}
```

The generic is a TypeScript assertion about the body—it does not validate JSON. Validate/map untrusted data at the boundary when correctness/security requires it.

Requests are immutable; `HttpHeaders` and `HttpParams` operations return new instances. Most `HttpClient` Observables are cold: each subscription sends a request. They usually emit once and complete, although interceptors can change this. Unsubscribing aborts the in-flight request; `switchMap` uses this to cancel stale reads.

## Responses and errors

```ts
load(id: string) {
  return this.http.get<UserDto>(`/api/users/${id}`, {observe: 'response'}).pipe(
    map(response => ({user: mapUser(response.body), etag: response.headers.get('ETag')})),
    retry({count: 2, delay: (_error, attempt) => timer(attempt * 500)}),
    catchError((error: HttpErrorResponse) =>
      throwError(() => new UserLoadError(id, error.status, {cause: error}))),
  );
}
```

`HttpErrorResponse.status === 0` commonly represents network/timeout/client failure; HTTP failures carry server status. Retry only safe/idempotent operations unless the API provides idempotency semantics. Preserve causal context and let the ownership layer decide user recovery.

## Functional interceptors — ✅ recommended

```ts
export function authInterceptor(req: HttpRequest<unknown>, next: HttpHandlerFn) {
  const token = inject(AuthService).token();
  const apiRequest = token && req.url.startsWith('/api/')
    ? req.clone({setHeaders: {Authorization: `Bearer ${token}`}})
    : req;
  return next(apiRequest);
}

export const appConfig: ApplicationConfig = {
  providers: [provideHttpClient(withInterceptors([authInterceptor, errorMetricInterceptor]))],
};
```

Requests/responses are immutable, so clone to change them. Interceptors form a chain in registration order; response flow unwinds in reverse through returned Observables. Functional interceptors are recommended because ordering/behavior is more predictable than DI multi-provider interceptors.

Do not attach secrets to untrusted origins. An access token in browser memory/storage is still available to code executing under XSS. Interceptors centralize mechanics, not authorization.

## Cancellation and stale data

```ts
results$ = this.search.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => this.http.get<Result[]>('/api/search', {params: {term}})),
);
```

New terms unsubscribe/abort old requests, preventing stale responses from winning. `mergeMap` would allow concurrency; `concatMap` queue; `exhaustMap` ignore new terms while one is running.

## Stable `httpResource`

```ts
id = signal<string | undefined>(undefined);
user = httpResource<User>(() => this.id() ? `/api/users/${this.id()}` : undefined);
```

This is ideal for signal-driven reads and exposes loading/error/metadata signals. It uses `HttpClient`, so interceptors and HTTP testing apply. Do not use a resource for writes: automatic cancellation can abort a mutation.

## XSRF

For mutating same-origin/relative requests, `HttpClient` reads the default `XSRF-TOKEN` cookie and sends `X-XSRF-TOKEN`. The server must issue and validate it; Angular implements only the client half. Configure names using `withXsrfConfiguration`. This is relevant to cookie-based authentication, not a universal bearer-token solution.

## Fetch versus XHR

Fetch is current default and works well with SSR. It does not provide upload progress events. `withXhr()` is the opt-in compatibility backend; server XHR is deprecated and intended for removal in Angular 23 because of redirect/security risks.

## Repeated request checklist

If an API “fires twice,” inspect:

1. multiple subscriptions to a cold Observable;
2. both `AsyncPipe` and manual subscription;
3. template method/getter constructing a new Observable;
4. retry/interceptor logic;
5. SSR request plus client request due to transfer-cache eligibility/configuration;
6. duplicate effects/lifecycle paths;
7. development tooling/network interpretation.

Use `shareReplay({bufferSize: 1, refCount: true})` only with an explicit cache lifetime/invalidation policy; it is not a magic deduplicator.

## Official sources

- [HTTP overview](https://angular.dev/guide/http)
- [Setup and Fetch default](https://angular.dev/guide/http/setup)
- [Making requests](https://angular.dev/guide/http/making-requests)
- [Interceptors](https://angular.dev/guide/http/interceptors)
- [HTTP testing](https://angular.dev/guide/http/testing)
- [`httpResource`](https://angular.dev/guide/http/http-resource)

