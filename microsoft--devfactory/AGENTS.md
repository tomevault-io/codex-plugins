# DevFactory Project - General Development Guidelines

## Project Overview

DevFactory is a **modular Infrastructure as Code (IaC) project** for streamlining Azure development environment setup with consistent patterns and best practices.

## **CORE RULES**

### Reference Specific File Type Instructions
Check the files in `/.github/instructions/*.instructions.md` for any additional instructions based on the file you are working on. Check each file in this folder and check `applyTo: '**/*.<extension>` to see which files the instructions apply to. For example:
- If `tf.instructions.md` exists and it contains`applyTo: '**/*.tf'`, then follow instructions for creating/editing Terraform `.tf` files.
- If `md.instructions.md` exists and it contains `applyTo: '**/*.md'`, then follow instructions for creating/editing Markdown `.md` files.
- and so on...

**REMINDER**: If you are creating a new file, follow the above instructions as you create this file. If you didn't, review the file and modify it to follow the instructions in the relevant `.instructions.md` file.


### File and Folder Structure
- When you are creating new files or folders, follow the standards in #file:../docs/file_structure.md
- If you need a reference for what each file or folder does, refer to #file:../docs/file_structure.md
- When you create a new file in the `/` root folder or the `docs/` folder, update #file:../docs/file_structure.md with a brief decription of the file's purpose and any relevant details.
- When you create a new folder, update #file:../docs/file_structure.md with a brief description of the folder's purpose and any relevant details.

### **ALWAYS** Document Changes
**ALL CODE CHANGES** must be documented in #file:../CHANGES_SUMMARY.md including:
- Brief description of the change
- Classification: bug fix, feature, or improvement  
- **Breaking change assessment** (YES/NO with justification)

### **ALWAYS** Follow Modular Patterns
- Consistent file structure across all modules
- Proper input validation and testing

## Communication Standards

**BE DIRECT AND PRAGMATIC**:
- Provide factual, actionable guidance
- Avoid hyperbole and excitement - stick to technical facts
- Think step-by-step and revalidate before responding
- Ensure responses are relevant to the codebase context

**AVOID**:
- Unnecessary apologizing or conciliatory statements
- Agreeing with users without factual basis ("You're right", "Yes")
- Verbose explanations when concise answers suffice

## CLI Standards

- When running commands, always do so from the root of the project directory. For example, if you are running `terraform init`, run it from the root of the project directory, not from within a module or subdirectory.
- When running commands, be mindful of the operating system as the paths and environment variables may differ. Use relative paths when possible.

## MCP Server Integration

**IF AVAILABLE**, use these Model Context Protocol servers:

- **Terraform MCP Server**: Seamless integration with Terraform Registry APIs for IaC automation
- **Azure MCP Server**: Connection to Azure services (Dev Center, Dev Box, Storage, Cosmos DB, etc.)

## Quality Standards

### **NEVER** Commit Without:
- Running validation tools (formatting, linting, testing)
- Testing examples to ensure they work  
- Updating relevant documentation
- Documenting changes in #file:../CHANGES_SUMMARY.md

### **ALWAYS** Ensure:
- Changes follow established project patterns
- New features include complete working examples
- Security considerations (no hardcoded credentials)
- Backward compatibility or proper breaking change documentation

---

**Project Goal**: Build a foundation for modern Azure development environments with high standards of quality, security, and maintainability.

---
> Source: [microsoft/devfactory](https://github.com/microsoft/devfactory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-05 -->
