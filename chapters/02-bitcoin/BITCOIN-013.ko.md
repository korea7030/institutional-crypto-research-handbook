---
knowledge_id: BITCOIN-013
title: 백서 섹션 12 — 결론(Conclusion)
subtitle: 신뢰 없는 전자 현금, 작업증명 기반 순서화, 공개 검증, 그리고 Bitcoin 설계의 종합
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 85 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Whitepaper
  - Protocol Design
  - Consensus
parent:
  - Bitcoin Whitepaper
prerequisites:
  - BITCOIN-001
  - BITCOIN-005
  - BITCOIN-006
  - BITCOIN-009
  - BITCOIN-012
related_topics:
  - Peer-to-Peer Electronic Cash
  - Proof of Work
  - Double Spend
  - Chain Selection
  - Probabilistic Finality
  - Incentives
  - Privacy
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-BLOCKCHAIN-001
  - REF-BTC-DEV-OPERATING-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-POW-001
tags:
  - bitcoin
  - whitepaper
  - conclusion
  - protocol-design
  - proof-of-work
  - consensus
  - trust-minimization
---

# 백서 섹션 12 — 결론(Conclusion)
> Deep Dive Series  
> Research Unit: BITCOIN-013

---

## Research Brief

```yaml
knowledge_id: BITCOIN-013
title: Whitepaper Section 12 — Conclusion
research_question: >
  How does the Bitcoin Whitepaper's conclusion synthesize digital signatures,
  proof-of-work, peer-to-peer networking, incentives, and probabilistic
  settlement into a system for electronic transactions without trusted third
  parties?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-001
  - BITCOIN-005
  - BITCOIN-006
  - BITCOIN-009
  - BITCOIN-012
parent: Bitcoin Whitepaper
previous: BITCOIN-012
next: BITCOIN-014
related_topics:
  - Trust Minimization
  - Transaction Chain
  - Proof-of-Work Chain
  - Network Propagation
  - Incentives
  - Privacy
  - Confirmation Risk
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
  - Full history of Bitcoin after 2009
  - Complete Bitcoin Core architecture
  - Monetary investment thesis
  - Non-Bitcoin consensus comparison
  - Legal or regulatory classification
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- 백서의 마지막 주장을 정확히 요약할 수 있다.
- 디지털 서명이 소유권 이전을 어떻게 정의하는지 설명할 수 있다.
- 공개 역사를 순서화하기 위해 왜 작업증명이 쓰이는지 설명할 수 있다.
- 왜 네트워크가 신뢰된 중앙 서버를 필요로 하지 않는지 설명할 수 있다.
- 인센티브가 정직한 참여를 어떻게 뒷받침하는지 설명할 수 있다.
- 왜 Bitcoin의 finality가 확률적인지 설명할 수 있다.
- 왜 전체 검증이 여전히 필요한지 설명할 수 있다.
- trust minimization과 trust elimination을 구분할 수 있다.
- 어떤 백서 주장이 설계 주장이고, 어떤 주장이 현대 구현 증거를 필요로 하는지 식별할 수 있다.
- 백서 결론을 이후 Bitcoin internals 주제와 연결할 수 있다.

---

## 2. 핵심 질문

1. 결론은 Bitcoin이 어떤 문제를 해결한다고 주장하는가?
2. 디지털 서명의 역할은 무엇인가?
3. 작업증명의 역할은 무엇인가?
4. 왜 시스템은 P2P 네트워크를 사용하는가?
5. 네트워크는 경쟁하는 역사들을 어떻게 해결하는가?
6. 왜 인센티브가 필요한가?
7. 결론은 이중지불 방지에 대해 무엇을 시사하는가?
8. 결론만으로는 무엇이 증명되지 않는가?
9. 현대 Bitcoin Core는 설계의 검증 측면을 어떻게 구현하는가?
10. 왜 "trustless"보다 "trust-minimized"가 더 정확한가?
11. 기관 리서처는 백서 전체에서 무엇을 가져가야 하는가?
12. 백서 시리즈를 마친 뒤 자연스럽게 이어지는 주제는 무엇인가?

---

## 3. Executive Summary

백서 섹션 12는 Bitcoin이 신뢰에 의존하지 않는 전자 거래 시스템을 제안한다고 결론짓는다. 설계는 소유권을 정의하는 디지털 서명에서 출발하지만, 서명만으로는 이중지불을 막을 수 없다. Bitcoin은 P2P 네트워크와 작업증명을 사용해 점점 더 바꾸기 어려워지는 공개 역사를 기록함으로써 이 순서 문제를 해결한다.[^ref-btc-wp]

결론은 앞선 섹션들을 하나로 묶는다.

| Component | Role |
|---|---|
| Digital signatures | 코인 이전을 승인 |
| Public transaction history | 이중지불 탐지를 가능하게 함 |
| Proof of work | 비용이 드는 작업으로 경쟁하는 역사를 순서화 |
| Peer-to-peer relay | 트랜잭션과 블록 전파 |
| Longest/greatest-work chain | 수용되는 유효한 역사 선택 |
| Incentives | 블록 생산자에게 네트워크 지원 보상 |
| Merkle trees and SPV | 일부 사용자에게 검증 및 저장 비용 절감 |
| Privacy model | 링크를 피하면 신원과 공개키를 분리 |

백서의 최종 주장은 Bitcoin이 모든 형태의 신뢰, 리스크, 운영 판단을 제거한다는 뜻이 아니다. 프로토콜 가정 아래에서, 신뢰된 중앙 타임스탬프 기관이나 정산 중개자의 필요를 제거한다는 뜻이다. 사용자는 여전히 암호학, 소프트웨어 정확성, 네트워크 연결성, 경제적 인센티브, 독립 검증에 의존한다.

현대 Bitcoin Core는 이 분리를 구현한다. 중앙 운영자를 사회적으로 신뢰하는 대신, 구현 로직으로 작업증명, 컨텍스트 기반 헤더 규칙, 블록 구조, 트랜잭션 유효성, UTXO 지출, 체인 활성화를 검증한다.[^ref-btc-core-pow][^ref-btc-core-validation]

기관 리서처에게 백서는 다음과 같은 시스템 설계 주장으로 읽혀야 한다.

```text
ownership authorization
    + public ordering
    + costly rewrite mechanism
    + peer-to-peer propagation
    + economic incentives
    =
