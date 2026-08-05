---
knowledge_id: ONCHAIN-METRICS-005
title: Active Addresses
subtitle: 참여의 대리변수, usage 신호, 그리고 activity와 user를 분리해야 하는 이유
version: 1.0.0
status: Reviewed
difficulty: L200
estimated_reading: 80 min
estimated_study: 180 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Network Activity
parent:
  - Onchain Metrics
prerequisites:
  - ONCHAIN-METRICS-004
related_topics:
  - Network Growth
  - Metric Limitations
primary_sources:
  - REF-CM-ACTIVEADDR-2026-001
tags:
  - active-addresses
  - network-activity
---

# Active Addresses
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-005

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-005
title: Active Addresses
research_question: >
  Active-address metric은 실제로 무엇을 세며, 왜 분석가는 address activity를
  고유한 인간 사용자 성장과 동일시해서는 안 되는가?
document_type: foundation
difficulty: L200
prerequisites:
  - ONCHAIN-METRICS-004
parent: Onchain Metrics
previous: ONCHAIN-METRICS-004
next: ONCHAIN-METRICS-006
```

## 1. Learning Objectives

- active address를 정의할 수 있다.
- activity count와 user count를 구분할 수 있다.
- batching과 contract usage가 metric을 어떻게 왜곡하는지 설명할 수 있다.

## 2. Executive Summary

Coin Metrics는 active-address metric을 일정 기간 동안 transaction에 참여한 address 수로 문서화한다.[^ref-cm-active]

이는 유용한 participation proxy다. 그러나 unique user의 직접 측정치는 아니다. 한 사용자가 많은 address를 통제할 수 있고, 하나의 address가 exchange batching이나 application-level aggregation을 대표할 수도 있다.

## 3. Interpretation

active address 증가가 의미할 수 있는 것은 다음과 같다.

- 더 강한 transactional activity,
- renewed speculation,
- application usage 증가,
- 혹은 spam과 bot behavior.

## 4. Limits

- exchange batching은 겉보기 count를 줄일 수 있다.
- UTXO와 account-based system은 다르게 동작한다.
- contract architecture는 얼마나 많은 address가 active하게 보이는지 바꾼다.

## 5. Institutional Thinking

- active address는 adoption count가 아니라 network-activity clue로 써야 한다.
- transaction count, fee, cohort context와 함께 봐야 한다.

## 6. References

[^ref-cm-active]: Coin Metrics Docs, active-address metric definitions such as `AdrActCnt`, accessed 2026-08-04, https://docs.coinmetrics.io/network-data/network-data-overview

## 7. Cross References

- Previous: ONCHAIN-METRICS-004 — Smart Money
- Next: ONCHAIN-METRICS-006 — SOPR

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
