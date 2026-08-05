---
knowledge_id: ONCHAIN-METRICS-006
title: SOPR
subtitle: 실현 손익을 읽는 렌즈로서의 Spent Output Profit Ratio
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
  - BITCOIN-014
related_topics:
  - MVRV
  - NUPL
primary_sources:
  - REF-GN-SOPR-2026-001
tags:
  - sopr
  - pnl
---

# SOPR
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-006

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-006
title: SOPR
research_question: >
  SOPR은 사용된 코인의 실현 profit/loss에 대해 무엇을 측정하며, 왜 1.0
  threshold가 자주 행동적 regime marker로 취급되는가?
document_type: foundation
difficulty: L400
prerequisites:
  - BITCOIN-014
parent: Onchain Metrics
previous: ONCHAIN-METRICS-005
next: ONCHAIN-METRICS-007
```

## 1. Learning Objectives

- SOPR을 정의할 수 있다.
- SOPR이 1보다 크거나 작은 것이 왜 중요한지 설명할 수 있다.
- realized profitability와 unrealized profitability를 구분할 수 있다.

## 2. Executive Summary

Glassnode는 SOPR을 사용된 output의 USD 실현 가치와 생성 시점 가치의 비율로 정의하며, 이는 사용된 코인이 profit 상태로 이동하는지 loss 상태로 이동하는지를 포착한다.[^ref-gn-sopr]

SOPR이 `1`보다 크면 사용된 코인들은 집계 기준으로 이익을 실현하고 있다. `1`보다 작으면 집계 기준으로 손실을 실현하고 있다.[^ref-gn-sopr]

## 3. Formula

개념적으로:

`SOPR = Realized Value / Value at Creation`

이며, 이는 해당 구간에 사용된 output 전체에 대해 계산된다.[^ref-gn-sopr]

## 4. Interpretation

분석가는 종종 `1.0`을 심리적·구조적 threshold로 본다.

- `1` above: holder가 이익 실현 가능,
- `1` below: realized loss-taking 우세.

## 5. Limits

- SOPR은 spent output만 본다.
- 움직이지 않은 코인에 대해 직접 말해주지 않는다.
- exchange 및 consolidation activity는 짧은 구간을 오염시킬 수 있다.

## 6. Institutional Thinking

- SOPR은 trend, holder segmentation, macro positioning과 결합될 때 가장 강하다.

## 7. References

[^ref-gn-sopr]: Glassnode Docs, "SOPR" metric guide, accessed 2026-08-04, https://docs.glassnode.com/guides-and-tutorials/metric-guides/sopr

## 8. Cross References

- Previous: ONCHAIN-METRICS-005 — Active Addresses
- Next: ONCHAIN-METRICS-007 — MVRV

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
