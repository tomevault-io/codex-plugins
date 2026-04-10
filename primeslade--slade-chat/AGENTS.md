# Backend Architecture

## AI Instructions

Role: You are a Senior Backend Engineer specializing in NestJS.
Constraint: You must strictly adhere to the layered architecture defined below.
Critical Rule: Never allow Controllers to talk to Prisma directly.
Critical Rule: Never allow Repositories to contain business logic.

## Core Technologies

### Framework

- **NestJS** - Progressive Node.js framework with TypeScript support, dependency injection, and modular architecture

### Database

- **PostgreSQL** - Relational database for persistent data storage
- **Prisma ORM** - Type-safe database client with schema migration and query building

### Caching

- **Redis** - In-memory data store for caching and session management
- **@nestjs/cache-manager** - NestJS integration for cache management with Redis adapter

### Real-time Communication

- **Socket.io** - WebSocket library for bidirectional event-based communication
- **@nestjs/websockets** - NestJS integration for WebSocket gateways

### Authentication

- **Better Auth** - Modern authentication solution with secure session management
- **@thallesp/nestjs-better-auth** - NestJS integration for Better Auth
- **Bcrypt** - Password hashing and encryption

### Validation & Types

- **Zod** - TypeScript-first schema validation for DTOs
- **Prisma Types** - Auto-generated types from database schema

### Additional Features

- **@nestjs/event-emitter** - Event-driven architecture for decoupled components
- **@google/genai** - Google Generative AI integration
- **@nestjs/swagger** - API documentation generation
- **@nestjs/schedule** - Task scheduling and cron jobs
- **Jose** - JWT operations and verification

---

## Architecture Layers

### Three-Layer Pattern: Repository → Service → Controller

```
┌─────────────────────────────────────────────┐
│ Controller Layer                            │
│ - Request/Response handling                 │
│ - Input validation (Zod)                    │
│ - Route definitions                         │
│ - Guards, Interceptors                      │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│ Service Layer                               │
│ - Business logic                            │
│ - Transaction management                    │
│ - Error handling                            │
│ - Event emission                            │
│ - Service orchestration                     │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│ Repository Layer                            │
│ - Prisma queries                            │
│ - Database operations                       │
│ - Raw SQL (if needed)                       │
│ - Transaction support                       │
└─────────────────────────────────────────────┘
```

### Layer Responsibilities

#### Repository Layer (`*.repository.ts`)

**What it does:**

- Direct interaction with Prisma client
- CRUD operations and complex queries
- Accepts optional transaction client for transactional operations
- Returns Prisma types or custom types from `@backend/src/shared/`

**What it doesn't do:**

- Business logic or validation
- Error transformation (only database errors)
- Event emission

**Example:**

```typescript
@Injectable()
export class MessagesRepository {
  constructor(private readonly prismaService: PrismaService) {}

  async createMessage(
    data: { content: string; senderId: string; roomId: string },
    trx?: Prisma.TransactionClient
  ): Promise<MessageWithSender> {
    const prisma = trx || this.prismaService;
    return prisma.message.create({
      data,
      include: { sender: true },
    });
  }
}
```

#### Service Layer (`*.service.ts`)

**What it does:**

- Implements business logic and rules
- Orchestrates multiple repository calls
- Manages Prisma transactions using `$transaction()`
- **Handles all error cases and transformations**
- Emits events using EventEmitter2
- Validates business rules

**What it doesn't do:**

- HTTP-specific logic (status codes, headers)
- Request parsing or response formatting

**Example:**

```typescript
@Injectable()
export class MessagesService {
  constructor(
    private readonly messagesRepository: MessagesRepository,
    private readonly eventEmitter: EventEmitter2
  ) {}

  async createMessage(data: {
    content: string;
    senderId: string;
    roomId: string;
  }): Promise<MessageWithSender> {
    const message = await this.messagesRepository.createMessage(data);
    this.eventEmitter.emit("message_created", message.roomId, message);
    return message;
  }

  async updateMessage(data: UpdateMessageDto, senderId: string) {
    const message = await this.messagesRepository.getMessageByMessageId(
      data.messageId
    );

    if (!message) {
      throw new NotFoundException("Message not found");
    }

    if (message.senderId !== senderId) {
      throw new ForbiddenException("Not authorized to update this message");
    }

    return this.messagesRepository.updateMessage(data, senderId);
  }
}
```

