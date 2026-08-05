---
knowledge_id: BITCOIN-026
title: Halving
subtitle: height 기반 subsidy step-down, issuance regime transition, miner revenue shift, 그리고 분석 경계
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 110 min
estimated_study: 320 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Economics
  - Monetary Policy
  - Issuance
parent:
  - Bitcoin Economics
prerequisites:
  - BITCOIN-007
  - BITCOIN-020
  - BITCOIN-025
  - POW-009
related_topics:
  - Block Subsidy
  - Monetary Policy
  - Issuance Rate
  - Fee Share
  - Miner Revenue
  - Security Budget
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-BLOCKCHAIN-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-CHAINPARAMS-001
  - REF-BTC-CORE-VALIDATION-TESTS-001
  - REF-BTC-CORE-AMOUNT-001
tags:
  - bitcoin
  - economics
  - halving
  - subsidy
  - issuance
  - miner-revenue
  - security-budget
  - consensus
---

# Halving
> Bitcoin Economics  
> Research Unit: BITCOIN-026

---

## Research Brief

```yaml
knowledge_id: BITCOIN-026
title: Halving
research_question: >
  What is a Bitcoin halving at the consensus layer, how does the block-height
  trigger reduce subsidy by 50 percent per era, how should analysts separate
  protocol certainty from calendar approximation and market interpretation, and
  why does halving matter for miner revenue and long-run security-budget
  analysis?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-007
  - BITCOIN-020
  - BITCOIN-025
  - POW-009
parent: Bitcoin Economics
previous: BITCOIN-025
next: BITCOIN-027
related_topics:
  - Subsidy Schedule
  - Miner Economics
  - Issuance Rate
  - Fee Market
  - Security Budget
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Protocol Structure
  - Technical Mechanics
  - Mathematical or Economic Model
  - Security Assumptions
  - Bitcoin Core Implementation
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Price prediction around specific halving cycles
  - Detailed historical market narratives
  - Altcoin halving comparisons
  - ETF or treasury strategy commentary
  - Full fee-market equilibrium theory
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- halving을 특정 height에서 일어나는 consensus-triggered block subsidy reduction으로 정의할 수 있다.
- halving이 calendar-date 기반이 아니라 block-height 기반인 이유를 설명할 수 있다.
- 특정 block이 어느 subsidy era에 속하는지 계산할 수 있다.
- halving과 total-supply cap, fee market, market-cycle narrative를 구분할 수 있다.
- halving이 수수료를 직접 바꾸지 않더라도 miner revenue composition을 바꾸는 이유를 설명할 수 있다.
- halving이 annualized issuance metric과 security-budget interpretation에 어떤 영향을 주는지 설명할 수 있다.
- halving behavior를 결정하는 Bitcoin Core code surface를 식별할 수 있다.

---

## 2. 핵심 질문

1. Bitcoin halving이란 무엇인가?
2. halving은 합의 규칙상 언제 발생하는가?
3. 사람들이 "4년마다"라고 말하더라도 왜 halving은 wall-clock event가 아닌가?
4. halving height에서 즉시 바뀌는 것은 무엇인가?
5. halving height에서 자동으로 바뀌지 않는 것은 무엇인가?
6. halving은 issuance rate에 어떤 영향을 주는가?
7. halving은 miner revenue composition을 어떻게 바꾸는가?
8. halving 이후 chain data에서 직접 측정할 수 있는 것은 무엇인가?
9. halving이 price나 security에 대해 증명하지 못하는 것은 무엇인가?

---

## 3. Executive Summary

Bitcoin halving은 새로 채굴되는 block의 block subsidy가 미리 정의된 block-height boundary에서 50% 감소하는 사건이다. 이는 consensus rule이지 social convention도, market prediction도 아니다.[^ref-btc-core-validation] [^ref-btc-core-chainparams]

백서는 새로 도입된 coin을 네트워크를 지지하는 node에 분배하는 issuance를 설명하고, 현재 Bitcoin Core 규칙은 이를 declining subsidy schedule로 구현한다. mainnet에서 subsidy-halving interval은 210,000 block이며, completed era가 하나 늘어날 때마다 subsidy는 이전 era의 절반이 된다.[^ref-btc-wp] [^ref-btc-core-validation] [^ref-btc-core-chainparams]

halving에서 바뀌는 것:

- valid block당 new BTC issuance가 50% 감소
- annualized issuance도 대체로 함께 감소
- fee level이 비례해 떨어지지 않는다면 miner revenue에서 fee share가 더 중요해짐

자동으로 바뀌지 않는 것:

- transaction fee rule
- 작업증명 규칙
- 기존 coin balance 총량
- market price
- miner profitability
- 실제 security sufficiency

따라서 halving은 정확한 consensus event이지만, 그 경제적 결과는 여전히 market behavior에 달려 있다.

---

## 4. 프로토콜 구조

### Subsidy-Era Boundary로서의 Halving

halving은 subsidy era의 경계로 이해하는 것이 가장 좋다.

```text
era n subsidy
-> height reaches next interval boundary
-> era n+1 subsidy = era n subsidy / 2
```

mainnet의 interval length는 210,000 block이다.[^ref-btc-core-chainparams]

### Consensus 의미

프로토콜 수준에서 halving은 좁은 한 가지 질문에 답한다.

```text
How much new BTC may a valid block create at this height?
```

이는 miner가 수익을 낼지, price가 반드시 어떻게 움직일지 같은 더 넓은 경제 질문에는 답하지 않는다.

### Height-Based, Not Date-Based

halving은 block height로 trigger되므로, "약 4년마다" 같은 표현은 shorthand일 뿐 exact protocol definition은 아니다. block production은 stochastic하기 때문에 halving의 calendar timing은 사전에 근사적으로만 추정할 수 있다.

---

## 5. Subsidy Transition Mechanics

### Core Rule

Bitcoin Core의 subsidy logic은 다음을 계산한다.

```text
halvings = floor(height / nSubsidyHalvingInterval)
```

그리고 initial subsidy `50 * COIN`에 right shift를 적용하며, 충분히 많은 halving 이후에는 zero-subsidy rule을 적용한다.[^ref-btc-core-validation]

### Mainnet Era Example

| Era | Height Range | Subsidy |
|---|---|---:|
| 0 | `0` to `209999` | 50 BTC |
| 1 | `210000` to `419999` | 25 BTC |
| 2 | `420000` to `629999` | 12.5 BTC |
| 3 | `630000` to `839999` | 6.25 BTC |
| 4 | `840000` to `1049999` | 3.125 BTC |

규칙 관점에서 경계는 즉시 바뀐다. 즉, block `839999`의 valid subsidy와 block `840000`의 valid subsidy는 다르다.[^ref-btc-core-validation] [^ref-btc-core-chainparams]

### Coinbase Constraint

halving boundary에서 최대 coinbase claim도 그에 맞춰 줄어든다.

```text
max coinbase value
= new era subsidy
+ included transaction fees
```

전환 height 이후에도 miner가 pre-halving subsidy를 청구하면 그 block은 consensus-invalid다.[^ref-btc-dev-blockchain] [^ref-btc-core-validation]

---

## 6. Technical Mechanics

### Satoshi Granularity

halving은 integer satoshi-denominated amount 위에서 구현된다. subsidy는 부동소수 macroeconomic variable이 아니라, consensus logic이 반환하는 정확한 integer amount다.[^ref-btc-core-amount] [^ref-btc-core-validation]

### Approximate Calendar Interpretation

"약 4년마다"라는 표현은 다음 계산에서 나온다.

```text
210000 blocks * 10 minutes per target block
≈ 2,100,000 minutes
≈ 1,458.3 days
≈ 3.99 years
```

이는 target-based approximation이지 exact real-time guarantee가 아니다.

### Halving Across What Stays Constant

halving이 바꾸지 않는 것:

- block-header proof-of-work field format
- difficulty-adjustment formula
- subsidy와 무관한 transaction validation rule
- transaction fee의 존재 자체
- 경계 이전에 이미 발행된 amount

halving은 하나의 직접적인 consensus quantity, 즉 block당 newly issued BTC만 바꾼다.

---

## 7. Validation Boundaries

### Halving은 Deterministic하다

block이 pre-halving인지 post-halving인지는 height와 consensus parameter로 객관적으로 결정된다. 어떤 재량 기관이 "이제 halving이다"라고 결정하는 구조가 아니다.

### Market Reaction은 Deterministic하지 않다

consensus는 subsidy step-down은 결정할 수 있지만, 다음은 결정하지 못한다.

- price가 오를지 내릴지
- hash rate가 즉시 오를지 내릴지
- fee level이 miner를 보상할지
- market participant가 이미 이벤트를 pricing-in했는지

### Security-Budget Interpretation

halving은 subsidy를 줄이지만, security는 total miner compensation이 attack cost에 비해 어느 정도인지에 달려 있다. fee revenue와 price condition이 바뀌면 nominal 50% subsidy reduction의 경제적 효과는 크게 달라질 수 있다.

---

## 8. Security Assumptions and Failure Modes

### Revenue Compression Risk

다른 조건이 같다면, halving은 miner revenue 중 newly issued portion을 절반으로 줄인다. fee와 price가 이를 상쇄하지 못하면 marginal miner의 profitability는 악화될 수 있다.

### Adjustment Dynamics

halving 이후 miner participation이 바뀌면, difficulty retargeting과 market adaptation이 일어나기 전까지 block interval이 일시적으로 target에서 벗어날 수 있다. 따라서 halving rule 자체는 단순하지만, mining economics와 operational timing과는 간접적으로 상호작용한다.

### Overinterpretation Risk

halving event 자체가 곧 다음을 증명하는 것은 아니다.

- future scarcity pricing
- mining capitulation
- security collapse
- fee-market maturity

이는 추가 증거가 필요한 가설들이다.

---

## 9. Mathematical or Economic Model

### Subsidy-Era Function

다음을 두자.

- `H` = block height
- `I` = mainnet에서 210000
- `S0` = initial subsidy

그러면:

```text
n = floor(H / I)
S(H) = S0 / 2^n
```

실제 구현은 integer arithmetic와 충분한 halving 이후 zero return으로 이루어진다.[^ref-btc-core-validation]

### Annualized Issuance Approximation

measurement window 동안 era subsidy가 `S`이고 average realized block interval이 `T`분이라면:

```text
blocks_per_year ≈ (365 * 24 * 60) / T
annual_issuance ≈ S * blocks_per_year
```

그래서 halving은 annualized new issuance도 대략 절반으로 줄인다. 그러나 exact yearly figure는 realized block timing과 측정 구간이 height boundary의 어디에 위치하는지에 달려 있다.

### Miner Revenue Mix

다음을 두자.

- `R` = miner gross revenue per block
- `S` = subsidy
- `F` = transaction fee

그러면:

```text
R = S + F
fee_share = F / (S + F)
```

경계에서 `F`가 그대로인데 `S`만 절반이 되면 fee share는 기계적으로 상승한다.

---

## 10. Bitcoin Core 구현

### `validation`

`GetBlockSubsidy`는 halving behavior의 직접적인 구현 surface다. block height와 consensus parameter를 valid subsidy amount로 변환한다.[^ref-btc-core-validation]

### `kernel/chainparams`

mainnet chain parameter는 `nSubsidyHalvingInterval = 210000`을 정의하며, 이를 통해 모든 halving boundary 위치가 결정된다.[^ref-btc-core-chainparams]

### `consensus/amount.h`

`CAmount`와 `COIN`은 subsidy value에 사용되는 integer accounting unit을 정의한다. 이는 halving이 부동소수 decimal number가 아니라 satoshi-denominated integer에 대해 집행된다는 점에서 중요하다.[^ref-btc-core-amount]

### Validation Test

Bitcoin Core는 block-subsidy와 subsidy-limit test를 포함한다. halving rule은 코드로는 짧지만 Bitcoin 공급 일정의 핵심이기 때문에 테스트가 중요하다.[^ref-btc-core-validation-tests]

---

## 11. 온체인 함의

### 직접 관측 가능한 효과

halving 이후 분석가는 다음을 측정할 수 있다.

- block당 subsidy 감소
- block당 gross issuance 감소
- block reward에서 fee share 변화
- 정확한 경계 height 이후의 issuance regime change

### 간접적 또는 조건부 효과

분석가는 다음도 연구할 수 있다.

- hash-rate adjustment
- 이벤트 전후의 block-interval deviation
- miner treasury behavior
- fee sensitivity

하지만 이는 halving rule 자체의 direct output이 아니라 조사 대상인 consequence다.

### Reporting Caution

halving을 보고할 때는 다음을 분리해야 한다.

- block-height event date
- estimated calendar timing
- subsidy per block
- annualized issuance estimate
- fee share
- market reaction

이들은 연관되어 있지만 서로 다른 측정값이다.

---

## 12. Institutional Thinking

halving은 consensus 관점에서는 완전히 예측 가능하지만, 경제적 outcome은 여전히 모호한 몇 안 되는 Bitcoin event 중 하나다.

### Practical Implications

- treasury와 research team은 protocol certainty와 market uncertainty를 분리해야 한다.
- mining analysis는 nominal subsidy drop과 realized fee-share compensation을 함께 추적해야 한다.
- risk system은 approximate halving date를 exact deterministic timestamp처럼 취급하면 안 된다.
- supply dashboard는 calendar date보다 height boundary를 우선 표시하는 편이 낫다.

---

## 13. Common Misinterpretations

### "Halving은 정확히 4년마다 일어난다"

틀렸다. mainnet에서 210,000 block마다 일어나며, 이는 대략 4년일 뿐이다.

### "Halving은 전체 BTC 공급을 절반으로 줄인다"

틀렸다. 기존 outstanding supply가 아니라 block당 신규 issuance를 절반으로 줄인다.

### "Halving이 수수료를 바꾼다"

틀렸다. 바뀌는 것은 subsidy다. fee는 transaction behavior와 market condition에 의해 결정된다.

### "Halving은 Price Increase를 보장한다"

틀렸다. price response는 market outcome이지 consensus rule이 아니다.

### "Halving Alone이 Security를 결정한다"

틀렸다. security는 total miner compensation, attacker cost, broader market condition에 달려 있다.

---

## 14. Research Questions

1. halving boundary 전후에 realized block interval과 hash rate는 얼마나 빨리 조정되는가?
2. post-halving miner revenue stabilization은 price, fee, cost reduction 중 무엇에서 얼마나 오는가?
3. trailing, forward-estimated, era-based 중 기관에 가장 유용한 annualized issuance metric은 무엇인가?
4. halving event study는 anticipation effect와 post-boundary effect를 어떻게 분리해야 하는가?
5. security-budget dashboard는 fee sufficiency를 과장하지 않으면서 fee share를 어떻게 보여 줘야 하는가?

---

## 15. Practical Exercises

### Exercise 1

주어진 block height에서 halving era와 valid subsidy를 계산하라.

### Exercise 2

halving boundary를 포함하는 sample에 대해 다음을 계산하라.

- subsidy per block
- fees per block
- fee share
- annualized issuance approximation

### Exercise 3

두 observer가 halving height에는 동의하면서도, 발생 전 expected calendar date에는 왜 다르게 예측할 수 있는지 설명하라.

### Exercise 4

다른 모든 block field가 유효하더라도, 왜 post-halving block이 previous era subsidy를 청구하면 invalid인지 설명하라.

---

## 16. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Halving interval and subsidy step-down | Directly specified | Bitcoin Core validation and chain params |
| Approximate four-year cadence | Analytical approximation from target spacing | Not the exact consensus rule |
| Fee-share mechanics | Directly specified plus arithmetic inference | Revenue identity plus subsidy change |
| Price and security implications | Inference from sources | Economic interpretation, not protocol guarantee |

---

## 17. Knowledge Graph

```text
Halving
├─ Consensus Trigger
│  ├─ block height
│  ├─ subsidy era
│  └─ 210000-block interval
├─ Direct Effects
│  ├─ lower subsidy
│  ├─ lower issuance
│  └─ new coinbase ceiling
├─ Indirect Effects
│  ├─ fee-share change
│  ├─ miner revenue pressure
│  ├─ hash-rate response
│  └─ annualized inflation change
├─ Implementation
│  ├─ GetBlockSubsidy
│  ├─ chainparams
│  ├─ CAmount
│  └─ subsidy tests
└─ Risks
   ├─ calendar-date confusion
   ├─ price overinterpretation
   └─ security-budget overstatement
