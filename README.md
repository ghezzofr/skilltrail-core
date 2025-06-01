# SkillTrail Core

## 📋 Project Overview

SkillTrail Core is a platform for creating and managing educational trails composed of coding challenges. A **Trail** is a structured learning path containing a sequence of **Challenges**, where each challenge represents a specific exercise with validation, tips, and attachments. The system uses a serverless microservices architecture on AWS to ensure scalability and flexibility.

## 🏗️ Architecture

### Technology Stack
- **Cloud Provider**: Amazon Web Services (AWS)
- **Architecture**: Event-driven serverless microservices
- **Programming Language**: Go (Golang)
- **Infrastructure as Code**: AWS CDK (Cloud Development Kit)
- **Database**: DynamoDB (NoSQL)
- **API Gateway**: AWS API Gateway
- **Computing**: AWS Lambda
- **Messaging**: Amazon EventBridge / SQS
- **Authentication**: AWS Cognito (custom)
- **File Storage**: Amazon S3

### Planned Microservices

#### 1. **User Service**
- User registration via AWS Cognito User Pools
- Email verification handled by Cognito
- Pre-registration Lambda trigger for email whitelist validation
- User profile management with Cognito attributes
- JWT token validation and user context resolution

#### 2. **Trail Service**
- Educational trail creation and management
- CRUD operations on trails
- Trail metadata (title, description, difficulty)
- Trail states (draft, published, archived)
- Challenge sequencing within trails

#### 3. **Challenge Service**
- Management of coding challenges with detailed descriptions
- **Complete RESTful API** with JSON, XML, form-data support
- Downloadable attachment system for each challenge
- Timed tips management with point penalties
- Scoring and time limit system
- Input/output management for validation
- Dynamic input generation support
- Custom output validation functions
- Challenge metadata and prerequisites
- **Content negotiation** for different response formats

#### 4. **Progress Service**
- User progress tracking for trails
- Challenge completion states
- Scoring system with tip penalties
- Time spent tracking per challenge
- Attempt and submission history
- Timed tip request management
- Advancement and performance statistics
- Activity timeline

#### 5. **File Management Service**
- Challenge attachment upload/download management
- Secure input/output file storage
- Signed URL generation for downloads
- File type and size validation
- Automatic temporary file cleanup

#### 6. **Validation Service**
- Custom validation function execution
- Output comparison with expected files
- User feedback generation
- Secure sandbox execution system
- Validation result logging

#### 7. **Live Challenge Service**
- Live challenge events with multiple participants
- Only one active live challenge at a time globally
- Time limit based on sum of challenge times
- Real-time tracking of all participant progress
- Server-Sent Events (SSE) for live updates
- Dynamic leaderboard with scores and states
- Participant state management (active, completed, abandoned)

#### 8. **Notification Service**
- Registration confirmation email sending
- Challenge/trail completion notifications
- System communications

## 📊 Data Model

### Main Entities

#### User
```json
{
  "userId": "uuid",
  "email": "user@example.com",
  "emailVerified": true,
  "createdAt": "timestamp",
  "lastLoginAt": "timestamp",
  "status": "active|suspended"
}
```

#### Trail
```json
{
  "trailId": "uuid",
  "title": "Trail Name",
  "description": "Detailed description",
  "estimatedMinutes": 20,
  "challenges": ["challengeId1", "challengeId2"],
  "status": "draft|published|archived",
  "createdBy": "userId",
  "createdAt": "timestamp",
  "updatedAt": "timestamp"
}
```

#### Challenge
```json
{
  "challengeId": "uuid",
  "title": "Challenge Name",
  "description": "Text describing the challenge in detail",
  "totalPoints": 100,
  "timeLimit": 3600,
  "prerequisites": ["challengeId"],
  "attachments": [
    {
      "attachmentId": "uuid",
      "filename": "starter-code.zip",
      "url": "s3://bucket/files/...",
      "description": "Starting code"
    }
  ],
  "tips": [
    {
      "tipId": "uuid",
      "text": "Hint to solve the problem",
      "attachment": {
        "filename": "hint-diagram.png",
        "url": "s3://bucket/tips/..."
      },
      "availableAfterSeconds": 300,
      "pointsDeduction": 10
    }
  ],
  "inputs": {
    "type": "files|generator",
    "files": [
      {
        "filename": "input1.txt",
        "url": "s3://bucket/inputs/..."
      }
    ],
    "generator": {
      "functionArn": "arn:aws:lambda:...",
      "parameters": {"size": "medium", "complexity": "high"}
    }
  },
  "validation": {
    "type": "files|function",
    "expectedOutputs": [
      {
        "filename": "output1.txt",
        "url": "s3://bucket/outputs/..."
      }
    ],
    "validationFunction": {
      "functionArn": "arn:aws:lambda:...",
      "timeout": 30
    }
  },
  "status": "draft|published|archived",
  "createdBy": "userId",
  "createdAt": "timestamp"
}
```

