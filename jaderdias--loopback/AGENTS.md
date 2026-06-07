# Deploying updates

After building a new release, restart the service with:

```bash
cargo build --release
systemctl --user stop loopback
cp target/release/loopback "$HOME/opt/loopback"
sudo setcap cap_net_raw+ep "$HOME/opt/loopback"
systemctl --user start loopback
```

Verify it started cleanly:

```bash
systemctl --user status loopback --no-pager

---
> Source: [JaderDias/loopback](https://github.com/JaderDias/loopback) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-07 -->
