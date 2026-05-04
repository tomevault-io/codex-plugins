# Communication

## Details blocks

When you see this formatting in my messages, I'm referring to a section that can be expanded to see more information:

```
> summary
| content
| content
```

The actual MD output would look something like this:

```markdown
  <details>
    <summary>summary</summary>
    <p>content<br>content</p>
  </details>
```

## ellipses

When you see a line ending in `...` in my messages, it often indicates that I have omitted some content for brevity because the omitted content is not essential to the main point being discussed.

# Code fencing

When embedding content in <details> blocks, double-check for proper code fencing. Especially ensure the fence closes properly with the same number of backticks as it opened with.

# Validating Changes

After any set of fixes to the conversion scripts, run the following and check the output:

```bash
python3 chat_to_markdown.py samples/chat.json samples/chat.md
```

It is OK to overwrite the existing output files while testing. Don't create new output files unless absolutely necessary. If you do create new output files, clean up after yourself.

---
> Source: [peckjon/copilot-chat-to-markdown](https://github.com/peckjon/copilot-chat-to-markdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-04 -->