trust-minimized settlement under explicit assumptions
```

---

## 4. 원래 설계

결론은 논문이 신뢰에 의존하지 않는 전자 거래 시스템을 제안했다고 말한다. 출발점은 디지털 서명으로 이루어진 일반적인 코인 개념이지만, 서명만으로는 이중지불 문제를 해결할 수 없다고 설명한다.[^ref-btc-wp]

논문이 추가한 메커니즘은 작업증명을 사용하는 P2P 네트워크다. 이 네트워크는 거래의 공개 역사를 기록하고, 공격자가 그 역사를 바꾸려면 작업증명을 다시 해야 하며, 정직한 노드가 체인을 계속 연장할수록 그 비용은 커진다.[^ref-btc-wp]

결론은 또한 실용적인 네트워크 동작을 강조한다. 노드는 네트워크를 떠났다가 다시 돌아와도, 자신이 없던 동안 무슨 일이 있었는지에 대한 증거로 가장 긴 작업증명 체인을 수용할 수 있다고 말한다.[^ref-btc-wp]

현대 용어로는 "longest chain"을 다음처럼 다듬어야 한다.

```text
valid chain with the most cumulative proof of work
```

Bitcoin Developer 문서 역시 포크가 발생했을 때 노드가 재생성하기 가장 어려운 유효 체인을 따른다고 설명한다.[^ref-btc-dev-blockchain]

---

## 5. 프로토콜 구조

### 백서 시스템 맵

```text
User creates transaction
    |
    v
Digital signature authorizes spend
    |
    v
Transaction is broadcast to peer-to-peer network
    |
    v
Miners collect transactions into candidate blocks
    |
    v
Proof of work makes block proposal costly
    |
    v
Nodes validate block and transactions
    |
    v
Valid chain with most cumulative work becomes local active history
    |
    v
