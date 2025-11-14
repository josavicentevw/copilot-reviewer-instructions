---
name: New Technology Stack
about: Propose adding support for a new programming language or framework
title: '[STACK] Add {Language/Framework} support'
labels: enhancement, new-stack
assignees: ''
---

## Technology Stack

**Language/Framework**: (e.g., Python, Go, Swift, Ruby, Rust)
**Version(s)**: (e.g., Python 3.11+, Go 1.21+)
**Ecosystem**: (e.g., Django/FastAPI for Python, Echo/Gin for Go)

## Justification

**Why should this stack be added?**

- How many teams use this stack?
- Is it widely adopted in the industry?
- Are there unique patterns that need validation?

## Key Patterns to Cover

List the most important patterns/rules for this stack:

### 1. [Pattern Category - e.g., Null Safety]

**Rule**: (Description of what to check)

**Example**:
```language
// ❌ BAD - Why this is wrong
badExample();

// ✅ GOOD - Why this is correct
goodExample();
```

### 2. [Another Pattern Category]

**Rule**: (Description)

**Example**:
```language
// ❌ BAD
// ...

// ✅ GOOD
// ...
```

### 3. [Third Pattern Category]

(Continue for all key patterns)

## Common Pitfalls

What mistakes do developers commonly make in this stack?

1. **Pitfall 1**: (e.g., Mutable default arguments in Python)
   - Why it's a problem
   - How to detect it
   - Correct alternative

2. **Pitfall 2**: (e.g., Race conditions in Go goroutines)
   - Why it's a problem
   - How to detect it
   - Correct alternative

## References

Please provide links to:

- [ ] Official style guide: (link)
- [ ] Official documentation: (link)
- [ ] Community best practices: (link)
- [ ] Common linter rules: (link)
- [ ] Security guidelines: (link)

## Proposed Structure

Outline the sections for `docs/stack-rules/{stack-name}-rules.md`:

1. [Section 1 - e.g., Type Safety]
2. [Section 2 - e.g., Error Handling]
3. [Section 3 - e.g., Concurrency Patterns]
4. [etc.]

## Contribution

- [ ] I'm willing to write the initial draft
- [ ] I can provide examples and review
- [ ] I'd like to help test with real PRs
- [ ] I'm just proposing, but can't contribute the code

## Additional Context

Any other information about this stack that would be helpful.

---

**Before submitting**, please:
- [ ] Check if this stack is already supported
- [ ] Review existing stack rules (Kotlin, React/TypeScript) for structure
- [ ] Read the [extensibility guide](../docs/stack-rules/README.md)
- [ ] Prepare comprehensive examples
