---
knowledge_id: BITCOIN-015
title: 트랜잭션 심화
subtitle: 트랜잭션 직렬화, 입력, 출력, 식별자, Locktime, Witness, 수수료, 그리고 검증 경계
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 105 min
estimated_study: 300 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Transactions
  - Validation
  - Serialization
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-003
  - BITCOIN-010
  - BITCOIN-014
  - POW-009
related_topics:
  - Transaction Inputs
  - Transaction Outputs
  - Transaction Identifiers
  - Segregated Witness
  - Locktime
  - Sequence Locks
  - Fees
  - Mempool Policy
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-TRANSACTIONS-001
  - REF-BIP-0068
  - REF-BIP-0125
  - REF-BIP-0141
  - REF-BIP-0143
  - REF-BIP-0341
  - REF-BTC-CORE-TRANSACTION-001
  - REF-BTC-CORE-TXID-001
  - REF-BTC-CORE-TX-CHECK-001
  - REF-BTC-CORE-TX-VERIFY-001
  - REF-BTC-CORE-INTERPRETER-001
  - REF-BTC-CORE-RBF-001
tags:
  - bitcoin
  - internals
  - transactions
  - txid
  - wtxid
  - locktime
  - sequence
  - segwit
  - fee
---

# 트랜잭션 심화
> Bitcoin Internals  
> Research Unit: BITCOIN-015

---

## Research Brief

