---
knowledge_id: ONCHAIN-METRICS-015
title: Metric Limitations
subtitle: label 오류, structural drift, 그리고 온체인 데이터에 model discipline이 필요한 이유
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 90 min
estimated_study: 210 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Research Method
parent:
  - Onchain Metrics
prerequisites:
  - ONCHAIN-METRICS-014
related_topics:
  - Smart Money
  - TVL
  - Exchange Flows
primary_sources:
  - REF-CM-NETDATA-2026-001
  - REF-GN-METGUIDES-2026-001
  - REF-ETH-BRIDGES-2026-001
tags:
  - limitations
  - methodology
---

# Metric Limitations
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-015

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-015
title: Metric Limitations
research_question: >
  Onchain metric의 주요 failure mode는 무엇이며, 분석가는 raw blockchain
  observability와 데이터를 투자 근거로 바꾸는 데 필요한 modeling assumption을
  어떻게 분리해야 하는가?
document_type: foundation
difficulty: L300
prerequisites:
  - ONCHAIN-METRICS-014
parent: Onchain Metrics
previous: ONCHAIN-METRICS-014
next:
```

## 1. Learning Objectives

- onchain analytics의 주요 limitation class를 식별할 수 있다.
- observable base data와 modeled entity inference를 구분할 수 있다.
- metric skepticism checklist를 만들 수 있다.

## 2. Executive Summary

온체인 metric은 blockchain data가 공개되어 있기 때문에 객관적으로 느껴진다. 하지만 시장에 의미 있는 많은 metric은 raw chain data 위에 강한 modeling을 얹어야만 만들어진다.

Coin Metrics의 network-data 문서는 metric이 정의, 변환, entity 처리에 의존함을 보여준다.[^ref-cm-netdata]

Glassnode의 metric guide 역시 널리 인용되는 signal이 construction choice와 heuristic threshold를 내포하고 있음을 보여준다.[^ref-gn-guides]

Ethereum의 bridge 문서는 또 다른 구조적 문제를 강조한다. 경제 활동이 이제 wrapped asset와 여러 체인에 걸쳐 분산되므로, 순진한 single-chain 해석은 점점 약해진다.[^ref-eth-bridges]

## 3. Limitation Classes

- entity-label error,
- address != user,
- internal transfer contamination,
- cross-chain fragmentation,
- recursion and double counting,
- wallet 또는 protocol design 변화로 인한 structural break,
- 과적합된 historical threshold.

## 4. Research Discipline

어떤 metric을 신뢰하기 전에 다음을 물어야 한다.

1. 무엇이 직접 관측되는가?
2. 무엇이 추론되는가?
3. 어떤 label이 필요한가?
4. 이전 cycle 이후 구조적으로 무엇이 바뀌었는가?
5. 같은 결과를 설명할 수 있는 다른 가설은 무엇인가?

## 5. Institutional Thinking

- metric은 결론이 아니라 evidence input이다.
- 강한 리서치는 metric absolutism이 아니라 triangulation에서 나온다.

## 6. References

[^ref-cm-netdata]: Coin Metrics Docs, network-data overview and metric methodology, accessed 2026-08-04, https://docs.coinmetrics.io/network-data/network-data-overview

[^ref-gn-guides]: Glassnode Docs, metric guides and indicator methodologies, accessed 2026-08-04, https://docs.glassnode.com/guides-and-tutorials/metric-guides

[^ref-eth-bridges]: ethereum.org, "Bridges," official documentation, page last update 2026-04-03, accessed 2026-08-04, https://ethereum.org/developers/docs/bridges

## 7. Cross References

- Previous: ONCHAIN-METRICS-014 — Network Growth
- Phase 7 complete

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
