# Go Stack-Specific Rules

This document provides Go-specific validation rules for code reviews. These rules supplement the general checklists and should be applied when reviewing Go code.

---

## 1. Error Handling {#error-handling}

### Error Handling Best Practices
- [ ] Always check errors explicitly
- [ ] Return errors instead of panicking
- [ ] Wrap errors with context using `fmt.Errorf` with `%w`
- [ ] Create custom error types for domain errors
- [ ] Use `errors.Is` and `errors.As` for error comparison

**Examples:**
```go
// ❌ BAD - Ignoring errors
func readFile(path string) string {
    data, _ := os.ReadFile(path) // Ignoring error
    return string(data)
}

// ❌ BAD - Panicking instead of returning error
func getUser(id string) User {
    user, err := userRepo.FindByID(id)
    if err != nil {
        panic(err) // Don't panic in library code
    }
    return user
}

// ✅ GOOD - Proper error handling
func readFile(path string) (string, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return "", fmt.Errorf("failed to read file %s: %w", path, err)
    }
    return string(data), nil
}

// ✅ GOOD - Custom error types
type UserNotFoundError struct {
    UserID string
}

func (e *UserNotFoundError) Error() string {
    return fmt.Sprintf("user not found: %s", e.UserID)
}

type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error on field '%s': %s", e.Field, e.Message)
}

// ✅ GOOD - Using custom errors
func getUser(id string) (*User, error) {
    user, err := userRepo.FindByID(id)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, &UserNotFoundError{UserID: id}
        }
        return nil, fmt.Errorf("failed to get user %s: %w", id, err)
    }
    return user, nil
}

// ✅ GOOD - Error checking with errors.Is and errors.As
func processUser(id string) error {
    user, err := getUser(id)
    if err != nil {
        var notFoundErr *UserNotFoundError
        if errors.As(err, &notFoundErr) {
            log.Printf("User %s not found, creating new user", notFoundErr.UserID)
            return createDefaultUser(id)
        }
        return fmt.Errorf("failed to process user: %w", err)
    }
    
    // Process user
    return nil
}

// ✅ GOOD - Sentinel errors
var (
    ErrInvalidInput = errors.New("invalid input")
    ErrUnauthorized = errors.New("unauthorized")
    ErrTimeout      = errors.New("operation timeout")
)

func validateInput(data string) error {
    if data == "" {
        return ErrInvalidInput
    }
    return nil
}
```

---

## 2. Context Usage {#context-usage}

### Context Best Practices
- [ ] Pass `context.Context` as first parameter
- [ ] Use `context.WithTimeout` or `context.WithDeadline` for operations with timeout
- [ ] Propagate context through call chain
- [ ] Check context cancellation in long-running operations
- [ ] Don't store context in structs

**Examples:**
```go
// ❌ BAD - No context
func fetchUser(id string) (*User, error) {
    resp, err := http.Get(fmt.Sprintf("/api/users/%s", id))
    // ...
}

// ❌ BAD - Context stored in struct
type UserService struct {
    ctx context.Context // Don't do this
}

// ✅ GOOD - Context as first parameter
func fetchUser(ctx context.Context, id string) (*User, error) {
    req, err := http.NewRequestWithContext(ctx, "GET", fmt.Sprintf("/api/users/%s", id), nil)
    if err != nil {
        return nil, fmt.Errorf("failed to create request: %w", err)
    }
    
    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        return nil, fmt.Errorf("failed to fetch user: %w", err)
    }
    defer resp.Body.Close()
    
    var user User
    if err := json.NewDecoder(resp.Body).Decode(&user); err != nil {
        return nil, fmt.Errorf("failed to decode user: %w", err)
    }
    
    return &user, nil
}

// ✅ GOOD - Context with timeout
func getUserWithTimeout(id string) (*User, error) {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    return fetchUser(ctx, id)
}

// ✅ GOOD - Checking context cancellation
func processLargeDataset(ctx context.Context, data []Item) error {
    for i, item := range data {
        // Check context cancellation periodically
        if i%100 == 0 {
            select {
            case <-ctx.Done():
                return ctx.Err()
            default:
            }
        }
        
        if err := processItem(ctx, item); err != nil {
            return fmt.Errorf("failed to process item %d: %w", i, err)
        }
    }
    return nil
}

// ✅ GOOD - Context values for request-scoped data
type contextKey string

const (
    userIDKey contextKey = "user_id"
    traceIDKey contextKey = "trace_id"
)

func withUserID(ctx context.Context, userID string) context.Context {
    return context.WithValue(ctx, userIDKey, userID)
}

func getUserID(ctx context.Context) (string, bool) {
    userID, ok := ctx.Value(userIDKey).(string)
    return userID, ok
}
```

