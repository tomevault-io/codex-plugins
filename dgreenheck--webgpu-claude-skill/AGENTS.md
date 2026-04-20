
# Three.js WebGPU Compute Shaders (TSL)

Compute shaders in TSL use `Fn()` wrapped with `.compute(count)` and dispatched via `renderer.computeAsync()`. Data lives in `instancedArray()` buffers accessed by `instanceIndex`.

Key patterns:
- `instancedArray(count, 'vec3')` for storage buffers
- `Fn(() => { ... }).compute(count)` to define a compute kernel
- `renderer.computeAsync(kernel)` to dispatch
- Workgroup barriers via `workgroupBarrier()` for shared memory sync

@skills/webgpu-threejs-tsl/docs/compute-shaders.md
@skills/webgpu-threejs-tsl/examples/particle-system.js
@skills/webgpu-threejs-tsl/templates/compute-shader.js

---
> Source: [dgreenheck/webgpu-claude-skill](https://github.com/dgreenheck/webgpu-claude-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-20 -->
