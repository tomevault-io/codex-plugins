# 👉 Code-generation instructions
- Always target **FastAPI** with **Python 3.12** for backend API development.
- Use **Semantic Kernel** for AI orchestration and agent management patterns.
- Follow **async/await** patterns throughout - agents, API endpoints, and Azure SDK calls.
- Use **Pydantic** models for request/response validation and configuration management.
- Implement **streaming responses** for chat endpoints using `StreamingResponse`.
- Use **type hints** throughout and include Google-style docstring summaries.
- Use `logging` (INFO level) instead of `print` for proper observability.
- Access secrets via `os.environ[...]` or Pydantic Settings; never commit literals.
- Default to **Python 3.12** features (e.g., `match` statements, type union syntax).
- When calling Azure SDKs, prefer their **async** variants (`BlobServiceClient`, `SearchClient`, etc.).
- Return HTTP results with FastAPI's native response types (`JSONResponse`, `StreamingResponse`).

# 👉 Frontend code-generation instructions  
- Use **React 18** with **TypeScript** and **Vite** for frontend development.
- Implement components with **Radix UI** primitives and **Tailwind CSS** for styling.
- Use **React hooks** (`useState`, `useEffect`, `useCallback`) for state management.
- Follow **Microsoft design patterns** with appropriate color schemes and spacing.
- Create **reusable components** in the `components/` directory with proper TypeScript interfaces.
- Implement **streaming chat interfaces** with proper loading states and error handling.
- Use **custom hooks** for API calls and complex state logic.
- Structure pages in `pages/` directory following the existing routing patterns.

# 👉 Agent and RAG pattern instructions
- Build **agent classes** that extend the base patterns (Retriever, Writer, Verifier, Curator).
- Use **Semantic Kernel** for AI orchestration and multi-agent workflows.
- Implement **RAG patterns**: Context-Aware Generation, Agentic QA with Verification, Adaptive KB Management.
- Follow **async patterns** for document processing, vector search, and LLM calls.
- Structure agents in `backend/app/agents/` with proper registration in `agent_configs.yaml`.
- Use **Azure AI Search** for vector storage and **Azure Document Intelligence** for processing.
- Implement **citation tracking** and **confidence scoring** for verification workflows.

# 👉 Test-generation instructions
- Use **pytest** + **pytest-asyncio** for backend testing with proper async test patterns.
- Use **FastAPI TestClient** for API endpoint testing and validation.
- Mock Azure SDK calls with `unittest.mock.AsyncMock` for isolated testing.
- Test **agent workflows** with proper mocking of LLM calls and Azure services.
- Aim for comprehensive coverage of RAG patterns, document processing, and API endpoints.
- Keep test payloads realistic but small; embed them inline rather than external files.
- For frontend, use **React Testing Library** + **Jest** for component and integration tests.
- Test **streaming responses** and **error handling** patterns thoroughly.

# 👉 Code-review instructions
- Verify **async/await** patterns are used correctly throughout the codebase.
- Ensure **agent registration** is properly configured in `agent_configs.yaml`.
- Check for proper **error handling** in streaming responses and agent workflows.
- Confirm **environment variables** are used for all Azure service configurations.
- Validate **type hints** and **Pydantic models** are properly defined.
- Review **RAG pattern implementations** for proper citation tracking and verification.
- Check **frontend components** follow Microsoft design patterns and accessibility standards.
- Ensure **Azure SDK** calls use async variants and proper credential management.

# 👉 Commit-message generation instructions
- Follow **Conventional Commits** (`feat:`, `fix:`, `test:`, `chore:` …).
- First line ≤ 72 chars, present-tense imperative.
- Include a short body describing *why* and reference issues `(fixes #123)`.

# 👉 Pull-request title/description instructions
- **Title**: Imperative, ≤ 60 chars (e.g., “feat: add vector search caching", "fix: resolve streaming timeout”).
- **Description**:  
  1. *What* & *why* (one paragraph explaining the RAG pattern or feature change).  
  2. “### Changes” bullet list.  
  3. “### Testing” explaining manual testing of RAG workflows and automated test coverage.  
  4. “### Notes” for new Azure services, environment variables, or breaking changes to agent configurations.

---
> Source: [Azure-Samples/adaptive-rag-workbench](https://github.com/Azure-Samples/adaptive-rag-workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-06 -->
