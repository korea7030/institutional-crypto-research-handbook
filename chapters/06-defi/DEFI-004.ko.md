---
knowledge_id: DEFI-004
title: Lending
subtitle: 공유 유동성 시장에 자본을 공급하고 borrow 수요에서 변동 수익을 얻는 구조
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
  - TOKEN-STANDARDS-001
  - SMART-CONTRACTS-001
related_topics:
  - Borrowing
  - Liquidation
  - Yield Farming
primary_sources:
  - REF-AAVE-SUPPLY-2026-001
  - REF-AAVE-LIQ-2026-001
tags:
  - lending
  - aave
  - defi-credit
---

# Lending
> DeFi  
> Research Unit: DEFI-004

---

## Research Brief

```yaml
knowledge_id: DEFI-004
title: Lending
research_question: >
  DeFi lending은 pool 수준에서 어떻게 작동하며, 공급자는 어떻게 수익을 얻고,
  예치 자본의 위험-수익 프로파일은 무엇에 의해 결정되는가?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-001
  - SMART-CONTRACTS-001
parent: DeFi
previous: DEFI-003
next: DEFI-005
```

## 1. Learning Objectives

- DeFi lending을 정확히 정의할 수 있다.
- 공급자 측 수익 발생 구조를 설명할 수 있다.
- utilization 연동 금리를 설명할 수 있다.
- pooled market에서 lender가 부담하는 리스크를 식별할 수 있다.

## 2. Executive Summary

Aave의 도움말 문서는 사용자가 Aave에 token을 공급하면 이자를 얻을 수 있고, 원한다면 공급한 token을 collateral로 사용할 수도 있다고 설명한다.[^ref-aave-supply]

공급된 자산은 overcollateralized borrowing을 지원하는 공유 liquidity pool로 들어간다.[^ref-aave-supply]

공급 수익은 다음에 달려 있다.

- borrow demand,
- market utilization,
- governance parameter,
- protocol-level risk control.

## 3. Lending Structure

### 3.1 Shared Pool

공급된 자산은 pool smart contract를 통해 차입자에게 제공된다.[^ref-aave-supply]

### 3.2 Rate Formation

Aave는 공급 금리가 borrow utilization rate와 governance parameter에 의해 결정된다고 설명한다.[^ref-aave-supply]

즉 lender 수익은 외부 은행식 정책 데스크가 고정하는 것이 아니라 protocol activity에 내생적으로 연결된다.

## 4. Economic Model

다음을 두자.

- `S` = 총 공급 유동성
- `B` = 총 차입 유동성
- utilization `U = B / S`

utilization이 높아질수록 일반적으로 borrowing rate가 올라가고, 이는 reserve factor와 protocol 설계를 감안할 때 더 높은 공급 수익으로 이어질 수 있다.

## 5. Risks

- smart-contract failure.
- liquidation이 제대로 작동하지 않을 때의 borrower collateral failure.
- utilization이 매우 높을 때 스트레스 상황에서의 liquidity risk.
- governance 및 oracle risk.

## 6. Institutional Thinking

- DeFi lending은 중앙화된 balance-sheet manager 없이 이루어지는 pooled credit intermediation으로 이해하는 편이 정확하다.
- 공급자 수익은 공짜 carry가 아니라, liquidation에 의존하는 시스템에 자본을 투입한 대가다.

## 7. References

[^ref-aave-supply]: Aave Help, "Supply Tokens," official documentation, accessed 2026-08-04, https://aave.com/help/supplying/supply-tokens

## 8. Cross References

- Previous: DEFI-003 — DEX
- Next: DEFI-005 — Borrowing

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
