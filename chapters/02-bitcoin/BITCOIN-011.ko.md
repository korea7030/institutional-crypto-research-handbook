---
knowledge_id: BITCOIN-011
title: 백서 섹션 10 — 프라이버시(Privacy)
subtitle: 공개 트랜잭션 그래프, 익명적 공개키, 주소 재사용, 거스름돈 휴리스틱, 그리고 온체인 링크 분석
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 90 min
estimated_study: 240 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Privacy
  - Transactions
  - On-Chain Analysis
parent:
  - Bitcoin Whitepaper
prerequisites:
  - BITCOIN-003
  - BITCOIN-009
  - BITCOIN-010
related_topics:
  - Public Ledger
  - Address Reuse
  - Change Detection
  - Multi-Input Heuristic
  - Wallet Privacy
  - Transaction Graph Analysis
  - Pseudonymity
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-TRANSACTIONS-001
  - REF-BTC-DEV-OPERATING-001
  - REF-BIP-0032
  - REF-BTC-CORE-RPC-CREATEWALLET-001
tags:
  - bitcoin
  - whitepaper
  - privacy
  - pseudonymity
  - address-reuse
  - transaction-graph
  - change-heuristics
---

# 백서 섹션 10 — 프라이버시(Privacy)
> Deep Dive Series  
> Research Unit: BITCOIN-011

---

## Research Brief

```yaml
knowledge_id: BITCOIN-011
title: Whitepaper Section 10 — Privacy
research_question: >
  What privacy model does the Bitcoin Whitepaper propose, how does it differ
  from traditional banking privacy, and how do transaction graph structure,
  address reuse, change outputs, and multi-input transactions affect modern
  on-chain analysis?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-003
  - BITCOIN-009
  - BITCOIN-010
parent: Bitcoin Whitepaper
previous: BITCOIN-010
next: BITCOIN-012
related_topics:
  - Public Transactions
  - Pseudonymity
  - New Key Pairs
  - Multi-Input Linkage
  - Change Heuristics
  - Wallet Privacy
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
  - Complete privacy-tool taxonomy
  - CoinJoin protocol design
  - Lightning privacy
  - Network-layer privacy
  - Regulatory compliance workflow design
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- 백서의 프라이버시 모델을 설명할 수 있다.
- Bitcoin의 가명성(pseudonymity)과 전통 금융의 기밀성을 구분할 수 있다.
- 왜 Bitcoin 트랜잭션이 공개적으로 보이는지 설명할 수 있다.
- 왜 공개키나 주소를 그 자체로 법적 신원으로 취급하면 안 되는지 설명할 수 있다.
- 왜 새 키 쌍을 쓰면 직접적인 링크를 줄일 수 있는지 설명할 수 있다.
- 왜 다중 입력 트랜잭션이 공통 통제를 드러낼 수 있는지 설명할 수 있다.
- 왜 거스름돈 출력 식별은 휴리스틱인지 설명할 수 있다.
- 프로토콜 사실과 분석 휴리스틱을 구분할 수 있다.
- 신원이 트랜잭션 이력과 연결되면 왜 프라이버시가 약해지는지 설명할 수 있다.
- 기관 온체인 분석가가 트랜잭션 그래프 데이터에서 무엇을 추론할 수 있고 없는지 식별할 수 있다.

---

## 2. 핵심 질문

1. 섹션 10이 말하는 프라이버시는 무엇인가?
2. Bitcoin은 전통적인 은행 프라이버시와 어떻게 다른가?
3. 왜 모든 트랜잭션이 공개적으로 발표되어야 하는가?
4. 공개키가 프라이버시에서 맡는 역할은 무엇인가?
5. 왜 백서는 각 트랜잭션마다 새로운 키 쌍을 쓰라고 권장하는가?
6. 왜 다중 입력 트랜잭션은 링크 가능성을 만드는가?
7. 주소 클러스터링은 합의 사실인가?
8. 거스름돈 탐지는 합의 사실인가?
9. 공개키나 주소가 실제 신원과 연결되면 무슨 일이 생기는가?
10. 분석가는 공개 트랜잭션 그래프에서 무엇을 추론할 수 있는가?
11. 오프체인 증거 없이는 무엇이 여전히 불확실한가?
12. 왜 프라이버시는 단일 지갑 설정이 아니라 시스템 특성인가?

---

## 3. Executive Summary

백서 섹션 10은 Bitcoin 프라이버시를 "공개 트랜잭션 + 연결되지 않는 공개키"의 조합으로 설명한다. 전통적인 은행 모델은 정보 접근을 제한함으로써 프라이버시를 보호한다. 반면 Bitcoin은 트랜잭션을 공개하되, 공개키는 익명으로 유지하려고 한다.[^ref-btc-wp]

이것은 기밀성(confidentiality)이 아니라 가명성(pseudonymity)이다. 트랜잭션 그래프는 보인다. 금액, 입력, 출력, 트랜잭션 순서, confirmation 맥락은 분석 가능하다. 직접적으로 인코딩되지 않는 것은 특정 키를 통제하는 사람이나 기관의 법적 신원이다.

백서는 공개키가 공통 소유자와 연결되지 않도록 각 트랜잭션에 새로운 키 쌍을 사용하는 것을 권장한다.[^ref-btc-wp] 현대 지갑은 보통 지갑 시드로부터 많은 키를 파생한다. BIP32는 hierarchical deterministic wallet를 표준화하여, 하나의 마스터 시드 구조에서 많은 child key를 파생할 수 있게 했다.[^ref-bip-0032]

백서는 또한 한계를 지적한다. 다중 입력 트랜잭션은 입력들이 같은 소유자에 속했다는 사실을 드러낼 수 있다.[^ref-btc-wp] 현대 온체인 분석은 이를 multi-input common ownership heuristic으로 일반화한다. 유용하지만 합의 규칙은 아니며, 협업형 트랜잭션에서는 실패할 수 있다.

핵심 프라이버시 모델은 다음과 같다.

```text
public transaction graph
    +
