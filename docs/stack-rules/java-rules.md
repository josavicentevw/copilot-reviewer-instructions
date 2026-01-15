# Java Stack-Specific Rules

This document provides Java-specific validation rules for code reviews. These rules supplement the general checklists and should be applied when reviewing Java code.

---

## 1. Null Safety {#null-safety}

### Optional Usage
- [ ] Use `Optional<T>` for methods that may return null
- [ ] Never return `null` from methods that return `Optional`
- [ ] Use `Optional.ofNullable()` when wrapping potentially null values
- [ ] Avoid `Optional.get()` without `isPresent()` check
- [ ] Use `orElse()`, `orElseGet()`, `orElseThrow()` appropriately
- [ ] Don't use Optional for class fields (use nullable types instead)

**Examples:**
```java
// ❌ BAD - Returning null from Optional method
public Optional<User> findUser(String id) {
    return null; // Should return Optional.empty()
}

// ❌ BAD - Using get() without isPresent() check
Optional<User> userOpt = findUser(id);
User user = userOpt.get(); // May throw NoSuchElementException

// ❌ BAD - Optional as class field
public class UserService {
    private Optional<UserRepository> repository; // Don't do this
}

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

// ✅ GOOD - Using ifPresent
findUser(id).ifPresent(user -> {
    logger.info("Found user: {}", user.getName());
    processUser(user);
});

// ✅ GOOD - Chaining Optional operations
String email = findUser(id)
    .map(User::getProfile)
    .map(Profile::getEmail)
    .orElse("no-email@example.com");
```

### Null Annotations
- [ ] Use `@NonNull` and `@Nullable` annotations (javax.annotation or org.jetbrains.annotations)
- [ ] Enable null-checking tools (SpotBugs, NullAway, Checker Framework)
- [ ] Validate input parameters for null
- [ ] Use `Objects.requireNonNull()` for parameter validation

**Examples:**
```java
import javax.annotation.Nonnull;
import javax.annotation.Nullable;
import java.util.Objects;

// ✅ GOOD - Null annotations
public class UserService {
    
    @Nonnull
    public User createUser(@Nonnull UserRequest request) {
        Objects.requireNonNull(request, "UserRequest cannot be null");
        Objects.requireNonNull(request.getEmail(), "Email cannot be null");
        
        // Implementation
        return user;
    }
    
    @Nullable
    public User findUserByEmail(@Nonnull String email) {
        Objects.requireNonNull(email, "Email cannot be null");
        return userRepository.findByEmail(email);
    }
}
```

---

## 2. Streams and Functional Programming {#streams}

### Stream Best Practices
- [ ] Use streams for collection operations (filter, map, reduce)
- [ ] Don't modify collections while streaming
- [ ] Use appropriate terminal operations (collect, findFirst, anyMatch, etc.)
- [ ] Avoid side effects in stream operations
- [ ] Use parallel streams only for CPU-intensive operations
- [ ] Close streams that use resources (Files.lines, etc.)

**Examples:**
```java
// ❌ BAD - Modifying collection during stream
List<User> users = getUsers();
users.stream()
    .filter(user -> user.isActive())
    .forEach(user -> users.remove(user)); // ConcurrentModificationException

// ❌ BAD - Side effects in stream
List<String> result = new ArrayList<>();
users.stream()
    .filter(User::isActive)
    .forEach(user -> result.add(user.getName())); // Side effect

// ✅ GOOD - Proper stream usage
List<String> activeUserNames = users.stream()
    .filter(User::isActive)
    .map(User::getName)
    .collect(Collectors.toList());

// ✅ GOOD - Stream with findFirst
Optional<User> adminUser = users.stream()
    .filter(User::isAdmin)
    .findFirst();

// ✅ GOOD - Grouping with Collectors
Map<String, List<User>> usersByStatus = users.stream()
    .collect(Collectors.groupingBy(User::getStatus));

// ✅ GOOD - Custom collector
Double totalPrice = orders.stream()
    .map(Order::getPrice)
    .collect(Collectors.summingDouble(Double::doubleValue));

// ✅ GOOD - Parallel stream for CPU-intensive work
List<Result> results = largeDataset.parallelStream()
    .map(data -> expensiveComputation(data))
    .collect(Collectors.toList());

// ✅ GOOD - Try-with-resources for file streams
try (Stream<String> lines = Files.lines(Paths.get("file.txt"))) {
    long count = lines
        .filter(line -> line.contains("error"))
        .count();
}
```

---

## 3. Exception Handling {#exception-handling}

### Exception Best Practices
- [ ] Use specific exception types (not generic `Exception` or `RuntimeException`)
- [ ] Create custom exception classes for domain errors
- [ ] Include meaningful error messages and context
- [ ] Use try-with-resources for AutoCloseable resources
- [ ] Don't swallow exceptions silently
- [ ] Log exceptions with full context

