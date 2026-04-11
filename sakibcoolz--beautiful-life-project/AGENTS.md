# GitHub Copilot Instructions for Beautiful Life Project

## Project Overview

Beautiful Life is a comprehensive digital health platform designed to support cancer patients through their journey. This is a **monorepo** containing both frontend (Next.js 14) and backend (Golang) applications with shared packages.

## Core Business Logic & Context

### 🎯 Platform Purpose
A digital health platform that empowers cancer patients through:
1. **Community-driven emotional support** with specialized groups
2. **AI-assisted care reminders** for medications and appointments
3. **Secure medical record management** with sharing capabilities
4. **Real-time communication** via chat and video calling
5. **Healthcare provider integration** for professional support

### 👥 User Types & Roles

#### 1. Patient (Primary User)
- Can join multiple communities based on cancer type, treatment stage, interests
- Manages personal medical records with encryption
- Sets up medication and health reminders
- Participates in community posts, chat, and video sessions
- Can grant access to caregivers and healthcare providers
- **Subscription Tiers**: Free (limited features) | Premium (₹199/month)

#### 2. Healthcare Provider (Doctor/Nurse)
- Requires professional verification (license number, hospital affiliation)
- Can view patient records **only with explicit patient permission**
- Provides expert guidance in communities
- Schedules telehealth appointments
- Monitors patient progress and adherence

#### 3. Caregiver/Family Member
- Connected to specific patients via invitation
- Access levels: view_only | manage_reminders | full_access
- Receives escalation alerts for missed critical reminders
- Can help organize medical records and appointments

#### 4. Community Admin/Moderator
- Creates and manages specialized support communities
- Moderates content and ensures community guidelines
- Schedules group video sessions and events
- Cannot access individual medical records

## 🏗️ Architecture & Technology

### Frontend (Next.js 14)
- **Server-Side Rendering (SSR)** for performance and SEO
- **API Routes** act as middleware between browser and Golang backend
- **Authentication**: JWT tokens stored in HTTP-only cookies (XSS protection)
- **State Management**: React Context + SWR for data fetching
- **Styling**: Tailwind CSS with custom design system
- **Location**: `apps/frontend/`

### Backend (Golang + Gin)
- **Clean Architecture**: Domain → Application → Interface → Infrastructure layers
- **Database**: PostgreSQL with GORM (domain entities → database models)
- **Authentication**: JWT with RS256 signing, middleware for route protection
- **File Storage**: AWS S3 with client-side encryption before upload
- **Real-time**: WebSocket for chat, Server-Sent Events for notifications
- **Location**: `apps/backend/`

### Communication Pattern
```
Browser → Next.js API Routes → Golang Backend → PostgreSQL/Redis
                ↓ (HTTP-only cookies)
         JWT Token Management
```

## 🔐 Security & Compliance

### Critical Security Requirements
1. **HIPAA Compliance**: All medical data encrypted at rest and in transit
2. **Access Control**: Role-based permissions with patient consent model
3. **Audit Logging**: Track all access to sensitive medical records
4. **Data Privacy**: User controls exactly what data is shared with whom
5. **Session Management**: 15-minute timeout for inactive sessions
6. **Encryption**: AES-256 for data at rest, TLS 1.3 for data in transit

### Authentication Flow
1. User submits credentials to Next.js `/api/auth/login`
2. Next.js calls Golang `/api/v1/auth/login` server-side
3. Golang validates credentials and generates JWT token
4. Next.js sets HTTP-only cookie with token
5. Subsequent requests include cookie, validated by middleware

## 🧩 Key Features & Business Rules

### 1. Community Management
- **Community Types**: public | private | expert_led | regional
- **Categories**: cancer_type | treatment_stage | age_group | interest | location
- **Moderation Levels**: strict | moderate | relaxed
- **Business Rule**: Free users can join up to 3 communities, Premium unlimited
- **Content Moderation**: AI-assisted flagging + human moderator review

