# Contributing to GitHub Copilot Review System

Thank you for your interest in contributing to the GitHub Copilot Pull Request Review System! This document provides guidelines and best practices for contributing to this project.

---

## Table of Contents

1. [Code of Conduct](#code-of-conduct)
2. [How Can I Contribute?](#how-can-i-contribute)
3. [Getting Started](#getting-started)
4. [Contribution Guidelines](#contribution-guidelines)
5. [Documentation Standards](#documentation-standards)
6. [Review Process](#review-process)
7. [Style Guide](#style-guide)
8. [Community](#community)

---

## Code of Conduct

### Our Pledge

We are committed to providing a welcoming and inclusive experience for everyone. We expect all contributors to:

- Use welcoming and inclusive language
- Be respectful of differing viewpoints and experiences
- Gracefully accept constructive criticism
- Focus on what is best for the community
- Show empathy towards other community members

### Our Standards

**Unacceptable behavior includes:**
- Harassment, intimidation, or discrimination of any kind
- Trolling, insulting/derogatory comments, and personal attacks
- Publishing others' private information without permission
- Other conduct which could reasonably be considered inappropriate

---

## How Can I Contribute?

### 1. Reporting Issues

Found a bug, false positive pattern, or missing rule? Please open an issue!

**Before submitting an issue:**
- Check existing issues to avoid duplicates
- Verify you're using the latest version
- Gather relevant context (examples, PRs, logs)

**Issue template:**
```markdown
**Type**: Bug / Enhancement / Documentation

**Description**:
Clear description of the issue or suggestion

**Current Behavior**:
What currently happens

**Expected Behavior**:
What should happen

**Examples**:
- PR #123 - Example of the issue
- Code snippet or screenshot

**Proposed Solution** (optional):
Your ideas for fixing/improving
```

### 2. Suggesting Enhancements

We welcome suggestions for:
- New technology stack support (Swift, Ruby, Rust, C#/.NET, PHP, mobile frameworks).
- Additional checklists or rules
- Better customization options
- Documentation improvements

**Enhancement template:**
```markdown
**Enhancement**: Brief title

**Problem**:
What problem does this solve?

**Proposed Solution**:
Detailed description of your proposal

**Alternatives Considered**:
Other approaches you've thought about

**Impact**:
How many teams/users would benefit?
```

### 3. Adding New Technology Stack Rules

Want to add support for a new programming language or framework?

**Steps:**
1. Check `docs/stack-rules/README.md` for the extensibility guide
2. Create a new file: `docs/stack-rules/{stack-name}-rules.md`
3. Follow the template structure from existing stack rules
4. Include comprehensive examples (❌ BAD vs ✅ GOOD)
5. Cover key areas: null safety, patterns, common pitfalls
6. Update `docs/stack-rules/README.md` with the new stack
7. Submit a pull request

**Example stacks we'd love to see:**
- Swift/SwiftUI (optionals, protocols, memory management)
- Ruby (Rails conventions, metaprogramming)
- Rust (ownership, lifetimes, error handling)

### 4. Improving Checklists

Found a missing security check or performance pattern?

**Steps:**
1. Identify the relevant checklist in `docs/review/`
2. Add the new rule with:
   - Clear checkbox item
   - Explanation of why it matters
   - Code examples (both bad and good)
   - References to authoritative sources (OWASP, official docs)
3. Ensure examples use Kotlin or React/TypeScript for consistency
4. Update the checklist's table of contents if needed
5. Submit a pull request

### 5. Writing Documentation

Good documentation is critical to this project!

**Documentation needs:**
- Clearer explanations of existing rules
- More real-world examples
- Troubleshooting guides
- Video tutorials or workshops
- Translations (while keeping source in English)

### 6. Sharing Feedback and Use Cases

Even if you don't submit code, sharing your experience helps!

**Ways to contribute feedback:**
- Share how your team uses the system
- Report false positive patterns you encounter
- Suggest improvements based on real usage
- Provide metrics and ROI data from your adoption

---

## Getting Started

### Prerequisites

- Git installed locally
- Markdown editor (VS Code, etc.)
- Basic understanding of Git workflow
- Familiarity with code review concepts

### Fork and Clone

1. **Fork the repository** on GitHub
2. **Clone your fork** locally:
   ```bash
   git clone https://github.com/YOUR-USERNAME/copilot-reviewer-instructions.git
   cd copilot-reviewer-instructions
   ```
3. **Add upstream remote** (replace with the actual repository URL):
   ```bash
   git remote add upstream https://github.com/OWNER/copilot-reviewer-instructions.git
   ```

### Create a Branch

Always create a new branch for your work:

```bash
# Update your local main branch
git checkout main
git pull upstream main

# Create a feature branch
git checkout -b feature/add-python-rules
# or
git checkout -b fix/security-checklist-typo
# or
git checkout -b docs/improve-getting-started
```

**Branch naming conventions:**
- `feature/{description}` - New features or enhancements
- `fix/{description}` - Bug fixes
- `docs/{description}` - Documentation improvements
- `refactor/{description}` - Code or structure refactoring

---

## Contribution Guidelines

### General Guidelines

1. **All documentation must be in English**
   - Main documentation files: English
   - Code examples and comments: English
   - Variable/function names: English
   - Templates can specify response language (e.g., Spanish) but template itself is English

2. **Follow existing structure and style**
   - Use the same heading hierarchy as existing files
   - Match the tone and format of similar documents
   - Include table of contents for long documents (>200 lines)

3. **Provide comprehensive examples**
   - Always include both ❌ BAD and ✅ GOOD examples
   - Use real-world scenarios when possible
   - Comment complex examples to explain the issue

4. **Reference authoritative sources**
   - Link to OWASP, official documentation, style guides
   - Cite industry best practices
   - Include references section at the end

5. **Test your changes**
   - Validate all Markdown links work
   - Check formatting renders correctly
   - Ensure code examples are syntactically correct
   - Test with a real PR if adding new rules

### Pull Request Process

1. **Commit your changes** with clear, conventional commit messages:
   ```bash
   git add .
   git commit -m "feat: add Python stack-specific rules
   
   - Add null safety and type hints requirements
   - Include common pitfalls (mutable defaults, exception handling)
   - Add examples for Django and FastAPI patterns
   - Update stack-rules README with Python entry"
   ```

2. **Push to your fork**:
   ```bash
   git push origin feature/add-python-rules
   ```

3. **Open a Pull Request** on GitHub:
   - Use a clear, descriptive title
   - Fill out the PR template completely
   - Link related issues (e.g., "Fixes #123")
   - Add screenshots if relevant
   - Request review from maintainers

4. **Respond to feedback**:
   - Address review comments promptly
   - Ask questions if anything is unclear
   - Make requested changes in new commits
   - Push updates to the same branch

5. **Merge requirements**:
   - All review comments addressed
   - CI checks passing (if applicable)
   - At least one maintainer approval
   - Up-to-date with main branch

### Commit Message Convention

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat:` - New feature or enhancement
- `fix:` - Bug fix
- `docs:` - Documentation changes
- `style:` - Formatting, missing semicolons, etc. (no code change)
- `refactor:` - Code refactoring (no functional change)
- `test:` - Adding or updating tests
- `chore:` - Maintenance tasks, dependency updates

**Examples:**
```bash
feat: add Python stack-specific rules

docs: improve security checklist with XSS examples

fix: correct broken link in README

refactor: reorganize metrics documentation structure
```

---

## Documentation Standards

### Markdown Best Practices

1. **Use consistent heading levels**:
   ```markdown
   # H1 - Document Title (only one per file)
   ## H2 - Major Sections
   ### H3 - Subsections
   #### H4 - Detailed Items
   ```

2. **Include table of contents** for files >200 lines:
   ```markdown
   ## Table of Contents
   
   1. [Section One](#section-one)
   2. [Section Two](#section-two)
   ```

3. **Use code blocks with language specification**:
   ````markdown
   ```typescript
   const example = "Use language-specific highlighting";
   ```
   ````

4. **Add horizontal rules for visual separation**:
   ```markdown
   ---
   ```

5. **Use proper link formatting**:
   ```markdown
   [Link Text](relative/path/to/file.md)
   [External Link](https://example.com)
   ```

### Code Example Standards

**Always use this format:**

```markdown
**Examples:**

```typescript
// ❌ BAD - Explanation of why this is wrong
const badExample = any;
function doSomethingBad() {
  // Problematic code
}

// ✅ GOOD - Explanation of why this is correct
const goodExample: string;
function doSomethingGood(): void {
  // Proper implementation
}
```
```

**For stack-specific rules**, use the appropriate language:
- Kotlin examples for `java-kotlin-rules.md`
- TypeScript/React for `react-typescript-rules.md`
- Python for `python-rules.md` (when created)

### Checklist Standards

**Format:**
```markdown
### Category Name {#anchor-id}

**Rule description**:
- [ ] Checkbox item with clear requirement
- [ ] Another requirement
- [ ] Third requirement

**Guidelines:**
- Explain when this applies
- Provide context

**Examples:**
[Code examples here]
```

---

## Review Process

### What We Look For

Reviewers will check for:

1. **Correctness**
   - Are the rules accurate and up-to-date?
   - Do examples follow current best practices?
   - Are there any misleading or incorrect statements?

2. **Completeness**
   - Does it cover all important aspects?
   - Are examples comprehensive?
   - Is the documentation self-contained?

3. **Consistency**
   - Does it match existing style and tone?
   - Are conventions followed?
   - Is formatting consistent with other files?

4. **Clarity**
   - Is it easy to understand?
   - Are examples clear and well-explained?
   - Is the language accessible to the target audience?

5. **Value**
   - Does this improve the project?
   - Will it help teams write better code?
   - Is it actionable and practical?

### Review Timeline

- **Initial response**: Within 3 business days
- **Full review**: Within 1 week for small PRs, 2 weeks for major additions
- **Follow-up**: Reviewers aim to respond to updates within 2 business days

### Becoming a Reviewer

Interested in helping review contributions?

**Criteria:**
- Active contributor with 3+ accepted PRs
- Deep understanding of code review best practices
- Familiarity with at least one technology stack (Kotlin, React/TypeScript, etc.)
- Commitment to providing constructive, helpful feedback

**Contact** VW:HUB or comment on an issue expressing interest.

---

## Style Guide

### Language and Tone

- **Professional but approachable** - Not too formal, not too casual
- **Clear and concise** - Avoid jargon when possible
- **Actionable** - Focus on "what to do" not just "what's wrong"
- **Inclusive** - Use "we", "you" instead of "he/she"
- **Educational** - Explain the "why" behind rules

**Examples:**

❌ **Too casual**: "Don't be dumb and use `any` everywhere"
✅ **Good**: "Avoid using `any` type unless absolutely necessary. Use specific types or `unknown` for better type safety."

❌ **Too vague**: "Performance might be bad"
✅ **Good**: "This N+1 query pattern will cause performance degradation as the dataset grows. Use a JOIN or eager loading instead."

### Terminology

Use consistent terminology across all documentation:

- **Pull Request (PR)** - Not "merge request"
- **Findings** - Not "issues" or "problems" (to avoid confusion with GitHub issues)
- **Severity levels**: Blocking, Important, Suggestion
- **Categories**: Security, Testing, Performance, Reliability, Readability, Code Conventions
- **Tech stacks**: Kotlin, React/TypeScript, Python, etc. (capitalized)

### File Organization

```
copilot-reviewer-instructions/
├── .github/
│   └── copilot-instructions.md
├── docs/
│   ├── review/           # Specialized checklists
│   └── stack-rules/      # Technology-specific rules
├── templates/            # Customizable templates
├── CONTRIBUTING.md       # This file
├── README.md            # Main documentation
└── LICENSE              # License file
```

**Naming conventions:**
- Use lowercase with hyphens: `security-checklist.md`
- Be descriptive: `react-typescript-rules.md` not `react-rules.md`
- Group related files in directories

---

## Community

### Communication Channels

- **GitHub Issues** - Bug reports, feature requests, questions
- **Pull Requests** - Code and documentation contributions
- **Discussions** - General questions, ideas, and community support
- **Slack** - Real-time chat and support (if applicable)

### Getting Help

**Stuck on something?**
1. Check existing documentation and issues
2. Search for similar questions
3. Ask in GitHub Discussions or open an issue
4. Tag relevant maintainers if urgent

**Want to help others?**
- Answer questions in issues
- Review open pull requests
- Share your experience and use cases
- Improve documentation based on common questions

### Recognition

We appreciate all contributions! Contributors will be:
- Listed in release notes for significant contributions
- Mentioned in the README (if desired)
- Invited to join the reviewer team after consistent contributions

---

## Development Workflow

### Making Changes

1. **Update your fork**:
   ```bash
   git checkout main
   git pull upstream main
   git push origin main
   ```

2. **Create feature branch**:
   ```bash
   git checkout -b feature/your-feature
   ```

3. **Make changes and commit**:
   ```bash
   git add .
   git commit -m "feat: your feature description"
   ```

4. **Keep your branch updated**:
   ```bash
   git fetch upstream
   git rebase upstream/main
   ```

5. **Push and create PR**:
   ```bash
   git push origin feature/your-feature
   ```

### Testing Your Changes

**Before submitting a PR:**

1. **Validate Markdown**:
   - Check all links work
   - Verify formatting renders correctly
   - Use a Markdown linter if available

2. **Test examples**:
   - Ensure code examples are syntactically correct
   - Verify examples compile/run (when possible)
   - Check that ❌ BAD examples actually demonstrate the issue

3. **Proofread**:
   - Check spelling and grammar
   - Ensure consistency with existing docs
   - Verify all required sections are included

4. **Self-review**:
   - Read your changes as if you're seeing them for the first time
   - Would you understand this without context?
   - Is anything unclear or missing?

---

## Questions?

If you have any questions about contributing, please:

1. Check this guide thoroughly
2. Search existing issues and discussions
3. Open a new issue with your question
4. Tag it with `question` label

---

## Thank You!

Your contributions make this project better for everyone. Whether you're fixing a typo, adding a new technology stack, or improving documentation, every contribution is valued.

**Happy contributing! 🚀**

---

**Maintainers:**
- VW D:H

**Last Updated:** 2025-11-14
