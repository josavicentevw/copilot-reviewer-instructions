# GitHub Copilot Pull Request Review Instructions

## 1. Purpose and Scope

These instructions define how GitHub Copilot should review Pull Requests in this repository. The goal is to provide consistent, actionable feedback that prioritizes critical aspects while minimizing noise from non-essential changes.

**Scope:**
- Focus on code quality, security, performance, and reliability
- Provide constructive feedback with clear corrective actions
- Adapt feedback based on the type and scope of changes

> **Note:** GitHub Copilot Review uses its own output format. These instructions guide **what to look for** and **how to prioritize findings**, not how to format the output.

---

## 2. Severity Classification

When reviewing code, classify findings by severity to help developers prioritize fixes:

### Blocking Issues (Must Fix)
Flag these as critical issues that should block the PR:
- Security vulnerabilities (credential exposure, SQL injection, XSS)
- Breaking changes without migration path
- High risk of service outage or data loss
- Critical compliance violations
- Null pointer exceptions in critical paths
- Missing authentication/authorization checks

### Important Issues (Should Fix)
Flag these as significant issues requiring attention:
- Medium impact on reliability or performance
- Missing tests for critical functionality
- Code quality issues affecting maintainability
- Observable degradation in user experience
- Missing error handling for external calls
- N+1 query patterns

### Suggestions (Consider Fixing)
Flag these as improvements to consider:
- Non-critical code quality improvements
- Refactoring opportunities
- Documentation enhancements
- Minor performance optimizations
- Code style recommendations

---

## 3. Review Priorities (In Order)

Review changes in this priority order:

1. **Security** - Vulnerabilities, secrets, authentication/authorization
2. **Reliability/Resilience** - Error handling, timeouts, idempotency
3. **Performance/Cost** - Query optimization, caching, resource usage
4. **Testing & Coverage** - Unit tests, integration tests, test quality
5. **Observability** - Logging, metrics, tracing, PII handling
6. **Readability & Developer Experience** - Code clarity, naming, complexity

---

## 4. Security Review Guidelines

Reference: [`docs/review/security-checklist.md`](../docs/review/security-checklist.md)

**Always check for:**

- **Hardcoded secrets:** API keys, passwords, tokens, credentials in code
  - Should use Key Vault, AWS Secrets Manager, or environment variables
- **SQL/NoSQL injection:** String concatenation in queries instead of parameterized queries
- **XSS vulnerabilities:** Unsanitized user input rendered in HTML
- **SSRF vulnerabilities:** User-controlled URLs in server-side requests
- **Missing authentication:** Endpoints without proper auth checks
- **Missing authorization:** Actions without permission verification
- **Insecure dependencies:** Libraries with known CVEs
- **PII in logs:** Sensitive data (emails, SSN, credit cards) logged without masking

**Example issues to flag:**
```
// BAD: Hardcoded credentials
const dbPassword = "secret123";

// BAD: SQL injection
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// BAD: PII in logs
console.log(`User logged in: ${user.email}, SSN: ${user.ssn}`);
```

---

## 5. Reliability Review Guidelines

Reference: [`docs/review/reliability-checklist.md`](../docs/review/reliability-checklist.md)

**Always check for:**

- **Missing timeouts:** HTTP calls, database queries without timeout configuration
- **Missing error handling:** Try-catch blocks that swallow exceptions silently
- **No retry logic:** External calls without retry with exponential backoff
- **No circuit breakers:** Critical external dependencies without circuit breaker pattern
- **Non-idempotent operations:** Operations with side effects that aren't idempotent
- **Missing rate limiting:** Public endpoints without rate limiting protection

**Example issues to flag:**
```
// BAD: No timeout
const response = await fetch(url);

// BAD: Swallowing exception
try {
  await processPayment(order);
} catch (e) {
  // Silent failure
}

// BAD: No retry logic for external call
const result = await externalApi.call(data);
```

---

## 6. Performance Review Guidelines

Reference: [`docs/review/performance-checklist.md`](../docs/review/performance-checklist.md)

**Always check for:**

- **N+1 queries:** Loops that execute database queries
- **Missing indexes:** Queries filtering on columns without indexes
- **Missing pagination:** Endpoints returning unbounded result sets
- **No caching:** Frequently accessed data without caching strategy
- **O(n²) algorithms:** Nested loops that could be optimized
- **Large payload responses:** APIs returning more data than needed
- **Missing connection pooling:** Database connections created per request

