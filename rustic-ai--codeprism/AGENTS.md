# GitHub CLI (gh) Command Best Practices

This rule documents the correct way to execute GitHub CLI (`gh`) commands when using the `run_terminal_cmd` tool to avoid common pager-related issues.

## The Problem

The `gh` CLI automatically detects when it's running in what it thinks is an interactive terminal and attempts to use a pager (like `less` or `more`) to display output. This causes issues with the `run_terminal_cmd` tool, resulting in errors like:

```
/usr/bin/head: cannot open '|' for reading: No such file or directory
```

## The Solution

Always pipe `gh` command output to `cat` to prevent the pager from being invoked:

### ✅ Correct Usage

```bash
gh run list --workflow=ci.yml --limit 5 | cat
gh run view 12345 --log | cat
gh run list --workflow=docker.yml --limit 1 --json databaseId --jq '.[0].databaseId' | cat
```

### ❌ Incorrect Usage

```bash
gh run list --workflow=ci.yml --limit 5
gh run view 12345 --log
gh run list --workflow=docker.yml --limit 1 --json databaseId --jq '.[0].databaseId'
```

## Common GitHub CLI Commands with Proper Piping

### Listing Workflow Runs
```bash
gh run list --workflow=<workflow-name> --limit <number> | cat
```

### Viewing Run Logs
```bash
gh run view <run-id> --log | cat
```

### Getting Run Status
```bash
gh run list --workflow=<workflow-name> --limit 1 | cat
```

### JSON Output with jq
```bash
gh run list --workflow=<workflow-name> --limit 1 --json databaseId --jq '.[0].databaseId' | cat
```

### Listing Issues
```bash
gh issue list | cat
```

### Viewing Pull Requests
```bash
gh pr list | cat
gh pr view <pr-number> | cat
```

## Why This Works

- The `| cat` prevents `gh` from detecting an interactive terminal
- This forces `gh` to output raw text instead of trying to use a pager
- The `cat` command simply passes through the output unchanged
- Other commands like `ls`, `git`, and `cargo` don't have this issue because they don't automatically invoke pagers

## When This Rule Applies

This rule specifically applies when:
- Using the `run_terminal_cmd` tool
- Executing any `gh` command that might produce multi-line output
- The command would normally trigger a pager in an interactive terminal

## Alternative Solutions

If piping to `cat` is not suitable for your use case, you can also:
- Set the `PAGER` environment variable to empty: `PAGER= gh run list`
- Use the `--no-pager` flag if available for the specific `gh` command

---
> Source: [rustic-ai/codeprism](https://github.com/rustic-ai/codeprism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-13 -->
