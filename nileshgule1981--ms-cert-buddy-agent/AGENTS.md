# GitHub Copilot Instructions for AI-102 Cert Buddy

This file provides context and instructions for GitHub Copilot when working in the AI-102 Cert Buddy workspace.

## Project Overview

AI-102 Cert Buddy is a certification study companion that helps users prepare for the Microsoft AI-102: Designing and Implementing a Microsoft Azure AI Solution exam. The project provides:

- Practice questions that feel exam-realistic without copying the exam
- Hands-on practice labs (10-20 minutes each)
- Personalized study plans based on skill area confidence

## Azure Product Name Terminology

**CRITICAL:** Always use current Microsoft product names. Never use retired names.

### Official Product Name Changes (Rename Table)

| **Retired Name (DO NOT USE)** | **Current Name (USE THIS)** |
|-------------------------------|------------------------------|
| Azure AD | Microsoft Entra ID |
| Azure AD Connect | Microsoft Entra Connect |
| Azure AD Connect Health | Microsoft Entra Connect Health |
| Azure AD Domain Services | Microsoft Entra Domain Services |
| Azure AD B2C | Microsoft Entra External ID |
| Azure AD B2B | Microsoft Entra External ID |
| Azure Active Directory | Microsoft Entra ID |
| Azure Cognitive Services | Azure AI services |
| Cognitive Services | Azure AI services |
| Computer Vision | Azure AI Vision |
| Custom Vision | Azure AI Custom Vision |
| Face API | Azure AI Face |
| Form Recognizer | Azure AI Document Intelligence |
| Language Understanding (LUIS) | Azure AI Language (conversational language understanding) |
| QnA Maker | Azure AI Language (question answering) |
| Speech Service | Azure AI Speech |
| Text Analytics | Azure AI Language |
| Translator | Azure AI Translator |
| Azure Search | Azure Cognitive Search |
| Azure Applied AI Services | Azure AI services |
| Content Moderator | Azure AI Content Safety |
| Personalizer | (retired - no direct replacement) |
| Anomaly Detector | Azure AI Anomaly Detector |
| Metrics Advisor | (retired - use Azure Monitor) |
| Immersive Reader | Azure AI Immersive Reader |
| Video Indexer | Azure Video Indexer |
| Azure Bot Service | Azure AI Bot Service |
| Bot Framework | Azure AI Bot Framework |
| Azure OpenAI Service | Azure OpenAI Service (no change) |

### AI-102 Specific Terminology

- **Always refer to "Azure AI services"** (not "Cognitive Services") when discussing the suite of AI capabilities
- **Use "Azure AI Vision"** for computer vision capabilities
- **Use "Azure AI Language"** for natural language processing capabilities
- **Use "Azure AI Document Intelligence"** (formerly Form Recognizer)
- **Use "conversational language understanding"** when referring to LUIS functionality within Azure AI Language
- **Use "question answering"** when referring to QnA Maker functionality within Azure AI Language

## Code Style and Best Practices

### General

- Follow Microsoft Azure naming conventions for all resources
- Use meaningful, descriptive names for resources, variables, and functions
- Include comments explaining complex logic or Azure-specific configurations
- Always include error handling and validation
- Use Azure SDK best practices for authentication and resource management

### Python

- Use Python 3.8+ features
- Follow PEP 8 style guidelines
- Use type hints where appropriate
- Prefer Azure SDK v2 (azure-ai-* packages) over legacy v1 packages
- Use async/await for Azure SDK operations when possible

### Azure CLI

- Use current Azure CLI syntax (az version 2.50+)
- Use `--query` with JMESPath for filtering output
- Use `--output` to specify JSON, table, or tsv format
- Always include `--resource-group` and `--location` parameters explicitly
- Use variables for repeated values (subscription ID, resource group, location)

### Bicep/ARM Templates

- Use Bicep over JSON ARM templates when possible
- Include parameter files for different environments
- Use modules for reusable components
- Include descriptions for all parameters
- Use secure parameters for sensitive values

## Exam Question Guidelines

When generating practice questions:

1. **Scenario-first**: Always open with a workplace scenario before asking the question
2. **Plausible distractors**: Use real Azure services/features that are genuinely related but incorrect
3. **No trick wording**: Questions should test knowledge, not reading comprehension
4. **Current terminology**: Always use the rename table above
5. **Exactly 4 options (A-D)**: Unless explicitly requested otherwise
6. **Exactly 1 correct answer**: Unless explicitly requested otherwise
7. **No contractions**: Use formal language
8. **Microsoft style**: Follow Microsoft sentence-style capitalization

## Lab Guidelines

When generating practice labs:

1. **Single objective**: Each lab should focus on one learning outcome
2. **10-20 minutes**: Keep labs short and focused
3. **Validation steps**: Include explicit checks after each major task
4. **Cleanup included**: Always provide steps to remove created resources
5. **Cost awareness**: Prefer free tier resources; warn if charges may apply
6. **Real commands**: All CLI/PowerShell commands must be valid and tested
7. **Current terminology**: Always use the rename table above

## MCP Server Usage

### Context7 MCP
- Use for accessing Microsoft Learn documentation
- Use for verifying current Azure CLI/PowerShell syntax
- Use for finding up-to-date SDK code examples

### Azure MCP
- Use for validating Azure commands and resource properties
- Use for checking resource availability and quotas
- Use for verifying portal navigation paths

### Markitdown MCP
- Use for converting documentation formats
- Use for processing external learning resources

## File Organization

```
.github/
  skills/                    # Copilot skills for different task types
    ai102-item-creator/      # Practice question generator
    ai102-lab-creator/       # Practice lab generator
    ai102-study-planner/     # Personalized study plan generator
  copilot-instructions.md    # This file

references/
  ai102-objectives.md        # Official exam skills outline
  style-guide.md            # Microsoft writing style rules
  fictional-companies.md     # Company names for scenarios

.vscode/
  mcp.json                  # MCP server configuration
```

## References

- [AI-102 Official Exam Page](https://learn.microsoft.com/en-us/credentials/certifications/exams/ai-102/)
- [Azure AI services Documentation](https://learn.microsoft.com/en-us/azure/ai-services/)
- [Microsoft Entra ID Documentation](https://learn.microsoft.com/en-us/entra/identity/)
- [Azure Cognitive Search Documentation](https://learn.microsoft.com/en-us/azure/search/)
- [Azure OpenAI Service Documentation](https://learn.microsoft.com/en-us/azure/ai-services/openai/)

## Responsible AI Principles

When generating content related to AI solutions, always consider and reference Microsoft's Responsible AI principles:

1. **Fairness**: AI systems should treat all people fairly
2. **Reliability and Safety**: AI systems should perform reliably and safely
3. **Privacy and Security**: AI systems should be secure and respect privacy
4. **Inclusiveness**: AI systems should empower everyone and engage people
5. **Transparency**: AI systems should be understandable
6. **Accountability**: People should be accountable for AI systems

---

**Last Updated:** March 2026

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nileshgule1981)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/nileshgule1981)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
