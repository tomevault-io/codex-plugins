# GitHub Copilot Instructions

This file provides repository-wide custom instructions for GitHub Copilot to understand the project context, architecture, and development standards.

## ⚡ Performance Optimization for AI Agents

**IMPORTANT**: Before reading any documentation files, use query scripts to reduce token usage by 85-95%.

### Available Query Tools

Located in `tools/query-scripts/`:

- **`quick-ref.sh`** - Ultra-fast lookups (< 500 tokens)
  ```bash
  ./tools/query-scripts/quick-ref.sh agents        # List all agents
  ./tools/query-scripts/quick-ref.sh vision        # Vision summary
  ./tools/query-scripts/quick-ref.sh structure     # Folder layout
  ```

- **`ai-context.sh`** - Context building (1-4K tokens)
  ```bash
  ./tools/query-scripts/ai-context.sh project      # Project overview
  ./tools/query-scripts/ai-context.sh architecture # Architecture summary
  ./tools/query-scripts/ai-context.sh conventions  # Coding standards
  ```

- **`query-docs.sh`** - Documentation search (500-3K tokens)
  ```bash
  ./tools/query-scripts/query-docs.sh --agent developer  # Find agent
  ./tools/query-scripts/query-docs.sh --issue 39         # Issue docs
  ./tools/query-scripts/query-docs.sh --topic auth       # Search topic
  ```

- **`query-metadata.sh`** - Metadata extraction (100-500 tokens)
  ```bash
  ./tools/query-scripts/query-metadata.sh --stats        # Statistics
  ./tools/query-scripts/query-metadata.sh --list-types   # Document types
  ```

**See `docs/guides/ai-navigation.md` for complete query script guide.**

**Token Savings Examples**:
- List agents: **99.2% savings** (200 tokens vs 25K)
- Project overview: **93.3% savings** (800 tokens vs 12K)
- Find issue docs: **98% savings** (2K tokens vs 100K)

### 🎯 Context Loading Decision Tree

**CRITICAL**: Choose the right context mechanism for maximum efficiency.

```
┌─ Need to understand AIAP structure/agents/vision?
│  └─► Use QUERY SCRIPTS (85-95% token savings)
│     Examples:
│     - ./tools/query-scripts/quick-ref.sh agents
│     - ./tools/query-scripts/ai-context.sh project
│     - ./tools/query-scripts/query-docs.sh --topic X
│
├─ Need to reference SPECIFIC CODE files/classes?
│  └─► Use # REFERENCES (targeted, 50-70% savings vs @workspace)
│     Examples:
│     - "Explain #MyClass.cs"
│     - "Fix bug in #src/services/UserService.ts"
│     - "#file1.cs #file2.cs compare these implementations"
│
├─ Need EXPLORATORY analysis of entire codebase?
│  └─► Use @workspace (comprehensive but token-heavy)
│     Examples:
│     - "@workspace How is authentication implemented?"
│     - "@workspace Find all database queries"
│     Use ONLY when query scripts and # references are insufficient
│
├─ Need PROJECT-WIDE RULES (coding standards, conventions)?
│  └─► Already loaded! (this .github/copilot-instructions.md)
│     Zero token cost - automatically applied to all interactions
│
└─ Working in CURRENT FILE only?
   └─► Default context (automatic, no action needed)
      Open related files in tabs for additional context
```

**Token Efficiency Comparison**:

| Task | ❌ @workspace | ✅ AIAP Query Scripts | Savings |
|------|---------------|----------------------|---------|
| List all agents | 25,000 tokens | 200 tokens | **99.2%** |
| Project overview | 12,000 tokens | 800 tokens | **93.3%** |
| Find issue docs | 100,000 tokens | 2,000 tokens | **98%** |
| Architecture summary | 15,000 tokens | 800 tokens | **94.7%** |

**Best Practice**: Start with query scripts for documentation, then escalate to # or @workspace only if needed.

## Project Overview

