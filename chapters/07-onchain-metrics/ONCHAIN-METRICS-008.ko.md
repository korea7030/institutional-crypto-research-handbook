---
knowledge_id: ONCHAIN-METRICS-008
title: NUPL
subtitle: 집계 내재 손익을 보여주는 Net Unrealized Profit and Loss
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
  - ONCHAIN-METRICS-007
related_topics:
  - MVRV
  - SOPR
primary_sources:
  - REF-GN-NUPL-2026-001
tags:
  - nupl
  - unrealized-pnl
---

# NUPL
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-008

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-008
title: NUPL
research_question: >
  NUPL은 네트워크에 내재된 aggregate unrealized profit/loss를 무엇으로
  추정하며, 연구자는 heuristic sentiment band를 deterministic signal로
  바꾸지 않으면서 이를 어떻게 사용해야 하는가?
document_type: foundation
difficulty: L400
prerequisites:
  - ONCHAIN-METRICS-007
parent: Onchain Metrics
previous: ONCHAIN-METRICS-007
next: ONCHAIN-METRICS-009
```

## 1. Learning Objectives

- NUPL을 정의할 수 있다.
- realized capitalization 및 market capitalization과의 관계를 설명할 수 있다.
- sentiment band를 신중하게 사용할 수 있다.

## 2. Executive Summary

Glassnode는 NUPL을 relative unrealized profit와 relative unrealized loss의 차이로 정의하며, 흔히 market cap과 realized cap 사이의 정규화된 차이로 단순화해 설명한다.[^ref-gn-nupl]

개념적으로:

`NUPL = (Market Cap - Realized Cap) / Market Cap`[^ref-gn-nupl]

## 3. Interpretation

- positive NUPL: unrealized profit 우세,
- negative NUPL: unrealized loss 우세.

분석가는 종종 NUPL을 cycle band로 나누지만, 이런 band는 설명적 heuristic일 뿐 고정 법칙은 아니다.

## 4. Limits

- 내재 cost basis는 구조적 변화를 늦게 반영할 수 있다.
- sentiment label은 과거 cycle에 과적합될 수 있다.
- regime 지속 기간은 예상보다 길어질 수 있다.

## 5. Institutional Thinking

- NUPL은 하루짜리 신호 생성기가 아니라 regime thermometer로 사용하는 편이 정확하다.

## 6. References

[^ref-gn-nupl]: Glassnode Docs, "NUPL" metric guide, accessed 2026-08-04, https://docs.glassnode.com/guides-and-tutorials/metric-guides/nupl

## 7. Cross References

- Previous: ONCHAIN-METRICS-007 — MVRV
- Next: ONCHAIN-METRICS-009 — Dormancy

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
