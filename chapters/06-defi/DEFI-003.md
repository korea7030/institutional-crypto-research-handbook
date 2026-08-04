---
knowledge_id: DEFI-003
title: DEX
subtitle: Decentralized Exchange Architecture, Self-Custody Execution, and Onchain Market Access
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 95 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Trading
  - Ethereum
parent:
  - DeFi
prerequisites:
  - DEFI-001
  - DEFI-002
related_topics:
  - MEV
  - Bridges
  - Token Standards
primary_sources:
  - REF-UNI-HOW-2026-001
  - REF-UNI-PROTOCOLS-2026-001
  - REF-UNI-SWAP-2026-001
tags:
  - dex
  - trading
  - uniswap
---

# DEX
> DeFi  
> Research Unit: DEFI-003

---

## Research Brief

```yaml
knowledge_id: DEFI-003
title: DEX
research_question: >
  What defines a decentralized exchange at the protocol layer, how does
  self-custodial execution differ from centralized exchange workflows, and what
  risks remain even when custody is removed?
document_type: foundation
difficulty: L300
prerequisites:
  - DEFI-001
  - DEFI-002
parent: DeFi
previous: DEFI-002
next: DEFI-004
related_topics:
  - MEV
  - Bridges
```

## 1. Learning Objectives

- Define a DEX in protocol terms.
- Distinguish custody, execution, and governance layers.
- Explain routing and swap safety parameters.
- Identify DEX-specific execution risks.

## 2. Executive Summary

Uniswap documents itself as a decentralized exchange protocol built on Ethereum and designed for decentralization, censorship resistance, and self-custody.[^ref-uni-how]

In a DEX, users generally retain asset custody until interacting with smart contracts. Execution is rule-based and public, not dependent on a centralized exchange operator maintaining an internal ledger.

This removes some centralized exchange risks, but it does not remove:

- smart-contract risk,
- oracle or pricing mistakes,
- frontrunning,
- adverse routing,
- or bridge and wrapped-asset dependencies.

## 3. Architecture

### 3.1 Core Exchange Logic

Uniswap's protocol overview describes a suite of onchain protocols centered on AMM contracts that hold liquidity and execute swaps.[^ref-uni-protocols]

### 3.2 Self-Custody

DEX execution does not require users to pre-deposit assets into a centralized operator balance sheet in the same way as a CEX.

### 3.3 Routing and Safety

Uniswap's swap guidance notes that smart-contract trading should use external price sources and safety parameters such as minimum output or maximum input bounds to reduce frontrun loss.[^ref-uni-swap]

## 4. Operational Differences vs CEX

- Settlement occurs onchain.
- Market data and transaction intent are public before inclusion.
- Execution depends on block inclusion and ordering.
- Assets remain in user-controlled wallets until contract interaction.

## 5. Economic and Market Implications

DEXes widen access and composability because other protocols can directly integrate exchange logic. However, composability also means DEXes become core infrastructure for:

- arbitrage,
- liquidations,
- collateral management,
- and token price formation.

## 6. Risks

- Slippage from thin liquidity.
- MEV from public mempools.
- Token approval abuse.
- Wrapped or bridged-asset misidentification.
- Smart-contract or router bugs.

## 7. Institutional Thinking

- A DEX is not simply "a venue"; it is programmable market infrastructure.
- Removing custody does not remove execution risk.
- Onchain transparency creates both auditability and extractable value opportunities.

## 8. References

[^ref-uni-how]: Uniswap Developers, "How Uniswap Works," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/get-started/concepts/how-uniswap-works

[^ref-uni-protocols]: Uniswap Developers, "Protocols Overview," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/protocols/overview

[^ref-uni-swap]: Uniswap Developers, "Implement a Swap," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/protocols/v2/guides/swapping

## 9. Cross References

- Previous: DEFI-002 — Liquidity Pools
- Next: DEFI-004 — Lending

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
