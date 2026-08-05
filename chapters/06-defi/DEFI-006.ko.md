---
knowledge_id: DEFI-006
title: Liquidation
subtitle: Permissionless 포지션 정리, 담보 압류, 그리고 초과담보 DeFi 신용의 안전 밸브
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 105 min
estimated_study: 250 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Risk
  - Ethereum
parent:
  - DeFi
prerequisites:
  - DEFI-005
related_topics:
  - MEV
  - DeFi Risks
  - Lending
primary_sources:
  - REF-AAVE-LIQ-2026-001
  - REF-AAVE-HFHELP-2026-001
tags:
  - liquidation
  - risk
  - aave
---

# Liquidation
> DeFi  
> Research Unit: DEFI-006

---

## Research Brief

```yaml
knowledge_id: DEFI-006
title: Liquidation
research_question: >
  DeFi liquidation은 초과담보 대출 시장의 지급능력을 어떻게 보전하며, 어떤
  경제적 incentive가 이를 작동하게 만들고, 왜 gas market과 MEV 경쟁에
  긴밀하게 연결되는가?
document_type: foundation
difficulty: L300
prerequisites:
  - DEFI-005
parent: DeFi
previous: DEFI-005
next: DEFI-007
```

## 1. Learning Objectives

- DeFi 맥락에서 liquidation을 정의할 수 있다.
- close factor와 liquidation bonus를 설명할 수 있다.
- liquidation이 permissionless하고 경쟁적인 이유를 설명할 수 있다.
- liquidation execution을 MEV dynamics와 연결할 수 있다.

## 2. Executive Summary

Aave는 borrower의 health factor가 1 아래로 떨어질 때 liquidation이 발생하며, 누구나 부채 일부를 상환하고 할인된 담보를 받을 수 있다고 문서화한다.[^ref-aave-liq]

liquidation은 예외적 사건이 아니다. 그것은 초과담보 대출을 작동시키는 집행 계층이다.

## 3. Core Mechanics

### 3.1 Trigger Condition

Aave는 health factor가 `1` 아래로 내려가면 liquidation 위험이 발생한다고 말한다.[^ref-aave-liq][^ref-aave-hf]

### 3.2 Permissionless Execution

liquidation은 permissionless하다. 즉, 자격을 갖춘 포지션에 대해 누구나 과정을 시작할 수 있다.[^ref-aave-liq]

### 3.3 Incentive

liquidator는 부채를 상환하고 보너스가 붙은 담보를 받는다. 이로써 외부 행위자가 시스템의 지급능력을 유지할 경제적 이유가 생긴다.[^ref-aave-liq][^ref-aave-hf]

## 4. Key Parameters

- health factor,
- liquidation threshold,
- close factor,
- liquidation bonus,
- protocol oracle의 asset price.

## 5. Economic Model

상환된 부채가 `D`, liquidation bonus가 `b`라면, 압류되는 담보 가치는 대략 다음과 같다.

`Collateral value transferred ≈ D * (1 + b)`

이는 oracle 환산 가격과 protocol rule의 영향을 받는다.

## 6. Competition and MEV

Aave는 liquidation이 매우 경쟁적이며, 종종 자동화된 bot이 필요하다고 명시한다.[^ref-aave-liq][^ref-aave-hf]

이는 liquidation을 다음과 연결한다.

- transaction-priority bidding,
- searcher infrastructure,
- MEV extraction.

## 7. Institutional Thinking

- liquidation efficiency는 DeFi credit market의 핵심 지급능력 변수다.
- 표면적으로는 담보가 충분해 보여도, 변동성 국면에서 liquidation throughput이 실패하면 시장은 빠르게 취약해진다.

## 8. References

[^ref-aave-liq]: Aave Docs, "Liquidations," official documentation, accessed 2026-08-04, https://aave.com/docs/developers/liquidations

[^ref-aave-hf]: Aave Help, "Health Factor & Liquidations," official documentation, accessed 2026-08-04, https://aave.com/help/borrowing/liquidations

## 9. Cross References

- Previous: DEFI-005 — Borrowing
- Next: DEFI-007 — Yield Farming

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
