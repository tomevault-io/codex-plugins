<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->
- [x] Verify that the copilot-instructions.md file in the .github directory is created. ✅ COMPLETED

- [x] Clarify Project Requirements ✅ COMPLETED
	<!-- MCP server for MLB data using TypeScript, capturing MLB API information for baseball statistics, player information, team data, game schedules, and live scores. -->

- [x] Scaffold the Project ✅ COMPLETED
	<!--
	✅ Created project structure with TypeScript configuration
	✅ Set up package.json with MCP SDK dependencies
	✅ Created main server file (src/index.ts) with MCP server implementation
	✅ Created MLB API client (src/mlb-api.ts) for interfacing with MLB Stats API
	✅ Implemented 6 core tools: get-standings, get-team-info, get-player-stats, get-schedule, get-live-game, search-players
	✅ Added comprehensive README with usage examples and API reference
	-->

- [x] Customize the Project ✅ COMPLETED
	<!--
	✅ Project customized for MLB data with comprehensive MCP server implementation
	✅ Added 6 core tools for MLB data access
	✅ Implemented proper error handling and structured responses
	✅ Created type-safe interfaces for MLB API data
	-->

- [x] Install Required Extensions ✅ COMPLETED
	<!-- No additional extensions required for this TypeScript MCP server project -->

- [x] Compile the Project ✅ COMPLETED
	<!--
	✅ Installed all dependencies successfully
	✅ Fixed TypeScript type issues with MCP SDK compatibility
	✅ Project compiles without errors
	✅ Build artifacts created in /build directory
	-->

- [x] Create and Run Task ✅ COMPLETED
	<!--
	✅ Created VS Code task for running the MLB MCP server
	✅ Task successfully executes "node build/index.js"
	✅ Server runs and displays "MLB MCP Server running on stdio"
	✅ Created .vscode/mcp.json configuration for MCP debugging
	 -->

- [x] Launch the Project ✅ COMPLETED
	<!--
	✅ Project successfully launched and tested
	✅ MCP server is running and responsive
	✅ Ready for integration with MCP clients like Claude Desktop
	 -->

- [x] Ensure Documentation is Complete ✅ COMPLETED
	<!--
	✅ README.md created with comprehensive documentation
	✅ Includes installation, usage, examples, and API reference
	✅ copilot-instructions.md updated with project completion status
	✅ All documentation reflects current project state
	 -->

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
- Use VS Code API tool only for VS Code extension projects.
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
- Work through each checklist item systematically.
- Keep communication concise and focused.
- Follow development best practices.

## MCP Server for MLB Data

This is a Model Context Protocol server that provides access to MLB (Major League Baseball) data through the MLB API. The server offers tools for retrieving team information, player statistics, game schedules, live scores, and historical data.

### MCP SDK Documentation References:
- Main MCP Repository: https://github.com/modelcontextprotocol
- TypeScript SDK: https://github.com/modelcontextprotocol/typescript-sdk
- Documentation: https://modelcontextprotocol.io/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Datamazin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
