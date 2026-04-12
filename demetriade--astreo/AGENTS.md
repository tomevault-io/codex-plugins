
# Astreo System Boundaries

- Never modify files under `PX4-Autopilot` unless explicitly requested by the project owner.
- Treat the astreo workspace (`astreo/src/`) as the primary implementation area for autonomy logic (control, behavior, interfaces, bringup).
- Keep flight controller integration behind the FlightControllerAdapter boundary; MAVROS is one implementation, px4_ros_com or others can be added without changing control/behavior logic.
- Prefer MAVLink abstraction at the adapter edge so FC/protocol changes are manageable.
- Design behavior to remain safe and autonomous when internet connectivity is unavailable.
- Cloud/backend components must not be mandatory for core mission execution.
- Prefer PX4 mission/auto modes as baseline control; use Offboard only for short, bounded reactive maneuvers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Demetriade)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Demetriade)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
