---
knowledge_id: BITCOIN-007
title: 백서 섹션 6 — 인센티브(Incentive)
subtitle: 코인베이스 보상, 보조금 스케줄, 거래 수수료, 채굴자 경제성, 그리고 보안 예산
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 80 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Incentives
  - Mining
  - Monetary Policy
parent:
  - Bitcoin Whitepaper
prerequisites:
  - BITCOIN-005
  - BITCOIN-006
  - POW-009
  - POW-014
related_topics:
  - Coinbase Transaction
  - Block Subsidy
  - Transaction Fees
  - Security Budget
  - Miner Incentives
  - Monetary Issuance
  - Fee Market
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-TRANSACTIONS-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-MINER-001
  - REF-BTC-CORE-CONSENSUS-001
  - REF-BTC-CORE-AMOUNT-001
  - REF-BTC-CORE-CHAINPARAMS-001
tags:
  - bitcoin
  - whitepaper
  - incentive
  - mining
  - coinbase
  - subsidy
  - fees
  - security-budget
---

# 백서 섹션 6 — 인센티브(Incentive)
> Deep Dive Series  
> Research Unit: BITCOIN-007

---

## Research Brief

```yaml
knowledge_id: BITCOIN-007
title: Whitepaper Section 6 — Incentive
research_question: >
  How does Bitcoin use the coinbase transaction, block subsidy, and
  transaction fees to incentivize miners, distribute new bitcoin, and
  fund proof-of-work security over time?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-005
  - BITCOIN-006
  - POW-009
parent: Bitcoin Whitepaper
previous: BITCOIN-006
next: BITCOIN-008
related_topics:
  - Coinbase Transaction
  - Block Subsidy
  - Transaction Fees
  - Coinbase Maturity
  - Security Budget
  - Fee Market
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Original Design
  - Protocol Structure
  - Technical Mechanics
  - Mathematical or Economic Model
  - Security Assumptions
  - Bitcoin Core Implementation
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Mining pool payout accounting
  - ASIC manufacturing economics
  - Complete fee-estimation algorithms
  - Long-term BTC price forecasting
  - Non-Bitcoin reward designs
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- 블록의 첫 번째 트랜잭션이 수행하는 인센티브 역할을 설명할 수 있다.
- 블록 보조금과 거래 수수료를 구분할 수 있다.
- 왜 보조금은 새로 발행되는 비트코인이고 수수료는 기존 가치의 이전인지 설명할 수 있다.
- 왜 채굴자가 임의의 블록 보상을 선택할 수 없는지 설명할 수 있다.
- Bitcoin Core가 보조금 스케줄을 어떻게 계산하는지 추적할 수 있다.
- 코인베이스 트랜잭션이 왜 허용 보상보다 적게 청구할 수는 있어도 더 많이 청구할 수는 없는지 설명할 수 있다.
- 왜 코인베이스 출력은 사용 전 성숙 기간이 필요한지 설명할 수 있다.
- 보조금 감소가 어떻게 보안 예산을 더 많이 수수료에 의존하게 만드는지 설명할 수 있다.
- 합의 수준의 보상 규칙과 마이닝 풀의 분배 정책을 구분할 수 있다.
- 온체인 데이터로부터 채굴자 수익을 분석하되 채굴자 신원이나 수익성을 과도하게 단정하지 않을 수 있다.

---

## 2. 핵심 질문

1. 백서가 말하는 "블록의 첫 번째 트랜잭션"은 정확히 무엇인가?
2. 코인베이스 트랜잭션은 채굴자에게 어떤 인센티브를 제공하는가?
3. 보조금과 거래 수수료의 차이는 무엇인가?
4. 백서에서 신규 발행을 금 채굴에 비유한 이유는 무엇인가?
5. Bitcoin Core는 블록 보조금을 어떻게 계산하는가?
6. Bitcoin Core는 최대 코인베이스 보상을 어떻게 강제하는가?
7. 왜 채굴자는 최대 보상보다 적게 청구할 수 있는가?
8. 왜 코인베이스 출력에는 성숙 기간이 있는가?
9. 보조금이 감소하면 무엇이 달라지는가?
10. 수수료 수익만으로 미래 보안이 자동 보장되는가?
11. 분석가는 코인베이스 출력으로부터 무엇을 추론할 수 있는가?
12. 코인베이스 출력만으로는 무엇을 추론할 수 없는가?

---

## 3. Executive Summary

백서 섹션 6은 Bitcoin의 인센티브 메커니즘을 소개한다. 블록의 첫 번째 트랜잭션은 블록 작성자에게 지급되는 새로운 코인을 만드는 특수 트랜잭션이다. 이것은 채굴자가 네트워크를 지원하도록 보상하고, 중앙 발행자 없이 새로운 비트코인을 배분하는 역할을 한다.[^ref-btc-wp]

이 섹션은 또한 거래 수수료를 두 번째 수익원으로 제시한다. 어떤 트랜잭션의 입력 값이 출력 값보다 크면, 그 차액은 수수료가 되며 해당 트랜잭션을 포함한 블록을 채굴한 채굴자가 이를 청구할 수 있다.[^ref-btc-wp]

현대 Bitcoin에서는 이 인센티브가 코인베이스 트랜잭션을 통해 표현된다. 허용되는 최대 코인베이스 출력 값은 다음과 같다.

```text
maximum coinbase claim = block subsidy at height + sum(included transaction fees)
```

Bitcoin Core는 `GetBlockSubsidy`를 사용해 보조금을 계산하며, 시작 보조금은 `50 * COIN`이고 체인의 반감기 간격에 따라 완료된 반감 횟수만큼 right shift를 적용한다. 메인넷은 `nSubsidyHalvingInterval = 210000`으로 설정한다.[^ref-btc-core-validation][^ref-btc-core-chainparams]

Bitcoin Core는 블록 연결 단계에서 보상 상한을 강제한다. 수수료를 계산한 뒤, 코인베이스 출력 값이 `nFees + GetBlockSubsidy(height, consensusParams)`를 초과하면 `bad-cb-amount`로 블록을 거부한다.[^ref-btc-core-validation]

이 인센티브 설계는 두 가지 별도 효과를 가진다.

1. **발행(Issuance):** 보조금은 합의 규칙에 따라 새로운 비트코인을 만든다.
2. **보안 예산(Security budget):** 보조금과 수수료의 합은 활성 체인에 채택된 블록의 채굴자에게 돌아가는 수익이다.

시간이 지나며 보조금이 감소할수록 수수료는 채굴자 수익에서 더 중요해진다. 그러나 이것이 수수료가 자동으로 충분해진다는 뜻은 아니다. 수수료 기반 보안은 블록 공간 수요, 수수료 시장 행태, 채굴자 비용 구조, BTC 가격, 트랜잭션 패턴, 공격 유인에 달려 있다. 이들은 경제 변수이지 직접적인 합의 보장이 아니다.

---

## 4. 원문 해석

백서 섹션 6은 네 가지 핵심 주장을 한다.

1. 블록의 첫 번째 트랜잭션은 특별하다.
2. 그 트랜잭션은 블록 생성자에게 귀속되는 새 코인을 시작한다.
3. 이것은 네트워크 지원에 대한 인센티브를 제공하고 초기 코인 배분을 수행한다.
4. 미리 정해진 발행이 끝난 뒤에는 거래 수수료가 궁극적으로 이 인센티브를 뒷받침할 수 있다.[^ref-btc-wp]

백서는 또한 인센티브 논증을 제시한다. 정직한 노드보다 더 많은 CPU 파워를 가진 공격자는 그 파워를 사람들을 속이는 데 쓸지, 아니면 새 코인을 생성하는 데 쓸지 선택해야 한다는 것이다. 텍스트는 정직한 채굴이 시스템과 자신의 부를 훼손하는 것보다 더 수익성이 높아야 한다고 주장한다.[^ref-btc-wp]

이 논증은 순수한 암호학이 아니라 경제학의 영역이다. 작업증명은 블록 생성에 비용을 부과하고, 보상은 정직한 블록 생성에 가치를 부여한다. Bitcoin의 보안 모델에는 두 요소가 모두 필요하다.

---

## 5. 문자적 해석

### "By convention, the first transaction in a block is a special transaction"

백서가 말하는 이 특수한 첫 트랜잭션이 현대 Bitcoin에서 말하는 코인베이스 트랜잭션이다. Bitcoin Developer 문서는 블록의 첫 트랜잭션을 코인베이스 트랜잭션으로 정의하며, 코인베이스 입력은 이전 outpoint가 없다고 설명한다.[^ref-btc-dev-transactions]

이것은 일반적인 지급 트랜잭션이 아니다. 일반 트랜잭션은 이전 출력을 소비한다. 코인베이스 트랜잭션은 합의 보상 규칙 아래에서 허용된 출력을 생성한다.

### "Starts a new coin owned by the creator of the block"

백서의 표현은 개념적이다. 현대 UTXO 용어로 말하면, 코인베이스 트랜잭션은 하나 이상의 새로운 UTXO를 만든다. 이 출력이 유효한 이유는 채굴자가 그렇게 주장해서가 아니다. 블록이 합의 규칙을 만족하고 코인베이스 값이 허용된 보조금과 수수료 합을 넘지 않을 때만 유효하다.

### "The steady addition of a constant amount of new coins"

백서는 보상을 일정한 발행 메커니즘으로 제시한다. 그러나 현재 Bitcoin Core 규칙은 영구적인 고정 보조금이 아니라 감소하는 보조금 스케줄을 구현한다. 초기 보조금은 50 BTC이며 메인넷에서는 210,000블록마다 절반으로 줄어든다.[^ref-btc-core-validation][^ref-btc-core-chainparams]

### "The incentive can also be funded with transaction fees"

거래 수수료는 입력 값과 출력 값의 차이다. 이 차액은 일반 출력에 직접 할당되지 않는다. 대신 코인베이스 보상 한도 안에서 채굴자가 가져갈 수 있게 된다.

---

## 6. 기술적 해석

### 코인베이스 보상의 구성 요소

블록 보상은 두 부분으로 구성된다.

| 구성 요소 | 출처 | 새 BTC 생성 여부 | 합의 한도 |
|---|---|---:|---|
| 블록 보조금 | 프로토콜 발행 스케줄 | Yes | `GetBlockSubsidy(height, params)` |
| 거래 수수료 | 포함된 트랜잭션의 입력 합 - 출력 합 | No | 유효하게 포함된 수수료 총합 |

보조금은 총 발행량을 늘린다. 수수료는 기존 비트코인을 트랜잭션 발신자에서 채굴자로 재분배한다.

### 최대 청구 규칙

블록 높이 `h`에서:

```text
subsidy = GetBlockSubsidy(h, consensusParams)
fees = sum(input_value(tx) - output_value(tx)) for included non-coinbase txs
max_coinbase_value = subsidy + fees
```

만약:

```text
coinbase_output_value > max_coinbase_value
```

라면 블록은 무효다.

반대로:

```text
coinbase_output_value <= max_coinbase_value
```

라면, 다른 모든 합의 규칙도 통과한다는 전제 아래 금액 규칙은 만족된다.

즉 적게 청구하는 것은 허용된다. 채굴자는 허용된 보상의 일부를 청구하지 않을 수 있다. 그러나 초과 청구는 허용되지 않는다.

### 코인베이스 성숙도

Bitcoin은 코인베이스 출력의 사용 가능 시점도 지연시킨다. Bitcoin Core는 `COINBASE_MATURITY = 100`으로 정의하며, 문서에서도 코인베이스 트랜잭션 출력은 그만큼의 새 블록이 쌓인 뒤에야 사용할 수 있다고 설명한다.[^ref-btc-core-consensus]

코인베이스 성숙도가 중요한 이유는 reorg로 인해 블록이 활성 체인에서 사라지면 그 코인베이스 출력도 함께 사라지기 때문이다. 성숙 기간은 나중에 활성 체인에서 제거될 수도 있는 블록의 보상을 너무 일찍 사용하는 리스크를 줄인다.

---

## 7. 프로토콜 구조

### 인센티브 파이프라인

```text
Miner selects valid transactions
        |
        v
