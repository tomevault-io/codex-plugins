<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Academic Idea Proposal System - Copilot Instructions

This is a Python project that builds an academic idea proposal system using LangGraph. The system uses multiple agents to search for research threads and combine ideas from different domains.

## Project Context
- Language: Python 3.8+
- Framework: LangGraph (built on LangChain)
- Purpose: Generate A+B type academic research ideas by combining insights from different fields
- Architecture: Multi-agent system with state management

## Code Style Guidelines
- Use type hints for all function parameters and return values
- Follow PEP 8 style guidelines
- Use async/await patterns where appropriate for API calls
- Create clear, descriptive variable and function names
- Add comprehensive docstrings for all classes and functions

## Key Patterns to Follow
- Use LangGraph's StateGraph for building agent workflows
- Implement proper state management with TypedDict and Annotated types
- Use the add_messages pattern for state updates
- Create modular agents that can be composed into larger workflows
- Handle API errors gracefully with proper exception handling

## Project Structure
- `/agents/` - Individual agent implementations
- `/tools/` - External API integrations (arXiv, Google Scholar, etc.)
- `/models/` - Data models and state definitions
- `/utils/` - Helper functions and utilities

## When suggesting code:
- Always include proper error handling
- Use environment variables for API keys
- Implement rate limiting for external API calls
- Create comprehensive logging for debugging
- Write modular, testable code
- Consider performance implications of concurrent operations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Oklahomawhore)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Oklahomawhore)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
