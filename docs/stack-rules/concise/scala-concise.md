# Scala Review Cheat Sheet (Concise)

Reference: [`docs/stack-rules/scala-rules.md`](../scala-rules.md) · Examples: [`Scala Examples`](../examples-only/scala-examples.md)

## Core Checks

- [ ] **Option/Null Safety**: Use `Option` instead of null, avoid `get`, prefer `map/flatMap/fold`.
- [ ] **Immutability**: Prefer `val`, immutable collections, case classes with `copy`.
- [ ] **Sealed Traits/ADTs**: Use sealed traits for closed hierarchies, exhaustive pattern matching.
- [ ] **For-Comprehensions**: Compose monadic operations cleanly, include guards, avoid deep nesting.
- [ ] **Either/Try**: Typed error handling, custom error ADTs, avoid exceptions for control flow.
- [ ] **Collections**: Functional operations, avoid loops/mutable buffers.
- [ ] **Futures**: Provide ExecutionContext, combine with `recover`, set timeouts, avoid blocking.
- [ ] **Type Classes/Implicits**: Keep implicit scopes focused, define type class instances properly.
- [ ] **Testing/Tooling**: ScalaTest/MUnit best practices, property tests, scalafmt/scalafix/wartremover enforcement.
- [ ] **Effect Systems**: Cats Effect/ZIO resources, cancellation, blocking boundaries.
- [ ] **Streaming**: fs2/Akka streams drained, back-pressure enforced, error handling/supervision.

## Quick Reminders

- Reference sections `#testing-tooling`, `#effect-systems`, `#streaming`.
- Use `sbt test`, `sbt fmt`, `sbt fix` in CI.
