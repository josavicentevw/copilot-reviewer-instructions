# Testing Checklist for Pull Request Reviews

This checklist ensures that code changes include appropriate tests and maintain high quality standards for test coverage and test design.

---

## 1. Unit Test Requirements {#unit-tests}

### Coverage for New Logic
- [ ] Unit tests exist for all new functions, methods, and classes
- [ ] Critical business logic paths are covered
- [ ] Edge cases and boundary conditions are tested
- [ ] Error handling paths are tested

### Test Quality
- [ ] Tests are independent and can run in any order
- [ ] Tests have clear, descriptive names that explain what they verify
- [ ] Each test verifies a single behavior or scenario
- [ ] Tests use the AAA pattern (Arrange, Act, Assert) or Given-When-Then

**Example:**
```typescript
// ✅ GOOD - Clear, focused unit test
describe('calculateDiscount', () => {
  it('should apply 10% discount for premium tier with volume > 100', () => {
    // Arrange
    const tier = 'premium';
    const volume = 150;
    
    // Act
    const discount = calculateDiscount(tier, volume);
    
    // Assert
    expect(discount).toBe(0.10);
  });
  
  it('should return 0 discount for free tier', () => {
    const discount = calculateDiscount('free', 100);
    expect(discount).toBe(0);
  });
});
```

### Assertions
- [ ] Tests use specific assertions (not just checking for no errors)
- [ ] Assertions verify actual behavior, not implementation details
- [ ] Multiple related assertions grouped logically

---

## 2. Contract and API Tests {#contract-tests}

### When API/Schema Changes
- [ ] API contract tests exist when endpoints are added or modified
- [ ] Request/response schema validation tests included
- [ ] Breaking changes identified and migration path provided
- [ ] Backward compatibility verified for public APIs

### Integration Tests
- [ ] Integration tests cover critical service interactions
- [ ] External dependencies properly mocked or use test doubles
- [ ] Database interactions tested with appropriate test database
- [ ] Message queue/event interactions validated

**Example:**
```java
// ✅ GOOD - API contract test
@Test
public void shouldReturnValidUserSchema() {
    Response response = given()
        .when()
        .get("/api/users/123")
        .then()
        .statusCode(200)
        .extract().response();
    
    // Verify response matches schema
    response.then().assertThat()
        .body(matchesJsonSchemaInClasspath("user-schema.json"));
}
```

---

## 3. Test Data Quality {#test-data}

### Realistic Test Data
- [ ] Test data represents realistic scenarios
- [ ] Edge cases covered (empty strings, null values, max/min boundaries)
- [ ] Different data types and formats tested
- [ ] Internationalization scenarios considered (UTF-8, special characters)

### No Real PII
- [ ] No production data or real PII in tests
- [ ] Synthetic or anonymized data used
- [ ] Sensitive data properly mocked
- [ ] Test data generation uses factories or builders

**Example:**
```python
# ❌ BAD - Real PII in tests
def test_user_creation():
    user = create_user("john.doe@acme.com", "123-45-6789")
    assert user.email == "john.doe@acme.com"

# ✅ GOOD - Synthetic test data
def test_user_creation():
    user = create_user("test+user@example.com", "000-00-0000")
    assert user.email == "test+user@example.com"
```

---

## 4. Coverage and Rationale {#coverage}

### Acceptable Coverage
- [ ] Critical paths have 100% coverage
- [ ] Overall test coverage meets project standards (typically 80%+)
- [ ] Coverage reports reviewed for gaps
- [ ] Uncovered code has documented rationale

### When Tests Are Not Applicable
- [ ] Rationale provided for missing tests (e.g., generated code, simple getters)
- [ ] Alternative verification methods documented (manual testing, type safety)
- [ ] Technical debt items created if tests should be added later

**Acceptable reasons for no tests:**
- Pure generated code (Swagger/OpenAPI client code)
- Simple data classes with no logic (DTOs, POJOs)
- Framework boilerplate that's already tested
- Code that will be removed soon (with tech debt ticket)