#### LiveChallenge
```json
{
  "liveChallengeId": "uuid",
  "title": "Weekly Coding Challenge",
  "description": "Weekly programming challenge",
  "trailId": "uuid",
  "challenges": ["challengeId1", "challengeId2", "challengeId3"],
  "totalTimeLimit": 10800,
  "maxParticipants": 100,
  "status": "scheduled|active|completed|cancelled",
  "startTime": "timestamp",
  "endTime": "timestamp",
  "participants": [
    {
      "userId": "uuid",
      "joinedAt": "timestamp",
      "status": "active|completed|abandoned",
      "currentChallengeIndex": 1,
      "totalScore": 180,
      "timeSpent": 3600,
      "challengeCompletions": [
        {
          "challengeId": "uuid",
          "completedAt": "timestamp",
          "score": 85,
          "timeSpent": 1200,
          "tipsUsed": 1,
          "attempts": 2
        }
      ]
    }
  ],
  "createdBy": "userId",
  "createdAt": "timestamp"
}
```

#### UserProgress
```json
{
  "progressId": "uuid",
  "userId": "uuid",
  "trailId": "uuid",
  "startedAt": "timestamp",
  "completedAt": "timestamp|null",
  "currentChallengeIndex": 0,
  "challengeProgresses": [
    {
      "challengeId": "uuid",
      "status": "not_started|in_progress|completed|skipped",
      "startedAt": "timestamp",
      "completedAt": "timestamp|null",
      "attempts": 1,
      "currentScore": 85,
      "maxPossibleScore": 100,
      "timeSpent": 1800,
      "tipsUsed": [
        {
          "tipId": "uuid",
          "requestedAt": "timestamp",
          "pointsDeducted": 10
        }
      ],
      "submissionHistory": [
        {
          "submittedAt": "timestamp",
          "score": 75,
          "feedback": "Output mostly correct, edge case missing"
        }
      ]
    }
  ],
  "overallProgress": 45.5,
  "totalScore": 340,
  "maxPossibleTotalScore": 400
}
```

## 🔄 System Events

### Main Events
- `UserRegistered`: New user registered
- `EmailVerified`: User email verified
- `TrailCreated`: New trail created
- `TrailPublished`: Trail published
- `ChallengeCreated`: New challenge created
- `LiveChallengeCreated`: New live challenge created
- `LiveChallengeStarted`: Live challenge started
- `UserJoinedLiveChallenge`: User joined live challenge
- `UserStartedTrail`: User started a trail
- `ChallengeStarted`: User started a challenge
- `TipRequested`: User requested a hint
- `ChallengeSubmitted`: Solution submitted for validation
- `ChallengeCompleted`: Challenge completed successfully
- `TrailCompleted`: Trail completed
- `LiveChallengeCompleted`: Live challenge completed
- `FileUploaded`: Attachment file uploaded
- `ValidationCompleted`: Output validation completed

### Event Flows

#### User Registration
1. `UserRegistered` → Notification Service sends verification email
2. `EmailVerified` → User Service activates account

#### Trail Start
1. `UserStartedTrail` → Progress Service creates tracking
2. Progress Service emits `ChallengeAvailable`

#### Challenge Completion
1. `ChallengeCompleted` → Progress Service updates state
2. If trail completed → `TrailCompleted`

## 🛠️ API Endpoints

### User Service
```
# Cognito Integration (most auth handled by Cognito directly)
GET /users/profile
PUT /users/profile
POST /users/resend-verification
POST /users/forgot-password
POST /users/reset-password
GET /users/{userId}/context

# Direct Cognito endpoints (used by frontend):
# POST https://cognito-idp.{region}.amazonaws.com/
# - InitiateAuth (sign in)
# - SignUp (registration with email whitelist Lambda)
# - ConfirmSignUp (email verification)
# - ResendConfirmationCode
# - ForgotPassword
# - ConfirmForgotPassword
```

### Trail Service
```
GET /trails
POST /trails
GET /trails/{trailId}
PUT /trails/{trailId}
DELETE /trails/{trailId}
POST /trails/{trailId}/publish
```

