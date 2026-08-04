---
knowledge_id: DEFI-001
title: AMM
subtitle: Automated Market Makers, Constant Functions, and Onchain Price Formation Without Order Books
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 100 min
estimated_study: 240 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Markets
  - Ethereum
parent:
  - DeFi
prerequisites:
  - TOKEN-STANDARDS-001
  - SMART-CONTRACTS-001
related_topics:
  - Liquidity Pools
  - DEX
  - MEV
primary_sources:
  - REF-UNI-HOW-2026-001
  - REF-UNI-GLOSSARY-2026-001
tags:
  - defi
  - amm
  - uniswap
---

# AMM
> DeFi  
> Research Unit: DEFI-001

---

## Research Brief

```yaml
knowledge_id: DEFI-001
title: AMM
research_question: >
  How do automated market makers replace order books with pool-based pricing,
  what invariant logic governs trades, and what execution tradeoffs follow from
  deterministic onchain liquidity?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-001
  - SMART-CONTRACTS-001
parent: DeFi
previous:
next: DEFI-002
related_topics:
  - Liquidity Pools
  - DEX
  - MEV
out_of_scope:
  - Full CFMM taxonomy
  - Formal proof of invariant safety
```

## 1. Learning Objectives

- Define an AMM as a smart-contract market structure.
- Explain constant-product pricing.
- Distinguish AMMs from order books.
- Explain slippage and price impact.
- Identify why AMMs are composable but path-dependent.

## 2. Executive Summary

Uniswap documents its protocol as an automated market maker that lets anyone swap tokens, provide liquidity, or create new markets directly onchain instead of relying on order books.[^ref-uni-how]

In the simple constant-product form, pool reserves `x` and `y` follow:

`x * y = k`[^ref-uni-how]

Traders do not match against posted limit orders. They trade against a pool whose price moves as reserve balances change. Larger trades relative to pool depth create larger price impact.

The analytical consequence is that AMMs make liquidity continuously available but not price-invariant. Execution quality depends on:

- pool depth,
- fee tier,
- routing,
- and transaction ordering.

## 3. Core Mechanics

### 3.1 Pool-Based Trading

Uniswap explains that liquidity pools are reserves of two ERC-20 tokens and that users swap against those reserves.[^ref-uni-how]

### 3.2 Constant Product

In early Uniswap versions, the constant-product invariant is commonly written as `x * y = k`.[^ref-uni-how][^ref-uni-glossary]

When a trader adds one token to the pool and removes the other, the pool state moves along the curve. This makes marginal price endogenous to trade size.

### 3.3 Price Impact

If a trade is small relative to reserves, execution is close to the current pool price. If a trade is large, execution moves deeper along the curve and worsens average fill price.

## 4. Mathematical Model

For reserves `(x, y)` and input `dx`, ignoring fees:

- new reserve of token X: `x' = x + dx`
- new reserve of token Y must satisfy: `x' * y' = k`
- so `y' = k / x'`
- output `dy = y - y'`

With fees, the effective input is reduced before applying the invariant.

## 5. Security and Market Implications

- AMMs are deterministic and public, so route quality and ordering are visible attack surfaces.
- Price discovery is endogenous to pool state and external arbitrage.
- AMMs can remain liquid during market stress, but execution quality may degrade sharply if pools are thin.

## 6. Institutional Thinking

- Treat AMMs as inventory-based markets, not as quote boards.
- A pool price is not a guarantee of size-independent execution.
- AMM markets depend on arbitrage to stay aligned with broader markets.

## 7. Evidence Classification

| Claim | Classification | Basis |
|---|---|---|
| Uniswap is an AMM instead of an order-book exchange | Directly specified | Uniswap docs |
| Constant product `x*y=k` governs early Uniswap pricing | Directly specified | Uniswap docs |
| Larger trades cause more price impact | Directly specified and inferred | Invariant mechanics |

## 8. References

[^ref-uni-how]: Uniswap Developers, "How Uniswap Works," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/get-started/concepts/how-uniswap-works

[^ref-uni-glossary]: Uniswap Developers, "Glossary," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/get-started/concepts/glossary

## 9. Cross References

- Next: DEFI-002 — Liquidity Pools

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
