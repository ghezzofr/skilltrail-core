---
applyTo: '**/*.go'
---
# GitHub Copilot Instructions - Go Design Patterns & Best Practices

## 🌟 General Guidelines

- Use always english for naming variables, functions, and comments.
- Follow idiomatic Go conventions (https://go.dev/doc/effective_go).
- Use named functions over long anonymous ones.
- Organize logic into small, composable functions.
- Prefer interfaces for dependencies to enable mocking and testing.
- Avoid unnecessary abstraction; keep things simple and readable.

## 🎯 Patterns to Follow
- Use **Clean Architecture** and **Repository Pattern**.
- Use custom error types for wrapping and handling business logic errors.
- Logging should be handled via `log/slog`.
- Use dependency injection via constructors (avoid global state).
- Keep `main.go` minimal—delegate to `internal`.

## 📚 Documentation Guidelines

### Documentation Requirements
- Every exported name (package, const, func, type, var) must have a doc comment immediately above the declaration, with no blank lines in between.
- Package comments start with "Package name ..." and explain the general purpose of the package.
- Add one or more examples of the usage of exported functions in the doc comments. In the example, if an error type is returned use the `errors.Is` function to check the error type.
- Always explain the return values of functions, including errors. In case an error type is returned, explain the error in the comment.
- Command (main) comments start with the program name and describe the behavior of the command.

### Documentation Style
- Type comments explain what the type represents and, if necessary, the properties of its instances (e.g., thread safety, useful zero value).
- Function comments explain what the function returns or what it does (for side effects). Use complete sentences and refer to parameters by name.
- For functions that return bool, use "reports whether ...".
- For grouped constants and variables, a single comment can introduce the group; for single constants/variables, use a complete comment.
- Use complete sentences, without unnecessary abbreviations or bullet points.

### Documentation Format
- Comments support paragraphs, headings (with #), lists (with - or numbers), code blocks (indented), links, and doc links ([Name] or [pkg.Name]).
- Avoid implementation details in doc comments; focus on what the API does.
- Special notes (TODO, BUG, Deprecated) follow the syntax MARKER(uid): body or "Deprecated: ...".

### Documentation Examples
```go
// Package users provides functionality for managing user accounts and authentication.
// It implements user registration, email verification, and profile management.
package users

// ErrUserNotFound indicates that a user with the specified identifier was not found.
var ErrUserNotFound = errors.New("user not found")

// User represents a platform user with email-based authentication.
// User instances are safe for concurrent access through repository methods.
type User struct {
    ID            string    `json:"id"`
    Email         string    `json:"email"`
    EmailVerified bool      `json:"emailVerified"`
    CreatedAt     time.Time `json:"createdAt"`
}

// UserService provides business logic for user management operations.
// All methods are safe for concurrent use.
type UserService struct {
    repo UserRepository
    log  *slog.Logger
}

// NewUserService creates a new UserService with the provided repository and logger.
// The repository must not be nil.
//
// Example:
//
//	repo := persistence.NewDynamoDBUserRepository(client, "users")
//	service := NewUserService(repo, slog.Default())
func NewUserService(repo UserRepository, logger *slog.Logger) *UserService {
    return &UserService{
        repo: repo,
        log:  logger,
    }
}

// CreateUser creates a new user account with email verification required.
// It returns the created user or an error if validation fails or the email already exists.
// Returns ErrEmailAlreadyExists if a user with the same email already exists.
//
// Example:
//
//	user := &User{Email: "user@example.com"}
//	created, err := service.CreateUser(ctx, user)
//	if errors.Is(err, ErrEmailAlreadyExists) {
//	    // Handle duplicate email
//	}
func (s *UserService) CreateUser(ctx context.Context, user *User) (*User, error) {
    // Implementation...
}

// IsEmailVerified reports whether the user's email address has been verified.
func (u *User) IsEmailVerified() bool {
    return u.EmailVerified
}
```

## 🏛️ Architecture Guidelines

### Clean Architecture Implementation
- Follow **Clean Architecture** principles with clear layer separation
- Implement **Repository Pattern** with interfaces in domain layer
- Use **Dependency Injection** for loose coupling
- Separate concerns across 4 layers: Entities, Domain, Infrastructure, Interface

### Layer Structure Rules
```
entities/     # Core business objects (no external dependencies)
domain/       # Business logic, repository interfaces, events
infrastructure/ # External integrations (DynamoDB, S3, EventBridge)
interfaces/   # HTTP handlers, Lambda wrappers, DTOs
```

## 📋 Go Best Practices

### Code Organization
- Use **standard Go project layout** (`cmd/`, `internal/`, `pkg/`)
- Group related functionality in packages
- Keep packages focused and cohesive
- Use meaningful package names (avoid `utils`, `common`)

### Error Handling
```go
// Always handle errors explicitly
if err != nil {
    return fmt.Errorf("failed to create user: %w", err)
}

// Use custom error types for domain errors
type ValidationError struct {
    Field   string
    Message string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("validation failed for %s: %s", e.Field, e.Message)
}
```

### Interface Design
```go
// Keep interfaces small and focused
type UserRepository interface {
    Create(ctx context.Context, user *entities.User) error
    GetByID(ctx context.Context, id string) (*entities.User, error)
    Update(ctx context.Context, user *entities.User) error
    Delete(ctx context.Context, id string) error
}

// Accept interfaces, return concrete types
func NewUserService(repo UserRepository) *UserService {
    return &UserService{repo: repo}
}
```

### Context Usage
```go
// Always pass context as first parameter
func (s *UserService) CreateUser(ctx context.Context, user *entities.User) error {
    // Use context for cancellation and timeouts
    return s.repo.Create(ctx, user)
}
```

### Struct Design
```go
// Use struct embedding for composition
type BaseEntity struct {
    ID        string    `json:"id" dynamodbav:"id"`
    CreatedAt time.Time `json:"createdAt" dynamodbav:"created_at"`
    UpdatedAt time.Time `json:"updatedAt" dynamodbav:"updated_at"`
}

type User struct {
    BaseEntity
    Email         string `json:"email" dynamodbav:"email"`
    EmailVerified bool   `json:"emailVerified" dynamodbav:"email_verified"`
}
```

## 🔧 Code Style Guidelines

### Naming Conventions
- Use **PascalCase** for exported functions, types, constants
- Use **camelCase** for unexported functions, variables
- Use **SCREAMING_SNAKE_CASE** for constants
- Use descriptive names (avoid abbreviations)

### Function Design
```go
// Keep functions small and focused (single responsibility)
// Use early returns to reduce nesting
func ValidateUser(user *entities.User) error {
    if user == nil {
        return ErrUserNil
    }
    
    if user.Email == "" {
        return ValidationError{Field: "email", Message: "required"}
    }
    
    if !isValidEmail(user.Email) {
        return ValidationError{Field: "email", Message: "invalid format"}
    }
    
    return nil
}
```

### Concurrency Patterns
```go
// Use channels for communication
// Use sync package for synchronization
// Always handle goroutine cleanup

func (s *LiveChallengeService) StartChallenge(ctx context.Context, challengeID string) error {
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()
    
    go s.broadcastUpdates(ctx, challengeID)
    go s.monitorTimeout(ctx, challengeID)
    
    return nil
}
```

## 📦 AWS Integration Patterns

### DynamoDB Repository Implementation
```go
type DynamoDBUserRepository struct {
    client    dynamodbiface.DynamoDBAPI
    tableName string
}

func (r *DynamoDBUserRepository) Create(ctx context.Context, user *entities.User) error {
    item, err := dynamodbattribute.MarshalMap(user)
    if err != nil {
        return fmt.Errorf("failed to marshal user: %w", err)
    }
    
    input := &dynamodb.PutItemInput{
        TableName: aws.String(r.tableName),
        Item:      item,
    }
    
    _, err = r.client.PutItemWithContext(ctx, input)
    if err != nil {
        return fmt.Errorf("failed to put item: %w", err)
    }
    
    return nil
}
```

### Lambda Function Structure
```go
// Use adapter pattern for Lambda integration
func main() {
    lambda.Start(handler)
}

func handler(ctx context.Context, request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
    // Setup dependencies
    userRepo := persistence.NewDynamoDBUserRepository(dynamoClient, tableName)
    userService := services.NewUserService(userRepo)
    userHandler := handlers.NewUserHandler(userService)
    
    // Route request
    return userHandler.Handle(ctx, request)
}
```

## 🧪 Testing Guidelines

### Unit Testing
```go
// Use table-driven tests
func TestUserService_CreateUser(t *testing.T) {
    tests := []struct {
        name    string
        user    *entities.User
        wantErr bool
    }{
        {
            name:    "valid user",
            user:    &entities.User{Email: "test@example.com"},
            wantErr: false,
        },
        {
            name:    "invalid email",
            user:    &entities.User{Email: "invalid"},
            wantErr: true,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Test implementation
        })
    }
}
```

### Mock Generation
```go
//go:generate mockgen -source=user_repository.go -destination=mocks/user_repository_mock.go
type UserRepository interface {
    Create(ctx context.Context, user *entities.User) error
}
```

## 🔐 Security Best Practices

### Input Validation
```go
// Validate all inputs at the boundary
func (h *UserHandler) CreateUser(ctx context.Context, req CreateUserRequest) error {
    if err := h.validator.Validate(req); err != nil {
        return fmt.Errorf("validation failed: %w", err)
    }
    
    user := req.ToEntity()
    return h.service.CreateUser(ctx, user)
}
```

### Error Handling Security
```go
// Don't expose internal errors to clients
func (h *UserHandler) handleError(err error) events.APIGatewayProxyResponse {
    var validationErr ValidationError
    if errors.As(err, &validationErr) {
        return response.BadRequest(validationErr.Error())
    }
    
    // Log actual error, return generic message
    log.Printf("internal error: %v", err)
    return response.InternalServerError("internal server error")
}
```

## 📊 Performance Guidelines

### Memory Management
- Use object pooling for frequently allocated objects
- Avoid memory leaks with proper cleanup
- Use `sync.Pool` for reusable objects

### Database Optimization
- Use batch operations when possible
- Implement proper pagination
- Use projection to limit returned fields

### Caching Strategies
- Implement repository-level caching
- Use Redis for distributed caching
- Cache frequently accessed data

## 🚀 Deployment Considerations

### Configuration Management
```go
type Config struct {
    DatabaseURL string `env:"DATABASE_URL" required:"true"`
    Port        int    `env:"PORT" default:"8080"`
    LogLevel    string `env:"LOG_LEVEL" default:"info"`
}

func LoadConfig() (*Config, error) {
    var cfg Config
    if err := env.Parse(&cfg); err != nil {
        return nil, fmt.Errorf("failed to parse config: %w", err)
    }
    return &cfg, nil
}
```

### Graceful Shutdown
```go
func main() {
    server := &http.Server{Addr: ":8080", Handler: router}
    
    go func() {
        if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatalf("server failed: %v", err)
        }
    }()
    
    // Wait for interrupt signal
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    
    // Graceful shutdown
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    if err := server.Shutdown(ctx); err != nil {
        log.Fatalf("server shutdown failed: %v", err)
    }
}
```