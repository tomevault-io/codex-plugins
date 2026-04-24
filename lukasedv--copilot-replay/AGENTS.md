# Copilot instructions for `copilot-replay`

Guidance for GitHub Copilot (and humans) working in this repository.

## What this project is

`copilot-replay` is a **standalone terminal app** that replays a GitHub
Copilot CLI session from its `events.jsonl`. It is used on stage for
demos: prerecord the work, replay it live, at controllable speed, with
a look close to the real Copilot CLI.

It does **not** contact the Copilot service, load the model, or
re-execute any tool. It reads `events.jsonl`, filters the interesting
events, and re-renders them with timing and animations.

## Stack and conventions

- **Node.js 18+**, pure ES modules, **zero runtime dependencies**. Do
  not add dependencies without a very strong reason. The "clone and
  `npm install -g .`" install story is a feature.
- **No TypeScript / no build step.** Type hints go in JSDoc if needed.
  Source runs directly.
- **Terminal I/O everything.** Output goes through
  `src/io.js#rawWrite` / `writeln`. All color goes through `src/ansi.js`
  (`fg.*`). Width measurement uses `stripAnsi`.
- **Layout is a singleton.** `src/layout.js` owns the scroll region and
  sticky footer. Anything that writes needs to respect
  `layout.scrollBottomRow` — this is why `io.js` exists.
- **Sanitize untrusted input.** Every string pulled from `events.jsonl`
  and `workspace.yaml` flows through `sanitizeString` / `sanitizeDeep`
  in `src/ansi.js` (see `sessions.loadEvents` / `listSessions`).
  Terminal-control bytes from a crafted session file would otherwise
  hijack the clipboard (OSC 52), rewrite the title, inject hyperlinks,
  etc. **Never render untrusted strings directly.**
- **Terminal restore on every crash path.** `src/cli.js#main` installs
  one idempotent `restore()` and wires it to `uncaughtException`,
  `unhandledRejection`, `SIGINT`, `SIGTERM`, and normal exit. Raw mode
  must never leak to the user's shell.

## Module layout

See `README.md` → "Project layout". The dependency DAG (no cycles):

```
ansi → io → format → markdown | banner | layout → anim → render
                                                         → player → keyboard
                                                                  → cli
sessions (stdlib only) → picker
```

When adding code:

- Put color/escape helpers in `ansi.js`.
- Put output helpers that interact with the footer in `io.js`.
- Put pure string helpers in `format.js` or `markdown.js`.
- Put rendering for a specific event type in `render.js#describeEvent`.
- Put timing/seek/step logic in `player.js`.
- Put key bindings in `keyboard.js`.

## Behaviour that must not regress

- Replay starts paused in a TTY. `space` begins playback.
- **Arrow keys** are context-sensitive:
  - playing → jump to the next/previous user prompt (landing just
    before typing starts);
  - paused → step forward/back one renderable event at a time,
    re-pausing at each landing point.
- User prompts always type at a fixed, human-readable pace — **not**
  scaled by `--speed`. Assistant thinking/output and tool output are
  scaled.
- Footer spans the full terminal width. Resizing redraws cleanly.
- Speed multiplier is always shown in the status bar (`4x`, `0.5x`, …).
- Non-TTY / piped output autoplays at `--speed` and does not use raw
  mode.

## When making changes

1. Edit source under `src/` or `bin/`.
2. Run `npm run check` (syntax check on every file).
3. Smoke test non-interactively:
   `node bin/copilot-replay.mjs <some-session-id> --speed 64 | tail`
   — the last line should be `━ replay complete`.
4. Smoke test in a real TTY if keybindings or layout changed.
5. Update `README.md` if user-visible behaviour changes.
6. Update `CHANGELOG.md` under the current unreleased section.
7. Bump `package.json` version on a release (semver).

## Commit style

- Short, imperative subject line ("Add step-through mode when paused").
- Body explains *why*, not just *what*.
- Include the `Co-authored-by: Copilot` trailer when Copilot helped.

## What not to do

- Don't add dependencies. Don't pull in `chalk`, `ink`, `blessed`, etc.
  The whole point is zero-install.
- Don't re-run Copilot or any model. This is a **replay**, not a
  simulator. No network, no subprocess for tools.
- Don't write to files outside `stdout` / `stderr`. No cache dirs,
  telemetry, crash dumps, etc.
- Don't bypass `sanitizeString` for event content. Any new event type
  added to `describeEvent` inherits the ingest-time sanitization — keep
  it that way.
- Don't introduce a build step or transpiler.

---
> Source: [Lukasedv/copilot-replay](https://github.com/Lukasedv/copilot-replay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-24 -->
