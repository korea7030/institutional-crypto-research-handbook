---
knowledge_id: ONCHAIN-METRICS-009
title: Dormancy
subtitle: holder의 인내와 지출 강도를 읽는 coin-age consumption 창
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
  - ONCHAIN-METRICS-008
related_topics:
  - Coin Days Destroyed
  - SOPR
primary_sources:
  - REF-GN-DORM-2026-001
tags:
  - dormancy
  - holder-behavior
---

# Dormancy
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-009

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-009
title: Dormancy
research_question: >
  Coin dormancy는 무엇을 측정하며, coin-age spending과 어떻게 연결되고,
  spike가 오래된 holder의 distribution을 시사할 수는 있어도 그 자체로
  capitulation이나 top을 증명하지는 못하는 이유는 무엇인가?
document_type: foundation
difficulty: L400
prerequisites:
  - ONCHAIN-METRICS-008
parent: Onchain Metrics
previous: ONCHAIN-METRICS-008
next: ONCHAIN-METRICS-010
```

## 1. Learning Objectives

- dormancy를 정의할 수 있다.
- dormancy와 transaction count를 구분할 수 있다.
- old-coin movement가 왜 중요한지 설명할 수 있다.

## 2. Executive Summary

Glassnode는 Average Coin Dormancy를, 사용된 output이 이동되기 전까지 평균 몇 일 동안 dormant 상태였는지를 보여주는 metric으로 문서화한다.[^ref-gn-dorm]

즉 dormancy는 단지 얼마나 많은 코인이 움직였는지가 아니라, 움직일 때 그 코인이 얼마나 오래 묵어 있었는지를 묻는다.

## 3. Interpretation

높은 dormancy는 다음을 시사할 수 있다.

- 오래된 holder의 distribution,
- 장기간 비활성 공급의 재가동,
- 초기 코호트의 구조적 재배치.

낮은 dormancy는 상대적으로 젊은 코인 이동이 우세함을 시사할 수 있다.

## 4. Limits

- exchange 내부 재정리는 데이터를 오염시킬 수 있다.
- old-coin movement가 항상 재량적 매도를 뜻하지는 않는다.
- cohort context가 중요하다.

## 5. Institutional Thinking

- dormancy는 cohort segmentation 및 realized-profit metric과 함께 볼 때 가장 정보력이 높다.

## 6. References

[^ref-gn-dorm]: Glassnode Docs, "Dormancy" metric guide, accessed 2026-08-04, https://docs.glassnode.com/guides-and-tutorials/metric-guides/dormancy

## 7. Cross References

- Previous: ONCHAIN-METRICS-008 — NUPL
- Next: ONCHAIN-METRICS-010 — Coin Days Destroyed

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
