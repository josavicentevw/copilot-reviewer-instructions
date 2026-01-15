# GitHub Copilot Pull Request Review Instructions

<!-- 
  CUSTOMIZATION GUIDE:
  This template is designed to be copied to your repository's .github/copilot-instructions.md
  
  Sections marked with [CUSTOMIZABLE] can be modified to fit your team's needs.
  Sections marked with [RECOMMENDED] contain best practices that should generally remain as-is.
  
  IMPORTANT: GitHub Copilot Review uses its own output format. These instructions guide 
  WHAT to look for and HOW to prioritize findings, not how to format the output.
  
  See templates/team-customization-guide.md for detailed instructions.
-->

---

## 1. Purpose and Scope

<!-- [RECOMMENDED] -->

These instructions define how GitHub Copilot should review Pull Requests in this repository. The goal is to provide consistent, actionable feedback that prioritizes critical aspects while minimizing noise from non-essential changes.

**Scope:**
- Focus on code quality, security, performance, and reliability
- Provide constructive feedback with clear corrective actions
- Adapt feedback based on the type and scope of changes

> **Note:** GitHub Copilot Review uses its own output format. These instructions guide **what to look for** and **how to prioritize findings**, not how to format the output.

---

## 2. Severity Classification

<!-- [CUSTOMIZABLE - Adjust severity thresholds based on your team's risk tolerance] -->

When reviewing code, classify findings by severity to help developers prioritize fixes:

### Blocking Issues (Must Fix)
Flag these as critical issues that should block the PR:
- Security vulnerabilities (credential exposure, SQL injection, XSS)
- Breaking changes without migration path
- High risk of service outage or data loss
- Critical compliance violations
- Null pointer exceptions in critical paths
- Missing authentication/authorization checks
<!-- CUSTOMIZE: Add team-specific blocking criteria here -->

### Important Issues (Should Fix)
Flag these as significant issues requiring attention:
- Medium impact on reliability or performance
- Missing tests for critical functionality
- Code quality issues affecting maintainability
- Observable degradation in user experience
- Missing error handling for external calls
- N+1 query patterns
<!-- CUSTOMIZE: Add team-specific important criteria here -->

### Suggestions (Consider Fixing)
Flag these as improvements to consider:
- Non-critical code quality improvements
- Refactoring opportunities
- Documentation enhancements
- Minor performance optimizations
- Code style recommendations
<!-- CUSTOMIZE: Add team-specific suggestion criteria here -->

---

## 3. Review Priorities (In Order)

<!-- [CUSTOMIZABLE - Reorder these priorities to match your team's focus areas] -->

Review changes in this priority order:

1. **Security** - Vulnerabilities, secrets, authentication/authorization
2. **Reliability/Resilience** - Error handling, timeouts, idempotency
3. **Performance/Cost** - Query optimization, caching, resource usage
4. **Testing & Coverage** - Unit tests, integration tests, test quality
5. **Observability** - Logging, metrics, tracing, PII handling
6. **Readability & Developer Experience** - Code clarity, naming, complexity

<!-- CUSTOMIZE: Add or reorder priorities based on your team's needs
Example additions:
7. **Accessibility** - WCAG compliance, keyboard navigation, screen readers
8. **Internationalization** - i18n support, locale handling
-->

---

## 4. Security Review Guidelines

<!-- [RECOMMENDED - Core security rules] -->

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

<!-- CUSTOMIZE: Add team-specific security requirements
Example:
- **Compliance:** GDPR, HIPAA, SOC 2 specific requirements
- **Secret rotation:** Maximum age for secrets (e.g., 90 days)
-->

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

<!-- [RECOMMENDED - Core reliability rules] -->

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

<!-- [RECOMMENDED - Core performance rules] -->

Reference: [`docs/review/performance-checklist.md`](../docs/review/performance-checklist.md)

**Always check for:**

- **N+1 queries:** Loops that execute database queries
- **Missing indexes:** Queries filtering on columns without indexes
- **Missing pagination:** Endpoints returning unbounded result sets
- **No caching:** Frequently accessed data without caching strategy
- **O(n²) algorithms:** Nested loops that could be optimized
- **Large payload responses:** APIs returning more data than needed
- **Missing connection pooling:** Database connections created per request

<!-- CUSTOMIZE: Add team-specific performance requirements
Example:
- **Response time SLA:** API endpoints must respond in <200ms (p95)
- **Bundle size:** Frontend bundles must be <500KB gzipped
-->

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

<!-- [RECOMMENDED - Core testing rules] -->

Reference: [`docs/review/testing-checklist.md`](../docs/review/testing-checklist.md)

**Always check for:**

- **Missing unit tests:** New logic without corresponding tests
- **Missing edge case tests:** Only happy path tested
- **Missing error case tests:** Exception paths not tested
- **Real PII in tests:** Production data or real personal information in test files
- **Flaky tests:** Tests with race conditions or external dependencies
- **Low test quality:** Tests that don't actually verify behavior

<!-- CUSTOMIZE: Add team-specific testing requirements
Example:
- **Minimum coverage:** 80% for new code, 70% for legacy
- **E2E tests:** Required for critical user journeys
- **Performance tests:** Required for endpoints handling >1000 req/min
-->

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

<!-- [RECOMMENDED - Core code quality rules] -->

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

## 9. Observability Guidelines

<!-- [CUSTOMIZABLE - Add team-specific observability requirements] -->

**Always check for:**

- **Structured logging:** Use structured formats (JSON), appropriate log levels
- **No PII in logs:** Verify sensitive data is masked
- **Metrics & traces:** Include relevant attributes and tags
- **Error context:** Ensure errors include sufficient debugging information

<!-- CUSTOMIZE: Add team-specific observability requirements
Example:
- **APM integration:** All services must report to DataDog/New Relic
- **SLO tracking:** Track availability, latency, and error rate SLOs
-->

---

## 10. Stack-Specific Guidelines

<!-- [CUSTOMIZABLE - Add/Remove based on your technology stacks] -->

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

<!-- CUSTOMIZE: Add your team's technology stacks
Example for Django:

### Django
Reference: [`docs/stack-rules/django-rules.md`](../docs/stack-rules/django-rules.md)

- Use select_related/prefetch_related for querysets
- Never commit secrets to settings.py
- Use Django REST Framework serializers properly
-->

---

## 11. Files to Exclude from Review

<!-- [CUSTOMIZABLE - Adjust based on your build tools and workflow] -->

**Skip reviewing these files:**

- Generated files: `dist/**`, `build/**`, `*.min.js`, `*.map`
- Lock files: `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` (unless CVE concerns)
- Auto-generated migrations (unless they modify production data)
- Third-party vendor code
- Files with only formatting changes

<!-- CUSTOMIZE: Add your team's exclusion patterns
Example:
- Auto-generated code: `**/*.generated.ts`, `**/migrations/*.py`
- Third-party code: `vendor/**`, `node_modules/**`
- Build artifacts: `target/**`, `out/**`, `.next/**`
-->

**Exception:** Review lock files if they introduce known vulnerabilities or license issues.

---

## 12. Code Conventions

<!-- [HIGHLY CUSTOMIZABLE - Replace with your team's conventions] -->

Reference: [`docs/review/code-conventions.md`](../docs/review/code-conventions.md)

**Team-specific conventions to enforce:**

- **Test builders:** Use builders for complex test objects, one per domain model
- **Naming conventions:** 
  - Documents: `UserDocument`, `OrderDocument`
  - Responses: `UserResponse`, `OrderResponse`
  - Database entities: `UserDB`, `RoleDB`
- **SBOM Inventory:** Follow component naming: `{microservice}:{version}`

<!-- CUSTOMIZE: Replace with your team's conventions
Example:
- **Error codes:** Use standardized format (ERR-{DOMAIN}-{NUMBER})
- **Feature flags:** Use LaunchDarkly with naming convention {team}-{feature}-{env}
- **API versioning:** Use URL versioning (v1, v2) not header-based
-->

---

## 13. Governance

<!-- [CUSTOMIZABLE - Update with your team's information] -->

**Owners:** VW D:H  <!-- CUSTOMIZE: Change to your team's handle -->
**Last Updated:** 2026-01-15  <!-- CUSTOMIZE: Update when making changes -->
**Review Cadence:** Quarterly or when major patterns change  

**How to Propose Changes:**
1. Open PR against this repository
2. Tag `VW D:H` for review  <!-- CUSTOMIZE: Change to your reviewers -->
3. Update "Last Updated" date upon merge

---

<!-- 
  NEXT STEPS AFTER CUSTOMIZING:
  
  1. Copy this file to .github/copilot-instructions.md in your repository
  2. Remove all <!-- CUSTOMIZE --> comments after making your changes
  3. Remove stack-specific sections that don't apply to your project
  4. Update the "Last Updated" date in section 13
  5. Test by creating a pull request and verifying Copilot uses your rules
  6. Iterate based on team feedback
  
  For detailed customization instructions, see:
  templates/team-customization-guide.md
-->