**Examples:**
```java
// ❌ BAD - Generic exception
public void processOrder(String orderId) throws Exception {
    throw new Exception("Something went wrong");
}

// ❌ BAD - Swallowing exception
try {
    processPayment(order);
} catch (Exception e) {
    // Silent failure - bad practice
}

// ❌ BAD - Catching too broad
try {
    processOrder(orderId);
} catch (Exception e) {
    // Catches everything including RuntimeExceptions
    logger.error("Error", e);
}

// ✅ GOOD - Custom exception classes
public class UserNotFoundException extends RuntimeException {
    private final String userId;
    
    public UserNotFoundException(String userId) {
        super("User not found: " + userId);
        this.userId = userId;
    }
    
    public String getUserId() {
        return userId;
    }
}

public class PaymentFailedException extends RuntimeException {
    private final String orderId;
    
    public PaymentFailedException(String message, String orderId, Throwable cause) {
        super(message, cause);
        this.orderId = orderId;
    }
    
    public String getOrderId() {
        return orderId;
    }
}

// ✅ GOOD - Specific exception handling
private static final Logger logger = LoggerFactory.getLogger(OrderService.class);

public Order processOrder(String orderId) {
    try {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));
        
        paymentService.processPayment(order.getPaymentDetails());
        order.setStatus(OrderStatus.PAID);
        return order;
        
    } catch (PaymentFailedException e) {
        logger.error("Payment failed for order {}", orderId, e);
        throw e;
    } catch (Exception e) {
        logger.error("Unexpected error processing order {}", orderId, e);
        throw new OrderProcessingException("Failed to process order " + orderId, orderId, e);
    }
}

// ✅ GOOD - Try-with-resources
public String readFile(String path) throws IOException {
    try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
        return reader.lines().collect(Collectors.joining("\n"));
    }
}

// ✅ GOOD - Multiple resources
public void copyFile(String source, String dest) throws IOException {
    try (InputStream in = new FileInputStream(source);
         OutputStream out = new FileOutputStream(dest)) {
        byte[] buffer = new byte[8192];
        int length;
        while ((length = in.read(buffer)) > 0) {
            out.write(buffer, 0, length);
        }
    }
}
```

---

## 4. Repository and JPA Patterns {#repository-jpa}

### JPA/Hibernate Best Practices
- [ ] Use parameterized queries (never string concatenation)
- [ ] Define proper indexes on entities
- [ ] Configure fetch types appropriately (LAZY vs EAGER)
- [ ] Use projections for read-only queries
- [ ] Avoid N+1 query problems

**Examples:**
```java
// ❌ BAD - SQL injection vulnerability
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    @Query("SELECT u FROM User u WHERE u.email = '" + email + "'")
    User findByEmail(String email);
}

// ❌ BAD - No indexes
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String email;
    
    @Column(nullable = false)
    private String status;
}

// ✅ GOOD - Parameterized query
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmail(@Param("email") String email);
    
    // Or use Spring Data JPA method naming
    Optional<User> findByEmail(String email);
    
    List<User> findByStatusAndCreatedAtAfter(String status, Instant createdAt);
}

// ✅ GOOD - Entity with proper indexes
@Entity
@Table(
    name = "users",
    indexes = {
        @Index(name = "idx_users_email", columnList = "email", unique = true),
        @Index(name = "idx_users_status", columnList = "status"),
        @Index(name = "idx_users_created_at", columnList = "created_at")
    }
)
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    @Column(nullable = false)
    private String status;
    
    @Column(name = "created_at", nullable = false)
    private Instant createdAt;
    
    // Getters and setters
}

// ✅ GOOD - Composite index
@Entity
@Table(
    name = "orders",
    indexes = {
        @Index(name = "idx_orders_user_status", columnList = "user_id,status"),
        @Index(name = "idx_orders_status_created", columnList = "status,created_at")
    }
)
public class Order {
    @Id
    private Long id;
    
    @Column(name = "user_id")
    private Long userId;
    
    @Column
    private String status;
    
    @Column(name = "created_at")
    private Instant createdAt;
}

// ✅ GOOD - Proper fetch type configuration
@Entity
public class Order {
    @Id
    private Long id;
    
    // LAZY for collections to avoid N+1
    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    private List<OrderItem> items;
    
    // Can be EAGER for single-valued associations
    @ManyToOne(fetch = FetchType.LAZY) // Still prefer LAZY
    @JoinColumn(name = "user_id")
    private User user;
}

// ✅ GOOD - Using EntityGraph to avoid N+1
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    @EntityGraph(attributePaths = {"items", "user"})
    @Query("SELECT o FROM Order o WHERE o.status = :status")
    List<Order> findByStatusWithDetails(@Param("status") String status);
}

// ✅ GOOD - Projection for read-only queries
public interface UserSummary {
    String getName();
    String getEmail();
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    List<UserSummary> findByStatus(String status);
}
```

