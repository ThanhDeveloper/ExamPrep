# Angular Security

Angular's protections reduce classes of bugs; they do not replace server authorization, validation, secure session design, CSP or dependency patching.

## XSS model

Angular treats template-bound values as untrusted and escapes/sanitizes by security context. Angular templates themselves are trusted executable code. Never concatenate user input into a template and compile it. AOT is the production default and avoids shipping runtime template compilation.

```html
<p>{{ userText() }}</p>                 <!-- escaped text -->
<div [innerHTML]="trustedArticle()"></div> <!-- HTML-context sanitization -->
```

Sanitization differs by HTML, style, URL and resource URL context. Resource URLs load executable code and cannot be made safe by string cleaning in the general case.

## Dangerous bypass

Bad:

```ts
safe = this.sanitizer.bypassSecurityTrustHtml(userInput);
```

This labels attacker-controlled HTML as trusted and disables Angular's protection. A bypass is a security review assertion, not a sanitizer. Prefer safe data plus templates. If genuinely necessary, validate/construct the minimal value close to a trusted source, use the exact context and audit it.

Direct DOM methods, `ElementRef.nativeElement.innerHTML`, and third-party renderers can bypass Angular sanitization. Prefer templates; otherwise call `DomSanitizer.sanitize` with the correct `SecurityContext` and enforce platform defenses.

## CSP and Trusted Types

Content Security Policy is defense in depth. Angular documents a nonce-based minimum policy for new apps and supports automatic/manual nonce configuration (`autoCsp`, `ngCspNonce`, `CSP_NONCE`). Generate a fresh unpredictable nonce per response; do not hard-code one.

Trusted Types enforcement protects DOM injection sinks. The base `angular` policy is required; additional policies (`angular#bundler`, `angular#unsafe-bypass`, `angular#unsafe-jit`, `angular#unsafe-upgrade`) should exist only when those features require them. Needing `unsafe-bypass` is an audit signal.

## XSRF/CSRF

For eligible mutating relative/same-origin requests, `HttpClient` reads `XSRF-TOKEN` and sends `X-XSRF-TOKEN`. The backend must set a user-specific verifiable token and reject mismatches. Configure alternate names with `withXsrfConfiguration`. Do not make state-changing GET endpoints.

CSRF and CORS are different. CORS governs response access/cross-origin permissions; it is not a substitute for CSRF defense on cookie-authenticated mutation endpoints.

## Authentication and authorization

- Route guards improve UX/navigation; users control browser code and requests. They are not security boundaries.
- Hide/show directives are not authorization.
- Server endpoints must authenticate and authorize every operation and object access.
- Avoid placing long-lived high-value tokens where injected script can read them; no browser storage defeats XSS.
- Interceptors attach credentials mechanically; scope by trusted origin and endpoint policy.
- Frontend validation is UX; backend validation protects data/invariants.

## SSR security

Keep request-specific data out of module-scope singleton values. Do not serialize secrets/personalized resource or HTTP transfer-cache entries into cacheable shared HTML. Angular 22 validates proxy headers and ignores forwarded headers by default; trust them only behind a proxy that overwrites/validates them. Keep SSR response body limits bounded.

## URL and navigation safety

Do not treat an Angular-sanitized link as an authorization decision. Validate redirect targets against an allow-list to prevent open redirects. Encode data as data; do not concatenate it into script/style/template contexts.

## Security review checklist

- Current supported Angular release and dependencies?
- Any `bypassSecurityTrust*`, direct DOM HTML, dynamic template/JIT usage?
- CSP nonces and Trusted Types enforced in actual response headers?
- Server authorization independent from guards/UI?
- Cookie flags, XSRF server validation and no state-changing GET?
- Tokens sent only to expected origins?
- SSR providers/transfer cache isolated per request/user?
- Third-party scripts minimized and governed by CSP/SRI where appropriate?

## Official sources

- [Angular security guide](https://angular.dev/best-practices/security)
- [HTTP setup/XSRF configuration](https://angular.dev/guide/http/setup)
- [Route guard warning](https://angular.dev/guide/routing/route-guards)
- [SSR security boundaries](https://angular.dev/guide/ssr)

