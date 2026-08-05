---
knowledge_id: BITCOIN-014
title: UTXO 모델
subtitle: 미사용 출력, Outpoint, Chainstate, 트랜잭션 검증, 수수료, 그리고 온체인 분석
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 95 min
estimated_study: 260 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - UTXO
  - Transactions
  - Validation
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-003
  - BITCOIN-010
  - BITCOIN-013
  - POW-009
related_topics:
  - Transaction Inputs
  - Transaction Outputs
  - Outpoints
  - Chainstate
  - CCoinsView
  - Fee Calculation
  - Double Spend Prevention
  - Coinbase Maturity
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-TRANSACTIONS-001
  - REF-BTC-DEV-BLOCKCHAIN-001
  - REF-BTC-CORE-TRANSACTION-001
  - REF-BTC-CORE-AMOUNT-001
  - REF-BTC-CORE-COINS-001
  - REF-BTC-CORE-TX-CHECK-001
  - REF-BTC-CORE-TX-VERIFY-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-VALIDATION-H-001
tags:
  - bitcoin
  - internals
  - utxo
  - transaction-validation
  - chainstate
  - coinsview
  - outpoint
---

# UTXO 모델
> Bitcoin Internals  
> Research Unit: BITCOIN-014

---

## Research Brief

```yaml
knowledge_id: BITCOIN-014
title: UTXO Model
research_question: >
  How does Bitcoin represent spendable value as unspent transaction outputs,
  how do full nodes update that set during block validation, and what does
  the UTXO model imply for fees, double-spend prevention, privacy, and
  institutional on-chain analysis?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-003
  - BITCOIN-010
  - BITCOIN-013
  - POW-009
parent: Bitcoin Internals
previous: BITCOIN-013
next: BITCOIN-015
related_topics:
  - Transaction Structure
  - Coinbase Transaction
  - Chainstate
  - Transaction Fees
  - Mempool
  - Script Validation
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
  - Complete script execution semantics
  - Full mempool policy
  - Wallet coin-selection algorithms
  - Complete database-engine internals
  - Non-Bitcoin account-model comparison beyond brief contrast
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- UTXO를 정확히 정의할 수 있다.
- 트랜잭션 출력과 미사용 트랜잭션 출력의 차이를 설명할 수 있다.
- outpoint가 무엇인지 설명할 수 있다.
- 트랜잭션이 오래된 UTXO를 소비하고 새로운 UTXO를 만드는 방식을 설명할 수 있다.
- 왜 Bitcoin이 합의 수준에서 가변 계정 잔액을 사용하지 않는지 설명할 수 있다.
- 입력 값이 거래 수수료를 어떻게 결정하는지 설명할 수 있다.
- UTXO 모델이 하나의 활성 체인 안에서 이중지불을 어떻게 막는지 설명할 수 있다.
- 왜 코인베이스 출력에 특별한 maturity 규칙이 있는지 설명할 수 있다.
- Bitcoin Core가 UTXO 세트를 어떻게 표현하고 조회하는지 식별할 수 있다.
- 합의 수준의 UTXO 사실과 wallet balance, address cluster, 분석 휴리스틱을 구분할 수 있다.

---

## 2. 핵심 질문

1. UTXO란 무엇인가?
2. UTXO는 어떻게 식별되는가?
3. `txid:vout`와 주소의 차이는 무엇인가?
4. 트랜잭션은 이전 출력을 어떻게 소비하는가?
5. 트랜잭션은 새로운 출력을 어떻게 만드는가?
6. 왜 풀노드는 UTXO 세트가 필요한가?
7. 거래 수수료는 UTXO 값으로부터 어떻게 계산되는가?
8. 하나의 활성 체인 안에서 이중지불이 무효가 되는 이유는 무엇인가?
9. 코인베이스로 생성된 출력은 무엇이 다른가?
10. Bitcoin Core는 `Coin`과 `CCoinsView`에 무엇을 저장하는가?
11. 분석가는 UTXO 데이터로부터 무엇을 추론할 수 있는가?
12. UTXO 데이터만으로는 무엇을 추론할 수 없는가?

---

## 3. Executive Summary

Bitcoin의 UTXO 모델은 지출 가능한 가치를 이산적인 미사용 트랜잭션 출력으로 표현한다. 하나의 트랜잭션은 하나 이상의 이전 출력을 소비하고, 하나 이상의 새 출력을 만든다. 현재 미사용 출력들의 집합이 바로 풀노드가 미래 지출을 검증할 때 사용하는 상태다.[^ref-btc-dev-transactions]

핵심 규칙은 다음과 같다.

```text
inputs reference existing unspent outputs
    -> transaction proves spending authority
    -> old outputs are marked spent
    -> new outputs become spendable
