# Go Review Cheat Sheet (Concise)

Reference: [`docs/stack-rules/go-rules.md`](../go-rules.md) · Examples: [`Go Examples`](../examples-only/go-examples.md)

## Core Checks

- [ ] **Error Handling**: Never ignore errors, wrap with context (`fmt.Errorf("%w")`), custom types + `errors.Is/As`.
- [ ] **Context Usage**: `context.Context` first arg, propagate through HTTP/DB, cancel long tasks, respect deadlines.
- [ ] **Goroutines & Concurrency**: Handle completion (WaitGroup/errgroup), avoid leaks, protect shared state, worker pools with cancellation.
- [ ] **Interfaces & DI**: Small interfaces defined at consumer, constructor injection, return structs, accept interfaces.
- [ ] **Database**: Prepared statements, `defer rows.Close()`, transactions with rollback, indexes, handle `sql.ErrNoRows`.
- [ ] **Struct/Method Design**: Pointer receivers for mutating methods, encapsulate sensitive fields, builder/constructor patterns.
- [ ] **Testing**: Table-driven tests, mocks/fakes, subtests, race detector, benchmark critical code.
- [ ] **HTTP & Networking**: Reuse clients with timeouts, context per request, close bodies, retries/backoff, streaming with `io.Copy`.
- [ ] **Observability**: Structured logging (zap/slog), metrics/tracing (otel), race/pprof diagnostics enabled.
- [ ] **Configuration/Security**: Env-based settings (viper/envconfig), secrets externalized, no hard-coded creds.

## Quick Reminders

- Run `go test ./...`, `go vet`, `golangci-lint`, `go test -race`.
- Reference sections `#http-networking`, `#observability`, `#configuration` for deeper details.