Later confirmations increase rewrite cost
```

### 신뢰 경계

Bitcoin은 트랜잭션 순서와 정산을 위해 중앙 발행자나 결제 처리자를 신뢰할 필요를 제거한다. 하지만 더 정확히 말하면, 몇 가지 기술적·경제적 가정에 의존한다.

| Reliance | Meaning |
|---|---|
| Cryptographic assumptions | 서명과 해시가 기대대로 동작 |
| Software correctness | 노드 구현이 합의 규칙을 올바르게 강제 |
| Network reachability | 유효한 블록과 트랜잭션이 충분히 전파 |
| Honest-majority work assumption | 공격자가 역사를 안정적으로 다시 쓸 만큼의 작업량을 지속적으로 확보하지 못함 |
| Economic incentives | 평상시에는 채굴자가 공격보다 유효한 블록 보상을 선호 |
| User operation | 사용자가 자신의 리스크 수준에 맞게 검증 |

그래서 "trustless"는 부정확하다. Bitcoin은 trust-minimized이고 verification-oriented한 시스템이라고 보는 편이 맞다.

---

## 6. 기술적 메커니즘

### 디지털 서명은 필요하지만 충분하지 않다

디지털 서명은 코인을 지출할 권한이 있음을 증명한다. 그러나 동일한 코인이 이미 다른 곳에서 지출되지 않았다는 사실까지는 그 자체로 증명하지 못한다. 이중지불 문제는 순서와 유일성의 문제다.

백서의 구조는 다음과 같다.

```text
signature chain
    -> ownership transfer

public proof-of-work chain
    -> ordering and double-spend resistance
```

### 역사에 무게를 부여하는 작업증명

작업증명은 역사에 "무게"를 부여한다. 과거 트랜잭션을 바꾸면 그 트랜잭션이 들어 있는 블록이 바뀌고, 그 블록의 해시가 바뀌며, 이후 작업증명 링크도 무효화된다. 공격자는 바뀐 블록을 다시 채굴해야 하고, 이후의 정직한 작업까지 따라잡아야 한다.

이것이 확률적 정산을 만든다.

```text
more confirmations
    -> more work above transaction
    -> higher replacement cost
    -> lower modeled attack probability when attacker has less hashpower
```

섹션 11은 이 따라잡기 확률을 수량화했다.[^ref-btc-wp]

### 네트워크 재합류

결론은 노드가 네트워크를 떠났다가 다시 돌아와도, 자신이 없는 동안 일어난 일의 증거로 작업증명 체인을 받아들일 수 있다고 말한다.[^ref-btc-wp]

그렇다고 노드가 임의의 헤더를 검증 없이 받아들인다는 뜻은 아니다. 현대 풀노드는 헤더, 블록, 트랜잭션, 체인 상태를 모두 검증한다. Bitcoin Developer 문서는 full node가 제네시스부터 검증하는 반면, SPV는 헤더와 머클 증명을 사용하는 더 약한 대안이라고 설명한다.[^ref-btc-dev-operating]

---

## 7. 수학적 또는 경제적 모델

### 작업량과 확률의 종합

결론은 섹션 11 모델에 의존한다.

```text
p = honest block-production probability
q = attacker block-production probability
z = honest-chain lead
```

`q < p`라면 정직 체인의 리드가 커질수록 공격자의 따라잡기 확률은 감소한다. `q >= p`라면 더 이상 confirmation만으로는 안전을 보장할 수 없다.[^ref-btc-wp]

### 인센티브 층

백서의 인센티브 섹션은 채굴자가 정직하게 참여할 경제적 이유를 제공한다. 수용된 블록은 보조금과 거래 수수료를 가져갈 수 있기 때문이다. 결론은 이 인센티브 층에 의존하는데, 작업증명 보안은 비용이 드는 자원이기 때문이다.

단순화하면:

```text
block reward + fees
    -> miner revenue opportunity
    -> hashpower supplied to valid chain
    -> higher cost to rewrite accepted history
