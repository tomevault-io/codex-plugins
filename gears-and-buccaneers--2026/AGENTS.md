# GitHub Copilot / Cursor Instructions for FRC Development

You are assisting with a **FIRST Robotics Competition (FRC)** project. This repository uses **Python**, **RobotPy**, and the **MagicBot** framework, with **CTRE Phoenix 6** for swerve.

Some Cursor setups have **MCP servers** (documentation search, issue trackers, etc.). **Do not assume** a specific MCP tool name exists. If the user has configured FRC or vendor doc search via MCP, use those tools when they are available; otherwise rely on **[RobotPy documentation](https://robotpy.readthedocs.io/)**, **[WPILib documentation](https://docs.wpilib.org/)**, and vendor sites (e.g. **[Phoenix 6](https://v6.docs.ctr-electronics.com/en/stable/)**, **[Choreo](https://choreo.autos/)**, **[PhotonVision](https://docs.photonvision.org/)**).

## When answering FRC questions

1. Prefer **current official docs** and the **declared RobotPy / game year in this repo** (see `pyproject.toml` and `README.md`) over memory alone.
2. **Default to the 2026 FRC season** unless the user or repo clearly specifies another year.
3. For RobotPy specifics (MagicBot, `wpilib`, simulation, deploy), point to RobotPy and WPILib Python docs.
4. If you cannot access documentation (no network, no MCP, or tools missing), say so and give a best-effort answer with clear caveats.

## Language and stack

- This codebase is **Python** with **RobotPy**; do not default to Java/C++ Command-based examples unless the user asks for them.
- Use **modern Python** typing (`list[str]`, `X | None`) consistent with the rest of the repo.

## Code style for FRC (this repo)

- Follow existing patterns: MagicBot components, Phoenix 6 swerve API, `constants.py` / `generated/` for robot parameters.
- Use vendor APIs correctly (Phoenix 6 for TalonFX / swerve, Photon when relevant).
- Handle **units** carefully (RPS vs rad/s, field vs robot frame).
- Prefer concise, purposeful comments; match the project’s docstring style (Google convention per Ruff).

## When docs are insufficient

1. Try broader search terms or the vendor’s own documentation site.
2. Be explicit when you are inferring from hardware behavior or similar seasons.

## Example interaction

**Student:** "How does Phoenix 6 swerve report velocity?"

**You should:** Summarize from CTRE Phoenix 6 docs (units, `get_state`, etc.), cite the official URL, and relate it to this repo’s `Drivetrain` / `SwerveDrivetrain` usage when relevant.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Gears-and-Buccaneers)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Gears-and-Buccaneers)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
