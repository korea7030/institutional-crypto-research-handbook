---
knowledge_id: ONCHAIN-METRICS-004
title: Smart Money
subtitle: label된 정교한 wallet, signal extraction, 그리고 survivorship bias
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 85 min
estimated_study: 200 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Entity Analysis
parent:
  - Onchain Metrics
prerequisites:
  - ONCHAIN-METRICS-003
related_topics:
  - Whale Activity
  - Metric Limitations
primary_sources:
  - REF-NANSEN-SMARTMONEY-2026-001
tags:
  - smart-money
  - wallet-labels
---

# Smart Money
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-004

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-004
title: Smart Money
research_question: >
  Labeled smart-money metric은 무엇을 포착하려 하며, 이런 label은 어떻게
  구축되고, 분석가가 과거 wallet performance로부터 quality를 추론할 때 어떤
  bias가 생기는가?
document_type: foundation
difficulty: L300
prerequisites:
  - ONCHAIN-METRICS-003
parent: Onchain Metrics
previous: ONCHAIN-METRICS-003
next: ONCHAIN-METRICS-005
```

## 1. Learning Objectives

- smart-money metric을 label-based heuristic으로 정의할 수 있다.
- wallet-quality label이 consensus fact가 아니라 model output인 이유를 설명할 수 있다.
- survivorship bias와 lookback bias를 식별할 수 있다.

## 2. Executive Summary

Nansen은 smart-money 스타일 wallet labeling을, 정교하고 역사적으로 성공적인 참여자를 식별하려는 entity-classification framework의 일부로 문서화한다.[^ref-nansen-smart]

이는 raw address가 원래 익명이기 때문에 유용할 수 있다. 그러나 smart-money dashboard의 품질은 결국 다음에 달려 있다.

- label set,
- performance criteria,
- lookback window,
- exclusion logic.

## 3. Why It Appeals

분석가는 서사가 명확해지기 전에 겉보기에 정보력이 높은 자본이 어디로 이동하는지 알고 싶어 한다.

## 4. Biases

- survivorship bias,
- hindsight classification,
- label leakage,
- strategy-regime change.

## 5. Institutional Thinking

- smart-money label은 ground truth가 아니라 research shortcut이다.
- 이를 lead generation 용도로 사용하되, 직접 wallet과 protocol context를 검증해야 한다.

## 6. References

[^ref-nansen-smart]: Nansen Docs, smart money and wallet-label methodology overview, accessed 2026-08-04, https://docs.nansen.ai

## 7. Cross References

- Previous: ONCHAIN-METRICS-003 — Whale Activity
- Next: ONCHAIN-METRICS-005 — Active Addresses

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
