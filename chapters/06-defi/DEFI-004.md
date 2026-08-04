---
knowledge_id: DEFI-004
title: Lending
subtitle: Supplying Capital to Shared Liquidity Markets and Earning Variable Yield from Borrow Demand
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 100 min
estimated_study: 240 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Credit
  - Ethereum
parent:
  - DeFi
prerequisites:
  - TOKEN-STANDARDS-001
  - SMART-CONTRACTS-001
related_topics:
  - Borrowing
  - Liquidation
  - Yield Farming
primary_sources:
  - REF-AAVE-SUPPLY-2026-001
  - REF-AAVE-LIQ-2026-001
tags:
  - lending
  - aave
  - defi-credit
---

# Lending
> DeFi  
> Research Unit: DEFI-004

---

## Research Brief

```yaml
knowledge_id: DEFI-004
title: Lending
research_question: >
  How does DeFi lending work at the pool level, how do suppliers earn yield, and
  what determines the risk-return profile of deposited capital?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-001
  - SMART-CONTRACTS-001
parent: DeFi
previous: DEFI-003
next: DEFI-005
```

## 1. Learning Objectives

- Define DeFi lending precisely.
- Explain supply-side yield generation.
- Explain utilization-linked rates.
- Identify lender risks in pooled markets.

## 2. Executive Summary

Aave's help documentation states that supplying tokens to Aave allows users to earn interest and optionally use supplied tokens as collateral.[^ref-aave-supply]

The supplied assets enter a shared liquidity pool that facilitates overcollateralized borrowing.[^ref-aave-supply]

Supply yield depends on:

- borrow demand,
- market utilization,
- governance parameters,
- and protocol-level risk controls.

## 3. Lending Structure

### 3.1 Shared Pool

Supplied assets are made available to borrowers through pool smart contracts.[^ref-aave-supply]

### 3.2 Rate Formation

Aave states that supply rates are determined by the borrow utilization rate and governance parameters.[^ref-aave-supply]

This means lender yield is endogenous to protocol activity, not fixed by an external bank-like policy desk.

## 4. Economic Model

Let:

- `S` = total supplied liquidity
- `B` = total borrowed liquidity
- utilization `U = B / S`

Higher utilization generally supports higher borrowing rates, which can feed higher supply yields, subject to reserve factors and protocol design.

## 5. Risks

- Smart-contract failure.
- Borrower collateral failure if liquidation is ineffective.
- Liquidity risk during stress if utilization is very high.
- Governance and oracle risk.

## 6. Institutional Thinking

- DeFi lending is best understood as pooled credit intermediation without centralized balance-sheet managers.
- Supplier yield is not free carry; it is compensation for deploying capital into a liquidation-dependent system.

## 7. References

[^ref-aave-supply]: Aave Help, "Supply Tokens," official documentation, accessed 2026-08-04, https://aave.com/help/supplying/supply-tokens

## 8. Cross References

- Previous: DEFI-003 — DEX
- Next: DEFI-005 — Borrowing

## Review Status

### Technical Review
Passed.

### Evidence Review
Passed.

### Editorial Review
Passed.

### Adversarial Review
Passed.

### Quality Gate

| Gate | Status |
|---|---|
| Research scope was followed | Pass |
| Required primary sources were reviewed | Pass |
| Material claims are cited | Pass |
| Fact and interpretation are separated | Pass |
| No unresolved critical review issue remains | Pass |
