# Instructions for Claude AI Assistant

## Project Context

This is a mathematical concepts notebook project for university-level mathematics courses (Calculus 1-4, Statistics, AI, etc.). Students use Jupyter notebooks for interactive learning.

## Notebook Interaction Guidelines

### 1. Import Management

**Always keep imports organized at the top of notebooks:**
- When adding new functionality that requires a package, add the import to the top cell
- Don't create separate import cells - always update and rerun the first cell. This keeps imports centralized and organized.
- If a package is missing from the environment, add it to `missing_packages.md`
- Keep imports grouped logically (standard library, third-party, local)

Example:
```python
# Standard library
import math

# Third-party packages
import numpy as np
import matplotlib.pyplot as plt
import sympy as sp
```

### 2. Explaining Concepts (Q&A Format)

When a user asks a question about a mathematical concept:

1. **Start with the prompt:** Create a markdown cell with the user's question/prompt
2. **Explain step-by-step:** Alternate between markdown cells (explanations) and code cells (demonstrations)
3. **One concept at a time:** Only answer what was asked - don't continue to solutions or related topics
4. **Interactive learning:** Wait for the user to ask follow-up questions rather than assuming what they want next

**Structure:**
- Cell 1 (Code): Import statements (update and rerun if new packages are needed)
** Answering the user **
- Cell 2 (Markdown): User's question/prompt
- Cell 3 (Markdown): Explanation of concept
- Cell 4 (Code): Code demonstration, visualization, or example
- Cell 5 (Markdown): Interpretation of results

### 3. Stay Focused

**DO:**
- Answer the specific question asked
- Provide clear, concise explanations
- Show working code examples
- Have patience and wait for follow-up questions

**DON'T:**
- Answer the exercise unless specifically asked to
- Continue to full solutions unless asked
- Assume the user wants related concepts explained
- Over-explain beyond the immediate question

## Key Reminder

**MOST IMPORTANT:** When new packages are needed, always add imports to the top cell and rerun it. Never create separate import cells throughout the notebook.

---
> Source: [PerErikGronvik/EASY_Claude_math_tutor](https://github.com/PerErikGronvik/EASY_Claude_math_tutor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-16 -->
