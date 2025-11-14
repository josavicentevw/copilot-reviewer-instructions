# Java/Kotlin Stack-Specific Rules

This document provides Kotlin and Java-specific validation rules for code reviews. These rules supplement the general checklists and should be applied when reviewing Kotlin or Java code.

---

## 1. Null Safety {#null-safety}

### Kotlin Null-Safety Requirements
- [ ] Use Kotlin's null-safe types (`String?` vs `String`)
- [ ] Avoid `!!` operator (null assertion) unless absolutely necessary with justification comment
- [ ] Use safe call operator `?.` and Elvis operator `?:` appropriately
- [ ] Leverage `let`, `run`, `also`, `apply` for null-safe operations
- [ ] No unnecessary null checks for non-nullable types

**Examples:**
```kotlin
// ❌ BAD - Using !! operator without justification
val name = user.name!!.uppercase()

// ❌ BAD - Unnecessary null check for non-nullable type
fun processUser(user: User) {
    if (user != null) { // User is already non-nullable
        // ...
    }
}

// ✅ GOOD - Safe call with Elvis operator
val name = user.name?.uppercase() ?: "UNKNOWN"

// ✅ GOOD - Using let for null-safe operations
user.name?.let { name ->
    logger.info("Processing user: $name")
    processName(name)
}

// ✅ ACCEPTABLE - !! with justification
// Using !! here is safe because we just validated the user exists in the previous line
val user = userRepository.findById(id) ?: throw UserNotFoundException(id)
val name = user.name!! // Name is required field, validated at creation
```

### Java Optional Usage
- [ ] Use `Optional<T>` for methods that may return null
- [ ] Never return `null` from methods that return `Optional`
- [ ] Use `Optional.ofNullable()` when wrapping potentially null values
- [ ] Avoid `Optional.get()` without `isPresent()` check
- [ ] Use `orElse()`, `orElseGet()`, `orElseThrow()` appropriately

**Examples:**
```java
// ❌ BAD - Returning null from Optional method
public Optional<User> findUser(String id) {
    return null; // Should return Optional.empty()
}

// ❌ BAD - Using get() without isPresent() check
Optional<User> userOpt = findUser(id);
User user = userOpt.get(); // May throw NoSuchElementException

// ✅ GOOD - Proper Optional usage
public Optional<User> findUser(String id) {
    User user = userRepository.findById(id);
    return Optional.ofNullable(user);
}

// ✅ GOOD - Safe Optional consumption
User user = findUser(id)
    .orElseThrow(() -> new UserNotFoundException(id));

String userName = findUser(id)
    .map(User::getName)
    .orElse("Unknown");
```

---

## 2. Data Classes and Sealed Classes {#data-classes}

### Data Classes
- [ ] Use `data class` for simple data holders
- [ ] Include only properties needed for equality/hashCode in primary constructor
- [ ] Override `toString()`, `equals()`, `hashCode()` only when default behavior insufficient
- [ ] Use `copy()` method for immutable updates

**Examples:**
```kotlin
// ✅ GOOD - Simple data class
data class UserResponse(
    val id: String,
    val name: String,
    val email: String,
    val createdAt: Instant
)

// ✅ GOOD - Data class with validation
data class UserRequest(
    val name: String,
    val email: String
) {
    init {
        require(name.isNotBlank()) { "Name cannot be blank" }
        require(email.contains("@")) { "Invalid email format" }
    }
}

// ✅ GOOD - Immutable update with copy
val updatedUser = user.copy(name = "New Name")
```

### Sealed Classes
- [ ] Use `sealed class` for representing restricted class hierarchies
- [ ] Prefer `sealed class` over enum when subclasses need different properties
- [ ] Leverage exhaustive `when` expressions with sealed classes
- [ ] Define sealed classes in same file as subclasses (or same package in Kotlin 1.5+)

