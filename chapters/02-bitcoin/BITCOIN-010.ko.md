---
knowledge_id: BITCOIN-010
title: 백서 섹션 9 — 가치의 결합과 분할(Combining and Splitting Value)
subtitle: UTXO 입력, 다중 출력, 거스름돈, 수수료, Fan-Out, 그리고 트랜잭션 구성
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 85 min
estimated_study: 230 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Transactions
  - UTXO
  - Wallets
parent:
  - Bitcoin Whitepaper
prerequisites:
  - BITCOIN-003
  - BITCOIN-007
  - BITCOIN-009
  - POW-009
related_topics:
  - UTXO Model
  - Transaction Inputs
  - Transaction Outputs
  - Change Output
  - Transaction Fees
  - Fan-Out
  - Privacy
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-TRANSACTIONS-001
  - REF-BTC-CORE-TRANSACTION-001
  - REF-BTC-CORE-TX-CHECK-001
  - REF-BTC-CORE-TX-VERIFY-001
  - REF-BTC-CORE-AMOUNT-001
tags:
  - bitcoin
  - whitepaper
  - combining-and-splitting-value
  - utxo
  - transactions
  - inputs
  - outputs
  - change
  - fees
---

# 백서 섹션 9 — 가치의 결합과 분할(Combining and Splitting Value)
> Deep Dive Series  
> Research Unit: BITCOIN-010

---

## Research Brief

```yaml
knowledge_id: BITCOIN-010
title: Whitepaper Section 9 — Combining and Splitting Value
research_question: >
  How does Bitcoin represent practical payments using transactions with
  multiple inputs and outputs, and what are the validation, fee, privacy,
  and on-chain-analysis implications of combining and splitting value?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-003
  - BITCOIN-007
  - BITCOIN-009
  - POW-009
parent: Bitcoin Whitepaper
previous: BITCOIN-009
next: BITCOIN-011
related_topics:
  - UTXO Model
  - Transaction Structure
  - Change Output
  - Transaction Fees
  - Multi-Input Heuristic
  - Fan-Out
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
  - Complete wallet coin-selection algorithms
  - Full Bitcoin privacy chapter
  - Script template taxonomy
  - Mempool package policy
  - Lightning channel construction
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- 왜 Bitcoin 트랜잭션이 여러 입력과 여러 출력을 지원하는지 설명할 수 있다.
- UTXO 모델에서 입력과 출력을 구분할 수 있다.
- 왜 하나의 큰 이전 출력이 지급과 거스름돈으로 나뉠 수 있는지 설명할 수 있다.
- 왜 여러 개의 작은 이전 출력을 하나의 큰 지급으로 결합할 수 있는지 설명할 수 있다.
- 거래 수수료를 입력 값 합계에서 출력 값 합계를 뺀 값으로 계산할 수 있다.
- 거스름돈이 특별한 합의 필드가 아니라 일반 출력이라는 점을 설명할 수 있다.
- fan-out이 왜 전체 트랜잭션 이력의 완전한 독립 사본을 들고 다닐 필요를 만들지 않는지 설명할 수 있다.
- 트랜잭션 유효성과 지갑의 구성 선택을 구분할 수 있다.
- 다중 입력 트랜잭션이 만드는 프라이버시 리스크를 설명할 수 있다.
- 어떤 트랜잭션 구성 관련 주장이 사실, 휴리스틱, 해석인지 식별할 수 있다.

---

## 2. 핵심 질문

1. 왜 코인을 개별적으로만 다루면 번거로운가?
2. Bitcoin에서 가치를 결합한다는 것은 무엇인가?
3. Bitcoin에서 가치를 분할한다는 것은 무엇인가?
4. 트랜잭션은 어떻게 이전 출력을 소비하는가?
5. 거스름돈 출력이란 무엇인가?
6. 거스름돈은 온체인에서 특별한 출력 유형으로 표시되는가?
7. 거래 수수료는 어떻게 표현되는가?
8. 왜 입력 값은 최소한 출력 값 이상이어야 하는가?
9. fan-out이란 무엇인가?
10. 왜 fan-out은 전체 트랜잭션 이력 복사를 요구하지 않는가?
11. 다중 입력 트랜잭션은 어떤 프라이버시 휴리스틱을 만드는가?
12. 분석가는 트랜잭션 구조에서 무엇을 추론할 수 있고, 무엇은 여전히 불확실한가?

---

## 3. Executive Summary

백서 섹션 9는 왜 Bitcoin 트랜잭션이 여러 입력과 여러 출력을 가질 수 있는지 설명한다. 가치의 모든 작은 단위를 각각 별도 트랜잭션으로 처리하는 것은 비현실적이다. 그래서 Bitcoin은 이전 출력들로부터 가치를 결합하고, 새로운 출력들로 다시 분할할 수 있게 설계되었다.[^ref-btc-wp]

핵심 구조는 다음과 같다.

```text
previous outputs consumed as inputs
    -> transaction
    -> new outputs created
