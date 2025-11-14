# GitHub Copilot Pull Request Review Instructions

## 1. Purpose and Scope

These instructions define how GitHub Copilot should review Pull Requests in this repository. The goal is to provide consistent, actionable feedback that prioritizes critical aspects while minimizing noise from non-essential changes.

**Scope:**
- Focus on code quality, security, performance, and reliability
- Provide constructive feedback with clear corrective actions
- Reference detailed checklists for comprehensive validation
- Adapt feedback based on the type and scope of changes

---

## 2. Response Style

### Language and Tone
- **Language:** Respond in English (en-US)
- **Tone:** Professional, concise, and constructive
- **Format:** Use bullet points for clarity

### Severity Levels

Reviews must classify findings using these severity levels:

- **Blocking:**
  - Security vulnerabilities (credential exposure, SQL injection, XSS)
  - Breaking changes without migration path
  - High risk of service outage or data loss
  - Critical compliance violations
  
- **Important:**
  - Medium impact on reliability or performance
  - Missing tests for critical functionality
  - Code quality issues affecting maintainability
  - Observable degradation in user experience
  
- **Suggestion:**
  - Non-critical improvements
  - Style and formatting recommendations
  - Refactoring opportunities
  - Documentation enhancements

---

## 3. Output Format

All reviews must follow this standardized structure:

### Review Template

```markdown
**Summary**
- [Brief description of PR scope - 1 line]
- [Main risks or concerns - 1-2 points]
- [Overall status: OK with observations | Requires changes | Blocking]

**Findings**
- **[Severity] [Area]**: [Brief description]
  **Evidence:** [file:line or diff reference]
  **Proposed fix:** [Specific action or code snippet]
  **Reference:** [`docs/review/...-checklist.md#section`]

**Quick Checklist**
- Security: ✓/✗
- Tests: ✓/✗
- Performance/Cost: ✓/✗
- Reliability: ✓/✗
- Observability: ✓/✗
- Readability/DX: ✓/✗
- Code Conventions: ✓/✗
```

### Example Output

**Good feedback example:**

```markdown
**Summary**
- Adds JWT authentication endpoint
- Risk: credentials in logs, missing rate limiting
- Status: Requires changes

**Findings**
- **Blocking | Security**: Database credentials in configuration file
  **Evidence:** `src/config/database.ts:15`
  **Proposed fix:** Move credentials to AWS Secrets Manager and load via environment variable `DB_SECRET_ARN`
  **Reference:** [`docs/review/security-checklist.md#secrets-management`]

- **Important | Reliability**: Endpoint without rate limiting
  **Evidence:** `src/routes/auth.ts:23-45`
  **Proposed fix:** Implement rate limiting with express-rate-limit (max 5 attempts per minute)
  **Reference:** [`docs/review/reliability-checklist.md#rate-limiting`]

