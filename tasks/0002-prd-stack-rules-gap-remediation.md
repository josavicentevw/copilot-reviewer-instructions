# PRD: Stack Rule Gap Remediation

## 1. Introduction / Overview
The stack-specific rule documents currently lack critical guidance in areas such as accessibility, testing expectations, security, observability, and concurrency. Reviewers rely on these documents to perform consistent, high-quality code reviews, yet several stack guides (React, Angular, Go, Java, Kotlin, Scala) miss reminders about common high-risk issues. This initiative will create new sections and augment existing ones so that every stack checklist covers the essential cross-cutting concerns (accessibility, testing, security) while remaining detailed enough for reviewers to apply immediately.

## 2. Goals
1. Ensure each of the six stack documents contains explicit best practices for accessibility, testing, and security (where relevant).
2. Introduce or update sections for other stack-specific gaps (observability in Go, concurrency in Java, coroutines in Kotlin, effect systems in Scala, etc.) so reviewers can catch frequent mistakes.
3. Provide actionable review checklists and examples that a junior reviewer can follow without additional tribal knowledge.

## 3. User Stories
- As a React reviewer, I need accessibility and testing rules so that I can flag missing ARIA labels or brittle tests without relying on personal experience.
- As an Angular reviewer, I want explicit guidance on template security and localization so that I can verify developers aren’t shipping unsafe bindings.
- As a Go reviewer, I need HTTP client, observability, and configuration checklists so I can catch hanging requests or missing structured logs.
- As a Java reviewer, I need concurrency, testing, and security best practices to ensure the service code scales safely and is covered by the proper tests.
- As a Kotlin reviewer, I need coroutine- and Flow-specific rules to ensure async code is structured and testable.
- As a Scala reviewer, I need testing, effect-system, and streaming guidance so I can review code bases that rely on Cats Effect/ZIO/fs2 confidently.

## 4. Functional Requirements
1. **React/TypeScript doc updates**
   - Add a dedicated Accessibility & i18n section covering semantic markup, keyboard traps, focus management, color contrast, localization, and safe HTML usage.
   - Add a Testing section outlining expectations for React Testing Library/Jest, mocking HTTP clients, avoiding implementation-detail assertions, and documenting required tools.
   - Update existing sections (e.g., HTTP or Component Patterns) with reminders about error boundaries, Suspense, and lazy loading for bundle-splitting.
2. **Angular doc updates**
   - Introduce Accessibility & Localization rules similar to React but tailored for Angular templates and CDK utilities.
   - Add a Testing section (Jasmine/Karma/Angular Testing Library) including async utilities, TestBed setup, and common pitfalls.
   - Create a Template Security & Sanitization subsection describing safe `[innerHTML]` usage, DomSanitizer patterns, and CSP considerations.
3. **Go doc updates**
   - Add an HTTP Client & Networking section covering `http.Client` reuse, timeouts, body closing, and context propagation.
   - Add Observability & Logging guidance (structured logging, metrics, tracing).
   - Add a Configuration & Secrets section detailing env var usage, `viper`/`envconfig`, and avoiding hard-coded credentials.
4. **Java doc updates**
   - Add Concurrency & Threading rules (thread pools, CompletableFuture, transactional boundaries, blocking warnings).
   - Add a Testing section that covers JUnit 5, Spring Boot slices, integration tests/Testcontainers, and mocking guidelines.
   - Add a Security & Validation section addressing input validation, SSRF mitigation, secrets handling, and secure logging practices.
5. **Kotlin doc updates**
   - Add a Coroutines & Flow section covering structured concurrency, dispatcher use, cancellation, exception handling, and Flow collection.
   - Add Testing guidance (MockK/Kotest, coroutine test dispatchers, Android ViewModel tests) and serialization/data-class copy caveats.
6. **Scala doc updates**
   - Add Testing & Tooling content (ScalaTest, property testing, async test helpers).
   - Add Effect Systems & Resource Safety coverage (Cats Effect, ZIO, `Resource`/`ZManaged` usage, interruption).
   - Add Streaming & Back-pressure rules for Akka Streams/fs2, including materialization, draining, and error handling.
7. **Checklist alignment**
   - Update each document’s “Review Checklist Summary” to include the new categories (Accessibility, Testing, Security, etc.) so reviewers see the gaps at a glance.
   - Ensure examples accompany each new section to maintain existing documentation standards.

## 5. Non-Goals
- Rewriting the general review checklists in `docs/review/`.
- Introducing new technology stacks beyond the six in scope.
- Automating linting or tooling changes; this effort is purely documentation.

## 6. Design Considerations
- Follow the existing Markdown structure (section headings, checklists with `[ ]`, code fences with language tags, “BAD/GOOD” examples).
- Keep new sections concise but actionable; avoid duplicating existing general guidance unless the stack-specific nuance differs.
- Maintain anchor links (`{#anchor}`) for each new section for referencing from `.github/copilot-instructions.md`.

## 7. Technical Considerations
- Each document change should be referenced in `.github/copilot-instructions.md` if new anchor sections are introduced.
- Ensure accessibility guidance references widely accepted standards (WCAG, ARIA) and testing guidance references preferred tools.
- Provide references/links at the end of each document when new topics are introduced (e.g., WCAG, React Testing Library docs, OpenTelemetry).

## 8. Success Metrics
1. Every stack document contains explicit checklist items that mention accessibility, testing, and security (where relevant).
2. Reviewers confirm (via internal feedback or pilot review) that the new sections reduce ad-hoc review comments about the previously missing topics.
3. Each document’s “Review Checklist Summary” now lists at least one item referencing the newly added sections.

## 9. Open Questions
1. Should any of the new sections be mirrored in the pull request template or general review checklist to reinforce the guidance?
2. Do we need design/system-team approval for the specific accessibility or security recommendations before publishing?
3. Are there additional stacks (e.g., mobile/Swift, backend frameworks) that should receive similar treatment in follow-up efforts?

