# GitHub Copilot Instructions for gh-firehose

## Project Overview

gh-firehose is a terminal-based visualization tool that displays real-time GitHub push events on a rotating ASCII globe. It fetches push events from the GitHub Events API and renders them with an animated Earth globe using accurate equirectangular projection.

## Tech Stack

- **Language**: Go 1.25+
- **Key Dependencies**:
  - `github.com/gdamore/tcell/v2` - Terminal UI library for rendering and event handling
- **External Tools**:
  - GitHub CLI (`gh`) - Optional, for authenticated API access with higher rate limits

## Repository Structure

- `main.go` - Main application code with globe rendering, event fetching, and terminal UI
- `main_test.go` - Unit tests for event filtering
- `images/` - Demo assets
- `scripts/` - Utility scripts
- `.github/workflows/` - CI/CD workflows

## Build & Run Commands

### Install Dependencies
```bash
go mod download
```

### Build
```bash
go build
```

### Run
```bash
go run main.go
# Or after building:
./gh-firehose
```

### Test
```bash
go test -v
```

### Install Locally
```bash
go install
```

## Code Style & Conventions

1. **Go Standards**: Follow standard Go conventions and formatting
   - Use `gofmt` for formatting
   - Follow [Effective Go](https://golang.org/doc/effective_go.html) guidelines

2. **Naming Conventions**:
   - Use camelCase for variables and functions
   - Use PascalCase for exported types and functions
   - Use descriptive names (e.g., `fetchPushEvents`, `renderGlobe`)

3. **Comments**:
   - Add comments for exported functions and types
   - Include inline comments for complex algorithms (e.g., coordinate transformations)
   - Keep comments concise and meaningful

4. **Error Handling**:
   - Always check and handle errors appropriately
   - Use error return values, not panics, except for truly unrecoverable situations
   - Provide context in error messages

5. **Concurrency**:
   - The application uses goroutines extensively for animations and event fetching
   - Always protect shared state with mutexes (`sync.Mutex`)
   - Use channels for communication between goroutines where appropriate

6. **Terminal UI**:
   - Use `tcell` library for all terminal operations
   - Handle screen resizing gracefully
   - Support standard quit keys: `q`, `Esc`, `Ctrl+C`

## Key Architectural Patterns

1. **Globe Rendering**:
   - Uses equirectangular projection bitmap (`earthBitmap`) for accurate landmass rendering
   - 3D sphere projection with latitude/longitude calculations
   - Character aspect ratio compensation (2.1:1) for proper circle rendering

2. **Event Processing**:
   - Asynchronous event fetching with fallback from authenticated to public API
   - FIFO queue for topics and users display
   - Smooth fade-in/fade-out animations for all text elements

3. **Animation System**:
   - Multiple goroutines for different animation layers
   - 50ms render loop for smooth 20fps animation
   - Spinner animations, fade effects, and flash effects on globe

## Testing Guidelines

- Write unit tests for new functionality
- Existing test pattern: Mock HTTP servers for API testing
- Test event filtering, data processing, and edge cases
- Keep tests focused and fast

## Git Workflow

1. Create feature branches from `main`
2. Make focused, atomic commits with descriptive messages
3. Run tests before committing
4. Keep commits small and logically grouped

## Boundaries & Restrictions

### DO NOT:
- Modify the `earthBitmap` constant unless updating the projection data
- Break the existing animation timing (globe rotation, fade speeds)
- Remove or modify the GitHub CLI fallback mechanism
- Change the terminal UI library from tcell to something else
- Add dependencies without strong justification

### DO:
- Test terminal rendering changes in an actual terminal
- Consider performance for rendering loops
- Maintain backward compatibility with Go 1.25+
- Keep the visual style consistent (green land, blue ocean, colored flashes)

## Common Tasks

### Adding New Event Types
1. Extend `GitHubEvent` struct if needed
2. Update `fetchPushEvents` filtering logic
3. Update event display formatting
4. Test with mock data

### Modifying Globe Rendering
1. Understand the coordinate transformation (`isLand`, `renderGlobe`)
2. Test with different terminal sizes
3. Verify aspect ratio calculations
4. Consider performance impact on render loop

### Adjusting Animation
1. Locate the relevant goroutine (fade, spinner, flash, etc.)
2. Modify timing constants carefully
3. Test for smooth transitions
4. Ensure thread-safe access to shared state

## Performance Considerations

- The render loop runs at ~20fps (50ms intervals)
- Event fetching is asynchronous to avoid blocking rendering
- Mutex locks should be held for minimal duration
- Flash effects are pruned regularly to prevent memory leaks

## External API Usage

- **GitHub Events API**: `https://api.github.com/events`
  - Public API: 60 requests/hour
  - Authenticated (via `gh` CLI): 5,000 requests/hour
- **GitHub Topics API**: `repos/{owner}/{repo}/topics`
  - Requires authentication via `gh` CLI

## Development Tips

1. Test in a real terminal, not just an IDE terminal
2. Terminal size affects layout - test with various sizes
3. Use `screen.Sync()` after resize events
4. Debug rendering issues by adding temporary status messages
5. Rate limiting may affect event refresh - use `gh` CLI for development

---
> Source: [leereilly/gh-firehose](https://github.com/leereilly/gh-firehose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-26 -->
