---
knowledge_id: DEFI-002
title: Liquidity Pools
subtitle: Shared Onchain Inventory, LP Positions, and the Capital Base Behind AMM Trading
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 95 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Markets
  - Ethereum
parent:
  - DeFi
prerequisites:
  - DEFI-001
related_topics:
  - DEX
  - Yield Farming
  - DeFi Risks
primary_sources:
  - REF-UNI-HOW-2026-001
  - REF-UNI-LIQ-2026-001
  - REF-UNI-CONC-2026-001
tags:
  - liquidity-pools
  - lp
  - uniswap
---

# Liquidity Pools
> DeFi  
> Research Unit: DEFI-002

---

## Research Brief

```yaml
knowledge_id: DEFI-002
title: Liquidity Pools
research_question: >
  What is a liquidity pool, how are LP claims represented across protocol
  versions, and what risk-return tradeoffs do providers face when supplying
  inventory to AMMs?
document_type: foundation
difficulty: L300
prerequisites:
  - DEFI-001
parent: DeFi
previous: DEFI-001
next: DEFI-003
related_topics:
  - DEX
  - Yield Farming
  - DeFi Risks
```

## 1. Learning Objectives

- Define a liquidity pool precisely.
- Distinguish LP capital from trading volume.
- Explain LP fee accrual.
- Explain concentrated-liquidity positions.
- Identify impermanent-loss exposure.

## 2. Executive Summary

Uniswap describes a liquidity pool as reserves of two ERC-20 tokens that traders swap against.[^ref-uni-how]

Liquidity providers deposit assets into these pools and earn fees from trade flow. Uniswap notes that LP shares differ by version:

- v2 mints fungible ERC-20 LP tokens,
- v3 represents positions as ERC-721 NFTs,
- v4 uses new internal accounting structures around position management.[^ref-uni-how]

Concentrated liquidity changes LP economics by letting providers allocate capital to specific price ranges rather than the full interval `(0, infinity)`.[^ref-uni-conc]

## 3. Pool Structure

### 3.1 Shared Inventory

The pool is the inventory base for swaps. Traders take one asset out and put the other in, moving price through reserve changes.

### 3.2 LP Claims

Uniswap explains that LP claims differ across protocol versions.[^ref-uni-how]

This matters analytically because LP positions are not always homogeneous:

- fungible share tokens imply pro-rata pool ownership,
- NFT-like positions imply individualized fee and range exposure.

### 3.3 Fee Accrual

Uniswap liquidity documentation states that liquidity providers earn a share of swap fees proportional to their position.[^ref-uni-liq]

## 4. Concentrated Liquidity

Uniswap documents concentrated liquidity as capital allocated inside a custom price range, improving capital efficiency but requiring active management.[^ref-uni-conc]

If price exits the selected interval, the position becomes inactive and stops earning fees until price reenters the range.[^ref-uni-conc]

## 5. Economic Model

For a simple pro-rata pool:

- LP share `s = lp_units / total_lp_units`
- claim on reserve X = `s * x`
- claim on reserve Y = `s * y`

For concentrated-liquidity systems, fee and inventory outcomes depend on:

- chosen lower and upper bounds,
- time spent in range,
- and realized trade flow.

## 6. Risks

Uniswap's glossary defines impermanent loss as the opportunity cost LPs experience when token prices change relative to simply holding the tokens.[^ref-uni-glossary]

Main LP risks:

- impermanent loss,
- active-management burden,
- smart-contract risk,
- thin fee income relative to volatility.

## 7. Institutional Thinking

- Pool TVL is not the same as usable depth at a given price.
- Concentrated liquidity improves capital efficiency but increases operational complexity.
- LP returns should be evaluated net of adverse inventory rebalancing.

## 8. References

[^ref-uni-how]: Uniswap Developers, "How Uniswap Works," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/get-started/concepts/how-uniswap-works

[^ref-uni-liq]: Uniswap Developers, "Liquidity Overview," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/liquidity/overview

[^ref-uni-conc]: Uniswap Developers, "Concentrated Liquidity," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/get-started/concepts/liquidity-providers/concentrated-liquidity

[^ref-uni-glossary]: Uniswap Developers, "Glossary," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/get-started/concepts/glossary

## 9. Cross References

- Previous: DEFI-001 — AMM
- Next: DEFI-003 — DEX

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
