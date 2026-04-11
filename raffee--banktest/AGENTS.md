- [x] Verify that the copilot-instructions.md file in the .github directory is created. (Completed Oct 29, 2025)

- [x] Clarify Project Requirements (C# banking integration webhook handler confirmed Oct 29, 2025)
	Ask for project type, language, and frameworks if not specified. Skip if already provided.

- [x] Scaffold the Project (BankingIntegration solution + projects scaffolded Oct 29, 2025)
	Ensure that the previous step has been marked as completed.
	Call project setup tool with projectType parameter.
	Run scaffolding command to create project files and folders.
	Use '.' as the working directory.
	If no appropriate projectType is available, search documentation using available tools.
	Otherwise, create the project structure manually using available file creation tools.

- [x] Customize the Project (Webhook handler and domain logic implemented Oct 29, 2025)
	Verify that all previous steps have been completed successfully and you have marked the step as completed.
	Develop a plan to modify the codebase according to user requirements.
	Apply modifications using appropriate tools and user-provided references.
	Skip this step for "Hello World" projects.

- [x] Install Required Extensions (No extensions required Oct 29, 2025)
	Only install extensions mentioned in the get_project_setup_info tool. Skip this step otherwise and mark as completed.

- [x] Compile the Project (dotnet test succeeded Oct 29, 2025)
	Verify that all previous steps have been completed.
	Install any missing dependencies.
	Run diagnostics and resolve any issues.
	Check for markdown files in the project folder for relevant instructions on how to do this.

- [x] Create and Run Task (Not required for dotnet CLI workflow Oct 29, 2025)
	Verify that all previous steps have been completed.
	Check https://code.visualstudio.com/docs/debugtest/tasks to determine if the project needs a task. If so, use the create_and_run_task tool to create and launch a task based on package.json, README.md, and project structure.
	Skip this step otherwise.

- [ ] Launch the Project
	Verify that all previous steps have been completed.
	Prompt the user for debug mode, launch only if confirmed.

- [x] Ensure Documentation is Complete (README refreshed and instructions cleaned Oct 29, 2025)
	Verify that all previous steps have been completed.
	Verify that README.md and this file exist and contain current project information.
	Remove outdated guidance from this checklist and keep it aligned with the repository.

## Execution Guidelines

Progress Tracking:
- If any tools are available to manage the above todo list, use them to track progress through this checklist.
- After completing each step, mark it complete and add a summary.
- Read the current todo list status before starting each new step.

Communication Rules:
- Avoid verbose explanations or printing full command outputs.
- If a step is skipped, state that briefly (for example, "No extensions needed").
- Do not explain project structure unless asked.
- Keep explanations concise and focused.

Development Rules:
- Use '.' as the working directory unless the user specifies otherwise.
- Avoid adding media or external links unless explicitly requested.
- Use placeholders only with a note that they should be replaced.
- Use the VS Code API tool only for VS Code extension projects.
- Once the project is created, it is already opened in Visual Studio Code—do not suggest commands to open this project again.
- If the project setup information has additional rules, follow them strictly.

Folder Creation Rules:
- Always use the current directory as the project root.
- If you are running any terminal commands, use the '.' argument to ensure that the current working directory is used.
- Do not create a new folder unless the user explicitly requests it (besides a .vscode folder for a tasks.json file).
- If any scaffolding command mentions that the folder name is not correct, inform the user so they can create a new folder with the correct name and reopen it in VS Code.

Extension Installation Rules:
- Only install extensions specified by the get_project_setup_info tool. Do not install any other extensions.

Project Content Rules:
- If the user has not specified project details, assume they want a "Hello World" project as a starting point.
- Avoid adding links of any type (URLs, files, folders, etc.) or integrations that are not explicitly required.
- Avoid generating images, videos, or any other media files unless explicitly requested.
- If you need to use any media assets as placeholders, note that these should be replaced with the actual assets later.
- Ensure all generated components serve a clear purpose within the user's requested workflow.
- If a feature is assumed but not confirmed, prompt the user for clarification before including it.
- If you are working on a VS Code extension, use the VS Code API tool with a query to find relevant VS Code API references and samples related to that query.

Task Completion Rules:
- The task is complete when:
	- The project is successfully scaffolded and compiled without errors.
	- The copilot-instructions.md file exists in the project.
	- README.md exists and is up to date.
	- The user is provided with clear instructions to debug or launch the project.
- Before starting a new task in the above plan, update progress in the plan.
- Work through each checklist item systematically.
- Keep communication concise and focused.
- Follow development best practices.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Raffee)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Raffee)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