```

이것은 백서 설계에 대한 해석이지, 보안을 결정론적으로 계산하는 공식은 아니다. 실제 채굴자 행동은 BTC 가격, 수수료, 하드웨어, 에너지 비용, 풀 인센티브, 규제, 공격 동기에 달려 있다.

### Confirmation 정책

결론은 보편적 confirmation 임계값을 규정하지 않는다. confirmation 정책은 리스크 의사결정이다.

```text
settlement confidence = function(
    confirmation depth,
    cumulative work,
    transaction value,
    attacker model,
    network view,
    validation method,
    business reversibility
)
```

---

## 8. 보안 가정

### 결론이 성립하려면 필요한 가정

결론은 다음에 의존한다.

1. 사용자가 서명과 트랜잭션 유효성을 검증할 수 있어야 한다.
2. 노드가 네트워크를 통해 경쟁하는 블록과 트랜잭션을 학습할 수 있어야 한다.
3. 작업증명은 공격자에게 여전히 비싸야 한다.
4. 정직한 채굴자는 보통 유효한 역사를 연장해야 한다.
5. 풀노드는 작업증명이 붙어 있어도 무효 블록을 거부해야 한다.
6. 가장 큰 누적 작업량을 가진 유효 체인이 결국 참여자에게 보이게 되어야 한다.
7. 사용자는 자신의 리스크 수준에 맞게 충분한 confirmation을 기다려야 한다.

### 결론이 주장하지 않는 것

결론은 다음을 주장하지 않는다.

- 결정론적 finality
- 완전한 프라이버시
- 0의 운영 리스크
- 지갑 탈취에 대한 보호
- SPV와 full validation의 동등성
- 채굴자가 무효 트랜잭션을 유효하게 만들 수 있다는 것
- 모든 참여자가 같은 네트워크 시야를 가진다는 것
- 트랜잭션 그래프 분석이 불가능하다는 것

### 남는 리스크

백서 설계가 의도대로 동작해도, 사용자는 여전히 다음 리스크를 가진다.

- 개인키 분실 또는 도난
- 소프트웨어 버그
- Eclipse 또는 분할된 네트워크 시야
- 낮은 confirmation 상태의 이중지불 리스크
- 거래소나 수탁기관 실패
- 주소 재사용과 클러스터링에 따른 프라이버시 유출
- 수수료 시장과 confirmation 지연의 불확실성

---

## 9. Bitcoin Core 구현

### 현대 구현에서 설계가 표현되는 방식

Bitcoin Core는 백서 그 자체는 아니지만, 현재 Bitcoin 동작에 대한 1차 구현 증거다. 이것은 신뢰된 조정자 대신 검증 코드로 설계를 구현한다.

관련 구현 경계는 다음과 같다.

| Component | Role |
|---|---|
| `CheckProofOfWork` / `DeriveTarget` | 목표값 규칙에 맞는 헤더 작업량 검증 |
| `ContextualCheckBlockHeader` | 요구 난이도 비트 같은 컨텍스트 기반 헤더 규칙 검증 |
| `CheckBlock` | 블록 구조, 머클 루트, 코인베이스 위치, 관련 블록 규칙 검증 |
| `ConnectBlock` | 유효한 블록 내용을 UTXO 상태에 연결하고 트랜잭션 효과 검증 |
| Active-chain logic | 작업량과 검증 상태에 따라 유효 체인 후보를 선택하고 활성화 |

이 구현 조각들은 백서의 결론을 뒷받침한다. 노드는 어떤 트랜잭션이 유효한지 중앙 당사자의 말을 믿지 않는다. 스스로 검증한다.[^ref-btc-core-pow][^ref-btc-core-validation]

### 합의 vs 로컬 정책

결론이 다루는 것은 합의 설계다. 이를 다음과 혼동하면 안 된다.

- mempool relay policy
- wallet fee estimation
- 거래소 confirmation rules
- 채굴자의 트랜잭션 선택 전략
- 블록 익스플로러 라벨
- 수탁 계정 잔액

이들은 중요하지만, Bitcoin의 합의 메커니즘 그 자체는 아니다.

---

## 10. 온체인 함의

### 백서 설계가 관측 가능하게 만드는 것

Bitcoin의 설계는 다음을 관측 가능하게 만든다.

- 트랜잭션
- 블록 헤더
- 트랜잭션 포함 여부
- 작업증명 체인의 성장
- confirmations
- reorg
- 트랜잭션 그래프 구조
- UTXO 지출과 생성
- 코인베이스 보상

이 관측 가능성이 있기 때문에 Bitcoin은 체인 데이터만으로 직접 분석할 수 있다.

### 여전히 해석이 필요한 것

다음은 해석 또는 외부 증거가 필요하다.

- 실제 세계 신원
- 트랜잭션 의도
- 채굴자 신원
- 지갑 소유권
- 거래소 고객 attribution
- 공격 동기
- reorg가 악의적이었는지 여부
- 거스름돈 판별이 맞는지 여부

백서의 투명성은 검증을 가능하게 하지만, 동시에 엄격한 증거 라벨링을 요구한다.

---

## 11. 기관 관점에서의 해석

### 백서를 리서치 프레임워크로 읽기

기관 리서처에게 Bitcoin 백서는 단지 역사 문서가 아니다. 이것은 Bitcoin 관련 주장을 평가하는 응축된 프레임워크다.

| Claim Type | Whitepaper Lens |
|---|---|
| Settlement | 이 트랜잭션은 얼마나 많은 작업량으로 보호되는가? |
| Validation | 누가 규칙을 검증했는가? |
| Trust | 어떤 중개자가 제거되었고, 어떤 가정은 남아 있는가? |
| Privacy | 무엇이 공개되고, 무엇이 단지 가명적인가? |
| Security | 어떤 공격자 모델을 가정하는가? |
| Incentives | 왜 채굴자는 프로토콜을 따를 것인가? |

### 실무 리서치 규칙

기관 분석은 다음을 따라야 한다.

- 중요한 결론에는 full-node 검증 데이터를 사용하고,
- 작업증명 confirmation과 법적 정산을 구분하며,
- 관측 가능한 그래프 사실과 attribution을 분리하고,
- confirmation threshold를 리스크 정책으로 다루며,
- Bitcoin을 완전히 trustless하다고 묘사하지 않고,
- 어떤 가정이 기술적, 경제적, 운영적인지 식별하며,
- 구현 동작이나 네트워크 조건이 바뀌면 결론도 갱신해야 한다.

### 백서 시리즈의 마무리와 다음 단계

섹션 12 이후 다음 단계는 백서 해석에서 Bitcoin internals로 넘어가는 것이다.

```text
Whitepaper concepts
    -> UTXO model
    -> transaction structure
    -> script
    -> mempool
    -> block and header internals
    -> mining
    -> difficulty
    -> network propagation
    -> forks and reorgs
