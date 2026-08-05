---
knowledge_id: DEFI-005
title: Borrowing
subtitle: 초과담보 온체인 신용, health factor 관리, 그리고 프로그래머블 레버리지
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 100 min
estimated_study: 240 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Credit
  - Ethereum
parent:
  - DeFi
prerequisites:
  - DEFI-004
related_topics:
  - Liquidation
  - Staking
  - DeFi Risks
primary_sources:
  - REF-AAVE-LIQ-2026-001
  - REF-AAVE-EMODE-2026-001
  - REF-AAVE-ISOLATION-2026-001
tags:
  - borrowing
  - leverage
  - aave
---

# Borrowing
> DeFi  
> Research Unit: DEFI-005

---

## Research Brief

```yaml
knowledge_id: DEFI-005
title: Borrowing
research_question: >
  DeFi pool에서 초과담보 차입은 어떻게 작동하며, 어떤 요소가 borrow capacity를
  제한하고, 분석가는 health factor를 정적인 비율이 아니라 동적으로 움직이는
  지급능력 buffer로 어떻게 이해해야 하는가?
document_type: foundation
difficulty: L300
prerequisites:
  - DEFI-004
parent: DeFi
previous: DEFI-004
next: DEFI-006
```

## 1. Learning Objectives

- 초과담보 차입을 정의할 수 있다.
- health factor를 설명할 수 있다.
- E-mode와 isolation mode를 리스크 특화 차입 설정으로 설명할 수 있다.
- borrow loop를 통해 레버리지가 어떻게 생기는지 식별할 수 있다.

## 2. Executive Summary

Aave는 차입 가능 규모가 collateral value, liquidation threshold, protocol parameter에 의해 결정되며, 이는 health factor 지표로 요약된다고 설명한다.[^ref-aave-liq][^ref-aave-emode]

주요 DeFi 대출 시장에서 차입은 보통 초과담보 방식이다. 차입자는 빌리는 금액보다 더 큰 가치를 가진 담보를 예치하고, 프로토콜은 포지션 안전성을 지속적으로 모니터링한다.

## 3. Core Mechanics

### 3.1 Borrow Capacity

borrow capacity는 담보 가치와 governance가 정한 risk parameter에 의해 결정된다.

### 3.2 Health Factor

Aave는 health factor를 다음과 같이 정의한다.

`(Total Collateral Value * Weighted Average Liquidation Threshold) / Total Borrow Value`[^ref-aave-liq]

이 값이 `1` 아래로 내려가면 해당 포지션은 liquidation 위험에 놓인다.[^ref-aave-liq]

### 3.3 Specialized Borrow Regimes

Aave의 E-mode는 특정 자산 카테고리에 대해 최적화된 LTV, liquidation-threshold, liquidation-bonus 설정을 통해 더 높은 효율을 허용한다.[^ref-aave-emode]

Isolation mode는 일부 변동성 높은 자산이 collateral로 사용되는 방식을 제한하고, approved stablecoin에 대해서만 차입을 허용한다.[^ref-aave-isolation]

## 4. Economic Implications

차입은 다음을 가능하게 한다.

- leverage,
- 현물 노출을 팔지 않고도 유동성 확보,
- arbitrage 또는 carry trade 자금 조달,
- 재귀적 collateral loop.

## 5. Risks

- collateral price decline,
- debt asset appreciation,
- correlation breakdown,
- parameter change,
- liquidation competition.

## 6. Institutional Thinking

- health factor는 정적인 안심 지표가 아니라 시장 변동성에 맞서 움직이는 buffer다.
- borrow demand와 leverage는 대개 liquidation 의존성이 가장 높은 구간에서 가장 강하게 나타난다.

## 7. References

[^ref-aave-liq]: Aave Docs, "Liquidations," official documentation, accessed 2026-08-04, https://aave.com/docs/developers/liquidations

[^ref-aave-emode]: Aave Help, "Efficiency Mode (E-mode)," official documentation, accessed 2026-08-04, https://aave.com/help/borrowing/e-mode

[^ref-aave-isolation]: Aave Help, "Isolation Mode," official documentation, accessed 2026-08-04, https://aave.com/help/supplying/isolation-mode

## 8. Cross References

- Previous: DEFI-004 — Lending
- Next: DEFI-006 — Liquidation

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
