---
knowledge_id: DEFI-012
title: DeFi Risks
subtitle: Smart Contract, Oracle, Governance, Liquidity, and Cross-Protocol Failure Modes
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 110 min
estimated_study: 260 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Risk
  - Ethereum
parent:
  - DeFi
prerequisites:
  - DEFI-004
  - DEFI-010
  - DEFI-011
related_topics:
  - Bridges
  - Liquidation
  - Tokenomics
primary_sources:
  - REF-AAVE-RISK-2026-001
  - REF-AAVE-UMBRELLA-2026-001
  - REF-ETH-BRIDGES-2026-001
  - REF-UNI-GLOSSARY-2026-001
tags:
  - defi-risks
  - risk-management
  - smart-contract-risk
---

# DeFi Risks
> DeFi  
> Research Unit: DEFI-012

---

## Research Brief

```yaml
knowledge_id: DEFI-012
title: DeFi Risks
research_question: >
  What are the major risk classes in DeFi, how do they interact across
  composable protocols, and how should researchers separate isolated contract
  risk from system-wide dependency risk?
document_type: foundation
difficulty: L300
prerequisites:
  - DEFI-004
  - DEFI-010
  - DEFI-011
parent: DeFi
previous: DEFI-011
next:
```

## 1. Learning Objectives

- Classify major DeFi risk types.
- Distinguish local vs systemic risk.
- Explain composability risk.
- Explain why yield can be downstream of hidden dependency chains.

## 2. Executive Summary

Aave governance and guidance materials identify core DeFi risk classes such as smart-contract risk, admin-key risk, oracle risk, and market risk.[^ref-aave-risk]

Ethereum.org's bridge documentation adds systemic risks from wrapped assets and bridge trust assumptions.[^ref-eth-bridges]

Uniswap's glossary highlights impermanent loss as a specific LP opportunity-cost risk.[^ref-uni-glossary]

Taken together, DeFi risk is best studied as a layered dependency system rather than a checklist of isolated contract bugs.

## 3. Major Risk Classes

### 3.1 Smart Contract Risk

Code can fail, be exploited, or interact unexpectedly with dependencies.

### 3.2 Oracle Risk

Incorrect price inputs distort borrowing, liquidation, and derivatives settlement.[^ref-aave-risk]

### 3.3 Governance and Admin Risk

Admin keys, guardians, or governance capture can change protocol behavior rapidly.[^ref-aave-risk][^ref-aave-umbrella]

### 3.4 Market and Liquidity Risk

Thin liquidity, volatile collateral, and stressed liquidations can break intended risk controls.

### 3.5 Bridge and Wrapped-Asset Risk

Bridged or wrapped assets import external settlement and validation risk.[^ref-eth-bridges]

## 4. Composability Risk

One position can depend simultaneously on:

- a lending protocol,
- an LP token,
- a bridge,
- an oracle,
- and a governance-controlled wrapper.

Failure may begin at any one layer and propagate.

## 5. Institutional Thinking

- Audit count alone is insufficient.
- Risk review should map dependency graphs, not just primary contracts.
- High yield often signals concentrated or layered risk, not free efficiency.

## 6. References

[^ref-aave-risk]: Aave Governance, "Asset Management Guidelines" and related risk discussion, accessed 2026-08-04, https://governance.aave.com/t/aave-asset-management-guidelines/5600

[^ref-aave-umbrella]: Aave Help, "Umbrella," official documentation, accessed 2026-08-04, https://aave.com/help/umbrella/umbrella

[^ref-eth-bridges]: ethereum.org, "Bridges," official documentation, page last update 2026-04-03, accessed 2026-08-04, https://ethereum.org/developers/docs/bridges

[^ref-uni-glossary]: Uniswap Developers, "Glossary," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/get-started/concepts/glossary

## 7. Cross References

- Previous: DEFI-011 — MEV
- Phase 6 complete

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
