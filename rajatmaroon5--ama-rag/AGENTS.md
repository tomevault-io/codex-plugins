# AMA RAG Chatbot - Development Guidelines

## Project Overview
RAG (Retrieval-Augmented Generation) chatbot built with:
- **Backend**: ASP.NET Core 8.0 Minimal APIs
- **Frontend**: Angular 17 with standalone components
- **Vector DB**: Pinecone (cloud-based)
- **LLM**: OpenAI (GPT-3.5/GPT-4)
- **Embeddings**: OpenAI primary, HuggingFace alternative
- **Document Types**: PDF, DOCX
- **Scale**: Small (100s of documents, <1GB)

## Project Structure

```
AMA-RAG/
├── AmaRAGBackend/          # ASP.NET Core backend
│   ├── Services/           # Business logic (Document, Chat, Embedding, etc.)
│   ├── Endpoints/          # API endpoints (Documents, Chat)
│   ├── Models/             # Data models and DTOs
│   ├── Data/               # Database context (future)
│   ├── Utils/              # Utility functions
│   └── Program.cs          # Configuration and startup
├── AmaRAGUI/               # Angular frontend
│   ├── src/app/
│   │   ├── services/       # API service
│   │   ├── components/     # UI components
│   │   ├── models/         # TypeScript interfaces
│   │   └── app.component.ts
│   ├── angular.json        # Angular configuration
│   └── package.json        # NPM dependencies
├── README.md               # Full documentation
└── QUICKSTART.md           # Getting started guide

## Development Standards

### Backend (.NET)
- **Pattern**: ASP.NET Core Minimal APIs
- **Structure**: Services (business logic) → Data access layer
- **API Versioning**: /api/v1/ prefix
- **Logging**: Serilog for structured logging
- **Error Handling**: Try-catch with logging in services
- **Dependency Injection**: Built-in .NET DI container
- **Frameworks**: Entity Framework Core (future), FluentValidation

### Frontend (Angular)
- **Pattern**: Standalone components (Angular 17+)
- **Reactive**: Forms with Reactive Forms module
- **State**: Service-based with RxJS Observables
- **UI**: Angular Material components
- **Styling**: SCSS with component-scoped styles

### Services & Components

#### Backend Services
- `DocumentService` - Upload, process, manage documents
- `EmbeddingService` - Generate embeddings (OpenAI/HuggingFace)
- `PineconeService` - Vector DB operations
- `ChatService` - RAG chat logic with LLM integration
- `ChunkingService` - Split documents into chunks

#### Frontend Components
- `AppComponent` - Main container with tabs
- `DocumentUploadComponent` - Drag-drop file upload
- `ChatComponent` - Q&A interface with parameters
- `DocumentListComponent` - View uploaded documents

### Vector Operations
- **Chunking**: 1000 character chunks with 100 character overlap
- **Embeddings**: OpenAI (1536 dims) or HuggingFace (384 dims)
- **Search**: Cosine similarity in Pinecone, threshold 0.7
- **Top-K**: Default 5 chunks, configurable

### Security
- API keys in `appsettings.json` (use Key Vault in production)
- CORS configured for `http://localhost:4200`
- Input validation on all endpoints
- File type validation (PDF, DOCX only)

## Configuration

### API Keys Required
- `OpenAI:ApiKey` - For LLM and embeddings
- `Pinecone:ApiKey`, `Pinecone:Environment`, `Pinecone:IndexName`
- `HuggingFace:ApiKey` - Optional, for alternative embeddings

### Key Endpoints
- `POST /api/v1/documents/upload` - Upload document
- `GET /api/v1/documents` - List documents
- `POST /api/v1/chat/ask` - Ask question

## Build & Run
- **Backend**: `dotnet run` in AmaRAGBackend (port 5000)
- **Frontend**: `ng serve` in AmaRAGUI (port 4200)
- **API Docs**: https://localhost:5000/swagger

## Quick Start
See [QUICKSTART.md](../QUICKSTART.md) for rapid setup instructions.

## Deployment Notes
- Backend: Container (Docker) or Azure App Service
- Frontend: Azure Static Web Apps, Netlify, or similar
- Environment variables for API keys in production

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rajatmaroon5) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
