# Instructions for AI Agents

## Repository Purpose

This repository contains comprehensive senior-level technical notes for:
- Microservices
- System Design
- Java
- REST API
- Low-Level Design (LLD)
- DBMS
- NoSQL

## Critical Workflow: Git Commits

**MANDATORY: After completing EACH task, you MUST commit and push:**

```bash
git add .
git commit -m "Descriptive message about completed work"
git push
```

### When to Commit

Commit immediately after:
- Creating any new note file
- Completing a section expansion
- Adding code examples or diagrams
- Finishing any logical unit of work
- Completing user-requested changes

**Do NOT wait to batch multiple tasks before committing. Commit after EACH completed task.**

## Content Standards

### Structure for Each Note

1. **What You'll Learn** - Clear learning objectives
2. **Why This Matters** - Real-world context and motivation
3. **Detailed Explanations** - Comprehensive technical depth
4. **Code Examples** - Production-quality examples in multiple languages (Java, JavaScript, Python, etc.)
5. **Real-World Scenarios** - Practical applications
6. **Comparison Tables** - Trade-offs and alternatives
7. **Best Practices** - Industry-standard approaches
8. **Anti-Patterns** - Common mistakes to avoid with ❌ markers

### Code Quality

- Use proper syntax highlighting with language identifiers
- Include inline comments for complex logic
- Provide complete, runnable examples (not pseudocode)
- Show multiple language implementations where relevant
- Use realistic variable names and scenarios

### Technical Depth

- Senior-level content: assume strong programming background
- Cover production concerns: scalability, performance, security
- Include specific metrics, numbers, and benchmarks
- Reference RFCs, standards, and industry best practices
- Explain trade-offs and decision criteria

### Formatting Rules

- Use Mermaid diagrams for architecture flows
- No emojis except ✅ and ❌ for good/bad examples
- No motivational language
- Prefer connected explanations over bullet lists
- Use bullets only for lists, comparisons, and enumerations
- Link to related files using relative paths

## File Organization

```
notes/
  ├── microservices/
  ├── system-design/
  ├── java/
  ├── rest-api/
  ├── lld/
  ├── dbms/
  └── nosql/
```

Each directory should have:
- Numbered files (01-, 02-, etc.) for sequential topics
- README.md with navigation and overview

## Expansion Guidelines

When expanding condensed notes:
- Preserve all existing content
- Add comprehensive explanations with context
- Include multiple code examples
- Add real-world scenarios and use cases
- Include comparison tables where relevant
- Maintain consistent depth across all sections

## Commit Message Format

```bash
# Good examples:
git commit -m "Add REST API fundamentals with HTTP semantics"
git commit -m "Expand Java concurrency with ThreadPool examples"
git commit -m "Add microservices service discovery patterns"

# Bad examples:
git commit -m "Update files"
git commit -m "WIP"
git commit -m "Changes"
```

## Response Style

- Be concise in chat responses
- Focus on action, not explanation of what you're about to do
- After completing work, confirm what was done (1-2 sentences)
- No need to explain which tools you're using
- Provide file links using markdown format

## Error Handling

If you encounter issues:
1. Read relevant files to understand context
2. Check file structure and naming conventions
3. Ensure changes align with existing patterns
4. Commit working progress before attempting fixes

## Remember

**The most important instruction: COMMIT AFTER EVERY COMPLETED TASK**

This ensures:
- Incremental progress is saved
- Work can be tracked and reviewed
- Changes can be rolled back if needed
- Collaboration is easier

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jot-singh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
