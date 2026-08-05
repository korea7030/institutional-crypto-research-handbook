---
knowledge_id: BITCOIN-025
title: Bitcoin Monetary Policy
subtitle: 고정 공급 규칙, 블록 보조금 일정, 반감기 메커닉, 발행 한계, 그리고 합의와 경제학의 경계
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 115 min
estimated_study: 340 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Economics
  - Monetary Policy
  - Consensus
parent:
  - Bitcoin Economics
prerequisites:
  - BITCOIN-007
  - BITCOIN-014
  - BITCOIN-020
  - POW-009
related_topics:
  - Block Subsidy
  - Halving
  - Supply Cap
  - Issuance
  - Security Budget
  - MoneyRange
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-BLOCKCHAIN-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-AMOUNT-001
  - REF-BTC-CORE-CHAINPARAMS-001
  - REF-BTC-CORE-VALIDATION-TESTS-001
tags:
  - bitcoin
  - economics
  - monetary-policy
  - subsidy
  - halving
  - issuance
  - supply-cap
  - consensus
---

# Bitcoin Monetary Policy
> Bitcoin Economics  
> Research Unit: BITCOIN-025

---

## Research Brief

```yaml
knowledge_id: BITCOIN-025
title: Bitcoin Monetary Policy
research_question: >
  What does Bitcoin's monetary policy mean at the consensus layer, how do block
  subsidy and halving rules determine issuance over time, what role do
  `MAX_MONEY` and amount bounds play, and where is the line between protocol
  supply rules and economic interpretation such as price, inflation, and
  security-budget sufficiency?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-007
  - BITCOIN-014
  - BITCOIN-020
  - POW-009
parent: Bitcoin Economics
previous: BITCOIN-024
next: BITCOIN-026
related_topics:
  - Subsidy Schedule
  - Halving
  - Issuance Rate
  - Security Budget
  - Supply Accounting
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
  - Macroeconomic comparisons to fiat systems
  - Price forecasting
  - Detailed fee-market equilibrium analysis
  - Regulatory treatment of Bitcoin as money
  - Monetary policy debates for non-Bitcoin chains
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Bitcoin monetary policy를 market outcome이 아니라 합의가 지배하는 issuance rule로 정의할 수 있다.
- subsidy issuance와 fee transfer를 구분할 수 있다.
- mainnet subsidy-halving interval과 그 중요성을 설명할 수 있다.
- 2,100만 BTC라는 수치가 subsidy schedule과 amount granularity의 점근적 결과라는 점을 설명할 수 있다.
- `MAX_MONEY` range check와 per-block subsidy rule을 구분할 수 있다.
- Bitcoin monetary policy 중 hard protocol constraint와 economic interpretation가 각각 무엇인지 설명할 수 있다.
- subsidy decline을 long-run security-budget question과 연결하되, consensus가 보장하는 범위를 과장하지 않을 수 있다.

---

## 2. 핵심 질문

1. Bitcoin monetary policy란 무엇인가?
2. miner revenue 중 어떤 부분이 새로운 BTC를 만들고, 어떤 부분이 기존 BTC를 옮길 뿐인가?
3. Bitcoin은 특정 height에서 block subsidy를 어떻게 결정하는가?
4. 왜 issuance는 시간이 갈수록 감소하는가?
5. 왜 total supply cap은 보통 2,100만 BTC라고 설명되는가?
6. `MAX_MONEY`는 무엇을 하고, 무엇을 하지 않는가?
7. Bitcoin inflation rate는 시간 기준으로 고정된 것인가, 아니면 block schedule 기준으로만 고정된 것인가?
8. subsidy가 감소하면 경제적으로 무엇이 바뀌는가?
9. consensus만으로 알 수 있는 것은 무엇이고, price나 demand에 달린 것은 무엇인가?

---

## 3. Executive Summary

Bitcoin monetary policy는 새 bitcoin 발행을 지배하는 프로토콜 규칙 집합이며, 특히 block subsidy schedule을 뜻한다. 이는 Bitcoin의 market price, purchasing power, fee level, realized security budget과 동일하지 않다. 그런 것들은 consensus issuance rule 위에서 형성되는 경제적 outcome이다.[^ref-btc-wp] [^ref-btc-core-validation]

백서는 새로 도입된 coin을 네트워크를 지지하는 node에 분배하는 reward mechanism을 설명하고, 현재 Bitcoin Core 규칙은 영구 고정 subsidy가 아니라 감소하는 block subsidy를 통해 이 issuance를 구현한다.[^ref-btc-wp] [^ref-btc-core-validation] [^ref-btc-core-chainparams]

Bitcoin mainnet에서는:

- subsidy가 `50 * COIN`에서 시작하고,
- subsidy halving interval은 `210000` block이며,
- 각 halving epoch가 끝날 때마다 subsidy는 right shift로 감소하고,
- 충분히 많은 halving 이후 subsidy는 integer granularity와 Core의 explicit handling 때문에 0이 된다.[^ref-btc-core-validation] [^ref-btc-core-chainparams]

널리 인용되는 2,100만 BTC cap은 이 issuance schedule과 satoshi 단위 integer accounting이 결합된 결과다. Bitcoin Core는 `MAX_MONEY = 21000000 * COIN`과 `MoneyRange`도 정의하지만, 이것이 발행 일정 전체를 대체하지는 않는다. 이는 amount-validity guard이지, per-block reward enforcement의 대체물이 아니다.[^ref-btc-core-amount]

분석가와 기관에게 중요한 구분은 다음이다.

- consensus는 valid block이 새로 발행할 수 있는 BTC 양을 결정한다.
- market은 그 issuance의 가치를 결정한다.
- miner는 subsidy와 fee를 함께 본다.
- 장기 security adequacy는 프로토콜이 보장하는 수치가 아니라 경제적 질문이다.

---

## 4. 프로토콜 구조

### 합의 계층의 Monetary Policy

Bitcoin에서 monetary policy는 주로 issuance constraint를 뜻한다.

- 누가 새 BTC를 만들 수 있는가: valid block의 valid coinbase transaction만 가능
- 블록당 얼마를 만들 수 있는가: 해당 height의 subsidy + 수집한 fee
- issuance가 시간에 따라 어떻게 바뀌는가: halving schedule
- 일반적인 amount-range validity constraint는 무엇인가[^ref-btc-core-validation] [^ref-btc-core-amount]

### Issuance vs Transfer

miner revenue의 두 구성 요소는 분리해서 봐야 한다.

| Component | Economic Meaning | New BTC Created? |
|---|---|---|
| Block subsidy | 프로토콜 발행 | Yes |
| Transaction fees | spender에서 miner로의 재분배 | No |

이 구분은 기초적이다. fee는 total BTC supply를 늘리지 않는다. subsidy만이 늘린다.

### Rule Shape

상위 수준에서는:

```text
max coinbase claim per block
= subsidy at block height
+ sum of included transaction fees
```

이 상한을 넘으면 consensus-invalid다.[^ref-btc-dev-blockchain] [^ref-btc-core-validation]

---

## 5. Subsidy Schedule

### Mainnet Halving Interval

Bitcoin Core의 mainnet consensus parameter는 다음과 같다.

```text
nSubsidyHalvingInterval = 210000
```

즉, 각 subsidy era는 210,000 block 동안 지속된다.[^ref-btc-core-chainparams]

### 기본 공식

Bitcoin Core는 `GetBlockSubsidy(height, consensusParams)`를 제공한다. 구현은 completed halving을 halving interval로 나눈 integer division으로 계산한 뒤, initial subsidy를 그 횟수만큼 right-shift하며, halving 횟수가 충분히 크면 zero를 반환한다.[^ref-btc-core-validation]

개념적으로는:

```text
halvings = floor(height / 210000)
if halvings >= 64:
    subsidy = 0
