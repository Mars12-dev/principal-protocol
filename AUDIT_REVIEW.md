# Principal Protocol Audit Review

## 1. Summary

This document evaluates the Principal Protocol architecture from a senior DeFi auditor perspective, with specific focus on Soroban contract design, Pendle-style yield split mechanics, and Stellar-specific risks. Each finding includes a status reflecting whether it has been addressed in the current codebase.

## 2. Findings

### 2.1 Strong concept, incomplete implementation detail
**Severity:** Medium  
**Status:** Partially addressed

The whitepaper describes a compelling protocol model but lacked concrete Soroban contract interfaces, storage layout, and asset flow invariants.

**Resolution:** `TECHNICAL_SPECIFICATION.md` now includes explicit contract entrypoints, storage key tables, and settlement math. `ARCHITECTURE.md` adds interaction diagrams and sequence flows. Prototype Soroban contracts are implemented for all five core modules.

**Remaining:** PT and YT are tracked internally in `PrincipalManager` rather than as separate SEP-41 token contracts. Production deployment requires dedicated token contracts for tradeable instruments.

---

### 2.2 Oracle and settlement risk
**Severity:** High  
**Status:** Addressed in specification; prototype integration pending

The whitepaper did not define an oracle trust model, freshness thresholds, fallback behaviour, or how oracle failures affect maturity settlement.

**Resolution:**
- `OracleAdapter` contract enforces monotonically increasing timestamps and rejects stale values.
- `is_fresh(max_stale_seconds)` uses `env.ledger().timestamp()` — not caller-supplied — preventing manipulation.
- `PrincipalManager` defines `MAX_ORACLE_STALENESS_SECS = 3600` and calls `assert_oracle_fresh` before every redemption.
- `SECURITY.md` documents the oracle failure response procedure.

**Remaining:** Production deployment requires a multi-source or issuer-signed feed. The current single-admin oracle is suitable for testnet only.

---

### 2.3 Permissioning and compliance gaps
**Severity:** High  
**Status:** Addressed

The whitepaper stated that permissioning should apply across the full lifecycle but did not specify how it is enforced on PT, YT, and SY token contracts.

**Resolution:**
- `Permissioning` contract implements `is_allowed(account)` and `is_allowed_for_asset(account, asset)`.
- `PrincipalManager.mint` calls `assert_permitted` before minting PT or YT.
- Eligibility entries use `persistent()` storage with `ELIGIBILITY_TTL_LEDGERS ≈ 30 days` TTL, providing automatic expiry for inactive participants.
- Batch `grant_accounts` reduces operational overhead for issuers onboarding multiple participants.

**Remaining:** Full integration of permissioning checks into SYWrapper deposit/withdraw flows.

---

### 2.4 PT/YT market architecture ambiguity
**Severity:** Medium  
**Status:** Open

The whitepaper proposed a PT/SY liquidity pool with indirect YT trading but did not explain pricing derivation or router execution paths.

**Remaining:** `MarketPool` and `Router` contracts are not yet implemented. This is the primary gap between the current prototype and a production protocol. These should be the next development milestone.

---

### 2.5 Settlement accounting and rounding
**Severity:** High  
**Status:** Addressed

The whitepaper omitted precise math for converting accounting values to underlying tokens at maturity.

**Resolution:**
- `TECHNICAL_SPECIFICATION.md` section 14 defines the deterministic settlement formula with floor division and `settlement_reserve` for rounding residuals.
- `PrincipalManager.redeem` implements: `from_yt = max(0, (final_rate - SCALE) * yt_amount / SCALE)`.
- PT redeems 1:1 with principal in underlying units; YT captures only positive yield above par.
- `overflow-checks = true` in the release profile; all arithmetic uses `i128`.

---

### 2.6 Governance and upgrade model
**Severity:** Medium  
**Status:** Addressed in specification

The whitepaper did not describe governance or contract upgrade paths.

**Resolution:** `SECURITY.md` section 7 defines the v1 policy: immutable core logic, parameter-only updates via timelocked governance, and recommended multisig setup. No upgrade entrypoint exists in v1 contracts — this is intentional.

---

### 2.7 Missing emergency controls
**Severity:** High  
**Status:** Addressed

Emergency pause logic was mentioned but not defined as a protocol requirement.

**Resolution:**
- `RiskControl` contract implements: `pause()` (callable by admin or any registered pauser), `unpause()` (admin only), and a rolling 24-hour deposit circuit breaker.
- Pause/unpause distinction between pausers and admin prevents a compromised pauser from cycling the protocol.
- `SECURITY.md` documents the full emergency response procedure.

---

### 2.8 Replay and expiration model for YT
**Severity:** Medium  
**Status:** Partially addressed

The whitepaper did not specify whether YT positions can be transferred or redeemed before maturity, or how pre-maturity valuation works.

**Resolution:** `PrincipalManager` enforces maturity check via `env.ledger().timestamp()`. Minting is blocked post-maturity; redemption is blocked pre-maturity.

**Remaining:** Pre-maturity YT transfer rules and implied yield pricing are not yet specified. This requires the `MarketPool` and `Router` implementation.

---

### 2.9 Fee model precision
**Severity:** Low  
**Status:** Open

Fees were described broadly without onchain parameters.

**Remaining:** Fee parameters (`fee_yield_bps`, `fee_swap_bps`), fee recipients, and accrual logic are not yet implemented. Define alongside `MarketPool` development.

---

### 2.10 Maturity source of truth
**Severity:** Medium  
**Status:** Addressed

The whitepaper did not define whether each PT/YT issuance has a single maturity or multiple.

**Resolution:** `PrincipalManager.initialize` takes a single `maturity` Unix timestamp. Each deployed `PrincipalManager` instance represents one maturity epoch. Multiple maturities are supported by deploying multiple instances. `TECHNICAL_SPECIFICATION.md` section 15 defines `maturity_timestamp`, `reference_value_timestamp`, and `issue_nonce` fields.

---

## 3. Recommended audit checklist

| Check | Status |
|---|---|
| Permissioned eligibility enforced at mint | Implemented (prototype) |
| Permissioned eligibility enforced at transfer | Open |
| Permissioned eligibility enforced at redemption | Implemented (prototype) |
| Oracle freshness check at settlement | Implemented (prototype) |
| Oracle multi-source validation | Open (production requirement) |
| Emergency pause wires into all flows | Partially (RiskControl exists; full wiring pending) |
| Maturity settlement formula and rounding | Specified; implemented in prototype |
| PT/YT burn and recombination invariants | Specified; recombination not yet implemented |
| Fee parameter caps and accrual accounting | Open |
| Admin access controls on all contracts | Implemented |
| Admin transfer / key rotation | Implemented on all five contracts |
| Integer overflow protection | Implemented (`overflow-checks = true`) |
| Reentrancy protection in SYWrapper | Implemented (checks-effects-interactions) |
| Unit tests for rounding edge cases | Partial |
| Integration tests for oracle failure | Open |

## 4. Open items for next milestone

1. Implement `MarketPool` (PT/SY liquidity pool) and `Router` (multi-step flow coordinator).
2. Separate PT and YT into standalone SEP-41 token contracts.
3. Wire `RiskControl.check_deposit` into SYWrapper and PrincipalManager.
4. Implement full cross-contract calls: PrincipalManager → OracleAdapter, PrincipalManager → Permissioning.
5. Add recombination entrypoint: `PrincipalManager.recombine(pt_amount, yt_amount) → sy_shares`.
6. Define fee parameters and treasury accrual.
7. Expand unit tests to cover all arithmetic edge cases and oracle failure paths.
8. Commission third-party security audit before mainnet.
