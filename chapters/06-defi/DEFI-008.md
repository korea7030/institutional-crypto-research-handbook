---
knowledge_id: DEFI-008
title: Staking
subtitle: Protocol Security Through Bonded Capital, Validator Rewards, and Liquid Staking Extensions Into DeFi
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 105 min
estimated_study: 250 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Consensus
  - Ethereum
parent:
  - DeFi
prerequisites:
  - ETHEREUM-FOUNDATION-007
  - TOKEN-STANDARDS-005
related_topics:
  - Restaking
  - Yield Farming
  - DeFi Risks
primary_sources:
  - REF-ETH-STAKING-2025-001
  - REF-ETH-POS-2026-001
  - REF-ETH-POS-RP-2026-001
tags:
  - staking
  - ethereum
  - validators
---

# Staking
> DeFi  
> Research Unit: DEFI-008

---

## Research Brief

```yaml
knowledge_id: DEFI-008
title: Staking
research_question: >
  What is staking on Ethereum, how do validator rewards and penalties work, and
  how does DeFi extend native staking through pooled and liquid staking models?
document_type: foundation
difficulty: L300
prerequisites:
  - ETHEREUM-FOUNDATION-007
  - TOKEN-STANDARDS-005
parent: DeFi
previous: DEFI-007
next: DEFI-009
```

## 1. Learning Objectives

- Define staking at the consensus layer.
- Explain validator rewards and penalties.
- Distinguish native, pooled, and liquid staking.
- Explain why liquid staking links consensus to DeFi.

## 2. Executive Summary

Ethereum.org explains that staking is the act of depositing ETH to activate validator software and help secure Ethereum while earning rewards.[^ref-eth-staking]

Ethereum's PoS documentation states that validators explicitly stake capital that can be destroyed if they act dishonestly.[^ref-eth-pos]

DeFi relevance arises because pooled and liquid staking systems transform validator economics into tokenized positions usable across onchain markets.[^ref-eth-staking]

## 3. Native Staking

The Ethereum Launchpad FAQ states that each validator requires at least 32 ETH to be activated.[^ref-eth-faq]

## 4. Rewards and Penalties

Ethereum.org documents validator rewards for valid participation and penalties or slashing for certain failures or malicious behavior.[^ref-eth-rp]

## 5. DeFi Extension

Ethereum.org notes that pooled staking and liquid staking let users stake less than 32 ETH and receive liquidity representations usable in DeFi.[^ref-eth-staking]

## 6. Risks

- validator operational failure,
- slashing,
- withdrawal constraints,
- staking-provider centralization,
- LST depeg risk.

## 7. Institutional Thinking

- Native staking secures Ethereum; liquid staking adds composability but also additional protocol layers and counterparty surfaces.

## 8. References

[^ref-eth-staking]: ethereum.org, "Ethereum staking: How does it work?," official documentation, page last updated 2025-02-12, accessed 2026-08-04, https://ethereum.org/staking/

[^ref-eth-pos]: ethereum.org, "Proof-of-stake (PoS)," official documentation, accessed 2026-08-04, https://ethereum.org/developers/docs/consensus-mechanisms/pos/

[^ref-eth-rp]: ethereum.org, "Proof-of-stake rewards and penalties," official documentation, accessed 2026-08-04, https://ethereum.org/developers/docs/consensus-mechanisms/pos/rewards-and-penalties/

[^ref-eth-faq]: Ethereum Staking Launchpad, "Validator FAQs," official documentation, accessed 2026-08-04, https://launchpad.ethereum.org/en/faq

## 9. Cross References

- Previous: DEFI-007 — Yield Farming
- Next: DEFI-009 — Restaking

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
