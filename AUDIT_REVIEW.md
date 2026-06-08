# Principal Protocol Audit Review

## 1. Summary

This audit review evaluates the Principal whitepaper and its implied architecture from a senior DeFi expert perspective on Stellar, with additional review of the design through the lens of a Pendle-style yield split architecture. The review identifies missing details, technical risks, and recommended fixes for a robust, general-purpose Stellar yield tokenization protocol.

## 2. Key Findings

### 2.1 Strong concept, weak implementation detail

The whitepaper describes a compelling protocol model, but it lacks concrete Soroban contract design and onchain state definitions. A full specification must provide explicit contract entrypoints, storage layout, and asset flow invariants.

### 2.2 Oracle and settlement risk

The paper depends on a USDY/USDC reference-value oracle, but it does not define:

- oracle trust model;
- fallback or redundancy;
- stale-price detection;
- how oracle failures affect maturity settlement.

**Recommendation:** add a dedicated oracle security section with freshness thresholds, multi-source validation, and emergency pause triggers.

### 2.3 Permissioning and compliance gaps

The whitepaper states that permissioning should apply across the full lifecycle, but it does not specify how permissioning is enforced on PT, YT, and SY token contracts.

**Recommendation:** define a shared compliance registry or eligibility check interface used by all token contracts. Ensure the asset wrapper and derivative instruments share the underlying USDY permission proof.

### 2.4 PT/YT market architecture ambiguity

The protocol proposes a PT/SY liquidity pool with indirect YT trading, but it does not explain how YT pricing is derived or how router paths are executed.

**Recommendation:** explicitly document the PT-SY-YT routing logic, price discovery mechanism, and whether YT is minted/redeemed at the pool or via an off-pool swap.

### 2.5 Generalized yield tokenization model

The design should be framed as a general yield-tokenization architecture rather than a narrow product for one asset class. Permissioned RWA assets are the first strategic case, but the protocol should support any onchain yield-bearing token that is suitable for split principal and yield exposure.

This is fundamentally a Pendle-style split architecture: one instrument captures discounted principal while the other captures claim to future yield. The documentation should be careful to describe the protocol in these abstract terms rather than as a narrow compliance or RWA-specific service.

**Recommendation:** make the documentation clearly describe the protocol as a generic yield tokenization protocol with permissioned RWA assets as an initial example.

### 2.5 Settlement accounting and rounding

The whitepaper omits the precise math for converting USDC accounting values to underlying USDY at maturity.

**Recommendation:** add exact formulas, decimal handling, and rounding rules for:

- PT redemption amount;
- YT yield amount;
- residual rounding distribution.

### 2.6 Governance and upgrade model

The proposed protocol does not describe governance or contract upgrade paths. Since permissioned assets and permissioning may evolve, this is a missing risk factor.

**Recommendation:** define a conservative upgrade model with delayed governance or multisig upgrade authority. For v1, prefer immutable core contracts with only parameter updates allowed.

## 3. Security Issues and Fixes

### 3.1 Missing emergency controls

The document mentions emergency pause logic, but it does not make it a protocol requirement.

**Fix:** require a `pause()` action on critical contracts, tracked by an onchain `RiskControl` module. Paused state should block issuance, trading, redemption, and oracle updates.

### 3.2 Unclear replay and expiration model for YT

The whitepaper does not cover whether YT positions can be transferred or redeemed before maturity if the underlying reference value changes.

**Fix:** specify that YT is transferable subject to permissioning and define the pre-maturity valuation model, including if YT carrying value is based on implied yield rather than on immediate underlying reference price.

### 3.3 Protocol fee model needs precision

Fees are described in broad terms, but not as onchain parameters.

**Fix:** define fee rates, fee recipients, and whether fees are taken in yield or swap amounts. Make fees configurable with governance and cap them to a safe maximum.

### 3.4 Lack of source-of-truth for maturity date

The whitepaper refers to maturity, but it does not define whether each PT/YT issuance has a single maturity or multiple maturities.

**Fix:** require each issuance to carry a clear `maturity_date` and `reference_value_timestamp`. Support separately tracked maturity epochs if multiple tranches are offered.

## 4. Technical Documentation Improvements

### 4.1 Add contract-level diagrams

The protocol should include:

- a contract interaction diagram;
- a token lifecycle diagram;
- a flow diagram for deposit → mint → trade → redeem.

### 4.2 Clarify asset and instrument names

The whitepaper uses `SY-USDY`, `PT-USDY`, and `YT-USDY`, but it should clearly state whether these are new Stellar asset contracts or Soroban tokens. This is critical for wallet integration.

### 4.3 Expand the risk section

The existing document should include a risk chapter covering:

- issuer counterparty risk;
- oracle failure risk;
- permissioning failure risk;
- liquidity risk for PT and YT markets.

### 4.4 Include a compliance / permissioning appendix

Since the protocol is explicitly permissioned, a dedicated appendix should explain how Stellar permissioned asset rules are inherited by derivative instruments, including any KYC/AML assumptions.

## 5. Recommended Audit Checklist

- Verify permissioned eligibility is enforced at mint, transfer, redemption.
- Verify oracle sources, freshness, and emergency pause.
- Verify maturity settlement formula and rounding.
- Verify PT/YT burn and recombination invariants.
- Verify fee parameter caps and fee accrual accounting.
- Verify contract access controls and upgrade constraints.
- Verify risk control state is globally respected by all protocol flows.

## 6. Conclusion

Principal Protocol is a strong strategic fit for Stellar yield-bearing assets, but the current technical documentation must be expanded before development begins. The focus should be on explicit contract interfaces, oracle security, permissioning enforcement, maturity accounting, and emergency governance.

## 7. Fixes Implemented (this repo)

I (senior auditor) applied the following fixes to the repository documentation:

- Added `ARCHITECTURE.md` with contract interaction diagrams and maturity flow.
- Added `SECURITY.md` with concrete oracle, permissioning, emergency controls and governance recommendations.
- Expanded `TECHNICAL_SPECIFICATION.md` with:
	- Oracle trust model and minimum requirements.
	- Deterministic settlement formula and rounding policy.
	- Explicit maturity fields and issuance identity rules.
	- Permissioning interface sketch and enforcement guidance.
	- Fee parameter definitions and governance caps.

## 8. Remaining action items for code implementation

- Implement onchain `Permissioning` contract and integrate checks into all token and wrapper contracts.
- Implement `OracleAdapter` with multi-source aggregation and onchain storage of payloads.
- Add `RiskControl` contract and wire `pause()`/`circuit_breaker()` into all critical flows.
- Write exhaustive unit and integration tests for rounding edge-cases and concurrency scenarios.

