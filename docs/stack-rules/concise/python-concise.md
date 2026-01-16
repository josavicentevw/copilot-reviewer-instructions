# Python Review Cheat Sheet (Concise)

Reference: [`docs/stack-rules/python-rules.md`](../python-rules.md) · Examples: [`Python Examples`](../examples-only/python-examples.md)

## Core Checks

- [ ] **Type Safety**: Type hints everywhere, `Optional` for nullable values, `Any` justified, mypy/pyright in CI.
- [ ] **Data Modeling**: Dataclasses/Pydantic models with validation, frozen dataclasses when immutable.
- [ ] **Error Handling**: Specific exceptions, no bare `except:`, context managers for resources, logging with context.
- [ ] **Database/ORM**: Parameterized queries, indexes, session/transaction management, avoid string SQL concatenation.
- [ ] **Async**: Use `async/await`, `asyncio.gather`, proper timeouts, async context managers, cancellation handling.
- [ ] **Dependency Injection**: Constructor injection, protocols/abstract base classes, FastAPI `Depends`.
- [ ] **Configuration**: Pydantic settings/environment variables, no hard-coded secrets.
- [ ] **Testing**: Pytest fixtures, parametrization, mocking external services, async tests.
- [ ] **Style**: PEP8, Black/isort, linting (ruff/pylint), consistent imports.
- [ ] **Accessibility/i18n**: Templates use semantic HTML, translation tags, accessible CLI outputs.
- [ ] **Security**: Secrets externalized, SQL injection guards, input validation, escaping user content, dependency auditing.
- [ ] **Tooling**: Coverage thresholds, bandit, pip-audit/safety, Hypothesis for property tests.

## Quick Reminders

- Reference sections `#accessibility`, `#testing-tooling`, `#security`.
- Keep `.env`/settings documented; verify `Settings` models fail fast on missing config.
