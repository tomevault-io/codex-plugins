# Copilot Instructions for this Workspace

## Environment
- This workspace targets **Zulu JDK with JavaFX**:
  - Distribution folder: `zulu25.32.21-ca-fx-jdk25.0.2-win_x64`
- Treat this JDK as the canonical runtime and toolchain for all Java work in this repo.

## Required JAVA_HOME
- Use this as `JAVA_HOME` on Windows:
  - `C:\git\eval\javafx\zulu25.32.21-ca-fx-jdk25.0.2-win_x64`
- Assume `java`, `javac`, and `jlink` come from `%JAVA_HOME%\bin`.
- When generating terminal commands for this workspace, prefer:
  - PowerShell:
    - `$env:JAVA_HOME = 'C:\git\eval\javafx\zulu25.32.21-ca-fx-jdk25.0.2-win_x64'`
    - `$env:Path = "$env:JAVA_HOME\\bin;$env:Path"`
  - Bash (Git Bash):
    - `export JAVA_HOME='/c/git/eval/javafx/zulu25.32.21-ca-fx-jdk25.0.2-win_x64'`
    - `export PATH="$JAVA_HOME/bin:$PATH"`

## JavaFX Prototyping Goals
- Primary use case: **fast JavaFX prototypes**.
- Prefer minimal, runnable examples over heavy architecture.
- Keep code concise and easy to iterate.
- Use Java 25 language features only when they improve clarity.

## JavaFX Compile/Run Conventions
- This JDK already includes JavaFX modules in `jmods`.
- For modular runs, prefer:
  - `--module-path "%JAVA_HOME%\jmods" --add-modules javafx.controls,javafx.fxml`
- For simple non-modular prototypes, allow a single `Main` class plus optional FXML/CSS.
- Default starter app should extend `javafx.application.Application`.

## Suggested Defaults for Generated Code
- Target package: `prototype` unless user asks otherwise.
- Include a tiny `Hello JavaFX` scene for first-run validation.
- Keep dependencies to JDK + built-in JavaFX modules only.
- Do not introduce external UI frameworks unless requested.

## Build Tool Guidance
- If a build tool is requested:
  - Prefer **Maven** with explicit JavaFX modules and Java release set to 25.
  - Keep plugin/config minimal for prototype speed.
- If no build tool is requested:
  - Prefer direct `javac`/`java` commands using `%JAVA_HOME%`.

## What Copilot Should Optimize For
- Commands that run out-of-the-box in this workspace.
- Windows-friendly paths and examples first.
- Quick edit-run-debug loop for JavaFX UI experiments.
- Minimal boilerplate, clear comments only when useful.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peterkir)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/peterkir)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
