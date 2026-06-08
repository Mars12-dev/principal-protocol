# Principal Protocol

A Soroban-native yield tokenization protocol for Stellar. Principal Protocol splits permissioned yield-bearing assets (e.g. USDY) into fixed-principal tokens (PT) and future-yield tokens (YT), enabling fixed-income exposure and yield trading on Stellar.

## Repository layout

```
contracts/
  oracle_adapter/      — USDY/USDC reference-value oracle with freshness checks
  permissioning/       — eligibility registry for permissioned assets
  sy_wrapper/          — standardized yield wrapper (SY-USDY)
  principal_manager/   — tokenization engine: mints/burns PT and YT, settles at maturity
  risk_control/        — pause flag and circuit-breaker controls
Cargo.toml             — workspace manifest
TECHNICAL_SPECIFICATION.md
ARCHITECTURE.md
SECURITY.md
AUDIT_REVIEW.md
CONTRIBUTING.md
DEPLOYMENT.md
```

## Quick start

```bash
# Build all contracts
cargo build --target wasm32-unknown-unknown --release

# Run all unit tests
cargo test
```

Requires the Rust toolchain with the `wasm32-unknown-unknown` target and [Stellar CLI](https://developers.stellar.org/docs/tools/developer-tools/cli/stellar-cli).

## Documentation

| Document | Purpose |
|---|---|
| [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md) | Full protocol spec, contract interfaces, settlement math |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Contract interaction diagrams and maturity flows |
| [SECURITY.md](SECURITY.md) | Oracle safety, permissioning controls, emergency procedures |
| [AUDIT_REVIEW.md](AUDIT_REVIEW.md) | Senior-auditor checklist and open issues |
| [DEPLOYMENT.md](DEPLOYMENT.md) | Testnet and mainnet deployment guide |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Development workflow and contribution guidelines |

## Protocol overview

```
User deposits USDY → SYWrapper mints SY-USDY
PrincipalManager splits SY-USDY → PT (fixed principal) + YT (future yield)
At maturity: OracleAdapter provides settlement rate → PT and YT redeemed
```

See [ARCHITECTURE.md](ARCHITECTURE.md) for the full interaction diagram.

## License

Apache 2.0
