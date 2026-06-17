
# secret — agent-safe credential handling

Backed by `secret` (macOS Keychain). The whole point: the human types the value
into a GUI box, it lives only in the Keychain, and **the plaintext never enters
the chat transcript or a plaintext file** — provided you follow the ironclad
rules below. This tool keeps secrets out of the *conversation*; it does NOT make
them un-leakable at the OS level (`ps`, env vars, a wrapped command that echoes
its own args, etc.). Your discipline is the security boundary — the `get`
command will happily print plaintext if you misuse it.

## When you need a credential that isn't stored yet

Do NOT ask the user to paste it in chat (that leaks it into history). Instead
**you** trigger the GUI prompt for them:

```
secret set <name>
```

This pops a native hidden-answer dialog. The user types the value and clicks
Save; it goes straight into the Keychain. Your Bash output only shows
`stored '<name>' ...` — you never see the value. Then tell the user it's stored.

## When you need to USE a stored secret

Inject it, never print it:

```
SOME_ENV="$(secret get <name>)" the-command-that-needs-it
```

- Pipe through `grep -v` of the secret's likely prefix if the wrapped command
  might echo it (e.g. `| grep -v 'postgresql://'`).
- The first `get` may pop a Keychain authorization dialog — click **Always Allow**.

## Ironclad rules

- NEVER `Read`, `cat`, `echo`, or otherwise surface the plaintext value.
- NEVER ask the user to paste a secret into the chat — trigger `secret set` instead.
- `secret list` shows only names (safe to run); use it to see what's available.

## Commands

| Command | What it does |
|---|---|
| `secret set <name>` | GUI prompt → store in Keychain (you trigger, user types) |
| `secret get <name>` | print value to stdout — only for `$(...)` injection |
| `secret list` | list stored names only (never values) |
| `secret rm <name>` | delete a secret |

---
> Source: [Fino-wind/agent-secret](https://github.com/Fino-wind/agent-secret) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-17 -->
