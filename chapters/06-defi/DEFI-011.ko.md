---
knowledge_id: DEFI-011
title: MEV
subtitle: transaction ordering value, searcher 경쟁, 그리고 블록 생산 안에 숨어 있는 시장
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 110 min
estimated_study: 270 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Market Structure
  - Ethereum
parent:
  - DeFi
prerequisites:
  - DEFI-003
  - DEFI-006
related_topics:
  - Liquidation
  - Bridges
  - DeFi Risks
primary_sources:
  - REF-ETH-MEV-2026-001
tags:
  - mev
  - searchers
  - validators
---

# MEV
> DeFi  
> Research Unit: DEFI-011

---

## Research Brief

```yaml
knowledge_id: DEFI-011
title: MEV
research_question: >
  Maximal extractable value란 무엇이며, transaction ordering control이 어떻게
  이를 만들고, 왜 MEV는 execution quality, validator incentive, DeFi protocol
  design을 재구성하는가?
document_type: foundation
difficulty: L400
prerequisites:
  - DEFI-003
  - DEFI-006
parent: DeFi
previous: DEFI-010
next: DEFI-012
```

## 1. Learning Objectives

- MEV를 정확히 정의할 수 있다.
- MEV 공급망에서 searcher와 validator의 역할을 설명할 수 있다.
- MEV를 arbitrage 및 liquidation과 연결할 수 있다.
- centralization 함의를 설명할 수 있다.

## 2. Executive Summary

ethereum.org는 maximal extractable value를, 표준 블록 보상과 gas fee를 넘어 블록 안에서 transaction을 포함·제외·재정렬함으로써 블록 생산에서 추출 가능한 최대 가치로 정의한다.[^ref-eth-mev]

같은 문서는 실제로는 searcher가 기회를 찾고 inclusion을 위해 높은 수수료를 지불하며, validator는 블록 생산을 통제함으로써 그 가치 일부를 가져간다고 설명한다.[^ref-eth-mev]

## 3. Sources of MEV

- DEX arbitrage,
- liquidation,
- sandwiching과 기타 order-flow exploitation,
- 상태 변화 주변의 backrun.

## 4. Economic Structure

searcher opportunity value가 `V_mev`이고 inclusion을 위해 지불한 fee가 `F`라면, 합리적 searcher는 다음 조건 아래 경쟁한다.

`F <= V_mev`

그리고 경쟁이 치열한 경우 `F`는 가용 가치 대부분에 가까워질 수 있다.[^ref-eth-mev]

## 5. Security and Centralization

ethereum.org는 MEV가 PoS 환경에서 proposer economics에 실질적 영향을 미치기 때문에 validator centralization을 가속할 수 있다고 지적한다.[^ref-eth-mev]

## 6. Institutional Thinking

- MEV는 가장자리 현상이 아니라 투명하고 programmable한 시장에 내재된 구조다.
- 많은 DeFi 사용자 경험 문제는 사실 transaction ordering 문제다.

## 7. References

[^ref-eth-mev]: ethereum.org, "Maximal extractable value (MEV)," official documentation, accessed 2026-08-04, https://ethereum.org/developers/docs/mev

## 8. Cross References

- Previous: DEFI-010 — Bridges
- Next: DEFI-012 — DeFi Risks

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
