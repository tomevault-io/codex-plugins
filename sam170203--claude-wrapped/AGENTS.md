
Run the following command exactly, then respond with only this message — nothing else:

**Claude Code Wrapped is opening in a new window** — use `SPACE` / `ENTER` to advance slides, `Q` to quit.

If the command fails, respond: "Wrapped failed to launch — make sure Python 3 is installed and try again."

```bash
python3 -c "
import subprocess, os, sys
script = '${CLAUDE_SKILL_DIR}/wrapped.py'
if not os.path.exists(script):
    print('Error: wrapped.py not found'); sys.exit(1)
kw = {'creationflags': subprocess.CREATE_NEW_CONSOLE} if os.name == 'nt' else {'start_new_session': True}
subprocess.Popen([sys.executable, script], **kw)
"
```

---
> Source: [sam170203/claude-wrapped](https://github.com/sam170203/claude-wrapped) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-16 -->
