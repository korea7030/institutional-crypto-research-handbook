---
knowledge_id: DEFI-006
title: Liquidation
subtitle: Permissionless Position Resolution, Collateral Seizure, and the Safety Valve of Overcollateralized DeFi Credit
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 105 min
estimated_study: 250 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Risk
  - Ethereum
parent:
  - DeFi
prerequisites:
  - DEFI-005
related_topics:
  - MEV
  - DeFi Risks
  - Lending
primary_sources:
  - REF-AAVE-LIQ-2026-001
  - REF-AAVE-HFHELP-2026-001
tags:
  - liquidation
  - risk
  - aave
---

# Liquidation
> DeFi  
> Research Unit: DEFI-006

---

## Research Brief

```yaml
knowledge_id: DEFI-006
title: Liquidation
research_question: >
  How do DeFi liquidations preserve solvency in overcollateralized lending
  markets, what economic incentives make them work, and why are they tightly
  connected to gas markets and MEV competition?
document_type: foundation
difficulty: L300
prerequisites:
  - DEFI-005
parent: DeFi
previous: DEFI-005
next: DEFI-007
```

## 1. Learning Objectives

- Define liquidation in DeFi terms.
- Explain close factors and liquidation bonuses.
- Explain why liquidations are permissionless and competitive.
- Connect liquidation execution to MEV dynamics.

## 2. Executive Summary

Aave documents liquidation as the process that occurs when a borrower's health factor drops below 1, allowing anyone to repay part of the debt and receive discounted collateral in return.[^ref-aave-liq]

Liquidation is not an edge case. It is the enforcement layer that makes overcollateralized lending work.

## 3. Core Mechanics

### 3.1 Trigger Condition

Aave states that a health factor below `1` risks liquidation.[^ref-aave-liq][^ref-aave-hf]

### 3.2 Permissionless Execution

Liquidations are permissionless, meaning any participant can initiate the process on an eligible position.[^ref-aave-liq]

### 3.3 Incentive

The liquidator repays debt and receives collateral plus a bonus, giving external actors economic reason to keep the system solvent.[^ref-aave-liq][^ref-aave-hf]

## 4. Key Parameters

- health factor,
- liquidation threshold,
- close factor,
- liquidation bonus,
- asset prices from protocol oracles.

## 5. Economic Model

If debt repaid is `D` and liquidation bonus is `b`, collateral seized is approximately:

`Collateral value transferred ≈ D * (1 + b)`

subject to oracle price conversions and protocol rules.

## 6. Competition and MEV

Aave explicitly notes that liquidations are highly competitive and often require automated bots.[^ref-aave-liq][^ref-aave-hf]

This ties liquidations to:

- transaction-priority bidding,
- searcher infrastructure,
- and MEV extraction.

## 7. Institutional Thinking

- Liquidation efficiency is a core solvency variable for DeFi credit markets.
- A market can look well-collateralized until liquidation throughput fails under volatility.

## 8. References

[^ref-aave-liq]: Aave Docs, "Liquidations," official documentation, accessed 2026-08-04, https://aave.com/docs/developers/liquidations

[^ref-aave-hf]: Aave Help, "Health Factor & Liquidations," official documentation, accessed 2026-08-04, https://aave.com/help/borrowing/liquidations

## 9. Cross References

- Previous: DEFI-005 — Borrowing
- Next: DEFI-007 — Yield Farming

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
