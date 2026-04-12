
# LangChain Development Rules

Follow these rules when working with LangChain components:

## Core Requirements
- Use LangChain >= 0.3.18
- use LangChain docs: https://python.langchain.com/docs/
- Implement proper callbacks
- Use type hints for all chains/agents
- Follow async patterns
- Handle errors gracefully

## Chain Structure
- Use proper inheritance from base classes
- Implement required abstract methods
- Add proper type hints
- Include docstrings
- Handle errors appropriately

## Agent Development
- Use structured output parsers
- Implement proper tool handling
- Add comprehensive error handling
- Include proper state management
- Monitor token usage

## Memory Management
- Use appropriate memory types
- Implement proper cleanup
- Handle state transitions
- Monitor memory usage
- Clear sensitive data

## Testing
- Mock LLM calls
- Test error scenarios
- Validate outputs
- Check memory handling
- Test async functionality

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tcashel)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/tcashel)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
