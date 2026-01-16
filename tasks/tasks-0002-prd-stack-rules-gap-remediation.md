## Relevant Files

- `docs/stack-rules/react-typescript-rules.md` - React/TypeScript checklist that needs new Accessibility, Testing, and resiliency guidance.
- `docs/stack-rules/angular-rules.md` - Angular checklist that must add accessibility/localization, testing, and template security sections.
- `docs/stack-rules/go-rules.md` - Go checklist that needs HTTP client/networking, observability, and configuration/secrets rules.
- `docs/stack-rules/java-rules.md` - Java checklist requiring concurrency, testing, and security updates.
- `docs/stack-rules/java-kotlin-rules.md` - Kotlin-specific additions for coroutines/Flow, testing, and serialization guidance.
- `docs/stack-rules/scala-rules.md` - Scala checklist to expand with testing tooling, effect systems, and streaming/back-pressure content.
- `docs/stack-rules/python-rules.md` - Python checklist needing accessibility/testing/security coverage similar to other stacks.
- `docs/stack-rules/concise` (new directory TBD) - Will host summarized versions of each stack document.
- `docs/stack-rules/examples-only` (new directory TBD) - Will host example-only references per stack/topic.
- `.github/copilot-instructions.md` - Must reference any new sections/anchors so reviewers know the documentation additions exist.

### Notes

- Follow the existing markdown style: numbered sections with `{#anchor}` ids, checklists with `[ ]`, and paired BAD/GOOD examples.
- Each new section should add at least one corresponding item to the document’s “Review Checklist Summary”.
- When adding new sections, update `.github/copilot-instructions.md` so the Stack-Specific Rules section links to the new anchors.

## Tasks

- [x] 1.0 Enhance React/TypeScript rules with accessibility, testing, and resiliency guidance
  - [x] 1.1 Draft a new “Accessibility & i18n” section covering semantic markup, keyboard/focus behavior, color contrast, localization, `aria-*` usage, and safe HTML patterns (include samples).
  - [x] 1.2 Add a “Testing” section describing expectations for React Testing Library/Jest, mocking data fetches, avoiding implementation-detail assertions, and using Storybook/visual tests where applicable.
  - [x] 1.3 Update the HTTP/components/performance portions with guidance on error boundaries, Suspense/data fetching, and lazy-loaded bundles (ensure at least one example).
  - [x] 1.4 Extend the “Review Checklist Summary” with accessibility, testing, and resiliency items and verify `.github/copilot-instructions.md` references the new sections.
- [x] 2.0 Expand Angular rules for accessibility/localization, testing, and template security
  - [x] 2.1 Create an Angular-specific “Accessibility & Localization” section covering ARIA usage, `cdkTrapFocus`, keyboard navigation, and i18n tooling (i18n pipes, translation extraction).
  - [x] 2.2 Add a “Testing” section outlining Jasmine/Karma/Angular Testing Library practices, async utilities (`fakeAsync`, `waitForAsync`), and common pitfalls (missing `tick`, improper TestBed setup).
  - [x] 2.3 Introduce a “Template Security & Sanitization” subsection on safe `[innerHTML]`, `DomSanitizer`, CSP, and preventing script injection.
  - [x] 2.4 Update the Angular checklist summary and `.github/copilot-instructions.md` entry to surface the new sections.
- [x] 3.0 Bolster Go rules with networking, observability, and configuration content
  - [x] 3.1 Add an “HTTP Client & Networking” section covering client reuse, timeouts, context propagation, closing bodies, and retry/backoff helpers.
  - [x] 3.2 Write an “Observability & Logging” section addressing structured logs, log levels, tracing (`otelhttp`, `trace.Span`), metrics, and race-detector/pprof expectations.
  - [x] 3.3 Introduce “Configuration & Secrets Management” guidance (env vars, `viper`/`envconfig`, secrets storage, avoiding hard-coded credentials).
  - [x] 3.4 Reflect these additions in the Go checklist summary and update `.github/copilot-instructions.md` references.
- [x] 4.0 Update Java and Kotlin rule files with concurrency, testing, security, and coroutine content
  - [x] 4.1 For Java: add a “Concurrency & Threading” section (thread pools, CompletableFuture, transactional boundaries, blocking caveats).
  - [x] 4.2 For Java: create a “Testing Strategy” section covering JUnit 5, Spring Boot slices, integration tests/Testcontainers, and mocking best practices.
  - [x] 4.3 For Java: add a “Security & Validation” section (input sanitization, SSRF defenses for RestTemplate/WebClient, secrets handling, logging PII).
  - [x] 4.4 For Kotlin: add a “Coroutines & Flow” section covering structured concurrency, dispatcher usage, cancellation, Flow collection, and error handling.
  - [x] 4.5 For Kotlin: add testing guidance (MockK/Kotest, coroutine test dispatchers, Android ViewModel testing) plus serialization/data-class copy caveats.
  - [x] 4.6 Update the checklist summaries in both Java and Kotlin files, and ensure `.github/copilot-instructions.md` references the new anchors.
- [x] 5.0 Strengthen Scala rules with testing, effect systems, and streaming/back-pressure guidance
  - [x] 5.1 Add a “Testing & Tooling” section (ScalaTest/MUnit, property testing, async helpers for Futures/effect types).
  - [x] 5.2 Introduce an “Effect Systems & Resource Safety” section covering Cats Effect `Resource`, ZIO `ZManaged`, fiber interruption, and blocking boundaries.
  - [x] 5.3 Add a “Streaming & Back-pressure” section for Akka Streams/fs2 (materialization, draining, error handling, cancellation).
  - [x] 5.4 Update the Scala checklist summary and `.github/copilot-instructions.md` to include the new topics.
- [x] 6.0 Align Python rules with the cross-cutting guidance
  - [x] 6.1 Add an “Accessibility & Localization” subsection (CLI/terminal output considerations, web frameworks, templating with proper semantics) if applicable.
  - [x] 6.2 Add a “Testing & Tooling” subsection covering pytest fixtures, async tests, property-based testing, and security linters (bandit).
  - [x] 6.3 Add a “Security & Configuration” subsection (secrets management, SQL injection guards, input validation, safe serialization).
  - [x] 6.4 Update the Python checklist summary and `.github/copilot-instructions.md` to reflect the new sections.
- [x] 7.0 Produce concise reference versions of each stack rule document
  - [x] 7.1 Create a summarized/concise markdown version of every stack file (React, Angular, Go, Java, Kotlin, Scala, Python) capturing only checklists and key bullets.
  - [x] 7.2 Ensure the concise docs live under a dedicated directory (e.g., `docs/stack-rules/concise/`) and cross-link them from the main README/instructions.
- [x] 8.0 Produce example-only companion documents for each stack
  - [x] 8.1 Extract the ✅/❌ examples from every stack file into example-only markdowns (grouped by topic) under `docs/stack-rules/examples-only/`.
  - [x] 8.2 Cross-link the example-only docs from the primary stack files (and concise versions) so reviewers can jump straight to code samples.