pseudonymous keys
    +
fresh-key discipline
    -
linkage from reuse, multi-input spends, change, timing, and off-chain identity
```

기관 리서치에서 섹션 10은 증거 규율의 출발점이다. 분석가는 관측 가능한 그래프 구조에 대해서는 강한 주장을 할 수 있다. 그러나 identity, ownership, control, intent는 추가 증거가 필요하며 더 낮은 신뢰 수준을 가져야 한다.

---

## 4. 원래 설계

백서는 Bitcoin 프라이버시를 전통 은행 프라이버시와 비교한다. 은행에서는 정보가 관련 당사자와 신뢰된 제3자에게만 제한된다. Bitcoin에서는 노드가 순서를 검증하고 이중지불을 막기 위해 모든 트랜잭션이 공개적으로 발표되어야 한다.[^ref-btc-wp]

따라서 Bitcoin의 프라이버시 전략은 트랜잭션 자체의 비밀성이 아니다. 핵심은 트랜잭션 식별자와 실제 세계 신원 사이의 분리다.

백서는 세 가지 핵심 아이디어를 제시한다.

1. 공개키는 익명적일 수 있다.
2. 각 트랜잭션마다 새 키 쌍을 사용하는 것이 좋다.
3. 다중 입력 트랜잭션은 입력이 같은 소유자에게 속했음을 드러낼 수 있다.[^ref-btc-wp]

이 섹션은 짧지만, 지금도 Bitcoin 프라이버시를 지배하는 긴장을 정의한다.

```text
public verifiability
versus
transaction graph privacy
```

Bitcoin은 독립 검증을 위해 공개 데이터가 필요하다. 하지만 바로 그 공개 데이터가 분석적 링크 가능성을 만든다.

---

## 5. 프로토콜 구조

### 공개 원장

Bitcoin 트랜잭션이 공개적으로 보이는 이유는, 네트워크가 코인이 이중지불되지 않았는지, 수용된 체인이 유효한 트랜잭션을 담고 있는지 검증해야 하기 때문이다. 섹션 8은 간이 검증조차 공개 헤더와 트랜잭션 포함 증명에 의존함을 보여주었다. 섹션 10은 그 프라이버시 결과를 설명한다. 공개 검증은 곧 트랜잭션 흐름을 노출한다.

Bitcoin Developer 문서는 full-node validation과 simplified payment verification을 구분한다. 이는 프라이버시 분석도 사용자가 실제로 어떤 데이터를 검증하고, 어떤 데이터는 단지 피어로부터 수신만 하는지 고려해야 함을 뜻한다.[^ref-btc-dev-operating]

### 가명적 키

Bitcoin 공개키와 주소는 실제 이름이 아니다. 그것들은 출력을 받고 지출하기 위해 쓰이는 암호학적 식별자 또는 인코딩이다. Bitcoin Developer 문서는 트랜잭션 출력이 스크립트에 가치를 잠그고, 입력이 이전 출력을 만족시키는 구조라고 설명한다.[^ref-btc-dev-transactions]

즉:

```text
address/key observed on-chain
    !=
