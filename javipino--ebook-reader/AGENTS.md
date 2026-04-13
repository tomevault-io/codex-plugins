# AI-Powered Ebook Reader - Development Instructions

## Project Overview

This is an AI-powered ebook reader application that enables users to read books with intelligent features including:
- AI-based character identification and analysis
- Text-to-speech conversion with character-specific voices
- Seamless switching between reading and listening modes
- Amazon Kindle synchronization (planned)

**Current Status**: Web application (mobile app planned for future)

## Tech Stack Decisions

### Frontend
- **Framework**: React 19 with TypeScript
- **Build Tool**: Vite
- **Styling**: TailwindCSS
- **Routing**: React Router v6
- **HTTP Client**: Axios
- **State Management**: Zustand (when needed)

### Backend
- **Framework**: .NET 8 Web API
- **Architecture**: Clean Architecture (3-layer)
  - `EbookReader.API`: Controllers and API endpoints
  - `EbookReader.Core`: Domain entities and business logic
  - `EbookReader.Infrastructure`: Database context and external services
- **Database**: PostgreSQL 16 (Npgsql.EntityFrameworkCore.PostgreSQL 8.0.11)
- **API Documentation**: Swagger/OpenAPI
- **Background Jobs**: Hangfire with PostgreSQL storage
- **Logging**: Serilog with console and file sinks
- **File Storage**: Abstracted (IFileStorageService) supporting both Azure Blob Storage and local filesystem
- **Ebook Parser**: VersOne.Epub 3.3.4 for EPUB format

### AI & Services
- **Character Analysis**: Azure OpenAI (GPT-4o or GPT-3.5-turbo)
  - Used to identify main characters in books
  - Extract character descriptions for voice assignment
- **Text-to-Speech**: ElevenLabs (primary - implemented)
  - **Model**: Eleven Flash v2.5 (`eleven_flash_v2_5`) - cheapest/fastest model
  - **Default Voice**: `6bNjXphfWPUDHuFkgDt3`
  - **Features**: WebSocket streaming for smooth continuous playback
  - **Pricing**: Pay-per-use, neural voices
  - **API**: WebSocket streaming via backend proxy at `/api/ttsstream/stream`
      - Auth: pass JWT as query string `access_token`
  - Configure via `ElevenLabs:*` in appsettings.json

- **Text-to-Speech (alternative - implemented)**: Azure Speech (Microsoft Cognitive Services Speech)
   - Used as an optional provider when ElevenLabs credits are exhausted
   - Configure via `AzureSpeech:*` in appsettings.json
   - Provider selection UI: `/admin/settings` (stored per-user in database; applied server-side during TTS streaming)
- **Alternative TTS options** (evaluated):
  - Google Cloud TTS: Free tier 1M chars/month, good quality
  - OpenAI TTS: $15/1M chars, excellent quality, 6 voices
- **Hybrid strategy** (future optimization):
  - Use cheaper TTS for narration (~70% of content)
  - Use premium TTS (ElevenLabs) for character dialogue (~30%)

### Infrastructure
- **Hosting**: Azure (or self-hosted Debian server)
  - App Service for .NET API
  - Static Web Apps for React frontend (future)
  - Azure Database for PostgreSQL (flexible server) or self-hosted PostgreSQL
  - Azure Blob Storage for ebook files
- **Local Development**: Docker Compose
- **CI/CD**: Azure DevOps (planned)

## Architecture Patterns

### Database Schema
- **User**: Authentication entity with Id (GUID), Username (unique), Email (unique), PasswordHash (BCrypt), CreatedAt, LastLoginAt
- **Book**: Core entity with UserId foreign key, title, author, file path, cover image, upload date (user-specific libraries)
- **Character**: Linked to Book, stores character name, description, assigned voice
- **Chapter**: Book content split into chapters for efficient TTS processing
- **ReadingProgress**: Tracks user's current position per book (UserId foreign key)
- **KindleAccount**: Stores encrypted Amazon credentials, sync status, marketplace per user (one account per user)
- **KindleBook**: Maps Kindle ASINs to Book entities, tracks Kindle reading positions

