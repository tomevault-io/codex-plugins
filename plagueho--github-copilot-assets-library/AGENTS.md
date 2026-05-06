This is a GitHub Copilot Assets Library that provides customization assets including Copilot Instructions, Prompt Files, Custom Chat Modes, and MCP Server configurations.

## Core Commands

- **Documentation**: Use README.md as the primary source - contains comprehensive asset descriptions
- **No build/test commands**: This is a documentation/asset library with no compiled code
- **File validation**: Ensure Markdown syntax is valid for `.chatmode.md`, `.prompt.md` and `.instruction.md` files

## Repository Architecture

### Major Components
- `.github/copilot-instructions.md` - Repository-level Copilot instructions (n)
- `/prompts/` - Reusable prompt files for common tasks
- `/chatmodes/` - Custom chat modes with specialized instructions and tools
- `/instructions/` - Custom instruction files for Copilot
- `/mcp/` - Sample Model Context Protocol (MCP) server configurations for external integrations
- `.devcontainer/` - Development container for standardized environment

### Key Asset Types
- **Prompt Files**: Task-specific prompts (create specs, GitHub issues, code reviews)
- **Chat Modes**: Specialized modes (expert engineers, planners, mentors, Azure modules)
- **MCP Servers**: External integrations (Microsoft Docs, GitHub, Playwright, Giphy, Azure DevOps)

## Style Rules

### File Naming
- Chat modes: `[descriptive-name].chatmode.md`
- Prompts: `[descriptive-name].prompt.md`
- Instructions: `[descriptive-name].instructions.md`
- Use lowercase with hyphens for file names

### Markdown Standards
- Follow front matter YAML structure exactly as shown in examples
- Use proper heading hierarchy (# ## ###)
- Include description and tools array in chat mode front matter
- Include mode and description in prompt file front matter

### Content Guidelines
- **Precise Language**: Use explicit, unambiguous instructions
- **Structured Format**: Use headings, bullets, tables for clarity
- **No Boilerplate**: Avoid generic advice, focus on specific guidance
- **Self-Contained**: Each asset should be complete and context-independent

## Repository-Specific Rules

### Chat Mode Development
- Always include comprehensive tool arrays based on functionality needs
- Structure instructions in clear sections with specific guidance
- Reference industry experts/thought leaders for authority (e.g., Martin Fowler, Uncle Bob)
- Specify target frameworks and versions explicitly

### Prompt File Standards
- Include variable substitution using `${input:VariableName}` syntax
- Provide clear task descriptions and expected outcomes
- Follow specification template structure for consistency
- Include examples and edge cases where applicable

### Commit Message Format
Follow VSCode settings pattern:
- First line: `[TYPE]: [50 char summary]` where TYPE is CHORE|FIX|CHANGE|BREAKING CHANGE|TESTS|SECURITY|COMPLEX
- Second line: blank
- Following lines: detailed summary with `-` bullets, prefix with `SECURITY:` or `BREAKING CHANGE:` as needed

### Development Practices
- Use .NET 8+ patterns and modern C# features when applicable
- Follow Azure Well-Architected Framework pillars for infrastructure guidance
- Prioritize security, operational excellence, performance, reliability, cost optimization
- Reference Microsoft documentation and packages preferentially
- Use MSTest, FluentAssertions, Moq for testing examples

This library serves as a reference implementation for GitHub Copilot customizations across various development scenarios.

## Code Review
When performing a code review:
- validate that there are changes in the `README.md` file that match the changes in the pull request. If there are no changes, or if the changes do not match, then the pull request is not ready to be merged.
- ensure that the values in the front matter are wrapped in single quotes.
- ensure that the `description` field in the front matter is not empty.
- on a `.instructions.md` file, ensure there is an `applyTo` property in the front matter that specifies the file or files to which the instructions apply.

---
> Source: [PlagueHO/github-copilot-assets-library](https://github.com/PlagueHO/github-copilot-assets-library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-06 -->