verified legal identity
```

이 등식이 강해지는 것은 오직 오프체인 증거가 주소, 키, 또는 클러스터를 사람, 서비스, 수탁자, 거래소, 기관과 연결할 때뿐이다.

### 새로운 키 쌍

각 수취마다 새 키 쌍을 쓰면 직접적인 주소 재사용 링크를 줄일 수 있다. BIP32는 계층형 지갑 구조에서 많은 키를 결정론적으로 생성할 수 있게 하므로, 각 개별 키를 따로 백업하지 않고도 fresh-key 사용을 عملي하게 만든다.[^ref-bip-0032]

새로운 키는 특정 링크 벡터 하나를 줄일 뿐이다. 다중 입력 지출, 거스름돈 패턴, 금액 상관관계, 타이밍, 네트워크 데이터, 외부 신원 정보는 여전히 활동을 연결할 수 있다.

### 다중 입력 링크

하나의 트랜잭션이 여러 입력을 소비한다면, 각 입력은 지출 승인이 필요하다. 백서는 다중 입력 트랜잭션이 입력들이 같은 소유자에게 속했음을 드러낸다고 말한다.[^ref-btc-wp]

현대 분석은 이것을 더 조심스럽게 표현해야 한다.

```text
multi-input transaction
    -> evidence of common spending control
    -> often interpreted as common ownership
    -> not absolute proof in collaborative transactions
```

---

## 6. 기술적 메커니즘

### 주소 재사용

주소 재사용은 동일한 수신 식별자를 여러 번 쓰는 것이다. 이 경우 여러 출력이 같은 주소나 스크립트 패턴과 연결되므로 직접적인 링크가 생긴다.

백서가 제안한 완화책은 새 키 쌍을 사용하는 것이다. 현대 지갑에서는 deterministic wallet가 시드에서 많은 수신 키를 생성할 수 있기 때문에, 사용자는 백업 편의성을 유지하면서도 재사용을 피할 수 있다.[^ref-bip-0032]

### 거스름돈 링크

섹션 9는 가치 분할을 설명했다. 전형적인 트랜잭션은 수취인 출력 하나와 거스름돈 출력 하나를 가진다. 하지만 거스름돈은 온체인에 표시되지 않기 때문에, 분석가는 다음과 같은 휴리스틱으로 이를 추론한다.

- 출력 script type
- 주소 재사용
- 출력 금액 패턴
- 지갑 fingerprint
- 이후 지출 행동
- round-number payment 가정
- 알려진 서비스 동작

이 휴리스틱은 유용할 수 있지만, 합의 사실은 아니다.

### 다중 입력 공통 소유 휴리스틱

다중 입력 휴리스틱은 다음과 같다.

```text
If multiple inputs are spent in the same transaction,
they are likely controlled by the same entity.
```

이 증거 기반은 일반적인 지갑 트랜잭션에 대해서는 강하다. 지출자는 모든 입력에 대해 승인을 제공해야 하기 때문이다. 그러나 여러 당사자가 의도적으로 하나의 트랜잭션을 함께 만들면 이 휴리스틱은 실패할 수 있다.

따라서 올바른 분류는 다음과 같다.

```text
FACT:
  these inputs were spent together in one transaction

INTERPRETATION:
  these inputs probably share common control

HEURISTIC:
  cluster these inputs as one entity
