- [x] Verify that the copilot-instructions.md file in the .github directory is created.
- [x] Clarify Project Requirements
- [x] Scaffold the Project
- [x] Customize the Project
- [x] Install Required Extensions
- [x] Compile the Project
- [x] Create and Run Task
- [ ] Launch the Project
- [ ] Ensure Documentation is Complete

## Execution Guidelines
PROGRESS TRACKING:
- Use available tools to manage the above todo list and mark steps complete with a summary.
- Read the current todo status before starting a new step.

COMMUNICATION RULES:
- Avoid verbose explanations or full command outputs.
- If a step is skipped, state that briefly.
- Do not explain project structure unless asked.

DEVELOPMENT RULES:
- Use '.' as the working directory unless the user specifies otherwise.
- Avoid adding media or external links unless explicitly requested.
- Use placeholders only with a note that they should be replaced.
- If you need to use media assets as placeholders, note that they should be replaced later.
- Ensure all generated components serve a clear purpose within the user's requested workflow.
- If a feature is assumed but not confirmed, prompt the user for clarification before including it.
- If working on a VS Code extension, use the VS Code API tool with a query to find relevant references and samples.

FOLDER CREATION RULES:
- Always use the current directory as the project root.
- When running terminal commands, use the '.' argument to keep the current working directory.
- Do not create a new folder unless the user explicitly requests it, besides a .vscode folder for a tasks.json file.
- If a scaffolding command complains about folder naming, ask the user to create a correctly named folder and reopen it in VS Code.

EXTENSION INSTALLATION RULES:
- Only install extensions specified by the get_project_setup_info tool. Do not install other extensions.

PROJECT CONTENT RULES:
- If the user has not specified project details, assume a "Hello World" project as a starting point.
- Avoid adding links or integrations that are not explicitly required.
- Avoid generating images, videos, or other media files unless explicitly requested.
- If media assets are used as placeholders, note that they should be replaced with the actual assets later.
- Ensure all generated components serve a clear purpose within the user's requested workflow.
- If a feature is assumed but not confirmed, prompt the user for clarification before including it.
- If working on a VS Code extension, use the VS Code API tool with a query to find relevant references and samples.

TASK COMPLETION RULES:
- Task is complete when the project is scaffolded and compiled without errors, copilot-instructions.md exists, README.md is up to date, and the user has clear instructions to debug/launch the project.
- Before starting a new task in the above plan, update progress in the plan.

- Work through each checklist item systematically.
- Keep communication concise and focused.
- Follow development best practices.

---
> Source: [Cevin2107/exams](https://github.com/Cevin2107/exams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-30 -->
