<!-- Project-specific Copilot / AI instructions. Keep short and actionable. -->
# Quick orientation for AI code agents

This is a WPILib-based FRC robot project (Gradle + WPILib). The goal of these notes is to help an AI agent be productive immediately by pointing to the repository's structure, key patterns, and the exact places to look for examples.

- Project layout (important paths)
  - `src/main/java/frc/robot/` — main robot code. Subpackages: `subsystems`, `commands`, `util`, `generated`.
  - `src/main/deploy/pathplanner` and `deploy/pathplanner` — Pathplanner trajectories and autos.
  - `build.gradle`, `gradlew(.bat)` — use the Gradle wrapper for builds and WPILib tasks.
  - `vendordeps/*.json` — vendor library manifests (Phoenix6, Pathplanner, Photon, AdvantageKit, etc.).

- Build & run (Windows developer examples)
  - Build: run the Gradle wrapper from project root: `.
    \gradlew.bat build` (PowerShell). Use `.
    \gradlew.bat test` to run unit tests if added.
  - Common WPILib tasks are available via the wrapper (for example `deploy`) — prefer the wrapper to the system Gradle.

- Key patterns & conventions (search these to reason about code)
  - Subsystems extend `SubsystemBase` and live in `src/main/java/frc/robot/subsystems`.
    Example: `HarvesterSubsystem.java` demonstrates motor configuration, Shuffleboard telemetry, and CAN-status checks.
  - Commands live in `src/main/java/frc/robot/commands` and usually call subsystem methods and `addRequirements(...)`.
    Example: `HarvesterDeploy.java` uses `harvester.setHarvDeployMagicMoPos(...)` and toggles spin when deployed.
  - Centralized constants: `Constants.java` and `BuildConstants.java`. Hardware IDs, gear ratios and conversion factors (e.g. `Constants.Harvester.DEPLOY_DEGREES_TO_ROTATIONS`) are authoritative — change there, not scattered throughout code.
  - Vendor API usage: CTRE Phoenix 6 is used (`com.ctre.phoenix6.*`). Look for `TalonFX`, `getConfigurator().apply(...)`, `MotionMagicVoltage`, `PositionVoltage`, `VelocityVoltage` when modifying motor control code.
  - Units: code uses `edu.wpi.first.units` conversions; follow existing unit patterns (do not mix raw rotations/deg/RPS without applying the constants in `Constants.Harvester`).

- Device & runtime diagnostics
  - Many subsystems publish Shuffleboard entries (see `HarvesterSubsystem.log()`): useful telemetry names: "Harv Deploy Pos (deg)", "Harv Spin Velo (RPS, output)", and the "CAN Status" tab.
  - CAN presence is checked via `StatusSignal.getStatus().isOK()` — prefer that check before assuming a device is present.
  - Motor configuration often retries `getConfigurator().apply(...)` in a loop — follow that retry pattern when adding new CAN devices.

- Integration points & data flows to be aware of
  - Pathplanner trajectories are in `deploy/pathplanner` and `src/main/deploy/pathplanner` — autonomous sequences reference these assets.
  - Code generation and generated sources are under `generated/` and `main/.../generated` — avoid editing generated files directly.

- Small examples to look up quickly
  - Configure motor + motion magic: `HarvesterSubsystem` (use of `MotionMagicConfigs`, `Slot0Configs`, `FeedbackConfigs`)
  - Position/velocity commands: `setHarvDeployMagicMoPos(...)`, `setDeployVelocity(...)`, `setSpinVelocity(...)` in `HarvesterSubsystem`.
  - Encoder zeroing: `deployMotor.setPosition(0.0, 0.05)` (note the short timeout parameter).

- Guidance for code changes
  - When adding hardware control, add constants in `Constants.*` first and use them throughout.
  - Keep Shuffleboard key names stable to avoid breaking dashboards.
  - Prefer using existing subsystem helper methods (e.g., `stopDeploy()`, `stopSpin()`, `zeroDeployEncoder()`) rather than manipulating motors directly from commands.

If anything above is unclear or you'd like more coverage (CI, unit-test layout, or example refactors), tell me which area to expand and I'll update this file accordingly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sean-zhen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