Transaction fees are implied by inputs minus outputs
        |
        v
Miner constructs coinbase transaction
        |
        v
Coinbase may claim subsidy + included fees
        |
        v
Miner searches for valid Proof of Work
        |
        v
Block is broadcast
        |
        v
Nodes validate PoW, block structure, transactions, scripts, and reward amount
        |
        v
If accepted into active chain, coinbase output exists but remains immature
        |
        v
After maturity, coinbase output can be spent
```

### 합의 vs 정책 vs 비즈니스 관행

| Category | Example | Consensus? |
|---|---|---:|
| 코인베이스는 첫 번째 트랜잭션이어야 함 | `CheckBlock`이 첫 트랜잭션 형태를 강제 | Yes |
| 코인베이스 보상은 보조금 + 수수료를 초과할 수 없음 | `ConnectBlock`이 `bad-cb-amount`로 거부 | Yes |
| 코인베이스 성숙도는 100블록 | `COINBASE_MATURITY` | Yes |
| 채굴자는 어떤 트랜잭션을 포함할지 선택 | 블록 구성 동작 | No, 유효성 한도 내 |
| 채굴자는 지급 주소나 스크립트 선택 | 코인베이스 출력 구성 | No, 유효성 한도 내 |
| 풀은 수익을 워커에게 분배 | 풀 회계 정책 | No |
| 거래소는 N confirmation을 기다림 | 비즈니스 정책 | No |

### 인센티브와 체인 선택

채굴자는 자신의 블록이 활성 체인에 남아야만 블록 보상을 얻는다. 유효하지만 stale이 된 블록은 보통 메인체인에서 사용 가능한 보조금과 수수료를 만들어내지 못한다.

이 점은 인센티브 섹션을 네트워크 섹션과 연결한다.

- 빠른 블록 전파는 stale 리스크를 줄인다.
- 피어의 검증 여부가 다른 노드들이 그 블록 위에 쌓을지를 결정한다.
- 누적 작업량이 어떤 유효 브랜치가 이길지를 결정한다.
- 코인베이스 성숙도는 보상의 최종 사용 가능성을 지연시킨다.

---

## 8. 수학적 또는 경제적 모델

### 보조금 스케줄

Bitcoin Core는 다음과 같이 계산한다.

```text
halvings = height / nSubsidyHalvingInterval
if halvings >= 64:
    subsidy = 0
