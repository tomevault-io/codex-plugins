# Copilot Instructions

## Project Guidelines
- Database column 'role' in engineers table needs to be VARCHAR(20) or larger to support all EngineerRole enum values including 'ACCOUNTANT' (9 chars). The fix involves updating BdaDbContext.cs line 29 to include .HasMaxLength(20) and then running EF Core migrations to update the actual database column.

---
> Source: [kdn8gbqph2-jpg/Work-Dashboard](https://github.com/kdn8gbqph2-jpg/Work-Dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-27 -->