**Example issues to flag:**
```
// BAD: N+1 query
for (const user of users) {
  user.orders = await db.orders.findByUserId(user.id);
}

// BAD: No pagination
app.get('/users', async (req, res) => {
  const users = await db.users.findAll(); // Could return millions
  res.json(users);
});
```

---

## 7. Testing Review Guidelines

Reference: [`docs/review/testing-checklist.md`](../docs/review/testing-checklist.md)

**Always check for:**

- **Missing unit tests:** New logic without corresponding tests
- **Missing edge case tests:** Only happy path tested
- **Missing error case tests:** Exception paths not tested
- **Real PII in tests:** Production data or real personal information in test files
- **Flaky tests:** Tests with race conditions or external dependencies
- **Low test quality:** Tests that don't actually verify behavior

**Example issues to flag:**
```
// BAD: Test with real PII
const testUser = {
  email: "john.doe@company.com",
  ssn: "123-45-6789"
};

// BAD: Test without assertion
it('should process order', async () => {
  await processOrder(order);
  // No assertions!
});
```

---

## 8. Code Quality Review Guidelines

Reference: [`docs/review/readability-checklist.md`](../docs/review/readability-checklist.md)

**Always check for:**

- **Nested ternaries:** Complex conditional expressions that should be functions
- **Magic numbers/strings:** Hardcoded values without named constants
- **Overly complex functions:** Functions with high cyclomatic complexity
- **Poor naming:** Unclear variable, function, or class names
- **Code duplication:** Repeated logic that should be extracted
- **Missing type annotations:** Dynamic types where static types would help (any in TypeScript)

**Example issues to flag:**
```
// BAD: Nested ternary
const price = tier === 'premium' ? (volume > 100 ? 0.8 : 0.9) : 1.0;

// BAD: Magic numbers
if (retryCount > 3) { setTimeout(fn, 5000); }

// BAD: Using any
function processData(data: any): any {
  return data.value * 2;
}
```

---

## 9. Stack-Specific Guidelines

### Kotlin/Java
Reference: [`docs/stack-rules/java-kotlin-rules.md`](../docs/stack-rules/java-kotlin-rules.md)
Concise reference: [`Kotlin/Java Cheat Sheet`](../docs/stack-rules/concise/java-kotlin-concise.md)
Examples: [`Kotlin/Java Examples`](../docs/stack-rules/examples-only/java-kotlin-examples.md)

- Flag Kotlin `!!` operator usage without justification and Java `Optional.get()` without presence checks
- Require parameterized queries and constructor injection; no catching generic `Exception`
- Ensure structured concurrency (no `GlobalScope`), lifecycle-aware Flow collection, and dispatcher injection for tests
- Require MockK/Kotest/coroutine test dispatchers plus verified serialization adapters for DTOs
- Enforce SLF4J structured logging with PII masking across Kotlin/Java services

### Java
Reference: [`docs/stack-rules/java-rules.md`](../docs/stack-rules/java-rules.md)
Concise reference: [`Java Cheat Sheet`](../docs/stack-rules/concise/java-concise.md)
Examples: [`Java Examples`](../docs/stack-rules/examples-only/java-examples.md)

- Flag misuse of `Optional`, null annotations, or unchecked stream operations
- Require executor-based concurrency (no ad-hoc threads), CompletableFuture timeouts, and synchronized shared state
- Demand JUnit 5/Spring slice tests with Testcontainers for data/integration coverage
- Ensure security best practices: Bean validation, SSRF protections for RestTemplate/WebClient, secrets via configuration properties
- Verify logging/observability guidelines (MDC, masking) and immutability/records usage

### React/TypeScript
Reference: [`docs/stack-rules/react-typescript-rules.md`](../docs/stack-rules/react-typescript-rules.md)
Concise reference: [`React/TypeScript Cheat Sheet`](../docs/stack-rules/concise/react-typescript-concise.md)
Examples: [`React/TypeScript Examples`](../docs/stack-rules/examples-only/react-typescript-examples.md)

- Flag `any` type usage without justification comment and require typed props/interfaces
- Flag missing dependencies or cleanup in hooks and ensure state management avoids derived state
- Require HTTP calls to include AbortController/timeout handling and resiliency patterns (error boundaries, Suspense fallbacks, lazy loading)
- Flag components without accessibility coverage (semantic markup, keyboard/focus management, safe HTML) or missing localization strategy
- Require behavior-focused testing with React Testing Library/Jest plus mocked HTTP/Storybook/visual coverage for critical UI