### 2. Medical Records Management
- **Record Types**: prescription | lab_report | scan_result | doctor_note | insurance_doc
- **Upload Process**: Validate → Encrypt → Upload to S3 → Store metadata in DB
- **Sharing Logic**: 
  - Patient creates temporary sharing link with expiration
  - Granular permissions: view | download | edit
  - Activity logging for compliance
- **OCR Integration**: Extract text from images for searchability

### 3. AI-Powered Reminders
- **Reminder Types**: medication | appointment | water | food | exercise | custom
- **Scheduling**: once | daily | weekly | monthly | custom (cron-like patterns)
- **Smart Features**:
  - Learn optimal timing from user behavior
  - Avoid notifications during sleep hours
  - Escalate to caregivers if critical reminders missed
  - Generate motivational messages based on progress
- **Adherence Tracking**: Calculate compliance percentages and trends

### 4. Video Calling (Google Meet Integration)
- **Session Types**: group_community | one_on_one | expert_session
- **Scheduling**: Integrated with Google Calendar
- **Business Rules**:
  - Free: 1 group session/week
  - Premium: Unlimited private and group sessions
- **Privacy**: All participants must consent before joining
- **Recording**: Only with explicit consent from all participants

### 5. Chat System
- **Conversation Types**: direct | group | community
- **Message Types**: text | image | document | voice | system
- **Features**: End-to-end encryption, reactions, threading, file sharing
- **Moderation**: Content filtering + user reporting system
- **Retention**: Configurable per community (default: 365 days)

### 6. AI Health Assistant
- **Provider**: OpenAI API (GPT-4)
- **Use Cases**:
  - Answer health questions (with disclaimer: not medical advice)
  - Medication interaction checking
  - Symptom assessment and guidance
  - Personalized diet and exercise suggestions
- **Safety**: Crisis detection → immediate escalation to support resources
- **Rate Limiting**: Free (5 messages/day) | Premium (unlimited)

### 7. Premium Subscription
- **Pricing**: ₹199/month or ₹1,999/year (₹166/month)
- **Payment Gateway**: Razorpay (India) | Stripe (International)
- **Features**:
  - Unlimited communities and video calls
  - Advanced AI assistant
  - Family access (up to 3 caregivers)
  - Priority support
  - Advanced analytics
  - Data export
- **Trial**: 7-day free trial for new users

## 📁 Monorepo Structure

```
beautiful-life-project/
├── apps/
│   ├── frontend/              # Next.js 14 application
│   │   ├── src/
│   │   │   ├── app/          # Next.js 14 app router
│   │   │   ├── components/   # React components
│   │   │   ├── lib/          # Utilities and helpers
│   │   │   └── types/        # TypeScript type definitions
│   │   ├── public/           # Static assets
│   │   └── package.json
│   └── backend/              # Golang application
│       ├── cmd/server/       # Application entry point
│       ├── internal/         # Private application code
│       │   ├── domain/       # Business entities
│       │   ├── application/  # Use cases/services
│       │   ├── infrastructure/ # External dependencies
│       │   └── interfaces/   # HTTP handlers, DTOs
│       ├── pkg/              # Public shared packages
│       └── go.mod
├── packages/
│   ├── shared-types/         # Shared TypeScript types
│   ├── ui-components/        # React component library
│   └── constants/            # Shared constants
├── docs/                     # Comprehensive documentation
└── .github/
    └── copilot-instructions.md  # This file
```

## 💡 Coding Guidelines

### General Principles
1. **Follow existing patterns**: Maintain consistency with codebase structure
2. **Security first**: Never expose sensitive data, always validate input
3. **Type safety**: Use TypeScript strictly, no `any` types
4. **Error handling**: Graceful degradation, user-friendly error messages
5. **Comments**: Explain "why" not "what", keep code self-documenting

