# Copilot Instructions — Hyperscanning Workshop

## Language
All code, comments, docstrings, and documentation must be written in English.

## File Management
Never create, delete, or edit any file without explicit user consent.

## Project Structure
- Notebooks: `01_foundations/` and `02_connectivity_metrics/`, by category (A, B, C...)
- Reusable functions: `src/` directory
- Naming: `A01_`, `A02_`, `B01_`, etc.
- Quick notebooks: `*_quick.ipynb` (import from src/ instead of inline definitions)

## Git Workflow
- One branch per notebook: `feature/<notebook_id>`
- Clear commit messages
- Merge to main after review

## Dependencies
Managed with Poetry. Core: numpy, scipy, matplotlib, mne, hypyp.

## Typed Python
- Complete type hints on all functions (params + return)
- Use `numpy.typing.NDArray` for arrays
- Use `typing` module (Optional, Union, Tuple, Callable)
- mypy strict compatible

## Coding Standards
- PEP 8 compliant
- NumPy-style docstrings
- Functions over 20 lines go to `src/`
- No hardcoded values

## Notebook Structure
- **Full notebooks**: See `docs/NOTEBOOK_TEMPLATE.md`
- **Quick notebooks**: See `docs/NOTEBOOK_QUICK_TEMPLATE.md`
- Structure: Introduction → Intuition → Implementation → Visualization → HyPyP comparison → Application → Summary → Discussion

## Notebook Cell Order (CRITICAL)
For ALL notebooks (full and quick):
1. Cell 1 = Header (`# [ID]: Title` + Duration + Prerequisites + Learning Objectives)
2. Cell 2 = Table of Contents (`## Table of Contents` with `#section-X-...` anchors)
3. Cell 3 = Imports (Code cell with `# =====` style headers)
4. Cells 4+ = Sections
5. Last 3 cells = Summary, External Resources, Discussion Questions

## Colors (CRITICAL)
- Always import: `from colors import COLORS`
- Use ONLY these keys: `signal_1`, `signal_2`, `signal_3`, `signal_4`, `signal_5`, `signal_6`
- For EEG bands: `delta`, `theta`, `alpha`, `beta`, `gamma`
- NEVER use: `primary`, `secondary`, `accent1`, `accent2` (these keys DO NOT EXIST)

## NotebookLM Resources (CRITICAL)
In External Resources section, use this EXACT format:
```markdown
### 🎧 NotebookLM Resources

- [📺 Video Overview](URL) - Video overview of [topic] concepts
- [📝 Quiz](URL) - Test your understanding
- [🗂️ Flashcards](URL) - Review key concepts
```
- ALWAYS say "Video" — NEVER say "Audio"
- The 📺 emoji is for VIDEO, not audio
- Description must start with "Video overview of..."

## Exercises Format (CRITICAL)
Use this EXACT format for exercises:
```markdown
---

<a id="section-N-exercises"></a>
## N. Exercises

### 🎯 Exercise 1: Title

**Task:** Description of what to do.

- Bullet point instructions
- More instructions

\`\`\`python
# Your code here
\`\`\`

<details>
<summary>💡 Click to reveal solution</summary>

\`\`\`python
# Solution code
\`\`\`

</details>
```
- Always include 🎯 emoji before exercise title
- Always use `<details>` for solutions
- Always include `<a id="..."></a>` anchor before section title

## Visualization
Follow STYLE_GUIDE.md for colors, dimensions, fonts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ramdam17)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Ramdam17)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
