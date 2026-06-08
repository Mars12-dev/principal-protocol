# Principal Protocol

A Soroban-native yield tokenization protocol for Stellar. Principal Protocol splits permissioned yield-bearing assets (e.g. USDY) into fixed-principal tokens (PT) and future-yield tokens (YT), enabling fixed-income exposure and yield trading on Stellar.

## How it works

```
User deposits USDY
       │
       ▼
  SYWrapper  ──── mints SY shares (exchange rate grows as yield accrues)
       │
       ▼
PrincipalManager ─── splits SY shares into:
       │                PT (Principal Token) — fixed redemption at maturity
       │                YT (Yield Token)     — captures yield above principal
       │
       ▼
  At maturity: OracleAdapter provides final rate → PT and YT redeemed for USDY
```

PT holders receive a fixed, predictable return. YT holders capture the variable yield above that fixed rate. Both instruments can be traded before maturity.

## Repository layout

```
contracts/
  oracle_adapter/      — USDY/USDC reference-value oracle with freshness and admin controls
  permissioning/       — account and per-asset eligibility registry
  sy_wrapper/          — standardized yield wrapper (SY-USDY)
  principal_manager/   — tokenization engine: mints/burns PT and YT, settles at maturity
  risk_control/        — global pause flag, pauser roles, and rolling circuit breaker
Cargo.toml             — workspace manifest (Soroban SDK 26.x)
TECHNICAL_SPECIFICATION.md
ARCHITECTURE.md
SECURITY.md
AUDIT_REVIEW.md
CONTRIBUTING.md
DEPLOYMENT.md
```

## Quick start

**Requirements:** Rust stable (≥ 1.79), `wasm32-unknown-unknown` target, [Stellar CLI](https://developers.stellar.org/docs/tools/developer-tools/cli/stellar-cli) ≥ 22.0.

```bash
# Add WASM target (once)
rustup target add wasm32-unknown-unknown

# Run all unit tests
cargo test

# Build all WASM artifacts
cargo build --target wasm32-unknown-unknown --release
```

WASM artifacts land in `target/wasm32-unknown-unknown/release/`.

## Contract overview

| Contract | Crate | Purpose |
|---|---|---|
| OracleAdapter | `principal_oracle_adapter` | Stores and validates USDY/USDC reference value |
| Permissioning | `principal_permissioning` | Account and asset eligibility registry |
| SYWrapper | `principal_sy_wrapper` | Holds underlying asset, issues SY shares at rolling exchange rate |
| PrincipalManager | `principal_manager` | Mints PT/YT from SY shares; redeems at maturity |
| RiskControl | `principal_risk_control` | Protocol-level pause, pauser roles, deposit circuit breaker |

## Key design properties

- **Asset-agnostic** — SYWrapper and PrincipalManager work with any Stellar yield-bearing asset, not only USDY.
- **Permissioned by default** — Permissioning contract enforces eligibility at every mint, transfer, and redemption.
- **Deterministic settlement** — settlement math uses fixed-point arithmetic with floor rounding; no floating-point.
- **Defense in depth** — three independent safety layers: oracle freshness checks, permissioning enforcement, and RiskControl circuit breaker.
- **Modern Soroban API** — all contracts use `require_auth()`, typed storage keys (`#[contracttype]`), typed errors (`#[contracterror]`), and `instance`/`persistent` storage tiers.

## Documentation

| Document | Purpose |
|---|---|
| [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md) | Full protocol spec, contract interfaces, settlement math, storage layout |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Contract interaction diagrams, sequence flows, storage tiers |
| [SECURITY.md](SECURITY.md) | Threat model, per-contract security properties, incident response |
| [AUDIT_REVIEW.md](AUDIT_REVIEW.md) | Senior-auditor findings, status tracking, open items |
| [DEPLOYMENT.md](DEPLOYMENT.md) | Step-by-step testnet and mainnet deployment with Stellar CLI |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Development workflow, code style, PR checklist |

## License

Apache 2.0
