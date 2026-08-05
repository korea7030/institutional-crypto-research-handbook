---
knowledge_id: ONCHAIN-METRICS-011
title: Realized Cap
subtitle: 마지막 온체인 이동을 기준으로 한 cost-basis 중심 가치평가
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 90 min
estimated_study: 210 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Valuation
parent:
  - Onchain Metrics
prerequisites:
  - ONCHAIN-METRICS-010
related_topics:
  - MVRV
  - NUPL
primary_sources:
  - REF-GN-REALCAP-2026-001
tags:
  - realized-cap
  - valuation
---

# Realized Cap
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-011

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-011
title: Realized Cap
research_question: >
  Realized capitalization이란 무엇이며, 왜 market cap과 다르고, 온체인
  valuation 작업에서 aggregate cost-basis proxy로 어떻게 기능하는가?
document_type: foundation
difficulty: L400
prerequisites:
  - ONCHAIN-METRICS-010
parent: Onchain Metrics
previous: ONCHAIN-METRICS-010
next: ONCHAIN-METRICS-012
```

## 1. Learning Objectives

- realized cap을 정의할 수 있다.
- realized cap과 market cap을 구분할 수 있다.
- realized cap이 여러 온체인 valuation metric의 기반이 되는 이유를 설명할 수 있다.

## 2. Executive Summary

Glassnode는 realized capitalization을 각 코인을 현재 spot price가 아니라 마지막으로 온체인에서 이동했을 때의 가격으로 평가하는 방식으로 문서화한다.[^ref-gn-realcap]

이 방식은 네트워크를 오늘의 mark-to-market 총합이 아니라, UTXO history 안에 내재된 aggregate cost-basis 추정치로 다시 본다는 의미다.

## 3. Why It Matters

realized cap은 다음의 기반 계층이다.

- MVRV,
- NUPL,
- cost-basis 스타일 valuation 작업,
- market-cycle stress analysis.

## 4. Limits

- 오프체인 ownership transfer는 보이지 않는다.
- wrapped 및 custodial 구조는 경제적 ownership을 흐릴 수 있다.
- 마지막 온체인 이동이 항상 실제 acquisition cost는 아니다.

## 5. Institutional Thinking

- realized cap은 blockchain data와 behavioral valuation을 연결하는 가장 유용한 다리 중 하나지만, 여전히 proxy다.

## 6. References

[^ref-gn-realcap]: Glassnode Docs, "Realized Cap" metric guide, accessed 2026-08-04, https://docs.glassnode.com/guides-and-tutorials/metric-guides/realized-cap

## 7. Cross References

- Previous: ONCHAIN-METRICS-010 — Coin Days Destroyed
- Next: ONCHAIN-METRICS-012 — Stablecoin Supply

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