```

보통 하나의 트랜잭션은 더 큰 이전 출력 하나를 소비하고, 두 개의 출력을 만든다. 하나는 상대방에게 보내는 지급 출력이고, 다른 하나는 나머지를 발신자에게 돌려주는 거스름돈 출력이다. 또는 더 큰 지급을 만들기 위해 여러 개의 작은 이전 출력을 입력으로 결합할 수도 있다.[^ref-btc-wp]

현대 Bitcoin은 이를 UTXO 모델로 표현한다. 코인베이스가 아닌 입력은 이전 outpoint를 참조하며, 이는 트랜잭션 ID와 출력 인덱스를 식별한다. 출력은 사토시 단위의 값과 locking script를 지정한다.[^ref-btc-dev-transactions][^ref-btc-core-transaction]

수수료는 별도의 합의 필드가 아니다. 수수료는 총 입력 값과 총 출력 값의 차이다.

```text
fee = sum(inputs) - sum(outputs)
```

Bitcoin Core의 `CheckTxInputs`는 모든 입력이 사용 가능한지 확인하고, 입력 값이 출력 값보다 작지 않은지 검사한 뒤, 입력 값에서 출력 값을 뺀 값을 거래 수수료로 계산한다.[^ref-btc-core-tx-verify]

분석상의 핵심 주의점은 프라이버시다. 여러 입력을 결합하면 하나의 트랜잭션이 모든 입력을 지출하기 위한 승인을 필요로 하므로, 공통 통제를 시사할 수 있다. 하지만 이것은 휴리스틱이지 보편적 사실은 아니다. 협업형 트랜잭션, CoinJoin 유사 구조, 커스터디 배칭, 지갑 동작은 단순 소유권 가정을 깨뜨릴 수 있다.

---

## 4. 원래 설계

백서 섹션 9는 코인을 개별적으로 다룰 수도 있지만, 1센트 단위마다 별도 트랜잭션을 만드는 것은 번거롭다고 말한다. 실용적인 결제를 지원하기 위해 트랜잭션은 여러 입력과 여러 출력을 가진다.[^ref-btc-wp]

백서는 두 가지 전형적 패턴을 제시한다.

| Pattern | Description |
|---|---|
| Single larger input | 더 큰 이전 출력을 소비하고 거스름돈을 반환 |
| Multiple smaller inputs | 여러 개의 작은 금액을 합쳐 하나의 지급을 만듦 |

또한 fan-out은 문제가 아니라고 말한다. 트랜잭션 이력의 완전한 독립 사본을 추출할 필요가 없기 때문이다.[^ref-btc-wp]

이 섹션은 짧지만 기초적이다. 이것은 섹션 2의 서명 체인 개념과, 실제 Bitcoin이 사용하는 UTXO 트랜잭션 모델 사이를 이어주는 다리다.

---

## 5. 문자적 해석

### "Combining Value"

가치를 결합한다는 것은 여러 이전 출력을 하나의 새 트랜잭션 입력으로 함께 사용하는 것이다.

예시:

```text
Input A: 0.40 BTC
Input B: 0.35 BTC
Input C: 0.30 BTC
Total:   1.05 BTC
```

이 트랜잭션은 이 입력들을 함께 사용해 총합이 `1.05 BTC` 이하인 출력들을 만들 수 있다.

### "Splitting Value"

가치를 분할한다는 것은 입력이 소비한 가치를 여러 개의 새 출력으로 나누는 것이다.

예시:

```text
Input total:     1.00 BTC
Payment output: 0.70 BTC
Change output:  0.29 BTC
Fee:            0.01 BTC
```

지급 출력과 거스름돈 출력은 모두 일반 출력이다. 합의는 하나를 "payment", 다른 하나를 "change"로 표시하지 않는다.

### "At Most Two Outputs"

백서는 보통 많아야 두 개의 출력, 즉 지급과 거스름돈이 있을 것이라고 말한다.[^ref-btc-wp] 이것은 초기 설계의 단순화이지 현재의 합의 한도가 아니다.

현대 Bitcoin 트랜잭션은 트랜잭션과 블록 한도 내에서 두 개보다 더 많은 출력을 가질 수 있다. Bitcoin Developer 문서는 트랜잭션이 가변 개수의 출력을 가진다고 설명하고, Bitcoin Core의 트랜잭션 모델도 출력을 `vout` 벡터에 저장한다.[^ref-btc-dev-transactions][^ref-btc-core-transaction]

### "Fan-Out"

fan-out은 하나의 트랜잭션이 여러 이전 트랜잭션에 의존하고, 그 이전 트랜잭션이 다시 더 많은 이전 트랜잭션에 의존하는 구조를 뜻한다. 백서는 이것이 문제되지 않는다고 말하는데, 전체 이력의 완전한 독립 사본이 필요하지 않기 때문이다.[^ref-btc-wp]

현대 용어로는, 검증은 현재 UTXO 뷰를 확인한다는 뜻이다. 노드는 해당 입력이 소비하는 이전 출력과 충분한 검증 상태만 필요하지, 모든 조상 트랜잭션을 소비 트랜잭션 내부에 묶어 넣을 필요는 없다.

---

## 6. 프로토콜 구조

### 트랜잭션 구성 요소

Bitcoin Developer 문서는 raw transaction을 다음과 같이 설명한다.

| Component | Role |
|---|---|
| Version | 트랜잭션 버전 번호 |
| Input count | 입력 개수 |
| Inputs | 소비되는 이전 출력 |
| Output count | 출력 개수 |
| Outputs | 새로 생성되는 사용 가능 출력 |
| Lock time | 선택적 finality 제약 |

코인베이스가 아닌 각 입력은 이전 outpoint를 참조한다. 각 출력은 값과 locking script를 담는다.[^ref-btc-dev-transactions]

### 입력과 출력 객체

Bitcoin Core는 `src/primitives/transaction.h`에서 이 개념을 다음 구조로 모델링한다.

| Object | Role |
|---|---|
| `COutPoint` | 트랜잭션 해시 + 출력 인덱스 |
| `CTxIn` | previous outpoint, scriptSig, sequence, witness를 담는 입력 |
| `CTxOut` | 값과 scriptPubKey를 담는 출력 |
| `CTransaction` | `vin`, `vout`, version, lock time을 담는 트랜잭션 |

이것은 구현 구조이지 별도의 합의 설명 문서가 아니다. 그러나 Bitcoin Core가 트랜잭션 데이터를 어떻게 표현하는지 보여주는 1차 구현 증거다.[^ref-btc-core-transaction]

### 가치 보존

코인베이스가 아닌 트랜잭션에 대해:

```text
sum(input values) >= sum(output values)
```

그 차이가 거래 수수료다. 출력 값이 입력 값을 초과하면, 그 트랜잭션은 승인되지 않은 가치를 만들려는 것이므로 무효다.[^ref-btc-core-tx-verify]

코인베이스 트랜잭션은 예외다. 코인베이스는 허용된 보조금을 생성하고 블록 보상 규칙에 따라 수수료를 청구한다. 이는 BITCOIN-007과 POW-009에서 다뤘다.

---

## 7. 기술적 메커니즘

### 입력 결합

어떤 지갑이 세 개의 UTXO를 가진다고 하자.

```text
UTXO A = 0.20 BTC
UTXO B = 0.30 BTC
UTXO C = 0.55 BTC
```

`0.90 BTC`를 지급하고 `0.01 BTC` 수수료를 내기 위해 지갑은 세 UTXO를 모두 결합할 수 있다.

```text
inputs = 0.20 + 0.30 + 0.55 = 1.05 BTC
outputs = 0.90 + 0.14 = 1.04 BTC
fee = 1.05 - 1.04 = 0.01 BTC
```

`0.14 BTC` 출력이 발신자에게 돌아간다면 경제적으로는 거스름돈이다. 하지만 온체인에서는 여전히 그냥 하나의 출력일 뿐이다.

### 출력 분할

하나의 입력이 여러 개의 출력을 만들 수도 있다.

```text
input = 1.00 BTC