---

## 5. Test Organization and Maintainability

### File Structure
- [ ] Test files located alongside source files or in parallel test directory
- [ ] Test file naming convention followed (e.g., `MyClass.test.ts`, `test_my_module.py`)
- [ ] Tests organized by feature or component

### Test Utilities
- [ ] Shared test utilities and helpers properly organized
- [ ] Test fixtures and factories reusable
- [ ] Mock/stub configurations centralized where appropriate

**Example structure:**
```
src/
  services/
    PaymentService.ts
    PaymentService.test.ts
  utils/
    validation.ts
    validation.test.ts
  __tests__/
    helpers/
      testFactory.ts
      mockData.ts
```

---

## 6. Test Performance

### Fast Execution
- [ ] Unit tests run in milliseconds (typically < 100ms each)
- [ ] No unnecessary sleeps or waits
- [ ] Database operations use in-memory databases or mocks for unit tests
- [ ] External API calls mocked in unit tests

### Resource Management
- [ ] Tests properly clean up resources (files, connections, etc.)
- [ ] No test data pollution (each test isolated)
- [ ] Parallel execution safe (no shared mutable state)

---

## 7. Mocking and Test Doubles

### Appropriate Use
- [ ] External dependencies mocked appropriately
- [ ] Mocks/stubs don't test implementation details
- [ ] Use real objects for simple dependencies
- [ ] Verify mock interactions when testing side effects

**Example:**
```javascript
// ✅ GOOD - Mock external service
describe('UserService', () => {
  it('should send welcome email when user is created', async () => {
    const emailService = {
      send: jest.fn().mockResolvedValue({ success: true })
    };
    
    const userService = new UserService(emailService);
    await userService.createUser({ email: 'test@example.com' });
    
    expect(emailService.send).toHaveBeenCalledWith({
      to: 'test@example.com',
      template: 'welcome'
    });
  });
});
```

---

## 8. Specialized Test Types

### Snapshot Tests
- [ ] Snapshots used appropriately (UI components, data structures)
- [ ] Snapshots reviewed on changes (not blindly updated)
- [ ] Large snapshots avoided (consider splitting or using serializers)

### End-to-End Tests
- [ ] E2E tests focus on critical user journeys
- [ ] E2E tests are stable and deterministic
- [ ] Appropriate wait strategies used (no arbitrary sleeps)

### Property-Based Tests
- [ ] Property-based tests for complex algorithms (when applicable)
- [ ] Input generators properly defined

---

## Review Checklist Summary

Use this quick checklist during PR reviews:

- [ ] **Unit Tests**: New logic covered, edge cases tested, quality assertions
- [ ] **Contract/API Tests**: Schema validated, breaking changes identified
- [ ] **Test Data**: Realistic scenarios, no real PII, proper mocking
- [ ] **Coverage**: Critical paths covered, rationale for gaps documented
- [ ] **Organization**: Proper file structure, reusable utilities
- [ ] **Performance**: Fast execution, proper resource cleanup
- [ ] **Mocking**: External dependencies mocked, not testing implementation details
- [ ] **Specialized**: Snapshots reviewed, E2E tests stable

---

## Testing Commands

Common commands for running tests:

**Node.js / TypeScript:**
```bash
npm test                    # Run all tests
npm test -- --coverage      # With coverage report
npx jest path/to/test.ts   # Run specific test file
```

**Python:**
```bash
pytest                      # Run all tests
pytest --cov=src           # With coverage
pytest tests/test_file.py  # Run specific test
```

**Java / Kotlin:**
```bash
./gradlew test             # Gradle
mvn test                   # Maven
```

---

## References

- [Test-Driven Development (TDD)](https://martinfowler.com/bliki/TestDrivenDevelopment.html)
- [Unit Testing Best Practices](https://docs.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)
- [Google Testing Blog](https://testing.googleblog.com/)
