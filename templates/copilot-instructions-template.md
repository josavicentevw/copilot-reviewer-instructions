# GitHub Copilot Pull Request Review Instructions

<!-- 
  CUSTOMIZATION GUIDE:
  This template is designed to be copied to your repository's .github/copilot-instructions.md
  
  Sections marked with [CUSTOMIZABLE] can be modified to fit your team's needs.
  Sections marked with [RECOMMENDED - DO NOT CHANGE] should remain as-is for consistency.
  
  See templates/team-customization-guide.md for detailed instructions.
-->

---

## 1. Purpose and Scope

<!-- [RECOMMENDED - DO NOT CHANGE] -->
<!-- This section defines the fundamental purpose of code reviews -->

These instructions define how GitHub Copilot should review Pull Requests in this repository. The goal is to provide consistent, actionable feedback that prioritizes critical aspects while minimizing noise from non-essential changes.

**Scope:**
- Focus on code quality, security, performance, and reliability
- Provide constructive feedback with clear corrective actions
- Reference detailed checklists for comprehensive validation
- Adapt feedback based on the type and scope of changes

---

## 2. Response Style

<!-- [CUSTOMIZABLE - Language] -->
<!-- Change language/locale if your team works in a different language -->

### Language and Tone
- **Language:** Respond in English (en-US)  <!-- CUSTOMIZE: Change to your team's language -->
- **Tone:** Professional, concise, and constructive
- **Format:** Use bullet points for clarity

### Severity Levels

<!-- [CUSTOMIZABLE - Severity Definitions] -->
<!-- Adjust severity thresholds based on your team's risk tolerance -->

Reviews must classify findings using these severity levels:

- **Blocking:**
  - Security vulnerabilities (credential exposure, SQL injection, XSS)
  - Breaking changes without migration path
  - High risk of service outage or data loss
  - Critical compliance violations
  <!-- CUSTOMIZE: Add team-specific blocking criteria here -->
  
- **Important:**
  - Medium impact on reliability or performance
  - Missing tests for critical functionality
  - Code quality issues affecting maintainability
  - Observable degradation in user experience
  <!-- CUSTOMIZE: Add team-specific important criteria here -->
  
- **Suggestion:**
  - Non-critical improvements
  - Style and formatting recommendations
  - Refactoring opportunities
  - Documentation enhancements
  <!-- CUSTOMIZE: Add team-specific suggestion criteria here -->

---

## 3. Output Format

<!-- [RECOMMENDED - DO NOT CHANGE] -->
<!-- Standard format ensures consistency across all reviews -->

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

<!-- [CUSTOMIZABLE - Priority Order] -->
<!-- Reorder these priorities to match your team's focus areas -->

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

## 5. Cross-Cutting Rules

<!-- [RECOMMENDED - DO NOT CHANGE CORE RULES] -->
<!-- These are fundamental best practices; customize only the specifics -->

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

<!-- CUSTOMIZE: Add team-specific security requirements
Example:
- **Compliance:** GDPR, HIPAA, SOC 2 specific requirements
- **Secret rotation:** Maximum age for secrets (e.g., 90 days)
-->

### Testing
Apply requirements from [`docs/review/testing-checklist.md`](../docs/review/testing-checklist.md):

- **Unit tests required:** For new logic and critical paths
- **Contract/API tests:** When schemas or public APIs change
- **Test data quality:** Realistic scenarios without real PII
- **Coverage rationale:** Explain if tests are not applicable

<!-- CUSTOMIZE: Add team-specific testing requirements
Example:
- **Minimum coverage:** 80% for new code, 70% for legacy
- **E2E tests:** Required for critical user journeys
- **Performance tests:** Required for endpoints handling >1000 req/min
-->

### Performance/Cost
Apply optimization rules from [`docs/review/performance-checklist.md`](../docs/review/performance-checklist.md):

- **Avoid N+1 queries:** Require proper joins or batching
- **Database indexes:** Verify indexes exist for filtered/sorted columns
- **Pagination/Streaming:** For responses with potentially large datasets
- **Caching strategy:** Define TTL and explicit invalidation
- **Algorithmic complexity:** Avoid unnecessary O(n²) operations

<!-- CUSTOMIZE: Add team-specific performance requirements
Example:
- **Response time SLA:** API endpoints must respond in <200ms (p95)
- **Bundle size:** Frontend bundles must be <500KB gzipped
-->

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