### Frontend (TypeScript/Next.js)
```typescript
// ✅ DO: Use server-side authentication
export async function getServerSideProps(context) {
  const user = await validateServerSideAuth(context.req);
  if (!user) {
    return { redirect: { destination: '/login', permanent: false } };
  }
  return { props: { user } };
}

// ✅ DO: Call Next.js API routes, not Golang directly from browser
const response = await fetch('/api/community/posts', {
  credentials: 'include', // Include HTTP-only cookies
});

// ❌ DON'T: Call Golang backend directly from browser
// This exposes backend URL and bypasses security
const response = await fetch('http://golang-backend:8080/api/v1/posts');

// ✅ DO: Use TypeScript interfaces matching backend
interface User {
  id: string;
  email: string;
  role: 'patient' | 'doctor' | 'caregiver';
  profile: UserProfile;
}

// ✅ DO: Handle errors gracefully
try {
  const data = await fetchUserData();
  return data;
} catch (error) {
  logger.error('Failed to fetch user data', error);
  return { error: 'Failed to load data' };
}
```

### Backend (Golang)
```go
// ✅ DO: Follow clean architecture layers
// Domain layer: Pure business logic
type User struct {
    ID    uuid.UUID
    Email string
    Role  UserRole
}

func (u *User) CanAccessMedicalRecord(recordOwnerID uuid.UUID) bool {
    return u.ID == recordOwnerID || u.Role == RoleDoctor
}

// Application layer: Use cases
type UserService interface {
    RegisterUser(ctx context.Context, req RegisterRequest) (*User, error)
}

// Interface layer: HTTP handlers
func (h *UserHandler) RegisterUser(c *gin.Context) {
    var req dto.RegisterRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, dto.ErrorResponse{...})
        return
    }
    // Call service layer
}

// ✅ DO: Use middleware for authentication
router.Use(middleware.AuthRequired())

// ✅ DO: Validate all input
if err := validator.Validate(req); err != nil {
    return errors.NewValidationError("invalid input", err)
}

// ✅ DO: Use structured logging
logger.Info("user registered", 
    "user_id", user.ID,
    "role", user.Role,
)

// ❌ DON'T: Return sensitive data in errors
// Bad: return errors.New("password hash xyz123 doesn't match")
// Good: return errors.New("invalid credentials")
```

### Database Queries
```go
// ✅ DO: Use parameterized queries (GORM prevents SQL injection)
db.Where("email = ? AND is_active = ?", email, true).First(&user)

// ✅ DO: Use transactions for multiple operations
err := db.Transaction(func(tx *gorm.DB) error {
    if err := tx.Create(&user).Error; err != nil {
        return err
    }
    if err := tx.Create(&profile).Error; err != nil {
        return err
    }
    return nil
})

// ✅ DO: Add indexes for frequent queries
// CREATE INDEX idx_users_email ON users(email);
// CREATE INDEX idx_medical_records_patient ON medical_records(patient_id);
```

## 🧪 Testing Guidelines

### Frontend Tests
```typescript
// Component tests with React Testing Library
describe('LoginForm', () => {
  it('should submit login form with valid credentials', async () => {
    render(<LoginForm />);
    fireEvent.change(screen.getByLabelText('Email'), {
      target: { value: 'user@example.com' },
    });
    fireEvent.click(screen.getByRole('button', { name: 'Login' }));
    await waitFor(() => {
      expect(mockLogin).toHaveBeenCalledWith('user@example.com', 'password');
    });
  });
});
```

### Backend Tests
```go
// Unit tests with testify
func TestUserService_RegisterUser(t *testing.T) {
    // Arrange
    mockRepo := new(MockUserRepository)
    service := NewUserService(mockRepo)
    
    // Act
    user, err := service.RegisterUser(ctx, validRequest)
    
    // Assert
    assert.NoError(t, err)
    assert.NotNil(t, user)
    assert.Equal(t, validRequest.Email, user.Email)
}
```

## 📊 Data Models & Relationships

