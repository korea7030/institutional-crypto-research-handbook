---
knowledge_id: DEFI-001
title: AMM
subtitle: Automated Market Maker, 상수 함수, 그리고 오더북 없이 이루어지는 온체인 가격 형성
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 100 min
estimated_study: 240 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Markets
  - Ethereum
parent:
  - DeFi
prerequisites:
  - TOKEN-STANDARDS-001
  - SMART-CONTRACTS-001
related_topics:
  - Liquidity Pools
  - DEX
  - MEV
primary_sources:
  - REF-UNI-HOW-2026-001
  - REF-UNI-GLOSSARY-2026-001
tags:
  - defi
  - amm
  - uniswap
---

# AMM
> DeFi  
> Research Unit: DEFI-001

---

## Research Brief

```yaml
knowledge_id: DEFI-001
title: AMM
research_question: >
  Automated market maker는 오더북을 어떻게 pool 기반 가격 결정으로 대체하며,
  어떤 invariant logic이 거래를 지배하고, deterministic한 온체인 유동성은
  어떤 execution tradeoff를 만들어내는가?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-001
  - SMART-CONTRACTS-001
parent: DeFi
previous:
next: DEFI-002
related_topics:
  - Liquidity Pools
  - DEX
  - MEV
out_of_scope:
  - Full CFMM taxonomy
  - Formal proof of invariant safety
```

## 1. Learning Objectives

- AMM을 smart-contract 기반 market structure로 정의할 수 있다.
- constant-product pricing을 설명할 수 있다.
- AMM과 order book을 구분할 수 있다.
- slippage와 price impact를 설명할 수 있다.
- AMM이 composable하면서도 path-dependent한 이유를 식별할 수 있다.

## 2. Executive Summary

Uniswap은 자신들의 프로토콜을 order book에 의존하지 않고 누구나 온체인에서 직접 토큰을 swap하고, liquidity를 공급하고, 새로운 market을 만들 수 있게 하는 automated market maker로 설명한다.[^ref-uni-how]

단순한 constant-product 형태에서는 pool reserve `x`와 `y`가 다음 관계를 따른다.

`x * y = k`[^ref-uni-how]

트레이더는 게시된 지정가 주문과 매칭되는 것이 아니라, reserve balance가 바뀌면서 가격이 이동하는 pool을 상대로 거래한다. pool depth에 비해 큰 거래일수록 price impact가 커진다.

분석적으로 보면 AMM은 유동성을 연속적으로 제공하지만, 가격을 size-independent하게 보장하지는 않는다. execution quality는 다음에 달려 있다.

- pool depth,
- fee tier,
- routing,
- transaction ordering.

## 3. Core Mechanics

### 3.1 Pool-Based Trading

Uniswap은 liquidity pool을 두 ERC-20 token의 reserve로 설명하며, 사용자는 이 reserve를 상대로 swap한다고 설명한다.[^ref-uni-how]

### 3.2 Constant Product

초기 Uniswap 버전에서 constant-product invariant는 일반적으로 `x * y = k`로 표현된다.[^ref-uni-how][^ref-uni-glossary]

트레이더가 한쪽 토큰을 pool에 넣고 다른 토큰을 꺼내면, pool state는 이 곡선을 따라 이동한다. 즉 한계 가격은 거래 규모에 따라 내생적으로 결정된다.

### 3.3 Price Impact

거래가 reserve에 비해 작으면 execution은 현재 pool price에 가깝다. 거래가 크면 곡선 더 깊은 지점까지 이동하므로 평균 체결 가격이 나빠진다.

## 4. Mathematical Model

reserve `(x, y)`와 입력 `dx`가 있을 때, 수수료를 무시하면:

- token X의 새로운 reserve: `x' = x + dx`
- token Y의 새로운 reserve는 `x' * y' = k`를 만족해야 한다.
- 따라서 `y' = k / x'`
- 출력 `dy = y - y'`

수수료가 있으면 invariant를 적용하기 전에 유효 입력이 먼저 줄어든다.

## 5. Security and Market Implications

- AMM은 deterministic하고 공개되어 있으므로 route quality와 ordering이 공격 surface가 된다.
- 가격 발견은 pool state와 외부 arbitrage에 의해 내생적으로 이루어진다.
- AMM은 시장 스트레스 중에도 유동성을 유지할 수 있지만, pool이 얕으면 execution quality가 급격히 악화될 수 있다.

## 6. Institutional Thinking

- AMM은 quote board가 아니라 inventory-based market으로 다뤄야 한다.
- pool price는 규모와 무관한 execution을 보장하지 않는다.
- AMM 시장은 broader market과의 정렬을 위해 arbitrage에 의존한다.

## 7. Evidence Classification

| Claim | Classification | Basis |
|---|---|---|
| Uniswap is an AMM instead of an order-book exchange | Directly specified | Uniswap docs |
| Constant product `x*y=k` governs early Uniswap pricing | Directly specified | Uniswap docs |
| Larger trades cause more price impact | Directly specified and inferred | Invariant mechanics |

## 8. References

[^ref-uni-how]: Uniswap Developers, "How Uniswap Works," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/get-started/concepts/how-uniswap-works

[^ref-uni-glossary]: Uniswap Developers, "Glossary," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/get-started/concepts/glossary

## 9. Cross References

- Next: DEFI-002 — Liquidity Pools

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
