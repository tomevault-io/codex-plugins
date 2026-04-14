<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Optics Ring Generator Project Instructions

This is a Rust CLI application for generating 3D printable precision optics support rings.

## Project Context
- Generate STL files for three types of support rings: Convex (CX), Concave (CC), and Three-point (3P)
- Input parameters: ring type, outer diameter, inner diameter
- Output: STL files named as "form-innerdiameter.stl"
- Target use case: 3D printing precision optics support rings

## Code Style Guidelines
- Use idiomatic Rust patterns and error handling
- Implement proper CLI argument parsing with clap
- Use appropriate geometric libraries for 3D mesh generation
- Follow Rust naming conventions (snake_case for functions, PascalCase for types)
- Include comprehensive documentation and examples

## Key Requirements
- CLI interface in English
- Generate valid STL files for 3D printing
- Support three ring geometries: convex, concave, three-point
- File naming convention: {type}-{inner_diameter}.stl

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/max-flug) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
