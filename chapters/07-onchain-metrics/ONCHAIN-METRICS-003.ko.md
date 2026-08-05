---
knowledge_id: ONCHAIN-METRICS-003
title: Whale Activity
subtitle: 대형 보유자 행동, 집중도 단서, 그리고 address 기반 추론의 한계
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 85 min
estimated_study: 200 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Ownership
parent:
  - Onchain Metrics
prerequisites:
  - ONCHAIN-METRICS-002
related_topics:
  - Smart Money
  - Exchange Flows
primary_sources:
  - REF-CM-ADDRBAL-2026-001
tags:
  - whales
  - concentration
---

# Whale Activity
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-003

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-003
title: Whale Activity
research_question: >
  분석가는 address balance와 transfer data를 이용해 whale activity를 어떻게
  연구해야 하며, 왜 address concentration은 beneficial ownership과 불완전하게만
  대응하는가?
document_type: foundation
difficulty: L300
prerequisites:
  - ONCHAIN-METRICS-002
parent: Onchain Metrics
previous: ONCHAIN-METRICS-002
next: ONCHAIN-METRICS-004
```

## 1. Learning Objectives

- 모호한 소셜미디어 용법에 기대지 않고 whale activity를 정의할 수 있다.
- large address와 large beneficial owner의 차이를 설명할 수 있다.
- concentration 및 transfer-size metric을 신중하게 사용할 수 있다.

## 2. Executive Summary

Coin Metrics는 특정 balance threshold를 넘는 address 수와 같은 balance-distribution metric을 문서화한다.[^ref-cm-addrbal]

이 metric들은 종종 whale activity의 대리변수로 사용된다. 집중도 변화, 대형 코호트의 accumulation, 큰 transfer regime를 식별하는 데는 유용할 수 있다. 그러나 하나의 address가 많은 사용자를 대표할 수 있고, 하나의 사용자가 많은 address를 통제할 수도 있다.

## 3. Common Proxies

- threshold 이상 address 수,
- 상위 balance band가 보유한 공급량,
- large-transfer count,
- 대형 entity의 exchange inflow/outflow.

## 4. Interpretive Cautions

- exchange wallet이 whale처럼 보일 수 있다.
- custody fragmentation은 whale을 숨길 수 있다.
- threshold 선택에 따라 서사가 임의적으로 달라질 수 있다.

## 5. Institutional Thinking

- whale 분석은 가능하면 address가 아니라 cohort 및 entity level에서 수행하는 편이 정확하다.
- address-level concentration을 beneficial-owner concentration의 직접 증거처럼 제시해서는 안 된다.

## 6. References

[^ref-cm-addrbal]: Coin Metrics Docs, balance-distribution and address-count metric documentation, accessed 2026-08-04, https://docs.coinmetrics.io/network-data/network-data-overview

## 7. Cross References

- Previous: ONCHAIN-METRICS-002 — Exchange Reserve
- Next: ONCHAIN-METRICS-004 — Smart Money

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
