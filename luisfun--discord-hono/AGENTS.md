## Naming Conventions
Convert the value to a variable name using the following rules:
- Replace `/` with `$`
- Replace `{VER}` with `_`
### Example
varValue: `/categories/{category}/tags/{tag}`
varName: `$categories$_$tags$_`
code: `const $categories$_$tags$_ = '/categories/{category}/tags/{tag}'`

---
> Source: [luisfun/discord-hono](https://github.com/luisfun/discord-hono) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-27 -->
