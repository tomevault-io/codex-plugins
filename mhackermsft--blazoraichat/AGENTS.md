# GitHub Copilot Repository Instructions

## Project Overview
This repository contains a .NET 9 Blazor Server application for AI-powered chat using Azure OpenAI. Key features include:
- Chat with Azure OpenAI models (GPT-4o, embeddings)
- Retrieval-augmented generation (RAG) over uploaded documents (TXT, DOCX, XLSX, PPTX, PDF, MD)
- Image upload and analysis (OCR via Azure Document Intelligence if model does not support images)
- EasyAuth authentication and user-specific knowledge base
- Streaming chat responses, retry logic, and chat history management
- Optional integration with Azure AI Search and PostgreSQL for knowledge storage
- Model Context Protocol (MCP) server support (experimental)

## Project Structure
- **Blazor Components**: Located in `BlazorAIChat/Components/` (Pages, Shared, Layout)
- **Services**: Business logic and integrations in `BlazorAIChat/Services/`
- **Models**: Data models in `BlazorAIChat/Models/`
- **Authentication**: Middleware and logic in `BlazorAIChat/Authentication/`
- **Utils**: Utility classes in `BlazorAIChat/Utils/`
- **Migrations**: Entity Framework migrations in `BlazorAIChat/Migrations/`
- **Static Assets**: JS libraries (e.g., dompurify) in `BlazorAIChat/wwwroot/Lib/`
- **Entry Point**: Application startup in `BlazorAIChat/Program.cs`

## Copilot Guidance
- Prioritize Blazor Server code patterns (not Razor Pages or MVC)
- Use .NET 9 features and APIs
- For authentication, prefer EasyAuth and cookie-based auth
- For document handling, support all listed formats and image OCR fallback
- For RAG, use semantic search and chunking as described in README
- For chat, support streaming responses and context window management
- For knowledge storage, support file system, PostgreSQL, and Azure AI Search
- For MCP, follow the configuration pattern in README
- Use dependency injection for services and EF Core for data access
- Follow existing naming conventions and folder structure
- Add comments and documentation for new features

## Maintenance
- **Important:** Update this `copilot-instructions.md` file with new instructions as the project evolves, especially when adding new features, changing architecture, or updating dependencies.
- Reference: https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot

---
Last updated: 2024-06

---
> Source: [mhackermsft/BlazorAIChat](https://github.com/mhackermsft/BlazorAIChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-03 -->
