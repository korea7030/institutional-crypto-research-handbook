---
knowledge_id: BITCOIN-016
title: Script & ScriptPubKey
subtitle: 잠금 조건, 잠금 해제 데이터, witness program, 표준 템플릿, 그리고 Script 검증 경계
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 110 min
estimated_study: 320 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Script
  - Transactions
  - Validation
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-014
  - BITCOIN-015
related_topics:
  - ScriptPubKey
  - ScriptSig
  - Witness
  - P2PKH
  - P2SH
  - P2WPKH
  - P2WSH
  - P2TR
  - Tapscript
  - Standardness
primary_sources:
  - REF-BTC-DEV-TRANSACTIONS-001
  - REF-BIP-0016
  - REF-BIP-0141
  - REF-BIP-0143
  - REF-BIP-0340
  - REF-BIP-0341
  - REF-BIP-0342
  - REF-BTC-CORE-SCRIPT-001
  - REF-BTC-CORE-INTERPRETER-001
  - REF-BTC-CORE-SOLVER-001
  - REF-BTC-CORE-POLICY-001
  - REF-BTC-CORE-ADDRESS-001
tags:
  - bitcoin
  - internals
  - script
  - scriptpubkey
  - scriptsig
  - witness
  - p2sh
  - segwit
  - taproot
---

# Script & ScriptPubKey
> Bitcoin Internals  
> Research Unit: BITCOIN-016

---

## Research Brief