outputs:
  recipient 1 = 0.20 BTC
  recipient 2 = 0.30 BTC
  change      = 0.49 BTC

fee = 0.01 BTC
```

이 구조는 배칭, 상거래 지급, 거래소 출금, 지갑 거스름돈 처리 등 다양한 패턴을 가능하게 한다. 프로토콜은 값과 스크립트는 검증하지만, 각 출력의 비즈니스 목적은 알지 못한다.

### 수수료 구성

수수료는 암묵적이다.

```text
fee = sum(previous outputs spent) - sum(new outputs created)
```

Bitcoin Core의 `CheckTxInputs`는 다음을 수행한다.

1. 모든 입력이 UTXO 뷰에 존재하는지 확인한다.
2. 존재하지 않거나 이미 사용된 입력을 거부한다.
3. 해당 시점에 필요한 코인베이스 성숙도 규칙을 강제한다.
4. 입력 값 총합을 계산한다.
5. 금액 범위를 확인한다.
6. `nValueIn < value_out`이면 거부한다.
7. 수수료를 `nValueIn - value_out`로 설정한다.[^ref-btc-core-tx-verify]

### 컨텍스트 비의존 검사

Bitcoin Core의 `CheckTransaction`은 UTXO 세트에 의존하지 않는 검사를 수행한다.

- 입력 벡터는 비어 있으면 안 된다.
- 출력 벡터도 비어 있으면 안 된다.
- 그 검사에서 직렬화 크기는 블록 weight 제약을 넘으면 안 된다.
- 출력 값은 음수일 수 없다.
- 개별 출력 값과 전체 출력 값은 범위 안에 있어야 한다.
- 중복 입력은 거부된다.
- 코인베이스가 아닌 트랜잭션은 null previous outpoint를 사용할 수 없다.
- 코인베이스 script 길이는 제한된다.[^ref-btc-core-tx-check]

이 검사는 UTXO 의존적인 수수료 및 사용 가능성 검사와 별도다.

### 거스름돈 출력

거스름돈은 지갑 구성 패턴이다.

```text
selected input value > payment amount + intended fee
```

지갑은 남는 금액에 대해 새 출력을 만든다. 이 출력은 보통 발신자가 다시 통제한다. 합의는 거스름돈을 표시하지 않는다. 분석가는 script type, 주소 재사용, 출력 금액 패턴, 지갑 동작, 이후 지출 방식 같은 휴리스틱으로 거스름돈을 추론한다.

---

## 8. 수학적 또는 경제적 모델

### 기본 회계

다음을 정의하자.

```text
I = sum(input values)
O = sum(output values)
F = transaction fee
```

그러면:

```text
F = I - O
```

유효성은 다음을 요구한다.

```text
I >= O
F >= 0
```

Bitcoin Core는 또한 `MoneyRange`로 통화 범위를 검사하며, `MAX_MONEY`는 `21000000 * COIN`, `COIN = 100000000` 사토시로 정의된다.[^ref-btc-core-amount]

### 거스름돈 계산

단순한 1회 지급 트랜잭션에서는:

```text
change = selected_input_value - payment_amount - fee
```

예시:

```text
selected input value = 1.000 BTC
payment amount       = 0.700 BTC
fee                  = 0.002 BTC
change               = 0.298 BTC
```

거스름돈 출력은 선택 사항이다. 지갑이 거스름돈을 만들지 않으면, 배정되지 않은 값은 추가 수수료가 된다.

### Fan-Out과 그래프 성장

트랜잭션 그래프는 fan-out될 수 있다.

```text
one transaction
    -> many outputs
    -> many later spends
    -> more outputs
