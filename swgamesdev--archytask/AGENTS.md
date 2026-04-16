# Agent Context & Rules

- Always respond in Japanese.
- Create all documents (artifacts like walkthrough.md, implementation_plan.md, task.md) in Japanese.

## Notification upon Task Completion

When your work is complete, always use terminal-notifier to send a notification to macOS.

```bash
terminal-notifier -title "GitHub Copilot" -message "Task completed" -sound default
```

### When to Notify
- When you finish writing the changes
- When file editing is complete
- When error correction is complete
- When feature implementation is complete
- When the user's request is fully resolved

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/swgamesdev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-13 -->
