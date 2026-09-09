# Angular Router

The Router maps a URL tree to a route configuration, redirects/matches, runs guards and resolvers, creates route injectors, loads code, and activates component views in outlets. URL state is durable, shareable browser state.

## Modern standalone setup

```ts
export const routes: Routes = [
  {path: '', pathMatch: 'full', redirectTo: 'users'},
  {
    path: 'users',
    providers: [UserFeatureStore],
    children: [
      {path: '', loadComponent: () => import('./users/list').then(m => m.UserList)},
      {
        path: ':id',
        canActivate: [authGuard],
        resolve: {user: userResolver},
        loadComponent: () => import('./users/detail').then(m => m.UserDetail),
      },
    ],
  },
  {path: '**', loadComponent: () => import('./not-found').then(m => m.NotFound)},
];

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes, withPreloading(PreloadAllModules))],
};
```

**Input:** navigation to `/users/42?tab=history#latest`. **Output:** route tree matches, feature code/providers resolve, guard/resolver complete, component activates in `RouterOutlet`; params/query/fragment become available. **Why:** route order is first-match wins, so specific routes precede wildcard.

## Navigation and links

```html
<a [routerLink]="['/users', user.id]" [queryParams]="{tab: 'history'}"
   fragment="latest" routerLinkActive="active">History</a>
<router-outlet />
```

Prefer `RouterLink` over manual click + `navigate` for actual links: semantics, accessibility, open-in-new-tab and URL generation work correctly. Programmatic navigation uses `navigate` with commands/extras or `navigateByUrl` with a URL/UrlTree.

## Reading route state

`ActivatedRoute.snapshot` is a point-in-time value. Reused components can receive new params without recreation, so subscribe/use router signals or bind route inputs when values can change. Path params identify hierarchical resources; query params describe optional filters/sort/paging; fragments identify document positions; navigation `state` is history state and not a durable replacement for URL/server storage.

## Guards

Functional guards are the generated/current style and may synchronously `inject` dependencies.

```ts
export const authGuard: CanActivateFn = (_route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);
  return auth.loggedIn() ? true : router.createUrlTree(['/login'], {
    queryParams: {returnUrl: state.url},
  });
};
```

Return `boolean`, `UrlTree`/`RedirectCommand`, Promise or Observable. The router uses the first async emission then unsubscribes. Return a redirect value; do not return false and imperatively navigate.

| Guard | Question |
|---|---|
| `CanActivate` | may this route activate? |
| `CanActivateChild` | may descendants activate? |
| `CanDeactivate` | may the current component be left? |
| `CanMatch` | should this route configuration match? false tries other candidates |

`CanLoad` is deprecated; use `CanMatch`. DI token/class entries in guard arrays are deprecated configuration in favor of plain functions, although guard interfaces/classes themselves may remain supported and can be wrapped. Guards run in the browser and are not security boundaries; servers must authorize every protected operation.

## Resolvers

```ts
export const userResolver: ResolveFn<User> = route =>
  inject(UserApi).get(route.paramMap.get('id')!);
```

Resolvers load required data before activation; they simplify a ready-on-entry screen but block navigation and need error/redirect handling. Prefer component/resource loading when skeleton-first navigation is better. Parent resolver data is available to child resolvers that execute later.

## Lazy loading

- `loadComponent`: standalone component chunk.
- `loadChildren`: lazy route array or legacy NgModule.
- An eager static import elsewhere can defeat splitting.
- Route providers create an `EnvironmentInjector` for the route/children.
- Preloading fetches lazy code after initial navigation; it does not activate routes or usually run their resolvers.

## Router events

`NavigationStart`, recognition, guard, resolve, activation, `NavigationEnd`, `NavigationCancel`, `NavigationError`, `NavigationSkipped` expose the lifecycle. Filter by event type and manage the subscription. Use events for global progress/telemetry only when route-specific designs are insufficient.

```ts
router.events.pipe(
  filter((e): e is NavigationEnd => e instanceof NavigationEnd),
  takeUntilDestroyed(),
).subscribe(e => analytics.page(e.urlAfterRedirects));
```

## Common failures

- Wildcard or broad parameter route placed before a specific route.
- `redirectTo: ''` without `pathMatch: 'full'`, causing overmatching.
- Reading a snapshot in a reused component and expecting updates.
- Guard performs authorization only on the client.
- Resolver never completes, so navigation hangs.
- Returning `false` then calling `navigate`, causing competing navigations.
- Barrel/static imports eagerly include a supposedly lazy component.
- Losing filter state in memory rather than URL query params.

## Official sources

- [Router overview](https://angular.dev/guide/routing)
- [Define routes](https://angular.dev/guide/routing/define-routes)
- [Read route state](https://angular.dev/guide/routing/read-route-state)
- [Navigate](https://angular.dev/guide/routing/navigate-to-routes)
- [Guards](https://angular.dev/guide/routing/route-guards)
- [Resolvers](https://angular.dev/guide/routing/data-resolvers)
- [Router events](https://angular.dev/guide/routing/lifecycle-and-events)

