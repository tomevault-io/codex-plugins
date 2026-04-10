# DragonRuby repo guardrails for AI agents

- Scope: this repo is the DragonRuby Game Toolkit drop (engine + docs + samples). The only game code you own lives under `mygame/`; everything else is reference material.
- Entry point: `mygame/app/main.rb` defines a single `tick args` function that the engine calls 60 fps. Keep all frame work inside `tick`; avoid extra threads or timers.
- State: use `args.state` for per-frame or persistent game data (Ruby object with lazy init via `||=`). Do not create globals.
- Rendering: write to `args.outputs` each frame; common lists are `labels`, `sprites`, and shapes. Either hash keys (`{ x:, y:, w:, h:, path:, angle: }`) or ordered arrays (`[x, y, w, h, path, angle]`) are accepted; see `mygame/app/main.rb` for the hash style.
- Coordinates: virtual canvas is always 1280x720; origin (0,0) is bottom-left. Engine letterboxes automatically, so no resolution branching.
- Input: read from `args.inputs.keyboard`, `args.inputs.mouse`, and controller helpers; patterns are in `mygame/app/main.rb` and `docs/guides/getting-started.md`.
- Hot reload: the runtime watches files—save changes while `dragonruby.exe` is running to see live reloads. No explicit build step.
- Running locally (Windows): from repo root run `./dragonruby.exe` (or double-click). To explore samples, pass the sample folder: `./dragonruby.exe samples/01_rendering_basics`.
- Assets: place game art/sfx under `mygame/sprites`, `mygame/sounds`, `mygame/fonts`, `mygame/data`; reference by filename in outputs (lazy-loaded on first use).
- Docs: authoritative API docs live in `docs/api/*.md`; tutorials in `docs/guides/`; sample patterns in `samples/` (ordered by difficulty). Mirror is at docs.dragonruby.org.
- Common patterns: keep initialization inline with first use (via `||=`), use `args.state.tick_count`/`Kernel.tick_count` for time, and favor simple data structs over classes.
- Performance: engine targets 60 fps without delta time; movement is typically per-frame integer math (see `docs/guides/getting-started.md` and samples for pacing).
- C extensions: reference `samples/12_c_extensions/*` for the expected directory layout and build scripts if adding native code.
- Publishing: `dragonruby-publish.exe` handles packaging; if you touch build/publish flows, prefer reusing that tool instead of custom scripts.
- Source control etiquette: this drop may contain vendor files; avoid modifying engine binaries (`dragonruby*.exe`) or docs unless intentionally updating upstream content.
- Contribution focus: prioritize changes in `mygame/` or new sample-style content; keep sample code minimal and readable for teaching purposes.

## Unit Testing

- Running tests: `./dragonruby mygame --test app/tests.rb --no-tick` runs tests headlessly.
- Test file location: place tests in `mygame/app/tests.rb` or any `.rb` file passed to `--test`.
- Test method signature: define methods prefixed with `test_` that accept `args` and `assert` parameters:
  ```ruby
  def test_example args, assert
    # setup
    game = MyGame.new
    game.args = args
    game.tick
    # assertions
    assert.true! condition, "failure message"
    assert.false! condition, "failure message"
    assert.equal! expected, actual, "optional message"
    assert.not_equal! expected, actual
    assert.ok!  # mark test passed without specific assertion
  end
  ```
- Assertion methods: `assert.true!`, `assert.false!`, `assert.equal!`, `assert.not_equal!`, `assert.ok!`.
- Starting tests programmatically: call `$gtk.reset 100` then `$gtk.tests.start` at end of test file.
- Benchmarking: use `GTK.benchmark` to compare code performance:
  ```ruby
  GTK.benchmark iterations: 1000,
                label_one: -> { code_block_1 },
                label_two: -> { code_block_2 }
  # OR by time
  GTK.benchmark seconds: 5,
                label_one: -> { code_block_1 },
                label_two: -> { code_block_2 }
  ```
- Test isolation: reset global state (`$game = nil`) at start of each test to avoid cross-test contamination.

## State Pattern (Game Programming Patterns)

This project uses the State pattern from https://gameprogrammingpatterns.com/state.html for player behavior. Key concepts:

- **Base state class**: `PlayerState` in `mygame/app/lib/player_state.rb` defines the interface with `enter`, `exit`, `handle_input(mouse)`, and `update` methods.
- **Concrete states**: `IdlePlayerState`, `ChargingPlayerState`, `BurstingPlayerState` each encapsulate their own behavior and data.
- **State transitions**: States return a new state object (or `nil` to stay). The owner calls `change_state(new_state)` which invokes `exit` on old state and `enter` on new state.
- **State-specific data**: Keep data relevant only to a state inside that state class (e.g., `charge_timer` in `ChargingPlayerState`, `burst_cooldown` in `BurstingPlayerState`).
- **Owner delegation**: The `Player` entity delegates `handle_input` and `update_state` to its current state, keeping the entity class simple.

Example state transition flow:
```ruby
# In state class - return new state or nil
def handle_input(mouse)
  return nil if mouse.key_held.left  # stay in this state
  IdlePlayerState.new(player)        # transition to idle
end

# In entity class - handle the transition
def change_state(new_state)
  @state.exit
  @state = new_state
  @state.enter
end
```

Benefits:
- Each state is self-contained and testable
- No boolean flag soup (`is_charging && !is_bursting && ...`)
- Adding new states doesn't require modifying existing ones
- Entry/exit actions run automatically on transitions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adamico)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/adamico)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
