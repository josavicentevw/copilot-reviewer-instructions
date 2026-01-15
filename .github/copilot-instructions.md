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

- Flag `!!` operator usage in Kotlin without justification
- Flag `Optional.get()` without `isPresent()` check in Java
- Require parameterized queries in repositories
- Flag catching generic `Exception` instead of specific types
- Require constructor injection over field injection

### React/TypeScript
Reference: [`docs/stack-rules/react-typescript-rules.md`](../docs/stack-rules/react-typescript-rules.md)

- Flag `any` type usage without justification comment
- Flag missing dependencies in useEffect/useCallback arrays
- Flag missing cleanup in useEffect (memory leaks)
- Require proper TypeScript interfaces for component props
- Flag HTTP calls without timeout/abort controller

### Python
Reference: [`docs/stack-rules/python-rules.md`](../docs/stack-rules/python-rules.md)

- Require type hints for function signatures
- Flag bare `except:` clauses
- Require context managers for resource management
- Flag string formatting in SQL queries (injection risk)
- Require async/await for I/O-bound operations

### Go
Reference: [`docs/stack-rules/go-rules.md`](../docs/stack-rules/go-rules.md)

- Flag ignored errors (using `_` for error return)
- Require context.Context as first parameter
- Flag goroutines without proper synchronization
- Require `defer rows.Close()` for database queries
- Flag missing error wrapping with context

### Scala
Reference: [`docs/stack-rules/scala-rules.md`](../docs/stack-rules/scala-rules.md)

- Flag `Option.get` usage (prefer `getOrElse`, `fold`, `map`)
- Flag `var` usage (prefer `val` for immutability)
- Require sealed traits for ADTs
- Flag mutable collections usage
- Require proper error handling with Either/Try

### Angular
Reference: [`docs/stack-rules/angular-rules.md`](../docs/stack-rules/angular-rules.md)

- Require OnPush change detection for performance
- Flag missing `trackBy` in `*ngFor` loops
- Require proper unsubscribe patterns for observables
- Flag manual subscriptions (prefer async pipe)
- Require typed reactive forms

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