---

## 3. Goroutines and Concurrency {#goroutines}

### Concurrency Best Practices
- [ ] Always handle goroutine completion (use sync.WaitGroup or channels)
- [ ] Avoid goroutine leaks (ensure goroutines can exit)
- [ ] Use channels for communication between goroutines
- [ ] Protect shared state with mutexes or use channels
- [ ] Use `context.Context` for cancellation

**Examples:**
```go
// ❌ BAD - Goroutine leak
func fetchUsers() {
    for _, id := range userIDs {
        go func(id string) {
            user, _ := fetchUser(context.Background(), id)
            fmt.Println(user)
        }(id) // No way to wait for completion
    }
}

// ❌ BAD - Race condition
type Counter struct {
    count int
}

func (c *Counter) Increment() {
    c.count++ // Race condition without mutex
}

// ✅ GOOD - WaitGroup for goroutine synchronization
func fetchUsers(ctx context.Context, userIDs []string) ([]*User, error) {
    var wg sync.WaitGroup
    userChan := make(chan *User, len(userIDs))
    errChan := make(chan error, len(userIDs))
    
    for _, id := range userIDs {
        wg.Add(1)
        go func(id string) {
            defer wg.Done()
            
            user, err := fetchUser(ctx, id)
            if err != nil {
                errChan <- err
                return
            }
            userChan <- user
        }(id)
    }
    
    // Wait for all goroutines
    go func() {
        wg.Wait()
        close(userChan)
        close(errChan)
    }()
    
    // Collect results
    var users []*User
    for user := range userChan {
        users = append(users, user)
    }
    
    // Check for errors
    select {
    case err := <-errChan:
        return nil, err
    default:
        return users, nil
    }
}

// ✅ GOOD - Mutex for shared state
type Counter struct {
    mu    sync.Mutex
    count int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

func (c *Counter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}

// ✅ GOOD - Using errgroup for concurrent operations
import "golang.org/x/sync/errgroup"

func fetchUsersParallel(ctx context.Context, userIDs []string) ([]*User, error) {
    g, ctx := errgroup.WithContext(ctx)
    userChan := make(chan *User, len(userIDs))
    
    for _, id := range userIDs {
        id := id // Capture loop variable
        g.Go(func() error {
            user, err := fetchUser(ctx, id)
            if err != nil {
                return err
            }
            userChan <- user
            return nil
        })
    }
    
    // Wait for all goroutines
    go func() {
        g.Wait()
        close(userChan)
    }()
    
    // Wait for completion and check error
    if err := g.Wait(); err != nil {
        return nil, err
    }
    
    // Collect results
    var users []*User
    for user := range userChan {
        users = append(users, user)
    }
    
    return users, nil
}

// ✅ GOOD - Worker pool pattern
func processItemsWithWorkers(ctx context.Context, items []Item) error {
    const numWorkers = 10
    itemChan := make(chan Item, len(items))
    
    // Start workers
    g, ctx := errgroup.WithContext(ctx)
    for i := 0; i < numWorkers; i++ {
        g.Go(func() error {
            for item := range itemChan {
                if err := processItem(ctx, item); err != nil {
                    return err
                }
            }
            return nil
        })
    }
    
    // Send items to workers
    go func() {
        for _, item := range items {
            itemChan <- item
        }
        close(itemChan)
    }()
    
    return g.Wait()
}
```

---

