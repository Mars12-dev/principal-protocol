
# Security Controls and Emergency Procedures

This document consolidates actionable security controls recommended by the audit and required for v1 deployment.

## Oracle security
- Require multi-source oracle or issuer-provided signed feed.
- Enforce `max_stale_seconds` (e.g., 600s) for reference values.
- Require `min_sources` and quorum aggregation; if not met, trigger `pause()`.
- Record `value`, `timestamp`, and `source_id` onchain for post-mortem.

## Permissioning and compliance
- Implement a `Permissioning` contract that exposes `is_allowed(address)` and `is_allowed_for_asset(address, asset)`.
- All minting, transfers, and redemptions must call `Permissioning`.
- Permissioning updates require multisig or governance delay.

## Emergency controls
- `pause()`: blocks issuance, trading, and redemption until manual unpause.
- `circuit_breaker()`: automated pause when oracle or pool invariants fail.
- `freeze()`: emergency freeze for legal or compromise events; requires guardian quorum to unfreeze.

## Governance and upgrades
- Core logic immutable by default; allow only parameter updates via timelocked governance.
- Contract replacements (if allowed) must be gated by a delay (e.g., 7 days) and multisig approval.

## Settlement accounting
- Use USDC-denominated reference values for PT principal accounting.
- Define deterministic rounding: prefer floor division for PT minting and distribute residual rounding remainders to a `settlement_reserve` managed by treasury.

## Testing and audits
- Unit tests for arithmetic edge cases (rounding, underflow/overflow).
- Integration tests for oracle failure scenarios and permissioning violations.
- Third-party security audit before mainnet launch.
