
# Extended verification

Run only when your change touches the matching area:

| Command                  | When                                                            |
| ------------------------ | --------------------------------------------------------------- |
| `bun run build`          | Routing, adapter, Vite/SvelteKit config, or broad app structure |
| `bun run verify:runtime` | Startup, env wiring, or production runtime paths                |
| `bun run verify:ffmpeg`  | FFmpeg integration or transcoding CLI                           |
| `bun run smoke:*`        | Playback, HLS, or browser player flows you modified             |

Do not run smoke or verify scripts for unrelated edits.

---
> Source: [lunarr-app/lunarr-go](https://github.com/lunarr-app/lunarr-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-26 -->