```

### 신원 링크

공개키, 주소, 또는 클러스터 하나가 실제 세계 신원과 연결되는 순간, 그 클러스터와 연결된 과거 및 미래 트랜잭션은 훨씬 더 쉽게 분석될 수 있다. 백서는 공개키를 소유자와 연결하는 것이 리스크라고 명시적으로 경고한다.[^ref-btc-wp]

이 때문에 거래소 입금, 출금 기록, 상인 인보이스, 공개 기부 주소, 소송 기록, 제재 리스트, 유출 데이터베이스가 온체인 attribution의 증거 수준을 크게 바꿀 수 있다.

---

## 7. 수학적 또는 분석적 모델

### 그래프 모델

Bitcoin은 방향 그래프로 모델링할 수 있다.

```text
transaction outputs -> transaction inputs -> new transaction outputs
```

분석가는 이를 종종 더 높은 수준의 그래프로 변환한다.

| Graph | Node | Edge |
|---|---|---|
| Transaction graph | Transaction | Spend relationship |
| UTXO graph | Output | Spent-by relationship |
| Address graph | Address/script | Co-spend or transfer relation |
| Entity graph | Cluster | Inferred control relation |

직접 체인 데이터에서 관측 가능한 것은 낮은 수준의 transaction graph와 UTXO graph뿐이다. Address graph와 entity graph에는 해석이 들어간다.

### 링크 신뢰 수준

유용한 신뢰 모델은 다음과 같다.

```text
observable fact
    -> protocol-level certainty

single heuristic
    -> limited confidence

multiple independent heuristics
    -> moderate confidence

on-chain + verified off-chain evidence
    -> higher confidence
```

예시:

```text
same address reused
    FACT: outputs share the same address/script
    INTERPRETATION: likely same receiver context

multi-input spend
    FACT: inputs were spent together
    HEURISTIC: likely common control

exchange KYC record + deposit address
    FACT if authenticated record is verified
    INTERPRETATION: address belongs to that customer or account context
```

### 프라이버시 손실 누적

프라이버시 손실은 누적적이다.

```text
address reuse
+ multi-input consolidation
+ identifiable exchange flow
+ distinctive amount/timing pattern
+ public disclosure
= stronger linkage
```

하나의 신호만으로는 신원을 증명하기 어렵다. 하지만 여러 독립 신호가 쌓이면 신뢰도가 높아질 수 있다.

---

## 8. 보안 가정

### Bitcoin 프라이버시가 가정하는 것

Bitcoin의 프라이버시 모델은 다음을 가정한다.

1. 사용자는 새 키를 만들 수 있다.
2. 사용자는 주소 재사용을 피한다.
3. 공개키나 주소가 자동으로 실제 세계 신원과 연결되지 않는다.
4. 지갑 구성은 불필요하게 서로 무관한 입력을 합치지 않는다.
5. 관찰자는 외부 정보 없이는 모든 가명을 사람과 안정적으로 매핑할 수 없다.

### 한계

이 가정은 쉽게 깨질 수 있다.

- 사용자가 주소를 재사용한다.
- 거래소가 신원 정보를 수집한다.
- 상인이 인보이스 주소를 재사용한다.
- 지갑이 UTXO를 통합한다.
- 거스름돈이 식별 가능하다.
- 네트워크 메타데이터가 트랜잭션 발신지를 노출한다.
- 공개적 발언이 주소와 신원을 연결한다.
- 수탁자가 많은 사용자를 하나의 배칭 트랜잭션으로 묶는다.

섹션 10은 강한 익명성을 약속하지 않는다. 이것은 행동, 지갑 설계, 그리고 링크 증거의 부재에 의존하는 프라이버시 모델을 설명할 뿐이다.

---

## 9. Bitcoin Core 구현

### Core 검증 vs 프라이버시

Bitcoin Core의 합의 검증은 실제 세계 신원을 부여하지 않는다. 그것은 트랜잭션 구조, 스크립트, UTXO 지출, 블록 포함, 체인 상태를 검증한다. 프라이버시는 별도의 합의 규칙이 아니다.

이 점은 중요하다.

```text
valid transaction
    !=