```yaml
knowledge_id: BITCOIN-015
title: Transactions in Depth
research_question: >
  What exactly is a Bitcoin transaction, how are its fields serialized and
  identified, how do inputs, outputs, locktime, sequence numbers, witness
  data, fees, and validation boundaries interact, and what should on-chain
  analysts avoid over-inferring from transaction data?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-003
  - BITCOIN-010
  - BITCOIN-014
parent: Bitcoin Internals
previous: BITCOIN-014
next: BITCOIN-016
related_topics:
  - UTXO Model
  - Script
  - Transaction Fees
  - Mempool
  - Segregated Witness
  - Taproot
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Protocol Structure
  - Serialization
  - Technical Mechanics
  - Mathematical or Economic Model
  - Security Assumptions
  - Bitcoin Core Implementation
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Complete Script opcode semantics
  - Wallet coin selection algorithms
  - Full mempool package relay behavior
  - Signature cryptography proofs
  - Non-Bitcoin transaction models except brief contrast
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Bitcoin 트랜잭션의 필드를 설명할 수 있다.
- transaction input과 transaction output을 구분할 수 있다.
- 입력이 outpoint를 통해 이전 출력을 어떻게 참조하는지 설명할 수 있다.
- `txid`와 `wtxid`의 차이를 설명할 수 있다.
- Segregated Witness가 legacy `txid` 정의를 바꾸지 않고 트랜잭션 직렬화를 어떻게 바꾸었는지 설명할 수 있다.
- 수수료가 입력과 출력 값에서 어떻게 파생되는지 설명할 수 있다.
- `nVersion`, `nLockTime`, 입력의 `nSequence`가 맡는 역할을 설명할 수 있다.
- 합의 유효성과 relay policy, wallet behavior를 분리할 수 있다.
- Bitcoin Core 소스에서 트랜잭션을 표현하고 검증하는 영역을 식별할 수 있다.
- 트랜잭션 데이터가 온체인 분석에서 무엇을 증명할 수 있고 없는지 설명할 수 있다.

---

## 2. 핵심 질문

1. 데이터 구조 수준에서 Bitcoin 트랜잭션이란 무엇인가?
2. legacy transaction에는 어떤 필드가 직렬화되는가?
3. witness serialization에는 어떤 추가 필드가 존재하는가?
4. 입력은 실제로 무엇을 지출하는가?
5. 출력은 실제로 무엇을 잠그는가?
6. 왜 주소는 합의 직렬화의 트랜잭션 필드가 아닌가?
7. `txid`와 `wtxid`는 어떻게 계산되는가?
8. locktime과 sequence는 transaction finality와 어떻게 상호작용하는가?
9. 수수료는 어떻게 계산되는가?
10. 어떤 검사는 context-free인가?
11. 어떤 검사는 UTXO context를 필요로 하는가?
12. 어떤 트랜잭션 속성이 policy이지 consensus는 아닌가?

---

## 3. Executive Summary

Bitcoin 트랜잭션은 직렬화된 state-transition instruction이다. 그것은 입력을 통해 기존 UTXO를 소비하고, 금액과 locking script를 가진 새 출력을 만든다. 트랜잭션 자체는 account balance를 담지 않는다. 대신 이전 출력에 대한 참조와 새 출력 정의를 담는다.[^ref-btc-dev-transactions]

기본 legacy transaction 구조는 다음과 같다.

```text
nVersion
txins
txouts
nLockTime
```

Segregated Witness는 witness-bearing transaction에 대해 marker, flag, witness field를 직렬화에 추가한다. BIP141는 legacy `txid`를 non-witness serialization의 double SHA256으로 유지하면서, `wtxid`를 witness serialization의 double SHA256으로 정의한다.[^ref-bip-0141]

트랜잭션 검증은 계층 구조를 가진다. `CheckTransaction`은 비어 있지 않은 입력/출력, 출력 값 범위, 중복 입력 감지, null previous-outpoint 규칙, 코인베이스 스크립트 크기 제한 같은 context-independent check를 수행한다.[^ref-btc-core-tx-check] `CheckTxInputs`는 UTXO-context check를 수행하며, 입력 가용성, 코인베이스 maturity, 입력/출력 값 일관성, 수수료 계산을 다룬다.[^ref-btc-core-tx-verify] 이후 script validation은 각 입력이 이전 출력의 spending condition을 만족하는지 검사한다.[^ref-btc-core-interpreter]

수수료는 명시적 필드가 아니다. 코인베이스가 아닌 트랜잭션에 대해:

```text
fee = sum(input UTXO values) - sum(output values)
```

분석가에게 트랜잭션은 outpoint, value, script type, confirmation status, graph topology에 대한 강한 증거를 제공한다. 그러나 real-world ownership, intent, customer identity, change status, economic purpose까지 그 자체로 증명하지는 못한다.

---

## 4. 프로토콜 구조

### 상태 전이로서의 트랜잭션

Bitcoin의 트랜잭션 모델은 백서의 chain-of-signatures 설계를 따른다. 소유권 이전은 이전 트랜잭션 데이터를 참조하고 다음 이전을 승인하는 방식으로 표현된다.[^ref-btc-wp]

프로토콜 수준에서 코인베이스가 아닌 트랜잭션은 다음을 말한다.

```text
spend these previous outputs
create these new outputs
optionally constrain finality by locktime and sequence
provide authorization material
```

트랜잭션은 참조한 이전 출력이 존재하고, 관련 view에서 미사용 상태이며, 충분한 가치를 가지고 있고, 유효한 승인으로 지출될 때만 유효하다.

### 핵심 필드

Bitcoin Developer 문서가 설명하는 표준 transaction format은 version, input count, inputs, output count, outputs, locktime을 포함한다.[^ref-btc-dev-transactions]

| Field | Meaning | Consensus relevance |
|---|---|---|
| `nVersion` | 트랜잭션 버전 | BIP68 sequence lock 같은 version-gated semantics 활성화 |
| Input count | 입력 수를 나타내는 CompactSize | 유효 트랜잭션은 비어 있을 수 없음 |
| Inputs | previous-output reference + spending data | 소비되는 UTXO와 승인 데이터를 식별 |
| Output count | 출력 수를 나타내는 CompactSize | 비어 있을 수 없음 |
| Outputs | value + locking script | 새로운 spendable output 정의 |
| `nLockTime` | block height 또는 time 기준의 earliest finality | sequence가 허용할 때 finality check에 사용 |

### 입력

코인베이스가 아닌 입력은 다음을 포함한다.

| Component | Meaning |
|---|---|
| Previous outpoint | 지출되는 UTXO의 `txid` + output index |
| ScriptSig | legacy unlocking data 또는 redeem data |
| Sequence | finality, relative-locktime, policy signaling 필드 |
| Witness | SegWit spending data, legacy `txid` 바깥에서 직렬화 |

previous outpoint가 핵심 연결고리다. 이것은 주소를 가리키지 않는다. 정확히 이전 트랜잭션의 특정 출력 하나를 가리킨다.

### 출력

출력은 다음을 포함한다.

| Component | Meaning |
|---|---|
| Value | 출력에 잠기는 사토시 수 |
| ScriptPubKey | 지출 조건을 정의하는 locking script |

Bitcoin Developer 문서는 transaction output을 pubkey script가 제공하는 조건 아래 특정 사토시 수를 잠그는 구조로 설명한다.[^ref-btc-dev-transactions]

주소는 보통 지갑/사용자 인터페이스 수준에서 목적지 또는 script template를 인코딩한 표현이다. 합의 트랜잭션 직렬화의 필드는 아니다.

### 코인베이스 트랜잭션

코인베이스 트랜잭션은 블록의 첫 번째 트랜잭션이다. 입력은 previous outpoint가 없으며 null hash와 `0xffffffff` index를 사용한다. 코인베이스는 합의 한도와 maturity rule 아래에서 블록 보조금과 수집된 수수료를 만든다.[^ref-btc-dev-transactions]

코인베이스는 이전 UTXO를 소비하는 대신 maturity 이후 새 spendable value를 만든다는 점에서 일반 트랜잭션과 분리해서 분석해야 한다.

---

## 5. 직렬화와 식별자

### Legacy Serialization

legacy transaction serialization은 다음에 커밋한다.

```text
[nVersion][txins][txouts][nLockTime]
```

`txid`는 이 직렬화에서 파생되는 transaction identifier다. BIP141는 witness data를 도입하면서도 이 legacy `txid` 정의를 그대로 유지한다.[^ref-bip-0141]

### Witness Serialization

witness transaction에 대해 BIP141는 다음 serialization form을 정의한다.

```text
[nVersion][marker][flag][txins][txouts][witness][nLockTime]
```

marker는 `0x00`, flag는 0이 아닌 값이며 현재는 `0x01`을 사용한다. 각 입력은 대응하는 witness field를 가진다. BIP141는 witness data가 script 자체는 아니라고 명시한다.[^ref-bip-0141]

### `txid`와 `wtxid`

구분은 다음과 같다.

| Identifier | Hashes witness data? | Main use |
|---|---:|---|
| `txid` | No | Legacy transaction identity and outpoint references |
| `wtxid` | Yes | Witness-aware relay, block witness commitment, and malleability-resistant tracking |

Bitcoin Core의 `transaction_identifier` template는 canonical transaction identifier type인 `txid`와 `wtxid`를 표현한다.[^ref-btc-core-txid]

legacy non-witness transaction에서는 witness serialization 차이가 없으므로 `txid`와 `wtxid`가 사실상 동일하다. witness transaction에서는 non-witness serialization이 그대로라면 witness data가 바뀌어도 `txid`는 안 바뀌고 `wtxid`만 바뀐다.

### 트랜잭션 Malleability 경계

SegWit 이전에는 서명 데이터가 `txid`에 해시되는 데이터의 일부였다. 그래서 유효한 서명 인코딩이나 scriptSig 데이터의 일부 변경이 의도된 지출을 바꾸지 않으면서도 transaction identifier를 바꿀 수 있었다. SegWit은 witness data를 legacy `txid` 계산 바깥으로 이동시켜, SegWit 지출에 대해 이 종류의 malleability를 줄였다.

그렇다고 모든 가능한 의미에서 모든 transaction semantics가 완전히 malleability-free가 되었다는 뜻은 아니다. 단지 outpoint reference에 쓰이는 identifier가 SegWit transaction에 대해 witness data에는 더 이상 커밋하지 않는다는 뜻이다.

---

## 6. 기술적 메커니즘

### 트랜잭션 구성

지갑의 transaction construction은 보통 다음 흐름을 따른다.

```text
select UTXOs
choose recipients and amounts
estimate fee
create outputs, including change if needed
set version, sequence, and locktime fields
produce signatures or witness data
broadcast or submit transaction
```

이 대부분은 wallet behavior다. 합의 검증은 최종적으로 만들어진 transaction을 프로토콜 규칙에 비춰 평가할 뿐이다.

### Context-Free Checks

context-free check는 현재 UTXO set을 몰라도 수행할 수 있다. Bitcoin Core의 `CheckTransaction`이 이 범주를 다룬다.[^ref-btc-core-tx-check]

예시는 다음과 같다.

- inputs는 비어 있으면 안 된다.
- outputs도 비어 있으면 안 된다.
- output value는 음수일 수 없다.
- output value는 범위를 벗어나면 안 된다.
- 중복 입력은 거부된다.
- coinbase script size는 합의 범위를 만족해야 한다.
- 코인베이스가 아닌 트랜잭션은 null previous outpoint를 사용할 수 없다.

이 검사는 malformed transaction을 일찍 걸러내지만, spendability까지 증명하지는 않는다.

### UTXO-Context Checks

UTXO-context check는 현재 미사용 출력 뷰가 필요하다. Bitcoin Core의 `CheckTxInputs`는 입력 가용성, 코인베이스 maturity, value consistency, fee calculation을 검증한다.[^ref-btc-core-tx-verify]

이 검사는 다음 질문에 답한다.

- 각 previous output이 실제로 존재하고 아직 미사용 상태인가?
- 입력 값은 출력 값을 커버하기에 충분한가?
- 코인베이스 출력은 지출할 만큼 충분히 오래되었는가?
- 이 트랜잭션은 얼마의 수수료를 지불하는가?

### Script Checks

script validation은 입력이 이전 출력의 locking condition을 만족하는지 확인한다. Bitcoin Core의 script interpreter는 `VerifyScript` 같은 경로를 통해 script evaluation과 verification을 구현한다.[^ref-btc-core-interpreter]

이 Research Unit은 script를 validation boundary로 다룬다. script language, 표준 script template, witness program, Taproot script-path behavior는 BITCOIN-016 이후 문서에서 다룬다.

### Finality Checks

절대 finality는 `nLockTime`을 사용한다. 트랜잭션은 특정 block height 또는 time이 되기 전까지 final이 아니도록 제약될 수 있다. Bitcoin Core는 `IsFinalTx`를, 특정 height와 time에서 트랜잭션이 final인지 그리고 블록에 포함될 수 있는지 확인하는 함수로 문서화한다.[^ref-btc-core-tx-verify]

상대 finality는 BIP68 아래의 입력 sequence number를 사용한다. BIP68은 disable flag가 꺼져 있고 transaction version이 2 이상인 경우 sequence number에 consensus-enforced relative-locktime 의미를 부여한다.[^ref-bip-0068]

이 finality mechanism은 활성화 조건이 충족될 때 consensus relevant하다.

### Replaceability Policy

sequence number는 mempool policy와도 상호작용한다. BIP125는 opt-in full replace-by-fee signaling을 정의한다. 어떤 입력의 sequence number가 `0xffffffff - 1`보다 작으면 그 트랜잭션은 replaceability를 signal하며, 조상 트랜잭션이 unconfirmed인 동안 자손도 replaceability를 상속할 수 있다.[^ref-bip-0125]

이것은 relay 및 mempool policy이며, 어떤 트랜잭션이 유효한 블록에 들어갈 수 있는지를 결정하는 합의 규칙은 아니다. Bitcoin Core는 `src/policy/rbf.cpp`의 `IsRBFOptIn` 같은 RBF policy code를 가진다.[^ref-btc-core-rbf]

---

## 7. 수학적 또는 경제적 모델

### 수수료 공식

코인베이스가 아닌 트랜잭션에 대해:

```text
I = sum(input UTXO values)
O = sum(output values)
F = I - O
```

유효성은 다음을 요구한다.

```text
I >= O
F >= 0
```

`F`가 transaction fee다. 이것은 직렬화 필드가 아니다. 입력 값과 출력 값 차이에서 암묵적으로 도출된다.

### Fee Rate

채굴자와 mempool policy는 흔히 fee rate를 기준으로 transaction을 평가한다.

```text
fee_rate = fee / virtual_size
```

SegWit은 weight-based accounting을 도입했다. 이는 witness byte를 non-witness byte보다 할인하면서도 SegWit 규칙 아래 블록 검증 한도는 유지한다.[^ref-bip-0141]

fee rate는 경제 및 policy 개념이다. 낮은 fee를 가진 transaction도 consensus-valid할 수 있지만, relay가 잘 안 되거나 confirmation이 느릴 수 있다.

### 예시

```text
inputs:
  0.03000000 BTC
  0.02000000 BTC