```

이것은 account model과 다르다. Bitcoin 합의는 각 사용자에 대한 가변 balance field를 유지하지 않는다. 지갑 잔액은 지갑의 키나 스크립트가 통제하는 UTXO 집합에서 파생된다.

어떤 출력이 영원히 UTXO인 것은 아니다. 활성 체인에서 아직 사용되지 않았을 때만 UTXO다. 일단 사용되면 spendable set에서 제거된다. reorganization으로 블록이 분리되면, 이전에 사용된 출력이 복원될 수 있고 분리된 트랜잭션이 만든 출력은 제거될 수 있다.

Bitcoin Core는 이를 transaction primitive, `Coin`, `CCoinsView`, `CCoinsViewCache`, validation function으로 모델링한다. Doxygen은 `CCoinsView`를 open transaction output dataset에 대한 view로 설명하며, `GetCoin`, `HaveCoin`, `BatchWrite` 같은 메서드를 포함한다고 문서화한다.[^ref-btc-core-coins] Bitcoin Core의 `CheckTxInputs`는 입력이 사용 가능한지 확인하고, 코인베이스 maturity를 강제하고, 입력 값이 출력 값보다 작지 않은지 확인하며, 수수료를 계산한다.[^ref-btc-core-tx-verify]

분석가에게 UTXO 데이터는 핵심 증거다. 그것은 수수료 계산, 지출 추적, 통합 분석, change heuristic, age 분석, 공급 상태 분석을 가능하게 한다. 하지만 ownership, identity, intent, wallet control은 추가 증거 없이는 해석에 머문다.

---

## 4. 원래 설계

Bitcoin 백서는 코인을 디지털 서명의 체인으로 소개한다. 각 소유자는 이전 트랜잭션 해시와 다음 소유자의 공개키에 서명함으로써 가치를 이전한다. 하지만 수취인은 이전 소유자가 그 코인을 이중지불하지 않았는지 검증할 방법이 필요하다고 백서는 지적한다.[^ref-btc-wp]

"UTXO"라는 용어 자체는 백서 문장에 중심적으로 등장하지 않지만, 모델은 섹션 2와 9에서 자연스럽게 도출된다.

- 트랜잭션은 이전 트랜잭션 출력을 참조한다.
- 가치는 여러 입력으로 결합될 수 있다.
- 가치는 여러 출력으로 분할될 수 있다.
- 미사용 출력이 지출 가능한 상태를 나타낸다.[^ref-btc-wp]

현대 Bitcoin 문서와 구현은 이 활성 지출 가능 집합을 설명할 때 UTXO라는 용어를 사용한다.

---

## 5. 프로토콜 구조

### UTXO 정의

UTXO는 다음과 같다.

```text
a transaction output
that exists in the active chain
and has not yet been spent
```

이것은 outpoint로 식별된다.

```text
outpoint = transaction id + output index
```

Bitcoin Core는 이를 `COutPoint`로 표현하며, 트랜잭션 입력은 `CTxIn`으로 이전 outpoint를 참조하고, 트랜잭션 출력은 `CTxOut`으로 값과 locking script를 담는다.[^ref-btc-core-transaction]

### 트랜잭션 상태 전이

코인베이스가 아닌 트랜잭션은 다음과 같은 상태 전이를 수행한다.

```text
before:
    UTXO set contains old outputs A, B, C

