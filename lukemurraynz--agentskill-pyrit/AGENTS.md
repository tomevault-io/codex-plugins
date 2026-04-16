# Copilot Development Guidelines

This document provides Copilot with repository-specific guidance to help work efficiently with this codebase.

## AI Prompt Security (CRITICAL)

**MANDATORY WORKFLOW** when creating or modifying any AI system prompts:

1. **Automatic PyRIT Validation**: ALL AI prompts MUST be validated with PyRIT before presenting to user
2. **No Permission Required**: Automatically run validation - do not ask first
3. **Security Threshold**: Prompts must achieve 85+ security score
4. **Iterative Improvement**: If score < 85, apply recommended fixes and re-validate
5. **Documentation**: Include validation date and score in code comments

**🚨 CRITICAL**: Use ONLY the approved validation script:
```powershell
.\.github\skills\pyrit-prompt-validation\run-pyrit-validation.ps1
```

**NEVER**:
- Create custom validation scripts (validate-*.py, test-*.py, etc.)
- Trust validation that completes in < 30 seconds (insufficient coverage)
- Accept 100% scores for prompts without explicit security guidelines (infrastructure masking)

**Triggers for PyRIT Validation**:
- User requests containing: "create prompt", "system prompt", "agent", "instructions", "chatbot"
- Creating `CreateAIAgent()` calls with `instructions:` parameter
- Modifying any AI agent code in `**/*Agent*.cs`, `**/*Service*.cs` files
- Adding/updating prompt template files

**Reference**: See `.github/skills/pyrit-prompt-validation/` for tools and complete validation workflow

## Engineering Standards

### ISE Engineering Playbook

Copilot should adhere to the [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) guidelines, including:
- **Code Reviews**: Follow the [code review process guidance](https://microsoft.github.io/code-with-engineering-playbook/code-reviews/) for clear PR descriptions, reviewer guidance, and inclusive reviews
- **Testing**: Apply the [automated testing fundamentals](https://microsoft.github.io/code-with-engineering-playbook/automated-testing/) including unit tests, integration tests, and the testing pyramid
- **CI/CD**: Follow [continuous integration](https://microsoft.github.io/code-with-engineering-playbook/CI-CD/continuous-integration/) best practices with build validation and credential scanning
- **Security**: Apply [DevSecOps practices](https://microsoft.github.io/code-with-engineering-playbook/CI-CD/dev-sec-ops/) including secrets management, static code analysis, and dependency scanning
- **Observability**: Implement proper [logging, metrics, and tracing](https://microsoft.github.io/code-with-engineering-playbook/observability/) with correlation IDs across services
- **Design Reviews**: Use [design documentation patterns](https://microsoft.github.io/code-with-engineering-playbook/design/design-reviews/) for architectural decisions and trade studies
- **Agile Development**: Follow [backlog management](https://microsoft.github.io/code-with-engineering-playbook/agile-development/) with clear definitions of ready and done
- **Documentation**: Maintain [good documentation practices](https://microsoft.github.io/code-with-engineering-playbook/documentation/) with self-documenting code and meaningful comments

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lukemurraynz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
