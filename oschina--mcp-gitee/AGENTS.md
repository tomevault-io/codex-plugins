# Claude Code

Add the remote MCP server to the `mcpServers` section:

```json
{
  "mcpServers": {
    "gitee": {
      "type": "http",
      "url": "https://api.gitee.com/mcp",
      "headers": {
        "Authorization": "Bearer <your personal access token>"
      }
    }
  }
}
```

---
> Source: [oschina/mcp-gitee](https://github.com/oschina/mcp-gitee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-29 -->