**Examples:**
```kotlin
// ✅ GOOD - Sealed class for result types
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val message: String, val cause: Throwable? = null) : Result<Nothing>()
    object Loading : Result<Nothing>()
}

// ✅ GOOD - Exhaustive when expression
fun <T> handleResult(result: Result<T>) {
    when (result) {
        is Result.Success -> logger.info("Success: ${result.data}")
        is Result.Error -> logger.error("Error: ${result.message}", result.cause)
        Result.Loading -> logger.info("Loading...")
    }
    // No else branch needed - compiler ensures exhaustiveness
}

// ✅ GOOD - Sealed class for domain events
sealed class OrderEvent {
    data class Created(val orderId: String, val customerId: String) : OrderEvent()
    data class Paid(val orderId: String, val amount: Double) : OrderEvent()
    data class Shipped(val orderId: String, val trackingNumber: String) : OrderEvent()
    data class Cancelled(val orderId: String, val reason: String) : OrderEvent()
}
```

---

## 3. Repository and DAO Patterns {#repository-patterns}

### Parameterized Queries
- [ ] All database queries use parameterized queries or prepared statements
- [ ] No string concatenation for building SQL/JPQL/HQL queries
- [ ] Use named parameters in JPA/Hibernate queries
- [ ] DAO methods have appropriate `@Query` annotations

**Examples:**
```kotlin
// ❌ BAD - SQL injection vulnerability
@Repository
interface UserRepository : JpaRepository<User, Long> {
    @Query("SELECT u FROM User u WHERE u.email = '$email'")
    fun findByEmail(email: String): User?
}

// ✅ GOOD - Parameterized query with named parameter
@Repository
interface UserRepository : JpaRepository<User, Long> {
    @Query("SELECT u FROM User u WHERE u.email = :email")
    fun findByEmail(@Param("email") email: String): User?
}

// ✅ GOOD - Using Spring Data JPA method naming
@Repository
interface UserRepository : JpaRepository<User, Long> {
    fun findByEmail(email: String): User?
    fun findByStatusAndCreatedAtAfter(status: Status, date: Instant): List<User>
}

// ✅ GOOD - Native query with parameters
@Repository
interface OrderRepository : JpaRepository<Order, Long> {
    @Query(
        value = "SELECT * FROM orders WHERE status = :status AND created_at > :since",
        nativeQuery = true
    )
    fun findRecentOrders(
        @Param("status") status: String,
        @Param("since") since: Instant
    ): List<Order>
}
```

### Required Database Indexes
- [ ] Entity classes have appropriate `@Table(indexes = ...)` annotations
- [ ] Indexes exist for foreign keys
- [ ] Indexes exist for columns used in WHERE clauses
- [ ] Composite indexes for multi-column queries
- [ ] Unique constraints defined where applicable

**Examples:**
```kotlin
// ✅ GOOD - Entity with proper indexes
@Entity
@Table(
    name = "users",
    indexes = [
        Index(name = "idx_users_email", columnList = "email", unique = true),
        Index(name = "idx_users_status", columnList = "status"),
        Index(name = "idx_users_created_at", columnList = "created_at")
    ]
)
data class UserEntity(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0,
    
    @Column(nullable = false, unique = true)
    val email: String,
    
    @Column(nullable = false)
    val status: String,
    
    @Column(name = "created_at", nullable = false)
    val createdAt: Instant = Instant.now()
)

// ✅ GOOD - Composite index for complex queries
@Entity
@Table(
    name = "orders",
    indexes = [
        Index(name = "idx_orders_user_status", columnList = "user_id,status"),
        Index(name = "idx_orders_status_created", columnList = "status,created_at DESC")
    ]
)
data class OrderEntity(
    @Id val id: Long,
    @Column(name = "user_id") val userId: Long,
    val status: String,
    @Column(name = "created_at") val createdAt: Instant
)
```

---

## 4. Dependency Injection and Configuration {#dependency-injection}

### Spring Dependency Injection
- [ ] Use constructor injection (preferred over field injection)
- [ ] Avoid `@Autowired` on fields (use constructor injection instead)
- [ ] Use `@Configuration` classes for bean definitions
- [ ] Avoid circular dependencies
- [ ] Use `@Qualifier` when multiple beans of same type exist