### Key Design Decisions

1. **Ebook Format Support**: EPUB only (MVP)
   - Use VersOne.Epub 3.3.4 parser
   - Add PDF support in future phases
   - MOBI/AZW require conversion to EPUB

2. **File Storage Strategy**: Environment-based abstraction
   - `IFileStorageService` interface supports multiple backends
   - **Local**: Default for development and self-hosted Debian
   - **Azure Blob**: Production Azure deployments
   - Configure via `FileStorage:Type` in appsettings.json

3. **Audio Format**: MP3 128kbps, 24kHz sample rate
   - Best balance of file size and speech quality
   - Generated on-demand, cached for reuse
   - Reduces TTS costs significantly

4. **Background Job Processing**: Hangfire with PostgreSQL
   - Long-running tasks (character analysis, TTS generation)
   - Web dashboard at `/hangfire` (development only)
   - Same database as application data

5. **Monitoring & Logging**: Serilog
   - Structured logging to console and rolling files
   - 7-day log retention
   - Logs stored in `logs/` directory (gitignored)

6. **On-Demand Audio Generation**: Don't pre-convert entire books
   - Generate audio chapter by chapter as user progresses
   - Cache generated audio for reuse
   - Reduces TTS costs significantly

7. **Character Voice Assignment**:
   - Step 1: Use Azure OpenAI to analyze book and identify main characters
   - Step 2: Present characters to user
   - Step 3: User selects voice for each character (or use AI suggestions)
   - Step 4: Store voice assignments in database

8. **Playback Speed**: Client-side control
   - Use HTML5 Audio API `playbackRate` property
   - No need to regenerate audio at different speeds
   - User preference stored per book

9. **CORS Configuration**: Frontend (localhost:5173, localhost:5174) allowed in development

10. **Authentication**: JWT-based authentication with BCrypt password hashing
   - 30-day token expiration (43200 minutes)
   - Tokens stored in localStorage on frontend
   - Axios interceptors auto-inject Bearer token
   - 401 responses trigger automatic logout
   - All books and reading progress are user-specific

11. **Kindle Integration**: Cookie-based authentication with encrypted storage
   - Uses browser session cookies extracted by user from Amazon
   - Cookies encrypted with AES before database storage
   - Supports multiple Amazon marketplaces (.com, .co.uk, .de, etc.)
   - Cookie validation before saving to ensure they are valid
   - Automatic sync via Hangfire background jobs (daily)
   - Manual sync available from frontend
   - Two-way reading progress synchronization
   - **Note**: Book download requires DRM handling (not fully implemented in MVP)
   - **How it works**: User logs into Amazon in browser, extracts session cookies via DevTools, pastes into the app. Cookies expire after a few weeks and need to be refreshed.

12. **Error Handling**: Consistent toast notifications across all pages
   - Uses react-hot-toast for all user-facing errors
   - No inline error displays or custom ErrorAlert components
   - Validation errors, API errors, and success messages all use toast
   - Books without files (Kindle metadata-only) show toast and redirect to library

## Project Structure

```
ebook_reader/
├── frontend/                    # React application
│   ├── src/
│   │   ├── components/         # Reusable components (Layout, ProtectedRoute)
│   │   ├── contexts/           # React contexts (AuthContext)
│   │   ├── pages/              # Route pages (Home, Library, Reader, LoginPage)
│   │   ├── services/           # API client (api.ts with JWT interceptors)
│   │   ├── App.tsx             # Main app component with routing
│   │   └── main.tsx            # Entry point
│   ├── package.json
│   ├── vite.config.ts
│   ├── tailwind.config.js
│   └── Dockerfile
├── backend/
│   ├── EbookReader.API/        # Web API controllers
│   │   ├── Controllers/        # BooksController, AuthController, KindleController
│   │   ├── Program.cs          # App configuration & DI, JWT setup
│   │   └── appsettings.*.json  # Configuration (JWT, FileStorage, Audio, Kindle)
│   ├── EbookReader.Core/       # Domain layer
│   │   ├── Entities/           # User, Book, Character, Chapter, ReadingProgress, KindleAccount, KindleBook
│   │   └── Interfaces/         # IFileStorageService, IBookService, IKindleService
│   ├── EbookReader.Infrastructure/  # Data layer
│   │   ├── Data/
│   │   │   └── EbookReaderDbContext.cs
│   │   └── Services/           # LocalFileStorageService, AzureBlobStorageService, BookService, KindleService, KindleBackgroundJobs
│   ├── EbookReader.sln
│   └── Dockerfile
├── infrastructure/             # Azure deployment (Bicep)
│   ├── main.bicep
│   └── README.md
├── docker-compose.yml
└── README.md
```