private transaction
```

어떤 트랜잭션은 완전히 유효하면서도 주소 재사용, 통합, 금액 패턴, 외부 링크를 통해 많은 정보를 드러낼 수 있다.

### 지갑의 주소 재사용 제어

Bitcoin Core는 주소 재사용과 관련된 wallet-level control을 노출한다. `createwallet` RPC에는 `avoid_reuse` 옵션이 있으며, 이는 지갑이 dirty address에서 지출하는 것을 피하도록 제어할 수 있고 wallet flag를 통해 토글될 수 있다고 문서에 설명되어 있다.[^ref-btc-core-rpc-createwallet]

이것은 지갑 동작이지 합의가 아니다. 즉 주소 재사용이 운영상 중요하긴 하지만, 노드는 주소를 재사용했다는 이유만으로 트랜잭션을 거부하지 않는다.

### 결정론적 키 생성

BIP32는 합의 규칙이 아니다. 그것은 hierarchical deterministic wallet를 위한 표준이다. 여기서의 중요성은 실무적이다. 즉, 백서가 권장한 "새 키 쌍 사용"을 대규모로 구현하기 쉽게 만든다.[^ref-bip-0032]

---

## 10. 온체인 함의

### 강한 관측 사실

분석가는 다음을 직접 관측할 수 있다.

- 트랜잭션 입력
- 트랜잭션 출력
- 재사용된 스크립트 또는 주소
- 다중 입력 지출
- 출력 금액
- 트랜잭션 타이밍
- UTXO 통합
- fan-out 패턴
- 명시적 규칙으로 만든 주소 클러스터

### 추론 가능한 것

분석가는 다음을 추론할 수 있다.

- 공통 입력 통제로 보이는 패턴
- 거스름돈 출력으로 보이는 항목
- 거래소 배칭으로 보이는 패턴
- self-transfer로 보이는 패턴
- 특정 서비스 지갑 동작
- 커스터디 플로우 가능성
- 지급 흐름 가능성

각 추론에는 주의사항이 필요하다.

### 알 수 없는 것

체인 데이터만으로는 보통 다음을 증명할 수 없다.

- 법적 신원
- 실질적 소유자
- 의도
- 오프체인 비즈니스 목적
- 협업형 트랜잭션에 정말 여러 소유자가 있었는지
- 거스름돈 휴리스틱이 맞는지
- 서비스가 내부적으로 특정 사용자에게 크레딧했는지

---

## 11. 기관 관점에서의 해석

### 리서치 규율

기관 수준의 프라이버시 분석은 반드시 다음을 분리해야 한다.

```text
observable graph fact
from
ownership inference
from
identity attribution
from
intent claim
```

이렇게 해야 온체인 분석에서 가장 흔한 오류, 즉 그래프 휴리스틱을 곧바로 입증된 신원 진술로 취급하는 실수를 막을 수 있다.

### 증거 점수화

권장 점수화:

| Claim Type | Example | Confidence |
|---|---|---|
| Observable fact | Address X appears in output Y | High |
| Structural relation | Inputs A and B were spent together | High |
| Heuristic cluster | Inputs A and B share common control | Moderate |
| Identity claim | Cluster belongs to Entity Z | Depends on off-chain evidence |
| Intent claim | Entity Z tried to hide funds | Low without direct evidence |

### 기관 통제

분석가는 다음을 해야 한다.

- 원시 트랜잭션 증거를 보존하고,
- 모든 클러스터링 규칙을 문서화하며,
- 대안 설명을 함께 보존하고,
- CoinJoin 유사 또는 협업 패턴을 휴리스틱 붕괴 사례로 표시하고,
- 단일 신호로 attribution하지 않으며,
- 서비스 수준 지갑과 최종 사용자 지갑을 구분하고,
- 내부 기록 없이 거래소 입출금 트랜잭션을 곧바로 개별 사용자 소유로 해석하지 않아야 한다.

---

## 12. 흔한 오해

### Misinterpretation 1: Bitcoin is anonymous.

틀렸다. Bitcoin은 익명 시스템이 아니라 가명 시스템이다. 트랜잭션은 공개되고, 키와 주소는 본질적으로 실제 세계 신원이 아니다.

### Misinterpretation 2: A new address guarantees privacy.

과장이다. 새 주소는 직접적인 주소 재사용 링크를 줄여주지만, 다중 입력 지출, 거스름돈, 타이밍, 금액, 오프체인 데이터는 여전히 활동을 연결할 수 있다.

### Misinterpretation 3: Multi-input transactions prove one owner.

과장이다. 일반적인 지갑 동작에서는 공통 지출 통제를 강하게 시사하지만, 협업형 트랜잭션은 이 소유권 휴리스틱을 깨뜨릴 수 있다.

### Misinterpretation 4: Change outputs are obvious.

틀렸다. 거스름돈은 표시되지 않는다. 추론할 뿐이다.

### Misinterpretation 5: Bitcoin Core prevents privacy mistakes at consensus level.

틀렸다. 합의는 트랜잭션 유효성만 검증한다. 주소 재사용을 막거나 익명성을 강제하지 않는다.

### Misinterpretation 6: A cluster label proves identity.

틀렸다. 클러스터 라벨은 attribution claim이다. 그 신뢰 수준은 증거 품질에 달려 있다.

---

## 13. 연구 질문

1. Bitcoin 사용자는 서로 다른 script type에 걸쳐 주소를 얼마나 자주 재사용하는가?
2. 현대 지갑 동작 아래에서 흔한 change heuristic의 정확도는 어느 정도인가?
3. 협업형 트랜잭션은 다중 입력 클러스터링을 얼마나 자주 깨뜨리는가?
4. 분석가는 ownership을 과도하게 단정하지 않으면서 consolidation을 어떻게 식별해야 하는가?
5. 거래소 배칭 패턴은 entity clustering에 어떤 영향을 주는가?
6. 지갑 기본 설정은 주소 재사용률에 어떤 영향을 주는가?
7. 기관 보고서에서 attribution confidence는 어떻게 표현해야 하는가?
8. 어떤 오프체인 소스가 identity attribution confidence를 실질적으로 높이는가?
9. 분석가는 서비스 지갑과 최종 사용자 지갑을 어떻게 구분해야 하는가?
10. self-custody 지갑과 custodial platform 사이에서 프라이버시 리스크는 어떻게 다른가?

---

## 14. Practical Exercises

1. 여러 입력을 가진 트랜잭션 하나를 고르고, 다음을 분류하라.
   - observable fact;
   - common-control inference;
   - identity claim.
2. 두 개의 출력을 가진 트랜잭션을 찾고, 왜 그중 하나가 거스름돈일 수 있는지 설명하라.
3. 거스름돈 추론이 틀릴 수 있는 이유 두 가지를 제시하라.
4. 재사용된 주소가 새 주소보다 왜 더 강한 직접 링크를 만드는지 설명하라.
5. BIP32가 합의를 바꾸지 않으면서도 fresh-key wallet 운영을 어떻게 지원하는지 설명하라.
6. cluster label을 과장하지 않는 attribution note를 작성하라.

---

## 15. 증거 분류

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Whitepaper Section 10 privacy model, fresh keys, and multi-input limitation | A |
| REF-BTC-DEV-TRANSACTIONS-001 | Official developer documentation | Transaction input/output and script model | A |
| REF-BTC-DEV-OPERATING-001 | Official developer documentation | Full-node and SPV operating model context | A |
| REF-BIP-0032 | BIP | Hierarchical deterministic wallet key derivation | A |
| REF-BTC-CORE-RPC-CREATEWALLET-001 | Official RPC documentation | Bitcoin Core wallet `avoid_reuse` option | B |

### Claim Ledger

| Claim ID | Claim | Classification | Evidence | Source |
|---|---|---|---|---|
| C001 | The whitepaper contrasts Bitcoin privacy with traditional banking privacy. | FACT | A | REF-BTC-WP-001 |
| C002 | Bitcoin transactions are publicly announced. | FACT | A | REF-BTC-WP-001 |
| C003 | The whitepaper recommends using a new key pair for each transaction. | FACT | A | REF-BTC-WP-001 |
| C004 | The whitepaper identifies multi-input transactions as a linkage risk. | FACT | A | REF-BTC-WP-001 |
| C005 | Transaction outputs lock value to scripts, and inputs satisfy previous outputs. | FACT | A | REF-BTC-DEV-TRANSACTIONS-001 |
| C006 | BIP32 standardizes hierarchical deterministic wallet key derivation. | FACT | A | REF-BIP-0032 |
| C007 | Bitcoin Core exposes an `avoid_reuse` wallet option. | FACT | B | REF-BTC-CORE-RPC-CREATEWALLET-001 |
| C008 | Change-output identification is heuristic rather than a consensus fact. | INTERPRETATION | B | REF-BTC-WP-001; REF-BTC-DEV-TRANSACTIONS-001 |
| C009 | Multi-input common ownership is useful but fallible. | HEURISTIC | B | REF-BTC-WP-001 |
| C010 | Identity attribution requires off-chain or additional corroborating evidence. | INTERPRETATION | B | REF-BTC-WP-001 |

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
BITCOIN-011 Privacy
|
+-- interprets: Whitepaper Section 10
|
+-- privacy_model
|   +-- public: transactions
|   +-- pseudonymous: keys/addresses
|   +-- recommended: fresh key pairs
|
+-- linkage_risks
|   +-- address reuse
|   +-- multi-input spends
|   +-- change detection
|   +-- timing and amount patterns
|   +-- off-chain identity links
|
+-- analysis_layers
|   +-- observable transaction graph
|   +-- heuristic clustering
|   +-- identity attribution
|   +-- intent interpretation
|
+-- leads_to: BITCOIN-012 Calculations
```