**Examples:**
```kotlin
// ❌ BAD - Field injection
@Service
class UserService {
    @Autowired
    private lateinit var userRepository: UserRepository
    
    @Autowired
    private lateinit var emailService: EmailService
}

// ✅ GOOD - Constructor injection
@Service
class UserService(
    private val userRepository: UserRepository,
    private val emailService: EmailService
) {
    fun createUser(request: UserRequest): User {
        // ...
    }
}

// ✅ GOOD - Configuration class
@Configuration
class AppConfiguration {
    
    @Bean
    fun objectMapper(): ObjectMapper {
        return ObjectMapper().apply {
            registerModule(JavaTimeModule())
            disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
        }
    }
    
    @Bean
    fun httpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .connectTimeout(10, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .build()
    }
}

// ✅ GOOD - Using @Qualifier for multiple beans
@Service
class PaymentService(
    @Qualifier("stripeClient") private val stripeClient: PaymentClient,
    @Qualifier("paypalClient") private val paypalClient: PaymentClient
) {
    // ...
}
```

### Configuration Properties
- [ ] Use `@ConfigurationProperties` for grouped properties
- [ ] Validate configuration properties with `@Validated`
- [ ] Use type-safe configuration classes
- [ ] Document required configuration in README

**Examples:**
```kotlin
// ✅ GOOD - Type-safe configuration properties
@ConfigurationProperties(prefix = "app.payment")
@Validated
data class PaymentConfig(
    @field:NotBlank
    val apiKey: String,
    
    @field:Min(1000)
    @field:Max(60000)
    val timeout: Long = 30000,
    
    @field:NotEmpty
    val allowedCurrencies: List<String> = listOf("USD", "EUR")
)

// ✅ GOOD - Using configuration properties
@Service
class PaymentService(
    private val paymentConfig: PaymentConfig
) {
    fun processPayment(amount: Double, currency: String) {
        require(currency in paymentConfig.allowedCurrencies) {
            "Currency $currency not supported"
        }
        // ...
    }
}
```

---

## 5. Logging and Exception Handling {#logging-exception-handling}

### Logging Patterns
- [ ] Use SLF4J with appropriate log levels
- [ ] Use structured logging (log context with MDC)
- [ ] No PII in logs (mask sensitive data)
- [ ] Log exceptions with full context
- [ ] Use parameterized logging (not string concatenation)

**Examples:**
```kotlin
// ❌ BAD - String concatenation in logging
logger.info("User " + userId + " logged in")
logger.error("Error processing order: " + e.message)

// ❌ BAD - PII in logs
logger.info("User ${user.email} with SSN ${user.ssn} created")

// ✅ GOOD - Parameterized logging
logger.info("User {} logged in", userId)
logger.error("Error processing order", e)

// ✅ GOOD - Structured logging with MDC
MDC.put("userId", userId)
MDC.put("orderId", orderId)
try {
    logger.info("Processing order")
    processOrder(orderId)
} finally {
    MDC.clear()
}

// ✅ GOOD - Masked PII
logger.info("User {} created", maskEmail(user.email))

// ✅ GOOD - Companion object for logger
class UserService(
    private val userRepository: UserRepository
) {
    companion object {
        private val logger = LoggerFactory.getLogger(UserService::class.java)
    }
    
    fun createUser(request: UserRequest): User {
        logger.info("Creating user with email {}", maskEmail(request.email))
        // ...
    }
}
```

### Exception Handling
- [ ] Use specific exception types (not generic `Exception` or `RuntimeException`)
- [ ] Create custom exception classes for domain errors
- [ ] Include meaningful error messages
- [ ] Use `@ControllerAdvice` for global exception handling in Spring
- [ ] Don't swallow exceptions (log or rethrow)

