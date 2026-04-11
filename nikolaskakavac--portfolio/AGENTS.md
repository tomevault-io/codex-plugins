<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->
- [x] Verify that the copilot-instructions.md file in the .github directory is created.

- [x] Clarify Project Requirements
	<!-- Ask for project type, language, and frameworks if not specified. Skip if already provided. -->

- [x] Scaffold the Project
	<!--
	Ensure that the previous step has been marked as completed.
	Call project setup tool with projectType parameter.
	Run scaffolding command to create project files and folders.
	Use '.' as the working directory.
	If no appropriate projectType is available, search documentation using available tools.
	Otherwise, create the project structure manually using available file creation tools.
	-->

- [ ] Customize the Project
	<!--
	Verify that all previous steps have been completed successfully and you have marked the step as completed.
	Develop a plan to modify codebase according to user requirements.
	Apply modifications using appropriate tools and user-provided references.
	Skip this step for "Hello World" projects.
	-->

- [x] Install Required Extensions
	No extensions required for this project.

- [x] Compile the Project
	Dependencies installed successfully. Dev server (Vite v4) verified.

- [x] Create and Run Task
	Dev server verified with `npm run dev`.

- [x] Launch the Project
	Dev server ready at http://localhost:5173/

- [x] Ensure Documentation is Complete
	README.md and copilot-instructions.md completed and cleaned.

<!--
## Execution Guidelines
PROGRESS TRACKING:
- If any tools are available to manage the above todo list, use it to track progress through this checklist.
- After completing each step, mark it complete and add a summary.
- Read current todo list status before starting each new step.

COMMUNICATION RULES:
- Avoid verbose explanations or printing full command outputs.
- If a step is skipped, state that briefly (e.g. "No extensions needed").
- Do not explain project structure unless asked.
- Keep explanations concise and focused.

DEVELOPMENT RULES:
- Use '.' as the working directory unless user specifies otherwise.
- Avoid adding media or external links unless explicitly requested.
- Use placeholders only with a note that they should be replaced.
- Once the project is created, it is already opened in Visual Studio Code—do not suggest commands to open this project in Visual Studio again.
- If the project setup information has additional rules, follow them strictly.

FOLDER CREATION RULES:
- Always use the current directory as the project root.
- If you are running any terminal commands, use the '.' argument to ensure that the current working directory is used ALWAYS.
- Do not create a new folder unless the user explicitly requests it besides a .vscode folder for a tasks.json file.
- If any of the scaffolding commands mention that the folder name is not correct, let the user know to create a new folder with the correct name and then reopen it again in vscode.

EXTENSION INSTALLATION RULES:
- Only install extension specified by the get_project_setup_info tool. DO NOT INSTALL any other extensions.

PROJECT CONTENT RULES:
- If the user has not specified project details, assume they want a "Hello World" project as a starting point.
- Avoid adding links of any type (URLs, files, folders, etc.) or integrations that are not explicitly required.
- Avoid generating images, videos, or any other media files unless explicitly requested.
- If you need to use any media assets as placeholders, let the user know that these are placeholders and should be replaced with the actual assets later.
- Ensure all generated components serve a clear purpose within the user's requested workflow.
- If a feature is assumed but not confirmed, prompt the user for clarification before including it.
- If you are working on a VS Code extension, use the VS Code API tool with a query to find relevant VS Code API references and samples related to that query.

TASK COMPLETION RULES:
- Your task is complete when:
  - Project is successfully scaffolded and compiled without errors
  - copilot-instructions.md file in the .github directory exists in the project
  - README.md file exists and is up to date
  - User is provided with clear instructions to debug/launch the project

Before starting a new task in the above plan, update progress in the plan.
-->

## Project Specifications

- **Type**: Bilingual Portfolio (SR/EN)
- **Framework**: React 18 + Vite v4
- **Design**: Modern minimal with gradient accents
- **Sections**: Hero, Services (5), Work (3), Process (4), Testimonials, About, Contact, Footer
- **Language Toggle**: React state-based SR/EN switch
- **Styling**: CSS Grid/Flexbox, CSS variables, animations

## Customization Checklist

Before deploying:
1. Replace `hello@yourdomain.com` with your email in App.jsx
2. Update social links and real project descriptions in Work section
3. Add real testimonials
4. Upload project screenshots to `src/assets/`
5. Adjust colors in `:root` CSS variables if desired
6. Review and update About section bio
7. Set up email form backend (Formspree, Netlify Forms, etc.)

## Running Commands

- **Dev**: `npm run dev` → http://localhost:5173
- **Build**: `npm run build` → `dist/` folder (requires Node 20.19+ or 22.12+)
- **Preview**: `npm run preview`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nikolaskakavac)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/nikolaskakavac)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