## Development Setup

### Prerequisites
- Node.js 18+
- .NET 8 SDK
- Docker Desktop
- PostgreSQL 16 (via Docker)

### Local Development

**Docker Compose (Frontend + Backend + Database)**
```bash
cd C:\Repos\Training\ebook_reader
docker-compose up -d --build
```

**Access Points**:
- Frontend: http://localhost:5174
- Backend API (HTTPS): https://localhost:5001 (HTTP: http://localhost:5000)
- Swagger (HTTPS): https://localhost:5001/swagger
- Hangfire Dashboard (HTTPS): https://localhost:5001/hangfire
- Database: localhost:5432

**HTTPS dev certificate (Windows)**

The backend container mounts a dev certificate from the host at `%USERPROFILE%\.aspnet\https\aspnetapp.pfx`.
Generate it if missing:
```bash
dotnet dev-certs https -ep "%USERPROFILE%\\.aspnet\\https\\aspnetapp.pfx" -p yourpassword
```

The password must match `ASPNETCORE_Kestrel__Certificates__Default__Password` in `docker-compose.yml`.

**Storage**

Uploads and generated assets are persisted via the `backend-storage` Docker volume (mounted at `/app/storage`).

### Environment Variables

**Backend (.env)**:
```
AZURE_OPENAI_ENDPOINT=your-endpoint
AZURE_OPENAI_KEY=your-key
GOOGLE_CLOUD_PROJECT_ID=your-project-id
GOOGLE_APPLICATION_CREDENTIALS=path-to-credentials.json
ConnectionStrings__DefaultConnection=Host=localhost;Port=5432;Database=EbookReader;Username=postgres;Password=postgres
Kindle__EncryptionKey=your-32-character-encryption-key-here
ElevenLabs__ApiKey=your-elevenlabs-api-key
ElevenLabs__DefaultVoiceId=6bNjXphfWPUDHuFkgDt3
AzureSpeech__Key=your-azure-speech-key
AzureSpeech__Region=your-azure-speech-region
AzureSpeech__DefaultVoiceName=en-US-JennyNeural
```

**Frontend (.env)**:
```
VITE_API_URL=https://localhost:5001
```

## Coding Conventions

### Frontend
- Use TypeScript for all new files
- Functional components with hooks
- File naming: PascalCase for components (`HomePage.tsx`)
- Export components as default
- Use Tailwind utility classes for styling
- API calls should go through `services/api.ts`
- **IMPORTANT**: All API endpoints MUST include `/api` prefix (e.g., `/api/books`, `/api/kindle/status`)

### Backend
- Follow Clean Architecture principles
- Controllers in `EbookReader.API/Controllers`
- Entities in `EbookReader.Core/Entities`
- DbContext and repositories in `EbookReader.Infrastructure`
- Use async/await for all database operations
- Return `ActionResult<T>` from controller actions
- Use DTOs for API responses (future enhancement)

### Database
- Use GUID for primary keys
- Required fields: explicitly mark with `[Required]` or `IsRequired()`
- Navigation properties: use `ICollection<T>` for one-to-many
- Cascade delete where appropriate
- Use UTC for all timestamps

## API Endpoints (Current & Planned)

**Authentication**
- `POST /api/auth/register` - Register new user (username, email, password)
- `POST /api/auth/login` - Login and receive JWT token

