# GitHub Copilot Instructions

## Command Execution Standards

### PowerShell Command Execution

When generating PowerShell commands:

- Use semicolons (`;`) as command separators instead of ampersands (`&&`) 
- Example: `command1; command2; command3` (correct) vs `command1 && command2 && command3` (incorrect for PowerShell)
- For conditional execution:
  - Use `if ($?) { command2 }` instead of `command1 && command2`
  - Use `command1; if (-not $?) { command2 }` instead of `command1 || command2`
- For background processes, use `Start-Process` or `&` operator
- For pipeline commands, use the pipe operator (`|`) as normal

### Bash/CMD Command Execution

When generating bash or CMD commands:

- Use `&&` for conditional execution (execute second command only if first succeeds)
- Use `||` for fallback execution (execute second command only if first fails)
- Use `;` for sequential execution (execute all commands in order)

### Documentation Format

- Always include clear explanations of what each command does
- For complex commands, break them down with comments
- For commands that might have side effects, include warnings

## Best Practices

- Always consider the specific shell environment when generating commands
- Escape special characters appropriately for the target shell
- Use absolute paths when file locations are known
- Consider error handling appropriate to the shell

## Debugging Workflows

### C# Semantic Model MCP Server

- Always check for these common issues first:
  - Port availability before binding (`Test-NetConnection -ComputerName localhost -Port 8080`)
  - Configuration file existence and validity (`Test-Path "path\to\mcp.json"`)
  - Docker container status (`docker ps -a | Select-String "semantic-model"`)
  
- Standard diagnostic commands:
  ```powershell
  # View process logs with masking for sensitive data
  ./publish/SemanticModelMcpServer.exe --verbose --mask-tokens;
  
  # Check Docker logs
  docker logs $(docker ps -q --filter "name=semantic-model") | Select-String -Pattern "ERROR|WARN";
  
  # Test network configuration
  Test-NetConnection -ComputerName localhost -Port 8080 | Format-List;
  ```

## VS Code Integration

```markdown
### VS Code Debugging Integration

- Launch configurations should be stored in `.vscode/launch.json`
- Debugging presets:
  - Use the "Attach to Process" configuration for running Docker instances
  - Use the ".NET Core Launch" configuration for local debugging
  
- Debug output commands:
  ```powershell
  # Clear terminal and output
  Clear-Host; Clear-Content -Path "logs/debug.log";
  
  # Watch log file in real-time
  Get-Content -Path "logs/debug.log" -Wait;
  ```

## Documentation Standards

```markdown
### Debugging Documentation Standards

- Always include these sections when documenting issues:
  - **Issue Description**: Clear statement of the problem
  - **Reproduction Steps**: Numbered list of steps to reproduce
  - **Root Cause**: Identified source of the issue
  - **Solution**: How the issue was resolved
  - **Prevention**: Changes made to prevent recurrence
  
- For critical issues, include before/after code snippets
- Reference specific log lines with line numbers
```

## Agent Debugging Workflow

- Run the debugging prompt using VS Code's Copilot interface:
  ```powershell
  # Open the prompt in VS Code
  code "d:\repos\promts\debugger-prompt.md";
  
  # Execute prompt with Copilot
  # 1. Select the prompt text
  # 2. Right-click and select "Copilot: Send to Chat"
  # 3. Or use keyboard shortcut: Shift+Ctrl+I
  ```

- Follow the self-modification directives in the prompt to update documentation
- Reference the lessons learned in debugging-mcp-server.md when troubleshooting similar issues

---
> Source: [RuniThomsen/mcp-fabric](https://github.com/RuniThomsen/mcp-fabric) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-29 -->
