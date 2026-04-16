- **Security Best Practices:**
  - **Common Vulnerabilities:**
    - **SQL Injection:** Prevent SQL injection by using parameterized queries or an ORM that automatically escapes user inputs.
    - **Cross-Site Scripting (XSS):** Protect against XSS by sanitizing user inputs and encoding outputs.
    - **Cross-Site Request Forgery (CSRF):** Implement CSRF protection using tokens or other mechanisms.
    - **Authentication and Authorization Flaws:** Secure authentication and authorization by using strong passwords, multi-factor authentication, and role-based access control.
    - **Insecure Direct Object References (IDOR):** Prevent IDOR by validating user access to resources before granting access.
  - **Input Validation:**
    - Validate all user inputs to prevent malicious data from entering the system. Use DTOs and validation pipes to enforce input constraints.
    - Sanitize user inputs to remove or escape potentially harmful characters.
    - Validate file uploads to prevent malicious files from being uploaded.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SOMONSOUM)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/SOMONSOUM)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
