# 👉 Code-generation instructions
- Always target **Azure Functions Python v2** (`azure-functions >= 1.23`, **blueprint model**).
- Create the global `app = func.FunctionApp()` once and call `app.register_blueprint(bp)`.
- **Durable orchestrators** must be `def` (sync) and use `yield` + `context.task_all(...)`.
- **Activity functions** should be `async def` and return JSON-serialisable data.
- Use **type hints** throughout and a one-line Google-style docstring summary.
- Use `logging` (level INFO) instead of `print`.
- Access secrets via `os.environ[...]`; never commit literals.
- Default to **Python 3.11** features (e.g., `match` statements are allowed).
- When calling Azure SDKs, prefer their **async** variants (`BlobServiceClient.from_connection_string`, etc.).
- Return HTTP results with `azure.functions.HttpResponse` and **do not** `await` sync methods (`req.get_json()`, `client.create_check_status_response()`).

# 👉 Test-generation instructions
- Use **pytest** + **pytest-asyncio**.
- Mark async tests with `@pytest.mark.asyncio`.
- Patch Azure SDK calls with `unittest.mock.AsyncMock / MagicMock`.
- Aim for 100 % branch coverage of orchestrator fan-out/fan-in paths.
- Keep test payloads small; embed them inline rather than external files.

# 👉 Code-review instructions
- Verify no synchronous APIs are accidentally `await`ed.
- Ensure every orchestrator is registered in a blueprint and yields on tasks.
- Check for missing logging, un-handled exceptions, or secret leakage.
- Confirm environment-variable names match those used in `local.settings.json`.
- Flag any direct network calls inside orchestrators (should be in activities).

# 👉 Commit-message generation instructions
- Follow **Conventional Commits** (`feat:`, `fix:`, `test:`, `chore:` …).
- First line ≤ 72 chars, present-tense imperative.
- Include a short body describing *why* and reference issues `(fixes #123)`.

# 👉 Pull-request title/description instructions
- **Title**: Imperative, ≤ 60 chars (e.g., “feat: add Cosmos vector search”).
- **Description**:  
  1. *What* & *why* (one paragraph).  
  2. “### Changes” bullet list.  
  3. “### Testing” explaining manual/automated tests.  
  4. “### Notes” for env vars, breaking changes, or deployment steps.

---
> Source: [Azure-Samples/snippy](https://github.com/Azure-Samples/snippy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-17 -->
