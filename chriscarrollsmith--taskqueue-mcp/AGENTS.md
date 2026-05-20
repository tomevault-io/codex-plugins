**CLI Testing**:
   - When testing CLI commands, pass the environment variable inline:
     ```typescript
     const { stdout } = await execAsync(
       `TASK_MANAGER_FILE_PATH=${tasksFilePath} tsx ${CLI_PATH} command`
     );
     ```
   - Use `tsx` instead of `node` for running TypeScript files directly

---
> Source: [chriscarrollsmith/taskqueue-mcp](https://github.com/chriscarrollsmith/taskqueue-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-19 -->