---

## 17. 참고문헌

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Section 10, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-transactions]: Bitcoin Developer Documentation, "Transactions," transaction inputs, outputs, and scripts, https://developer.bitcoin.org/reference/transactions.html, accessed 2026-08-04.

[^ref-btc-dev-operating]: Bitcoin Developer Documentation, "Operating Modes," full node and SPV verification context, https://developer.bitcoin.org/devguide/operating_modes.html, accessed 2026-08-04.

[^ref-bip-0032]: Pieter Wuille, "BIP32: Hierarchical Deterministic Wallets," Bitcoin Improvement Proposals, https://bips.dev/32/, accessed 2026-08-04.

[^ref-btc-core-rpc-createwallet]: Bitcoin Developer Documentation, "createwallet RPC," `avoid_reuse` wallet option, https://developer.bitcoin.org/reference/rpc/createwallet.html, accessed 2026-08-04.

---

## 18. 교차 참조

### Parent

- Bitcoin Whitepaper

### Previous

- BITCOIN-010 — Whitepaper Section 9 — Combining and Splitting Value

### Next

- BITCOIN-012 — Whitepaper Section 11 — Calculations

### Related

- BITCOIN-003 — Whitepaper Section 2 — Transactions
- BITCOIN-009 — Whitepaper Section 8 — Simplified Payment Verification
- BITCOIN-014 — UTXO Model
- BITCOIN-015 — Transactions in Depth
- BITCOIN-017 — Mempool
- BITCOIN-021 — Nodes & Network Propagation

