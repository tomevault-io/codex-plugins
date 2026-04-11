When possible make code easy to change in the future.
Isolate distinct concerns into distinct components because that makes code easy to change.
Create components with single reponsibilities because that makes them easy to change.
Name things well becasue that makes code easy to read and make changes to.
Allow for design decisions to be reversed because that makes code easy to change when new requirements arrive.

Adhere to DRY (don't repeat yourself) principle if possible. Centralize definition of the same knowledge and use it in all places it's used. When possible attempt to refactor code to obey this principle.

Design systems to have orthogonal functionality.

Check for python typing information in method signatures is correct and if documentation needs to be updated to reflect existing funcionality.

## Documentation Style

Write documentation that is very direct and verbose. Detailed when needed but not explain too much extra context:
- Use clear, simple language
- Be comprehensive and detailed when information is needed
- Focus on what users need to do, not why or how it works internally
- Avoid redundant information and unnecessary context
- Keep code comments minimal - only explain complex logic, not obvious actions

When adding comments to code only explain current functionality and not how the code has recently changed.

Read `CONTRIBUTING.md` file in the root of the repository for complete contributing guidelines. Follow them closely

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/blackwhitehere)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/blackwhitehere)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
