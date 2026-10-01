# GitHub Copilot instructions

This repository follows the cross-tool `AGENTS.md` standard. Read [`/AGENTS.md`](../AGENTS.md) for the full guide: the `MeshFracture` API, the bake-at-load / look-up-at-impact pipeline, the material rules, build and preview commands, and the pull-request checklist.

The rules that matter most:

- **"Burst" here is the `FractureBurst` explosion component, not Unity's Burst compiler.** The runtime is single-threaded managed C# over `UnityEngine` types. Never add `[BurstCompile]`, `IJob*`, `NativeArray<T>`, or `Unity.Mathematics` / `Unity.Collections` / `Unity.Jobs` usings — none are dependencies.
- **Bake at load, look up at impact.** `MeshFragmenter.Fragment` is a synchronous O(n²) CPU call; call `FragmentCache.RequestPreBake` on a loading screen, then `FragmentCache.TryGet` at the moment of impact.
- **Fragment materials must be transparent, per-burst instances, with ZWrite ON.** Copy `MaterialFactory.ConfigureFractureTransparent`; don't reintroduce a custom `.shader` (it renders black on WebGL2).
- **Source meshes must be Read/Write-enabled**, or `Fragment` reads empty vertex arrays.
- **Two submeshes per fragment:** 0 = exterior, 1 = cap/interior — pass both materials to `Initialize`.
- **No `using LeapOfLegends.*`.** Runtime code lives in the `MeshFracture` namespace; demo code under `MeshFractureDemo`.

---
> Source: [sinanata/unity-mesh-fracture](https://github.com/sinanata/unity-mesh-fracture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-10-01 -->
