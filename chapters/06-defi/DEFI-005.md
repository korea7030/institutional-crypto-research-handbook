---
knowledge_id: DEFI-005
title: Borrowing
subtitle: Overcollateralized Onchain Credit, Health Factor Management, and Programmable Leverage
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
  - DEFI-004
related_topics:
  - Liquidation
  - Staking
  - DeFi Risks
primary_sources:
  - REF-AAVE-LIQ-2026-001
  - REF-AAVE-EMODE-2026-001
  - REF-AAVE-ISOLATION-2026-001
tags:
  - borrowing
  - leverage
  - aave
---

# Borrowing
> DeFi  
> Research Unit: DEFI-005

---

## Research Brief

```yaml
knowledge_id: DEFI-005
title: Borrowing
research_question: >
  How does overcollateralized borrowing work in DeFi pools, what constrains
  borrow capacity, and how should analysts think about health factor as a dynamic
  solvency buffer rather than a static ratio?
document_type: foundation
difficulty: L300
prerequisites:
  - DEFI-004
parent: DeFi
previous: DEFI-004
next: DEFI-006
```

## 1. Learning Objectives

- Define overcollateralized borrowing.
- Explain health factor.
- Explain E-mode and isolation mode as risk-specific borrowing configurations.
- Identify how leverage emerges from borrow loops.

## 2. Executive Summary

Aave explains that borrowing power depends on collateral value, liquidation thresholds, and protocol parameters, summarized in the health factor metric.[^ref-aave-liq][^ref-aave-emode]

Borrowing in major DeFi markets is usually overcollateralized. The borrower posts collateral worth more than the borrowed amount, and the protocol monitors position safety continuously.

## 3. Core Mechanics

### 3.1 Borrow Capacity

Borrow capacity depends on collateral value and risk parameters set by governance.

### 3.2 Health Factor

Aave defines health factor as:

`(Total Collateral Value * Weighted Average Liquidation Threshold) / Total Borrow Value`[^ref-aave-liq]

Below `1`, the position risks liquidation.[^ref-aave-liq]

### 3.3 Specialized Borrow Regimes

Aave's E-mode allows higher efficiency for optimized asset categories with specific LTV, liquidation-threshold, and liquidation-bonus settings.[^ref-aave-emode]

Isolation mode restricts how certain volatile assets can be used as collateral, limiting borrowing to approved stablecoins.[^ref-aave-isolation]

## 4. Economic Implications

Borrowing enables:

- leverage,
- liquidity without selling spot exposure,
- funding of arbitrage or carry trades,
- and recursive collateral loops.

## 5. Risks

- collateral price decline,
- debt asset appreciation,
- correlation breakdown,
- parameter changes,
- liquidation competition.

## 6. Institutional Thinking

- Health factor is not a static comfort metric; it is a moving buffer against market volatility.
- Borrow demand and leverage are often strongest precisely where liquidation dependence is highest.

## 7. References

[^ref-aave-liq]: Aave Docs, "Liquidations," official documentation, accessed 2026-08-04, https://aave.com/docs/developers/liquidations

[^ref-aave-emode]: Aave Help, "Efficiency Mode (E-mode)," official documentation, accessed 2026-08-04, https://aave.com/help/borrowing/e-mode

[^ref-aave-isolation]: Aave Help, "Isolation Mode," official documentation, accessed 2026-08-04, https://aave.com/help/supplying/isolation-mode

## 8. Cross References

- Previous: DEFI-004 — Lending
- Next: DEFI-006 — Liquidation

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
