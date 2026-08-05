---
knowledge_id: ONCHAIN-METRICS-013
title: TVL
subtitle: 보편적 건강 점수가 아니라 자본 배분 metric으로서의 Total Value Locked
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 85 min
estimated_study: 200 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - DeFi
parent:
  - Onchain Metrics
prerequisites:
  - DEFI-012
related_topics:
  - Stablecoin Supply
  - Metric Limitations
primary_sources:
  - REF-LLAMA-TVL-2026-001
tags:
  - tvl
  - defi
---

# TVL
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-013

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-013
title: TVL
research_question: >
  Total value locked는 실제로 무엇을 집계하며, 자본이 recursive하거나,
  subsidized되어 있거나, price-driven일 때 왜 TVL이 protocol health를
  과대평가할 수 있는가?
document_type: foundation
difficulty: L300
prerequisites:
  - DEFI-012
parent: Onchain Metrics
previous: ONCHAIN-METRICS-012
next: ONCHAIN-METRICS-014
```

## 1. Learning Objectives

- TVL을 정의할 수 있다.
- TVL이 net inflow뿐 아니라 asset-price appreciation으로도 상승하는 이유를 설명할 수 있다.
- recursion과 double-counting risk를 식별할 수 있다.

## 2. Executive Summary

DefiLlama는 TVL을 DeFi protocol과 chain 전반에 보유된 aggregate value로 추적한다.[^ref-llama-tvl]

TVL은 자본 배분과 protocol relevance를 이해하는 데 유용하지만, 보편적 품질 metric처럼 오용되는 경우가 많다.

## 3. Why TVL Can Mislead

- 가격 상승만으로도 신규 자본 없이 TVL이 불어날 수 있다.
- 재귀적 담보 사용은 이중 계산을 만들 수 있다.
- incentive program은 mercenary liquidity를 끌어들일 수 있다.
- bridge-wrapped asset는 동일한 경제적 자본을 여러 층에서 반복 반영할 수 있다.

## 4. Institutional Thinking

- TVL은 stock-of-capital metric이지 profitability metric이 아니다.
- fee, user, retention, risk concentration과 함께 봐야 한다.

## 5. References

[^ref-llama-tvl]: DefiLlama methodology and protocol TVL dashboards, accessed 2026-08-04, https://defillama.com

## 6. Cross References

- Previous: ONCHAIN-METRICS-012 — Stablecoin Supply
- Next: ONCHAIN-METRICS-014 — Network Growth

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
