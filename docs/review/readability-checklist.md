# Code Readability and Developer Experience Checklist

This checklist ensures code is clear, maintainable, and provides a good developer experience. Focus on clean code principles and team productivity.

---

## 1. Avoiding Nested Ternaries {#avoid-nested-ternaries}

### Clear Conditional Logic
- [ ] No nested ternary operators
- [ ] Complex conditions extracted to named functions
- [ ] Early returns used to reduce nesting
- [ ] Switch statements or if-else chains for multiple conditions

**Example:**
```typescript
// ❌ BAD - Nested ternary
const price = tier === 'premium' 
  ? (volume > 100 ? 0.8 : 0.9) 
  : tier === 'standard' 
    ? (volume > 50 ? 0.95 : 1.0)
    : 1.0;

// ✅ GOOD - Extracted to function with clear logic
function calculatePriceMultiplier(tier: string, volume: number): number {
  if (tier === 'premium') {
    return volume > 100 ? 0.8 : 0.9;
  }
  
  if (tier === 'standard') {
    return volume > 50 ? 0.95 : 1.0;
  }
  
  return 1.0;
}

const price = calculatePriceMultiplier(tier, volume);
```

---

## 2. Cyclomatic Complexity {#complexity}

### Function Complexity
- [ ] Functions have reasonable cyclomatic complexity (typically < 10)
- [ ] Long functions broken into smaller, focused functions
- [ ] Each function has single responsibility
- [ ] Complex business logic well-structured and commented

**Complexity Guidelines:**
- 1-5: Simple, easy to test
- 6-10: Moderate, still manageable
- 11-20: Complex, consider refactoring
- 20+: Very complex, should be refactored

**Example:**
```python
# ❌ BAD - High complexity (too many branches)
def process_order(order):
    if order.status == 'pending':
        if order.payment_method == 'credit_card':
            if order.amount > 1000:
                if order.user.is_verified:
                    # ... complex logic
                else:
                    # ... more logic
            else:
                # ... more logic
        elif order.payment_method == 'paypal':
            # ... even more logic
    elif order.status == 'processing':
        # ... continues...

# ✅ GOOD - Reduced complexity with early returns and extracted functions
def process_order(order):
    if order.status != 'pending':
        return handle_non_pending_order(order)
    
    validate_order(order)
    
    if order.payment_method == 'credit_card':
        return process_credit_card_payment(order)
    elif order.payment_method == 'paypal':
        return process_paypal_payment(order)
    
    raise UnsupportedPaymentMethodError(order.payment_method)
```

---

## 3. Clear Naming Conventions {#naming}

### Expressive Names
- [ ] Variable and function names clearly express intent
- [ ] Avoid abbreviations unless universally understood
- [ ] Boolean variables named as questions (is, has, can, should)
- [ ] Functions named as verbs or verb phrases

**Naming Guidelines:**
- **Variables**: Nouns or noun phrases (`user`, `orderTotal`, `activeCustomers`)
- **Functions**: Verbs (`calculateTotal`, `fetchUser`, `validateInput`)
- **Booleans**: Questions (`isValid`, `hasPermission`, `canEdit`)
- **Constants**: UPPER_SNAKE_CASE (`MAX_RETRIES`, `API_TIMEOUT`)

**Example:**
```javascript
// ❌ BAD - Unclear names
const d = new Date();
function proc(data) { /* ... */ }
const flg = true;
const x = getUserData();

// ✅ GOOD - Clear, expressive names
const currentDate = new Date();
function processPayment(paymentData) { /* ... */ }
const isAuthenticated = true;
const userData = getUserData();
```

### Consistent Naming
- [ ] Naming conventions consistent across codebase
- [ ] Same concept named consistently (not user/customer/account for same thing)
- [ ] Project/team naming conventions followed

---

## 4. Purposeful Comments {#comments}

### When to Comment
- [ ] Comments explain "why", not "what" (code explains "what")
- [ ] Complex algorithms have explanatory comments
- [ ] Non-obvious business rules documented
- [ ] TODOs have context and owner/ticket reference