transaction:
    spends A and B
    creates D and E

after:
    A and B removed
    C remains
    D and E added
```

이 때문에 Bitcoin의 상태는 계정 잔액 목록이 아니라 spendable output의 집합이다.

### 코인베이스 예외

코인베이스 트랜잭션은 이전 출력을 소비하지 않고 새 출력을 만든다. 그것은 보조금 + 수수료 한도에 묶이며, 지출 전 maturity 규칙을 따라야 한다. Bitcoin Core의 트랜잭션 및 검증 경로는 코인베이스를 별도로 다룬다.[^ref-btc-core-tx-check][^ref-btc-core-tx-verify]

---

## 6. 기술적 메커니즘

### 출력 지출

어떤 UTXO를 지출하려면, 트랜잭션 입력은 다음을 만족해야 한다.

1. 이전 출력을 outpoint로 참조해야 한다.
2. 이전 출력의 스크립트가 요구하는 unlocking data 또는 witness data를 제공해야 한다.
3. 컨텍스트 비의존 트랜잭션 검사를 통과해야 한다.
4. UTXO 가용성 및 값 검사를 통과해야 한다.
5. 해당 플래그 아래 script validation을 통과해야 한다.

이 문서는 UTXO 가용성과 value accounting에 초점을 둔다. script validation은 이후 트랜잭션 및 스크립트 문서에서 다룬다.

### 수수료 회계

수수료는 UTXO 값에서 도출된다.

```text
fee = sum(input UTXO values) - sum(new output values)
```

Bitcoin Core의 `CheckTxInputs`는 `nValueIn`을 계산하고, 이를 output value와 비교하며, 입력 값이 출력 값보다 작으면 거부하고, 차액을 수수료로 반환한다.[^ref-btc-core-tx-verify]

### 이중지불 방지

하나의 활성 체인 안에서 어떤 출력은 한 번만 지출될 수 있다. 트랜잭션이 특정 outpoint를 지출하면, 그 출력은 UTXO 세트에서 제거된다. 이후 다른 트랜잭션이 같은 outpoint를 다시 지출하려 하면, 더 이상 사용 가능한 coin이 view에 없기 때문에 실패한다.

이것은 chainstate 위에서 이루어지는 로컬 검증 규칙이지, 전역적인 account-locking service가 아니다.

### Reorganization 동작

reorg가 중요한 이유는 UTXO 세트가 활성 체인에 묶여 있기 때문이다. 블록이 분리되면 그 효과를 되돌려야 한다.

- 분리된 트랜잭션이 만든 출력은 제거되고,
- 분리된 트랜잭션이 소비했던 출력은 undo data를 통해 복원되며,
- chainstate는 새 active tip을 반영해야 한다.

Bitcoin Developer 문서는 블록체인을 풀노드가 검증하는 순서 있는 트랜잭션 이력으로 설명하고, Bitcoin Core는 validation interface에서 `DisconnectBlock`과 `ConnectBlock`을 `CCoinsViewCache` 위에서 작동하는 블록 분리/연결 연산으로 노출한다.[^ref-btc-dev-blockchain][^ref-btc-core-validation-h]

---

## 7. 수학적 또는 경제적 모델

### 파생 상태로서의 잔액

지갑 맥락에서:

```text
wallet balance = sum(UTXO values controlled by wallet keys/scripts)
```

이것은 wallet-level calculation이지 합의 필드가 아니다.

### 트랜잭션 유효성 회계

코인베이스가 아닌 트랜잭션에 대해:

```text
I = sum(input UTXO values)
O = sum(output values)
F = I - O
```

합의 유효성은 다음을 요구한다.

```text
I >= O
F >= 0
```

금액은 범위 안에도 있어야 한다. Bitcoin Core는 `CAmount`, `COIN = 100000000`, `MAX_MONEY = 21000000 * COIN`, `MoneyRange`를 정의한다.[^ref-btc-core-amount]

### UTXO 파편화

지갑은 많은 작은 UTXO를 가질 수 있다.

```text
100 outputs * 0.001 BTC = 0.100 BTC wallet-controlled value
```

많은 작은 UTXO를 한 번에 쓰면 트랜잭션 크기와 수수료 부담이 커질 수 있다. 반대로 UTXO를 통합하면 미래의 입력 개수를 줄일 수 있지만, 여러 입력을 한 번에 지출하므로 프라이버시를 약화시킬 수 있다.

이것은 경제적·프라이버시적 트레이드오프이지 합의 규칙은 아니다.

---

## 8. 보안 가정

### UTXO 모델이 가정하는 것

UTXO 모델은 다음에 의존한다.

1. 트랜잭션 ID와 출력 인덱스가 이전 출력을 유일하게 식별한다.
2. 풀노드가 올바른 active-chain UTXO set을 유지한다.
3. 입력은 UTXO 세트에 없는 출력을 지출할 수 없다.
4. 스크립트가 참조된 출력의 지출을 승인한다.
5. 일반 트랜잭션은 입력이 출력을 커버해야 하므로 가치를 임의로 만들 수 없다.
6. reorg 처리 시 UTXO 상태 변경이 올바르게 되돌려지고 다시 적용된다.

### 이것이 증명하지 않는 것

UTXO 검증은 다음을 증명하지 않는다.

- 실제 세계 소유권
- 트랜잭션 의도
- 지갑 신원
- 거래소 고객 신원
- 어떤 출력이 거스름돈인지
- 여러 입력이 한 사람의 것인지
- 어떤 트랜잭션이 state transition 외에 경제적으로 어떤 의미를 갖는지

이들은 분석적 추론이다.

---

## 9. Bitcoin Core 구현

### 트랜잭션 Primitive

Bitcoin Core의 `src/primitives/transaction.h`는 다음을 정의한다.

| Structure | Role |
|---|---|
| `COutPoint` | 트랜잭션 해시와 출력 인덱스로 이전 출력을 식별 |
| `CTxIn` | 이전 outpoint를 참조하는 트랜잭션 입력 |
| `CTxOut` | 값과 locking script를 담는 트랜잭션 출력 |
| `CTransaction` | `vin`과 `vout`를 포함하는 immutable transaction object |

이 구조들이 UTXO 지출과 생성 표현의 기초다.[^ref-btc-core-transaction]

### Coin과 Coins View

Bitcoin Core의 `coins.h`는 `Coin`과 `CCoinsView` 계층을 정의한다. `CCoinsView`는 open transaction output dataset에 대한 view로 문서화되며 `GetCoin`, `PeekCoin`, `HaveCoin`, `GetBestBlock`, `BatchWrite` 같은 메서드를 제공한다.[^ref-btc-core-coins]

`validation.h`의 `CoinsViews` helper는 계층화된 view hierarchy를 구성한다. Doxygen은 가장 아래 레벨을 디스크상의 LevelDB 데이터베이스로 설명하고, 가장 위 캐시는 cache setting이 허용하는 만큼의 coin을 메모리에 보유한다고 설명한다.[^ref-btc-core-validation-h]

### `CheckTransaction`

`CheckTransaction`은 컨텍스트 비의존 트랜잭션 검사를 수행한다. 빈 입력/출력 벡터, 잘못된 출력 값, 중복 입력, 코인베이스가 아닌데 null previous outpoint를 쓰는 경우, 잘못된 코인베이스 script size 범위를 거부한다.[^ref-btc-core-tx-check]

이 검사는 참조된 출력이 실제로 미사용인지 여부를 알 필요가 없다.

### `CheckTxInputs`

`CheckTxInputs`는 UTXO 컨텍스트 검사를 수행한다. 이 함수는 입력이 view에 존재하는지 확인하고, 코인베이스 maturity를 강제하며, 입력/출력 값 일관성을 확인하고, 거래 수수료를 계산한다.[^ref-btc-core-tx-verify]

이것이 UTXO 세트가 실제로 필요한 핵심 검증 경계다.

### `ConnectBlock`

`ConnectBlock`은 블록을 연결할 때 유효한 블록 트랜잭션의 효과를 UTXO view에 적용한다. 이것은 chainstate의 coins view와 함께 동작하며 active-chain validation path의 일부다.[^ref-btc-core-validation]

내부 구조는 Bitcoin Core 버전에 따라 바뀔 수 있지만, 합의 호환 구현은 반드시 동일한 유효 상태 전이 결과를 내야 한다.

---

## 10. 온체인 함의

### 관측 가능한 사실

분석가는 다음을 직접 관측할 수 있다.

- transaction output
- output value
- output script
- input outpoint
- 특정 출력이 활성 체인에서 이후 지출되었는지 여부
- 블록 높이나 시간을 기준으로 한 UTXO age
- 입력 개수와 출력 개수
- consolidation 및 fan-out 패턴

### 파생 지표

자주 쓰이는 UTXO 파생 지표는 다음과 같다.

| Metric | Description |
|---|---|
| UTXO count | 현재 미사용 출력 수 |
| Realized output age | 출력이 생성된 이후 경과 시간 |
| Coin days destroyed | 지출 시점의 값 x 나이 |
| Consolidation ratio | 많은 입력에서 적은 출력으로 |
| Fan-out ratio | 적은 입력에서 많은 출력으로 |
| Fee rate context | 트랜잭션 weight 및 입출력 구조 대비 수수료 |

이 지표는 방법론을 신중히 다뤄야 한다. 데이터 제공자마다 필터, 엔터티 조정, dust 처리 방식이 다를 수 있다.

### 추론의 한계

UTXO 데이터만으로는 다음을 증명할 수 없다.

- 누가 출력을 소유하는지
- 어떤 출력이 거스름돈인지
- 어떤 지출이 매도인지
- 어떤 통합이 단일 사용자 것인지 수탁기관 것인지
- 어떤 출력이 영구 손실되었는지
- 장기 dormant 상태가 conviction을 뜻하는지

이들은 해석이므로 라벨링해야 한다.

---

## 11. 기관 관점에서의 해석

### 왜 UTXO 이해가 중요한가

기관 Bitcoin 리서치는 UTXO 이해에 의존한다. UTXO는 다음의 기반이기 때문이다.

- 수수료 계산
- 공급 상태
- 트랜잭션 추적
- 지갑 운영
- 커스터디 통제
- proof-of-reserves 설계
- 거래소 플로우 해석
- dormant supply metric
- realized-cap 계열 지표
- 프라이버시 및 클러스터링 분석

### 리서치 워크플로 통제

리서치 시스템은 다음을 해야 한다.

- 눈에 보이는 출력만이 아니라 이전 출력 값에서 수수료를 계산하고,
- 미사용 출력과 주소를 구분하며,
- 어떤 chain tip을 기준으로 UTXO 상태를 봤는지 기록하고,
- reorg를 명시적으로 처리하며,
- 합의 유효성과 mempool policy를 분리하고,
- change와 clustering을 휴리스틱으로 라벨링하며,
- wallet balance를 온체인 필드로 취급하지 않고,
- 고위험 분석은 풀노드 또는 재현 가능한 데이터셋으로 교차 검증해야 한다.

### 커스터디와 운영

수탁자와 기관에게 UTXO 관리는 다음에 직접 영향을 준다.

- 출금 배칭
- 수수료 노출
- 통합 시점
- 프라이버시 유출
- 서명 복잡성
- dust 관리
- proof-of-reserves 출력 선택
- 복구 및 회계 워크플로

따라서 UTXO 관리는 프로토콜 관심사이자 운영 규율이다.

---

## 12. 흔한 오해

### Misinterpretation 1: Bitcoin stores account balances.

틀렸다. Bitcoin 합의는 spendable output을 추적한다. wallet balance는 통제되는 UTXO에서 파생된다.

### Misinterpretation 2: An address is a UTXO.

틀렸다. 하나의 주소나 스크립트는 여러 출력을 받을 수 있다. 각 출력은 outpoint로 식별되는 별개의 객체다.

### Misinterpretation 3: A transaction spends an address.

틀렸다. 트랜잭션 입력은 이전 출력을 지출한다. 주소는 사용자 친화적 인코딩 또는 추상화다.

### Misinterpretation 4: Change outputs are consensus-labeled.

틀렸다. change는 지갑 행동이며 분석가가 추론하는 것이다. 합의는 표시하지 않는다.

### Misinterpretation 5: UTXO age proves investor intent.

틀렸다. dormancy는 관측 가능하지만 intent는 해석이다.

### Misinterpretation 6: A reorg only changes headers.

틀렸다. 활성 체인이 바뀌면 UTXO 상태도 분리된 블록과 새로 연결된 블록을 반영해야 한다.

---

## 13. 연구 질문

1. Bitcoin의 UTXO count는 서로 다른 fee regime에 따라 어떻게 변해왔는가?
2. 수탁기관은 높은 수수료 환경에서 UTXO consolidation을 어떻게 관리하는가?
3. 어떤 UTXO age metric이 거래소/수탁기관 행동에 가장 덜 왜곡되는가?
4. 기관 데이터셋에서 dust output은 어떻게 처리해야 하는가?
5. script type 변화는 UTXO 분석에 어떤 영향을 주는가?
6. modern wallet type 전반에서 change heuristic은 얼마나 신뢰할 수 있는가?
7. reorg는 UTXO 기반 지표에서 어떻게 표현해야 하는가?
8. UTXO fragmentation은 거래 수수료 노출에 어떤 영향을 주는가?
9. 어떤 UTXO 패턴이 거래소, 채굴자, 수탁기관, 개인 지갑을 구분하는가?
10. 풀노드 기반 UTXO 상태는 제3자 analytics 데이터와 어떻게 reconcile할 수 있는가?

---

## 14. Practical Exercises

1. `0.10 BTC`, `0.25 BTC`, `0.40 BTC`의 세 UTXO를 사용해 `0.50 BTC`를 지급하고 `0.002 BTC` 수수료를 내는 트랜잭션을 구성하라. change를 계산하라.
2. 다섯 개의 수신 출력을 가진 주소가 왜 하나의 온체인 balance object를 갖는 것이 아닌지 설명하라.
3. 어떤 transaction input이 주어졌을 때 그것이 소비하는 previous outpoint를 식별하라.
4. 코인베이스의 허용된 청구가 아닌 한, 출력 값이 입력 값보다 크면 왜 무효인지 설명하라.
5. reorg가 이전에 사용된 UTXO를 어떻게 복원할 수 있는지 설명하라.
6. "this output is change"를 fact, interpretation, heuristic, unknown 중 하나로 분류하라.

---

## 15. 증거 분류

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Transaction chain, combining/splitting value, and double-spend problem | A |
| REF-BTC-DEV-TRANSACTIONS-001 | Official developer documentation | Transaction inputs, outputs, outpoints, and coinbase exception | A |
| REF-BTC-DEV-BLOCKCHAIN-001 | Official developer documentation | Validation, forks, and chain behavior context | A |
| REF-BTC-CORE-TRANSACTION-001 | Primary implementation source | `COutPoint`, `CTxIn`, `CTxOut`, `CTransaction` | A |
| REF-BTC-CORE-AMOUNT-001 | Primary implementation source | `CAmount`, `COIN`, `MAX_MONEY`, `MoneyRange` | A |
| REF-BTC-CORE-COINS-001 | Primary implementation source | `Coin`, `CCoinsView`, `CCoinsViewCache`, UTXO set access | A |
| REF-BTC-CORE-TX-CHECK-001 | Primary implementation source | Context-free transaction checks | A |
| REF-BTC-CORE-TX-VERIFY-001 | Primary implementation source | UTXO-dependent input validation and fee calculation | A |
| REF-BTC-CORE-VALIDATION-001 | Primary implementation source | `ConnectBlock` and block connection state transitions | A |
| REF-BTC-CORE-VALIDATION-H-001 | Primary implementation source | `ConnectBlock`, `DisconnectBlock`, and `CoinsViews` declarations | A |

### Claim Ledger

| Claim ID | Claim | Classification | Evidence | Source |
|---|---|---|---|---|
| C001 | Bitcoin transactions consume previous outputs and create new outputs. | FACT | A | REF-BTC-WP-001; REF-BTC-DEV-TRANSACTIONS-001 |
| C002 | A UTXO is a transaction output that remains unspent in the active chain. | FACT | A | REF-BTC-DEV-TRANSACTIONS-001; REF-BTC-CORE-COINS-001 |
| C003 | Bitcoin Core represents previous-output references with `COutPoint`. | FACT | A | REF-BTC-CORE-TRANSACTION-001 |
| C004 | `CCoinsView` is a view on the open transaction output dataset. | FACT | A | REF-BTC-CORE-COINS-001 |
| C005 | `CheckTransaction` performs context-free checks such as rejecting duplicate inputs and invalid output values. | FACT | A | REF-BTC-CORE-TX-CHECK-001 |
| C006 | `CheckTxInputs` checks UTXO availability and computes fees from input value minus output value. | FACT | A | REF-BTC-CORE-TX-VERIFY-001 |
| C007 | `ConnectBlock` applies block transaction effects to a coins view. | FACT | A | REF-BTC-CORE-VALIDATION-001; REF-BTC-CORE-VALIDATION-H-001 |
| C008 | Wallet balance is derived from controlled UTXOs rather than stored as a consensus account balance. | INTERPRETATION | B | REF-BTC-DEV-TRANSACTIONS-001; REF-BTC-CORE-COINS-001 |
| C009 | Change detection and ownership clustering are heuristic analyses, not consensus facts. | INTERPRETATION | B | REF-BTC-WP-001 |
| C010 | UTXO management affects institutional fee, privacy, and custody operations. | INTERPRETATION | B | REF-BTC-DEV-TRANSACTIONS-001 |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| HEURISTIC | Practical analysis rule requiring caveats |
| UNKNOWN | Evidence is insufficient |

---

## 16. 지식 그래프

```text
BITCOIN-014 UTXO Model
|
+-- builds_on: BITCOIN-010 Combining and Splitting Value
+-- builds_on: POW-009 Coinbase Transaction
|
+-- UTXO
|   +-- identified_by: outpoint = txid + vout
|   +-- contains: value + locking script
|   +-- state: unspent or spent
|
+-- transaction
|   +-- consumes: existing UTXOs
|   +-- creates: new outputs
|   +-- fee: input values - output values
|
+-- Bitcoin Core
|   +-- transaction primitives
|   +-- Coin
|   +-- CCoinsView
|   +-- CheckTransaction
|   +-- CheckTxInputs
|   +-- ConnectBlock
|
+-- analysis
    +-- facts: outpoints, values, spend status
    +-- heuristics: change, ownership, clustering
    +-- operations: consolidation, batching, custody
