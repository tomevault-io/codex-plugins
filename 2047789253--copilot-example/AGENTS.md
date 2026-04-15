# Copilot Customization Example Project

This workspace demonstrates GitHub Copilot's customization capabilities including skills, instructions, prompts, hooks, and custom agents.

## Project Structure

```
.github/
├── copilot-instructions.md               # This file - global guidance for all tasks
├── MCP_INTEGRATION.md                    # Guide for MCP server setup and usage
├── skills/
│   └── remotion/SKILL.md                 # Skill for building videos with React
├── prompts/
│   └── generate-unit-tests.prompt.md     # Prompt for generating test cases
├── agents/
│   ├── code-reviewer.agent.md            # Custom agent for code review
│   └── remotion-video-builder.agent.md   # Custom agent for video creation with MCP tools
├── mcp-servers/
│   └── remotion-mcp/                     # Local MCP server for Remotion tools
│       ├── index.js                      # MCP server implementation
│       ├── package.json                  # Dependencies
│       └── README.md                     # Setup instructions
├── instructions/                         # (Optional) File-specific instructions
└── hooks/                                # (Optional) Lifecycle automation
```

## Current Capabilities

### Available Skills

**Remotion** (`/remotion`) - Use when creating videos programmatically with React

- Quick start commands
- Core concepts (frames, fps, animations)
- Common patterns (multi-scene, text animation, media handling)
- Rendering output formats
- Troubleshooting guide

Type `/` in chat to access this skill.

### Available Prompts

**Generate Unit Tests** (`/generate-unit-tests`) - Generate comprehensive test cases for any function or class

- Happy path scenarios
- Edge cases and boundary conditions
- Error handling and exceptions
- Mocking for external dependencies
- Multi-language support (JavaScript, Python, Java, Go, etc.)

Use this prompt to quickly generate test cases for code snippets.

### Available Custom Agents

**Code Reviewer** (`code-reviewer` agent) - Strict code review with bug hunting and security analysis

- Identifies bugs, logic errors, and crashes
- Spots security vulnerabilities (injection, auth, data exposure)
- Suggests performance optimizations
- Enforces code quality standards
- Provides actionable feedback with examples

Select this agent when you need detailed code review feedback. It has read-only access (no editing) to maintain reviewer independence.

**Remotion Video Builder** (`remotion-video-builder` agent) - Create and render Remotion videos with MCP tools

- Writes React video components
- Calls Remotion MCP rendering tools
- Validates compositions
- Generates MP4, WebM, and other formats
- Provides performance guidance

Select this agent when you want to create or render videos programmatically.

## Development Guidelines

### Code Style

- Use **2-space indentation** for all files
- Follow **Prettier** for formatting (if applicable)
- Keep lines under **100 characters** when possible
- Use **TypeScript** for new JavaScript projects (prefer `.ts` over `.js`)

### File Naming

- Components: PascalCase (e.g., `VideoComponent.tsx`)
- Utilities/helpers: camelCase (e.g., `formatTime.ts`)
- Constants: UPPER_SNAKE_CASE (e.g., `API_BASE_URL`)

### Documentation

- Keep README.md up-to-date with project status
- Document setup steps in contributing guides if expanding this project
- Use JSDoc comments for exported functions
- Link to relevant skill files when they apply

## How to Use This Project

### 1. Ask About Remotion

"Help me create a 5-second video intro with Remotion"  
"How do I animate text fading in Remotion?"  
"Show me the best way to add background music to a Remotion video"

Copilot will automatically load the Remotion skill.

### 2. Extend with More Skills

To add a new skill (e.g., Testing, Database Migrations):

- Create `.github/skills/<skill-name>/SKILL.md`
- Include quick start, core concepts, common patterns, and troubleshooting
- Reference additional scripts/templates in subdirectories
- Use clear descriptions to help discovery

### 3. Add File-Specific Instructions

For language-specific or framework-specific guidelines:

- Create `.github/instructions/<name>.instructions.md`
- Use `applyTo` glob patterns (e.g., `**/*.py`, `src/components/**`)
- Keep focused on one concern per file

### 4. Create Reusable Prompts

For single-task templates you use repeatedly:

- Create `.github/prompts/<name>.prompt.md`
- Include clear input/output formats
- Add `argument-hint` for usage guidance

### 5. Define Custom Agents

For orchestrated workflows with role-based tool restrictions:

- Create `.github/agents/<name>.agent.md`
- Specify tool access levels
- Define clear constraints and responsibilities

### 6. Enforce with Hooks

For deterministic automation (prevent dangerous operations, auto-validate):

- Create `.github/hooks/<name>.json`
- Use lifecycle events: `PreToolUse`, `PostToolUse`, `SessionStart`, `Stop`
- Reference validation scripts

## Best Practices

✅ **DO**

- Keep instructions keyword-rich for AI discovery
- Link to docs instead of copying content
- Separate concerns across files
- Update customizations when practices change
- Use relative paths in skill resources (`./scripts/`, `./templates/`)

❌ **DON'T**

- Use both `copilot-instructions.md` AND `AGENTS.md` (pick one)
- Make global instructions too broad (`applyTo: "**"` should be rare)
- Duplicate documentation—reference instead
- Create monolithic SKILL.md files—use resource subdirectories
- Embed credentials or secrets in instruction files

## MCP Integration

This workspace has Git integration enabled via MCP:

- Search GitHub repos (`mcp_my-mcp-server_search_repositories`)
- View GitHub repo contents and README
- Get file listings and commit history
- Useful for referencing external projects or examples

### Remotion MCP Server

A local MCP server provides direct access to Remotion rendering tools:

- **remotion_render**: Render compositions to MP4/WebM
- **remotion_preview**: Start local preview server
- **remotion_list_compositions**: List available compositions
- **remotion_validate**: Validate composition configs

**Setup**:

```bash
cd .github/mcp-servers/remotion-mcp
npm install
```

Then use the `remotion-video-builder` agent to access these tools.

See [MCP Integration Guide](.github/MCP_INTEGRATION.md) for details.

## Next Steps

1. **Setup Remotion MCP Server**:

   ```bash
   cd .github/mcp-servers/remotion-mcp
   npm install
   ```

   Then restart VS Code for Copilot to load the server.

2. **Try the Remotion skill**: Type `/` and select `remotion` to learn about Remotion

3. **Generate tests**: Type `/` and select `generate-unit-tests` to create test cases for your code

4. **Get a code review**: Select the `code-reviewer` agent when you want detailed code analysis

5. **Create videos**: Select the `remotion-video-builder` agent to create and render videos with MCP tools

6. **Add more customizations**: Create additional skills, prompts, or agents following the patterns above

7. **Document your workflow**: Update this file as your customization strategy evolves

## Resources

- [VS Code Copilot Customization Docs](https://code.visualstudio.com/docs/copilot/customization)
- [Skills Reference](https://code.visualstudio.com/docs/copilot/customization/agent-skills)
- [Instructions Reference](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [Prompts Reference](https://code.visualstudio.com/docs/copilot/customization/prompt-files)
- [Custom Agents Reference](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- [Hooks Reference](https://code.visualstudio.com/docs/copilot/customization/hooks)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/2047789253) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