```

이 전환이 중요한 이유는, 백서는 설계를 세웠지만 현대 분석에는 구현을 이해하는 프로토콜 지식이 필요하기 때문이다.

---

## 12. 흔한 오해

### Misinterpretation 1: Bitcoin eliminates all trust.

틀렸다. Bitcoin은 트랜잭션 순서와 정산에서 신뢰된 중개자 의존을 줄여준다. 하지만 여전히 암호학, 소프트웨어, 네트워크, 경제, 운영 가정에 의존한다.

### Misinterpretation 2: Digital signatures alone solve double spending.

틀렸다. 서명은 지출 권한을 부여할 뿐이다. 이중지불 저항은 작업증명 기반 순서화와 공개 검증이 담당한다.

### Misinterpretation 3: The longest chain means most blocks.

과장이다. 현대 분석에서는 가장 많은 누적 작업량을 가진 유효 체인으로 표현해야 한다.

### Misinterpretation 4: Bitcoin finality is deterministic.

틀렸다. Bitcoin은 확률적 정산을 제공한다. confirmation은 가정 아래 역전 확률을 낮춘다.

### Misinterpretation 5: Public transactions mean no privacy at all.

과장이다. Bitcoin은 가명 시스템이며, 프라이버시는 링크를 피하는지와 지갑/사용자 행동에 달려 있다.

### Misinterpretation 6: The whitepaper alone describes all current Bitcoin behavior.

틀렸다. 백서는 원래 설계 문서다. 현재 동작은 합의 규칙, Bitcoin Core 구현, BIP, 최신 프로토콜 문서를 함께 확인해야 한다.

---

## 13. 연구 질문

1. 오늘날 가장 강하게 유지되는 백서 가정은 무엇인가?
2. 지속적으로 가장 많이 모니터링해야 하는 백서 가정은 무엇인가?
3. 기관 보고서는 "trustless"를 어떻게 과장 없이 표현해야 하는가?
4. 커스터디, 거래소, 리테일 워크플로에 따라 confirmation 정책은 어떻게 달라져야 하는가?
5. 분석가는 백서 개념과 Bitcoin Core 구현 증거를 어떻게 결합해야 하는가?
6. 백서에 없는 현대 Bitcoin 동작에는 무엇이 있는가?
7. 온체인 분석에서 프라이버시 관련 주장은 어떻게 한정해서 표현해야 하는가?
8. reorg 리스크가 0이 아닐 때 settlement confidence는 어떻게 보고해야 하는가?
9. 백서를 읽은 뒤 반드시 알아야 할 Bitcoin internals 주제는 무엇인가?
10. popular narrative와 1차 자료가 충돌할 때 리서처는 어떻게 처리해야 하는가?

---

## 14. Practical Exercises

1. "trustless"라는 단어를 쓰지 않고 Bitcoin 설계를 한 단락으로 요약하라.
2. 왜 디지털 서명만으로는 이중지불이 해결되지 않는지 설명하라.
3. 신뢰된 결제 중개자를 제거한 뒤에도 남는 가정 세 가지를 적어라.
4. 6 confirmations을 받은 트랜잭션이 왜 수학적으로 final이 아닌지 설명하라.
5. Bitcoin 트랜잭션 그래프에서 observable fact 하나와 interpretation 하나를 식별하라.
6. 왜 백서 외에 현재 구현 증거가 추가로 필요한지 설명하는 짧은 연구 노트를 작성하라.

---

## 15. 증거 분류

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Whitepaper Section 12 conclusion and prior-section synthesis | A |
| REF-BTC-DEV-BLOCKCHAIN-001 | Official developer documentation | Forks, proof-of-work chain behavior, and most-difficult-chain explanation | A |
| REF-BTC-DEV-OPERATING-001 | Official developer documentation | Full-node and SPV operating models | A |
| REF-BTC-CORE-VALIDATION-001 | Primary implementation source | Block, transaction, contextual validation, and chain activation logic | A |
| REF-BTC-CORE-POW-001 | Primary implementation source | Proof-of-work target derivation and validation | A |

### Claim Ledger

| Claim ID | Claim | Classification | Evidence | Source |
|---|---|---|---|---|
| C001 | The whitepaper concludes that Bitcoin proposes electronic transactions without relying on trust. | FACT | A | REF-BTC-WP-001 |
| C002 | The whitepaper states that digital signatures alone do not solve double spending. | FACT | A | REF-BTC-WP-001 |
| C003 | The whitepaper uses proof of work to record public transaction history that becomes harder to change. | FACT | A | REF-BTC-WP-001 |
| C004 | Nodes can leave and rejoin and accept the proof-of-work chain as evidence of what happened while absent. | FACT | A | REF-BTC-WP-001 |
| C005 | Modern documentation describes valid forks as resolved by the most difficult chain to recreate. | FACT | A | REF-BTC-DEV-BLOCKCHAIN-001 |
| C006 | Full nodes provide stronger security than SPV by validating the chain. | FACT | A | REF-BTC-DEV-OPERATING-001 |
| C007 | Bitcoin Core validates proof of work through target derivation and hash comparison. | FACT | A | REF-BTC-CORE-POW-001 |
| C008 | Bitcoin Core validates block and transaction rules rather than trusting a central coordinator. | FACT | A | REF-BTC-CORE-VALIDATION-001 |
| C009 | Bitcoin is better described as trust-minimized than as eliminating all trust. | INTERPRETATION | B | REF-BTC-WP-001 |
| C010 | The next research phase should move from whitepaper interpretation into implementation-aware Bitcoin internals. | INTERPRETATION | B | REF-BTC-WP-001; REF-BTC-CORE-VALIDATION-001 |

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
BITCOIN-013 Conclusion
|
+-- synthesizes: Bitcoin Whitepaper Sections 1-12
|
+-- ownership_layer
|   +-- uses: digital signatures
|   +-- limitation: does not solve double spending alone
|
+-- ordering_layer
|   +-- uses: proof of work
|   +-- creates: public transaction history
|   +-- protects: rewrite cost
|
+-- network_layer
|   +-- uses: peer-to-peer relay
|   +-- allows: nodes leave and rejoin
|
+-- validation_layer
|   +-- uses: full nodes
|   +-- rejects: invalid blocks and transactions
|
+-- institutional_lens
    +-- trust minimization
    +-- probabilistic settlement
    +-- evidence-based on-chain analysis
    +-- implementation-aware research
```