## 4. Interfaces and Dependency Injection {#interfaces}

### Interface Best Practices
- [ ] Define interfaces at the point of use (consumer side)
- [ ] Keep interfaces small (1-3 methods)
- [ ] Accept interfaces, return structs
- [ ] Use dependency injection via constructor functions
- [ ] Define interfaces for testing and mocking

**Examples:**
```go
// ❌ BAD - Large interface
type UserService interface {
    CreateUser(user *User) error
    UpdateUser(user *User) error
    DeleteUser(id string) error
    GetUser(id string) (*User, error)
    ListUsers() ([]*User, error)
    SearchUsers(query string) ([]*User, error)
    ValidateUser(user *User) error
}

// ❌ BAD - Concrete dependency
type OrderService struct {
    userRepo *UserRepository // Concrete type
}

// ✅ GOOD - Small, focused interfaces
type UserGetter interface {
    GetUser(ctx context.Context, id string) (*User, error)
}

type UserCreator interface {
    CreateUser(ctx context.Context, user *User) error
}

type UserRepository interface {
    UserGetter
    UserCreator
    DeleteUser(ctx context.Context, id string) error
}

// ✅ GOOD - Accept interfaces, return structs
type OrderService struct {
    userGetter UserGetter
    emailer    Emailer
}

func NewOrderService(userGetter UserGetter, emailer Emailer) *OrderService {
    return &OrderService{
        userGetter: userGetter,
        emailer:    emailer,
    }
}

func (s *OrderService) CreateOrder(ctx context.Context, userID string, items []Item) (*Order, error) {
    user, err := s.userGetter.GetUser(ctx, userID)
    if err != nil {
        return nil, fmt.Errorf("failed to get user: %w", err)
    }
    
    order := &Order{
        ID:     generateID(),
        UserID: userID,
        Items:  items,
    }
    
    if err := s.emailer.SendOrderConfirmation(ctx, user.Email, order); err != nil {
        // Log but don't fail
        log.Printf("Failed to send email: %v", err)
    }
    
    return order, nil
}

// ✅ GOOD - Interface for testing
type Emailer interface {
    SendOrderConfirmation(ctx context.Context, email string, order *Order) error
}

// Mock implementation for testing
type mockEmailer struct {
    sendCalled bool
}

func (m *mockEmailer) SendOrderConfirmation(ctx context.Context, email string, order *Order) error {
    m.sendCalled = true
    return nil
}
```

---

## 5. Database and SQL {#database-sql}

### Database Best Practices
- [ ] Use prepared statements (prevents SQL injection)
- [ ] Handle `sql.ErrNoRows` explicitly
- [ ] Use transactions for multi-statement operations
- [ ] Close rows and statements properly
- [ ] Use `database/sql` with proper drivers

