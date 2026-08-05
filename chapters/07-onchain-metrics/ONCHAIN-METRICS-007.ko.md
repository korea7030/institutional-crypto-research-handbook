---
knowledge_id: ONCHAIN-METRICS-007
title: MVRV
subtitle: 상대가치 평가 프레임워크로서의 Market Value 대 Realized Value
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 95 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Valuation
parent:
  - Onchain Metrics
prerequisites:
  - ONCHAIN-METRICS-006
related_topics:
  - NUPL
  - Realized Cap
primary_sources:
  - REF-GN-MVRV-2026-001
tags:
  - mvrv
  - valuation
---

# MVRV
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-007

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-007
title: MVRV
research_question: >
  MVRV는 market capitalization과 realized capitalization을 어떻게 비교하며,
  왜 단기 매매 trigger보다 cycle-aware valuation signal로 더 자주 쓰이는가?
document_type: foundation
difficulty: L400
prerequisites:
  - ONCHAIN-METRICS-006
parent: Onchain Metrics
previous: ONCHAIN-METRICS-006
next: ONCHAIN-METRICS-008
```

## 1. Learning Objectives

- MVRV를 정의할 수 있다.
- realized cap이 cost basis proxy로 쓰이는 이유를 설명할 수 있다.
- MVRV를 단일 시점의 확신이 아니라 regime context로 사용할 수 있다.

## 2. Executive Summary

Glassnode는 MVRV를 market cap과 realized cap의 비율로 문서화한다.[^ref-gn-mvrv]

핵심 직관은 market cap이 현재 spot valuation을 반영하고, realized cap은 코인이 마지막으로 이동했을 때의 집계 가치에 가까운 값을 반영한다는 점이다. 따라서 둘의 비율은 현재 valuation이 내재 cost basis 대비 얼마나 늘어났는지에 대한 대략적 척도를 제공한다.

## 3. Formula

`MVRV = Market Cap / Realized Cap`[^ref-gn-mvrv]

## 4. Interpretation

- high MVRV: 네트워크에 큰 unrealized profit이 내재,
- low MVRV: realized cost basis 대비 valuation 압축.

## 5. Limits

- 모든 regime에 통하는 보편적 threshold는 없다.
- supply migration pattern과 market structure는 진화한다.
- derivatives와 ETF channel은 cycle 간 직접 비교 가능성을 약화시킬 수 있다.

## 6. Institutional Thinking

- MVRV는 정밀한 타이밍보다 valuation regime을 프레이밍하는 데 더 유용하다.

## 7. References

[^ref-gn-mvrv]: Glassnode Docs, "MVRV Ratio" metric guide, accessed 2026-08-04, https://docs.glassnode.com/guides-and-tutorials/metric-guides/mvrv

## 8. Cross References

- Previous: ONCHAIN-METRICS-006 — SOPR
- Next: ONCHAIN-METRICS-008 — NUPL

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