---

## Review Status

### Technical Review

Passed.

- Whitepaper Section 10의 프라이버시 주장과 현대 지갑 동작을 분리했다.
- 공개 트랜잭션 가시성, 가명적 키, fresh-key 실천, 다중 입력 링크, change heuristic을 분리했다.
- BIP32를 합의 규칙이 아닌 wallet key derivation standard로 한정했다.
- Bitcoin Core의 `avoid_reuse`를 합의 검증이 아닌 wallet behavior로 한정했다.

### Evidence Review

Passed.

- Whitepaper Section 10 관련 주장은 백서를 직접 인용한다.
- 트랜잭션 구조 관련 주장은 공식 Bitcoin Developer 문서를 인용한다.
- HD wallet 관련 주장은 BIP32를 인용한다.
- 지갑 구현 동작은 공식 RPC 문서를 인용한다.
- 클러스터링과 attribution 주장은 해석 또는 휴리스틱으로 표시했다.

### Editorial Review

Passed.

- Markdown 헤딩은 프로젝트 딥다이브 구조를 따른다.
- 메타데이터는 완전하다.
- 표와 코드 펜스는 닫혀 있다.
- privacy, pseudonymity, public key, address reuse, change, multi-input heuristic, attribution 용어를 일관되게 사용했다.

### Adversarial Review

Passed.

- Bitcoin이 익명이라고 주장하지 않았다.
- 새 주소가 프라이버시를 보장한다고 주장하지 않았다.
- 클러스터링을 신원 증명으로 취급하지 않았다.
- 관측 가능한 그래프 사실과 ownership/intent 해석을 분리했다.

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