### Python
Reference: [`docs/stack-rules/python-rules.md`](../docs/stack-rules/python-rules.md)
Concise reference: [`Python Cheat Sheet`](../docs/stack-rules/concise/python-concise.md)
Examples: [`Python Examples`](../docs/stack-rules/examples-only/python-examples.md)

- Require type hints for function signatures and forbid bare `except:`
- Require context managers for resources and parameterized SQL queries (no string formatting)
- Ensure async code uses `async/await`, proper timeouts, and pytest async fixtures/property tests
- Flag missing accessibility/localization in templates (semantic HTML, translation helpers)
- Enforce security practices: secrets via settings/env, bandit/ruff/mypy in CI, escape user content

### Go
Reference: [`docs/stack-rules/go-rules.md`](../docs/stack-rules/go-rules.md)
Concise reference: [`Go Cheat Sheet`](../docs/stack-rules/concise/go-concise.md)
Examples: [`Go Examples`](../docs/stack-rules/examples-only/go-examples.md)

- Flag ignored errors or missing wrapping/context propagation
- Require `context.Context` as the first parameter and enforce timeout/cancellation on HTTP/DB calls
- Flag goroutines without synchronization or leaked channels; enforce closing resources (`defer rows.Close()`, `resp.Body.Close()`)
- Ensure HTTP clients are reused with timeouts, retries/backoff, and `Abort` on cancellation
- Require structured logging/metrics/tracing plus environment-driven configuration and secrets management

### Scala
Reference: [`docs/stack-rules/scala-rules.md`](../docs/stack-rules/scala-rules.md)
Concise reference: [`Scala Cheat Sheet`](../docs/stack-rules/concise/scala-concise.md)
Examples: [`Scala Examples`](../docs/stack-rules/examples-only/scala-examples.md)

- Flag `Option.get` usage (prefer `getOrElse`, `fold`, `map`) and mutable vars/collections
- Require sealed traits for ADTs and Either/Try-based error handling
- Enforce ScalaTest/MUnit best practices plus property testing (no `Thread.sleep`, use `eventually`)
- Require effect-system safety (Cats Effect/ZIO Resource/ZManaged, no blocking default pools)
- Flag fs2/Akka streams without draining/back-pressure or error handling

### Angular
Reference: [`docs/stack-rules/angular-rules.md`](../docs/stack-rules/angular-rules.md)
Concise reference: [`Angular Cheat Sheet`](../docs/stack-rules/concise/angular-concise.md)
Examples: [`Angular Examples`](../docs/stack-rules/examples-only/angular-examples.md)

- Require TypeScript strict mode, OnPush change detection, and `trackBy` in `*ngFor`
- Require unsubscribe patterns for observables (takeUntil, async pipe) and typed HTTP/forms
- Flag missing accessibility/i18n coverage (semantic templates, keyboard focus, Angular i18n/RTL support)
- Require behavior-focused tests (TestBed/Angular Testing Library) with async utilities handled correctly
- Flag unsafe template patterns (`[innerHTML]` without sanitization, incorrect `DomSanitizer` usage, missing CSP considerations)

---

## 10. Files to Exclude from Review

**Skip reviewing these files:**

- Generated files: `dist/**`, `build/**`, `*.min.js`, `*.map`
- Lock files: `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` (unless CVE concerns)
- Auto-generated migrations (unless they modify production data)
- Third-party vendor code
- Files with only formatting changes

**Exception:** Review lock files if they introduce known vulnerabilities or license issues.

---

## 11. Code Conventions

Reference: [`docs/review/code-conventions.md`](../docs/review/code-conventions.md)

**Team-specific conventions to enforce:**

- **Test builders:** Use builders for complex test objects, one per domain model
- **Naming conventions:** 
  - Documents: `UserDocument`, `OrderDocument`
  - Responses: `UserResponse`, `OrderResponse`
  - Database entities: `UserDB`, `RoleDB`
- **SBOM Inventory:** Follow component naming: `{microservice}:{version}`

---

## 12. Governance

**Owners:** VW D:H  
**Last Updated:** 2026-01-15  
**Review Cadence:** Quarterly or when major patterns change  

**How to Propose Changes:**
1. Open PR against this repository
2. Tag `VW D:H` for review
3. Update "Last Updated" date upon merge