```

또한 fan-in도 가능하다.

```text
many previous outputs
    -> one transaction
```

백서의 요지는 이 조상 이력을 트랜잭션 안에 통째로 복사할 필요가 없다는 것이다. 트랜잭션은 이전 출력을 outpoint로 참조하고, 검증은 노드 상태를 사용해 그 출력이 사용 가능하고 지출 가능한지 확인한다.

---

## 9. 보안 가정

### 이 모델이 요구하는 것

가치의 결합과 분할은 다음에 의존한다.

1. 출력이 트랜잭션 ID와 출력 인덱스로 유일하게 참조될 수 있어야 한다.
2. 풀노드는 미사용 출력에 대한 올바른 뷰를 유지해야 한다.
3. 서명과 스크립트가 참조된 출력의 지출을 정당화해야 한다.
4. 검증이 가치 보존을 강제해야 한다.
5. 네트워크가 동일 출력에 대한 중복 지출을 같은 활성 체인 안에서 거부해야 한다.

### 이 모델이 요구하지 않는 것

이 모델은 다음을 요구하지 않는다.

- 중앙 계정 잔액
- 코인 단위마다 하나의 트랜잭션
- 각 트랜잭션 내부에 포함된 완전한 과거 이력
- 프로토콜 차원의 거스름돈 출력 라벨
- 잔액 계산을 맡는 신뢰된 제3자

### 프라이버시 경계

섹션 9는 실용적 결제를 가능하게 하지만, 동시에 링크 가능성도 만든다. 다중 입력 트랜잭션은 보통 모든 입력에 대한 지출 승인이 필요했기 때문에 같은 주체가 입력들을 통제했음을 시사한다.

하지만 이 역시 합의 사실이 아니라 휴리스틱이다. 협업형 트랜잭션은 의도적으로 여러 주체의 입력을 하나로 합칠 수 있다. 이 점은 백서 섹션 10의 프라이버시 논의에서 핵심이 된다.

---

## 10. Bitcoin Core 구현

### 트랜잭션 데이터 구조

Bitcoin Core의 트랜잭션 primitive는 구현 모델을 다음과 같이 제공한다.

```text
CTransaction
    vin:  vector of CTxIn
    vout: vector of CTxOut