#### Controller Layer (`*.controller.ts`)

**What it does:**

- Defines HTTP routes and methods
- Validates request input using Zod schemas
- Applies guards for authentication/authorization
- Calls service methods
- Formats responses using `ControllerResponse<T>` type
- Applies interceptors

**What it doesn't do:**

- Business logic
- Direct database access
- Error handling (handled by filters)

**Example:**

```typescript
@UseGuards(HttpRoomGuard)
@Controller("messages")
export class MessagesController {
  constructor(private readonly messagesService: MessagesService) {}

  @Get("room/:roomId")
  async getMessages(
    @Param("roomId") roomId: string,
    @Query(new ZodValidationPipe(getMessagesBodySchema))
    query: GetMessagesBodyDto
  ): Promise<ControllerResponse<MessageWithSender[]>> {
    const { messages, nextCursor } = await this.messagesService.getMessages({
      roomId,
      ...query,
    });

    return {
      data: messages,
      meta: { nextCursor },
    };
  }
}
```

---

## Cross-Cutting Concerns

### Filters (`src/common/filters/`)

- `all-exceptions.filter.ts` - Catches all unhandled exceptions
- `http-exception.filter.ts` - Handles NestJS HTTP exceptions
- `prisma-client-exception.filter.ts` - Transforms Prisma errors to HTTP errors

### Guards (`src/common/guards/`)

- `http-room.guard.ts` - Validates user access to rooms in HTTP requests
- `message-sender.guard.ts` - Validates message ownership
- `ws-rooms.guard.ts` - Validates user access to rooms in WebSocket connections

### Interceptors (`src/common/interceptors/`)

- `response.interceptor.ts` - Standardizes response format

### Pipes (`src/common/pipes/`)

Zod Validation Pipe:

```typescript
@Query(new ZodValidationPipe(getMessagesBodySchema))
query: GetMessagesBodyDto
```

---

## Type System

### Shared Types (`@backend/src/shared/`)

```
src/shared/
├── index.ts
├── messages/
│   ├── types.ts
│   └── zodSchemas.ts
├── rooms/
│   ├── types.ts
│   └── zodSchemas.ts
└── users/
    ├── types.ts
    └── zodSchemas.ts
```

#### Types (`types.ts`)

```typescript
export type { Message } from "generated/prisma/client";

export type MessageWithSender = Prisma.MessageGetPayload<{
  include: {
    sender: true;
  };
}>;
```

#### Zod Schemas (`zodSchemas.ts`)

```typescript
export const createMessageSchema = z.object({
  roomId: z.string(),
  content: z.string().max(2000),
});

export const createMessageBodySchema = createMessageSchema.omit({
  roomId: true,
});

export type CreateMessageDto = z.infer<typeof createMessageSchema>;
export type CreateMessageBodyDto = z.infer<typeof createMessageBodySchema>;
```

### Type Flow

```
Controller → Service → Repository → Database
   ↓           ↓          ↓
BodyDto     FullDto   PrismaTypes
(Zod)       (Zod)     (Prisma)
```

---

## Transaction Management

```typescript
async complexOperation(data: ComplexDto): Promise<Result> {
  return this.prismaService.$transaction(async (trx) => {
    const step1 = await this.repo1.create(data.part1, trx);
    const step2 = await this.repo2.update(data.part2, trx);
    return { step1, step2 };
  });
}
```

---

## Error Handling

All business logic errors thrown in the service layer:

```typescript
if (!resource) throw new NotFoundException("Resource not found");
if (resource.ownerId !== userId) throw new ForbiddenException("Access denied");
if (invalidCondition) throw new BadRequestException("Invalid input");
if (duplicateExists) throw new ConflictException("Resource already exists");
```

Filter transformations:

```
PrismaClientKnownRequestError: P2025 → NotFoundException
PrismaClientKnownRequestError: P2002 → ConflictException
```

---

## Development Guidelines

### Layer Rules

#### Repository ✓ Can / ✗ Cannot

✓ Use Prisma client, write raw SQL, accept transaction client, return Prisma/shared types, throw Prisma errors  
✗ Business logic, validate business rules, transform exceptions, emit events, call other services

#### Service ✓ Can / ✗ Cannot

✓ Business logic, validate rules, call multiple repositories, manage transactions, emit events, throw HTTP exceptions, call other services  
✗ Access Prisma directly, handle HTTP-specific logic, format responses