```

---

## 17. 참고문헌

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Sections 2 and 9, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-transactions]: Bitcoin Developer Documentation, "Transactions," inputs, outputs, outpoints, raw transaction format, and coinbase input exception, https://developer.bitcoin.org/reference/transactions.html, accessed 2026-08-04.

[^ref-btc-dev-blockchain]: Bitcoin Developer Documentation, "Block Chain," transaction data and chain validation context, https://developer.bitcoin.org/devguide/block_chain.html, accessed 2026-08-04.

[^ref-btc-core-transaction]: Bitcoin Core Contributors, `src/primitives/transaction.h`, `COutPoint`, `CTxIn`, `CTxOut`, and `CTransaction`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/primitives_2transaction_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-amount]: Bitcoin Core Contributors, `src/consensus/amount.h`, `CAmount`, `COIN`, `MAX_MONEY`, and `MoneyRange`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/amount_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-coins]: Bitcoin Core Contributors, `src/coins.h` and `src/coins.cpp`, `Coin`, `CCoinsView`, `CCoinsViewCache`, and UTXO set access methods, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/coins_8h_source.html and https://doxygen.bitcoincore.org/coins_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-tx-check]: Bitcoin Core Contributors, `src/consensus/tx_check.cpp`, `CheckTransaction`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/tx__check_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-tx-verify]: Bitcoin Core Contributors, `src/consensus/tx_verify.cpp`, `CheckTxInputs`, input availability, coinbase maturity, input/output value checks, and fee calculation, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/tx__verify_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-validation]: Bitcoin Core Contributors, `src/validation.cpp`, `ConnectBlock`, `UpdateCoins`, and chainstate block connection logic, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/validation_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-validation-h]: Bitcoin Core Contributors, `src/validation.h`, `ConnectBlock`, `DisconnectBlock`, and `CoinsViews`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/validation_8h_source.html, accessed 2026-08-04.

---

## 18. 교차 참조

### Parent

- Bitcoin Internals

### Previous

- BITCOIN-013 — Whitepaper Section 12 — Conclusion

### Next

- BITCOIN-015 — Transactions in Depth

### Related

- BITCOIN-003 — Whitepaper Section 2 — Transactions
- BITCOIN-007 — Whitepaper Section 6 — Incentive
- BITCOIN-010 — Whitepaper Section 9 — Combining and Splitting Value
- BITCOIN-015 — Transactions in Depth
- BITCOIN-016 — Script & ScriptPubKey
- BITCOIN-017 — Mempool
- BITCOIN-023 — Chain Reorganization
- POW-009 — Coinbase Transaction and Mining Commitments

---

## Review Status

### Technical Review

Passed.

- UTXO, transaction output, outpoint, wallet balance, address를 분리했다.
- 컨텍스트 비의존 트랜잭션 검사와 UTXO 의존 검증을 분리했다.
- `COutPoint`, `CTxIn`, `CTxOut`, `CTransaction`, `Coin`, `CCoinsView`, `CCoinsViewCache`, `CheckTransaction`, `CheckTxInputs`, `ConnectBlock`, `DisconnectBlock` 참조를 Bitcoin Core 소스와 대조했다.
- reorg가 UTXO 상태에 주는 영향을 구현 내부를 과도하게 단정하지 않고 포함했다.

### Evidence Review

Passed.

- 백서 기반 transaction-chain 주장은 백서를 인용한다.
- transaction format과 outpoint 주장은 공식 Bitcoin Developer 문서를 인용한다.
- 현재 구현 관련 주장은 Bitcoin Core Doxygen 소스 참조를 인용한다.
- wallet balance, ownership, change, clustering 문장은 interpretation 또는 heuristic으로 표시했다.

### Editorial Review

Passed.

- Markdown 헤딩은 프로젝트 딥다이브 구조를 따른다.
- 메타데이터는 완전하다.
- 표와 코드 펜스는 닫혀 있다.
- UTXO, outpoint, input, output, chainstate, coins view, wallet balance 용어를 일관되게 사용했다.

### Adversarial Review

Passed.

- Bitcoin을 account-balance ledger로 설명하지 않았다.
- 주소를 UTXO로 취급하지 않았다.
- change나 ownership clustering을 합의 사실로 취급하지 않았다.
- active-chain UTXO 상태와 historical transaction data를 구분했다.

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
