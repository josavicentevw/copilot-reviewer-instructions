# React/TypeScript Review Cheat Sheet (Concise)

Reference: [`docs/stack-rules/react-typescript-rules.md`](../react-typescript-rules.md) · Examples: [`React/TypeScript Examples`](../examples-only/react-typescript-examples.md)

## Core Checks

- [ ] **TypeScript Strict**: No `any` without justification, interfaces/types defined, `strict` enabled.
- [ ] **Hooks**: Dependencies exhaustive, useMemo/useCallback to avoid churn, cleanup in effects/custom hooks.
- [ ] **Components**: Functional components with typed props, composition preferred, avoid derived state.
- [ ] **State/Context**: Independent state slices, functional updates, memoized context values/consumers.
- [ ] **HTTP/Data**: AbortController/timeouts, custom errors, retries, cleanup on unmount.
- [ ] **Performance**: React.memo/useMemo/useCallback for expensive operations, avoid inline objects.
- [ ] **Accessibility & i18n**: Semantic HTML, keyboard/focus handling, `aria-*`, localization helpers, sanitized HTML.
- [ ] **Testing**: RTL behavior tests, mocked HTTP (MSW), Storybook/visual regressions, end-to-end smoke coverage.
- [ ] **Resiliency**: Error boundaries around async trees, Suspense fallbacks, lazy-loaded bundles, request cancellation/retry UI.

## Quick Reminders

- Prefer composition/custom hooks over prop drilling for reusable logic.
- Keep HTTP clients/config in shared modules with logging/error handling.
- Reference sections `#accessibility-i18n`, `#testing`, `#resiliency` for deeper details.
