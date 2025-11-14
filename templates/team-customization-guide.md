# Team Customization Guide

This guide helps teams adopt and customize the GitHub Copilot PR review system for their specific needs.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Step-by-Step Customization Process](#step-by-step-customization-process)
3. [Common Customization Scenarios](#common-customization-scenarios)
4. [Team-Specific Rule Examples](#team-specific-rule-examples)
5. [Adding New Technology Stacks](#adding-new-technology-stacks)
6. [Customizing Review Priorities](#customizing-review-priorities)
7. [Troubleshooting](#troubleshooting)
8. [Best Practices](#best-practices)
9. [Maintenance and Updates](#maintenance-and-updates)

---

## Quick Start

**For teams who want to get started immediately:**

1. **Copy the template** to your repository:
   ```bash
   mkdir -p .github
   cp templates/copilot-instructions-template.md .github/copilot-instructions.md
   ```

2. **Customize basic settings** (5 minutes):
   - Update team name and contact information
   - Set your default language (if not English)
   - Adjust severity level thresholds

3. **Add your technology stacks** (10-15 minutes):
   - Reference existing stack rules or create new ones
   - Add links to your tech stack documentation

4. **Test with a sample PR**:
   - Create a test PR and review Copilot's feedback
   - Iterate on instructions based on results

5. **Announce to team**:
   - Share customization in team channel
   - Document any team-specific conventions

---

## Step-by-Step Customization Process

### Step 1: Initial Setup

**Goal**: Create your team's customized instruction file

#### 1.1 Copy the Template

```bash
# Navigate to your repository root
cd /path/to/your/repo

# Create .github directory if it doesn't exist
mkdir -p .github

# Copy the template
cp /path/to/copilot-reviewer-instructions/templates/copilot-instructions-template.md .github/copilot-instructions.md
```

**Why this location?** GitHub Copilot automatically reads `.github/copilot-instructions.md` for PR reviews.

#### 1.2 Open and Review

Open `.github/copilot-instructions.md` in your editor and review:
- All `<!-- CUSTOMIZE -->` comments
- All `[CUSTOMIZABLE]` and `[RECOMMENDED]` markers
- Inline `CUSTOMIZE:` guidance

---

### Step 2: Basic Configuration

**Goal**: Set fundamental team settings

#### 2.1 Update Metadata (Section 8)

```markdown
## 8. Metadata and Governance

**Owners:** @your-team-name             <!-- CUSTOMIZE: Your team handle -->
**Last Updated:** 2025-01-XX             <!-- CUSTOMIZE: Update date -->
**Review Cadence:** Quarterly            <!-- CUSTOMIZE: Your review schedule -->
```

#### 2.2 Set Language and Tone (Section 2)

Most teams keep English, but you can change:

```markdown
### Language and Tone
- **Language:** Respond in Spanish (es-ES)   <!-- If your team prefers Spanish -->
- **Tone:** Professional, concise, and constructive
- **Format:** Use bullet points for clarity
```

#### 2.3 Adjust Severity Levels (Section 2)

Review and modify if needed:

```markdown
- **Blocking:**
  - Security vulnerabilities (credential exposure, SQL injection, XSS)
  - Breaking changes without migration path
  <!-- CUSTOMIZE: Add team-specific blocking criteria -->
  - [Your criteria: e.g., "Missing required documentation for public APIs"]
```

**Example customization:**

```markdown
- **Blocking:**
  - Security vulnerabilities (credential exposure, SQL injection, XSS)
  - Breaking changes without migration path
  - Production database migrations without rollback plan
  - Missing E2E tests for critical user flows
  - Public API changes without version bump
```

---

### Step 3: Configure Review Priorities

**Goal**: Align review focus with team priorities

#### 3.1 Review Default Priorities (Section 4)

The template provides:
1. Security
2. Reliability/Resilience
3. Performance/Cost
4. Testing & Coverage
5. Observability
6. Readability & Developer Experience

#### 3.2 Reorder or Add Priorities

**Example: Team prioritizing performance**

```markdown
## 4. Review Priorities (In Order)

Review changes in this priority order:

1. **Security** - Vulnerabilities, secrets, authentication/authorization
2. **Performance/Cost** - Query optimization, caching, resource usage  <!-- Moved up -->
3. **Reliability/Resilience** - Error handling, timeouts, idempotency
4. **Testing & Coverage** - Unit tests, integration tests, test quality
5. **Observability** - Logging, metrics, tracing, PII handling
6. **Readability & Developer Experience** - Code clarity, naming, complexity
```

**Example: Team adding new priority**

```markdown
## 4. Review Priorities (In Order)

Review changes in this priority order:

1. **Security** - Vulnerabilities, secrets, authentication/authorization
2. **Accessibility (a11y)** - WCAG compliance, keyboard navigation, screen readers  <!-- NEW -->
3. **Reliability/Resilience** - Error handling, timeouts, idempotency
4. **Performance/Cost** - Query optimization, caching, resource usage
5. **Testing & Coverage** - Unit tests, integration tests, test quality
6. **Observability** - Logging, metrics, tracing, PII handling
7. **Readability & Developer Experience** - Code clarity, naming, complexity
```

---

### Step 4: Add Technology Stack Rules

**Goal**: Configure rules for your team's tech stack

#### 4.1 Review Available Stack Rules

Check which stack rules exist in `docs/stack-rules/`:
- `java-kotlin-rules.md` - For Kotlin/Java projects
- `react-typescript-rules.md` - For React/TypeScript projects
- `README.md` - Guide for adding new stacks

#### 4.2 Reference Existing Stack Rules

**Example: Kotlin backend + React frontend**

```markdown
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
```

#### 4.3 Add New Stack Rules (If Needed)

If your team uses a technology not covered, create new rules following the guide in `docs/stack-rules/README.md`.

**Example: Adding Python/Django rules**

1. Create `docs/stack-rules/python-django-rules.md` (see README for template)
2. Reference it in your instructions:

```markdown
### Python/Django
Reference: [`docs/stack-rules/python-django-rules.md`](../docs/stack-rules/python-django-rules.md)

- **Type hints:** Use type hints for all function signatures
- **Django ORM:** Use parameterized queries, avoid raw SQL
- **Settings:** Use environment variables, never hardcode secrets
- **Migrations:** Always provide rollback migrations
```

---

### Step 5: Customize Cross-Cutting Rules

**Goal**: Adapt security, testing, and other rules to your context

#### 5.1 Security Rules Customization

**Example: Adding team-specific secret patterns**

```markdown
### Security
Apply validation rules from [`docs/review/security-checklist.md`](../docs/review/security-checklist.md):

- **No secrets in code:** Check for hardcoded credentials, API keys, tokens
  - Require use of AWS Secrets Manager (team uses AWS)
  - Check for Stripe API keys (team uses Stripe)
  - Verify Datadog API keys not in code
```

#### 5.2 Testing Requirements Customization

**Example: Team-specific coverage thresholds**

```markdown
### Testing
Apply requirements from [`docs/review/testing-checklist.md`](../docs/review/testing-checklist.md):

- **Unit tests required:** For new logic and critical paths
- **Coverage thresholds:** 
  - Critical modules (payment, auth): 95%+
  - Business logic: 80%+
  - UI components: 70%+
- **Contract/API tests:** Required for all public API changes
```

#### 5.3 Performance Rules Customization

**Example: Team-specific performance budgets**

```markdown
### Performance/Cost
Apply optimization rules from [`docs/review/performance-checklist.md`](../docs/review/performance-checklist.md):

- **Database queries:** 
  - All list endpoints must use pagination
  - Query timeout: 5 seconds max
  - N+1 queries are blocking issues
- **API response time:** 
  - P95 latency < 200ms for read operations
  - P95 latency < 500ms for write operations
```

---

### Step 6: Configure Exclusions

**Goal**: Reduce noise from non-essential changes

#### 6.1 Review Default Exclusions (Section 7)

The template excludes:
- Generated files (`dist/**`, `build/**`, `*.min.js`)
- Lockfiles (unless CVEs or license issues)
- Non-technical markdown
- Auto-formatted code

#### 6.2 Add Team-Specific Exclusions

**Example: Team-specific generated files**

```markdown
**Ignore by default:**

- Generated files: `dist/**`, `build/**`, `*.min.js`, `*.map`, `*-lock.json`
- GraphQL generated: `src/generated/**`, `*.graphql.ts`        <!-- Team uses GraphQL codegen -->
- Protobuf generated: `src/proto/**/*.pb.ts`                   <!-- Team uses protobuf -->
- Database migrations: `migrations/**` (review only schema changes)
- Lockfiles: Unless they introduce CVEs or incompatible licenses
- Non-technical markdown: Except `docs/review/*` and API documentation
- Auto-formatted code: If only formatting changed
```

---

### Step 7: Add Team-Specific Conventions

**Goal**: Document your team's coding standards

#### 7.1 Use Section 6 for Team Conventions

```markdown
## 6. Stack-Specific Rules

<!-- ... existing stack rules ... -->

### Team-Specific Conventions

#### Naming Conventions
- **Components:** PascalCase (`UserProfile`, `PaymentForm`)
- **Hooks:** camelCase with `use` prefix (`useAuth`, `useFetchData`)
- **Constants:** UPPER_SNAKE_CASE (`API_TIMEOUT`, `MAX_RETRIES`)
- **Database tables:** snake_case (`user_profiles`, `payment_transactions`)

#### File Organization
- **Test files:** Co-located with source files (`UserService.ts`, `UserService.test.ts`)
- **Component structure:** 
  ```
  components/
    UserProfile/
      UserProfile.tsx
      UserProfile.test.tsx
      UserProfile.module.css
      index.ts
  ```

#### Git Commit Messages
- Follow Conventional Commits: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`
- Reference ticket: `feat: add payment processing (PROJ-123)`

#### Pull Request Requirements
- **Title:** Must reference ticket number
- **Description:** Use PR template, include testing notes
- **Size:** Max 400 lines changed (excluding generated files)
- **Tests:** Must include tests for new functionality
```

---

## Common Customization Scenarios

### Scenario 1: Startup Team (Speed > Perfection)

**Goal**: Fast iteration with essential safeguards

```markdown
## 4. Review Priorities (In Order)

1. **Security** - Blocking issues only (credentials, XSS, SQL injection)
2. **Reliability/Resilience** - Focus on error handling for critical paths
3. **Testing & Coverage** - Unit tests for business logic (70% threshold)
4. **Readability & Developer Experience** - Keep it simple

**Adjusted Severity Levels:**

- **Blocking:**
  - Security vulnerabilities
  - Breaking changes to public APIs
  
- **Important:**
  - Missing error handling in critical paths
  - No tests for new business logic
  
- **Suggestion:**
  - Performance optimizations
  - Code style improvements
  - Documentation enhancements
```

### Scenario 2: Enterprise Team (High Compliance)

**Goal**: Comprehensive reviews with strict standards

```markdown
## 4. Review Priorities (In Order)

1. **Security** - Full OWASP Top 10 validation
2. **Compliance** - GDPR, SOC2, HIPAA requirements  <!-- NEW priority -->
3. **Reliability/Resilience** - 99.99% uptime requirements
4. **Testing & Coverage** - 90%+ coverage, E2E tests required
5. **Observability** - Full tracing, audit logs for all operations
6. **Performance/Cost** - Cost optimization mandatory
7. **Readability & Developer Experience** - Maintainability critical

**Adjusted Severity Levels:**

- **Blocking:**
  - Security vulnerabilities
  - Compliance violations (PII leaks, missing audit logs)
  - Missing tests for critical paths
  - Production database changes without DBA review
  - Breaking changes without 6-month deprecation period
  
- **Important:**
  - Performance regressions
  - Missing observability (logs, metrics, traces)
  - Code complexity > 10
  - Missing API documentation
```

### Scenario 3: Open Source Project

**Goal**: Community-friendly, well-documented code

```markdown
## 2. Response Style

### Language and Tone
- **Language:** Respond in English (en-US)
- **Tone:** Welcoming and educational (explain "why" for contributors)
- **Format:** Use bullet points, provide learning resources

## 4. Review Priorities (In Order)

1. **Readability & Developer Experience** - Code clarity for new contributors
2. **Documentation** - README, API docs, code comments  <!-- NEW priority -->
3. **Testing & Coverage** - Examples and test cases
4. **Security** - Vulnerabilities, especially in dependencies
5. **Performance/Cost** - Resource efficiency
6. **Reliability/Resilience** - Error handling

**Custom Review Notes:**
- Provide links to project contribution guide
- Suggest improvements with examples
- Thank contributors for their time
- Explain architectural decisions when relevant
```

### Scenario 4: Mobile App Team

**Goal**: Performance, offline support, and UX focus

```markdown
## 4. Review Priorities (In Order)

1. **Performance/Cost** - Battery usage, memory efficiency, network calls
2. **User Experience** - Loading states, error messages, accessibility
3. **Offline Support** - Data persistence, sync strategies  <!-- NEW priority -->
4. **Security** - Secure storage, API security
5. **Testing & Coverage** - Unit tests, UI tests
6. **Reliability/Resilience** - Network failure handling

**Mobile-Specific Rules:**

### React Native
- **Bundle size:** Monitor JS bundle size, lazy load when possible
- **Network calls:** 
  - All API calls must have timeout (5 seconds)
  - Implement retry logic with exponential backoff
  - Cache responses when appropriate
- **Offline support:**
  - Use AsyncStorage or SQLite for persistence
  - Implement sync queue for offline actions
- **Performance:**
  - Use FlatList for long lists (not ScrollView)
  - Optimize images (use appropriate sizes, caching)
  - Avoid unnecessary re-renders (React.memo, useMemo)
```

---

## Team-Specific Rule Examples

### Example 1: Payment Processing Team

```markdown
### Payment Processing Requirements

**Blocking Issues:**
- **Idempotency:** All payment endpoints must use idempotency keys
  ```typescript
  router.post('/api/payments', async (req, res) => {
    const idempotencyKey = req.headers['idempotency-key'];
    if (!idempotencyKey) {
      return res.status(400).json({ error: 'Idempotency-Key required' });
    }
    // ...
  });
  ```

- **PCI Compliance:** No credit card data in logs, database, or error messages
  - Mask card numbers: `****-****-****-1234`
  - Use payment gateway tokens only

- **Testing:** Payment flows must have integration tests with test mode API keys

**Reference:** [`docs/team/payment-security.md`]
```

### Example 2: Data Platform Team

```markdown
### Data Pipeline Requirements

**Important Issues:**
- **Schema evolution:** Database migrations must be backward compatible
  - Add columns as nullable first
  - Deprecate old columns before removing
  - Provide migration scripts for existing data

- **Data quality:** All ETL jobs must include data validation
  ```python
  def validate_user_data(df: DataFrame) -> DataFrame:
      assert df['email'].notna().all(), "Email cannot be null"
      assert df['created_at'] <= datetime.now(), "Created date cannot be future"
      return df
  ```

- **Observability:** All pipelines must emit metrics
  - Records processed
  - Processing time
  - Error count and types

**Reference:** [`docs/team/data-quality-standards.md`]
```

### Example 3: Microservices Team

```markdown
### Microservices Requirements

**Blocking Issues:**
- **Service contracts:** OpenAPI spec updated for API changes
- **Breaking changes:** Require API versioning (`/v1/users`, `/v2/users`)
- **Circuit breakers:** All inter-service calls must use circuit breakers
  ```kotlin
  @CircuitBreaker(name = "userService", fallbackMethod = "getUserFallback")
  fun getUser(id: String): User {
      return userServiceClient.getUser(id)
  }
  ```

**Important Issues:**
- **Timeouts:** All service calls must have timeouts (default 5s)
- **Distributed tracing:** All endpoints must propagate trace context
- **Service mesh:** Use Istio annotations for traffic management

**Reference:** [`docs/team/microservices-patterns.md`]
```

---

## Adding New Technology Stacks

If your team uses a technology not currently covered, follow these steps:

### Step 1: Check Existing Stack Rules

Review `docs/stack-rules/README.md` to see if similar stack rules exist that you can adapt.

### Step 2: Create New Stack Rule Document

Create a new file in `docs/stack-rules/` following the template structure:

```markdown
# [Technology Name] Stack-Specific Rules

## 1. [Core Feature 1]
- [ ] Checklist item
- [ ] Another item

**Examples:**
```[language]
// ❌ BAD
code example

// ✅ GOOD
code example
```

## 2. [Core Feature 2]
...

## Review Checklist Summary
- [ ] Feature 1 checked
- [ ] Feature 2 checked
```

### Step 3: Reference in Instructions

Add reference in Section 6 of your `.github/copilot-instructions.md`:

```markdown
### [Technology Name]
Reference: [`docs/stack-rules/[technology]-rules.md`](../docs/stack-rules/[technology]-rules.md)

- **Key rule 1:** Brief description
- **Key rule 2:** Brief description
```

**See `docs/stack-rules/README.md` for complete walkthrough and Python/Django example.**

---

## Customizing Review Priorities

### Default Priority Order

The template provides this default order:
1. Security
2. Reliability/Resilience
3. Performance/Cost
4. Testing & Coverage
5. Observability
6. Readability & Developer Experience

### When to Customize

Consider reordering or adding priorities when:
- **Your domain has unique requirements** (e.g., accessibility for public-facing apps)
- **Your team has specific quality gates** (e.g., compliance for healthcare apps)
- **Your product stage dictates focus** (e.g., MVP vs mature product)

### How to Add New Priorities

1. **Identify the new priority** (e.g., Accessibility, Compliance, Documentation)
2. **Define what it includes** (clear criteria)
3. **Add to Section 4** in priority order
4. **Create supporting checklist** (optional, in `docs/review/` if extensive)

**Example: Adding Accessibility Priority**

```markdown
## 4. Review Priorities (In Order)

1. **Security** - Vulnerabilities, secrets, authentication/authorization
2. **Accessibility (a11y)** - WCAG 2.1 AA compliance
3. **Reliability/Resilience** - Error handling, timeouts, idempotency
4. **Performance/Cost** - Query optimization, caching, resource usage
5. **Testing & Coverage** - Unit tests, integration tests, test quality
6. **Observability** - Logging, metrics, tracing, PII handling
7. **Readability & Developer Experience** - Code clarity, naming, complexity

---

## 5. Cross-Cutting Rules

### Accessibility
Apply validation rules from [`docs/review/accessibility-checklist.md`](../docs/review/accessibility-checklist.md):

- **Semantic HTML:** Use appropriate HTML5 elements (`<nav>`, `<main>`, `<button>`)
- **ARIA labels:** All interactive elements have accessible names
- **Keyboard navigation:** All functionality accessible via keyboard
- **Color contrast:** WCAG AA minimum contrast ratios (4.5:1 for normal text)
- **Screen reader testing:** Test with VoiceOver (macOS) or NVDA (Windows)
```

Then optionally create `docs/review/accessibility-checklist.md` with detailed rules.

---

## Troubleshooting

### Issue: Copilot Reviews Are Too Noisy

**Symptoms:** Too many minor suggestions, hard to find important feedback

**Solutions:**
1. **Adjust severity thresholds** - Raise the bar for "Suggestion" level feedback
2. **Expand exclusions** - Add more file patterns to ignore (Section 7)
3. **Focus priorities** - Reduce number of priorities or reorder to focus on critical areas
4. **Update cross-cutting rules** - Add explicit guidance to skip non-critical issues

**Example fix:**
```markdown
## 2. Response Style

### Severity Levels

- **Suggestion:**
  - Non-critical improvements
  - Style and formatting recommendations (ONLY if violating team standards)
  - Refactoring opportunities (ONLY if complexity > 15)
  - Documentation enhancements (ONLY for public APIs)
```

### Issue: Copilot Misses Important Issues

**Symptoms:** Critical bugs or security issues not flagged

**Solutions:**
1. **Check priority order** - Ensure critical areas are prioritized (Section 4)
2. **Add explicit rules** - Make blocking criteria very specific (Section 2)
3. **Reference checklists** - Ensure cross-cutting rules reference detailed checklists (Section 5)
4. **Add examples** - Provide bad/good code examples in team conventions (Section 6)

**Example fix:**
```markdown
### Security
Apply validation rules from [`docs/review/security-checklist.md`](../docs/review/security-checklist.md):

- **No secrets in code:** Check for hardcoded credentials, API keys, tokens
  - **BLOCKING:** Search for patterns: `password =`, `api_key =`, `secret =`, `token =`
  - **BLOCKING:** Check for AWS keys: `AKIA[A-Z0-9]{16}`
  - **BLOCKING:** Check for private keys: `BEGIN RSA PRIVATE KEY`
```

### Issue: Stack-Specific Rules Not Applied

**Symptoms:** Language-specific best practices not enforced

**Solutions:**
1. **Verify stack rules exist** - Check `docs/stack-rules/` directory
2. **Add explicit references** - Link to stack rules in Section 6
3. **Provide examples** - Add bad/good code examples in stack rules
4. **Test with sample PR** - Create PR with known issues to verify

**Example fix:**
```markdown
### React/TypeScript
Reference: [`docs/stack-rules/react-typescript-rules.md`](../docs/stack-rules/react-typescript-rules.md)

**Critical Rules:**
- **TypeScript strict mode:** Avoid `any` unless justified with comment
  - **BLOCKING:** Using `any` without `// Using any because...` comment
  
- **React hooks:** Follow hooks rules, proper dependency arrays
  - **BLOCKING:** Missing dependencies in useEffect (enable eslint rule)
  
- **HTTP timeouts:** All HTTP clients must specify timeouts
  - **BLOCKING:** `fetch()` or `axios()` calls without timeout parameter
```

### Issue: Too Many False Positives

**Symptoms:** Copilot flags valid code as problematic

**Solutions:**
1. **Add context to rules** - Specify when rules apply or don't apply
2. **Provide exceptions** - Document acceptable exceptions
3. **Update exclusions** - Exclude file patterns that trigger false positives
4. **Refine examples** - Improve bad/good examples to be more specific

**Example fix:**
```markdown
### Testing
Apply requirements from [`docs/review/testing-checklist.md`](../docs/review/testing-checklist.md):

- **Unit tests required:** For new logic and critical paths
  - **Exceptions:** 
    - Simple DTOs and data classes
    - Generated code (GraphQL, Protobuf)
    - Configuration files
    - Type definitions without logic
```

### Issue: Reviews Too Slow or Incomplete

**Symptoms:** Copilot takes too long or times out on large PRs

**Solutions:**
1. **Reduce scope** - Expand exclusions to skip large generated files
2. **Simplify rules** - Remove overly complex or redundant rules
3. **Split PRs** - Encourage smaller PRs (add to team conventions)
4. **Focus on changes** - Add guidance to focus only on diff, not entire files

**Example fix:**
```markdown
## 7. Exclusions (Reduce Noise)

**Ignore by default:**

- Generated files: `dist/**`, `build/**`, `*.min.js`, `*.map`, `*-lock.json`
- Large data files: `**/*.json` (if > 1000 lines)  <!-- NEW -->
- Vendor code: `vendor/**`, `node_modules/**`     <!-- NEW -->
- Documentation: `docs/**/*.md` (except review checklists)

**Focus on diff only:**
- Review only changed lines and surrounding context (±5 lines)
- Don't review unchanged code unless directly related to changes
```

---

## Best Practices

### 1. Start Simple, Iterate

- **Don't customize everything at once** - Start with basics (metadata, language, priorities)
- **Test with sample PRs** - Create PRs with known issues to verify behavior
- **Gather team feedback** - Review with team after 1-2 weeks
- **Iterate incrementally** - Add customizations based on real feedback

### 2. Document Team Conventions

- **Keep conventions in instructions** - Don't rely on tribal knowledge
- **Link to team docs** - Reference detailed guides in separate files
- **Provide examples** - Show good/bad code for each convention
- **Update regularly** - Review and update conventions quarterly

### 3. Balance Strictness and Pragmatism

- **Not everything is blocking** - Reserve blocking for critical issues
- **Allow exceptions** - Document when rules don't apply
- **Context matters** - Different rules for different codebases (e.g., prototype vs production)
- **Developer experience** - Don't make reviews frustrating with excessive noise

### 4. Maintain Team Alignment

- **Share changes** - Announce updates in team channel
- **Version control** - Track changes to instructions in git
- **Regular reviews** - Review effectiveness quarterly
- **Ownership** - Assign team member to maintain instructions

### 5. Leverage Existing Checklists

- **Reference, don't duplicate** - Link to detailed checklists in `docs/review/`
- **Extend when needed** - Add team-specific items to checklists
- **Keep DRY** - Update checklist once, all teams benefit
- **Contribute back** - Share useful additions with main repository

### 6. Test Thoroughly

- **Sample PRs** - Create PRs with known issues (security, performance, etc.)
- **Different scenarios** - Test with various PR sizes and types
- **Edge cases** - Test with generated files, large diffs, etc.
- **Team review** - Have team members review sample feedback

### 7. Educate the Team

- **Onboarding** - Include Copilot review system in new hire onboarding
- **Training sessions** - Review sample feedback with team
- **Documentation** - Keep team wiki updated with customization notes
- **Share learnings** - Discuss interesting findings from reviews in team meetings

---

## Maintenance and Updates

### Regular Maintenance Tasks

#### Monthly
- [ ] Review false positives reported by team
- [ ] Update exclusions if new file patterns emerge
- [ ] Check for new security vulnerabilities in dependencies

#### Quarterly
- [ ] Full review of instructions with team
- [ ] Update technology stack rules if tech stack changed
- [ ] Review severity levels and adjust if needed
- [ ] Update examples with recent code patterns
- [ ] Check for updates to base repository (if using shared checklists)

#### When Technology Changes
- [ ] Add new stack rules when adopting new technology
- [ ] Update stack rules when upgrading major versions
- [ ] Remove deprecated technology rules
- [ ] Update examples to reflect current patterns

### Updating from Base Repository

If you're using shared checklists from this repository:

```bash
# Check for updates
git remote add upstream https://github.com/your-org/copilot-reviewer-instructions.git
git fetch upstream

# Review changes to checklists
git diff HEAD..upstream/main docs/review/

# Merge updates if desired
git merge upstream/main
```

### Version Control Best Practices

**Commit message format:**
```
type(scope): description

Examples:
feat(review): add accessibility priority
fix(kotlin): correct null safety examples
docs(guide): update troubleshooting section
chore(template): update metadata section
```

**Track changes:**
- Update "Last Updated" date in metadata section
- Maintain CHANGELOG.md with significant changes
- Tag releases if multiple teams use your template

**Example CHANGELOG.md:**
```markdown
# Changelog

## [1.2.0] - 2025-01-15
### Added
- Accessibility priority to review order
- Team-specific payment processing rules
- Exclusion for GraphQL generated files

### Changed
- Increased test coverage threshold to 80%
- Reordered priorities to emphasize performance

### Fixed
- Corrected Kotlin null safety examples
- Fixed broken links to checklist files

## [1.1.0] - 2024-12-01
...
```

### Feedback Loop

**Collect feedback:**
1. **Anonymous form** - Allow team to report issues or suggestions
2. **Retrospectives** - Discuss Copilot reviews in sprint retros
3. **Metrics** - Track false positive/negative rates
4. **Examples** - Collect good/bad review examples

**Act on feedback:**
1. **Prioritize** - Address blocking issues first
2. **Experiment** - A/B test changes if possible
3. **Communicate** - Share what changed and why
4. **Iterate** - Continuous improvement over perfection

---

## Next Steps

After completing customization:

1. **Create sample PRs** - Test with various scenarios
2. **Team review** - Get feedback from team members
3. **Soft launch** - Enable for subset of repositories first
4. **Monitor** - Watch for false positives/negatives
5. **Iterate** - Refine based on real-world usage
6. **Document** - Add team wiki page with customization notes
7. **Share** - Present to team in demo or standup

---

## Support and Resources

### Documentation
- [Main README](../README.md) - Overview and getting started
- [Security Checklist](../docs/review/security-checklist.md) - Detailed security validation
- [Testing Checklist](../docs/review/testing-checklist.md) - Testing requirements
- [Performance Checklist](../docs/review/performance-checklist.md) - Performance optimization
- [Reliability Checklist](../docs/review/reliability-checklist.md) - Resilience patterns
- [Readability Checklist](../docs/review/readability-checklist.md) - Code clarity standards
- [Code Conventions](../docs/review/code-conventions.md) - Team-specific conventions
- [Stack Rules README](../docs/stack-rules/README.md) - Adding new technology stacks

### Getting Help
- **Team Owner:** Contact `@platform-team` (or your designated owner)
- **Issues:** Report in repository issues or team channel
- **Discussions:** Use team Slack channel or email list

### Contributing
If you develop useful customizations, consider contributing them back:
1. Fork the base repository
2. Add your examples to this guide
3. Submit PR with improvements
4. Share with other teams

---

**Questions or suggestions?** Open an issue or contact the platform team!