**Examples:**
```kotlin
// ❌ BAD - Generic exception
throw Exception("Something went wrong")

// ❌ BAD - Swallowing exception
try {
    processPayment(order)
} catch (e: Exception) {
    // Silent failure - bad practice
}

// ✅ GOOD - Custom exception classes
class UserNotFoundException(userId: String) : 
    RuntimeException("User not found: $userId")

class PaymentFailedException(
    message: String,
    val orderId: String,
    cause: Throwable? = null
) : RuntimeException(message, cause)

class InsufficientBalanceException(
    val requiredAmount: Double,
    val availableAmount: Double
) : RuntimeException("Insufficient balance: required $requiredAmount, available $availableAmount")

// ✅ GOOD - Global exception handler
@ControllerAdvice
class GlobalExceptionHandler {
    
    companion object {
        private val logger = LoggerFactory.getLogger(GlobalExceptionHandler::class.java)
    }
    
    @ExceptionHandler(UserNotFoundException::class)
    fun handleUserNotFound(e: UserNotFoundException): ResponseEntity<ErrorResponse> {
        logger.warn("User not found: {}", e.message)
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(ErrorResponse(
                error = "USER_NOT_FOUND",
                message = e.message ?: "User not found"
            ))
    }
    
    @ExceptionHandler(PaymentFailedException::class)
    fun handlePaymentFailed(e: PaymentFailedException): ResponseEntity<ErrorResponse> {
        logger.error("Payment failed for order {}", e.orderId, e)
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
            .body(ErrorResponse(
                error = "PAYMENT_FAILED",
                message = e.message ?: "Payment processing failed",
                details = mapOf("orderId" to e.orderId)
            ))
    }
}

// ✅ GOOD - Proper exception handling with logging
fun processOrder(orderId: String): Order {
    return try {
        val order = orderRepository.findById(orderId)
            ?: throw OrderNotFoundException(orderId)
        
        paymentService.processPayment(order.paymentDetails)
        order.copy(status = OrderStatus.PAID)
    } catch (e: PaymentFailedException) {
        logger.error("Payment failed for order {}", orderId, e)
        throw e
    } catch (e: Exception) {
        logger.error("Unexpected error processing order {}", orderId, e)
        throw OrderProcessingException("Failed to process order $orderId", orderId, e)
    }
}
```

---

## 6. Kotlin-Specific Best Practices {#kotlin-best-practices}

### Extension Functions
- [ ] Use extension functions for utility methods
- [ ] Don't overuse extensions on platform types
- [ ] Group related extensions in same file
- [ ] Document public extension functions

**Examples:**
```kotlin
// ✅ GOOD - Extension functions for common operations
fun String.isValidEmail(): Boolean {
    return this.contains("@") && this.contains(".")
}

fun Instant.isOlderThan(duration: Duration): Boolean {
    return this.plus(duration).isBefore(Instant.now())
}

fun <T> List<T>.second(): T? {
    return this.getOrNull(1)
}

// ✅ GOOD - Usage
val email = "user@example.com"
if (email.isValidEmail()) {
    // ...
}
```

### Scope Functions
- [ ] Use appropriate scope functions (`let`, `run`, `with`, `apply`, `also`)
- [ ] Don't nest scope functions deeply (max 2 levels)
- [ ] Use `let` for null-safe operations
- [ ] Use `apply` for object configuration

**Examples:**
```kotlin
// ✅ GOOD - Using let for null-safe operations
user.email?.let { email ->
    sendEmail(email)
}

// ✅ GOOD - Using apply for object configuration
val user = User().apply {
    name = "John Doe"
    email = "john@example.com"
    status = UserStatus.ACTIVE
}

// ✅ GOOD - Using also for side effects
val result = calculateResult()
    .also { logger.info("Calculation result: $it") }
    .also { cacheResult(it) }
```

---

## Review Checklist Summary

Quick checklist for Kotlin/Java code reviews:

- [ ] **Null Safety**: Proper use of null-safe types, avoid `!!`, use Optional correctly
- [ ] **Data/Sealed Classes**: Appropriate use of data classes and sealed classes
- [ ] **Repository**: Parameterized queries, proper indexes defined
- [ ] **Dependency Injection**: Constructor injection, proper configuration
- [ ] **Logging**: SLF4J with structured logging, no PII, parameterized messages
- [ ] **Exceptions**: Specific exception types, custom exceptions, global handler
- [ ] **Kotlin Best Practices**: Proper use of extensions and scope functions

---

## References

- [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html)
- [Spring Boot Best Practices](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Effective Kotlin](https://kt.academy/book/effectivekotlin)
- [JPA Best Practices](https://thoughts-on-java.org/jpa-best-practices/)
