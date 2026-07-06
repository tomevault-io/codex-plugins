
# Three.js WebGPU Post-Processing (TSL)

Post-processing uses the `PostProcessing` class from `three/webgpu` with node-based effect composition. Effects chain via TSL nodes rather than EffectComposer passes.

```javascript
import { PostProcessing } from 'three/webgpu';
import { pass, mrt, output, emissive } from 'three/tsl';
import { bloom } from 'three/addons/tsl/display/BloomNode.js';

const postProcessing = new PostProcessing(renderer);
const scenePass = pass(scene, camera);
postProcessing.outputNode = scenePass.add(bloom(scenePass.getTextureNode()));
```

Call `postProcessing.renderAsync()` instead of `renderer.renderAsync()`.

@skills/webgpu-threejs-tsl/docs/post-processing.md
@skills/webgpu-threejs-tsl/examples/post-processing.js

---
> Source: [hexianWeb/lego-stylized-nature](https://github.com/hexianWeb/lego-stylized-nature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-06 -->
