
When the conversation is about to end and task is a success, ask if linters should be run. If yes
run the following linters and fix the linting issues

```bash
    black --check --line-length=88 sparse_attention_hub tests scripts
```

```bash
        isort \
            --profile=black \
            --line-length=88 \
            --multi-line=3 \
            -p sparse_attention_hub \
            sparse_attention_hub tests scripts
```

```bash
    flake8 \
        --max-line-length=88 \
        --extend-ignore=E203,W503,E501 \
        --exclude=build,dist,.git,__pycache__,.pytest_cache,.venv \
        sparse_attention_hub tests scripts
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sahiljoshi515)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/sahiljoshi515)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
