# GSAP library for animation

When importing the plugin, because we're only using ESM, NOT CommonJS, then do it like this: `"gsap/dist/PluginName"`

```tsx
import { gsap } from "gsap";
import ScrollTrigger from "gsap/dist/ScrollTrigger";
import GSAPSplitText from "gsap/dist/SplitText";

gsap.registerPlugin(ScrollTrigger, GSAPSplitText);
```

---
> Source: [dogokit/dogokit-corgi](https://github.com/dogokit/dogokit-corgi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-14 -->