---

## 17. 참고문헌

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Section 12 and supporting Sections 1-11, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-blockchain]: Bitcoin Developer Documentation, "Block Chain," forks, proof-of-work, and most-difficult-chain behavior, https://developer.bitcoin.org/devguide/block_chain.html, accessed 2026-08-04.

[^ref-btc-dev-operating]: Bitcoin Developer Documentation, "Operating Modes," full-node and SPV security model, https://developer.bitcoin.org/devguide/operating_modes.html, accessed 2026-08-04.

[^ref-btc-core-validation]: Bitcoin Core Contributors, `src/validation.cpp`, block validation, transaction connection, contextual checks, and active-chain processing, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/validation_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-pow]: Bitcoin Core Contributors, `src/pow.cpp`, `DeriveTarget`, `CheckProofOfWork`, `GetNextWorkRequired`, and related proof-of-work validation logic, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/pow_8cpp_source.html, accessed 2026-08-04.

---

## 18. 교차 참조

### Parent

- Bitcoin Whitepaper

### Previous

- BITCOIN-012 — Whitepaper Section 11 — Calculations

### Next

- BITCOIN-014 — UTXO Model

### Related

- BITCOIN-003 — Whitepaper Section 2 — Transactions
- BITCOIN-005 — Whitepaper Section 4 — Proof of Work
- BITCOIN-006 — Whitepaper Section 5 — Network
- BITCOIN-007 — Whitepaper Section 6 — Incentive
- BITCOIN-009 — Whitepaper Section 8 — Simplified Payment Verification
- BITCOIN-011 — Whitepaper Section 10 — Privacy
- POW-010 — Chain Selection
- POW-011 — Cumulative Chainwork
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- Whitepaper Section 12 주장과 현대 구현 세부 사항을 분리했다.
- 디지털 서명, 작업증명 순서화, 네트워크 릴레이, 인센티브, 확률적 정산을 분리했다.
- "Longest chain"을 누적 작업량 기준으로 번역하되, 문서 근거와 함께 한정했다.
- Bitcoin Core의 validation과 proof-of-work 참조는 구현 증거로 한정했다.

### Evidence Review

Passed.

- 백서 결론 관련 주장은 Section 12를 직접 인용한다.
- 체인 선택과 operating mode 관련 주장은 공식 Bitcoin Developer 문서를 인용한다.
- 현재 구현 관련 주장은 Bitcoin Core 소스를 인용한다.
- trust minimization과 기관 해석은 interpretation으로 표시했다.

### Editorial Review

Passed.

- Markdown 헤딩은 프로젝트 딥다이브 구조를 따른다.
- 메타데이터는 완전하다.
- 표와 코드 펜스는 닫혀 있다.
- trust minimization, proof of work, validation, public history, probabilistic settlement 용어를 일관되게 사용했다.

### Adversarial Review

Passed.

- Bitcoin이 모든 형태의 신뢰를 제거한다고 쓰지 않았다.
- 결정론적 finality를 주장하지 않았다.
- 백서를 현대 Bitcoin 전체 동작의 완전한 설명으로 취급하지 않았다.
- 관측 가능한 프로토콜 사실과 기관 해석을 분리했다.

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
