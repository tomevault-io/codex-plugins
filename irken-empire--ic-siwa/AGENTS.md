# IC-SIWA Copilot Instructions

IC-SIWA (Sign-In with Avalanche for Internet Computer) is a Rust canister library and provider that adapts the EIP-4361 sign-in flow for Avalanche wallets on the Internet Computer blockchain.

## Project Layout

```shell
canisters/ic_siwa_provider/   # Main SIWA provider canister (cdylib)
canisters/test_canister_rs/   # Rust integration test canister
canisters/test_canister_ts/   # Astro/TypeScript test app
libs/ic_siwa/                 # Core Rust SIWA library
libs/ic_siwa_ts/              # TypeScript client library
config/                       # YAML config per environment (development/testnet/mainnet)
docs/spec.md                  # Source of truth for SIWA behaviour
```

## Technology Stack

- **Backend**: Rust stable, compiled to `wasm32-unknown-unknown` for IC canisters
- **IC SDK**: `ic-cdk 0.20`, `ic-cdk-macros 0.20`, `ic-cdk-management-canister 0.1`
  - Management canister calls (e.g. `raw_rand`) use `ic_cdk_management_canister::` not `ic_cdk::management_canister::` (moved in 0.20)
- **Cryptography**: `k256` for ECDSA, `sha3` for Keccak256, `ic-stable-structures` for persistence
- **Frontend**: Astro + TypeScript + DaisyUI (test canister only)
- **Dev environment**: All tooling via `devenv` (Nix). Never suggest global installs.
- **Secrets**: `secretspec` + 1Password. Use env fallback: `SECRETSPEC_PROVIDER=env devenv shell --quiet -- <cmd>`

## Build & Test

```bash
ic-siwa build          # Build all canisters
ic-siwa test           # Run all tests
ic-siwa deploy         # Deploy to local dfx replica
cargo check            # Fast type-check (all workspace crates)
cargo clippy           # Lint (warnings are errors)
cargo fmt              # Format
```

## Rust Guidelines

- Use workspace deps from root `Cargo.toml`; do not add duplicate versions in crate-level `Cargo.toml`
- Explicit error handling — no `.unwrap()` in library or canister code; propagate or map errors
- All public API items must have `///` doc comments
- Naming: `snake_case` for everything (Rust default)
- Compile target is `wasm32-unknown-unknown`; avoid platform-specific crates

## IC / Canister Patterns

- **Randomness**: `ic_cdk_management_canister::raw_rand().await` (ic-cdk 0.20+)
- **Caller identity**: `ic_cdk::api::msg_caller()` (not the removed `ic_cdk::caller()`)
- **Time**: `ic_cdk::api::time()` returns nanoseconds
- **Stable storage**: use `ic-stable-structures` (`StableBTreeMap` etc.) for data that survives upgrades
- **Async**: annotate canister entry points with `#[ic_cdk::update]` / `#[ic_cdk::query]`; use `.await` for inter-canister calls
- **Security**: never trust caller-supplied data without validation; use caller principal as the identity key

## SIWA Protocol

- Implements EIP-4361 adapted for Avalanche C-Chain (Ethereum-compatible addresses, `personal_sign` prefix)
- Message fields defined in `docs/spec.md` — always consult before changing the message format
- Addresses are 0x-prefixed, EIP-55 checksummed
- Nonces are 16-byte hex-encoded random values from `raw_rand`

## Code Quality Gates

Before any commit, all of these must pass inside `devenv shell`:

1. `cargo fmt --check`
2. `cargo clippy -- -D warnings`
3. `cargo test`
4. `cargo build --target wasm32-unknown-unknown --release`

## Configuration

- YAML source of truth in `config/{development,testnet,mainnet}.yaml`
- Never hardcode chain IDs, canister IDs, or RPC URLs in source — read from config
- Secrets (identity keys, API keys) go in `secretspec.toml`, never in source or YAML

## Commit Convention

Conventional commits enforced via pre-commit hooks: `type(scope): description`
Types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/irken-empire) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