else:
    subsidy = 50 * COIN >> halvings
```

메인넷은 다음과 같이 설정한다.

```text
nSubsidyHalvingInterval = 210000
COIN = 100000000 satoshis
```

따라서 첫 보조금은 다음과 같다.

```text
50 * 100000000 = 5,000,000,000 satoshis
```

네 번의 반감기가 끝난 뒤 보조금은 다음이 된다.

```text
50 BTC / 2^4 = 3.125 BTC
```

이는 메인넷에서 높이 840,000 이후 적용되는 현재 보조금 구간이다.

### 채굴자의 기대 수익

다음을 정의하자.

```text
alpha = miner share of effective network hash rate
S = block subsidy
F = transaction fees in the block
p_stale = probability the block becomes stale
```

단순화한 블록당 기대 수익 기회는 다음과 같이 쓸 수 있다.

```text
expected_reward_share ~= alpha * (S + F) * (1 - p_stale)
```

이것은 합의 공식이 아니라 경제적 근사다. 실제 채굴 수익은 분산, 풀 분배 규칙, 가동률, 블록 전파, 트랜잭션 선택, 수수료 변동성, 헤징, 전기요금, 자금조달, 장비 감가상각에 좌우된다.

### 보안 예산

활성 체인에 채택된 한 블록에 대해:

```text
security_budget_per_block = subsidy + fees
```

일정 기간에 대해:

```text
security_budget_interval = sum(subsidy + fees for accepted blocks in interval)
```

이 지표는 마이닝 인센티브를 통해 체인을 방어하는 수익 규모를 추정한다는 점에서 유용하다. 다만 이것은 총 채굴 비용과 동일하지 않다. 시장 상황에 따라 채굴자는 손실을 감수하거나 이익을 낼 수 있다.

### 수수료 전이

백서 섹션 6은 미리 정해진 코인 발행이 끝난 뒤 인센티브가 전적으로 거래 수수료로 전환될 수 있다고 말한다.[^ref-btc-wp]

이것은 설계 주장인 동시에 장기 경제 가설이다.

- 합의 규칙은 시간이 지나며 보조금을 줄일 수 있다.
- 프로토콜은 수수료가 채굴자 보상이 되도록 허용할 수 있다.
- 실제로 수수료 수익이 충분한지는 미래 블록 공간 수요와 채굴자 경제성에 달려 있다.

"전환될 수 있다"는 표현을, 어떤 미래 수요 환경에서도 항상 충분하다는 증거로 받아들이면 안 된다.

---

## 9. 보안 가정

### 인센티브 정렬

백서의 인센티브 논증은 큰 해시파워를 가진 채굴자가 Bitcoin 가치를 훼손하는 공격보다 정직 채굴 수익을 선호해야 한다는 생각에 의존한다.[^ref-btc-wp]

여기에는 몇 가지 가정이 들어 있다.

1. 채굴 보상이 경제적으로 의미 있는 수준이다.
2. 공격 이익이 정직 채굴 이익과 Bitcoin 신뢰성 보존 가치보다 낮다.
3. 채굴자는 Bitcoin 또는 채굴 인프라에 장기적 노출을 가진다.
4. 정직한 풀노드는 비용이 큰 작업증명이 붙어 있어도 무효 블록은 거부한다.
5. 네트워크가 유효한 블록 위로 충분히 빨리 수렴하여 정직 채굴이 stale 리스크로 과도하게 불이익을 받지 않는다.

### 인센티브 논증의 한계

이 인센티브 논증은 절대적이지 않다.

- 공격자는 이익보다 파괴를 더 중시할 수 있다.
- 단기 공격자는 장기 BTC 가치에 관심이 없을 수 있다.
- 차입 또는 임대 해시파워는 인센티브 구조를 바꿀 수 있다.
- 마이닝 풀은 해시 소유자와 풀 운영자 사이의 principal-agent 문제를 만들 수 있다.
- 수수료 급등은 단기 reorg 유인을 만들 수 있다.
- 보조금 감소는 보안 예산과 공격 비용의 관계를 바꿀 수 있다.

이들은 코인베이스 보상 규칙 자체의 실패가 아니라, 경제적·운영상의 리스크다.

### 제약 조건으로서의 풀노드

채굴자는 유효한 블록을 통해서만 보상받는다. 채굴자는 허용된 코인베이스 값을 초과해 자기 자신에게 보상할 수 없는데, 풀노드가 그런 블록을 거부하기 때문이다. Bitcoin Core의 `ConnectBlock`은 코인베이스 출력 값을 수수료 + 보조금과 비교함으로써 이를 강제한다.[^ref-btc-core-validation]

이 덕분에 인센티브 메커니즘은 permissionless mining과 양립한다. 누구나 채굴을 시도할 수 있지만, 모두가 독립적으로 보상 청구를 검증한다.

---

## 10. Bitcoin Core 구현

### 보조금 계산

Bitcoin Core는 `validation.h`에서 `GetBlockSubsidy`를 선언하고, `validation.cpp`에서 이를 정의한다.[^ref-btc-core-validation]

이 함수는 다음을 수행한다.

1. 완료된 반감 횟수를 `nHeight / consensusParams.nSubsidyHalvingInterval`로 계산한다.
2. `halvings >= 64`이면 0을 반환한다.
3. `50 * COIN`에서 시작한다.
4. `halvings`만큼 right shift한다.
5. 결과 보조금을 반환한다.[^ref-btc-core-validation]

메인넷 합의 파라미터는 `nSubsidyHalvingInterval = 210000`으로 설정한다.[^ref-btc-core-chainparams]

### 코인베이스 구성

Bitcoin Core의 블록 어셈블러는 새 블록 템플릿을 만들면서 후보 코인베이스 트랜잭션을 구성한다. `BlockAssembler::CreateNewBlock`에서 코인베이스 트랜잭션을 생성하고, 코인베이스 출력 값을 다음으로 설정한다.

```text
nFees + GetBlockSubsidy(nHeight, chainparams.GetConsensus())
```

또한 BIP34가 요구하는 대로 코인베이스 `scriptSig`를 블록 높이로 시작한다.[^ref-btc-core-miner]

이것은 후보 블록 구성 단계다. 이것만으로 블록이 유효해지는 것은 아니다. 블록은 여전히 유효한 작업증명을 찾아야 하고, 피어의 검증도 통과해야 한다.

### 코인베이스 보상 검증

블록 연결 과정에서 Bitcoin Core는 다음을 계산한다.

```text
blockReward = nFees + GetBlockSubsidy(pindex->nHeight, params.GetConsensus())
```

코인베이스 출력 값이 이 값을 초과하면, 블록은 `bad-cb-amount`로 거부된다.[^ref-btc-core-validation]

### 금액 상한

Bitcoin Core는 다음을 정의한다.

```text
COIN = 100000000
MAX_MONEY = 21000000 * COIN
```

그리고 `MoneyRange`는 금액이 음수가 아니고 `MAX_MONEY`를 초과하지 않는지 확인한다.[^ref-btc-core-amount]

이 상한은 블록별 보조금 스케줄과 동일한 개념이 아니다. 이것은 Bitcoin 금액 전반에 대한 일반적인 유효성 범위이며, 블록별 보상 한도는 별도로 보조금 + 수수료 규칙으로 강제된다.

### 코인베이스 성숙도

`COINBASE_MATURITY = 100`은 `consensus.h`에 정의되어 있고, 검증 코드는 코인베이스 출력의 사용 가능 여부와 reorg 관련 mempool 유효성을 다룰 때 이 성숙도 로직을 사용한다.[^ref-btc-core-consensus][^ref-btc-core-validation]

---

## 11. 온체인 함의

### 분석가가 관측할 수 있는 것

블록 및 트랜잭션 데이터에서 분석가는 다음을 관측할 수 있다.

- 코인베이스 트랜잭션
- 코인베이스 출력 값
- 코인베이스 보상을 받는 출력 스크립트
- 블록 높이
- 블록에 포함된 트랜잭션 목록
- 입력 값을 알 수 있다면 총 수수료
- 코인베이스가 의도적으로 적게 청구한 것처럼 보이는지 여부
- 코인베이스 출력이 사용 가능할 만큼 성숙했는지 여부
- 풀 태그 같은 코인베이스 메타데이터(단, attribution 한계 포함)

### 분석가가 직접 추론할 수 없는 것

코인베이스 데이터만으로는 다음을 신뢰성 있게 추론할 수 없다.

- 채굴자의 실제 세계 신원
- 마이닝 풀의 정확한 수익 분배
- 채굴자의 전기 비용
- 채굴자의 실제 수익성
- 풀 태그의 진위
- 태그에 적힌 주체가 वास्तव로 해당 블록을 찾았는지 여부
- 오프체인 수수료 공유 계약

### 코인베이스 값 검증

실무적인 검증 절차는 다음과 같다.

```text
1. Get block height.
2. Compute subsidy for that height.
3. For every non-coinbase transaction:
   fee = sum(spent input values) - sum(outputs)
