---
knowledge_id: ONCHAIN-METRICS-014
title: Network Growth
subtitle: 신규 address 형성, usage 확장, 그리고 adoption 추론
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 85 min
estimated_study: 200 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Network Activity
parent:
  - Onchain Metrics
prerequisites:
  - ONCHAIN-METRICS-005
related_topics:
  - Active Addresses
  - Metric Limitations
primary_sources:
  - REF-CM-NEWADDR-2026-001
tags:
  - network-growth
  - adoption
---

# Network Growth
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-014

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-014
title: Network Growth
research_question: >
  분석가는 new-address 및 first-use 계열 metric을 adoption evidence로 어떻게
  사용해야 하며, 왜 network-growth signal은 wallet architecture 변화와
  artificial activity에 취약한가?
document_type: foundation
difficulty: L300
prerequisites:
  - ONCHAIN-METRICS-005
parent: Onchain Metrics
previous: ONCHAIN-METRICS-013
next: ONCHAIN-METRICS-015
```

## 1. Learning Objectives

- network-growth 스타일 metric을 정의할 수 있다.
- new address와 new user를 구분할 수 있다.
- address creation policy가 왜 중요한지 설명할 수 있다.

## 2. Executive Summary

Coin Metrics는 new funded address count 같은 address-creation 및 first-funding 계열 metric을 문서화한다.[^ref-cm-newaddr]

이 metric은 기존 address의 반복 activity만이 아니라 새로운 address 참여를 포착하려 하기 때문에 유용한 growth proxy가 될 수 있다. 하지만 wallet architecture, exchange batching, spam, protocol design은 진짜 adoption 변화 없이도 이 수치를 바꿀 수 있다.

## 3. Institutional Thinking

- network growth는 active address, fee, retained balance, application-level usage와 함께 읽어야 한다.
- 빠른 address 성장이라도 retention이 약하면, 더 느리지만 sticky한 user expansion보다 의미가 작을 수 있다.

## 4. References

[^ref-cm-newaddr]: Coin Metrics Docs, new-address and first-funded-address metric documentation, accessed 2026-08-04, https://docs.coinmetrics.io/network-data/network-data-overview

## 5. Cross References

- Previous: ONCHAIN-METRICS-013 — TVL
- Next: ONCHAIN-METRICS-015 — Metric Limitations

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
