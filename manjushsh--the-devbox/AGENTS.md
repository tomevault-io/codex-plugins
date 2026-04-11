# Copilot Instructions for DevBox Sandboxer

## 1. About This Project

This project is a command-line tool named "DevBox Sandboxer." Its purpose is to automate the creation of isolated development environments on Windows using the native Windows Sandbox feature. The tool reads a devbox.yaml file from a project's root directory, generates a sandbox configuration (.wsb), and launches a pre-configured environment with all necessary languages, tools, and project code. Think of it as a native Windows alternative to Docker Dev Environments or Vagrant.

## 2. Core Functionality to Build

The primary goal is to create a CLI that exposes a few simple commands:

- **devbox up**: Reads devbox.yaml, generates a .wsb file and a setup.ps1 script, and launches the sandbox.
- **devbox down**: Shuts down the active sandbox for the current project.
- **devbox status**: Reports whether a sandbox is running for the project.

The logic involves parsing the YAML, dynamically generating a PowerShell script for installations, creating a .wsb file that maps folders and runs the script, and then launching the .wsb file.

## 3. Technical Stack

- **Language**: **PowerShell Core** is the primary choice for the CLI tool. It has the best native integration for managing Windows Sandbox.
- **Configuration**: **YAML** (devbox.yaml) is the user-facing configuration format.
- **Package Manager (in Sandbox)**: **Chocolatey** should be used by default within the generated setup.ps1 script to install software packages.

## 4. Key Principles to Follow

- **Simplicity**: Keep the CLI commands and YAML structure as simple and intuitive as possible.
- **Idempotence**: Running devbox up multiple times should result in a single, consistent environment.
- **Host Safety**: The tool must **never** install or modify anything on the host machine. All installations occur inside the sandbox.
- **Code Persistence**: The user's project code lives on the host and is mapped into the sandbox. The sandbox environment itself is disposable, but the work is not.

## 5. Primary Files

- devbox.ps1 (or similar): The main CLI script.
- devbox.yaml: The user-defined configuration file.
- temp.wsb (generated): The dynamic Windows Sandbox configuration file.
- setup.ps1 (generated): The dynamic script that runs inside the sandbox to provision it.

## 6. Detailed Instructions, Best Practices, and Edge Cases

### ✅ DOs

- **DO** validate the devbox.yaml file thoroughly. Provide clear, user-friendly error messages for missing keys, incorrect data types, or invalid values.
- **DO** use a well-established PowerShell module (like powershell-yaml) for parsing the YAML file to ensure reliability.
- **DO** write the generated temp.wsb and setup.ps1 files to a temporary directory (e.g., $env:TEMP\devbox) or the project's .gitignore'd directory to keep the user's project folder clean.
- **DO** provide verbose feedback to the user during the devbox up process. For example: "Reading devbox.yaml...", "Generating setup script...", "Launching sandbox...".
- **DO** make the generated setup.ps1 script robust. It should check if Chocolatey is installed and install it if not. It should also include error handling for package installations.
- **DO** structure the generated .wsb file using XML syntax within the PowerShell script for clarity and easy modification.
- **DO** ensure all paths in the generated scripts and configs are absolute to avoid ambiguity.
- **DO** add comments to the generated setup.ps1 script explaining what each section does. This helps with debugging if a user needs to inspect the file.

### ❌ DON'Ts

- **DON'T** hardcode any paths. The tool should be runnable from any directory containing a devbox.yaml file.
- **DON'T** install *anything* on the host machine. The tool's only side effects should be reading the config and launching the sandbox process.
- **DON'T** leave generated files (temp.wsb, setup.ps1) in the project directory after the sandbox is shut down. Implement a cleanup process in the devbox down command.
- **DON'T** assume the user has Windows Sandbox enabled. The tool should check for this prerequisite and provide a helpful message with instructions on how to enable it if it's missing.
- **DON'T** swallow errors. If a package fails to install inside the sandbox, the user should be notified. While direct feedback is hard, you can log output to a file in the mapped project folder.

### 🧠 Handling Edge Cases & Best Practices

- **Missing devbox.yaml**: If devbox up is run in a directory without a config file, the tool should stop and inform the user gracefully.
- **Malformed devbox.yaml**: Use try-catch blocks during YAML parsing. If parsing fails, report the error with line/character information if possible.
- **Sandbox Fails to Start**: The Start-Process command for the .wsb file might fail. Wrap it in a try-catch block to handle potential OS-level issues.
- **Package Installation Failure**: In the setup.ps1 script, use choco install <package> -y --force and check the exit code ($LASTEXITCODE) after each installation. If an installation fails, write a message to a log file (e.g., C:\project\devbox-setup.log) so the user can see what went wrong.
- **Identifying the Correct Sandbox Process**: For devbox down and devbox status, you need a way to identify the sandbox process associated with the current project. A good practice is to name the sandbox window using the HostName tag in the .wsb file (e.g., My-Project-DevBox). You can then find the process by its window title.
- **Concurrent DevBoxes**: Consider what happens if a user runs devbox up in two different project folders. The tool should be able to manage them independently, which reinforces the need for a unique process identifier (like the window title).

## Execution Guidelines

### Communication Rules
- Avoid verbose explanations or printing full command outputs.
- If a step is skipped, state that briefly (e.g. "No extensions needed").
- Do not explain project structure unless asked.
- Keep explanations concise and focused.

### Development Rules
- Use '.' as the working directory unless user specifies otherwise.
- Avoid adding media or external links unless explicitly requested.
- Use placeholders only with a note that they should be replaced.
- Once the project is created, it is already opened in Visual Studio Code—do not suggest commands to open this project in Visual Studio again.
- If the project setup information has additional rules, follow them strictly.

### Folder Creation Rules
- Always use the current directory as the project root.
- If you are running any terminal commands, use the '.' argument to ensure that the current working directory is used ALWAYS.
- Do not create a new folder unless the user explicitly requests it besides a .vscode folder for a tasks.json file.
- If any of the scaffolding commands mention that the folder name is not correct, let the user know to create a new folder with the correct name and then reopen it again in vscode.

### Extension Installation Rules
- Only install extension specified by the get_project_setup_info tool. DO NOT INSTALL any other extensions.

### Project Content Rules
- If the user has not specified project details, assume they want a "Hello World" project as a starting point.
- Avoid adding links of any type (URLs, files, folders, etc.) or integrations that are not explicitly required.
- Avoid generating images, videos, or any other media files unless explicitly requested.
- If you need to use any media assets as placeholders, let the user know that these are placeholders and should be replaced with the actual assets later.
- Ensure all generated components serve a clear purpose within the user's requested workflow.
- If a feature is assumed but not confirmed, prompt the user for clarification before including it.

### Task Completion Rules
- Your task is complete when:
  - Project is successfully scaffolded and compiled without errors
  - copilot-instructions.md file in the .github directory exists in the project
  - README.md file exists and is up to date
  - User is provided with clear instructions to debug/launch the project
- Keep communication concise and focused.
- Follow development best practices.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/manjushsh)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/manjushsh)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