```yaml
knowledge_id: BITCOIN-016
title: Script & ScriptPubKey
research_question: >
  How does Bitcoin Script express spending conditions, how do scriptPubKey,
  scriptSig, witness data, redeemScript, witnessScript, and Taproot relate,
  and how should analysts distinguish consensus script validity, policy
  standardness, wallet address encoding, and entity-level interpretation?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-014
  - BITCOIN-015
parent: Bitcoin Internals
previous: BITCOIN-015
next: BITCOIN-017
related_topics:
  - Transaction Validation
  - UTXO Model
  - Segregated Witness
  - Taproot
  - Standardness
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
  - Exhaustive opcode reference
  - Formal proof of ECDSA or Schnorr security
  - Miniscript policy language deep dive
  - Wallet descriptor engineering beyond brief orientation
  - Full mempool package policy
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- `scriptPubKey`를 output locking script로 정의할 수 있다.
- `scriptSig`와 witness data를 spending data로 정의할 수 있다.
- 주소와 script가 같은 개념이 아닌 이유를 설명할 수 있다.
- P2PKH, P2SH, P2WPKH, P2WSH, P2TR의 차이를 설명할 수 있다.
- redeemScript와 witnessScript가 지출 시점에 무엇을 드러내는지 설명할 수 있다.
- consensus script validity와 policy standardness를 구분할 수 있다.
- Bitcoin Core에서 script 표현, script 검증, 표준 출력 템플릿 분류가 어디서 이루어지는지 식별할 수 있다.
- script type이 privacy와 온체인 해석에 어떤 영향을 주는지 설명할 수 있다.
- script template만으로 실세계 소유 구조나 custody 설계를 단정하지 않을 수 있다.

---

## 2. 핵심 질문

1. Bitcoin Script는 어떤 방식으로 지출 조건을 표현하는가?
2. `scriptPubKey`, `scriptSig`, witness는 각각 어떤 역할을 하는가?
3. 주소는 왜 합의 객체가 아니라 표시 형식인가?
4. P2SH와 SegWit는 검증과 데이터 공개 시점을 어떻게 바꿨는가?
5. P2TR과 tapscript는 이전 템플릿과 무엇이 다른가?
6. consensus-valid script와 standard relay script는 왜 다를 수 있는가?
7. Bitcoin Core는 script를 어떤 자료구조와 함수로 처리하는가?
8. 분석가는 script type에서 무엇을 말할 수 있고 무엇은 말할 수 없는가?

---

## 3. Executive Summary

Bitcoin Script는 UTXO를 어떤 조건에서 지출할 수 있는지를 표현하는 간단한 stack-based 언어다. 출력에는 locking condition이 `scriptPubKey` 형태로 저장되고, 입력에는 이를 만족시키기 위한 spending data가 `scriptSig` 또는 witness에 저장된다.[^ref-btc-dev-transactions]

주소는 script 그 자체가 아니다. 주소는 특정 표준 script template을 사람이 다루기 쉬운 문자열로 인코딩한 표현 계층이다. 따라서 합의 검증은 주소가 아니라 실제 locking script와 spending data를 대상으로 수행된다.[^ref-btc-core-address]

표준적인 스크립트 계열은 대체로 다음과 같이 정리할 수 있다.

- P2PK: 공개키에 직접 잠금
- P2PKH: 공개키 해시에 잠금
- P2SH: redeemScript 해시에 잠금
- P2WPKH / P2WSH: witness program 기반 SegWit 잠금
- P2TR: x-only key와 선택적 script path를 가진 Taproot 잠금

Bitcoin Core는 `CScript`로 script byte sequence를 표현하고, interpreter 계층에서 script execution과 서명 검증을 수행하며, solver와 policy 계층에서 표준 템플릿을 분류한다.[^ref-btc-core-script] [^ref-btc-core-interpreter] [^ref-btc-core-solver]

분석 측면에서 script type은 strong signal이지만 definitive identity proof는 아니다. 특정 출력이 P2WSH인지 P2TR인지, multisig 구조가 지출 시 공개되었는지, witness discount를 받는지 등은 말할 수 있다. 반면 그 출력이 정확히 어느 기관의 어떤 내부 승인 정책을 반영하는지는 script만으로 단정할 수 없다.

---

## 4. 프로토콜 구조

### 잠금과 해제

Bitcoin의 output은 value와 `scriptPubKey`를 포함한다. `scriptPubKey`는 "이 출력을 어떤 조건에서 쓸 수 있는가"를 정의한다. 입력은 이전 output을 참조하고, 그 조건을 만족시키기 위한 데이터를 `scriptSig` 또는 witness에 제공한다.[^ref-btc-dev-transactions]

개념적으로는 다음과 같다.

```text
UTXO = value + scriptPubKey
spend = outpoint reference + unlocking data
```

### 검증의 큰 흐름

코인베이스가 아닌 일반 지출은 대체로 다음 과정을 거친다.

1. 이전 output을 outpoint로 찾는다.
2. 해당 output의 `scriptPubKey`를 읽는다.
3. 현재 입력의 `scriptSig` 또는 witness를 해제 데이터로 준비한다.
4. 합의 규칙에 따라 script engine이 이를 평가한다.
5. 조건을 만족하면 해당 입력은 유효하다.

### 주소와 script의 분리

주소는 wallet/UI 표현 계층이다. 합의 엔진은 Bech32 주소나 Base58Check 주소를 직접 검증하지 않는다. 그것은 해석된 결과로부터 실제 script template을 복원하거나 표시할 뿐이다.[^ref-btc-core-address]

### 출력 템플릿의 진화

Bitcoin의 주요 출력 템플릿은 보안, 효율성, 업그레이드 유연성의 필요에 따라 진화했다.

| Template | Locking idea | Spend-time disclosure |
|---|---|---|
| P2PK | 공개키 자체 | 서명 |
| P2PKH | 공개키 해시 | 공개키 + 서명 |
| P2SH | redeemScript 해시 | redeemScript + 필요한 해제 데이터 |
| P2WPKH | key-hash witness program | witness에 공개키 + 서명 |
| P2WSH | script-hash witness program | witnessScript + witness stack |
| P2TR | Taproot output key | key path는 Schnorr 서명, script path는 tapscript branch |

---

## 5. 주요 스크립트 템플릿

### P2PK

초기 형태의 잠금 방식이다. 공개키를 직접 포함하고 해당 키의 서명을 요구한다. 오늘날 분석 빈도는 낮지만 역사적 출력에서 여전히 중요하다.

### P2PKH

가장 널리 알려진 고전적 템플릿이다.

```text
OP_DUP OP_HASH160 <pubKeyHash> OP_EQUALVERIFY OP_CHECKSIG
```

지출 시 공개키와 서명이 드러난다. 주소 형태로는 보통 Base58Check P2PKH 주소로 표현된다.

### P2SH

BIP16은 script 자체 대신 redeemScript의 해시에 잠그는 방식을 도입했다.[^ref-bip-0016]

```text
OP_HASH160 <scriptHash> OP_EQUAL
```

이 구조의 핵심은 복잡한 spending policy를 output 생성 시점이 아니라 spend 시점에 공개할 수 있다는 점이다. 분석가에게는 "잠금 시점에는 해시만 보이고, 지출 시점에 실제 redeemScript가 공개된다"는 점이 중요하다.

### P2WPKH / P2WSH

BIP141는 witness program이라는 새 구조를 도입했다.[^ref-bip-0141]

```text
OP_0 <20-byte keyhash>   # P2WPKH
OP_0 <32-byte scripthash> # P2WSH
```

잠금 script는 짧아지고, spending data는 witness로 분리된다. legacy `txid`에는 witness가 포함되지 않지만 `wtxid`에는 포함된다. 이로써 malleability와 블록 공간 효율성 측면에서 중요한 변화가 생긴다.

### Nested SegWit

P2SH 안에 SegWit program을 넣는 방식이다. 이전 인프라 호환성을 위한 과도기적 구조로 이해하면 된다. 주소, 표시 형식, 내부 잠금 구조를 분리해서 봐야 한다.

### P2TR

BIP341은 Taproot output을 정의한다.[^ref-bip-0341] P2TR은 x-only public key 기반 output key를 사용하며, key path spend와 script path spend를 모두 지원한다. key path spend는 비교적 간결하고, script path spend는 필요한 branch만 공개한다.

### Tapscript

BIP342는 script path에서 사용되는 tapscript 규칙을 정의한다.[^ref-bip-0342] 이는 이전 script 버전과 동일하지 않으며, Taproot용 서명 검증과 opcode 의미론에 맞는 별도 규칙 집합이다.

---

## 6. Technical Mechanics

### `scriptSig`와 witness의 역할 분리

legacy spend에서는 해제 데이터가 `scriptSig`에 들어간다. SegWit spend에서는 핵심 해제 데이터가 witness로 이동한다. 따라서 입력을 볼 때는 다음을 분리해야 한다.

- `scriptSig`에 무엇이 들어 있는가
- witness에 무엇이 들어 있는가
- 실제 잠금 조건은 이전 output의 `scriptPubKey`가 무엇이었는가

### redeemScript와 witnessScript

- `redeemScript`: P2SH spend 시 공개되는 실제 script
- `witnessScript`: P2WSH spend 시 witness에 공개되는 실제 script

둘 다 "해시 뒤에 숨겨져 있다가 spend 시점에 드러나는 script"라는 공통점이 있지만, 직렬화 위치와 서명 해시 규칙, 비용 구조가 다르다.[^ref-bip-0143]

### script 평가 경계

script는 입력이 이전 output을 적법하게 지출하는지 확인하는 데 사용된다. 그러나 모든 transaction 속성이 script로 검증되는 것은 아니다. 예를 들어 입력/출력 값 보존, 코인베이스 maturity, 중복 입력 여부 같은 사항은 script interpreter 바깥의 transaction validation 계층에서 다뤄진다.

### template 인식

Bitcoin Core는 표준적인 출력 형태를 식별해 주소 표시, wallet 처리, 정책 판단에 활용한다. 이때 중요한 점은 template 인식이 합의 그 자체는 아니라는 점이다. 합의는 byte-level script validity에 의해 결정되고, 표준 템플릿 분류는 상위 계층의 편의 또는 정책이다.[^ref-btc-core-solver]

---

## 7. Mathematical or Economic Model

### script 해시 잠금의 기본 아이디어

P2SH와 P2WSH는 다음 구조를 사용한다.

```text
commitment = HASH(script)
```

출력 생성 시점에는 해시만 공개하고, 지출 시점에만 실제 script를 공개한다. 이 구조는 잠금 시점 데이터 노출을 줄이고, 복잡한 지출 정책을 뒤로 미룬다.

### witness discount와 비용 구조

SegWit 이후 spending data 일부가 witness 영역으로 이동하면서 weight 계산에서 할인된다. 이 때문에 동일한 논리 구조라도 P2WSH, P2TR key path, script path 등은 서로 다른 비용 구조를 가진다. 분석가는 "보안 구조"와 "공간 비용"을 함께 봐야 한다.

### 정보 노출의 경제적 의미

복잡한 script는 보안 정책을 강화할 수 있지만, spend 시점 공개 데이터가 커지고 분석 표면도 넓어진다. 반대로 간결한 key path spend는 비용과 노출을 줄일 수 있다. 이는 privacy, 비용, 운영 유연성 사이의 trade-off다.

---

## 8. Security Assumptions

### 서명 체계 가정

P2PK, P2PKH, P2WPKH, P2TR key path 등은 적절한 공개키 암호 가정을 전제로 한다. 본 문서는 ECDSA/Schnorr의 형식적 보안 증명을 다루지 않지만, 합의 검증은 해당 서명 검증 규칙의 정확한 구현에 의존한다.[^ref-bip-0340]

### 해시 함수 가정

P2SH와 P2WSH는 script hash commitment가 충돌 저항성을 가진다는 가정에 의존한다. 해시 commitment가 약하면 의도하지 않은 script 대체 위험이 생긴다.

### 해석 과잉 위험

script template을 보고 곧바로 실세계 주체의 승인 정책을 추정하는 것은 보안적으로도 분석적으로도 위험하다. 동일한 template이 전혀 다른 운영 모델에서 사용될 수 있기 때문이다.

---

## 9. Bitcoin Core 구현

### 핵심 소스 영역

| Area | Role |
|---|---|
| `script/script.h` | `CScript`와 opcode 표현 |
| `script/interpreter.*` | script execution, signature checking, verify flags |
| `script/solver.*` | 표준 템플릿 분류와 destination 추론 |
| `policy/policy.*` | standardness 판단 |
| `addresstype.*` | address encoding/decoding 보조 |

### `CScript`

Bitcoin Core는 script를 본질적으로 opcode와 pushdata의 byte sequence로 다룬다. 이는 "주소 객체"가 아니라 직렬화된 검증 프로그램이라는 점을 분명히 보여 준다.[^ref-btc-core-script]

### interpreter

interpreter 계층은 stack machine 평가, 서명 확인, witness program 처리, tapscript 규칙 적용을 담당한다.[^ref-btc-core-interpreter]

### solver와 policy

solver는 script가 알려진 표준 템플릿에 해당하는지 분류할 수 있다. policy 계층은 이를 이용해 기본 노드가 relay/mining 대상으로 취급할지를 결정한다.[^ref-btc-core-policy]

---

## 10. Consensus, Policy, and Presentation

### Consensus

consensus는 "이 spend가 유효한 블록에 포함될 수 있는가?"를 묻는다. script가 합의 규칙에 맞게 평가되고 다른 transaction 규칙도 만족하면 합의상 유효하다.

### Policy

policy는 "기본 설정 노드가 이 transaction을 relay하거나 채굴 대상으로 취급할 것인가?"를 묻는다. nonstandard script는 consensus-valid일 수 있어도 기본 정책에서는 거부될 수 있다.[^ref-btc-core-policy]

### Presentation

presentation은 "소프트웨어가 이 script를 인간에게 어떻게 보여 줄 것인가?"를 다룬다. 주소 문자열, script type 라벨, wallet UI 설명은 모두 이 계층의 문제다. 이는 합의 규칙과 동일하지 않다.

---

## 11. 온체인 함의

### Strong Evidence

script 데이터는 다음 주장에 강한 근거를 제공한다.

- 특정 output이 어떤 locking template를 사용했는가
- 지출 시 redeemScript 또는 witnessScript가 공개되었는가
- SegWit/Taproot 경로가 사용되었는가
- witness가 포함된 spend인지 아닌지
- 특정 spend가 key path인지 script path인지

### Weak Evidence

script 데이터는 다음 주장에는 약한 근거만 제공한다.

- 정확한 소유 주체
- 내부 승인 절차
- 거래 상대방 관계
- self-custody 여부와 custodial 여부
- privacy intent

### 공개 시점의 차이

P2SH, P2WSH, P2TR script path는 "언제 무엇이 드러나는가"가 중요하다. 잠금 시점에 보이지 않던 정책이 spend 시점에만 공개될 수 있기 때문이다. 이는 시계열 분석에서 중요한 포인트다.

---

## 12. Institutional Thinking

### custody 해석

기관 분석가는 multisig 흔적이나 complex script 공개만으로 특정 custody model을 단정하면 안 된다. 동일한 기업이라도 hot wallet, cold wallet, recovery path, vendor-managed path가 서로 다른 template를 쓸 수 있다.

### privacy 평가

Taproot key path 사용은 script path 노출을 줄여 분석 표면을 축소할 수 있다. 그러나 이것만으로 privacy가 보장되지는 않는다. 주소 재사용, 입출금 패턴, timing correlation 같은 다른 요소가 여전히 중요하다.

### 운영 통제

기관 시스템은 최소한 다음을 기록하는 것이 좋다.

- output script type
- spend path type
- redeemScript/witnessScript 공개 여부
- witness 포함 여부
- txid / wtxid 구분
- spend 전후 시점 정보

---

## 13. Common Misinterpretations

### "주소가 곧 script다"

아니다. 주소는 특정 script template의 인코딩 표현이다.

### "P2SH면 항상 multisig다"

아니다. P2SH는 redeemScript 해시에 잠그는 일반적 메커니즘일 뿐이다.

### "script가 복잡하면 더 안전하다"

항상 그렇지 않다. 복잡성은 구현 위험, 비용, 공개 정보 증가를 동반할 수 있다.

### "Taproot는 완전한 익명성을 준다"

아니다. Taproot는 일부 경로의 정보 노출을 줄일 수 있지만, ownership anonymity를 보장하지 않는다.

### "consensus-valid면 기본 노드에서 항상 relay된다"

아니다. policy standardness와 relay 규칙이 따로 존재한다.

---

## 14. Research Questions

1. 현재 주요 거래소 입출금 흐름에서 P2TR adoption은 어느 정도인가?
2. P2WSH와 P2TR script path 공개 패턴은 custody 구조 추정에 어떤 한계를 가지는가?
3. nonstandard but consensus-valid script는 실제로 얼마나 자주 관찰되는가?
4. wallet 소프트웨어별 address encoding 선택은 분석 편향을 어떻게 만든는가?
5. key path spend 중심 전환이 multisig 가시성에 어떤 영향을 주는가?

---

## 15. Practical Exercises

### Exercise 1: 출력 템플릿 식별

서로 다른 세 transaction을 선택해 각 output의 `scriptPubKey`를 분류하라.

- P2PKH
- P2SH
- P2WPKH
- P2WSH
- P2TR

### Exercise 2: spend 시점 공개 정보 비교

P2SH spend와 P2WSH spend를 하나씩 선택하고 다음을 비교하라.

- 잠금 시점에 보이는 정보
- spend 시점에 새로 공개되는 정보
- 분석가가 그제서야 알 수 있게 되는 정보

### Exercise 3: txid와 witness 관계

SegWit transaction 하나를 선택해 `txid`와 `wtxid`를 비교하고, witness가 어느 식별자에 반영되는지 설명하라.

### Exercise 4: consensus와 policy 분리

다음 진술을 분류하라.

| Statement | Consensus | Policy | Presentation |
|---|---:|---:|---:|
| 출력이 P2TR template로 보인다 | No | No | Yes |
| 입력이 이전 output 조건을 만족한다 | Yes | No | No |
| 기본 노드가 nonstandard script를 relay하지 않는다 | No | Yes | No |
| 주소 문자열이 bc1p로 시작한다 | No | No | Yes |

---

## 16. Evidence Classification

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-DEV-TRANSACTIONS-001 | Developer documentation | 트랜잭션과 script 구성 개요 | A |
| REF-BIP-0016 | Consensus/policy BIP | P2SH 정의 | A |
| REF-BIP-0141 | Consensus BIP | SegWit와 witness program 정의 | A |
| REF-BIP-0143 | Signature-hash BIP | SegWit signature digest rules | A |
| REF-BIP-0340 | Cryptographic BIP | Schnorr signature rules | A |
| REF-BIP-0341 | Consensus BIP | Taproot output과 spend path | A |
| REF-BIP-0342 | Consensus BIP | Tapscript rules | A |
| REF-BTC-CORE-SCRIPT-001 | Primary implementation source | `CScript`와 script 표현 | A |
| REF-BTC-CORE-INTERPRETER-001 | Primary implementation source | script execution과 verify flags | A |
| REF-BTC-CORE-SOLVER-001 | Primary implementation source | 표준 템플릿 분류 | A |
| REF-BTC-CORE-POLICY-001 | Primary implementation source | standardness와 relay policy | A |
| REF-BTC-CORE-ADDRESS-001 | Primary implementation source | address encoding/decoding | A |

### Claim Ledger

| Claim | Classification | Evidence |
|---|---|---|
| `scriptPubKey`는 output locking condition을 표현한다 | FACT | Developer docs and Core script source |
| `scriptSig`와 witness는 spending data 계층이다 | FACT | Developer docs, BIP141, interpreter source |
| 주소는 script 자체가 아니라 표시 인코딩이다 | FACT | Address handling source and transaction structure |
| P2SH는 redeemScript 해시에 잠근다 | FACT | BIP16 |
| P2WSH는 witnessScript 해시에 잠근다 | FACT | BIP141 |
| P2TR는 key path와 script path를 지원한다 | FACT | BIP341 |
| nonstandard script는 consensus-valid일 수 있다 | FACT | Core policy/source distinction |
| script template만으로 실세계 소유 주체를 증명할 수 있다 | COUNTERCLAIM | Rejected; over-inference |
| Taproot 사용이 익명성을 보장한다 | COUNTERCLAIM | Rejected; privacy benefit is conditional |
| spend-time script disclosure는 분석 표면을 바꾼다 | INTERPRETATION | Derived from P2SH/P2WSH/P2TR design |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | 직접적인 1차 자료로 뒷받침됨 |
| INTERPRETATION | 사실을 기반으로 한 분석적 종합 |
| POLICY | 합의가 아니라 relay/mining/wallet 관행 |
| HEURISTIC | 실무적 규칙이지만 반례가 있음 |
| UNKNOWN | 근거가 부족함 |

---

## 17. Knowledge Graph

```text
BITCOIN-016 Script & ScriptPubKey
|
+-- builds_on: BITCOIN-014 UTXO Model
+-- builds_on: BITCOIN-015 Transactions in Depth
|
+-- output
|   +-- contains: value
|   +-- contains: scriptPubKey
|
+-- input
|   +-- references: outpoint
|   +-- provides: scriptSig or witness
|
+-- templates
|   +-- P2PKH
|   +-- P2SH
|   +-- P2WPKH
|   +-- P2WSH
|   +-- P2TR
|
+-- hidden_until_spend
|   +-- redeemScript
|   +-- witnessScript
|   +-- taproot script path
|
+-- Bitcoin Core
|   +-- CScript
|   +-- interpreter
|   +-- solver
|   +-- policy
|
+-- analysis
    +-- strong: template, spend path, witness use
    +-- weak: entity identity, custody model
