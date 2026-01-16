# Kotlin/Java Review Cheat Sheet (Concise)

Reference: [`docs/stack-rules/java-kotlin-rules.md`](../java-kotlin-rules.md) · Examples: [`Kotlin/Java Examples`](../examples-only/java-kotlin-examples.md)

## Core Checks

- [ ] **Null Safety**: Kotlin nullable types, avoid `!!`, safe calls; Java Optional usage rules.
- [ ] **Data/Sealed Classes**: Use `data class` for immutability, sealed hierarchies for ADTs, exhaustive `when` statements.
- [ ] **Repositories**: Parameterized queries, indexes, avoid string concatenation in JPQL/SQL.
- [ ] **Dependency Injection**: Constructor injection, configuration properties, qualifiers for multiple beans.
- [ ] **Logging/Exceptions**: SLF4J with structured messages, no PII, custom exceptions + `@ControllerAdvice`.
- [ ] **Kotlin Idioms**: Use scope/extension functions wisely, avoid nested scope chains, prefer immutability.
- [ ] **Coroutines & Flow**: Structured concurrency (`viewModelScope`, `CoroutineScope`), dispatcher injection, lifecycle-aware Flow collection, avoid `GlobalScope`.
- [ ] **Testing & Serialization**: MockK/Kotest/Mockito tests with coroutine dispatchers, coroutine test utilities, serialization adapters verified (Kotlinx/Moshi).

## Quick Reminders

- Reference sections `#coroutines-flow` and `#testing-serialization` for more detail.
- Share dispatcher providers for swapping schedulers in tests.
