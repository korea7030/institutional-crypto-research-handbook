---
knowledge_id: DEFI-002
title: Liquidity Pools
subtitle: 공유 온체인 inventory, LP 포지션, 그리고 AMM 거래를 떠받치는 자본 기반
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 95 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Markets
  - Ethereum
parent:
  - DeFi
prerequisites:
  - DEFI-001
related_topics:
  - DEX
  - Yield Farming
  - DeFi Risks
primary_sources:
  - REF-UNI-HOW-2026-001
  - REF-UNI-LIQ-2026-001
  - REF-UNI-CONC-2026-001
tags:
  - liquidity-pools
  - lp
  - uniswap
---

# Liquidity Pools
> DeFi  
> Research Unit: DEFI-002

---

## Research Brief

```yaml
knowledge_id: DEFI-002
title: Liquidity Pools
research_question: >
  Liquidity pool이란 정확히 무엇이며, protocol 버전에 따라 LP claim은 어떻게
  표현되고, 유동성 공급자는 inventory를 AMM에 제공함으로써 어떤 위험-수익
  tradeoff를 부담하는가?
document_type: foundation
difficulty: L300
prerequisites:
  - DEFI-001
parent: DeFi
previous: DEFI-001
next: DEFI-003
related_topics:
  - DEX
  - Yield Farming
  - DeFi Risks
```

## 1. Learning Objectives

- liquidity pool을 정확히 정의할 수 있다.
- LP 자본과 거래량을 구분할 수 있다.
- LP fee accrual을 설명할 수 있다.
- concentrated-liquidity position을 설명할 수 있다.
- impermanent-loss 노출을 식별할 수 있다.

## 2. Executive Summary

Uniswap은 liquidity pool을 트레이더가 swap하는 대상이 되는 두 ERC-20 token reserve로 설명한다.[^ref-uni-how]

유동성 공급자(liquidity provider)는 이 pool에 자산을 deposit하고 trade flow에서 발생하는 수수료를 얻는다. Uniswap은 protocol version에 따라 LP share 표현 방식이 다르다고 설명한다.

- v2는 대체가능 ERC-20 LP token을 mint하고,
- v3는 포지션을 ERC-721 NFT로 표현하며,
- v4는 포지션 관리 주위에 새로운 internal accounting structure를 사용한다.[^ref-uni-how]

Concentrated liquidity는 공급자가 전체 구간 `(0, infinity)`에 자본을 흩뿌리는 대신 특정 가격 구간에 자본을 배치하게 만들어 LP economics를 바꾼다.[^ref-uni-conc]

## 3. Pool Structure

### 3.1 Shared Inventory

pool은 swap을 위한 inventory base다. 트레이더는 한 자산을 넣고 다른 자산을 빼내며, reserve 변화에 따라 가격이 이동한다.

### 3.2 LP Claims

Uniswap은 protocol version에 따라 LP claim 구조가 다르다고 설명한다.[^ref-uni-how]

이 점은 분석적으로 중요하다. LP position이 항상 동질적인 것은 아니기 때문이다.

- 대체가능 share token은 pro-rata pool ownership을 의미하고,
- NFT 형태 포지션은 개별화된 fee 및 range exposure를 의미한다.

### 3.3 Fee Accrual

Uniswap의 liquidity 문서는 liquidity provider가 자신의 포지션 비율에 따라 swap fee를 나누어 가진다고 설명한다.[^ref-uni-liq]

## 4. Concentrated Liquidity

Uniswap은 concentrated liquidity를 custom price range 안에 자본을 배치하는 구조로 문서화하며, 이는 capital efficiency를 높이지만 더 적극적인 관리가 필요하다고 설명한다.[^ref-uni-conc]

가격이 선택된 구간 밖으로 벗어나면 해당 포지션은 inactive가 되어, 가격이 다시 구간 안으로 들어오기 전까지 수수료를 벌지 못한다.[^ref-uni-conc]

## 5. Economic Model

단순한 pro-rata pool의 경우:

- LP share `s = lp_units / total_lp_units`
- reserve X에 대한 claim = `s * x`
- reserve Y에 대한 claim = `s * y`

concentrated-liquidity system에서는 fee와 inventory 결과가 다음에 의존한다.

- 선택한 lower/upper bound,
- range 안에 머문 시간,
- 실제 trade flow.

## 6. Risks

Uniswap의 glossary는 impermanent loss를, 토큰 가격 변화로 인해 LP가 단순 보유와 비교해 겪는 opportunity cost라고 정의한다.[^ref-uni-glossary]

LP의 주요 리스크는 다음과 같다.

- impermanent loss,
- active-management 부담,
- smart-contract risk,
- 변동성 대비 얇은 fee income.

## 7. Institutional Thinking

- pool TVL은 특정 가격에서 실제로 사용 가능한 depth와 동일하지 않다.
- concentrated liquidity는 capital efficiency를 높이지만 operational complexity도 키운다.
- LP 수익은 adverse inventory rebalancing을 차감한 순수익 기준으로 평가해야 한다.

## 8. References

[^ref-uni-how]: Uniswap Developers, "How Uniswap Works," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/get-started/concepts/how-uniswap-works

[^ref-uni-liq]: Uniswap Developers, "Liquidity Overview," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/liquidity/overview

[^ref-uni-conc]: Uniswap Developers, "Concentrated Liquidity," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/get-started/concepts/liquidity-providers/concentrated-liquidity

[^ref-uni-glossary]: Uniswap Developers, "Glossary," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/get-started/concepts/glossary

## 9. Cross References

- Previous: DEFI-001 — AMM
- Next: DEFI-003 — DEX

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