**Example:**
```java
// ❌ BAD - Obvious comment (doesn't add value)
// Increment counter by 1
counter++;

// ❌ BAD - Outdated or wrong comment
// Check if user is admin (code actually checks if user is verified)
if (user.isVerified()) { /* ... */ }

// ✅ GOOD - Explains "why" and business context
// Apply 10% discount for loyalty program members as per marketing campaign Q4-2024
if (user.isLoyaltyMember()) {
    discount = 0.10;
}

// ✅ GOOD - Explains complex algorithm
// Use Boyer-Moore algorithm for efficient string matching in large texts
// Average case: O(n/m), Best case: O(n/m), Worst case: O(nm)
const index = boyerMooreSearch(text, pattern);

// ✅ GOOD - TODO with context
// TODO(jdoe): Refactor to use new PaymentGatewayV2 after migration
// See ticket: PROJ-1234
const result = await legacyPaymentGateway.process(payment);
```

### Documentation Comments
- [ ] Public APIs have JSDoc/JavaDoc/docstring documentation
- [ ] Parameters and return values documented
- [ ] Exceptions/errors documented
- [ ] Usage examples provided for complex APIs

---

## 5. Type Visibility and Safety {#types}

### TypeScript
- [ ] Types explicitly defined for public APIs
- [ ] Avoid `any` unless absolutely necessary (with justification comment)
- [ ] Use union types and type guards appropriately
- [ ] Interfaces/types defined for complex objects

**Example:**
```typescript
// ❌ BAD - Using 'any'
function processData(data: any): any {
  return data.value * 2;
}

// ✅ GOOD - Explicit types
interface ProcessableData {
  value: number;
  metadata?: Record<string, string>;
}

function processData(data: ProcessableData): number {
  return data.value * 2;
}
```

### Kotlin/Java
- [ ] Nullable types handled explicitly (Kotlin)
- [ ] Optional<T> used appropriately (Java)
- [ ] Avoid `!!` operator in Kotlin
- [ ] Generics used for type safety

### Python
- [ ] Type hints used for function signatures
- [ ] Complex data structures use dataclasses or Pydantic models
- [ ] Type checking enabled (mypy, pyright)

---

## 6. Code Organization

### File Structure
- [ ] Related code grouped together
- [ ] Files have single responsibility
- [ ] File length reasonable (typically < 500 lines)
- [ ] Imports organized and unused imports removed

### Function Organization
- [ ] Public functions before private functions
- [ ] Helper functions near their usage
- [ ] Logical grouping of related functions

**Example structure:**
```typescript
// ✅ GOOD - Well-organized file
// Imports
import { User } from './types';
import { validateEmail } from './validators';

// Constants
const MAX_LOGIN_ATTEMPTS = 3;

// Types
interface LoginCredentials {
  email: string;
  password: string;
}

// Main public functions
export async function login(credentials: LoginCredentials): Promise<User> {
  validateCredentials(credentials);
  const user = await authenticateUser(credentials);
  return user;
}

// Helper private functions
function validateCredentials(credentials: LoginCredentials): void {
  if (!validateEmail(credentials.email)) {
    throw new ValidationError('Invalid email format');
  }
}

async function authenticateUser(credentials: LoginCredentials): Promise<User> {
  // Implementation
}
```

---

## 7. Code Duplication

### DRY Principle (Don't Repeat Yourself)
- [ ] No duplicated code blocks
- [ ] Common logic extracted to reusable functions
- [ ] Shared constants defined once
- [ ] Duplication acceptable only when abstraction would harm clarity