### Challenge Service (RESTful)
```
# Challenge CRUD - Supports JSON, XML, form-data
GET    /challenges
POST   /challenges
GET    /challenges/{challengeId}
PUT    /challenges/{challengeId}
PATCH  /challenges/{challengeId}
DELETE /challenges/{challengeId}

# Challenge Attachments
GET    /challenges/{challengeId}/attachments
POST   /challenges/{challengeId}/attachments
GET    /challenges/{challengeId}/attachments/{attachmentId}
PUT    /challenges/{challengeId}/attachments/{attachmentId}
DELETE /challenges/{challengeId}/attachments/{attachmentId}

# Challenge Tips
GET    /challenges/{challengeId}/tips
POST   /challenges/{challengeId}/tips
GET    /challenges/{challengeId}/tips/{tipId}
PUT    /challenges/{challengeId}/tips/{tipId}
DELETE /challenges/{challengeId}/tips/{tipId}
POST   /challenges/{challengeId}/tips/{tipId}/request

# Challenge I/O
GET    /challenges/{challengeId}/inputs
PUT    /challenges/{challengeId}/inputs
GET    /challenges/{challengeId}/validation
PUT    /challenges/{challengeId}/validation
POST   /challenges/{challengeId}/validate

# Supported Content-Types for all operations:
# - application/json
# - application/xml
# - application/x-www-form-urlencoded
# - multipart/form-data (for attachments)
```

### File Management Service
```
POST /files/upload
GET /files/{fileId}/download
DELETE /files/{fileId}
GET /files/{fileId}/metadata
```

### Validation Service
```
POST /validate/submission
GET /validate/{validationId}/status
GET /validate/{validationId}/result
```

### Live Challenge Service
```
GET /live-challenges
POST /live-challenges
GET /live-challenges/{liveChallengeId}
PUT /live-challenges/{liveChallengeId}
DELETE /live-challenges/{liveChallengeId}
POST /live-challenges/{liveChallengeId}/start
POST /live-challenges/{liveChallengeId}/join
GET /live-challenges/{liveChallengeId}/participants
GET /live-challenges/{liveChallengeId}/leaderboard
GET /live-challenges/{liveChallengeId}/stream
```

### Progress Service
```
GET /users/{userId}/progress
POST /users/{userId}/trails/{trailId}/start
POST /users/{userId}/challenges/{challengeId}/start
PUT /users/{userId}/challenges/{challengeId}/submit
POST /users/{userId}/challenges/{challengeId}/tips/{tipId}/request
GET /users/{userId}/trails/{trailId}/progress
GET /users/{userId}/challenges/{challengeId}/progress
```

## 🌐 API Design and Formats

### Challenge Service - RESTful Design

The **Challenge Service** implements a fully RESTful API that supports different content formats to maximize compatibility and integration flexibility.

#### Supported Content Types

- **JSON** (`application/json`): Primary format for frontend interaction
- **XML** (`application/xml`): For legacy system integration
- **Form Data** (`application/x-www-form-urlencoded`): For traditional HTML forms
- **Multipart** (`multipart/form-data`): For attachment uploads

#### Content Negotiation

The service uses standard HTTP headers for content negotiation:

```http
# JSON Request
Content-Type: application/json
Accept: application/json

# XML Request
Content-Type: application/xml
Accept: application/xml

# Form data Request
Content-Type: application/x-www-form-urlencoded
Accept: application/json
```

#### Request Examples

**Create challenge with JSON:**
```http
POST /challenges
Content-Type: application/json
Accept: application/json

{
  "title": "Algorithm Challenge",
  "description": "Implement a sorting algorithm",
  "type": "coding",
  "totalPoints": 100,
  "timeLimit": 3600
}
```

**Create challenge with XML:**
```http
POST /challenges
Content-Type: application/xml
Accept: application/xml

<challenge>
  <title>Algorithm Challenge</title>
  <description>Implement a sorting algorithm</description>
  <type>coding</type>
  <totalPoints>100</totalPoints>
  <timeLimit>3600</timeLimit>
</challenge>
```

**Partial update with PATCH:**
```http
PATCH /challenges/{challengeId}
Content-Type: application/json

{
  "totalPoints": 120,
  "timeLimit": 4200
}
```

## 🔐 Security and Authentication

