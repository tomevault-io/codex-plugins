- [x] Verify that the copilot-instructions.md file in the .github directory is created.
- [x] Clarify Project Requirements
- [x] Scaffold the Project
- [x] Customize the Project
- [x] Install Required Extensions (none required)
- [x] Compile the Project
- [x] Create and Run Task (not needed)
- [x] Launch the Project (waiting on user request)
- [x] Ensure Documentation is Complete

Execution Guidelines
- Use manage_todo_list for multi-step work and update after each action.
- Keep explanations concise and avoid dumping full command output.
- Mention skipped steps briefly (e.g., “No extensions needed”).

Development Rules
- Use the workspace root (.) for commands and avoid creating extra folders.
- Limit placeholders and explain when one is used.
- Only call VS Code API docs for extension projects.
- Never revert user changes unless explicitly asked.

Folder Creation Rules
- Treat the current directory as the project root.
- Run CLI scaffolding directly in this folder.
- Only add .vscode/tasks.json when a task is required.

Extension Installation Rules
- Install only the extensions specified by get_project_setup_info.

Project Content Rules
- Default to Hello World if requirements are missing.
- Add external links or media only when requested or essential.
- Ensure every generated artifact supports the stated requirements.

Task Completion Rules
- Project scaffolded and compiled without errors.
- README.md and this instruction file stay current.
- Provide users with clear guidance to run or debug the project.
- Update this checklist before starting new work.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kuldeep-bhadouriya)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Kuldeep-bhadouriya)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
