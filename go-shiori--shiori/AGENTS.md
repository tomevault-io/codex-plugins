# Shiori Test Commands

# Run the entire test suite
make unittest

# Run SQLite database tests only
go test -timeout 10s -count=1 -tags test_sqlite_only ./internal/database

---
> Source: [go-shiori/shiori](https://github.com/go-shiori/shiori) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