This is the AIAP (AI-Augmented Project) System—a universal framework for integrating GitHub Copilot and AI agents into software engineering workflows across ANY technology stack.

AIAP demonstrates best practices for:
- Providing rich context to AI assistants through structured documentation
- Defining specialized AI agent roles for different development tasks
- Establishing workflow governance through templates and conventions
- Managing multiple projects with different technology stacks under unified AI orchestration

**AIAP is stack-agnostic**: It works with .NET, Python, TypeScript, Java, mobile platforms, and any other technology through plugin-based adapters.

## AIAP Framework Architecture

### What AIAP Contains
- **AI Agent Profiles**: 18+ specialized agents for different development roles
- **Orchestration System**: Multi-agent workflow coordination
- **Query Scripts**: Token-optimized documentation retrieval (85-95% reduction)
- **Templates**: `.aiap/` configuration templates for project integration
- **Documentation**: Comprehensive guides and architecture decisions

### What AIAP Does NOT Contain
- ❌ Project-specific source code (.NET, Python, etc.)
- ❌ Stack-specific implementations
- ❌ Example applications

**Projects are managed externally** and integrated via `.aiap/` configuration folder.

See [AIAP Multi-Project Architecture](docs/architecture/aiap-multi-project-architecture.md) for details.

## Development Tools
- **Git** for version control with Conventional Commits
- **GitHub Actions** for CI/CD
- **EditorConfig** for consistent coding styles
- **MCP Servers**: Model Context Protocol for extended AI capabilities

## AIAP Integration Principles

### How Projects Integrate with AIAP

Projects add a `.aiap/` configuration folder to enable AI-augmented capabilities:

```
your-project/                    # Any stack (.NET, Python, etc.)
├── .aiap/                       # AIAP configuration (ADD THIS)
│   ├── project-context.yml      # Project identity & tech stack
│   ├── stack-config.yml         # Stack-specific configuration
│   └── integration-hooks.yml    # Tool integrations
├── src/                         # Your source code (UNCHANGED)
├── tests/                       # Your tests (UNCHANGED)
└── docs/                        # Your documentation (UNCHANGED)
```

### Stack Adapters

AIAP uses plugin-based adapters to understand different technology stacks:

- **`.NET`**: Understands .csproj, NuGet, C# complexity metrics
- **`Python`**: Understands requirements.txt, pip, Python conventions
- **`TypeScript`**: Understands package.json, npm, TypeScript config
- **`Java`**: Understands pom.xml, Maven/Gradle, Java conventions
- **`Generic`**: Basic file structure analysis for any stack

See `src/aiap-core/integrations/` for adapter implementations.

### Multi-Project Orchestration

AIAP can work with:
- **Single project**: Independent AI assistance
- **Multiple projects**: Cross-project coordination and dependency analysis
- **Different stacks**: Unified AI orchestration across .NET, Python, TypeScript, etc.

### Code Organization Principles

**For AIAP Framework Code** (this repository):
- Group by functionality (agents, tools, integrations)
- Keep stack adapters plugin-based and isolated
- Maintain clear separation between framework and project concerns

**For Your Projects** (external repositories):
- Follow your stack's conventions
- Add `.aiap/` folder for AIAP integration
- AIAP reads your conventions through stack adapters

### Folder Policies

**Documentation Workspace (`ai-agent-docs-web/`)**
- **Purpose**: Workspace for Docs Web Agent to draft documentation content
- **Structure**:
  - `README.md` - Workflow explanation (committed)
  - `_work/` - Draft content directory (git-ignored, never committed)
    - `_work/drafts/` - Draft documentation pages
    - `_work/outlines/` - Content outlines and plans
    - `_work/notes/` - Research and reference notes
- **Workflow**: Draft → Review → Publish
  1. Docs Web Agent drafts content in `_work/`
  2. Human reviewers check drafts
  3. Approved content moves to `docs-site/` via PR
- **Guardrails**:
  - Only Docs Web Agent writes to this directory
  - Drafts are never committed to version control
  - All published content requires human review