**Example:**
```javascript
// ❌ BAD - Duplicated validation logic
function createUser(data) {
  if (!data.email || !data.email.includes('@')) {
    throw new Error('Invalid email');
  }
  if (!data.password || data.password.length < 8) {
    throw new Error('Password too short');
  }
  // ... create user
}

function updateUser(id, data) {
  if (!data.email || !data.email.includes('@')) {
    throw new Error('Invalid email');
  }
  if (data.password && data.password.length < 8) {
    throw new Error('Password too short');
  }
  // ... update user
}

// ✅ GOOD - Extracted validation
function validateUserData(data, isUpdate = false) {
  if (!data.email || !data.email.includes('@')) {
    throw new Error('Invalid email');
  }
  
  const passwordRequired = !isUpdate;
  if ((passwordRequired && !data.password) || 
      (data.password && data.password.length < 8)) {
    throw new Error('Password too short');
  }
}

function createUser(data) {
  validateUserData(data);
  // ... create user
}

function updateUser(id, data) {
  validateUserData(data, true);
  // ... update user
}
```

---

## 8. Magic Numbers and Strings

### Named Constants
- [ ] No magic numbers in code
- [ ] String literals used once or defined as constants
- [ ] Configuration values externalized
- [ ] Enums used for fixed sets of values

**Example:**
```python
# ❌ BAD - Magic numbers and strings
def calculate_price(amount):
    if amount > 1000:
        return amount * 0.9
    elif amount > 500:
        return amount * 0.95
    return amount

def get_user_role(user):
    if user.role == "admin":
        return 1
    elif user.role == "moderator":
        return 2
    return 3

# ✅ GOOD - Named constants and enums
from enum import Enum

class DiscountTier:
    PREMIUM_THRESHOLD = 1000
    PREMIUM_DISCOUNT = 0.10
    STANDARD_THRESHOLD = 500
    STANDARD_DISCOUNT = 0.05

class UserRole(Enum):
    ADMIN = 1
    MODERATOR = 2
    USER = 3

def calculate_price(amount: float) -> float:
    if amount > DiscountTier.PREMIUM_THRESHOLD:
        return amount * (1 - DiscountTier.PREMIUM_DISCOUNT)
    elif amount > DiscountTier.STANDARD_THRESHOLD:
        return amount * (1 - DiscountTier.STANDARD_DISCOUNT)
    return amount
```

---

## 9. Error Messages

### User-Friendly Errors
- [ ] Error messages are clear and actionable
- [ ] Technical jargon avoided in user-facing errors
- [ ] Errors provide context and next steps
- [ ] No stack traces or sensitive info in user-facing errors

**Example:**
```typescript
// ❌ BAD - Unhelpful error
throw new Error('Error');
throw new Error('Failed');

// ✅ GOOD - Clear, actionable error
throw new ValidationError(
  'Email address is required. Please provide a valid email address.'
);

throw new PaymentError(
  'Payment could not be processed. Please check your card details and try again.',
  { code: 'CARD_DECLINED', retryable: true }
);
```

---

## Review Checklist Summary

Use this quick checklist during PR reviews:

- [ ] **Ternaries**: No nested ternaries, complex conditions extracted
- [ ] **Complexity**: Functions focused, complexity < 10
- [ ] **Naming**: Clear, expressive names following conventions
- [ ] **Comments**: Explain "why", not "what", TODOs have context
- [ ] **Types**: Explicit types, avoid `any`, proper null handling
- [ ] **Organization**: Logical structure, single responsibility
- [ ] **DRY**: No duplication, common logic extracted
- [ ] **Constants**: No magic numbers/strings, named constants used
- [ ] **Errors**: Clear, actionable error messages

---

## Tools for Code Quality

**Linters:**
```bash
# TypeScript/JavaScript
eslint src/
prettier --check src/

# Python
flake8 src/
black --check src/
pylint src/

# Java/Kotlin
./gradlew ktlintCheck
```

**Complexity Analysis:**
```bash
# JavaScript/TypeScript
npx complexity-report src/

# Python
radon cc src/ -a
```

---

## References

- [Clean Code (Book) by Robert C. Martin](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
- [Google Style Guides](https://google.github.io/styleguide/)
- [Refactoring Guru - Code Smells](https://refactoring.guru/refactoring/smells)
- [The Art of Readable Code](https://www.oreilly.com/library/view/the-art-of/9781449318482/)