---

## 5. Dependency Injection with Spring {#dependency-injection}

### Spring DI Best Practices
- [ ] Use constructor injection (preferred over field injection)
- [ ] Avoid `@Autowired` on fields
- [ ] Use `@Configuration` classes for bean definitions
- [ ] Avoid circular dependencies
- [ ] Use `@Qualifier` when multiple beans of same type exist

**Examples:**
```java
// ❌ BAD - Field injection
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private EmailService emailService;
}

// ❌ BAD - Setter injection for required dependencies
@Service
public class UserService {
    private UserRepository userRepository;
    
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

// ✅ GOOD - Constructor injection
@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
    
    public User createUser(UserRequest request) {
        // Implementation
    }
}

// ✅ GOOD - Constructor injection with Lombok
import lombok.RequiredArgsConstructor;

@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    public User createUser(UserRequest request) {
        // Implementation
    }
}

// ✅ GOOD - Configuration class
@Configuration
public class AppConfiguration {
    
    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(new JavaTimeModule());
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        return mapper;
    }
    
    @Bean
    public RestTemplate restTemplate() {
        RestTemplate restTemplate = new RestTemplate();
        
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        factory.setConnectTimeout(10000);
        factory.setReadTimeout(30000);
        restTemplate.setRequestFactory(factory);
        
        return restTemplate;
    }
}

// ✅ GOOD - Using @Qualifier for multiple beans
@Service
public class PaymentService {
    private final PaymentClient stripeClient;
    private final PaymentClient paypalClient;
    
    public PaymentService(
        @Qualifier("stripeClient") PaymentClient stripeClient,
        @Qualifier("paypalClient") PaymentClient paypalClient
    ) {
        this.stripeClient = stripeClient;
        this.paypalClient = paypalClient;
    }
}
```

### Configuration Properties
- [ ] Use `@ConfigurationProperties` for grouped properties
- [ ] Validate configuration with `@Validated`
- [ ] Use type-safe configuration classes
- [ ] Document required configuration

**Examples:**
```java
// ✅ GOOD - Type-safe configuration properties
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.validation.annotation.Validated;

import javax.validation.constraints.Min;
import javax.validation.constraints.Max;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotEmpty;
import java.util.List;

@ConfigurationProperties(prefix = "app.payment")
@Validated
public class PaymentConfig {
    
    @NotBlank
    private String apiKey;
    
    @Min(1000)
    @Max(60000)
    private long timeout = 30000;
    
    @NotEmpty
    private List<String> allowedCurrencies = List.of("USD", "EUR");
    
    // Getters and setters
    public String getApiKey() {
        return apiKey;
    }
    
    public void setApiKey(String apiKey) {
        this.apiKey = apiKey;
    }
    
    public long getTimeout() {
        return timeout;
    }
    
    public void setTimeout(long timeout) {
        this.timeout = timeout;
    }
    
    public List<String> getAllowedCurrencies() {
        return allowedCurrencies;
    }
    
    public void setAllowedCurrencies(List<String> allowedCurrencies) {
        this.allowedCurrencies = allowedCurrencies;
    }
}

// Enable configuration properties
@Configuration
@EnableConfigurationProperties(PaymentConfig.class)
public class AppConfiguration {
}

// ✅ GOOD - Using configuration properties
@Service
public class PaymentService {
    private final PaymentConfig config;
    
    public PaymentService(PaymentConfig config) {
        this.config = config;
    }
    
    public void processPayment(double amount, String currency) {
        if (!config.getAllowedCurrencies().contains(currency)) {
            throw new IllegalArgumentException("Currency not supported: " + currency);
        }
        // Process payment
    }
}
```

---

## 6. Logging Best Practices {#logging}

### SLF4J Logging
- [ ] Use SLF4J with appropriate log levels
- [ ] Use parameterized logging (not string concatenation)
- [ ] Log exceptions with full context
- [ ] Use MDC for structured logging
- [ ] Don't log PII (mask sensitive data)

**Examples:**
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;

// ❌ BAD - String concatenation
logger.info("User " + userId + " logged in");
logger.error("Error: " + e.getMessage());

// ❌ BAD - PII in logs
logger.info("User {} with SSN {} created", user.getEmail(), user.getSsn());

// ✅ GOOD - Parameterized logging
private static final Logger logger = LoggerFactory.getLogger(UserService.class);

public class UserService {
    
