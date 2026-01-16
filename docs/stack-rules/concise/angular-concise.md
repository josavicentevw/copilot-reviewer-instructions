# Angular Review Cheat Sheet (Concise)

Reference: [`docs/stack-rules/angular-rules.md`](../angular-rules.md) · Examples: [`Angular Examples`](../examples-only/angular-examples.md)

## Core Checks

- [ ] **Type Safety**: `strict` mode on, no `any`, typed inputs/outputs, guards for runtime data.
- [ ] **Component Architecture**: OnPush for pure views, smart vs presentational split, unsubscribe (`takeUntil`/async pipe), `trackBy` on `*ngFor`.
- [ ] **RxJS Usage**: Prefer async pipe, tear down subscriptions, compose with `switchMap`, `catchError`, `shareReplay`.
- [ ] **Dependency Injection**: Constructor injection, proper provider scopes, injection tokens for configs.
- [ ] **Forms**: Reactive forms for complex flows, typed controls (Angular 14+), custom validators, template error messaging.
- [ ] **State Management**: NgRx actions/reducers/effects follow conventions, selectors for derived state, immutable updates.
- [ ] **HTTP/Interceptors**: Typed HTTP calls, interceptors for auth/error/logging, retry/backoff for transient failures.
- [ ] **Performance**: Lazy-loaded modules, virtual scrolling, avoid heavy work in templates, memoize inputs.
- [ ] **Accessibility & Localization**: Semantic templates, `cdkTrapFocus`, keyboard reachability, Angular i18n/RTL support.
- [ ] **Testing**: TestBed/Angular Testing Library with async utilities, interceptor/service tests via HttpTestingController, Storybook coverage.
- [ ] **Template Security**: Minimal `[innerHTML]`, sanitized DOM, cautious DomSanitizer usage, CSP compliance.

## Quick Reminders

- Reference sections `#accessibility-localization`, `#testing`, `#template-security`.
- Keep translation IDs (`i18n`) in sync with extracted message catalogs.
