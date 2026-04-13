# GitHub Copilot Instructions

## Project Overview
This is a Zulip bot with modular features including anonymous posting, private access control, and admin controls.

## Code Style Guidelines
- Follow PEP 8 style guidelines for Python code
- Include comments and docstrings for all functions and classes
- Use type hints for function parameters and return values
- Use async/await patterns with trio for asynchronous operations
- Keep functions focused and modular

## Architecture
- `core/`: Core bot functionality (client, dispatcher, models)
- `features/`: Feature modules (admin controls, anonymous posting, private access)
- `storage/`: Data persistence layer
- `utils/`: Utility functions (matching, scheduling)

## Best Practices
- Log important events and errors using the logger
- Handle errors gracefully with appropriate error messages
- Use the dispatcher pattern for handling different message types
- Store data using the file_store module
- Keep configuration separate in config.py
- Update the readme with new features or changes
- Do not store sensitive information in the codebase

## Testing Considerations
- Ensure async functions are properly awaited
- Mock external Zulip API calls in tests
- Test error handling paths

## Dependencies
- Uses the Zulip Python API client
- Uses trio for async operations
- See requirements.txt for full dependency list

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adjn)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/adjn)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