else:
    subsidy = 50 BTC >> halvings
```

이는 wall-clock schedule이 아니라 block-height schedule이다.

### Era 직관

선택한 mainnet era는 다음과 같다.

| Era | Height Range | Subsidy |
|---|---|---:|
| 0 | `0` to `209999` | 50 BTC |
| 1 | `210000` to `419999` | 25 BTC |
| 2 | `420000` to `629999` | 12.5 BTC |
| 3 | `630000` to `839999` | 6.25 BTC |
| 4 | `840000` to `1049999` | 3.125 BTC |

이 값들은 halving rule과 satoshi precision의 직접적인 결과다.[^ref-btc-core-validation] [^ref-btc-core-chainparams]

---

## 6. Supply Cap과 Amount Bound

### 왜 "21 Million"인가

subsidy series는 유한 정밀도의 기하급수적 감소다.

```text
50 + 25 + 12.5 + 6.25 + ...
```

이를 block당으로 보고 era마다 `210000` block씩 반복하면, 실수 계산에서는 무한 급수의 합이 100 BTC per-block-family scale로 수렴하고, 여기에 era length를 곱하면 2,100만 BTC가 된다. Bitcoin에서는 issuance가 satoshi로 이산화되어 있으므로, 실제 결과는 추상적 연속 모델이 아니라 integer issuance rule에 의해 구현되는 점근적 cap이다.[^ref-btc-core-validation] [^ref-btc-core-validation-tests]

### `MAX_MONEY`

Bitcoin Core는 다음을 정의한다.

```text
COIN = 100000000 satoshis
MAX_MONEY = 21000000 * COIN
```

그리고 `MoneyRange(n)`는 amount가 non-negative이고 `MAX_MONEY`보다 크지 않은지 검사한다.[^ref-btc-core-amount]

### `MAX_MONEY`가 아닌 것

`MAX_MONEY`가 monetary policy 전체는 아니다.

이는 다음에 답하지 못한다.

- height `h`에서 subsidy가 얼마인가
- coinbase가 fee + subsidy를 과다 청구했는가
- issuance timing이 맞는가
- 특정 시점의 cumulative issued supply가 경제적으로 어떤 의미를 갖는가

이는 general amount-range guard일 뿐, subsidy accounting의 대체물이 아니다.

---

## 7. Technical Mechanics

### Block Reward Enforcement

coinbase transaction은 해당 block의 subsidy + fee를 초과하는 total output value를 가지면 invalid다.[^ref-btc-dev-blockchain] [^ref-btc-core-validation]

즉, miner가 마음대로 monetary policy를 정하는 것이 아니다. 프로토콜이 부과한 상한 안에서 coinbase transaction을 조립할 뿐이다.

### Integer Granularity

Bitcoin amount는 integer satoshi로 추적된다. 따라서 issuance는 연속적인 decimal economics가 아니라 정확한 satoshi unit으로 감소한다. 남은 fraction이 1 satoshi보다 작아지고, Core가 large halving count에서 explicit zero를 반환하기 때문에 결국 subsidy는 0이 된다.[^ref-btc-core-amount] [^ref-btc-core-validation]

### Block Time vs Calendar Time

halving schedule은 calendar date가 아니라 block height 기준이다. 따라서 연간 human time 기준의 "Bitcoin inflation"은 목표 interval 주변에서 실제 block production이 흔들리므로 근사적으로만 예측 가능하다.

이는 protocol issuance를 year-over-year supply growth로 옮길 때 중요하다.

---

## 8. Validation Boundaries

### Consensus는 Quantity는 알지만 Value는 모른다

consensus가 결정할 수 있는 것:

- valid block에서 허용되는 최대 new BTC issuance
- claimed coinbase value가 과도한지 여부
- amount가 valid range 안에 있는지 여부

consensus가 결정할 수 없는 것:

- BTC/USD price
- miner profitability
- real purchasing-power dilution
- fee가 장기 security에 충분한지 여부

### Monetary Policy vs Monetary Economics

Bitcoin monetary policy는 프로토콜 형식으로는 deterministic하지만, 그 경제적 의미는 contingent하다.

- fixed issuance가 fixed price를 뜻하지는 않는다.
- falling subsidy가 자동적인 security failure를 뜻하지는 않는다.
- 낮은 measured inflation이 낮은 volatility를 뜻하지는 않는다.
- capped supply가 특정 market equilibrium을 뜻하지는 않는다.

---

## 9. Security Assumptions and Failure Modes

### Declining Subsidy

subsidy가 감소하면 miner revenue는 상대적으로 fee에 더 많이 의존하게 된다. 이는 security-budget question의 더 많은 부분이 deterministic issuance에서 uncertain market demand와 fee formation으로 이동함을 뜻한다.

이는 구조적 사실이지만, Bitcoin이 반드시 insecure해진다는 결론은 아니다. security sufficiency는 attacker cost, fee demand, BTC price, miner cost structure, strategic behavior에 달려 있다.

### Policy Change Risk

Bitcoin의 monetary policy가 "fixed"라는 말은 네트워크가 관련 consensus rule을 계속 집행하는 한에서만 성립한다. issuance를 바꾸려면 그에 따른 coordination과 compatibility consequences를 가진 consensus change가 필요하다.

### Interpretation Risk

분석가는 종종 여러 개념을 "inflation"으로 압축한다.

- block-schedule issuance
- circulating-supply growth
- free float 대비 dilution
- fiat purchasing-power change

이들은 동일한 metric이 아니다.

---

## 10. Mathematical or Economic Model

### Subsidy Function

다음을 두자.

- `H` = block height
- `I` = halving interval
- `S0` = initial subsidy

그러면 block subsidy function은 대략:

```text
n = floor(H / I)
S(H) = S0 / 2^n
```

이며, 실제 Bitcoin Core 구현은 integer right shift와 eventual zero return으로 실현된다.[^ref-btc-core-validation]

### Total-Issuance Approximation

satoshi truncation을 무시한 직관:

```text
total_supply_limit
= 210000 * (50 + 25 + 12.5 + ...)
= 210000 * 100
= 21000000 BTC
```

이것이 공급 cap에 대한 표준 monetary-policy intuition이다.

### Approximate Issuance Rate

특정 기간의 평균 block interval이 `T`분이고, 그 era의 subsidy가 `S`라면 approximate annual gross issuance는:

```text
annual_issuance ≈ S * blocks_per_year
blocks_per_year ≈ (365 * 24 * 60) / T
```

이는 approximation이다. 실제 block timing은 stochastic하고 era transition은 calendar boundary가 아니라 precise height에서 일어나기 때문이다.

---

## 11. Bitcoin Core 구현

### `validation`

Bitcoin Core는 validation layer에서 `GetBlockSubsidy`를 노출한다. 이 function이 consensus-aware block validation과 관련 test에서 사용되는 declining subsidy schedule을 구현한다.[^ref-btc-core-validation]

### `kernel/chainparams`

mainnet chain parameter는 `nSubsidyHalvingInterval = 210000`을 정의해 generic subsidy function을 실제 네트워크 정책에 연결한다.[^ref-btc-core-chainparams]

### `consensus/amount.h`

`consensus/amount.h`는 `CAmount`, `COIN`, `MAX_MONEY`, `MoneyRange`를 정의한다. 이는 consensus와 validation logic 전반에서 쓰이는 amount-range primitive다.[^ref-btc-core-amount]

### Validation Test

Bitcoin Core는 subsidy halving과 subsidy-limit behavior에 대한 validation test를 포함한다. Bitcoin의 monetary policy는 코드 표면은 작지만 결과는 매우 크기 때문에 이 test가 중요하다.[^ref-btc-core-validation-tests]

---

## 12. 온체인 함의

### 직접 측정 가능한 것

chain data와 consensus rule을 통해 분석가는 다음을 계산할 수 있다.

- height별 subsidy
- 특정 height까지의 cumulative subsidy issued
- miner revenue에서 fee share
- 특정 구간의 gross issuance

### Consensus만으로 직접 읽을 수 없는 것

consensus rule은 다음을 직접 드러내지 않는다.

- economically available float
- lost coin
- true effective circulating supply
- fiat-denominated inflation experience
- miner가 경제적으로 secure하다고 느꼈는지 여부

### Supply Reporting 주의

serious supply reporting은 다음을 구분해야 한다.

- protocol-issued supply
- active-chain supply
- estimated circulating supply
- estimated economically liquid supply

이들은 다른 분석 계층이다.

---

## 13. Institutional Thinking

기관은 Bitcoin monetary policy를 좁지만 신뢰도 높은 프로토콜 계층으로 보고, 더 넓은 경제 시스템 안에 위치시켜야 한다.

### Practical Implications

- Issuance는 block-height 수준에서는 높은 예측 가능성을 가진다.
- calendar-based issuance estimate는 approximation으로 라벨링해야 한다.
- fee-revenue analysis를 "new supply" metric과 혼합하면 안 된다.
- security-budget commentary는 deterministic subsidy decline과 uncertain fee demand를 분리해야 한다.
- supply dashboard는 gross issued BTC, active-chain issued BTC, economically adjusted estimate 중 무엇을 추적하는지 명시해야 한다.

---

## 14. Common Misinterpretations

### "수수료는 신규 공급의 일부다"

틀렸다. fee는 spender에서 miner로 기존 BTC를 이전할 뿐이다.

### "`MAX_MONEY`만으로 Bitcoin monetary policy가 정의된다"

틀렸다. `MAX_MONEY`는 amount bound다. subsidy schedule이 issuance rule이다.

### "Bitcoin inflation은 calendar year마다 고정된다"

틀렸다. issuance는 block height schedule에 의해 고정되고, annualized calendar estimate는 realized block timing에 의존한다.

### "2,100만은 정확히 정해진 날짜들에 따라 모든 coin이 즉시 발행된다는 뜻이다"

틀렸다. cap은 discrete halving과 satoshi granularity를 통해 장기적으로 점근한다.

### "Subsidy 감소는 자동으로 insecure한 Bitcoin을 뜻한다"

틀렸다. security-budget question을 제기할 뿐이며, 답은 경제 조건, fee demand, attacker cost에 달려 있다.

---

## 15. Research Questions

1. 기관은 block-schedule issuance와 calendar-yearized issuance를 혼동 없이 어떻게 함께 제시해야 하는가?
2. era를 거치며 miner revenue variance 중 얼마나 많은 부분이 subsidy decline이 아니라 fee volatility에서 오는가?
3. treasury, trading, risk team에 가장 유용한 supply metric은 무엇인가?
4. 장기 security-budget monitoring은 subsidy, fee share, hash-cost estimate를 어떻게 결합해야 하는가?
5. lost-coin uncertainty는 protocol issuance와 분리해 어떻게 표현해야 하는가?

---

## 16. Practical Exercises

### Exercise 1

height `0`, `210000`, `420000`, `630000`, `840000`의 block subsidy를 consensus rule로 계산하라.

### Exercise 2

halving boundary 전후 block sample을 사용해 다음을 분리하라.

- coinbase output value
- block subsidy
- transaction fee

### Exercise 3

halving era별 cumulative issued subsidy table을 만들고 geometric-series approximation과 비교하라.

### Exercise 4

왜 `MAX_MONEY`만으로 특정 coinbase amount가 valid한지 증명할 수 없는지 설명하라.

---

## 17. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Subsidy schedule and halving interval | Directly specified | Bitcoin Core validation and chain parameters |
| `COIN`, `MAX_MONEY`, `MoneyRange` | Directly specified | `consensus/amount.h` |
| 21 million cap intuition | Directly specified plus analytical framing | Derived from subsidy schedule; Core tests help confirm behavior |
| Fee vs subsidy distinction | Directly specified | Developer docs and validation logic |
| Security-budget interpretation | Inference from sources | Economic reasoning built on protocol facts |

---

## 18. Knowledge Graph

```text
Bitcoin Monetary Policy
├─ Issuance Rules
│  ├─ block subsidy
│  ├─ halving interval
│  ├─ height-based schedule
│  └─ eventual zero subsidy
├─ Amount Bounds
│  ├─ CAmount
│  ├─ COIN
│  ├─ MAX_MONEY
│  └─ MoneyRange
├─ Miner Revenue
│  ├─ subsidy
│  ├─ fees
│  └─ security budget
├─ Measurement
│  ├─ issued supply
│  ├─ annualized issuance
│  ├─ fee share
│  └─ active-chain supply
└─ Risks
   ├─ supply-metric confusion
   ├─ fee/subsidy confusion
   └─ security-budget overstatement