```

---

## 18. 참고문헌

### Primary Sources

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," especially Section 6. https://bitcoin.org/bitcoin.pdf
[^ref-btc-dev-blockchain]: Bitcoin Developer Reference, "Block Chain," including coinbase, subsidy, and fee description. https://developer.bitcoin.org/reference/block_chain.html
[^ref-btc-core-validation]: Bitcoin Core Doxygen, `validation.h`, including `GetBlockSubsidy`. https://doxygen.bitcoincore.org/validation_8h_source.html
[^ref-btc-core-chainparams]: Bitcoin Core Doxygen, `kernel/chainparams.cpp`, including mainnet `nSubsidyHalvingInterval = 210000`. https://doxygen.bitcoincore.org/kernel_2chainparams_8cpp_source.html
[^ref-btc-core-validation-tests]: Bitcoin Core Doxygen, `validation_tests.cpp`, including block-subsidy and subsidy-limit tests. https://doxygen.bitcoincore.org/validation__tests_8cpp.html
[^ref-btc-core-amount]: Bitcoin Core Doxygen, `consensus/amount.h`, including `CAmount` and `COIN`. https://doxygen.bitcoincore.org/amount_8h_source.html

### Supporting Interpretation Notes

- Where this document discusses annualized issuance, miner profitability, fee-share significance, or security-budget interpretation, those statements are inferences built from consensus subsidy rules and economic accounting identities rather than explicit protocol guarantees.

---

## 19. 교차 참조

### Previous

- BITCOIN-025 — Bitcoin Monetary Policy

### Next

- BITCOIN-027 — Fee Market

### Related

- BITCOIN-007 — Whitepaper Section 6 — Incentive
- BITCOIN-018 — Transaction Fees
- BITCOIN-020 — Mining
- BITCOIN-025 — Bitcoin Monetary Policy
- BITCOIN-027 — Fee Market
- BITCOIN-028 — Security Budget
- POW-009 — Coinbase Transaction and Block Subsidy

---

## Review Status

### Technical Review

Passed.

- halving을 broader monetary policy, fee market, price narrative와 분리했다.
- height-based trigger logic과 calendar approximation을 구분했다.
- coinbase ceiling과 miner revenue mix를 분리해 설명했다.
- 구현 참조는 subsidy, amount, chain parameter, test로 제한했다.

### Evidence Review

Passed.

- 백서와 developer reference가 issuance framing을 뒷받침한다.
- Core validation과 chain parameter가 halving mechanic을 뒷받침한다.
- amount primitive가 integer-accounting discussion을 뒷받침한다.
- validation test가 구현 검증을 뒷받침한다.
- economic implication은 inference로 표시했다.

### Editorial Review

Passed.

- 구조는 프로젝트 deep-dive format을 따른다.
- metadata가 완전하다.
- halving, subsidy era, issuance, fee share, annualized issuance, consensus trigger 용어가 일관된다.
- table과 code fence가 닫혀 있다.

### Adversarial Review

Passed.

- halving을 exact calendar event로 혼동하지 않았다.
- issuance reduction을 existing supply reduction으로 혼동하지 않았다.
- subsidy change 때문에 fee가 자동으로 변한다고 주장하지 않았다.
- price/security outcome이 halving으로 보장된다고 주장하지 않았다.
- annualized issuance precision을 과장하지 않았다.

### Quality Gate

| Gate | Status |
|---|---|
| Research scope was followed | Pass |
| Required primary sources were reviewed | Pass |
| Source ledger was completed | Pass |
| Claim ledger was completed | Pass |
| Material claims are cited | Pass |
| Fact and interpretation are separated | Pass |
| Consensus and policy are separated | Pass |
| Historical and current behavior are separated | Pass |
| Mathematical examples were verified | Pass |
| Source-code references were verified | Pass |
| Counter evidence is included | Pass |
| Unknowns are acknowledged | Pass |
| Knowledge graph is present | Pass |
| Cross references are valid | Pass |
| No invented sources are present | Pass |
| No unresolved critical review issue remains | Pass |
