# Java Review Cheat Sheet (Concise)

Reference: [`docs/stack-rules/java-rules.md`](../java-rules.md) · Examples: [`Java Examples`](../examples-only/java-examples.md)

## Core Checks

- [ ] **Null Safety**: Use Optional for nullable returns, never return null Optional, `@NonNull/@Nullable` annotations, `Objects.requireNonNull`.
- [ ] **Streams**: No side effects, no modifying collections mid-stream, prefer immutable collectors, close resource-backed streams.
- [ ] **Exceptions**: Specific exceptions, try-with-resources, custom domain errors, logging with context, no swallowed exceptions.
- [ ] **JPA/Repositories**: Parameterized queries, indexes, fetch types, EntityGraph to avoid N+1, projections for read-only views.
- [ ] **Dependency Injection**: Constructor injection, configuration properties validated, avoid field injection.
- [ ] **Logging**: SLF4J parameterized logging, MDC context, mask PII.
- [ ] **Immutability**: Records/data classes for DTOs, defensive copies, immutable collections.
- [ ] **Concurrency**: ExecutorService over raw threads, CompletableFuture chains with timeouts, synchronized shared state, avoid blocking reactive threads.
- [ ] **Testing**: JUnit 5, Spring slice tests, Testcontainers/integration coverage, Mockito/MockK with clear expectations.
- [ ] **Security/Validation**: Bean Validation annotations, input sanitization, SSRF-safe HTTP clients, secrets via config, no logging secrets.

## Quick Reminders

- Reference sections `#concurrency`, `#testing`, `#security` for nuance.
- Enforce build tooling: `mvn/gradle test`, SpotBugs, Checkstyle, PMD, SonarQube.
