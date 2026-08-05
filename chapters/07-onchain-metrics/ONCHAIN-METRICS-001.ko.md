---
knowledge_id: ONCHAIN-METRICS-001
title: Exchange Flows
subtitle: 시장 의도를 읽기 위한 렌즈로서의 입출금 추적
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 85 min
estimated_study: 200 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Market Structure
parent:
  - Onchain Metrics
prerequisites:
  - DEFI-003
related_topics:
  - Exchange Reserve
  - Whale Activity
  - Metric Limitations
primary_sources:
  - REF-CM-FLOWIN-2026-001
  - REF-CM-FLOWOUT-2026-001
tags:
  - exchange-flows
  - onchain-metrics
---

# Exchange Flows
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-001

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-001
title: Exchange Flows
research_question: >
  Exchange inflow와 outflow metric은 무엇을 측정하며, labeled exchange entity를
  바탕으로 어떻게 구성되고, flow spike로부터 무엇을 추론할 수 있고 무엇은
  추론할 수 없는가?
document_type: foundation
difficulty: L300
prerequisites:
  - DEFI-003
parent: Onchain Metrics
previous:
next: ONCHAIN-METRICS-002
```

## 1. Learning Objectives

- exchange inflow와 outflow를 정의할 수 있다.
- exchange address labeling 의존성을 설명할 수 있다.
- flow 관측과 trader motive를 구분할 수 있다.
- flow 데이터를 단일 요인 증거가 아니라 맥락 증거로 사용할 수 있다.

## 2. Executive Summary

Coin Metrics는 exchange inflow/outflow 계열 metric을 exchange-labeled entity와 관련된 transfer activity로 문서화한다.[^ref-cm-flowin][^ref-cm-flowout]

분석가는 종종 큰 inflow를 잠재적 매도 압력으로, 큰 outflow를 self-custody 이동 또는 단기 매도 가능 물량 축소로 해석한다. 이 해석은 유용할 수 있지만 보장되지는 않는다. 내부 treasury shuffling, omnibus wallet 관리, venue 간 routing은 모두 표면적 결론을 왜곡할 수 있다.

## 3. Core Mechanics

exchange flow metric은 다음에 의존한다.

- 어떤 주소가 exchange 소속인지 식별하는 일,
- 해당 entity로 들어오고 나가는 transfer를 집계하는 일,
- 내부 entity 이동을 제외할지 통합할지 결정하는 일.

이 metric은 entity labeling 위에 서 있기 때문에, raw transfer volume만큼이나 methodology quality가 중요하다.

## 4. Analytical Use

exchange inflow spike는 보통 다음 국면에서 관찰된다.

- panic condition,
- 급등 후 이익 실현,
- derivative collateral movement.

exchange outflow spike는 보통 다음 국면에서 관찰된다.

- 장기 custody 전환,
- ETF 또는 custodian withdrawal,
- DeFi 또는 staking으로의 이동.

## 5. Metric Limits

- 모든 exchange wallet이 알려져 있지는 않다.
- 알려진 exchange flow 중 일부는 내부 이동이다.
- exchange로의 deposit이 즉각 매도를 증명하지는 않는다.
- withdrawal이 accumulation을 증명하지는 않는다.

## 6. Institutional Thinking

- exchange flow는 action의 직접 증거가 아니라 intent에 인접한 증거로 다뤄야 한다.
- 가격, derivatives, reserve, entity concentration 데이터와 함께 봐야 한다.

## 7. References

[^ref-cm-flowin]: Coin Metrics Docs, exchange inflow metric documentation and transfer methodology, accessed 2026-08-04, https://docs.coinmetrics.io/network-data/network-data-overview

[^ref-cm-flowout]: Coin Metrics Docs, exchange outflow metric documentation and transfer methodology, accessed 2026-08-04, https://docs.coinmetrics.io/network-data/network-data-overview

## 8. Cross References

- Next: ONCHAIN-METRICS-002 — Exchange Reserve

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