<!-- CUSTOMIZE: Add team-specific observability requirements
Example:
- **APM integration:** All services must report to DataDog/New Relic
- **SLO tracking:** Track availability, latency, and error rate SLOs
-->

### Readability & Developer Experience
Apply standards from [`docs/review/readability-checklist.md`](../docs/review/readability-checklist.md):

- **No nested ternaries:** Extract to named functions
- **Cyclomatic complexity:** Keep functions focused and testable
- **Clear naming:** Use expressive, unambiguous names
- **Purposeful comments:** Only when adding non-obvious context
- **Type visibility:** Leverage type systems (TypeScript, Kotlin types)

### Code Conventions
Apply team-specific conventions from [`docs/review/code-conventions.md`](../docs/review/code-conventions.md):

<!-- [HIGHLY CUSTOMIZABLE] -->
<!-- This section should reference your team's specific coding conventions -->

- **Test builders:** Use builders for complex test objects, one per domain model
- **Naming conventions:** Document/Response suffixes (e.g., `UserDocument`, `UserResponse`)
- **SBOM Inventory:** Follow component naming and versioning standards

<!-- CUSTOMIZE: Replace or extend with your team's conventions
Example:
- **Error codes:** Use standardized error code format (ERR-{DOMAIN}-{NUMBER})
- **Feature flags:** Use LaunchDarkly with naming convention {team}-{feature}-{env}
- **API versioning:** Use URL versioning (v1, v2) not header-based
-->

---

## 6. Stack-Specific Rules

<!-- [CUSTOMIZABLE - Add/Remove Stacks] -->
<!-- Add sections for the technology stacks your team uses -->

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

<!-- CUSTOMIZE: Add your team's technology stacks
Example for Python/Django:

### Python/Django
Reference: [`docs/stack-rules/python-django-rules.md`](../docs/stack-rules/python-django-rules.md)

- **Type hints:** All function signatures must have type hints
- **Django ORM:** Use select_related/prefetch_related, avoid N+1 queries
- **Settings:** Never commit secrets to settings.py, use environment variables
-->

---

## 7. Exclusions (Reduce Noise)

<!-- [HIGHLY CUSTOMIZABLE] -->
<!-- Modify this section based on your build tools and workflow -->

**Ignore by default:**

- Generated files: `dist/**`, `build/**`, `*.min.js`, `*.map`, `*-lock.json`
- Lockfiles: Unless they introduce CVEs or incompatible licenses
- Non-technical markdown: Except `docs/review/*` and API documentation
- Auto-formatted code: If only formatting changed

<!-- CUSTOMIZE: Add your team's exclusion patterns
Example:
- Auto-generated code: `**/*.generated.ts`, `**/migrations/*.py`
- Third-party code: `vendor/**`, `node_modules/**`
- Build artifacts: `target/**`, `out/**`, `.next/**`
-->

**Comment only if:**
- Lockfile changes introduce security vulnerabilities
- Generated files contain concerning patterns
- License compatibility issues detected

<!-- CUSTOMIZE: Add conditions when excluded files should be reviewed
Example:
- Migration files modify production data
- Generated OpenAPI clients have security implications
-->

---

## 8. Metadata and Governance

<!-- [CUSTOMIZABLE - Team Information] -->
<!-- Update with your team's information and processes -->

**Owners:** VW D:H  <!-- CUSTOMIZE: Change to your team's handle -->
**Last Updated:** 2025-11-14  <!-- CUSTOMIZE: Update when making changes -->
**Review Cadence:** Quarterly or when major patterns change  

**How to Propose Changes:**
1. Open PR against this repository  <!-- CUSTOMIZE: Change to your team's repo -->
2. Tag `VW D:H` for review  <!-- CUSTOMIZE: Change to your reviewers -->
3. Update "Last Updated" date upon merge

---

## 9. Example Feedback Scenarios

<!-- [RECOMMENDED - DO NOT CHANGE] -->
<!-- These examples demonstrate good review feedback -->

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

---

<!-- 
  NEXT STEPS AFTER CUSTOMIZING:
  
  1. Copy this file to .github/copilot-instructions.md in your repository
  2. Remove all <!-- CUSTOMIZE --> comments after making your changes
  3. Update the "Last Updated" date in section 8
  4. Test by creating a pull request and verifying Copilot uses your rules
  5. Iterate based on team feedback
  
  For detailed customization instructions, see:
  templates/team-customization-guide.md
-->
