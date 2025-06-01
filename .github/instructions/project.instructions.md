# GitHub Copilot Instructions - SkillTrail Core Project Context

## 📋 Project Overview

**SkillTrail Core** is a serverless microservices platform for creating and managing educational trails composed of coding challenges. The system uses AWS services and follows event-driven architecture patterns.

### Core Concepts
- **Trail**: A structured learning path containing a sequence of challenges
- **Challenge**: A specific coding exercise with validation, tips, and attachments
- **Live Challenge**: Real-time competitive events with multiple participants
- **Progress Tracking**: User advancement through trails with scoring and analytics

## 🏗️ System Architecture

### Technology Stack
- **Language**: Go 1.24.2
- **Cloud**: AWS (Lambda, DynamoDB, S3, EventBridge, API Gateway)
- **Infrastructure**: AWS CDK (Cloud Development Kit)
- **Architecture**: Event-driven serverless microservices
- **Authentication**: Custom AWS Cognito integration

### Microservices Structure
```
1. User Service        - Registration, authentication, profiles
2. Trail Service       - Trail CRUD, publishing, management
3. Challenge Service   - Challenge CRUD with RESTful API (JSON/XML/form-data)
4. Progress Service    - User progress tracking, scoring
5. File Management     - S3 attachments, signed URLs
6. Validation Service  - Output validation, custom functions
7. Live Challenge      - Real-time events with SSE updates
8. Notification        - Email confirmations, system alerts
```

## 📊 Domain Entities

### Primary Entities
- **User**: Email-based registration with verification codes
- **Trail**: Collection of challenges with metadata and states
- **Challenge**: Rich entity with tips, attachments, inputs, validation
- **UserProgress**: Tracking progression through trails with scoring
- **LiveChallenge**: Competitive events with participant management

### Key Relationships
- Trail contains multiple Challenges (ordered sequence)
- User has Progress for each started Trail
- Challenge can have multiple Tips (time-gated with penalties)
- Challenge has Attachments (downloadable files)
- LiveChallenge references a Trail and tracks Participants

## 🔄 Event-Driven Patterns

### Core Events
```
UserRegistered → EmailVerified → UserActivated
TrailCreated → TrailPublished → UserCanStart
ChallengeStarted → ChallengeSubmitted → ChallengeCompleted
TipRequested → PointsDeducted → ProgressUpdated
LiveChallengeStarted → ParticipantJoined → RealTimeUpdates
```

### Event Flow Examples
- User registration triggers email verification workflow
- Challenge completion updates progress and may complete trail
- Live challenge actions broadcast to all participants via SSE

## 🎯 Business Rules

### Challenge System
- Each challenge has time limits and point values
- Tips are available after time delays with point penalties
- Multiple submission attempts allowed with history tracking
- Custom validation functions for output verification

### Live Challenge Constraints
- Only ONE active live challenge globally at any time
- Time limit is cumulative of all component challenges
- Real-time leaderboard with participant status tracking
- SSE streams for live updates (participant-joined, progress, leaderboard)

### Progress Tracking
- Scoring system with tip penalties and time bonuses
- Trail completion requires all challenges in sequence
- Detailed analytics on attempts, time spent, tips used

## 🔐 Security Model

### Authentication Flow
- Email whitelist for registration eligibility
- Email verification with time-limited codes
- JWT tokens for stateless authentication
- API rate limiting and abuse protection

### Authorization Levels
- **Public**: Read published trails and challenges
- **Authenticated**: Start trails, submit solutions, track progress
- **Creator**: Create/modify trails and challenges (future feature)

## 🌐 API Design Patterns

### RESTful Implementation
The Challenge Service exemplifies full RESTful design:
- Complete CRUD operations (GET, POST, PUT, PATCH, DELETE)
- Content negotiation (JSON, XML, form-data, multipart)
- Nested resources (attachments, tips, inputs, validation)
- Proper HTTP status codes and error handling

### Content Type Support
- **JSON**: Primary format for modern frontends
- **XML**: Legacy system integration
- **Form Data**: Traditional HTML forms
- **Multipart**: File upload handling

## 🧪 Testing Strategy

### Test Pyramid Approach
- **Unit Tests**: Domain logic and business rules
- **Integration Tests**: Repository implementations and external services
- **Contract Tests**: Event schemas and API contracts
- **E2E Tests**: Complete user journeys through trails

### Mock Strategy
- Mock all external dependencies (DynamoDB, S3, EventBridge)
- Use interfaces for easy test double injection
- LocalStack for local AWS service simulation

## 📈 Monitoring and Observability

### Key Metrics to Track
- Trail completion rates and user engagement
- Challenge success rates and common failure points
- Live challenge participation and performance
- System performance (latency, errors, throughput)

### AWS Observability Tools
- CloudWatch for logs, metrics, and dashboards
- X-Ray for distributed tracing across microservices
- CloudTrail for audit logs and security monitoring

## 🚀 Development Workflow

### Clean Architecture Implementation
Each microservice follows the same 4-layer structure:
1. **Entities**: Core business objects (no dependencies)
2. **Domain**: Business logic, repository interfaces, events
3. **Infrastructure**: AWS integrations, external services
4. **Interfaces**: HTTP handlers, Lambda wrappers, DTOs

### Repository Pattern Usage
- Domain layer defines repository interfaces
- Infrastructure layer provides concrete implementations
- Easy to swap implementations (DynamoDB ↔ PostgreSQL)
- Facilitates unit testing with mocks

### Dependency Injection Pattern
- Constructor injection for all dependencies
- Interface-based design for testability
- Configuration management through environment variables

## 🔄 Common Implementation Patterns

### Event Publishing
```go
// Domain events are published after successful operations
if err := s.repo.CreateTrail(ctx, trail); err != nil {
    return err
}
s.eventPublisher.Publish(ctx, events.TrailCreated{TrailID: trail.ID})
```

### Error Handling
```go
// Wrap errors with context, don't expose internals
if err := s.validateChallenge(challenge); err != nil {
    return fmt.Errorf("challenge validation failed: %w", err)
}
```

### AWS Service Integration
```go
// Use AWS SDK interfaces for mockability
type S3Client interface {
    PutObject(input *s3.PutObjectInput) (*s3.PutObjectOutput, error)
    GetObject(input *s3.GetObjectInput) (*s3.GetObjectOutput, error)
}
```

## 📝 Code Generation Guidelines

When generating code for SkillTrail Core:
1. Follow Clean Architecture layer separation
2. Implement proper error handling with wrapped errors
3. Use context.Context for all operations
4. Include appropriate struct tags for JSON and DynamoDB
5. Generate corresponding tests with table-driven patterns
6. Implement proper logging and observability
7. Follow Go naming conventions and best practices
8. Include proper documentation and comments

## 🎯 Domain-Specific Considerations

### Challenge Validation
- Support both file-based and function-based validation
- Handle custom Lambda function execution for validation
- Provide detailed feedback on validation failures

### File Management
- Generate signed URLs for secure file access
- Implement proper file type and size validation
- Handle temporary file cleanup and lifecycle management

### Real-Time Features
- Use Server-Sent Events for live challenge updates
- Implement proper connection management and reconnection
- Handle participant state synchronization

### Scoring Algorithm
- Calculate points with time bonuses and tip penalties
- Track detailed submission history and analytics
- Support different challenge types (coding, quiz, project)