4. Sum all fees.
5. Compare coinbase output value with subsidy + fees.
6. If coinbase value is greater, the block is invalid.
7. If coinbase value is lower, the miner underclaimed.
```

모든 워크플로에서 전체 검증을 재구현할 필요는 없지만, 이 모델은 왜 코인베이스 값이 임의 발행이 아닌지 설명해 준다.

### 보안 예산 지표

자주 쓰이는 분석 지표는 다음과 같다.

| Metric | Formula | Use |
|---|---|---|
| Block reward | subsidy + fees | 블록당 채굴자 수익 상한 |
| Fee share | fees / (subsidy + fees) | 수수료 시장 중요도 |
| Daily security budget | sum(block rewards over day) | 채굴자 수익 프록시 |
| Annualized security budget | daily budget * 365 | 비교용 보안 지표 |
| Coinbase spend maturity | coinbase height + 100 | 사용 가능 시점 체크 |

채굴자 경제성을 평가할 때는 이 지표들을 BTC 기준과 법정통화 기준으로 모두 봐야 한다. 채굴 비용은 대체로 법정통화로 표시되고 보상은 BTC로 표시되기 때문이다.

---

## 12. 기관 관점에서의 해석

### 정산과 채굴자 인센티브

기관은 채굴자 인센티브가 추상 개념이 아니라는 점을 이해해야 한다. 이는 다음에 직접 영향을 준다.

- confirmation 신뢰성
- 혼잡 시 필요한 수수료 수준
- 비정상적으로 높은 수수료 블록 이후의 reorg 유인
- 마이닝 중앙화 압력
- 장기 보안 예산 논쟁
- 커스터디와 거래소의 confirmation 정책

### 보조금 감소

보조금이 줄어들수록 네트워크는 채굴자 보상을 위해 거래 수수료에 더 의존하게 된다. 이 때문에 다음과 같은 연구 질문이 생긴다.

- 블록 공간 수요가 충분한 수수료를 만들어낼 것인가?
- 수수료 변동성이 단기 reorg 유인을 늘릴 것인가?
- 채굴자가 트랜잭션 선택에 더 민감해질 것인가?
- 채굴자에 대한 out-of-band payment가 더 중요해질 것인가?
- 풀 집중도가 트랜잭션 포함에 영향을 줄 것인가?

이 문제들은 섹션 6만으로 결론나지 않는다. 섹션 6은 메커니즘을 제시할 뿐이며, 현대 분석은 수수료 시장 데이터와 채굴자 행동을 함께 봐야 한다.

### 기업을 위한 통제

기관 시스템은 다음을 갖춰야 한다.

- 독립적인 풀노드 운영
- 검증된 체인 데이터를 기준으로 입금 확인
- 금액 및 리스크에 따른 confirmation 차등 적용
- reorg와 stale block 상태 모니터링
- 수수료 시장 스트레스 모니터링
- 고액 입금을 제3자 API 상태만 보고 크레딧하지 않기
- 코인베이스 기반 채굴자 attribution을 휴리스틱으로 취급하기

### 투자 리서치 관점

Bitcoin 리서치에서 섹션 6은 통화정책과 보안을 연결한다.

```text
issuance schedule
    -> miner revenue
    -> hashpower incentive
    -> attack cost
    -> settlement confidence
    -> institutional usability