**Examples:**
```go
// ❌ BAD - SQL injection vulnerability
func findUserByEmail(email string) (*User, error) {
    query := fmt.Sprintf("SELECT * FROM users WHERE email = '%s'", email)
    row := db.QueryRow(query)
    // ...
}

// ❌ BAD - Not closing rows
func listUsers() ([]*User, error) {
    rows, err := db.Query("SELECT id, name, email FROM users")
    if err != nil {
        return nil, err
    }
    // Missing: defer rows.Close()
    
    var users []*User
    for rows.Next() {
        var user User
        if err := rows.Scan(&user.ID, &user.Name, &user.Email); err != nil {
            return nil, err
        }
        users = append(users, &user)
    }
    return users, nil
}

// ✅ GOOD - Prepared statement
func findUserByEmail(ctx context.Context, email string) (*User, error) {
    query := "SELECT id, name, email, created_at FROM users WHERE email = ?"
    
    var user User
    err := db.QueryRowContext(ctx, query, email).Scan(
        &user.ID,
        &user.Name,
        &user.Email,
        &user.CreatedAt,
    )
    
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, &UserNotFoundError{Email: email}
        }
        return nil, fmt.Errorf("failed to query user: %w", err)
    }
    
    return &user, nil
}

// ✅ GOOD - Proper rows handling
func listUsers(ctx context.Context) ([]*User, error) {
    query := "SELECT id, name, email, created_at FROM users ORDER BY created_at DESC"
    
    rows, err := db.QueryContext(ctx, query)
    if err != nil {
        return nil, fmt.Errorf("failed to query users: %w", err)
    }
    defer rows.Close()
    
    var users []*User
    for rows.Next() {
        var user User
        if err := rows.Scan(&user.ID, &user.Name, &user.Email, &user.CreatedAt); err != nil {
            return nil, fmt.Errorf("failed to scan user: %w", err)
        }
        users = append(users, &user)
    }
    
    // Check for errors during iteration
    if err := rows.Err(); err != nil {
        return nil, fmt.Errorf("error iterating rows: %w", err)
    }
    
    return users, nil
}

// ✅ GOOD - Transaction handling
func createUserWithProfile(ctx context.Context, user *User, profile *Profile) error {
    tx, err := db.BeginTx(ctx, nil)
    if err != nil {
        return fmt.Errorf("failed to begin transaction: %w", err)
    }
    defer tx.Rollback() // Rollback if not committed
    
    // Insert user
    result, err := tx.ExecContext(ctx,
        "INSERT INTO users (id, name, email) VALUES (?, ?, ?)",
        user.ID, user.Name, user.Email,
    )
    if err != nil {
        return fmt.Errorf("failed to insert user: %w", err)
    }
    
    // Insert profile
    _, err = tx.ExecContext(ctx,
        "INSERT INTO profiles (user_id, bio, avatar) VALUES (?, ?, ?)",
        user.ID, profile.Bio, profile.Avatar,
    )
    if err != nil {
        return fmt.Errorf("failed to insert profile: %w", err)
    }
    
    // Commit transaction
    if err := tx.Commit(); err != nil {
        return fmt.Errorf("failed to commit transaction: %w", err)
    }
    
    return nil
}
```

---

## 6. Struct and Method Design {#structs-methods}

### Struct Best Practices
- [ ] Use pointer receivers for methods that modify state
- [ ] Use value receivers for methods that don't modify state
- [ ] Keep structs focused (single responsibility)
- [ ] Use embedded structs for composition
- [ ] Export only necessary fields

**Examples:**
```go
// ❌ BAD - Exposing all fields
type User struct {
    ID             string
    Name           string
    Email          string
    HashedPassword string // Should not be exported
    Salt           string // Should not be exported
}

// ❌ BAD - Inconsistent receiver types
type Counter struct {
    count int
}

func (c Counter) Increment() {  // Value receiver
    c.count++ // Won't modify original
}

func (c *Counter) Value() int { // Pointer receiver
    return c.count
}

// ✅ GOOD - Proper encapsulation
type User struct {
    ID        string
    Name      string
    Email     string
    createdAt time.Time // Unexported
}

func (u *User) CreatedAt() time.Time {
    return u.createdAt
}

// ✅ GOOD - Consistent receiver types
type Counter struct {
    mu    sync.Mutex
    count int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

func (c *Counter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}

// ✅ GOOD - Constructor function
func NewUser(id, name, email string) *User {
    return &User{
        ID:        id,
        Name:      name,
        Email:     email,
        createdAt: time.Now(),
    }
}

// ✅ GOOD - Composition with embedded structs
type TimestampedEntity struct {
    CreatedAt time.Time
    UpdatedAt time.Time
}

type User struct {
    TimestampedEntity
    ID    string
    Name  string
    Email string
}

// ✅ GOOD - Builder pattern for complex construction
type UserBuilder struct {
    user *User
}

func NewUserBuilder() *UserBuilder {
    return &UserBuilder{
        user: &User{},
    }
}

func (b *UserBuilder) WithID(id string) *UserBuilder {
    b.user.ID = id
    return b
}

func (b *UserBuilder) WithName(name string) *UserBuilder {
    b.user.Name = name
    return b
}

func (b *UserBuilder) WithEmail(email string) *UserBuilder {
    b.user.Email = email
    return b
}

func (b *UserBuilder) Build() (*User, error) {
    if b.user.ID == "" {
        return nil, errors.New("user ID is required")
    }
    if b.user.Email == "" {
        return nil, errors.New("user email is required")
    }
    return b.user, nil
}
```

