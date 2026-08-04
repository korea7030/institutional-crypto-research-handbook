---
knowledge_id: DEFI-007
title: Yield Farming
subtitle: Incentive-Driven Capital Migration, Liquidity Mining, and Reward-Layer Economics on Top of Base Protocol Yield
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 95 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Incentives
  - Ethereum
parent:
  - DeFi
prerequisites:
  - DEFI-002
  - DEFI-004
related_topics:
  - Staking
  - Tokenomics
  - DeFi Risks
primary_sources:
  - REF-UNI-LM-2026-001
  - REF-AAVE-SAFETY-INC-2026-001
tags:
  - yield-farming
  - liquidity-mining
  - incentives
---

# Yield Farming
> DeFi  
> Research Unit: DEFI-007

---

## Research Brief

```yaml
knowledge_id: DEFI-007
title: Yield Farming
research_question: >
  How does yield farming layer incentive rewards on top of base DeFi positions,
  what behaviors does it subsidize, and why can incentive-driven liquidity be
  temporary, reflexive, and operationally fragile?
document_type: foundation
difficulty: L300
prerequisites:
  - DEFI-002
  - DEFI-004
parent: DeFi
previous: DEFI-006
next: DEFI-008
```

## 1. Learning Objectives

- Define yield farming precisely.
- Distinguish base yield from subsidy yield.
- Explain liquidity mining.
- Identify reflexive incentive risks.

## 2. Executive Summary

Uniswap's liquidity-mining documentation describes external reward programs that incentivize in-range liquidity provision through configurable incentives with reward tokens, target pools, and time windows.[^ref-uni-lm]

Aave's safety-incentives documentation describes rewards paid to stakers who accept slashing risk in the Safety Module, with dynamic reward logic in Umbrella.[^ref-aave-safety]

Together these show that yield farming is not one protocol primitive. It is an incentive layer added on top of a base position such as:

- LP capital,
- supplied lending liquidity,
- or staked insurance capital.

## 3. Mechanism

Base yield may come from:

- swap fees,
- borrow interest,
- staking rewards.

Farm yield is additional token compensation to encourage targeted behavior.

## 4. Economic Model

Total observed yield can be approximated as:

`Y_total = Y_base + Y_incentive`

If `Y_incentive` dominates, liquidity may leave when incentives decay.

## 5. Risks

- mercenary capital,
- token-price dependence,
- hidden lockups or slashing exposure,
- rapid APY collapse when programs end.

## 6. Institutional Thinking

- Incentive APR is not the same as durable protocol revenue.
- Farming can bootstrap activity, but it can also camouflage weak organic demand.

## 7. References

[^ref-uni-lm]: Uniswap Developers, "Liquidity Mining," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/protocols/v3/concepts/liquidity-mining

[^ref-aave-safety]: Aave Help, "Safety Incentives," official documentation, accessed 2026-08-04, https://aave.com/help/umbrella/safety-incentives

## 8. Cross References

- Previous: DEFI-006 — Liquidation
- Next: DEFI-008 — Staking

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