    public void login(String userId) {
        logger.info("User {} logged in", userId);
    }
    
    public void processOrder(String orderId) {
        logger.error("Error processing order {}", orderId, exception);
    }
}

// ✅ GOOD - Structured logging with MDC
public void processOrder(String orderId, String userId) {
    MDC.put("orderId", orderId);
    MDC.put("userId", userId);
    try {
        logger.info("Processing order");
        // Process order logic
        logger.info("Order processed successfully");
    } catch (Exception e) {
        logger.error("Failed to process order", e);
        throw e;
    } finally {
        MDC.clear();
    }
}

// ✅ GOOD - Masked PII
public void createUser(User user) {
    logger.info("Creating user with email {}", maskEmail(user.getEmail()));
    // Create user
}

private String maskEmail(String email) {
    int atIndex = email.indexOf('@');
    if (atIndex <= 1) return "***@***";
    return email.charAt(0) + "***@" + email.substring(atIndex + 1);
}

// ✅ GOOD - Appropriate log levels
logger.trace("Detailed trace information");           // Very detailed
logger.debug("Debug information: {}", debugData);     // Debugging
logger.info("User {} created successfully", userId);  // Important info
logger.warn("Retry attempt {} failed", attempt);      // Warning
logger.error("Failed to process payment", exception); // Error
```

---

## 7. Immutability and Records {#immutability}

### Java Records (Java 14+)
- [ ] Use records for immutable data carriers
- [ ] Leverage compact constructors for validation
- [ ] Use records for DTOs and value objects
- [ ] Avoid records with mutable components

**Examples:**
```java
// ✅ GOOD - Simple record
public record UserResponse(
    String id,
    String name,
    String email,
    Instant createdAt
) {}

// ✅ GOOD - Record with validation
public record UserRequest(
    String name,
    String email
) {
    public UserRequest {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be blank");
        }
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Invalid email format");
        }
    }
}

// ✅ GOOD - Record with derived values
public record Point(double x, double y) {
    public double distanceFromOrigin() {
        return Math.sqrt(x * x + y * y);
    }
}

// ❌ BAD - Record with mutable component
public record UserData(String name, List<String> roles) {}
// List is mutable, can be modified externally

// ✅ GOOD - Record with immutable component
public record UserData(String name, List<String> roles) {
    public UserData {
        roles = List.copyOf(roles); // Make defensive copy
    }
}
```

### Collections Immutability
- [ ] Use `List.of()`, `Set.of()`, `Map.of()` for immutable collections
- [ ] Use `Collections.unmodifiableList()` for wrapping mutable collections
- [ ] Make defensive copies when needed

**Examples:**
```java
// ✅ GOOD - Immutable collections
List<String> roles = List.of("ADMIN", "USER");
Set<String> permissions = Set.of("READ", "WRITE");
Map<String, String> config = Map.of("theme", "dark", "lang", "en");

// ✅ GOOD - Defensive copy in constructor
public class User {
    private final List<String> roles;
    
    public User(List<String> roles) {
        this.roles = List.copyOf(roles); // Defensive copy
    }
    
    public List<String> getRoles() {
        return roles; // Already immutable
    }
}
```

---

## Review Checklist Summary

Quick checklist for Java code reviews:

- [ ] **Null Safety**: Proper Optional usage, null annotations, Objects.requireNonNull
- [ ] **Streams**: Appropriate stream operations, avoid side effects
- [ ] **Exceptions**: Specific exception types, try-with-resources, proper handling
- [ ] **JPA**: Parameterized queries, proper indexes, fetch types configured
- [ ] **DI**: Constructor injection, configuration properties validated
- [ ] **Logging**: SLF4J with parameterized messages, MDC for context, no PII
- [ ] **Immutability**: Records for DTOs, immutable collections
- [ ] **Code Style**: Follow Java conventions, use Lombok when appropriate

---

## Tools for Code Quality

**Linters and Formatters:**
```bash
# Maven
mvn checkstyle:check
mvn spotbugs:check
mvn pmd:check

# Gradle
./gradlew checkstyleMain
./gradlew spotbugsMain
./gradlew pmdMain

# Formatting
mvn spotless:apply
./gradlew spotlessApply
```

**Static Analysis:**
```bash
# SpotBugs for bug detection
# PMD for code quality
# Checkstyle for style enforcement
# SonarQube for comprehensive analysis
```

---

## References

- [Effective Java (3rd Edition) by Joshua Bloch](https://www.oreilly.com/library/view/effective-java/9780134686097/)
- [Spring Framework Documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/)
- [Java Code Conventions](https://www.oracle.com/java/technologies/javase/codeconventions-contents.html)
- [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
- [JPA Best Practices](https://thoughts-on-java.org/jpa-best-practices/)
