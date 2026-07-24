# Shiori Test Commands

# Run the entire test suite
make unittest

# Run SQLite database tests only
go test -timeout 10s -count=1 -tags test_sqlite_only ./internal/database

---
> Source: [uparrows/shiori_cn](https://github.com/uparrows/shiori_cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