### AWS Cognito Integration
- **User Pools**: Complete user management with sign-up, sign-in, and profile management
- **Email Verification**: Built-in email verification with customizable templates
- **JWT Tokens**: Automatic JWT token generation and validation
- **Password Policies**: Configurable password complexity requirements
- **MFA Support**: Optional multi-factor authentication
- **Social Providers**: Future support for Google, GitHub, etc.
- **Custom Attributes**: Additional user metadata stored in Cognito
- **Pre-Registration Hooks**: Lambda triggers for email whitelist validation

### Authentication Flow
1. **User Registration**: POST to Cognito User Pool with email whitelist validation
2. **Email Verification**: Cognito sends verification email automatically
3. **Sign In**: Cognito validates credentials and returns JWT tokens
4. **Token Validation**: API Gateway validates JWT tokens automatically
5. **User Context**: Extract user information from validated JWT claims

### Authorization Levels
- **Public**: Read published trails and challenges
- **Authenticated**: Start trails, submit solutions, track progress
- **Admin**: Manage trails, challenges, and user accounts (future)

### Security Features
- **API Rate Limiting**: AWS API Gateway throttling
- **CORS Configuration**: Proper cross-origin resource sharing
- **Input Validation**: Request validation at API Gateway level
- **SQL Injection Protection**: DynamoDB NoSQL prevents SQL injection
- **Secure File Access**: S3 signed URLs with expiration

### Why AWS Cognito for SkillTrail Core

**AWS Cognito provides all the authentication features we need out-of-the-box:**

#### ✅ Features We Get for Free
- **User Registration & Sign-In**: Complete flows with security best practices
- **Email Verification**: Automatic email sending with customizable templates
- **Password Management**: Secure hashing, complexity rules, forgot password flows
- **JWT Token Management**: Automatic generation, validation, and refresh
- **Session Management**: Token expiration and refresh token handling
- **Security Features**: Rate limiting, account lockout, suspicious activity detection
- **Scalability**: Handles millions of users without infrastructure management

#### 🔧 Custom Integration Points
- **Email Whitelist**: Pre-registration Lambda trigger to validate allowed emails
- **Custom Attributes**: Store additional user metadata (preferences, roles)
- **User Context Service**: Extract user info from JWT tokens for our services
- **API Gateway Integration**: Automatic JWT validation for all protected endpoints

#### 🏗️ Architecture Benefits
- **Serverless**: No infrastructure to manage for authentication
- **Cost-Effective**: Pay only for active users
- **Security Compliance**: SOC 2, PCI DSS, ISO 27001 compliant
- **Integration**: Native integration with API Gateway and other AWS services
- **Monitoring**: Built-in CloudWatch metrics and logs

#### 📱 Frontend Integration
```javascript
// Frontend can directly interact with Cognito
import { Auth } from 'aws-amplify';

// Register user
await Auth.signUp({
  username: email,
  password: password,
  attributes: { email }
});

// Verify email
await Auth.confirmSignUp(email, verificationCode);

// Sign in
const user = await Auth.signIn(email, password);

// Get JWT token for API calls
const session = await Auth.currentSession();
const token = session.getIdToken().getJwtToken();
```

#### 🛡️ Email Whitelist Implementation
```go
// Lambda trigger for pre-registration validation
func PreSignUpHandler(ctx context.Context, event events.CognitoEventUserPoolsPreSignup) (events.CognitoEventUserPoolsPreSignup, error) {
    email := event.Request.UserAttributes["email"]
    
    if !isEmailWhitelisted(email) {
        return event, errors.New("email not in whitelist")
    }
    
    // Auto-confirm email if in whitelist
    event.Response.AutoConfirmUser = true
    event.Response.AutoVerifyEmail = true
    
    return event, nil
}
```

## 📁 Repository Structure

```
skilltrail-core/
├── docs/                          # Documentation
├── infrastructure/                # IaC (AWS CDK)
│   ├── environments/
│   ├── stacks/
│   └── constructs/
├── services/                      # Microservices (Go)
│   ├── user-service/
│   ├── trail-service/
│   ├── challenge-service/
│   ├── progress-service/
│   ├── file-management-service/
│   ├── validation-service/
│   ├── live-challenge-service/
│   └── notification-service/
├── shared/                        # Shared code
│   ├── events/
│   ├── schemas/
│   └── utils/
├── tests/                         # Integration tests
└── scripts/                      # Deployment scripts
```

## 🚀 Development Roadmap