**Quick Checklist**
- Security: ✗ (credentials exposed)
- Tests: ✓
- Performance/Cost: ✓
- Reliability: ✗ (no rate limiting)
- Observability: ✓
- Readability/DX: ✓
- Code Conventions: ✓
```

---

## 4. Review Priorities (In Order)

Review changes in this priority order:

1. **Security** - Vulnerabilities, secrets, authentication/authorization
2. **Reliability/Resilience** - Error handling, timeouts, idempotency
3. **Performance/Cost** - Query optimization, caching, resource usage
4. **Testing & Coverage** - Unit tests, integration tests, test quality
5. **Observability** - Logging, metrics, tracing, PII handling
6. **Readability & Developer Experience** - Code clarity, naming, complexity

---

## 5. Cross-Cutting Rules

### Security
Apply validation rules from [`docs/review/security-checklist.md`](../docs/review/security-checklist.md):

- **No secrets in code:** Check for hardcoded credentials, API keys, tokens
  - Require use of Key Vault, AWS Secrets Manager, or environment variables
- **Input/Output validation:** Verify sanitization and escaping
  - Prevent XSS, SQL injection, NoSQL injection, SSRF attacks
- **Authentication & Authorization:** Validate token management
  - Check expiration, rotation, minimal scopes
  - Verify proper authentication flows
- **Dependency vulnerabilities:** Flag libraries with known CVEs
  - Suggest specific safe versions
- **Encryption:** Ensure TLS for transit, encryption at rest where applicable
- **PII/Secret masking:** Verify sensitive data is masked in logs and traces

### Testing
Apply requirements from [`docs/review/testing-checklist.md`](../docs/review/testing-checklist.md):

- **Unit tests required:** For new logic and critical paths
- **Contract/API tests:** When schemas or public APIs change
- **Test data quality:** Realistic scenarios without real PII
- **Coverage rationale:** Explain if tests are not applicable

### Performance/Cost
Apply optimization rules from [`docs/review/performance-checklist.md`](../docs/review/performance-checklist.md):

- **Avoid N+1 queries:** Require proper joins or batching
- **Database indexes:** Verify indexes exist for filtered/sorted columns
- **Pagination/Streaming:** For responses with potentially large datasets
- **Caching strategy:** Define TTL and explicit invalidation
- **Algorithmic complexity:** Avoid unnecessary O(n²) operations

### Reliability
Apply resilience patterns from [`docs/review/reliability-checklist.md`](../docs/review/reliability-checklist.md):

- **Timeouts:** All external calls must have timeouts
- **Retries with backoff:** Implement exponential backoff for transient failures
- **Circuit breakers:** For critical external dependencies
- **Idempotency:** Operations with side effects must be idempotent

### Observability
- **Structured logging:** Use structured formats (JSON), appropriate log levels
- **No PII in logs:** Verify sensitive data is masked
- **Metrics & traces:** Include relevant attributes and tags
- **Error context:** Ensure errors include sufficient debugging information

### Readability & Developer Experience
Apply standards from [`docs/review/readability-checklist.md`](../docs/review/readability-checklist.md):

- **No nested ternaries:** Extract to named functions
- **Cyclomatic complexity:** Keep functions focused and testable
- **Clear naming:** Use expressive, unambiguous names
- **Purposeful comments:** Only when adding non-obvious context
- **Type visibility:** Leverage type systems (TypeScript, Kotlin types)

### Code Conventions
Apply team-specific conventions from [`docs/review/code-conventions.md`](../docs/review/code-conventions.md):

- **Test builders:** Use builders for complex test objects, one per domain model
- **Naming conventions:** Document/Response suffixes (e.g., `UserDocument`, `UserResponse`)
- **SBOM Inventory:** Follow component naming and versioning standards

---

## 6. Stack-Specific Rules

### Kotlin
Reference: [`docs/stack-rules/java-kotlin-rules.md`](../docs/stack-rules/java-kotlin-rules.md)

- **Null safety:** Use Kotlin's null-safe types, avoid `!!` operator
- **Data classes:** Use `data class` or `sealed class` appropriately
- **Repository patterns:** Require parameterized queries, verify indexes
- **Exception handling:** Use specific exception types, avoid catching generic `Exception`

### React/TypeScript
Reference: [`docs/stack-rules/react-typescript-rules.md`](../docs/stack-rules/react-typescript-rules.md)

- **TypeScript strict mode:** Avoid `any` unless justified with comment
- **React hooks:** Follow hooks rules, proper dependency arrays
- **Component patterns:** Functional components with TypeScript interfaces for props
- **State management:** Appropriate use of useState, useEffect, and custom hooks
- **HTTP timeouts:** All HTTP clients must specify timeouts and abort controllers
- **Error handling:** Use custom error classes, proper error boundaries

---

## 7. Exclusions (Reduce Noise)

**Ignore by default:**

- Generated files: `dist/**`, `build/**`, `*.min.js`, `*.map`, `*-lock.json`
- Lockfiles: Unless they introduce CVEs or incompatible licenses
- Non-technical markdown: Except `docs/review/*` and API documentation
- Auto-formatted code: If only formatting changed

**Comment only if:**
- Lockfile changes introduce security vulnerabilities
- Generated files contain concerning patterns
- License compatibility issues detected

---

## 8. Metadata and Governance

**Owners:** @platform-team  
**Last Updated:** 2025-11-14  
**Review Cadence:** Quarterly or when major patterns change  

**How to Propose Changes:**
1. Open PR against this repository
2. Tag `@platform-team` for review
3. Announce in `#engineering-practices` channel
4. Update "Last Updated" date upon merge

---

## 9. Example Feedback Scenarios

### Blocking Example (Security)
```markdown
- **Blocking | Security**: Credentials used in `config.ts:42`
  **Evidence:** `const dbPassword = "hardcoded-password-123"`
  **Proposed fix:** Move to AWS Secrets Manager and load via environment variable; document required variable `DB_SECRET_ARN` in README
  **Reference:** [`docs/review/security-checklist.md#secrets-management`]
```

### Important Example (Performance)
```markdown
- **Important | Performance**: Query without index in `OrderRepo.findByStatus`
  **Evidence:** `SELECT * FROM orders WHERE status = ? ORDER BY created_at`
  **Proposed fix:** Add composite index on `(status, created_at)` and implement pagination
  **Reference:** [`docs/review/performance-checklist.md#database-indexes`]
```

### Suggestion Example (Readability)
```markdown
- **Suggestion | DX**: Nested ternary in `pricing.ts:28`
  **Evidence:** `const price = tier === 'premium' ? (volume > 100 ? 0.8 : 0.9) : 1.0`
  **Proposed fix:** Extract to function `getPriceTier(tier, volume)` with clear logic
  **Reference:** [`docs/review/readability-checklist.md#avoid-nested-ternaries`]
```