**Books** (Require JWT authentication)
- `GET /api/books` - List current user's books
- `GET /api/books/{id}` - Get book details (user must own book)
- `POST /api/books` - Upload book for current user
- `DELETE /api/books/{id}` - Delete user's book
- `POST /api/books/{id}/analyze-characters` - Trigger AI character analysis (planned)

**Characters**
- `GET /api/books/{bookId}/characters` - List characters for book (planned)
- `PUT /api/characters/{id}/voice` - Assign voice to character (planned)

**Audio**
- `POST /api/books/{bookId}/chapters/{number}/audio` - Generate chapter audio (planned)
- `GET /api/books/{bookId}/chapters/{number}/audio` - Stream chapter audio (planned)

**Text-to-Speech (TTS)** (Require JWT authentication)
- `WS /api/ttsstream/stream` - WebSocket endpoint for streaming TTS audio
- `POST /api/tts/convert` - Convert text to speech (legacy REST endpoint)
- `GET /api/tts/voices` - Get available TTS voices

**Reading Progress**
- `GET /api/books/{bookId}/progress` - Get user's reading position (planned)
- `PUT /api/books/{bookId}/progress` - Update reading position (planned)

**Kindle Integration** (Require JWT authentication)
- `GET /api/kindle/status` - Get Kindle account connection status
- `POST /api/kindle/connect` - Connect Amazon Kindle account
- `DELETE /api/kindle/disconnect` - Disconnect Kindle account
- `POST /api/kindle/sync` - Manually sync Kindle library
- `POST /api/kindle/sync/progress/{bookId}` - Sync reading progress for specific bo

**Note**: All API routes are prefixed with `/api` as configured in the controller route attribute `[Route("api/[controller]")]`ok
- `POST /api/kindle/push/progress/{bookId}` - Push local reading progress to Kindle

## Future Features & Roadmap

### Phase 1: MVP (Current)
- [x] Basic project structure
- [x] Database schema with User entity
- [x] Frontend UI skeleton with routing
- [x] User authentication (JWT)
- [x] Protected routes
- [x] User-specific data isolation
- [x] Book upload functionality
- [x] Chapter parsing with VersOne.Epub
- [x] Kindle account connection
- [x] Kindle library sync (manual and automatic)
- [x] Basic reader view
- [x] Standardized error handling with toast notifications

### Phase 2: AI Integration
- [ ] Azure OpenAI integration for character analysis
- [x] ElevenLabs TTS integration (Eleven Flash v2.5 model - cheapest)
- [ ] Word-level highlighting during TTS playback (removed for WebSocket streaming)
- [ ] Character voice assignment UI
- [x] Audio generation pipeline (on-demand per page)
- [x] Audio player in reader view with play/pause/stop
- [x] Playback speed controls (0.5x - 2.0x)
- [x] WebSocket streaming for smooth continuous audio playback

### Phase 3: Enhanced Features
- [ ] Reading progress tracking
- [x] Playback speed controls
- [ ] Bookmarks and highlights
- [ ] Audio caching strategy
- [ ] User profile management
- [ ] Password reset functionality

### Phase 4: External Integrations
- [x] Amazon Kindle API integration (unofficial)
- [x] Kindle library sync
- [x] Kindle reading progress sync
- [x] Metadata-only books support (FilePath nullable)
- [ ] Full DRM book download support
- [ ] Sync reading position across devices

### Phase 5: Mobile App
- [ ] React Native mobile app
- [ ] Offline reading support
- [ ] Background audio playback

## Cost Optimization Strategies

1. **TTS**: Start with Google Cloud (free tier), scale to hybrid model (OpenAI narration + ElevenLabs dialogue)
2. **Storage**: Use Azure Blob Storage cool tier for ebook files
3. **Database**: Azure PostgreSQL Flexible Server (burstable tier) or self-hosted on Debian
4. **Compute**: Azure App Service B1 tier (~$13/month) covered by $50 credit

## Known Issues & Technical Debt