**Documentation Sites**
- **`docs-site/`** - Docusaurus website with Mermaid diagram support
  - Published documentation lives in `docs-site/docs/`
  - Changelog/news in `docs-site/blog/`
  - Built-in Mermaid for flowcharts, sequence diagrams, etc.
  - Built and deployed to GitHub Pages

**Project Documentation (`docs/`)**
- Architecture Decision Records (ADRs)
- Deep technical documentation
- Design guides
- Referenced by the documentation websites

## Coding Standards

### General Principles
- **Clarity over Cleverness**: Write code that is easy to understand
- **DRY (Don't Repeat Yourself)**: Extract common patterns into reusable components
- **YAGNI (You Aren't Gonna Need It)**: Don't add functionality until it's needed
- **Fail Fast**: Validate inputs early and provide clear error messages
- **Immutability**: Prefer immutable objects where appropriate

### C# Conventions
- Follow [Microsoft C# Coding Conventions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
- **PascalCase** for class names, methods, properties, and constants
- **camelCase** for local variables and parameters
- **_camelCase** for private fields (with underscore prefix)
- Use **var** when type is obvious from the right side
- Always use braces for control structures, even single-line statements
- Place opening braces on new lines (Allman style)
- Maximum line length: 120 characters

### Naming Conventions
- Use descriptive, meaningful names
- Avoid abbreviations unless widely understood
- Interfaces should start with 'I' (e.g., `IRepository`)
- Use async suffix for async methods (e.g., `GetUserAsync`)
- Boolean variables/methods should read like questions (e.g., `IsValid`, `CanExecute`)

### Documentation Requirements
- Add XML documentation comments for all public APIs
- Include `<summary>`, `<param>`, `<returns>`, and `<exception>` tags
- Document "why" in comments, not "what" (code should be self-documenting)
- Keep comments up-to-date with code changes
- Use meaningful commit messages following Conventional Commits

## Testing Requirements

### Test Coverage
- Aim for **80%+ code coverage** on business logic
- **100% coverage** on critical paths (authentication, payment, etc.)
- Every bug fix must include a regression test

### Test Structure
- Follow **Arrange-Act-Assert (AAA)** pattern
- Use descriptive test method names: `MethodName_Scenario_ExpectedBehavior`
- One assertion per test when possible (logical unit)
- Tests should be isolated and independent
- Use test data builders for complex object creation

### Test Categories
- **Unit Tests**: Test individual components in isolation
- **Integration Tests**: Test component interactions
- **End-to-End Tests**: Test complete user workflows
- Use `[Trait]` attributes to categorize tests

## Security Guidelines

### Input Validation
- **Never trust user input**: Validate and sanitize all inputs
- Use parameterized queries to prevent SQL injection
- Validate file uploads (type, size, content)
- Implement rate limiting on public endpoints

### Authentication & Authorization
- Use industry-standard protocols (OAuth 2.0, OpenID Connect)
- Never store passwords in plain text
- Implement proper RBAC (Role-Based Access Control)
- Use HTTPS for all communications

### Sensitive Data
- Never commit secrets to version control
- Use environment variables or secure vaults for configuration
- Redact sensitive data in logs
- Follow GDPR/privacy requirements for personal data

## Performance Considerations

- Use async/await for I/O-bound operations
- Implement caching strategies where appropriate
- Profile before optimizing
- Consider pagination for large data sets
- Use appropriate HTTP status codes and headers

## Error Handling

- Use exceptions for exceptional cases only
- Create custom exception types for domain-specific errors
- Log all exceptions with context
- Return meaningful error messages to clients
- Implement global exception handling

## Commit Message Format

Follow [Conventional Commits](https://www.conventionalcommits.org/) specification:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### Types
- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation changes
- **style**: Code style changes (formatting, missing semicolons, etc.)
- **refactor**: Code refactoring without changing functionality
- **perf**: Performance improvements
- **test**: Adding or updating tests
- **chore**: Maintenance tasks (dependencies, build config, etc.)
- **ci**: CI/CD configuration changes

### Examples
```
feat(auth): add password reset functionality
fix(api): handle null reference in user controller
docs(readme): update installation instructions
test(users): add integration tests for user service
```

## Pull Request Guidelines

### Before Creating a PR
- Ensure all tests pass locally
- Run linters and code formatters
- Update relevant documentation
- Add or update tests for your changes
- Verify no sensitive data is included

### PR Description Should Include
- **What**: Brief description of changes
- **Why**: Reason for the changes (link to issue)
- **How**: Implementation approach if not obvious
- **Testing**: How the changes were tested
- **Screenshots**: For UI changes

### Review Process
- At least one approval required before merge
- All CI checks must pass
- Code owner approval for sensitive areas
- Address all review comments or discuss

## Documentation Standards

### Code Comments
- Explain "why", not "what"
- Keep comments concise and relevant
- Update comments when code changes
- Use TODO comments sparingly with assignee

### README Files
- Every project/module should have a README
- Include purpose, setup, usage examples
- Keep it up-to-date
- Link to additional documentation

### API Documentation
- Document all public endpoints
- Include request/response examples
- Specify authentication requirements
- Document error responses

## AI Agent Collaboration

When working with AI agents (including GitHub Copilot):

### Quick Agent Invocation

All agents can be invoked using short, memorable names:

```bash
# Core agents - frequently used
/agent stakeholder    # Business & requirements
/agent architect      # System design & ADRs
/agent dev            # Code implementation
/agent qa             # Testing & quality assurance
/agent docs           # Technical documentation
/agent devops         # CI/CD & deployment
/agent security       # Security review
/agent perf           # Performance optimization

# Specialist agents
/agent bugfix         # Bug fixing
/agent review         # Code review
/agent refactor       # Refactoring
/agent planner        # Implementation planning
```

**Complete Reference**: See [AGENTS.md](../AGENTS.md) for full agent list and [Agent Naming Convention](../docs/architecture/agent-naming-convention.md) for details.

### Available AI Agents

This repository includes specialized AI agents for specific tasks. All agents now include:
- **MCP Server Integration**: Access to Model Context Protocol servers for extended capabilities
- **Clearly Defined Tools**: Explicit tool lists following least privilege principle
- **Non-Goals**: Documented boundaries to prevent scope creep
- **Official Documentation Links**: Direct references to GitHub Copilot and MCP documentation

#### Core Enterprise Agents (10)

1. **Stakeholder Agent** (`.github/agents/stakeholder-agent.agent.md`) - `/agent stakeholder`
   - Business requirements, ROI analysis, user stories
   - **MCP**: github, memory

2. **Architect Agent** (`.github/agents/architect-agent.agent.md`) - `/agent architect`
   - System architecture, ADRs, technology decisions
   - **MCP**: github, sequential-thinking

3. **Developer Agent** (`.github/agents/developer-agent.agent.md`) - `/agent dev`
   - Feature implementation, bug fixing, refactoring
   - **MCP**: github, sequential-thinking

4. **Tester/QA Agent** (`.github/agents/tester-qa-agent.agent.md`) - `/agent qa`
   - Test strategy, test automation, quality assurance
   - **MCP**: github, sequential-thinking

5. **Documentation Agent** (`.github/agents/documentation-agent.agent.md`) - `/agent docs`
   - Technical writing in `docs/`, ADRs, guides
   - **MCP**: github, sequential-thinking

6. **DevOps/Release Agent** (`.github/agents/devops-release-agent.agent.md`) - `/agent devops`
   - CI/CD pipelines, deployment, release management
   - **MCP**: github, fetch, time

7. **Security/Audit Agent** (`.github/agents/security-audit-agent.agent.md`) - `/agent security`
   - Security review, vulnerability scanning, compliance
   - **MCP**: github, fetch

8. **Performance Agent** (`.github/agents/performance-agent.agent.md`) - `/agent perf`
   - Performance optimization, profiling, monitoring
   - **MCP**: github, time

9. **Compliance/Regulator Agent** (`.github/agents/compliance-regulator-agent.agent.md`) - `/agent compliance`
   - GDPR, licensing, regulatory compliance
   - **MCP**: github, fetch

10. **Change Management Agent** (`.github/agents/change-management-agent.agent.md`) - `/agent change-mgmt`
    - Feature flags, versioning, impact analysis
    - **MCP**: github, sequential-thinking

#### Specialized Support Agents

0. **AI Prompt Architect** (`.github/agents/ai-prompt-architect.agent.md`) **[MASTER ORCHESTRATOR]** - `/agent prompt-architect`
   - Creation, refinement, and orchestration of all AI agents
   - Prompt engineering and custom instructions optimization
   - AI system documentation and compliance
   - **MCP**: github, sequential-thinking, memory, fetch, time
   - **Role**: Meta-agent responsible for the entire AI agent ecosystem

11. **Bug Fix Teammate** (`.github/agents/bug-fix-teammate.agent.md`) - `/agent bugfix`
    - Systematic debugging, root cause analysis
    - **MCP**: github, sequential-thinking

12. **Code Reviewer** (`.github/agents/code-reviewer.agent.md`) - `/agent review`
    - Automated code quality and security reviews
    - **MCP**: github, sequential-thinking

13. **Refactoring Specialist** (`.github/agents/refactoring-specialist.agent.md`) - `/agent refactor`
    - Code cleanup and improvement without behavior changes
    - **MCP**: github, sequential-thinking

14. **Implementation Planner** (`.github/agents/implementation-planner.agent.md`) - `/agent planner`
    - Technical planning, feature breakdown
    - **MCP**: github, sequential-thinking

15. **Test Specialist** (`.github/agents/test-specialist.agent.md`) - `/agent test-spec`
    - Test design, coverage analysis
    - **MCP**: github, sequential-thinking

16. **Documentation Assistant** (`.github/agents/documentation-assistant.agent.md`) - `/agent doc-assist`
    - General documentation creation
    - **MCP**: github, sequential-thinking

17. **Docs Web Agent** (`.github/agents/docs-web.agent.md`) - `/agent docs-web`
    - Documentation website management (Docusaurus)
    - **MCP**: github, sequential-thinking
    - **Workflow**: Draft in `ai-agent-docs-web/_work/` → Review → Publish to `docs-site/`
    - **Guardrails**: Never modifies application code, always requires human review

### Model Context Protocol (MCP) Overview

The **Model Context Protocol** is an open standard that extends GitHub Copilot agents with specialized capabilities:

- **Official MCP Documentation**: [modelcontextprotocol.io](https://modelcontextprotocol.io/)
- **GitHub MCP Server**: [github/github-mcp-server](https://github.com/github/github-mcp-server)
- **MCP Integration Guide**: See [docs/architecture/mcp-integration.md](../docs/architecture/mcp-integration.md)

**Key MCP Servers in This Repository:**
- `github/github-mcp-server` - Repository operations, GitHub API access
- `modelcontextprotocol/server-fetch` - External HTTP/HTTPS requests
- `modelcontextprotocol/server-memory` - Session context and state management
- `modelcontextprotocol/server-time` - Timestamps and duration tracking
- `modelcontextprotocol/server-sequential-thinking` - Step-by-step reasoning
- `modelcontextprotocol/server-everything` - General-purpose testing

See `.mcp.json` in the repository root for complete configuration.

### Multi-Agent Workflow

This repository implements a **multi-agent orchestration** pattern inspired by Microsoft's Copilot Studio:

```
User/Stakeholder → Architect → Developer → Tester → Code Reviewer → 
Security → Performance → Documentation → DevOps → Compliance → 
Change Management → Release
```

**Learn More:**
- [Multi-Agent Orchestration Blog](https://www.microsoft.com/en-us/microsoft-copilot/blog/copilot-studio/multi-agent-orchestration-maker-controls-and-more-microsoft-copilot-studio-announcements-at-microsoft-build-2025/)
- [Agent Workflow Architecture](../docs/architecture/agent-workflow.md)

### AI Output Storage Requirements

**CRITICAL: All AI agents MUST follow the artifact storage guidelines in `/ARTIFACT_GUIDELINES.md`**

#### File Storage Decision Tree

1. **Temporary/Draft Files** → `/artifacts/workspace/{agent-name}/`
   - Git-ignored, never committed
   - Clean up when work is complete

2. **Source Code** → `/src/{language}/{layer}/`
   - Requires human review before commit
   - Follow language-specific conventions

3. **Test Code** → `/tests/{language}/{test-type}/`
   - Requires human review before commit
   - Follow testing framework conventions

4. **Human Documentation** → `/docs/{category}/`
   - **Humans ONLY** (AI can draft in workspace, human approves)
   - ADRs, guides, API reference

5. **AI Summaries** → `/artifacts/summaries/{YYYY-MM}/`
   - Use `/templates/agent-outputs/summary-template.md`
   - File naming: `summary-{topic}-{YYYY-MM-DD}.md`
   - ≤2000 words (Issue/PR analysis, meeting notes)

6. **AI Reports** → `/artifacts/reports/{YYYY-MM}/`
   - Use `/templates/agent-outputs/report-template.md`
   - File naming: `report-{category}-{YYYY-MM-DD}.md`
   - 2000-5000 words (Audits, assessments)

7. **AI Analysis** → `/artifacts/analysis/{YYYY-MM}/`
   - Use `/templates/agent-outputs/analysis-template.md`
   - File naming: `analysis-{topic}-{YYYY-MM-DD}.md`
   - 5000+ words (Deep-dive research)

#### Mandatory Metadata for AI Outputs

**ALL files in `/artifacts/{summaries|reports|analysis}/` MUST include YAML frontmatter:**

```yaml
---
type: {summary|report|analysis}
generator: {agent-name}
timestamp: {ISO-8601}
source: {issue/pr/meeting URL}
version: 1.0
tags: [{tag1}, {tag2}, {tag3}]
---
```

**Example**:
```yaml
---
type: summary
generator: architect-agent
timestamp: 2025-11-09T14:23:45Z
source: https://github.com/tlejmi/ai-augmented-project-template/issues/39
version: 1.0
tags: [issue-39, architecture, folder-structure]
---
```

#### Template Compliance

- ✅ **Use templates** from `/templates/agent-outputs/`
- ✅ **Complete metadata** required (YAML frontmatter)
- ✅ **Correct naming** (type-topic-YYYY-MM-DD.md)
- ✅ **Proper location** (/artifacts/{type}/{YYYY-MM}/)
- ❌ **Validation fails** if any requirement missing

#### Protected Directories

**Humans ONLY**:
- `/src/**` - Source code (AI assists, human commits)
- `/tests/**` - Test code (AI assists, human commits)
- `/docs/**` - Documentation (AI drafts in workspace, human approves)

**AI Agents ONLY**:
- `/artifacts/summaries/**`
- `/artifacts/reports/**`
- `/artifacts/analysis/**`
- `/artifacts/workspace/**` (temporary)

**BOTH CANNOT WRITE**:
- `/.github/workflows/**` (admin approval required)
- `/CHANGELOG.md` (auto-generated only)
- `/LICENSE` (immutable)

#### Workflow Example

**Creating an Issue Analysis**:
1. Draft in `/artifacts/workspace/architect-agent/draft-analysis.md`
2. Research and write content
3. Apply `/templates/agent-outputs/summary-template.md`
4. Add YAML frontmatter with all required fields
5. Save to `/artifacts/summaries/2025-11/summary-issue-39-analysis-2025-11-09.md`
6. Commit to repository

**See `/ARTIFACT_GUIDELINES.md` for complete details, examples, and troubleshooting.**

---

### For AI Agents Reading This

#### 🎯 System Awareness Requirements (MANDATORY)

**Before making ANY changes (code, documentation, or configuration), you MUST:**

1. **Review VISION.md** - Understand the strategic direction, pillars, principles, and roadmap
   - Verify your changes align with all 5 strategic pillars
   - Ensure compliance with all 7 core principles
   - Check impact on current phase and future roadmap items
   
2. **Validate Against Specifications** - Check all relevant specs in `specs/` directory
   - Review `specs/agents/` for agent capability requirements
   - Check `specs/prompts/` for prompt template standards
   - Verify `specs/orchestration/` for workflow patterns
   
3. **Assess Holistic Impact** - Consider how your changes affect:
   - Project vision and strategic goals
   - Related components and dependencies
   - Other agent responsibilities and workflows
   - Documentation consistency across all files
   - Future maintainability and extensibility
   
4. **Follow ALL Documented Conventions**:
   - **Conventional Commits**: `<type>(<scope>): <description>` format (CONTRIBUTING.md)
   - **ARTIFACT_GUIDELINES.md**: File placement, YAML frontmatter, naming conventions
   - **Coding Standards**: Follow your project's stack conventions
   - **Testing Requirements**: Follow your project's testing standards
   
5. **Validate Pre-Commit Requirements**:
   - Run language-specific formatters (e.g., `prettier`, `black`, `dotnet format`)
   - Build and compile to ensure no errors
   - Run all tests to verify nothing breaks
   - Verify commit message follows Conventional Commits format

**🚨 CRITICAL**: AI agents are not just following human instructions easily - you are responsible for understanding the **full context**, **impact**, and **consequences** of every change. Always ask: "How does this change affect the project's vision, roadmap, and related systems?"

#### Working Guidelines

1. **Context First**: Always read AGENTS.md and relevant agent profiles before starting work
2. **Follow Storage Rules**: Use `/ARTIFACT_GUIDELINES.md` decision tree for file placement
3. **Use Templates**: Apply correct template from `/templates/agent-outputs/`
4. **Include Metadata**: YAML frontmatter mandatory for all AI outputs
5. **Follow Conventions**: Adhere strictly to the coding standards and architecture guidelines above
6. **Use MCP Tools**: Leverage Model Context Protocol servers for extended capabilities
7. **Be Specific**: In commits and PRs, provide clear, detailed descriptions
8. **Ask Questions**: If requirements are unclear, ask for clarification before proceeding
9. **Test Thoroughly**: Ensure all changes are tested and don't break existing functionality
10. **Document Changes**: Update relevant documentation when making changes
11. **Respect Boundaries**: Documentation agents work only on docs, code agents only on code
12. **Check Official Docs**: Reference [GitHub Copilot Agent Documentation](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents) for best practices

### For Human Developers
1. **Provide Clear Tasks**: Give AI agents well-scoped, specific tasks with clear acceptance criteria
2. **Review AI Output**: Always review AI-generated code before merging
3. **Iterate with Feedback**: Use PR comments to guide AI agents to improvements
4. **Leverage Strengths**: Assign AI agents tasks they excel at (testing, docs, bug fixes, refactoring)
5. **Maintain Oversight**: AI is an assistant, not a replacement for human judgment
6. **Validate Artifacts**: Check AI outputs in `/artifacts/` follow templates and have proper metadata

## Best Practices for AI-Generated Code

- **Validate Security**: Check for security vulnerabilities in AI-generated code
- **Check Dependencies**: Ensure AI doesn't introduce unnecessary dependencies
- **Review Logic**: Verify business logic correctness
- **Test Coverage**: Ensure adequate tests are included
- **Code Quality**: Check adherence to project standards
- **Performance**: Review for performance implications

## Resources

### GitHub Copilot Documentation
- [Creating Custom Agents](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents) - Agent profile specification and configuration
- [GitHub Copilot Agents Overview](https://docs.github.com/en/copilot/concepts/extensions/agents) - Agent concepts and extension model
- [Agent Mode 101](https://github.blog/ai-and-ml/github-copilot/agent-mode-101-all-about-github-copilots-powerful-mode/) - Comprehensive guide to agent mode workflows
- [GitHub Copilot Best Practices](https://docs.github.com/en/copilot/tutorials/coding-agent/get-the-best-results) - Getting the best results from agents

### Model Context Protocol (MCP)
- [Model Context Protocol Official Site](https://modelcontextprotocol.io/) - MCP specification and standards
- [MCP GitHub Organization](https://github.com/modelcontextprotocol) - Official MCP server implementations
- [GitHub MCP Server](https://github.com/github/github-mcp-server) - Official GitHub MCP implementation
- [MCP Servers Collection](https://github.com/modelcontextprotocol/servers) - Collection of official MCP servers

### Community Examples
- [Awesome GitHub Copilot](https://github.com/github/awesome-copilot) - Community-contributed customizations and examples
- [Awesome Copilot Blog](https://developer.microsoft.com/blog/introducing-awesome-github-copilot-customizations-repo) - Custom instructions examples
- [Multi-Agent Orchestration](https://www.microsoft.com/en-us/microsoft-copilot/blog/copilot-studio/multi-agent-orchestration-maker-controls-and-more-microsoft-copilot-studio-announcements-at-microsoft-build-2025/) - Enterprise multi-agent patterns

### Competitive Landscape & Strategic Context

**AIAP's Unique Position**: While other AI coding tools focus on single-project assistance, AIAP uniquely provides **multi-project orchestration** and **organizational knowledge management** across diverse technology stacks.

**Competitive Analysis**:
- See [OpenAI Codex Strategic Analysis](artifacts/analysis/2025-11/analysis-openai-codex-agents-strategic-comparison-2025-11-15.md) for comprehensive comparison
- See [Integration Insights Summary](artifacts/summaries/2025-11/summary-openai-codex-integration-insights-2025-11-15.md) for quick overview

**Key Differentiators**:
1. **Multi-Project Support**: Orchestrate .NET + Python + Angular + Android + iOS simultaneously (unmatched)
2. **Knowledge Capture**: YAML frontmatter + templates + persistent artifacts (unique)
3. **Token Optimization**: Query scripts provide 85-95% savings vs standard context loading (proven)
4. **RAG-Ready**: Explicit roadmap for semantic search over organizational knowledge (transparent)
5. **Non-Intrusive**: Just add `.aiap/` folder—no code changes required (lowest friction)

**OpenAI Codex/Agents Resources** (for competitive awareness):
- [OpenAI Codex Official](https://openai.com/codex/) - AI developer co-pilot
- [OpenAI Agents SDK](https://platform.openai.com/docs/guides/agents-sdk) - Agent orchestration SDK
- [OpenAI AgentKit](https://developers.openai.com/resources/agents) - Visual workflow builder

**Strategic Positioning**: AIAP and OpenAI Codex are **complementary, not competitive**. Organizations can use both:
- **OpenAI Codex**: For autonomous execution within individual projects
- **AIAP System**: For multi-project coordination and knowledge management

### Resources for Stack-Specific Development

When integrating AIAP with your projects, refer to these resources:

#### .NET Projects
- [.NET Documentation](https://learn.microsoft.com/en-us/dotnet/)
- [C# Programming Guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/)
- [ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/)
- [Entity Framework Core](https://learn.microsoft.com/en-us/ef/core/)

#### Python Projects
- [Python Documentation](https://docs.python.org/)
- [PEP 8 Style Guide](https://peps.python.org/pep-0008/)
- [Django](https://www.djangoproject.com/) / [Flask](https://flask.palletsprojects.com/)

#### TypeScript/JavaScript Projects
- [TypeScript Documentation](https://www.typescriptlang.org/docs/)
- [Node.js Documentation](https://nodejs.org/docs/)
- [Angular](https://angular.io/) / [React](https://react.dev/) / [Vue](https://vuejs.org/)

### Development Tools
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Semantic Versioning](https://semver.org/)

---

**Note**: These instructions are living documentation. Update them as the project evolves and new patterns emerge.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tlejmi)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/tlejmi)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
