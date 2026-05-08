# Commit Message Instructions

Generate commit messages in the following format:
```
[type]: [short description of all changes]
🟢 [file name] for each new file
🛠️ [file name] -> [brief description of change] for each modified file
🔴 [file name] for each deleted file
```

- **Type**: Choose one of `feat`, `refac`, `counter`, `fix`, `dev`, `deploy` based on the change:
  - `feat`: New feature or addition.
  - `refac`: Code refactoring without functional changes.
  - `counter`: Updates to counters or metrics.
  - `fix`: Bug fixes.
  - `dev`: Development-related changes (e.g., tooling, scripts).
  - `deploy`: Deployment-related changes.
- **Short description**: Summarize all changes in 50-72 characters, using present tense verbs.
- **File details**:
  - Use `🟢` for new files, followed by the file name.
  - Use `🛠️` for modified files, followed by the file name and a brief description of the change (e.g., "add user auth logic").
  - Use `🔴` for deleted files, followed by the file name.
- Ensure the message is clear, concise, and lists all affected files.

Example:
```
feat: Add user authentication
🟢 src/auth.js
🛠️ src/app.js -> Add auth middleware
🔴 src/old-auth.js

---
> Source: [Bridge-Packages/go-openai-realtime](https://github.com/Bridge-Packages/go-openai-realtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-08 -->
