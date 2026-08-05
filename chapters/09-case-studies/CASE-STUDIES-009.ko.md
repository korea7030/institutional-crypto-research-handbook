---
knowledge_id: CASE-STUDIES-009
title: Layer 2 Expansion
subtitle: rollup adoption, Ethereum의 network-of-networks thesis, 그리고 scaling turn
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 95 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - Case Studies
  - Ethereum
  - Scaling
parent:
  - Case Studies
prerequisites:
  - DEFI-010
  - CASE-STUDIES-006
related_topics:
  - Layer 2
  - Bridges
primary_sources:
  - REF-ETH-L2-2026-001
  - REF-ETH-SCALING-2026-001
  - REF-ARB-DOCS-2026-001
  - REF-OP-BRIDGE-2026-001
tags:
  - case-study
  - layer-2
  - scaling
---

# Layer 2 Expansion
> Case Studies  
> Research Unit: CASE-STUDIES-009

---

## Research Brief

```yaml
knowledge_id: CASE-STUDIES-009
title: Layer 2 Expansion
research_question: >
  Ethereum의 layer 2 expansion은 어떻게 post-Merge 이후의 핵심 market 및
  product trend가 되었으며, 연구자는 이를 단순한 chain proliferation이
  아니라 scaling strategy로 왜 분석해야 하는가?
document_type: case-study
difficulty: L300
prerequisites:
  - DEFI-010
  - CASE-STUDIES-006
parent: Case Studies
previous: CASE-STUDIES-008
next: CASE-STUDIES-010
```

## 1. Observation

2026년 무렵 Ethereum은 더 이상 하나의 execution environment만이 아니었다. ethereum.org는 Ethereum을 수백 개의 blockchain이 그 위에 구축된 네트워크로 명시적으로 설명했다.[^ref-eth-l2]

## 2. Source Context

ethereum.org의 layer 2 문서는 Ethereum의 strength와 security가 다른 네트워크가 그 위에 구축될 수 있는 플랫폼을 제공하며, 이러한 네트워크를 통해 Ethereum이 더 저렴하고 빠르며 접근 가능한 환경이 되었다고 설명한다.[^ref-eth-l2]

ethereum.org의 scaling roadmap은 Ethereum이 layer 2, 즉 rollup을 통해 확장된다고 명시한다.[^ref-eth-scaling]

Arbitrum과 Optimism 문서는 dedicated rollup platform과 standard bridge system을 통해 이 변화의 구현 측면을 보여준다.[^ref-arb][^ref-op]

## 3. Event Structure

layer 2 expansion은 단순히 더 많은 체인이 생긴 사건이 아니었다. 그것은 Ethereum scaling model의 전략적 재정의였다.

- mainnet 밖에서의 execution,
- Ethereum 위 settlement와 data anchoring,
- bridge를 통한 asset movement,
- 더 저렴한 환경을 향한 application migration.

## 4. Why It Mattered

이 변화는 분석가가 Ethereum 성장을 측정하는 방식을 바꾸었다.

- L1 fee나 L1 address만이 아니라,
- rollup과 bridge-linked environment 전반의 총 ecosystem usage를 보게 만들었다.

## 5. Lessons

- scaling 성공은 표면 metric을 분절시키면서도 base platform은 강화할 수 있다.
- L2 성장은 user economics를 개선하지만 bridge, fragmentation, governance complexity도 추가한다.
- 2022년 이후 Ethereum thesis는 점점 "network of networks"가 되었다.

## 6. Institutional Thinking

- 연구자는 L2 proliferation을 default로 dilution으로 취급해서는 안 된다.
- 올바른 질문은 L2 성장이 Ethereum settlement demand와 ecosystem stickiness를 강화하는가다.

## 7. References

[^ref-eth-l2]: ethereum.org, "Layer 2," page last updated July 23, 2026, accessed 2026-08-04, https://ethereum.org/layer-2

[^ref-eth-scaling]: ethereum.org, "Scaling Ethereum," page last updated June 24, 2026, accessed 2026-08-04, https://ethereum.org/roadmap/scaling/

[^ref-arb]: Arbitrum Docs, official documentation homepage, accessed 2026-08-04, https://docs.arbitrum.io/

[^ref-op]: Optimism Docs, "Using the Standard Bridge," accessed 2026-08-04, https://docs.optimism.io/app-developers/guides/bridging/standard-bridge

## 8. Cross References

- Previous: CASE-STUDIES-008 — Memecoin Cycles
- Next: CASE-STUDIES-010 — AI x Crypto Narratives

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