1. **DTOs**: Using entities directly in API responses (should add DTOs)
2. **Validation**: Input validation needs to be added to API endpoints (email format, password strength)
3. **Refresh Tokens**: Currently using long-lived tokens (30 days), should implement refresh token pattern
4. **Password Reset**: No password reset functionality yet
5. **Email Verification**: Email addresses are not verified
6. **Rate Limiting**: No rate limiting on authentication endpoints
7. **Kindle DRM**: Cannot download DRM-protected Kindle books (only metadata synced)
8. **Reading UI for Kindle Books**: Books without files (Kindle metadata-only) cannot be opened - user redirected with toast notification

## Testing Strategy (Future)

- **Frontend**: Vitest + React Testing Library
- **Backend**: xUnit + Moq
- **E2E**: Playwright
- **API**: Swagger-based contract testing

## Security Considerations

- [x] JWT authentication with BCrypt password hashing
- [x] User-specific authorization for book access
- [ ] Add refresh token pattern (currently using long-lived tokens)
- [ ] Email verification for new accounts
- [ ] Password reset functionality
- [ ] Rate limiting on authentication endpoints
- [ ] Sanitize file uploads
- [ ] Validate ebook file formats
- [ ] Rate limiting on TTS endpoints
- [ ] Secure storage of API keys (Azure Key Vault)
- [ ] HTTPS enforcement in production
- [ ] Consider Azure AD B2C for enterprise (future)

## Performance Considerations

- Audio files should be streamed, not downloaded entirely
- Implement pagination for large book libraries
- Consider CDN for static assets (Azure CDN)
- Database indexing on frequently queried fields
- Lazy loading for chapter content

## Language Support

- Primary: English and Spanish
- TTS services support both languages natively
- Character analysis prompts should support multilingual books
- UI should be i18n-ready for future localization

## References & Documentation

- Azure OpenAI: https://learn.microsoft.com/azure/ai-services/openai/
- Google Cloud TTS: https://cloud.google.com/text-to-speech
- OpenAI TTS Pricing: $15/1M characters
- ElevenLabs Pricing: Starting at $5/month for 30K characters
- React Router v6: https://reactrouter.com/
- Entity Framework Core: https://learn.microsoft.com/ef/core/
- JWT Authentication: https://jwt.io/
- BCrypt.Net: https://github.com/BcryptNet/bcrypt.net

---

## Documentation Maintenance Guidelines

**CRITICAL**: When implementing new features, making architectural changes, or resolving technical debt, ALWAYS update both:

1. **README.md** (user-facing documentation)
2. **This file** (`.github/copilot-instructions.md` - AI development guidelines)

### What to Update When

**New Feature Implementation:**
- Add to Features section in README
- Update Tech Stack if new libraries added
- Document new API endpoints
- Update Database Schema if entities changed
- Add to Coding Conventions if new patterns introduced
- Check off roadmap items
- Update Project Structure if new directories/files

**Bug Fixes:**
- Remove from Known Issues & Technical Debt
- Update relevant code examples if patterns changed

**Configuration Changes:**
- Update Environment Variables section
- Update appsettings.json examples
- Update docker-compose.yml references

**Dependency Updates:**
- Update version numbers in Tech Stack
- Note breaking changes in Known Issues if applicable

**Architecture Changes:**
- Update Key Design Decisions
- Update Architecture Patterns
- Update Project Structure
- Update Coding Conventions

### Sections Requiring Frequent Updates

- **Tech Stack Decisions**: Library versions, new packages
- **Database Schema**: Entity changes, relationships
- **API Endpoints**: New endpoints, authentication requirements
- **Roadmap**: Check off completed items, add new phases
- **Known Issues**: Remove resolved, add new blockers
- **Security Considerations**: Check off implemented items
- **Project Structure**: New directories, important files

### Best Practices

1. **Be Specific**: Include version numbers, exact file paths, command examples
2. **Keep It Current**: Update immediately when implementing changes, not later
3. **Be Honest**: Document technical debt and known issues clearly
4. **Think Future**: These docs guide future development decisions
5. **User Perspective**: README should help users; copilot-instructions help developers/AI

**Remember**: Good documentation prevents confusion, ensures consistency, and accelerates development. Treat it as part of the code, not an afterthought.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/javipino)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/javipino)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