```

이 연결은 일부는 기계적이고 일부는 경제적이다. 보조금 스케줄은 합의다. 그러나 그 보조금의 시장 가치는 합의가 아니다. 수수료 수요는 자생적으로 형성되며, 채굴자 비용은 외생 변수다. 보안 예산 분석은 이 층위를 분리해야 한다.

---

## 13. 흔한 오해

### Misinterpretation 1: Miners create any amount they want.

틀렸다. 채굴자는 코인베이스 트랜잭션을 구성하지만, 코인베이스 출력 값이 보조금 + 수수료를 초과하면 풀노드가 블록을 거부한다.[^ref-btc-core-validation]

### Misinterpretation 2: Transaction fees are newly created bitcoin.

틀렸다. 수수료는 입력 값이 출력 값을 초과함으로써 암묵적으로 지불되는 기존 비트코인이다. 새로 발행되는 것은 보조금이다.

### Misinterpretation 3: The coinbase reward is immediately spendable.

틀렸다. Bitcoin Core 합의 규칙 아래에서는 코인베이스 출력이 사용 가능해지기 전에 100블록 성숙 기간이 필요하다.[^ref-btc-core-consensus]

### Misinterpretation 4: A coinbase tag proves miner identity.

틀렸다. 코인베이스 메타데이터는 자기 보고이거나 운영상 삽입된 정보다. attribution에 도움은 되지만, 추가 증거 없이는 실제 세계의 신원을 증명하지 않는다.

### Misinterpretation 5: Fee transition is already solved by protocol design.

과장이다. 프로토콜은 수수료 기반 채굴 인센티브로의 전환을 허용한다. 그러나 미래의 수수료 수익이 충분한지는 경제 문제다.

### Misinterpretation 6: Security budget equals miner cost.

틀렸다. 보안 예산은 활성 체인에 채택된 블록으로부터 उपलब्ध한 채굴자 수익이다. 채굴 비용은 하드웨어, 전기, 금융, 운영, 지역, 헤징에 달려 있다.

---

## 14. 연구 질문

1. 보조금 시대별로 채굴자 수익에서 수수료 비중은 어떻게 변해왔는가?
2. 수수료 수익 변동성은 보조금 수익 변동성과 비교해 어떤가?
3. 고수수료 블록은 단기 reorg 유인을 높이는가?
4. 마이닝 풀 사이에서 코인베이스 지급 스크립트는 얼마나 집중되어 있는가?
5. 코인베이스 태그는 지급 주소 클러스터링과 비교해 얼마나 신뢰할 수 있는가?
6. 보조금 감소는 소규모 채굴자의 생존에 어떤 영향을 주는가?
7. 수수료 변동성이 높을 때 기관은 confirmation 정책을 어떻게 설정해야 하는가?
8. 어떤 수수료 시장 조건이 Bitcoin의 보안 예산 프로파일을 실질적으로 바꿀 수 있는가?
9. 채굴자가 의도적으로 보상을 적게 청구하는 경우는 얼마나 자주 발생하며 왜 그런가?
10. 코인베이스 성숙도는 채굴자와 풀의 reorg 리스크에 어떤 영향을 주는가?

---

## 15. Practical Exercises

1. 높이 `840,001`에 있고 수수료가 `0.25 BTC`인 블록의 최대 유효 코인베이스 출력 값을 계산하라.
2. 높이 `900,000`의 코인베이스 출력이 왜 높이 `900,050`의 블록에서 지출되면 안 되는지 설명하라.
3. 최근 블록 하나를 골라 다음을 계산하라.

```text
fee_share = fees / (subsidy + fees)
```

4. 같은 블록에 대해 코인베이스 태그 attribution과 지급 출력 클러스터링을 비교하라.
5. stale 블록의 코인베이스 트랜잭션이 왜 사용 가능한 메인체인 비트코인을 만들지 못하는지 설명하라.

---

## 16. 증거 분류

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Whitepaper Section 6 incentive design | A |
| REF-BTC-DEV-TRANSACTIONS-001 | Official developer documentation | Coinbase input structure and transaction format | A |
| REF-BTC-CORE-VALIDATION-001 | Primary implementation source | `GetBlockSubsidy`, `ConnectBlock`, `bad-cb-amount` | A |
| REF-BTC-CORE-MINER-001 | Primary implementation source | `BlockAssembler::CreateNewBlock` coinbase construction | A |
| REF-BTC-CORE-CONSENSUS-001 | Primary implementation source | `COINBASE_MATURITY` consensus constant | A |
| REF-BTC-CORE-AMOUNT-001 | Primary implementation source | `COIN`, `MAX_MONEY`, `MoneyRange` | A |
| REF-BTC-CORE-CHAINPARAMS-001 | Primary implementation source | Mainnet `nSubsidyHalvingInterval` | A |

### Claim Ledger

| Claim ID | Claim | Classification | Evidence | Source |
|---|---|---|---|---|
| C001 | Whitepaper Section 6 defines the first transaction in a block as a special reward transaction. | FACT | A | REF-BTC-WP-001 |
| C002 | Transaction fees are the difference between transaction inputs and outputs. | FACT | A | REF-BTC-WP-001 |
| C003 | Bitcoin Core computes subsidy with `GetBlockSubsidy`. | FACT | A | REF-BTC-CORE-VALIDATION-001 |
| C004 | Mainnet sets `nSubsidyHalvingInterval` to 210,000. | FACT | A | REF-BTC-CORE-CHAINPARAMS-001 |
| C005 | Bitcoin Core rejects excessive coinbase output value as `bad-cb-amount`. | FACT | A | REF-BTC-CORE-VALIDATION-001 |
| C006 | Bitcoin Core's block assembler constructs candidate coinbase output value as fees plus subsidy. | FACT | A | REF-BTC-CORE-MINER-001 |
| C007 | Coinbase outputs require 100-block maturity. | FACT | A | REF-BTC-CORE-CONSENSUS-001 |
| C008 | Fees redistribute existing BTC while subsidy issues new BTC. | INTERPRETATION | A | REF-BTC-WP-001; REF-BTC-CORE-VALIDATION-001 |
| C009 | Future fee adequacy is an economic question rather than a direct consensus guarantee. | INTERPRETATION | B | REF-BTC-WP-001 |
| C010 | Coinbase metadata alone does not prove real-world miner identity. | INTERPRETATION | B | REF-BTC-DEV-TRANSACTIONS-001 |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| HEURISTIC | Practical research rule requiring context |
| OPEN | Unresolved or future-dependent question |

---

## 17. 지식 그래프

```text
BITCOIN-007 Incentive
|
+-- interprets: Whitepaper Section 6
|
+-- uses: Coinbase Transaction
|   +-- first transaction in block
|   +-- creates permitted reward outputs
|   +-- matures after 100 blocks
|
+-- reward = Subsidy + Fees
|   +-- subsidy: new issuance
|   +-- fees: existing BTC transfer
|
+-- enforced_by: Bitcoin Core
|   +-- GetBlockSubsidy(height, params)
|   +-- ConnectBlock bad-cb-amount check
|   +-- COINBASE_MATURITY
|
+-- affects: Security Budget
|   +-- miner revenue
|   +-- hashpower incentive
|   +-- fee-market dependence
|
+-- leads_to: BITCOIN-008 Reclaiming Disk Space
```

---

## 18. 참고문헌

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Section 6, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-transactions]: Bitcoin Developer Documentation, "Transactions — Coinbase Input: The Input Of The First Transaction In A Block," https://developer.bitcoin.org/reference/transactions.html, accessed 2026-08-04.

[^ref-btc-core-validation]: Bitcoin Core Contributors, `src/validation.cpp`, functions `GetBlockSubsidy`, `ConnectBlock`, and coinbase reward validation, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/validation_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-miner]: Bitcoin Core Contributors, `src/node/miner.cpp`, `BlockAssembler::CreateNewBlock` coinbase construction, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/miner_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-consensus]: Bitcoin Core Contributors, `src/consensus/consensus.h`, `COINBASE_MATURITY`, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/consensus_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-amount]: Bitcoin Core Contributors, `src/consensus/amount.h`, `COIN`, `MAX_MONEY`, and `MoneyRange`, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/amount_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-chainparams]: Bitcoin Core Contributors, `src/kernel/chainparams.cpp`, mainnet consensus parameters including `nSubsidyHalvingInterval`, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/kernel_2chainparams_8cpp_source.html, accessed 2026-08-04.

---

## 19. 교차 참조

### Parent

- Bitcoin Whitepaper

### Previous

- BITCOIN-006 — Whitepaper Section 5 — Network

### Next

- BITCOIN-008 — Whitepaper Section 7 — Reclaiming Disk Space

### Related

- BITCOIN-024 — Bitcoin Monetary Policy
- BITCOIN-026 — Fee Market
- BITCOIN-027 — Security Budget
- POW-008 — Bitcoin Mining
- POW-009 — Coinbase Transaction and Mining Commitments
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- Whitepaper Section 6와 이후 구현 세부 사항을 분리했다.
- 보조금, 수수료, 코인베이스 구성, 보상 검증, 성숙도를 분리했다.
- `GetBlockSubsidy`, `BlockAssembler::CreateNewBlock`, `ConnectBlock`, `COINBASE_MATURITY`, `COIN`, `MAX_MONEY`를 Bitcoin Core 소스와 대조했다.
- 적게 청구하는 경우와 초과 청구하는 경우를 구분했다.

### Evidence Review

Passed.

- 백서 관련 주장은 섹션 6을 직접 인용한다.
- 현재 합의 및 구현 관련 주장은 Bitcoin Core 소스를 인용한다.
- 코인베이스 구조 관련 주장은 공식 Bitcoin Developer 문서를 인용한다.
- 수수료 충분성과 보안 예산에 대한 경제적 진술은 해석으로 표시했다.

### Editorial Review

Passed.

- Markdown 헤딩은 프로젝트 딥다이브 구조를 따른다.
- 메타데이터는 완전하다.
- 표와 코드 펜스는 닫혀 있다.
- subsidy, fees, coinbase, block reward, security budget, maturity 용어를 일관되게 사용했다.

### Adversarial Review

Passed.

- 채굴자가 임의의 비트코인을 만들 수 있다는 식의 오해를 피했다.
- 수수료가 새로 발행되는 코인이라는 식의 오해를 피했다.
- 수수료 시장의 충분성을 보장된 사실로 취급하지 않았다.
- 코인베이스 메타데이터 attribution의 한계를 포함했다.

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
