# jreg Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-10-23

## Active Technologies

- Java 21 (LTS) with virtual threads for high-concurrency performance + Spring Boot 3.2+ (web framework), AWS SDK for Java 2.x (S3 integration), Jackson (JSON processing), Micrometer (metrics), SLF4J + Logback (logging) (001-oci-registry-server)

## Project Structure

```text
src/
tests/
```

## Commands

### Build and Run
- **Compile**: `mvn compile -q`
- **Run application in background (PowerShell)**:
  ```powershell
  $job = Start-Job -ScriptBlock { Set-Location $PWD; $env:SPRING_PROFILES_ACTIVE="local"; mvn spring-boot:run 2>&1 }
  Start-Sleep -Seconds 5  # Wait for startup
  ```
- **Check job status**: `Get-Job | Select-Object Id, State`
- **View job logs**: `Receive-Job -Id <JobId> | Select-Object -Last 50`
- **Stop and restart**:
  ```powershell
  Get-Job | Stop-Job; Get-Job | Remove-Job -Force
  mvn compile -q  # If code was modified
  $job = Start-Job -ScriptBlock { Set-Location $PWD; $env:SPRING_PROFILES_ACTIVE="local"; mvn spring-boot:run 2>&1 }
  Start-Sleep -Seconds 5
  ```

### CRITICAL: Code Modification Workflow
When modifying Java code, you MUST follow this sequence:
1. **Edit the code** using appropriate tools
2. **Compile**: `mvn compile -q`
3. **Stop server**: `Get-Job | Stop-Job; Get-Job | Remove-Job -Force`
4. **Restart server**: `$job = Start-Job -ScriptBlock { Set-Location $PWD; $env:SPRING_PROFILES_ACTIVE="local"; mvn spring-boot:run 2>&1 }; Start-Sleep -Seconds 5`
5. **Test the changes**

**DO NOT rely on Spring DevTools auto-reload** - it may fail with ClassNotFoundException. Always stop and restart after code changes.

### Testing with ORAS
- **Check registry**: `curl http://localhost:5000/v2/`
- **Push artifact**: `oras push localhost:5000/<repo>:<tag> <file> --plain-http`
- **Pull artifact**: `oras pull localhost:5000/<repo>:<tag> --plain-http -o <output-dir>`
- **List tags**: `oras repo tags localhost:5000/<repo> --plain-http`
- **Fetch manifest**: `oras manifest fetch localhost:5000/<repo>:<tag> --plain-http`
- **Delete manifest**: `oras manifest delete localhost:5000/<repo>:<tag> --plain-http --force`
- **Attach referrer**: `oras attach localhost:5000/<repo>:<tag> --artifact-type <type> <file> --plain-http`
- **Discover referrers**: `oras discover localhost:5000/<repo>:<tag> --plain-http`
- **Delete referrer**: `oras manifest delete localhost:5000/<repo>@<digest> --plain-http --force`

## Code Style

Java 21 (LTS) with virtual threads for high-concurrency performance: Follow standard conventions

## Recent Changes

- 001-oci-registry-server: Added Java 21 (LTS) with virtual threads for high-concurrency performance + Spring Boot 3.2+ (web framework), AWS SDK for Java 2.x (S3 integration), Jackson (JSON processing), Micrometer (metrics), SLF4J + Logback (logging)

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shizhMSFT)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/shizhMSFT)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