```

`COutPoint`는 트랜잭션 해시와 출력 인덱스를 통해 소비되는 이전 출력을 식별한다. `CTxOut`는 출력 값과 locking script를 저장한다.[^ref-btc-core-transaction]

### `CheckTransaction`

`CheckTransaction`은 UTXO 컨텍스트를 보기 전에 트랜잭션 구조를 검증한다. 빈 입력, 빈 출력, 잘못된 출력 금액, 중복 입력, 코인베이스가 아닌데 null previous outpoint를 가진 경우를 거부한다.[^ref-btc-core-tx-check]

이것은 섹션 9의 모델과 직접 연결된다. 여러 입력과 여러 출력은 허용되지만, 구조적으로는 유효해야 한다.

### `CheckTxInputs`

`CheckTxInputs`는 UTXO 컨텍스트 의존적이다. 이 함수는 입력이 UTXO 뷰에 존재하는지 확인하고, 코인베이스 성숙도를 강제하고, 입력 값 총합을 계산하며, 입력 값이 출력 값보다 작은 트랜잭션을 거부하고, 수수료를 반환한다.[^ref-btc-core-tx-verify]

이것이 일반 트랜잭션에 대한 가치 보존의 구현 수준 강제다.

### 금액 규칙

Bitcoin Core는 금액을 사토시 단위의 부호 있는 64비트 정수인 `CAmount`로 표현한다. `MoneyRange`는 값이 음수가 아니고 `MAX_MONEY`보다 크지 않은지 검사한다.[^ref-btc-core-amount]

이 금액 검사는 overflow와 승인되지 않은 가치 생성을 줄여준다. 그러나 그것이 거스름돈이나 지급 의도를 식별해 주지는 않는다.

---

## 11. 온체인 함의

### 관측 가능한 사실

디코딩된 트랜잭션에서 분석가는 다음을 직접 볼 수 있다.

- 입력 개수
- 입력이 참조하는 previous outpoint
- 출력 개수
- 출력 금액
- 출력 스크립트 유형
- 입력 값을 알고 있다면 거래 수수료
- 전체 직렬화가 있으면 트랜잭션 크기와 weight
- 여러 이전 출력을 결합하는 입력 집합 여부

### 추론 가능한 것

분석가는 다음을 추론할 수 있다.

- 지급 출력으로 보이는 항목
- 거스름돈 출력으로 보이는 항목
- 가능한 공통 입력 통제
- UTXO 통합(consolidation) 행동
- 배칭 행동
- 거래소 또는 커스터디 출금 패턴
- 지갑 행동

이들은 해석이다. 신뢰 수준 라벨과 주의사항이 필요하다.

### 알 수 없는 것

트랜잭션 구조만으로는 보통 다음을 증명할 수 없다.

- 실제 세계의 발신자 신원
- 실제 세계의 수신자 신원
- 어느 출력이 거스름돈인지
- 입력들이 한 사람의 것인지 여러 협력자의 것인지
- 왜 지갑이 특정 입력을 선택했는지
- 추가 증거 없이 그 트랜잭션이 거래소 출금인지, self-transfer인지, 일반 지급인지

---

## 12. 기관 관점에서의 해석

### 왜 이 섹션이 중요한가

섹션 9는 짧지만 분석상 영향이 큰 백서 섹션이다. 이것은 Bitcoin이 계정 원장처럼 동작하지 않는 이유를 설명한다. 사용자는 하나의 가변 잔액에서 지출하는 것이 아니라, 개별적인 이전 출력을 소비하고 새로운 출력을 만든다.

기관에게 이것은 다음에 영향을 준다.

- 입금 attribution
- 지갑 클러스터링
- 트랜잭션 모니터링
- 수수료 회계
- UTXO 관리
- 커스터디 운영
- 프라이버시 리스크
- tax-lot 및 cost-basis 워크플로
- 포렌식 그래프 분석

### 운영 통제

기관은 다음을 수행해야 한다.

- 전체 입력/출력 컨텍스트에서 수수료를 계산하고,
- 모든 2출력 트랜잭션을 지급 + 거스름돈으로 가정하지 말고,
- 다중 입력 클러스터링을 휴리스틱으로 다루며,
- UTXO 파편화와 통합을 추적하고,
- 거스름돈 판별 방법론을 문서화하며,
- 검증된 트랜잭션 사실과 attribution 주장을 분리하고,
- 고가치 분석에는 풀노드 또는 독립 검증된 데이터를 사용해야 한다.

### 리서치 규율

좋은 트랜잭션 분석은 사실에서 시작해야 한다.

```text
input count
output count
input values
output values
scripts
confirmation context
```

그 다음에야 해석으로 넘어가야 한다.

```text
possible change
possible owner cluster
possible exchange behavior
possible consolidation
```

이 분리는 흔한 분석 과잉 해석을 막아 준다.

---

## 13. 흔한 오해

### Misinterpretation 1: Bitcoin has account balances like a bank database.

틀렸다. Bitcoin은 사용 가능한 출력(UTXO)을 사용한다. 지갑 잔액은 특정 키나 스크립트가 통제하는 UTXO를 합산해 계산한다.

### Misinterpretation 2: Change outputs are marked on-chain.

틀렸다. 거스름돈은 트랜잭션 구성과 이후 행동을 보고 추론하는 것이다. 특별한 합의 플래그는 없다.

### Misinterpretation 3: Every two-output transaction has one payment and one change output.

과장이다. 많은 트랜잭션이 այդ 패턴을 따르지만, 배칭, 협업형 트랜잭션, self-transfer, 지갑별 동작 때문에 다른 의미를 가질 수 있다.

### Misinterpretation 4: Multi-input transactions prove one owner.

과장이다. 다중 입력 트랜잭션은 흔히 공통 통제를 시사하지만, 협업형 지출은 이 휴리스틱을 깨뜨릴 수 있다.

### Misinterpretation 5: Fees are an explicit output paid to miners.

틀렸다. 수수료는 입력 값과 출력 값 차이 중 배정되지 않은 부분이다. 채굴자는 블록의 코인베이스 트랜잭션을 통해 전체 수수료를 청구한다.

### Misinterpretation 6: Fan-out requires each transaction to contain its full history.

틀렸다. 트랜잭션은 이전 출력을 참조한다. 노드는 검증된 상태와 이전 블록 데이터를 사용하지, 각 트랜잭션 안에 완전한 이력 번들을 넣지 않는다.

---

## 14. 연구 질문

1. Bitcoin 트랜잭션은 얼마나 자주 1입력 2출력 패턴을 사용하는가?
2. 흔히 쓰이는 거스름돈 출력 휴리스틱은 script type별로 얼마나 신뢰할 수 있는가?
3. 배칭은 출력 개수 해석을 어떻게 바꾸는가?
4. 거래소와 self-custody 지갑은 입력/출력 패턴에서 어떻게 다른가?
5. UTXO 통합은 수수료 시장 상황에 따라 어떻게 반응하는가?
6. 다중 입력 클러스터링은 어떤 false positive를 만드는가?
7. 협업형 트랜잭션은 소유권 휴리스틱을 어떻게 약화시키는가?
8. 기관은 거스름돈 판별 신뢰 수준을 어떻게 점수화해야 하는가?
9. 출력 분할은 미래 수수료 부담에 어떤 영향을 주는가?
10. 어떤 트랜잭션 그래프 지표가 운영용 지갑 행동을 가장 잘 식별하는가?

---

## 15. Practical Exercises

1. 어떤 트랜잭션이 `0.4 BTC`, `0.3 BTC`, `0.2 BTC` 입력을 소비하고 `0.5 BTC`, `0.39 BTC` 출력을 만든다면 수수료를 계산하라.
2. 두 번째 출력이 왜 거스름돈일 수는 있지만, 값만으로 거스름돈이라고 증명할 수는 없는지 설명하라.
3. 트랜잭션 하나를 디코딩하고 각 입력의 previous outpoint를 식별하라.
4. 다섯 개 입력을 가진 트랜잭션이 왜 여전히 유효할 수 있는지 설명하라.
5. 코인베이스 보상 규칙 아래의 예외가 아니라면, 출력 값이 입력 값보다 큰 트랜잭션이 왜 무효인지 설명하라.
6. 다중 입력 소유권 휴리스틱이 실패할 수 있는 사례를 하나 들어라.

---

## 16. 증거 분류

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Whitepaper Section 9 on combining and splitting value | A |
| REF-BTC-DEV-TRANSACTIONS-001 | Official developer documentation | Raw transaction format, inputs, outputs, outpoints, and value rules | A |
| REF-BTC-CORE-TRANSACTION-001 | Primary implementation source | `COutPoint`, `CTxIn`, `CTxOut`, and `CTransaction` structures | A |
| REF-BTC-CORE-TX-CHECK-001 | Primary implementation source | `CheckTransaction` context-free transaction checks | A |
| REF-BTC-CORE-TX-VERIFY-001 | Primary implementation source | `CheckTxInputs`, UTXO checks, and fee calculation | A |
| REF-BTC-CORE-AMOUNT-001 | Primary implementation source | `CAmount`, `COIN`, `MAX_MONEY`, and `MoneyRange` | A |

### Claim Ledger

| Claim ID | Claim | Classification | Evidence | Source |
|---|---|---|---|---|
| C001 | Whitepaper Section 9 states that transactions can contain multiple inputs and outputs to combine and split value. | FACT | A | REF-BTC-WP-001 |
| C002 | The whitepaper describes a normal pattern of one payment output and one change output. | FACT | A | REF-BTC-WP-001 |
| C003 | Modern Bitcoin transactions can contain variable numbers of inputs and outputs. | FACT | A | REF-BTC-DEV-TRANSACTIONS-001; REF-BTC-CORE-TRANSACTION-001 |
| C004 | Non-coinbase inputs reference previous outpoints. | FACT | A | REF-BTC-DEV-TRANSACTIONS-001; REF-BTC-CORE-TRANSACTION-001 |
| C005 | Bitcoin Core rejects duplicate transaction inputs. | FACT | A | REF-BTC-CORE-TX-CHECK-001 |
| C006 | Bitcoin Core rejects non-coinbase transactions with null previous outpoints. | FACT | A | REF-BTC-CORE-TX-CHECK-001 |
| C007 | Bitcoin Core rejects ordinary transactions whose input value is below output value. | FACT | A | REF-BTC-CORE-TX-VERIFY-001 |
| C008 | Bitcoin Core computes transaction fee as input value minus output value. | FACT | A | REF-BTC-CORE-TX-VERIFY-001 |
| C009 | Change output identification is heuristic rather than a consensus fact. | INTERPRETATION | B | REF-BTC-WP-001; REF-BTC-DEV-TRANSACTIONS-001 |
| C010 | Multi-input common ownership is a useful but fallible heuristic. | INTERPRETATION | B | REF-BTC-WP-001 |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| HEURISTIC | Practical analysis rule requiring caveats |
| UNKNOWN | Evidence is insufficient |

---

## 17. 지식 그래프

```text
BITCOIN-010 Combining and Splitting Value
|
+-- interprets: Whitepaper Section 9
|
+-- transaction
|   +-- contains: inputs
|   +-- contains: outputs
|   +-- computes: fee = inputs - outputs
|
+-- combining
|   +-- uses: multiple previous outputs
|   +-- enables: larger payment from smaller UTXOs
|
+-- splitting
|   +-- creates: multiple new outputs
|   +-- enables: payment + change
|
+-- validation
|   +-- CheckTransaction
|   +-- CheckTxInputs
|   +-- MoneyRange
|
+-- analysis_risks
    +-- change detection is heuristic
    +-- multi-input ownership is heuristic
    +-- fan-out does not require full embedded history
