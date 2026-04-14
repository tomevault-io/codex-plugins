# CLAUDE.md

@agents.md

## Claude Code-Specific Notes

### File Path Requirement
Always use **absolute Windows paths** with drive letters and backslashes for all file operations.
- Correct: `D:\projects\KafkaLens\ViewModels\MainViewModel.cs`
- Wrong: `./ViewModels/MainViewModel.cs`

### Memory
Persistent memory for this project is stored at:
`C:\Users\Pravin Chaudhary\.claude\projects\D--projects-KafkaLens\memory\`

### Workflow
- Do not run the application — the user runs it themselves.
- Do not ask the user to run commands — show the commands and let them decide.
- Always read a file before editing it.
- Use `TodoWrite` to track multi-step tasks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fatichar)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/fatichar)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
