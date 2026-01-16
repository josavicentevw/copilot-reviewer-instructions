# Java Kotlin Examples

## 1. Null Safety {#null-safety}
### Example 1
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
### Example 2
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

## 2. Data Classes and Sealed Classes {#data-classes}
### Example 1
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
### Example 2
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

## 3. Repository and DAO Patterns {#repository-patterns}
### Example 1
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
### Example 2
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

## 4. Dependency Injection and Configuration {#dependency-injection}
### Example 1
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
### Example 2
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

## 5. Logging and Exception Handling {#logging-exception-handling}
### Example 1
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
### Example 2
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

## 6. Kotlin-Specific Best Practices {#kotlin-best-practices}
### Example 1
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
### Example 2
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

## 7. Coroutines and Flow {#coroutines-flow}
### Example 1
```kotlin
// ❌ BAD - Using GlobalScope
GlobalScope.launch {
    repository.sync()
}

// ✅ GOOD - Structured concurrency
class SyncViewModel(
    private val repository: SyncRepository
) : ViewModel() {
    fun sync() = viewModelScope.launch {
        withContext(Dispatchers.IO) {
            repository.sync()
        }
    }
}
```
### Example 2
```kotlin
// ✅ GOOD - Collecting with repeatOnLifecycle
viewLifecycleOwner.lifecycleScope.launch {
    viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiState.collect { state ->
            render(state)
        }
    }
}
```

## 8. Testing & Serialization {#testing-serialization}
### Example 1
```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
class UserRepositoryTest {
    private val dispatcher = StandardTestDispatcher()

    @Before
    fun setup() {
        Dispatchers.setMain(dispatcher)
    }

    @After
    fun tearDown() {
        Dispatchers.resetMain()
    }

    @Test
    fun `emits user`() = runTest {
        val repository = UserRepository(api, dispatcher)
        val result = repository.loadUser("123")
        assertEquals("123", result.id)
    }
}
```
