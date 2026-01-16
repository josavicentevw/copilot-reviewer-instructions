# Stack-Specific Rules

This directory contains technology stack-specific validation rules that supplement the general code review checklists.

## Available Stack Rules

### Backend Languages

- **[Java Rules](./java-rules.md)**: Null safety with Optional, streams and functional programming, JPA/Hibernate best practices, Spring dependency injection, logging, and immutability patterns for Java projects.
- **[Kotlin Rules](./java-kotlin-rules.md)**: Null safety, data classes, sealed traits, repository patterns, dependency injection, logging, and exception handling for Kotlin projects.
- **[Python Rules](./python-rules.md)**: Type hints, dataclasses and Pydantic models, error handling, SQLAlchemy/ORM patterns, async/await, dependency injection, and testing with pytest.
- **[Scala Rules](./scala-rules.md)**: Option types, case classes and sealed traits, for-comprehensions, Either/Try error handling, collections, implicits, and Future patterns.
- **[Go Rules](./go-rules.md)**: Error handling, context usage, goroutines and concurrency, interfaces, database/SQL patterns, struct design, and testing best practices.

### Frontend Frameworks

- **[React/TypeScript Rules](./react-typescript-rules.md)**: TypeScript strict mode, React hooks, component patterns, state management, HTTP client requirements, and performance optimization for React/TypeScript projects. ([Cheat Sheet](./concise/react-typescript-concise.md) · [Examples](./examples-only/react-typescript-examples.md))
- **[Angular Rules](./angular-rules.md)**: TypeScript strict mode, component architecture, RxJS observables, dependency injection, forms and validation, state management with NgRx, and performance optimization for Angular projects. ([Cheat Sheet](./concise/angular-concise.md) · [Examples](./examples-only/angular-examples.md))

### Quick Reference & Examples

- **Backend Cheat Sheets**: [Go](./concise/go-concise.md), [Java](./concise/java-concise.md), [Kotlin/Java](./concise/java-kotlin-concise.md), [Python](./concise/python-concise.md), [Scala](./concise/scala-concise.md)
- **Backend Examples**: [Go](./examples-only/go-examples.md), [Java](./examples-only/java-examples.md), [Kotlin/Java](./examples-only/java-kotlin-examples.md), [Python](./examples-only/python-examples.md), [Scala](./examples-only/scala-examples.md)

## How to Use Stack-Specific Rules

When reviewing code:

1. **Apply general checklists first**: Start with the general review checklists in `docs/review/`
2. **Apply stack-specific rules**: Then apply the relevant stack-specific rules from this directory
3. **Reference both in findings**: When reporting issues, reference both the general checklist and stack-specific rule

**Example:**
```markdown
**Important | TypeScript**: Usage of `any` type without justification
**Evidence:** `src/utils/helper.ts:42`
**Proposed fix:** Replace `any` with proper interface `ProcessableData`
**Reference:** [`docs/stack-rules/react-typescript-rules.md#typescript-strict`]
```

---

## Adding New Stack-Specific Rules

To add rules for a new technology stack (Python, Swift, Go, etc.):

### 1. Create New Stack Rules File

Create a new markdown file in this directory following the naming convention:

```bash
docs/stack-rules/{language}-{framework}-rules.md
```

Examples:
- `python-django-rules.md`
- `swift-ios-rules.md`
- `go-rules.md`
- `csharp-dotnet-rules.md`
- **Concise references** (summaries): `concise/react-typescript-concise.md`, etc.
- **Example-only guides**: `examples-only/react-typescript-examples.md`, etc.

### 2. Follow the Standard Structure

Use this template structure for consistency:

```markdown
# {Language/Framework} Stack-Specific Rules

Brief introduction explaining scope and purpose.

---

## 1. {Topic Category} {#anchor-id}

### {Sub-topic}
- [ ] Checklist item 1
- [ ] Checklist item 2

**Examples:**
\```{language}
// ❌ BAD - Description
code example

// ✅ GOOD - Description
code example
\```

---

## Review Checklist Summary

Quick checklist for {Stack} code reviews:

- [ ] **{Category}**: Brief summary
- [ ] **{Category}**: Brief summary

---

## References

- [Link to official documentation]
- [Link to best practices guide]
```