```

---

## 18. 참고문헌

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Section 9, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-transactions]: Bitcoin Developer Documentation, "Transactions," raw transaction format, transaction inputs and outputs, outpoints, and coinbase input exception, https://developer.bitcoin.org/reference/transactions.html, accessed 2026-08-04.

[^ref-btc-core-transaction]: Bitcoin Core Contributors, `src/primitives/transaction.h`, `COutPoint`, `CTxIn`, `CTxOut`, and `CTransaction`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/primitives_2transaction_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-tx-check]: Bitcoin Core Contributors, `src/consensus/tx_check.cpp`, `CheckTransaction`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/tx__check_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-tx-verify]: Bitcoin Core Contributors, `src/consensus/tx_verify.cpp`, `CheckTxInputs`, input availability, coinbase maturity, input/output value checks, and fee calculation, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/tx__verify_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-amount]: Bitcoin Core Contributors, `src/consensus/amount.h`, `CAmount`, `COIN`, `MAX_MONEY`, and `MoneyRange`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/amount_8h_source.html, accessed 2026-08-04.

---

## 19. 교차 참조

### Parent

- Bitcoin Whitepaper

### Previous

- BITCOIN-009 — Whitepaper Section 8 — Simplified Payment Verification

### Next

- BITCOIN-011 — Whitepaper Section 10 — Privacy

### Related

- BITCOIN-003 — Whitepaper Section 2 — Transactions
- BITCOIN-007 — Whitepaper Section 6 — Incentive
- BITCOIN-014 — UTXO Model
- BITCOIN-015 — Transactions in Depth
- BITCOIN-017 — Mempool
- BITCOIN-026 — Fee Market
- POW-009 — Coinbase Transaction and Mining Commitments

---

## Review Status

### Technical Review

Passed.

- Whitepaper Section 9와 현재 트랜잭션 구현 세부 사항을 분리했다.
- 입력 결합, 출력 분할, 거스름돈, 수수료 계산, fan-out을 분리했다.
- `COutPoint`, `CTxIn`, `CTxOut`, `CTransaction`, `CheckTransaction`, `CheckTxInputs`, `MoneyRange`를 Bitcoin Core 소스와 대조했다.
- 거스름돈 출력과 다중 입력 소유권을 합의 라벨이 아닌 휴리스틱으로 설명했다.

### Evidence Review

Passed.

- Whitepaper Section 9 관련 주장은 백서를 직접 인용한다.
- 트랜잭션 포맷 관련 주장은 공식 Bitcoin Developer 문서를 인용한다.
- 현재 구현 관련 주장은 Bitcoin Core 소스를 인용한다.
- 온체인 attribution과 거스름돈 판별 관련 주장은 해석 또는 휴리스틱으로 표시했다.

### Editorial Review

Passed.

- Markdown 헤딩은 프로젝트 딥다이브 구조를 따른다.
- 메타데이터는 완전하다.
- 표와 코드 펜스는 닫혀 있다.
- input, output, outpoint, UTXO, change, fee, fan-out 용어를 일관되게 사용했다.

### Adversarial Review

Passed.

- 거스름돈이 온체인에 명시적으로 표시된다고 암시하지 않았다.
- 다중 입력 소유권을 증명으로 취급하지 않았다.
- 백서의 "at most two outputs"를 합의 한도로 오해하지 않도록 했다.
- 일반 트랜잭션의 가치 보존과 코인베이스 보상 생성을 구분했다.

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
