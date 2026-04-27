# Vibe WPF Development Repository

This repository contains a WPF application with live inspection capabilities for rapid UI development and iteration.

## Project Structure

- **MyWpfApp/** - Main WPF application
- **VibeExtensions/** - Extension library providing HTTP server for live inspection
- **.github/skills/** - Agent Skills for AI-assisted development
  - **screenshot/** - Capture UI screenshots
  - **tree/** - Inspect visual tree structure
  - **restart/** - Restart the application

## Running the Application

Start the application in watch mode for hot reload during development:

```powershell
cd MyWpfApp
dotnet watch run
```

The app will automatically rebuild and restart when you modify `.cs` or `.xaml` files.

## Live Inspection Skills

This repository includes four Agent Skills for inspecting the running application. These are automatically discovered by Copilot and can be invoked based on your prompts.

### Start Skill
Start the WPF application in watch mode:
```powershell
.\.github\skills\start\start-app.ps1
```
Automatically waits for the app to be ready. Only needs to be run once per session.

### Screenshot Skill
Capture the current UI as a PNG screenshot:
```powershell
.\.github\skills\screenshot\get-screenshot.ps1
```
Saves with timestamp to `screenshots/` folder and opens it. Now includes automatic retry logic to wait for the app to be ready.

### Visual Tree Skill
Get the complete WPF visual tree as formatted JSON:
```powershell
.\.github\skills\tree\get-tree.ps1
```
Shows element types, names, and hierarchy.

### Restart Skill
Trigger `dotnet watch` to restart the application:
```powershell
.\.github\skills\restart\restart-app.ps1
```
Useful for applying XAML changes that don't hot reload automatically.

### Legacy Script
The original combined script is still available:
```powershell
.\get-vibe.ps1 [ui|tree|restart]
```

## Development Workflow for AI Assistants

When iterating on WPF UI:

1. **Start the app** (first time only):
   ```powershell
   .\.github\skills\start\start-app.ps1
   ```
   **CRITICAL**: When using the `run_in_terminal` tool, you MUST set `isBackground=true` because `dotnet watch run` and `dotnet run` block indefinitely. The terminal should remain open and running throughout your development session. The app will stay open and automatically rebuild/restart when files change.
   
   Alternatively, you can manually start in watch mode:
   ```powershell
   cd MyWpfApp
   dotnet watch run
   ```

2. **Make changes** to XAML or C# files (in your editor, not the terminal)

3. **Restart and capture results** using the skills:
   - `.\.github\skills\restart\restart-app.ps1` - Restart the app to apply XAML changes (now waits and verifies restart)
   - `.\.github\skills\screenshot\get-screenshot.ps1` - Verify visual appearance (includes automatic retry)
   - `.\.github\skills\tree\get-tree.ps1` - Inspect element structure and naming
   
   **Quick iteration**: Combine commands to restart and capture in one line:
   ```powershell
   .\.github\skills\restart\restart-app.ps1; .\.github\skills\screenshot\get-screenshot.ps1
   ```
   Note: No need for manual `Start-Sleep` anymore - the skills handle waiting automatically!

4. **Iterate** - Make changes, restart, and capture screenshots

**Note on Hot Reload**: WPF XAML hot reload is limited and often requires a restart for changes to apply. The restart skill automates this instead of manually pressing Ctrl+R in the watch terminal.

This enables rapid feedback loops without manual restarts.

## VibeServer Details

The VibeExtensions library provides an HTTP server (initialized in App.xaml.cs) that exposes:
- `http://localhost:5010/ui/` - PNG screenshot endpoint
- `http://localhost:5010/tree/` - JSON visual tree endpoint
- `http://localhost:5010/restart/` - Graceful shutdown endpoint

These endpoints allow external processes to observe the running application state.

---
> Source: [jonathanpeppers/vibe-wpf](https://github.com/jonathanpeppers/vibe-wpf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-26 -->