#### Controller ✓ Can / ✗ Cannot

✓ Define routes, validate input with Zod, apply guards/interceptors, call service methods, format responses  
✗ Business logic, access repositories directly, handle exceptions, emit events

### Code Standards

- **TypeScript Strict Mode**: Enabled
- **No `any` types**: Use proper typing
- **Export shared types**: From `@backend/src/shared/`
- **Zod for validation**: All DTOs use Zod schemas
- **Prisma for queries**: Never use raw SQL unless necessary
- **Error handling in services**: Always validate and throw appropriate exceptions

---

## Project Structure

```
backend/
├── src/
│   ├── app.module.ts
│   ├── main.ts
│   ├── prisma.service.ts
│   ├── prisma.module.ts
│   ├── common/
│   │   ├── decorators/
│   │   ├── filters/
│   │   ├── guards/
│   │   ├── interceptors/
│   │   ├── pipes/
│   │   └── types/
│   ├── shared/
│   │   ├── index.ts
│   │   ├── messages/
│   │   ├── rooms/
│   │   └── users/
│   ├── chat/
│   │   ├── chat.gateway.ts
│   │   ├── chat.module.ts
│   │   └── guards/
│   ├── messages/
│   ├── rooms/
│   └── users/
├── prisma/
│   ├── schema.prisma
│   └── migrations/
└── generated/
    └── prisma/
        └── client/
```

---

# Frontend Architecture

## AI Instructions

Role: You are a Senior Frontend Engineer specializing in Next.js and TypeScript.
Constraint: You must strictly adhere to the "Integration Patterns" defined below.
Critical Rule: All Data Fetching (GET) must follow the `API → Hook → App` pattern.
Critical Rule: All Mutations (POST/PUT/DELETE) must follow the `API → App` pattern (skipping hooks).

## Core Technologies

- **TypeScript** - Type-safe JavaScript
- **Next.js** - React framework with SSR, routing, and optimizations
- **Tailwind CSS** - Utility-first CSS framework
- **shadcn/ui** - Re-usable component library built on Radix UI and Tailwind
- **Socket.io** - WebSocket library for real-time communication
- **Better Auth** - Modern authentication solution

### Theme

- **Design System**: Black and white minimalist
- High contrast, clean interface focusing on content and functionality

---

## Architecture Pattern

### Integration Pattern: API → Hook → App

```
┌─────────────────────────────────────────────┐
│ API Layer (lib/api)                         │
│ - HTTP requests                             │
│ - Socket.io connections                     │
│ - API client functions                      │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│ Hook Layer (hooks/)                         │
│ - Custom React hooks                        │
│ - State management                          │
│ - Business logic                            │
│ - Data transformation                       │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│ App Layer (app/, components/)               │
│ - UI components                             │
│ - Pages and routes                          │
│ - User interactions                         │
└─────────────────────────────────────────────┘
```

### Exception: Mutations (API → App directly)

For POST, PUT, DELETE — skip the hook layer and call the API directly from event handlers.

**Why:** Mutations are one-time user-triggered actions; the hook abstraction adds unnecessary complexity.

---

## Layer Responsibilities

### API Layer (`lib/api`)

- Handles all external communication
- Manages HTTP requests and responses
- Establishes and maintains Socket.io connections

### Hook Layer (`hooks/`)

- Consumes API layer functions
- Manages component state and side effects
- Handles loading states, errors, and caching
- Provides clean interface for UI components

### App Layer (`app/` & `components/`)

- Consumes hooks for data and functionality
- Focuses purely on UI rendering and user interactions
- Composes shadcn/ui components with custom components

---

## Development Guidelines

- All GET requests: **API → Hook → App**
- All mutations: **API → App** (call API directly from event handlers)
- Use TypeScript strictly — avoid `any` types (do not edit existing `any` types)
- Follow minimalist black and white theme throughout
- Keep components focused on presentation
- Extract reusable logic into hooks (for queries only)

---

## Project Structure

```
frontend/
├── app/                  # Next.js app router pages
├── components/           # Reusable UI components (shadcn/ui + custom)
├── hooks/               # Custom React hooks (for queries/subscriptions)
├── lib/
│   └── api/            # API layer implementations
├── contexts/            # React contexts for global state
└── public/              # Static assets
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PrimeSlade)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/PrimeSlade)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
