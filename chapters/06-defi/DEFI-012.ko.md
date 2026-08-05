---
knowledge_id: DEFI-012
title: DeFi Risks
subtitle: Smart Contract, Oracle, Governance, Liquidity, 그리고 cross-protocol failure mode
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 110 min
estimated_study: 260 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Risk
  - Ethereum
parent:
  - DeFi
prerequisites:
  - DEFI-004
  - DEFI-010
  - DEFI-011
related_topics:
  - Bridges
  - Liquidation
  - Tokenomics
primary_sources:
  - REF-AAVE-RISK-2026-001
  - REF-AAVE-UMBRELLA-2026-001
  - REF-ETH-BRIDGES-2026-001
  - REF-UNI-GLOSSARY-2026-001
tags:
  - defi-risks
  - risk-management
  - smart-contract-risk
---

# DeFi Risks
> DeFi  
> Research Unit: DEFI-012

---

## Research Brief

```yaml
knowledge_id: DEFI-012
title: DeFi Risks
research_question: >
  DeFi의 주요 risk class는 무엇이며, composable protocol 사이에서 어떻게
  상호작용하고, 연구자는 고립된 contract risk와 시스템 전체의 dependency risk를
  어떻게 구분해야 하는가?
document_type: foundation
difficulty: L300
prerequisites:
  - DEFI-004
  - DEFI-010
  - DEFI-011
parent: DeFi
previous: DEFI-011
next:
```

## 1. Learning Objectives

- 주요 DeFi risk type을 분류할 수 있다.
- local risk와 systemic risk를 구분할 수 있다.
- composability risk를 설명할 수 있다.
- yield가 숨겨진 dependency chain의 downstream 결과일 수 있는 이유를 설명할 수 있다.

## 2. Executive Summary

Aave의 governance 및 안내 자료는 smart-contract risk, admin-key risk, oracle risk, market risk와 같은 핵심 DeFi risk class를 식별한다.[^ref-aave-risk]

ethereum.org의 bridge 문서는 wrapped asset와 bridge trust assumption에서 비롯되는 systemic risk를 추가로 보여준다.[^ref-eth-bridges]

Uniswap의 glossary는 impermanent loss를 LP가 겪는 특정 opportunity-cost risk로 강조한다.[^ref-uni-glossary]

이들을 종합하면, DeFi risk는 고립된 contract bug의 체크리스트가 아니라 층위화된 dependency system으로 연구하는 편이 정확하다.

## 3. Major Risk Classes

### 3.1 Smart Contract Risk

코드는 실패할 수 있고, exploit될 수 있으며, dependency와 예상치 못한 방식으로 상호작용할 수 있다.

### 3.2 Oracle Risk

잘못된 가격 입력은 borrowing, liquidation, derivatives settlement를 왜곡한다.[^ref-aave-risk]

### 3.3 Governance and Admin Risk

admin key, guardian, governance capture는 protocol behavior를 빠르게 바꿀 수 있다.[^ref-aave-risk][^ref-aave-umbrella]

### 3.4 Market and Liquidity Risk

얇은 유동성, 변동성 높은 담보, 스트레스 상황의 liquidation은 의도한 risk control을 무너뜨릴 수 있다.

### 3.5 Bridge and Wrapped-Asset Risk

bridged 또는 wrapped asset는 외부 settlement와 validation risk를 시스템 안으로 가져온다.[^ref-eth-bridges]

## 4. Composability Risk

하나의 포지션이 동시에 다음에 의존할 수 있다.

- a lending protocol,
- an LP token,
- a bridge,
- an oracle,
- a governance-controlled wrapper.

실패는 어느 한 계층에서 시작해도 전체로 전파될 수 있다.

## 5. Institutional Thinking

- audit 개수만으로는 충분하지 않다.
- risk review는 primary contract만이 아니라 dependency graph를 그려야 한다.
- 높은 수익률은 공짜 효율성이 아니라 집중되었거나 층위화된 리스크를 신호하는 경우가 많다.

## 6. References

[^ref-aave-risk]: Aave Governance, "Asset Management Guidelines" and related risk discussion, accessed 2026-08-04, https://governance.aave.com/t/aave-asset-management-guidelines/5600

[^ref-aave-umbrella]: Aave Help, "Umbrella," official documentation, accessed 2026-08-04, https://aave.com/help/umbrella/umbrella

[^ref-eth-bridges]: ethereum.org, "Bridges," official documentation, page last update 2026-04-03, accessed 2026-08-04, https://ethereum.org/developers/docs/bridges

[^ref-uni-glossary]: Uniswap Developers, "Glossary," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/get-started/concepts/glossary

## 7. Cross References

- Previous: DEFI-011 — MEV
- Phase 6 complete

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