outputs:
  0.04400000 BTC recipient
  0.00590000 BTC change

fee:
  0.05000000 - 0.04990000 = 0.00010000 BTC
```

사토시 기준으로:

```text
5,000,000 sats - 4,990,000 sats = 10,000 sats
```

트랜잭션은 "fee = 10,000 sats"라고 직접 쓰지 않는다. 노드가 지출된 UTXO에서 그 값을 도출한다.

### 출력 순서

Bitcoin 합의는 recipient output이 change output보다 먼저 와야 한다고 요구하지 않는다. output ordering은 wallet behavior다. change를 추론하는 분석가는 출력 위치를 많아야 약한 신호 하나 정도로만 다뤄야 한다.

---

## 8. 서명 커밋먼트와 Sighash 범위

### Legacy Sighash

서명은 signature hash type과 script rule이 결정하는 transaction digest에 커밋한다. legacy signature hashing에는 역사적인 복잡성과 성능 문제가 있다.

### SegWit Version 0

BIP143은 version 0 witness program에 대한 transaction digest algorithm을 정의한다. 이것은 중복 해싱을 줄이고, 지출되는 입력의 value에도 커밋하도록 설계되었기 때문에 offline signer가 전체 previous transaction을 신뢰된 소스로부터 받지 않아도 된다.[^ref-bip-0143]

이것이 운영상 중요한 이유는, signer가 자신이 무엇을 승인하는지 알아야 하기 때문이다. 서명이 커밋하는 value가 잘못되면, 실제 지출에 대해 서명이 유효하지 않게 된다.

### Taproot

BIP341은 Taproot를 SegWit version 1로 정의하며, Schnorr signature와 Taproot spend를 위한 공통 signature message function을 도입한다.[^ref-bip-0341]

이 문서에서 중요한 점은 구조적이다. 서로 다른 output type이 서로 다른 signature message algorithm을 쓸 수 있지만, 입력이 출력을 소비하고 출력이 미래 지출 조건을 정의한다는 상위 transaction model은 유지된다.

---

## 9. 보안 가정

### 트랜잭션 검증이 강제하는 것

트랜잭션 검증은 다음을 강제한다.

- 구조적 well-formedness
- 하나의 트랜잭션 안에서 중복 입력 금지
- 유효한 금액 범위
- 지출되는 UTXO의 가용성
- 일반 트랜잭션에서 outputs가 inputs를 초과하는 inflation 금지
- 코인베이스 maturity
- 적용 가능한 finality rule
- 각 지출에 대한 script satisfaction

### 이것이 강제하지 않는 것

트랜잭션 검증은 다음을 강제하지 않는다.

- 그 지급이 실제 invoice에 대응하는지
- 어떤 주소가 특정 엔터티에 속하는지
- 어떤 출력이 change인지
- 여러 입력이 한 사람 것인지
- 트랜잭션이 경제적으로 합리적인지
- zero-confirmation transaction이 교체되지 않을 것인지
- 로컬 mempool 수용이 전역 네트워크 수용을 뜻하는지

이들은 application, wallet, policy, 또는 분석의 문제다.

### 공격 및 실패 모드

중요한 transaction-level risk는 다음과 같다.

| Risk | Description | Boundary |
|---|---|---|
| Double spend attempt | 이미 지출된 outpoint 재사용 시도 | 한 활성 체인 안에서는 consensus가 거부 |
| Unconfirmed replacement | 더 높은 수수료의 conflict가 mempool transaction을 교체 | Policy와 miner behavior |
| Malleability | identifier 또는 witness 관련 추적 변화 | transaction type에 따라 다름 |
| Fee underpayment | transaction은 유효하지만 경제적으로 매력 없음 | Relay/mining policy |
| Change misclassification | 분석가가 출력 소유권을 잘못 라벨링 | Heuristic failure |
| Signing wrong transaction | signer가 예상과 다른 output/value에 커밋 | Wallet/signing security |

---

## 10. Bitcoin Core 구현

### 트랜잭션 Primitive

Bitcoin Core의 `src/primitives/transaction.h`는 주요 transaction structure를 정의한다.[^ref-btc-core-transaction]

| Structure | Role |
|---|---|
| `COutPoint` | previous transaction hash + output index |
| `CTxIn` | previous outpoint를 참조하고 script/sequence data를 담는 입력 |
| `CTxOut` | output value + scriptPubKey |
| `CTransaction` | inputs, outputs, version, locktime을 가진 immutable transaction object |
| `CMutableTransaction` | 최종 transaction 생성 전 사용하는 mutable construction form |

Bitcoin Core의 `CTransaction`은 네트워크에서 전파되고 블록에 포함되는 기본 transaction object다.[^ref-btc-core-transaction]

### 트랜잭션 식별자

Bitcoin Core는 `transaction_identifier`를 통해 identifier type을 분리한다. 이것은 canonical `txid`와 `wtxid` type을 표현한다.[^ref-btc-core-txid]

이 구분은 code review에서 중요하다. witness-aware identifier가 필요한 함수는 witness data가 중요할 때 legacy identifier를 조용히 받아들이면 안 된다.

### 합의 트랜잭션 검사

Bitcoin Core의 consensus transaction check는 여러 파일에 나뉘어 있다.

| Source area | Function | Role |
|---|---|---|
| `src/consensus/tx_check.cpp` | `CheckTransaction` | Context-free transaction validity |
| `src/consensus/tx_verify.cpp` | `IsFinalTx` | Absolute finality |
| `src/consensus/tx_verify.cpp` | `SequenceLocks` | BIP68 relative finality |
| `src/consensus/tx_verify.cpp` | `CheckTxInputs` | UTXO-context validity and fee computation |
| `src/script/interpreter.cpp` | `VerifyScript` | Script satisfaction |

정확한 파일 구성은 시간이 지나며 바뀔 수 있지만, 이 참조는 2026-08-04 기준 Bitcoin Core Doxygen 문서를 반영한다.

### Policy Code

RBF는 consensus code가 아니라 policy code에 구현된다. `src/policy/rbf.cpp`에는 mempool context에서 명시적 및 상속형 BIP125 replaceability를 검사하는 `IsRBFOptIn`이 있다.[^ref-btc-core-rbf]

기관 분석가는 RBF signaling을 invalidity marker로 취급하면 안 된다. 이것은 unconfirmed transaction에 대한 replaceability 및 settlement-risk 신호다.

---

## 11. Consensus, Policy, Wallet Behavior

### Consensus

consensus rule은 어떤 transaction이 유효한 블록에 포함될 수 있는지를 결정한다.

예시:

- transaction structure는 유효해야 한다.
- outputs는 valid money range 안에 있어야 한다.
- inputs는 spendable UTXO를 참조해야 한다.
- scripts는 검증을 통과해야 한다.
- finality rule을 만족해야 한다.
- coinbase value와 maturity rule을 지켜야 한다.

### Policy

policy rule은 어떤 transaction을 노드가 기본적으로 relay하거나 mine할지를 결정한다.

예시:

- dust policy
- standard script form
- minimum relay fee
- mempool replacement rule
- ancestor/descendant limit
- package acceptance rule

어떤 비표준 transaction은 consensus-valid할 수 있지만, 많은 노드가 relay하지 않을 수 있다.

### Wallet Behavior

wallet behavior는 transaction이 어떻게 구성되는지를 결정한다.

예시:

- coin selection
- change address selection
- output ordering
- fee estimation
- RBF signaling choice
- address reuse policy

wallet behavior는 온체인 패턴을 만들지만, 직접적인 consensus rule은 아니다.

---

## 12. 온체인 함의

### 강한 증거

transaction data는 다음 주장에 대해 강한 근거를 준다.

- input outpoint
- output value
- output script
- transaction identifier
- witness 존재 여부
- block inclusion
- confirmation depth
- UTXO 값이 알려졌을 때의 지불 수수료
- template가 명확할 때의 script template 분류

### 약한 또는 휴리스틱 증거

transaction data만으로는 다음 주장에 대한 근거가 약하다.

- change output 식별
- wallet ownership
- entity clustering
- 거래소 입금/출금 라벨링
- user intent
- 지급의 경제적 범주
- 어떤 consolidation이 운영적, 전술적, 우연한 것인지

이런 주장은 caveat와 가능하면 외부 corroboration이 필요하다.

### 분석 워크플로

엄격한 transaction analysis workflow는 다음과 같다.

1. transaction field를 파싱한다.
2. inputs와 previous outputs를 식별한다.
3. coinbase인지 non-coinbase인지 확인한다.
4. input value, output value, fee를 계산한다.
5. output script를 분류한다.
6. confirmation status와 reorg risk를 확인한다.
7. unconfirmed 상태라면 opt-in RBF 같은 policy signal을 식별한다.
8. factual state를 세운 뒤에만 heuristic을 적용한다.
9. confidence와 counterexample을 라벨링한다.

---

## 13. 기관 관점에서의 해석

### Custody

수탁기관은 transaction structure를 중요하게 본다. 모든 출금, 통합, sweep, cold-storage 이동은 UTXO 상태 전이이기 때문이다. 통제는 다음을 검증해야 한다.

- destination output
- change handling
- fee reasonableness
- signing scope
- expected script type
- input provenance
- unconfirmed라면 replacement policy

### Risk

리스크 팀은 final settlement와 mempool visibility를 구분해야 한다. 미확정 트랜잭션은 정산 완료와 동일하지 않다. RBF signaling, 낮은 fee rate, ancestor dependency, mempool divergence는 장기 consensus validity를 바꾸지 않으면서도 단기 settlement expectation에 영향을 줄 수 있다.

### Accounting

회계 시스템은 프로토콜 수준에서 Bitcoin을 단일 account balance로 모델링하면 안 된다. balance view는 기관이 통제하는 UTXO에서 파생되며, pending spend, confirmation, change output, reorg risk를 함께 고려해야 한다.

### Compliance Analytics

컴플라이언스 분석은 과장하지 않아야 한다. transaction graph는 output flow를 보여줄 수 있지만, outputs와 사람/기관 사이의 매핑은 독립 attribution data에 근거하지 않는 한 확률적이다.

---

## 14. 흔한 오해

### "A Transaction Sends From an Address"

합의 수준에서는 틀렸다. 트랜잭션은 이전 출력을 지출한다. 주소는 목적지 스크립트를 표현하는 사용자 친화적 인코딩 또는 descriptor일 뿐, authoritative sender field가 아니다.

### "The Fee Is Written in the Transaction"

아니다. 수수료는 입력 값과 출력 값에서 추론된다.

### "Every Input Belongs to the Same Owner"

단순 clustering에서는 자주 가정하지만, 보장되지는 않는다. CoinJoin과 협업형 트랜잭션은 이 가정을 의도적으로 깨뜨린다.

### "RBF Means Fraud"

아니다. RBF는 미확정 트랜잭션을 교체하는 policy mechanism이다. fee bumping, 수정, 또는 이중지불 시도에 사용될 수 있다. 해석은 문맥에 달려 있다.

### "SegWit Removed All Malleability"

아니다. SegWit은 witness data에 대한 identifier boundary를 바꾸고, SegWit spend의 중요한 실무적 malleability 문제를 고친다. 분석가는 여전히 transaction type과 signature scope를 이해해야 한다.

### "A Valid Transaction Must Relay Everywhere"

아니다. consensus validity와 relay policy는 다르다. 어떤 transaction은 consensus-valid하지만 많은 노드의 mempool에는 받아들여지지 않을 수 있다.

---

## 15. 연구 질문

1. transaction structure는 체인 데이터에서 무엇을 증명할 수 있는지의 경계를 어떻게 결정하는가?
2. change output을 식별할 때 분석가는 confidence를 어떻게 분류해야 하는가?
3. 어떤 운영 신호가 consolidation과 payment batching을 구분하는가?
4. `txid`와 `wtxid`는 transaction tracking system에 어떤 영향을 주는가?
5. 기관은 replaceability를 signal하는 미확정 트랜잭션을 어떻게 다뤄야 하는가?
6. 서로 다른 signature hash mode는 "the transaction was signed"의 의미를 어떻게 바꾸는가?
7. hardware signing workflow는 wrong-output 또는 wrong-fee signing을 막기 위해 어떤 control을 가져야 하는가?
8. Taproot spend는 분석가에게 script visibility를 어떻게 바꾸는가?

---

## 16. Practical Exercises

### Exercise 1: Parse a Transaction

confirmed non-coinbase transaction 하나를 선택하고 다음을 식별하라.

- version;
- input count;
- 각 previous outpoint;
- output count;
- 각 output value;
- 각 output script type;
- locktime;
- witness data 존재 여부.

### Exercise 2: Compute Fee

같은 transaction에 대해:

1. 각 입력이 지출하는 previous output을 가져온다.
2. input value를 모두 합산한다.
3. output value를 모두 합산한다.
4. fee를 계산한다.
5. block explorer 결과와 비교한다.

### Exercise 3: Identify Validation Boundaries

각 문장을 분류하라.

| Statement | Consensus | Policy | Wallet behavior | Analytics heuristic |
|---|---:|---:|---:|---:|
| Output value must not be negative | Yes | No | No | No |
| Transaction signals opt-in RBF | No | Yes | Maybe | No |
| Second output is change | No | No | Maybe | Yes |
| Input spends a previous outpoint | Yes | No | No | No |
| Fee rate is attractive to miners | No | Maybe | No | Interpretation |

### Exercise 4: Compare `txid` and `wtxid`

SegWit transaction 하나를 조사하고 다음을 기록하라.

- legacy `txid`;
- 가능한 경우 witness transaction id;
- 직렬화 형태에 witness data가 포함되는지 여부;
- 왜 outpoint가 여전히 `txid`를 참조하는지.

---

## 17. 증거 분류

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Transaction-chain model and double-spend framing | A |
| REF-BTC-DEV-TRANSACTIONS-001 | Official developer documentation | Transaction fields, inputs, outputs, outpoints, coinbase input | A |
| REF-BIP-0068 | Consensus BIP | Relative locktime semantics for sequence numbers | A |
| REF-BIP-0125 | Policy BIP | Opt-in full RBF signaling and replacement policy | B |
| REF-BIP-0141 | Consensus BIP | SegWit serialization, `txid`, `wtxid`, witness data | A |
| REF-BIP-0143 | Consensus BIP | Version 0 witness signature digest algorithm | A |
| REF-BIP-0341 | Consensus BIP | Taproot and SegWit version 1 signature rules | A |
| REF-BTC-CORE-TRANSACTION-001 | Primary implementation source | `COutPoint`, `CTxIn`, `CTxOut`, `CTransaction`, `CMutableTransaction` | A |
| REF-BTC-CORE-TXID-001 | Primary implementation source | `transaction_identifier`, `txid`, `wtxid` typing | A |
| REF-BTC-CORE-TX-CHECK-001 | Primary implementation source | Context-free transaction checks | A |
| REF-BTC-CORE-TX-VERIFY-001 | Primary implementation source | Finality, sequence locks, input validation, fee calculation | A |
| REF-BTC-CORE-INTERPRETER-001 | Primary implementation source | Script verification boundary | A |
| REF-BTC-CORE-RBF-001 | Primary implementation source | RBF policy implementation | A |

### Claim Ledger

| Claim | Classification | Evidence |
|---|---|---|
| A transaction consumes previous outputs and creates new outputs | FACT | Whitepaper, developer documentation |
| Outpoints identify previous outputs by transaction id and index | FACT | Developer documentation, Bitcoin Core transaction primitives |
| `txid` excludes witness data under SegWit | FACT | BIP141 |
| `wtxid` includes witness data under SegWit | FACT | BIP141 |
| Fees are implied by input minus output value | FACT | Developer documentation, `CheckTxInputs` |
| `CheckTransaction` is context-free | FACT | Bitcoin Core `tx_check.cpp` |
| `CheckTxInputs` requires UTXO context | FACT | Bitcoin Core `tx_verify.cpp` |
| BIP68 gives sequence numbers relative-locktime meaning under defined conditions | FACT | BIP68 |
| BIP125 replaceability is mempool policy | FACT | BIP125, Bitcoin Core RBF policy source |
| Address labels and ownership clusters are not consensus facts | INTERPRETATION | Derived from transaction structure and UTXO model |
| Output ordering can identify change reliably by itself | COUNTERCLAIM | Rejected; output ordering is wallet behavior and weak heuristic |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| HEURISTIC | Practical rule with known counterexamples |
| POLICY | Relay, mempool, or mining convention rather than consensus |
| UNKNOWN | Evidence is insufficient |

---

## 18. 지식 그래프

```text
BITCOIN-015 Transactions in Depth
|
+-- builds_on: BITCOIN-014 UTXO Model
+-- builds_on: BITCOIN-010 Combining and Splitting Value
|
+-- transaction
|   +-- fields: version, inputs, outputs, locktime
|   +-- segwit_fields: marker, flag, witness
|
+-- input
|   +-- references: outpoint
|   +-- carries: scriptSig, sequence, witness
|
+-- output
|   +-- contains: value
|   +-- locks_with: scriptPubKey
|
+-- identifiers
|   +-- txid: non-witness serialization hash
|   +-- wtxid: witness serialization hash
|
+-- validation
|   +-- context_free: CheckTransaction
|   +-- utxo_context: CheckTxInputs
|   +-- script_context: VerifyScript
|   +-- finality: IsFinalTx, SequenceLocks
|
+-- policy
|   +-- RBF: BIP125
|   +-- relay: standardness and fee policy
|
+-- analysis
    +-- facts: outpoints, outputs, fee, ids
    +-- heuristics: ownership, change, clustering
