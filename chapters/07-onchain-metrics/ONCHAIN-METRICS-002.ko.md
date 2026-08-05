---
knowledge_id: ONCHAIN-METRICS-002
title: Exchange Reserve
subtitle: 공급 가용성 신호로서의 거래소 보유 물량
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
  - ONCHAIN-METRICS-001
related_topics:
  - Exchange Flows
  - Stablecoin Supply
primary_sources:
  - REF-CM-EXCHSUPPLY-2026-001
tags:
  - exchange-reserve
  - supply
---

# Exchange Reserve
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-002

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-002
title: Exchange Reserve
research_question: >
  Exchange reserve는 무엇을 측정하며, labeled exchange-held balance와 어떻게
  연결되고, 분석가는 supply availability와 방향성 있는 가격 예측을 왜
  분리해야 하는가?
document_type: foundation
difficulty: L300
prerequisites:
  - ONCHAIN-METRICS-001
parent: Onchain Metrics
previous: ONCHAIN-METRICS-001
next: ONCHAIN-METRICS-003
```

## 1. Learning Objectives

- exchange reserve를 정의할 수 있다.
- reserve와 flow의 차이를 설명할 수 있다.
- exchange reserve가 supply-overhang narrative에서 왜 중요할 수 있는지 설명할 수 있다.
- reserve labeling 한계를 식별할 수 있다.

## 2. Executive Summary

exchange reserve metric은 특정 시점에 exchange entity가 보유한 자산 공급량을 추정한다.[^ref-cm-exchsupply]

exchange flow가 일정 기간의 이동을 본다면, exchange reserve는 stock을 본다. 분석가는 종종 reserve 감소를 즉시 사용 가능한 공급 축소의 증거로, reserve 증가를 venue 위 재고 증가의 증거로 해석한다. 이 프레이밍은 유용하지만 labeling quality와 venue structure에 조건부다.

## 3. Core Mechanics

개념적으로 reserve는 다음과 같다.

`Exchange Reserve = Sum of balances across labeled exchange-controlled addresses`

어려운 부분은 계산이 아니라 entity map이다.

## 4. Interpretation

잠재적으로 유의미한 패턴은 다음과 같다.

- 장기 accumulation phase에서의 reserve 감소,
- 강한 distribution 이전의 reserve 증가,
- BTC와 stablecoin 사이 reserve 행태의 divergence.

## 5. Limitations

- omnibus wallet은 최종 사용자 ownership을 가릴 수 있다.
- custodian과 exchange가 겹칠 수 있다.
- cross-chain wrapper는 순 reserve 이해를 복잡하게 만든다.
- off-exchange OTC inventory는 보이지 않는다.

## 6. Institutional Thinking

- reserve는 supply-location metric이지 직접적인 sell-order metric이 아니다.
- reserve는 반드시 flow, price, derivatives positioning과 함께 봐야 한다.

## 7. References

[^ref-cm-exchsupply]: Coin Metrics Docs, exchange supply and entity-balance methodology, accessed 2026-08-04, https://docs.coinmetrics.io/network-data/network-data-overview

## 8. Cross References

- Previous: ONCHAIN-METRICS-001 — Exchange Flows
- Next: ONCHAIN-METRICS-003 — Whale Activity

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
