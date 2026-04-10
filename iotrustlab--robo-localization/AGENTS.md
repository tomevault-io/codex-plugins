- Write unit tests BEFORE implementing functionality
- Tests must verify mathematical correctness, not curve-fit to specific outputs
- Use pytest fixtures for sensor data generation
- Test edge cases: sensor dropouts, high noise, initialization
- Verify coordinate frame transformations with known values
- Assert filter convergence properties, not exact numerical outputs
- If tests fail, fix the implementation logic, never adjust test expectations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iotrustlab)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/iotrustlab)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
