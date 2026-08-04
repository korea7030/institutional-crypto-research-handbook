---
knowledge_id: DEFI-010
title: Bridges
subtitle: Cross-Chain Asset Movement, Message Passing, and the Trust Tradeoffs Behind Interoperability
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 105 min
estimated_study: 250 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Interoperability
  - Ethereum
parent:
  - DeFi
prerequisites:
  - TOKEN-STANDARDS-005
  - DEFI-003
related_topics:
  - Wrapped Assets
  - DeFi Risks
  - MEV
primary_sources:
  - REF-ETH-BRIDGES-2026-001
tags:
  - bridges
  - cross-chain
  - interoperability
---

# Bridges
> DeFi  
> Research Unit: DEFI-010

---

## Research Brief

```yaml
knowledge_id: DEFI-010
title: Bridges
research_question: >
  How do blockchain bridges move assets and messages across chains, what trust
  models exist, and why are bridges among the highest-consequence risk surfaces
  in DeFi?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-005
  - DEFI-003
parent: DeFi
previous: DEFI-009
next: DEFI-011
```

## 1. Learning Objectives

- Define blockchain bridges.
- Distinguish lock-and-mint, burn-and-mint, and atomic-swap designs.
- Distinguish trusted and trust-minimized bridges.
- Explain why bridge risk is systemic.

## 2. Executive Summary

Ethereum.org explains that bridges connect blockchain networks and support asset, message, and data transfer across otherwise siloed chains.[^ref-eth-bridges]

The same source identifies major mechanisms:

- lock and mint,
- burn and mint,
- atomic swaps.[^ref-eth-bridges]

It also highlights security, convenience, connectivity, and data-passing ability as core bridge tradeoffs.[^ref-eth-bridges]

## 3. Bridge Models

### 3.1 Lock and Mint

Assets are locked on the source chain and represented on the destination chain.

### 3.2 Burn and Mint

A representation is burned on one chain and minted on another.

### 3.3 Liquidity-Network Transfers

Liquidity networks rely on counterpart liquidity instead of canonical custody paths.

## 4. Trust Model

Ethereum.org distinguishes trusted and trustless bridge categories, emphasizing that external validators introduce additional trust assumptions.[^ref-eth-bridges]

## 5. Risks

Ethereum.org explicitly states that bridges have accounted for several of the largest DeFi hacks and identifies:

- smart-contract risk,
- systemic risk from wrapped assets,
- counterparty risk,
- unresolved open issues.[^ref-eth-bridges]

## 6. Institutional Thinking

- Bridging is not simple transport; it is risk transformation.
- Bridge usage imports the bridge's security model into every downstream protocol using the bridged asset.

## 7. References

[^ref-eth-bridges]: ethereum.org, "Bridges," official documentation, page last update 2026-04-03, accessed 2026-08-04, https://ethereum.org/developers/docs/bridges

## 8. Cross References

- Previous: DEFI-009 — Restaking
- Next: DEFI-011 — MEV

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
