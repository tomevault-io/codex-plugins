<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## ApiClient - HTTP API Testing CLI Tool

A lightweight, powerful command-line HTTP client for testing and debugging APIs.

### Project Structure

- `apiclient/cli.py` - Typer CLI interface with commands
- `apiclient/client.py` - HTTP request logic and response handling
- `apiclient/config.py` - Request storage and configuration management
- `apiclient/formatter.py` - Rich library output formatting
- `apiclient/__main__.py` - Entry point for CLI

### Key Features Implemented

✅ HTTP Methods: GET, POST, PUT, PATCH, DELETE, HEAD
✅ Request Storage: Save/load requests as JSON
✅ Colorized Output: Using Rich library for beautiful terminal output
✅ Error Handling: Connection errors, timeouts, JSON parsing
✅ Request History: Track recent requests
✅ Headers & Body: Full support for custom headers and request bodies

### Available Commands

- `get` - Make a GET request
- `post` - Make a POST request
- `put` - Make a PUT request
- `patch` - Make a PATCH request
- `delete` - Make a DELETE request
- `head` - Make a HEAD request
- `save` - Save a request for later
- `list` - List all saved requests
- `show` - Show details of a saved request
- `run` - Execute a saved request
- `delete-request` - Delete a saved request

### Development Notes

Python 3.8+ required. Virtual environment configured with:
- requests 2.31.0
- typer 0.9.0
- rich 13.7.0
- pydantic 2.5.0

Config and request storage located in `~/.apiclient/`

### To Run

```bash
python -m apiclient --help
```

### Testing

Test endpoint working: `https://jsonplaceholder.typicode.com`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HughKnightOCE)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/HughKnightOCE)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
