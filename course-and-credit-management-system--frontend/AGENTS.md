# UniPortal Copilot Instructions

Follow these standards for all code generation and project assistance.

## Business Rules
- **Credit Limit**: Enforce the 24-credit ceiling unless an admin override is detected.
- **Priority**: Always prioritize `is_retake: true` enrollments over new course selections.
- **Authentication**: Ensure `must_reset_password` checks are handled at the route level.

## Styling & UI
- **Primary Brand**: `#0d4a8f` (BG/Buttons), `#ffc20e` (Alerts).
- **Table Design**: `bg-[#0d4a8f]` headers with white text. Alternating row colors (`#ffffff` and `#f5f5f5`).
- **Icons**: Use Google Material icons via standard `material-icons-outlined` classes.

## Architecture
- Use **Generics** in API requests to maintain type safety.
- Centralize all constants and interfaces in [types.ts](types.ts).
- Refer to [lib/api.ts](lib/api.ts) for all backend communication logic.

## Environment & Scripts
- **Terminal Execution**: Keep in mind the active environment runs **PowerShell** on **Windows**.
- **No Bash Syntax**: Never use bash `heredoc` (`cat << 'EOF'`) or `> file` redirections with complex content directly.
- **Node Scripts**: When creating patch scripts, use the `create_file` tool to create a physical `.cjs` file and then execute it via `node file.cjs`. Do not run inline `node -e "..."` scripts with quotes, as PowerShell escaping causes `SyntaxError: Invalid or unexpected token`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Course-and-Credit-Management-System) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-11 -->