### Core Entities
```
User (1) ──→ (1) Patient/Doctor/Caregiver
User (1) ──→ (N) MedicalRecords
User (1) ──→ (N) Reminders
User (1) ──→ (N) CommunityPosts
User (N) ←→ (N) Communities (via CommunityMembers)
User (N) ←→ (N) ChatConversations (via ConversationParticipants)
Community (1) ──→ (N) CommunityPosts
Community (1) ──→ (N) VideoSessions
MedicalRecord (1) ──→ (N) MedicalRecordShares
Reminder (1) ──→ (N) ReminderLogs
```

## 🚀 Common Development Tasks

### Adding a New API Endpoint

1. **Define domain entity** (`internal/domain/entity.go`)
2. **Create repository interface** (`internal/domain/repository.go`)
3. **Implement service** (`internal/application/service.go`)
4. **Create HTTP handler** (`internal/interfaces/http/handlers/handler.go`)
5. **Add DTOs** (`internal/interfaces/http/dto/dto.go`)
6. **Register route** (`internal/interfaces/http/routes/api.go`)
7. **Add Next.js API route** (`apps/frontend/src/app/api/route.ts`)
8. **Create frontend service** (`apps/frontend/src/lib/api.ts`)
9. **Write tests** (both frontend and backend)

### Adding a New Feature Flag
```go
// Backend
type FeatureFlags struct {
    EnableAIChat      bool
    EnableVideoCall   bool
    EnablePremiumTier bool
}

// Check in handler
if !featureFlags.EnableVideoCall {
    return errors.New("feature not available")
}
```

### Database Migration
```bash
# Create new migration
cd apps/backend
migrate create -ext sql -dir internal/infrastructure/database/migrations -seq add_video_sessions_table

# Run migrations
make migrate-up

# Rollback
make migrate-down
```

## 🔍 Debugging Tips

1. **Check logs first**: Both Next.js and Golang have structured logging
2. **Verify authentication**: Ensure JWT token is present in HTTP-only cookie
3. **Test API endpoints**: Use Swagger UI at `http://localhost:8080/swagger`
4. **Check database**: Verify data integrity and relationships
5. **Monitor Redis**: Check session storage and cache
6. **Review network tab**: Inspect request/response in browser DevTools

## 📚 Key Documentation References

- **Business Logic**: `docs/business-logic.md` - Complete feature specifications
- **Backend Architecture**: `docs/backend_arch.md` - Golang structure and patterns
- **Stakeholder Guide**: `docs/business-logic-stackholders.md` - Non-technical overview
- **Roadmap**: `docs/roadmap.md` - Development timeline and milestones

## 🎯 When Working on Code

### Always Consider:
1. **Is this HIPAA compliant?** (medical data encryption, access logging)
2. **Is input validated?** (prevent injection attacks)
3. **Are errors handled gracefully?** (user-friendly messages)
4. **Is it performant?** (database indexes, caching)
5. **Is it tested?** (unit and integration tests)
6. **Is it documented?** (code comments for complex logic)
7. **Does it follow existing patterns?** (maintain consistency)

### Red Flags to Avoid:
❌ Exposing sensitive data in logs or error messages
❌ Calling Golang backend directly from browser (bypass Next.js API routes)
❌ Storing JWT tokens in localStorage (XSS vulnerability)
❌ Using `any` type in TypeScript (lose type safety)
❌ Hardcoding API URLs or secrets
❌ Skipping input validation
❌ Ignoring error handling
❌ Not logging important actions (audit trail)

## 🤖 AI Assistant Specific Guidance

When generating code suggestions:
1. **Match the existing architecture**: Clean architecture for Golang, server-side patterns for Next.js
2. **Follow security best practices**: Never suggest insecure patterns
3. **Use established patterns**: Reference existing code structure
4. **Include error handling**: Always handle errors gracefully
5. **Add type safety**: Use TypeScript interfaces and Go types strictly
6. **Consider performance**: Suggest efficient database queries and caching
7. **Think about testing**: Suggest testable code with clear interfaces

---

**Remember**: This is a healthcare platform. Security, privacy, and reliability are paramount. When in doubt, prioritize security over convenience.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sakibcoolz)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/sakibcoolz)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