```

---

## 18. 참고문헌

[^ref-btc-dev-transactions]: Bitcoin Developer Documentation, "Transactions," transaction structure, inputs, outputs, and script overview, https://developer.bitcoin.org/devguide/transactions.html, accessed 2026-08-04.
[^ref-bip-0016]: Gavin Andresen, "BIP 16: Pay to Script Hash," 2012-01-03, https://bips.dev/16/, accessed 2026-08-04.
[^ref-bip-0141]: Eric Lombrozo, Johnson Lau, and Pieter Wuille, "BIP 141: Segregated Witness (Consensus layer)," 2015-12-21, https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki, accessed 2026-08-04.
[^ref-bip-0143]: Johnson Lau and Pieter Wuille, "BIP 143: Transaction Signature Verification for Version 0 Witness Program," 2016-01-03, https://bips.dev/143/, accessed 2026-08-04.
[^ref-bip-0340]: Pieter Wuille et al., "BIP 340: Schnorr Signatures for secp256k1," 2020-01-19, https://bips.dev/340/, accessed 2026-08-04.
[^ref-bip-0341]: Pieter Wuille et al., "BIP 341: Taproot: SegWit version 1 spending rules," 2020-01-19, https://bips.dev/341/, accessed 2026-08-04.
[^ref-bip-0342]: Pieter Wuille et al., "BIP 342: Validation of Taproot Scripts," 2020-01-19, https://bips.dev/342/, accessed 2026-08-04.
[^ref-btc-core-script]: Bitcoin Core Contributors, `src/script/script.h`, `CScript` definitions, Bitcoin Core Doxygen documentation, https://doxygen.bitcoincore.org/script_2script_8h_source.html, accessed 2026-08-04.
[^ref-btc-core-interpreter]: Bitcoin Core Contributors, `src/script/interpreter.cpp` and related interpreter documentation, https://doxygen.bitcoincore.org/interpreter_8cpp.html, accessed 2026-08-04.
[^ref-btc-core-solver]: Bitcoin Core Contributors, `src/script/solver.cpp`, standard template recognition, Bitcoin Core Doxygen documentation, https://doxygen.bitcoincore.org/solver_8cpp_source.html, accessed 2026-08-04.
[^ref-btc-core-policy]: Bitcoin Core Contributors, `src/policy/policy.cpp`, standardness and relay policy checks, Bitcoin Core Doxygen documentation, https://doxygen.bitcoincore.org/policy_8cpp_source.html, accessed 2026-08-04.
[^ref-btc-core-address]: Bitcoin Core Contributors, `src/key_io.cpp` and related address type handling, Bitcoin Core Doxygen documentation, https://doxygen.bitcoincore.org/key__io_8cpp_source.html, accessed 2026-08-04.

---

## 19. 교차 참조

### Parent

- Bitcoin Internals

### Previous

- BITCOIN-015 — Transactions in Depth

### Next

- BITCOIN-017 — Mempool

### Related

- BITCOIN-014 — UTXO Model
- BITCOIN-015 — Transactions in Depth
- BITCOIN-017 — Mempool
- BITCOIN-018 — Transaction Fees
- BITCOIN-030 — SegWit
- BITCOIN-031 — Taproot

---

## Review Status

### Technical Review

Passed.

- script layer와 transaction validation layer를 분리했다.
- P2SH, SegWit, Taproot의 공개 시점 차이를 정리했다.
- consensus, policy, presentation을 별도 계층으로 구분했다.

### Evidence Review

Passed.

- 주요 템플릿과 witness program 설명은 BIP16, BIP141, BIP341, BIP342에 연결했다.
- Core 구현 설명은 `CScript`, interpreter, solver, policy 계층에 연결했다.

### Editorial Review

Passed.

- 구조는 기존 deep-dive 형식을 유지한다.
- 용어는 `scriptPubKey`, `scriptSig`, witness, redeemScript, witnessScript로 일관화했다.

### Adversarial Review

Passed.

- 주소와 script를 혼동하지 않았다.
- template만으로 ownership를 단정하지 않았다.
- standardness를 consensus와 혼동하지 않았다.

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