```

---

## 19. 참고문헌

### Primary Sources

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," especially Section 6. https://bitcoin.org/bitcoin.pdf
[^ref-btc-dev-blockchain]: Bitcoin Developer Reference, "Block Chain," including coinbase subsidy and fee description. https://developer.bitcoin.org/reference/block_chain.html
[^ref-btc-core-validation]: Bitcoin Core Doxygen, `validation.h`, including `GetBlockSubsidy`. https://doxygen.bitcoincore.org/validation_8h_source.html
[^ref-btc-core-amount]: Bitcoin Core Doxygen, `consensus/amount.h`, including `CAmount`, `COIN`, `MAX_MONEY`, and `MoneyRange`. https://doxygen.bitcoincore.org/amount_8h_source.html
[^ref-btc-core-chainparams]: Bitcoin Core Doxygen, `kernel/chainparams.cpp`, including mainnet `nSubsidyHalvingInterval = 210000`. https://doxygen.bitcoincore.org/kernel_2chainparams_8cpp_source.html
[^ref-btc-core-validation-tests]: Bitcoin Core Doxygen, `validation_tests.cpp`, including `block_subsidy_test` and `subsidy_limit_test`. https://doxygen.bitcoincore.org/validation__tests_8cpp.html

### Supporting Interpretation Notes

- Where this document discusses security-budget sufficiency, circulating-supply interpretation, or annualized issuance metrics, those statements are inferences built on the consensus issuance rules rather than direct protocol guarantees.

---

## 20. 교차 참조

### Previous

- BITCOIN-024 — Chain Reorganization

### Next

- BITCOIN-026 — Halving

### Related

- BITCOIN-007 — Whitepaper Section 6 — Incentive
- BITCOIN-014 — UTXO Model
- BITCOIN-018 — Transaction Fees
- BITCOIN-020 — Mining
- POW-009 — Coinbase Transaction and Block Subsidy
- BITCOIN-026 — Halving
- BITCOIN-027 — Fee Market

---

## Review Status

### Technical Review

Passed.

- consensus issuance rule와 economic interpretation을 분리했다.
- subsidy schedule, halving interval, amount bound, fee distinction을 각각 분리해 설명했다.
- `MAX_MONEY`를 block-by-block subsidy enforcement와 명시적으로 구분했다.
- 구현 참조는 validation, amount, chain parameter, validation-test surface로 제한했다.

### Evidence Review

Passed.

- 백서와 developer reference가 incentive와 subsidy framing을 뒷받침한다.
- Core validation과 chain parameter가 halving mechanic을 뒷받침한다.
- `consensus/amount.h`가 amount-bound discussion을 뒷받침한다.
- validation test가 monetary policy가 명시적으로 테스트된다는 점을 뒷받침한다.
- economic interpretation은 필요한 부분에서 inference로 표시했다.

### Editorial Review

Passed.

- 구조는 프로젝트 deep-dive format을 따른다.
- metadata가 완전하다.
- 용어는 subsidy, fee, issuance, supply cap, halving interval, `MAX_MONEY`, `MoneyRange`로 일관된다.
- table과 code fence가 닫혀 있다.

### Adversarial Review

Passed.

- fee와 issuance를 혼동하지 않았다.
- `MAX_MONEY`만으로 subsidy schedule이 정의된다고 주장하지 않았다.
- height-based issuance를 exact wall-clock guarantee로 바꾸지 않았다.
- security outcome이 subsidy만으로 결정된다고 주장하지 않았다.
- circulating-supply estimate의 경제적 정밀도를 과장하지 않았다.

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
