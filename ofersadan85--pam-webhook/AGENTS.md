# Copilot instructions for `pam-webhook`

## Build, test, and lint commands

- Build release Linux PAM module:
  - `cargo build --release`
  - See `.justfile`
  - Output artifact: `target/release/libpam_webhook.so`
- Build specific modes:
  - null/no-op: `cargo build --release --no-default-features`
  - logging-only: `cargo build --release --no-default-features --features=logging`
  - webhook-only: `cargo build --release --no-default-features --features=webhook`
  - combined logging+webhook: `cargo build --release --all-features`
- Run complete test matrix (all feature modes):
  - `just test`
- Run all validation checks (format + lint + matrix):
  - `just check`

## High-level architecture

- This crate is a Rust `cdylib` intended to be loaded by Linux-PAM (configured in PAM as `pam_webhook.so`).
- `src/lib.rs` exports all six Linux-PAM service hooks:
  - `pam_sm_authenticate`, `pam_sm_setcred`, `pam_sm_acct_mgmt`, `pam_sm_open_session`, `pam_sm_close_session`, `pam_sm_chauthtok`.
- `src/handlers/mod.rs` defines:
  - `PamEventHandler` trait with default no-op hook methods.
  - `MultiHandler`, which composes enabled handlers and executes each hook in order.
- Feature behavior is additive:
  - no features: no handlers active, hooks return success with no side effects
  - `logging`: `LoggingHandler`
  - `webhook`: `WebhookHandler`
  - `logging + webhook`: both handlers are active via `MultiHandler`
- Integration tests in `tests/ffi_hooks.rs` validate feature-mode side effects from the built `.so`.

## Key conventions in this repo

- Treat this as a scaffold: hooks intentionally return success while webhook/policy logic is added incrementally.
- Keep exported FFI hooks ABI-stable:
  - preserve `#[unsafe(no_mangle)] pub extern "C"` signatures and PAM hook names exactly.
- Reuse shared helpers (for example `get_item` and config parsing) instead of per-hook custom logic when adding behavior.
- Avoid logging secrets; existing diagnostics only include hook name, flags, and selected PAM metadata.
- Assume Linux-only operation; do not add non-Linux stubs or platform fallbacks.
- Prefer mode-aware tests that verify observable behavior for each feature combination, including `--all-features`.

---
> Source: [ofersadan85/pam-webhook](https://github.com/ofersadan85/pam-webhook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-07 -->