```

---

## 19. 참고문헌

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Sections 2 and 9, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-transactions]: Bitcoin Developer Documentation, "Transactions," transaction format, inputs, outputs, outpoints, locktime, and coinbase input, https://developer.bitcoin.org/reference/transactions.html, accessed 2026-08-04.

[^ref-bip-0068]: Mark Friedenbach, BtcDrak, Nicolas Dorier, and kinoshitajona, "BIP 68: Relative lock-time using consensus-enforced sequence numbers," 2015-05-28, https://bips.dev/68/, accessed 2026-08-04.

[^ref-bip-0125]: David A. Harding and Peter Todd, "BIP 125: Opt-in Full Replace-by-Fee Signaling," 2015-12-04, https://bips.dev/125/, accessed 2026-08-04.

[^ref-bip-0141]: Eric Lombrozo, Johnson Lau, and Pieter Wuille, "BIP 141: Segregated Witness (Consensus layer)," 2015-12-21, https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki, accessed 2026-08-04.

[^ref-bip-0143]: Johnson Lau and Pieter Wuille, "BIP 143: Transaction Signature Verification for Version 0 Witness Program," 2016-01-03, https://bips.dev/143/, accessed 2026-08-04.

[^ref-bip-0341]: Pieter Wuille, Jonas Nick, and Anthony Towns, "BIP 341: Taproot: SegWit version 1 spending rules," 2020-01-19, https://bips.xyz/341, accessed 2026-08-04.

[^ref-btc-core-transaction]: Bitcoin Core Contributors, `src/primitives/transaction.h`, `COutPoint`, `CTxIn`, `CTxOut`, `CTransaction`, and `CMutableTransaction`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/primitives_2transaction_8h.html, accessed 2026-08-04.

[^ref-btc-core-txid]: Bitcoin Core Contributors, `src/primitives/transaction_identifier.h`, `transaction_identifier` template for `txid` and `wtxid`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/classtransaction__identifier.html, accessed 2026-08-04.

[^ref-btc-core-tx-check]: Bitcoin Core Contributors, `src/consensus/tx_check.cpp`, `CheckTransaction`, Bitcoin Core Doxygen 31.99.0 source documentation, https://doxygen.bitcoincore.org/tx__check_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-tx-verify]: Bitcoin Core Contributors, `src/consensus/tx_verify.h` and `src/consensus/tx_verify.cpp`, `IsFinalTx`, `SequenceLocks`, `GetTransactionSigOpCost`, and `CheckTxInputs`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/tx__verify_8h.html, accessed 2026-08-04.

[^ref-btc-core-interpreter]: Bitcoin Core Contributors, `src/script/interpreter.cpp`, script evaluation and `VerifyScript`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/interpreter_8cpp.html, accessed 2026-08-04.

[^ref-btc-core-rbf]: Bitcoin Core Contributors, `src/policy/rbf.cpp`, `IsRBFOptIn` and RBF mempool policy logic, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/policy_2rbf_8cpp_source.html, accessed 2026-08-04.

---

## 20. 교차 참조

### Parent

- Bitcoin Internals

### Previous

- BITCOIN-014 — UTXO Model

### Next

- BITCOIN-016 — Script & ScriptPubKey

### Related

- BITCOIN-003 — Whitepaper Section 2 — Transactions
- BITCOIN-008 — Whitepaper Section 7 — Reclaiming Disk Space
- BITCOIN-009 — Whitepaper Section 8 — Simplified Payment Verification
- BITCOIN-010 — Whitepaper Section 9 — Combining and Splitting Value
- BITCOIN-014 — UTXO Model
- BITCOIN-016 — Script & ScriptPubKey
- BITCOIN-017 — Mempool
- BITCOIN-023 — Chain Reorganization
- POW-009 — Coinbase Transaction and Mining Commitments

---

## Review Status

### Technical Review

Passed.

- transaction field와 wallet concept를 분리했다.
- `txid`와 `wtxid` 정의를 BIP141와 Bitcoin Core identifier 문서에 맞춰 점검했다.
- context-free, UTXO-context, script, finality, policy check를 분리했다.
- BIP68 sequence-lock semantics와 BIP125 RBF policy signaling을 구분했다.
- 수수료 계산을 직렬화 필드가 아니라 암묵적 value difference로 표현했다.

### Evidence Review

Passed.

- transaction structure 관련 주장은 공식 Bitcoin Developer 문서를 인용한다.
- SegWit serialization 관련 주장은 BIP141를 인용한다.
- signature digest 관련 주장은 BIP143와 BIP341을 인용한다.
- 구현 관련 주장은 현재 Bitcoin Core Doxygen 또는 소스 문서를 인용한다.
- RBF 관련 주장은 policy로 라벨링하고 BIP125 및 Bitcoin Core policy source를 인용한다.

### Editorial Review

Passed.

- Markdown 헤딩은 프로젝트 딥다이브 구조를 따른다.
- 메타데이터는 완전하다.
- 표와 코드 펜스는 닫혀 있다.
- transaction, input, output, outpoint, scriptSig, scriptPubKey, witness, `txid`, `wtxid`, fee 용어를 일관되게 사용했다.

### Adversarial Review

Passed.

- 주소를 consensus sender field로 설명하지 않았다.
- RBF를 fraud나 consensus invalidity로 취급하지 않았다.
- change detection과 ownership clustering을 사실로 취급하지 않았다.
- SegWit이 모든 transaction ambiguity를 제거한다고 주장하지 않았다.
- consensus validity와 mempool relay policy를 구분했다.

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