### Phase 1: MVP Core
- [ ] User Service (registration, authentication)
- [ ] Trail Service (CRUD trails)
- [ ] Challenge Service (CRUD challenges with attachments)
- [ ] File Management Service (upload/download)
- [ ] Infrastructure setup (DynamoDB, S3, API Gateway)

### Phase 2: Progress & Validation
- [ ] Progress Service with advanced scoring
- [ ] Validation Service for outputs
- [ ] Timed tips system
- [ ] Event-driven architecture
- [ ] Notification Service

### Phase 3: Advanced Features
- [ ] Dynamic input generation
- [ ] Custom validation functions
- [ ] Detailed analytics and reporting
- [ ] Sandbox execution system
- [ ] External platform integrations

## 🧪 Testing Strategy

### Test Levels
- **Unit Tests**: Each microservice
- **Integration Tests**: API endpoints
- **Contract Tests**: Events between services
- **E2E Tests**: Complete user flows

### Test Environment
- **Local**: LocalStack for AWS services
- **Staging**: Dedicated AWS environment
- **Production**: Blue/Green deployment

## 📈 Monitoring and Observability

### Key Metrics
- API endpoint latency
- Email verification success rate
- Trail/challenge completion rate
- Errors per microservice

### AWS Tools
- **CloudWatch**: Logs and metrics
- **X-Ray**: Distributed tracing
- **CloudTrail**: Audit logs

## 🔧 Development Setup

### Prerequisites
- Go 1.24.2
- AWS CLI configured
- AWS CDK v2
- Docker (for local testing)

### Quick Start Commands
```bash
# Clone repository
git clone <repo-url>
cd skilltrail-core

# Setup infrastructure
cd infrastructure
npm install
cdk deploy

# Deploy services
./scripts/deploy-all.sh

# Run tests
go test ./...
```

### Live Challenge - Server-Sent Events

The **Live Challenge Service** uses Server-Sent Events (SSE) to provide real-time updates on participant progress during an active challenge.

#### SSE Endpoint
```
GET /live-challenges/{liveChallengeId}/stream
Accept: text/event-stream
```

#### SSE Events Transmitted

**Participant joins:**
```
event: participant-joined
data: {
  "userId": "uuid",
  "username": "john_doe",
  "joinedAt": "timestamp"
}
```

**Progress update:**
```
event: participant-progress
data: {
  "userId": "uuid",
  "currentChallengeIndex": 2,
  "challengeStatus": "completed",
  "score": 95,
  "timeSpent": 1500,
  "tipsUsed": 0
}
```

**Leaderboard update:**
```
event: leaderboard-update
data: {
  "rankings": [
    {
      "userId": "uuid",
      "username": "alice",
      "totalScore": 280,
      "timeSpent": 4200,
      "challengesCompleted": 3,
      "status": "active"
    }
  ]
}
```

**Challenge completed:**
```
event: challenge-completed
data: {
  "liveChallengeId": "uuid",
  "completedAt": "timestamp",
  "winner": {
    "userId": "uuid",
    "username": "winner",
    "finalScore": 300
  },
  "totalParticipants": 25
}
```

#### Live Challenge Rules

- **Single active challenge**: Only one LiveChallenge can be in "active" state at a time
- **Cumulative time limit**: Total time is the sum of `timeLimit` of component challenges
- **Real-time updates**: Every participant action is broadcast to all connected clients
- **Participant states**: `active`, `completed`, `abandoned`
- **Disconnection handling**: Clients can reconnect and receive current state

## 🏛️ Clean Architecture

### Architecture Principles

The project follows **Clean Architecture** principles to ensure maintainability, testability, and independence from external frameworks. Each microservice is structured with clear separation of concerns:

### Layer Structure

#### 1. **Entities Layer**
Core business objects with business rules and logic independent of any external concerns.

```
entities/
├── user.go
├── trail.go
├── challenge.go
├── progress.go
└── live_challenge.go
```

#### 2. **Domain Layer**
Business logic and domain services that orchestrate entities and define business rules.

```
domain/
├── services/
│   ├── user_service.go
│   ├── trail_service.go
│   ├── challenge_service.go
│   └── progress_service.go
├── repositories/
│   ├── user_repository.go
│   ├── trail_repository.go
│   ├── challenge_repository.go
│   └── progress_repository.go
└── events/
    ├── user_events.go
    ├── trail_events.go
    └── challenge_events.go
```

#### 3. **Infrastructure Layer**
External concerns like databases, APIs, file systems, and third-party services.

