---
knowledge_id: DEFI-008
title: Staking
subtitle: bonded capital을 통한 프로토콜 보안, validator reward, 그리고 DeFi로 확장되는 liquid staking
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
  Ethereum에서 staking은 무엇이며, validator reward와 penalty는 어떻게
  작동하고, DeFi는 native staking을 pooled 및 liquid staking 모델로 어떻게
  확장하는가?
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

- consensus layer에서 staking을 정의할 수 있다.
- validator reward와 penalty를 설명할 수 있다.
- native, pooled, liquid staking을 구분할 수 있다.
- liquid staking이 consensus를 DeFi와 연결하는 이유를 설명할 수 있다.

## 2. Executive Summary

ethereum.org는 staking을 validator software를 활성화하고 Ethereum 보안을 돕기 위해 ETH를 deposit하고 그 대가로 reward를 받는 행위로 설명한다.[^ref-eth-staking]

Ethereum의 PoS 문서는 validator가 명시적으로 자본을 stake하며, 부정직하게 행동하면 그 자본이 파괴될 수 있다고 설명한다.[^ref-eth-pos]

DeFi 관점의 중요성은 pooled staking과 liquid staking system이 validator economics를 온체인 시장에서 활용 가능한 tokenized position으로 바꾼다는 점에 있다.[^ref-eth-staking]

## 3. Native Staking

Ethereum Launchpad FAQ는 validator 하나를 활성화하려면 최소 32 ETH가 필요하다고 말한다.[^ref-eth-faq]

## 4. Rewards and Penalties

ethereum.org는 올바른 참여에 대한 validator reward와, 특정 실패 또는 악의적 행위에 대한 penalty 또는 slashing을 문서화한다.[^ref-eth-rp]

## 5. DeFi Extension

ethereum.org는 pooled staking과 liquid staking을 통해 사용자가 32 ETH 미만으로도 staking에 참여하고, DeFi에서 사용할 수 있는 liquidity representation을 받을 수 있다고 설명한다.[^ref-eth-staking]

## 6. Risks

- validator operational failure,
- slashing,
- withdrawal constraint,
- staking-provider centralization,
- LST depeg risk.

## 7. Institutional Thinking

- native staking은 Ethereum을 보안하지만, liquid staking은 composability를 추가하는 대신 추가적인 protocol layer와 counterparty surface도 도입한다.

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
