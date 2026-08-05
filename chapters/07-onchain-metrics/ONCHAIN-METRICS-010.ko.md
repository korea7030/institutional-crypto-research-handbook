---
knowledge_id: ONCHAIN-METRICS-010
title: Coin Days Destroyed
subtitle: 연령 가중 transfer activity와 공급 재활성화
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 90 min
estimated_study: 210 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Holder Behavior
parent:
  - Onchain Metrics
prerequisites:
  - ONCHAIN-METRICS-009
related_topics:
  - Dormancy
  - SOPR
primary_sources:
  - REF-GN-CDD-2026-001
tags:
  - cdd
  - coin-days-destroyed
---

# Coin Days Destroyed
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-010

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-010
title: Coin Days Destroyed
research_question: >
  Coin Days Destroyed는 transaction activity를 coin age로 어떻게 가중하며,
  왜 raw transfer count보다 공급 재활성화를 더 잘 드러낼 수 있는가?
document_type: foundation
difficulty: L400
prerequisites:
  - ONCHAIN-METRICS-009
parent: Onchain Metrics
previous: ONCHAIN-METRICS-009
next: ONCHAIN-METRICS-011
```

## 1. Learning Objectives

- Coin Days Destroyed를 정의할 수 있다.
- age weighting을 설명할 수 있다.
- CDD와 raw onchain volume을 구분할 수 있다.

## 2. Executive Summary

Glassnode는 Coin Days Destroyed를, 이전에 dormant 상태였던 코인이 움직일 때 축적된 coin day가 소멸되는 현상으로 문서화한다.[^ref-gn-cdd]

예를 들어 코인 1개가 100일 동안 가만히 있다가 움직이면, 어제 움직인 코인 1개보다 더 많은 coin day를 파괴한다.

## 3. Formula

개념적으로:

`CDD = Sum(Coin Amount * Days Since Last Move)`[^ref-gn-cdd]

## 4. Interpretation

- high CDD: 오래된 공급이 활성화,
- low CDD: 젊은 공급의 이동이 우세.

## 5. Institutional Thinking

- CDD는 가격 상승 국면에서 unusually old-coin activity가 동반될 때 특히 유용하다. 이는 역사적으로 인내심 있던 holder가 시장에 참여하고 있음을 시사할 수 있다.

## 6. References

[^ref-gn-cdd]: Glassnode Docs, "CDD (Coin Days Destroyed)" metric guide, accessed 2026-08-04, https://docs.glassnode.com/guides-and-tutorials/metric-guides/cdd

## 7. Cross References

- Previous: ONCHAIN-METRICS-009 — Dormancy
- Next: ONCHAIN-METRICS-011 — Realized Cap

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