### 3. Common Categories to Include

Consider including these categories (adapt to your stack):

- **Type Safety / Null Handling**
- **Framework-Specific Patterns** (ORM, dependency injection, etc.)
- **Error Handling and Logging**
- **Testing Patterns**
- **Performance Considerations**
- **Security Best Practices**
- **Code Organization**

### 4. Update Main Instructions

After creating the new stack rules file:

1. Open `.github/copilot-instructions.md`
2. Locate the `## 6. Stack-Specific Rules` section
3. Add your new stack following the existing pattern:

```markdown
### {Language/Framework}
Reference: [`docs/stack-rules/{filename}.md`](../docs/stack-rules/{filename}.md)

- **{Key topic}**: Brief description
- **{Key topic}**: Brief description
- **{Key topic}**: Brief description
```

### 5. Update This README

Add your new stack to the "Available Stack Rules" section at the top of this file.

---

## Guidelines for Writing Stack Rules

### Do's ✅
- **Be specific**: Provide concrete, actionable rules
- **Include examples**: Show both bad and good code examples
- **Use checkboxes**: Make rules easy to verify during reviews
- **Add references**: Link to official documentation and best practices
- **Focus on common issues**: Prioritize frequently occurring problems
- **Keep it practical**: Rules should be enforceable in code reviews

### Don'ts ❌
- **Don't duplicate general rules**: Only include stack-specific guidance
- **Don't be too prescriptive**: Allow for reasonable variations
- **Don't include style preferences**: Focus on correctness and best practices
- **Don't create huge files**: Split into multiple files if needed
- **Don't forget examples**: Every rule should have code examples

---

## Example: Adding Python/Django Rules

Let's walk through adding rules for Python with Django:

**Step 1: Create file**
```bash
touch docs/stack-rules/python-django-rules.md
```

**Step 2: Add content**
```markdown
# Python/Django Stack-Specific Rules

## 1. Type Hints and MyPy {#type-hints}

### Type Hint Requirements
- [ ] All function signatures have type hints
- [ ] Use `Optional[T]` for potentially None values
- [ ] MyPy enabled in CI/CD pipeline

**Examples:**
\```python
# ❌ BAD - No type hints
def process_user(user_id):
    return get_user(user_id)

# ✅ GOOD - Proper type hints
def process_user(user_id: str) -> User:
    return get_user(user_id)
\```

## 2. Django ORM Best Practices {#django-orm}

### Query Optimization
- [ ] Use `select_related()` for foreign keys
- [ ] Use `prefetch_related()` for many-to-many
- [ ] Avoid N+1 queries

**Examples:**
\```python
# ❌ BAD - N+1 query
users = User.objects.all()
for user in users:
    print(user.profile.bio)  # Causes N queries

# ✅ GOOD - Optimized query
users = User.objects.select_related('profile')
for user in users:
    print(user.profile.bio)  # Single query
\```
```

**Step 3: Update `.github/copilot-instructions.md`**

Add to the Stack-Specific Rules section:

```markdown
### Python/Django
Reference: [`docs/stack-rules/python-django-rules.md`](../docs/stack-rules/python-django-rules.md)

- **Type hints**: All functions typed, MyPy validation
- **Django ORM**: select_related/prefetch_related, avoid N+1
- **Settings**: Never commit secrets, use environment variables
```

**Step 4: Update this README**

Add to "Available Stack Rules":
```markdown
- **[Python/Django Rules](./python-django-rules.md)**: Type hints, Django ORM optimization, settings management, and testing patterns.
```

---

## Maintenance

### Reviewing and Updating Rules

- **Quarterly review**: Review rules every 3 months for relevance
- **Framework updates**: Update when major framework versions release
- **Team feedback**: Incorporate learnings from actual code reviews
- **Metrics-driven**: Add rules based on frequently caught issues

### Proposing Changes

To propose changes to existing stack rules:

1. Open a PR with proposed changes
2. Tag `VW D:H` for review
3. Update "Last Updated" date in the file

---

## Questions?

For questions or suggestions about stack-specific rules:
- Open an issue in this repository
- Contact VW D:H
- Propose a PR with your suggested changes