```
infrastructure/
├── persistence/
│   ├── dynamodb/
│   │   ├── user_repository_impl.go
│   │   ├── trail_repository_impl.go
│   │   └── challenge_repository_impl.go
│   └── s3/
│       └── file_storage_impl.go
├── messaging/
│   ├── eventbridge/
│   │   └── event_publisher_impl.go
│   └── sqs/
│       └── queue_consumer_impl.go
└── external/
    ├── cognito/
    │   └── auth_service_impl.go
    └── ses/
        └── email_service_impl.go
```

#### 4. **Interface/Presentation Layer**
HTTP handlers, Lambda functions, and external interfaces.

```
interfaces/
├── http/
│   ├── handlers/
│   │   ├── user_handler.go
│   │   ├── trail_handler.go
│   │   └── challenge_handler.go
│   ├── middleware/
│   │   ├── auth_middleware.go
│   │   └── validation_middleware.go
│   └── dto/
│       ├── user_dto.go
│       ├── trail_dto.go
│       └── challenge_dto.go
└── lambda/
    ├── user_lambda.go
    ├── trail_lambda.go
    └── challenge_lambda.go
```

### Microservice Structure Example

Each microservice follows this standard structure:

```
challenge-service/
├── cmd/
│   └── main.go                    # Entry point
├── entities/
│   ├── challenge.go               # Core business entity
│   ├── attachment.go
│   └── tip.go
├── domain/
│   ├── services/
│   │   └── challenge_service.go   # Business logic
│   ├── repositories/
│   │   └── challenge_repository.go # Repository interface
│   └── events/
│       └── challenge_events.go    # Domain events
├── infrastructure/
│   ├── persistence/
│   │   └── dynamodb_challenge_repository.go # Repository implementation
│   ├── storage/
│   │   └── s3_file_storage.go
│   └── messaging/
│       └── eventbridge_publisher.go
├── interfaces/
│   ├── http/
│   │   ├── handlers/
│   │   │   └── challenge_handler.go # HTTP handlers
│   │   ├── middleware/
│   │   └── dto/
│   │       └── challenge_dto.go   # Data transfer objects
│   └── lambda/
│       └── challenge_lambda.go    # Lambda wrapper
├── internal/
│   ├── config/
│   │   └── config.go              # Configuration
│   └── utils/
│       └── validator.go           # Utilities
├── tests/
│   ├── unit/
│   ├── integration/
│   └── mocks/
├── go.mod
└── go.sum
```

### Repository Pattern Implementation

#### Repository Interface (Domain Layer)
```go
// domain/repositories/challenge_repository.go
type ChallengeRepository interface {
    Create(ctx context.Context, challenge *entities.Challenge) error
    GetByID(ctx context.Context, id string) (*entities.Challenge, error)
    Update(ctx context.Context, challenge *entities.Challenge) error
    Delete(ctx context.Context, id string) error
    ListByTrail(ctx context.Context, trailID string) ([]*entities.Challenge, error)
    ListPublished(ctx context.Context, limit int, offset int) ([]*entities.Challenge, error)
}
```

#### Repository Implementation (Infrastructure Layer)
```go
// infrastructure/persistence/dynamodb_challenge_repository.go
type DynamoDBChallengeRepository struct {
    client    dynamodbiface.DynamoDBAPI
    tableName string
}

func (r *DynamoDBChallengeRepository) Create(ctx context.Context, challenge *entities.Challenge) error {
    // DynamoDB implementation
}

func (r *DynamoDBChallengeRepository) GetByID(ctx context.Context, id string) (*entities.Challenge, error) {
    // DynamoDB implementation
}
```

### Dependency Injection

Each microservice uses dependency injection to maintain loose coupling:

```go
// cmd/main.go
func main() {
    // Infrastructure setup
    dynamoClient := dynamodb.New(session.Must(session.NewSession()))
    
    // Repository implementations
    challengeRepo := persistence.NewDynamoDBChallengeRepository(dynamoClient, "challenges")
    
    // Domain services
    challengeService := services.NewChallengeService(challengeRepo)
    
    // HTTP handlers
    challengeHandler := handlers.NewChallengeHandler(challengeService)
    
    // Setup routes and start server
    setupRoutes(challengeHandler)
}
```

### Benefits of This Architecture

- **Testability**: Easy to mock repositories and test business logic
- **Maintainability**: Clear separation of concerns
- **Flexibility**: Easy to swap implementations (DynamoDB → PostgreSQL)
- **Independence**: Business logic independent of frameworks
- **Scalability**: Each layer can evolve independently