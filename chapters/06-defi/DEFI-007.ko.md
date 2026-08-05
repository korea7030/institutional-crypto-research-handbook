---
knowledge_id: DEFI-007
title: Yield Farming
subtitle: incentive에 의해 유도되는 자본 이동, liquidity mining, 그리고 기본 프로토콜 수익 위에 얹히는 reward-layer economics
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 95 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Incentives
  - Ethereum
parent:
  - DeFi
prerequisites:
  - DEFI-002
  - DEFI-004
related_topics:
  - Staking
  - Tokenomics
  - DeFi Risks
primary_sources:
  - REF-UNI-LM-2026-001
  - REF-AAVE-SAFETY-INC-2026-001
tags:
  - yield-farming
  - liquidity-mining
  - incentives
---

# Yield Farming
> DeFi  
> Research Unit: DEFI-007

---

## Research Brief

```yaml
knowledge_id: DEFI-007
title: Yield Farming
research_question: >
  Yield farming은 base DeFi position 위에 incentive reward를 어떻게 얹으며,
  어떤 행동을 보조금으로 유도하고, 왜 incentive-driven liquidity는 일시적이고
  반사적이며 운영적으로 취약할 수 있는가?
document_type: foundation
difficulty: L300
prerequisites:
  - DEFI-002
  - DEFI-004
parent: DeFi
previous: DEFI-006
next: DEFI-008
```

## 1. Learning Objectives

- yield farming을 정확히 정의할 수 있다.
- base yield와 subsidy yield를 구분할 수 있다.
- liquidity mining을 설명할 수 있다.
- 반사적인 incentive risk를 식별할 수 있다.

## 2. Executive Summary

Uniswap의 liquidity-mining 문서는 reward token, target pool, time window를 설정해 in-range liquidity provision을 유도하는 외부 reward program을 설명한다.[^ref-uni-lm]

Aave의 safety-incentives 문서는 Safety Module에서 slashing risk를 감수하는 staker에게 지급되는 reward를 설명하며, Umbrella에서는 동적인 reward logic이 적용된다고 말한다.[^ref-aave-safety]

이 둘을 함께 보면 yield farming은 하나의 단일 프로토콜 primitive가 아니다. 그것은 다음과 같은 base position 위에 추가되는 incentive layer다.

- LP capital,
- 공급된 lending liquidity,
- 또는 staking된 insurance capital.

## 3. Mechanism

base yield는 다음에서 나올 수 있다.

- swap fee,
- borrow interest,
- staking reward.

farm yield는 특정 행동을 유도하기 위한 추가 token compensation이다.

## 4. Economic Model

관측되는 총 수익은 대략 다음처럼 표현할 수 있다.

`Y_total = Y_base + Y_incentive`

만약 `Y_incentive`가 지배적이라면, incentive가 줄어들 때 liquidity가 빠져나갈 수 있다.

## 5. Risks

- mercenary capital,
- token-price dependence,
- 숨겨진 lockup 또는 slashing exposure,
- 프로그램 종료 시 급격한 APY 붕괴.

## 6. Institutional Thinking

- incentive APR은 지속 가능한 protocol revenue와 동일하지 않다.
- farming은 초기 activity를 부트스트랩할 수 있지만, 약한 organic demand를 가리는 역할도 할 수 있다.

## 7. References

[^ref-uni-lm]: Uniswap Developers, "Liquidity Mining," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/protocols/v3/concepts/liquidity-mining

[^ref-aave-safety]: Aave Help, "Safety Incentives," official documentation, accessed 2026-08-04, https://aave.com/help/umbrella/safety-incentives

## 8. Cross References

- Previous: DEFI-006 — Liquidation
- Next: DEFI-008 — Staking

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
