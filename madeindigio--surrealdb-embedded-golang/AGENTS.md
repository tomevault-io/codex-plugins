# Surrealdb embedded library in Rust for Golang — Project Instructions

This is a shared and static library of surrealdb embedding features build in rust for Golang library

## 🛠️ Work Methodology

### Essential First Steps

1. **Check Serena onboarding**: Use `mcp_serena_check_onboarding_performed`
2. **Read memories**: Use `mcp_serena_list_memories` and read relevant ones for context
3. **Search knowledge base**: Use remembrances tools for recovering related information for the task and before context
4. **Check the plan**: Review `.serena/memories/plan.md` for current tasks

### Development Workflow

- Use always english as language for all code, comments, and documentation although the user may write in other languages
- **Knowledge storage**: Save findings using remembrances tools for future reference

### External Research

- Use web search (google/perplexity/brave) for additional information when needed
- Use Context7 for API documentation and library usage patterns

### Structure of the project

- The folder `surrealdb_embedded_rs` contains the Rust code for the surrealdb embedded library, must to implement the same surrealdb golang library features only for embedding surrealdb on golang projects without external surrealdb server dependencies.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/madeindigio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