---

## 7. Testing {#testing}

### Testing Best Practices
- [ ] Use table-driven tests
- [ ] Test error cases
- [ ] Use subtests for organization
- [ ] Mock external dependencies
- [ ] Use test helpers

**Examples:**
```go
// ✅ GOOD - Table-driven tests
func TestValidateEmail(t *testing.T) {
    tests := []struct {
        name    string
        email   string
        wantErr bool
    }{
        {
            name:    "valid email",
            email:   "user@example.com",
            wantErr: false,
        },
        {
            name:    "missing @",
            email:   "userexample.com",
            wantErr: true,
        },
        {
            name:    "empty email",
            email:   "",
            wantErr: true,
        },
        {
            name:    "missing domain",
            email:   "user@",
            wantErr: true,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := validateEmail(tt.email)
            if (err != nil) != tt.wantErr {
                t.Errorf("validateEmail() error = %v, wantErr %v", err, tt.wantErr)
            }
        })
    }
}

// ✅ GOOD - Testing with mock
type mockUserGetter struct {
    user *User
    err  error
}

func (m *mockUserGetter) GetUser(ctx context.Context, id string) (*User, error) {
    return m.user, m.err
}

func TestOrderService_CreateOrder(t *testing.T) {
    ctx := context.Background()
    
    tests := []struct {
        name       string
        userGetter UserGetter
        wantErr    bool
    }{
        {
            name: "success",
            userGetter: &mockUserGetter{
                user: &User{ID: "1", Name: "John", Email: "john@example.com"},
                err:  nil,
            },
            wantErr: false,
        },
        {
            name: "user not found",
            userGetter: &mockUserGetter{
                user: nil,
                err:  &UserNotFoundError{UserID: "1"},
            },
            wantErr: true,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            service := NewOrderService(tt.userGetter, &mockEmailer{})
            
            order, err := service.CreateOrder(ctx, "1", []Item{})
            if (err != nil) != tt.wantErr {
                t.Errorf("CreateOrder() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            
            if !tt.wantErr && order == nil {
                t.Error("CreateOrder() returned nil order")
            }
        })
    }
}

// ✅ GOOD - Test helpers
func assertNoError(t *testing.T, err error) {
    t.Helper()
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
}

func assertEqual(t *testing.T, got, want interface{}) {
    t.Helper()
    if !reflect.DeepEqual(got, want) {
        t.Errorf("got %v, want %v", got, want)
    }
}
```

---

## Review Checklist Summary

Quick checklist for Go code reviews:

- [ ] **Errors**: Always check errors, wrap with context, use custom types
- [ ] **Context**: Pass as first parameter, use for timeout/cancellation
- [ ] **Goroutines**: No leaks, proper synchronization, avoid race conditions
- [ ] **Interfaces**: Small interfaces, dependency injection, consumer-side definition
- [ ] **Database**: Prepared statements, close resources, handle transactions
- [ ] **Structs**: Proper encapsulation, consistent receivers, composition
- [ ] **Testing**: Table-driven tests, mock dependencies, test error cases

---

## Tools for Code Quality

**Linters and Formatters:**
```bash
# Format code
go fmt ./...
gofmt -s -w .

# Imports
goimports -w .

# Linting
golangci-lint run

# Vet
go vet ./...

# Static analysis
staticcheck ./...
```

**golangci-lint configuration (.golangci.yml):**
```yaml
linters:
  enable:
    - gofmt
    - govet
    - errcheck
    - staticcheck
    - unused
    - gosimple
    - structcheck
    - varcheck
    - ineffassign
    - deadcode
    - typecheck
    - bodyclose
    - noctx
    - sqlclosecheck
```

---

## References

- [Effective Go](https://golang.org/doc/effective_go)
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)
- [Go Proverbs](https://go-proverbs.github.io/)
- [Standard Library Documentation](https://pkg.go.dev/std)
