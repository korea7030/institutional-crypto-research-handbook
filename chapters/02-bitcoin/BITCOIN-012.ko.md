---
knowledge_id: BITCOIN-012
title: 백서 섹션 11 — 계산(Calculations)
subtitle: 공격자의 따라잡기 확률, Confirmation, Poisson 근사, 그리고 확률적 정산
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 95 min
estimated_study: 260 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Probability
  - Security
  - Proof of Work
parent:
  - Bitcoin Whitepaper
prerequisites:
  - BITCOIN-005
  - BITCOIN-009
  - BITCOIN-010
  - POW-010
  - POW-011
  - POW-014
related_topics:
  - Double Spend
  - Confirmations
  - Probabilistic Finality
  - Poisson Distribution
  - Chainwork
  - Settlement Risk
  - Attacker Hashrate
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-BLOCKCHAIN-001
  - REF-BTC-DEV-PAYMENT-PROCESSING-001
  - REF-BTC-DEV-OPERATING-001
tags:
  - bitcoin
  - whitepaper
  - calculations
  - confirmations
  - double-spend
  - probability
  - settlement-risk
---

# 백서 섹션 11 — 계산(Calculations)
> Deep Dive Series  
> Research Unit: BITCOIN-012

---

## Research Brief

```yaml
knowledge_id: BITCOIN-012
title: Whitepaper Section 11 — Calculations
research_question: >
  How does the Bitcoin Whitepaper model the probability that an attacker can
  catch up from behind, and how should institutions interpret confirmation
  depth as probabilistic settlement evidence rather than deterministic finality?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-005
  - BITCOIN-009
  - BITCOIN-010
  - POW-010
  - POW-011
  - POW-014
parent: Bitcoin Whitepaper
previous: BITCOIN-011
next: BITCOIN-013
related_topics:
  - Proof of Work
  - Chain Selection
  - Chainwork
  - Double-Spend Probability
  - Confirmations
  - Settlement Risk
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
  - Full stochastic-process proof
  - Modern empirical hashrate estimation
  - Fee-sniping and selfish-mining models
  - Exchange-specific confirmation policies
  - Real-time attack-cost pricing
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- 백서에서 말하는 공격자의 따라잡기 확률이 무엇인지 설명할 수 있다.
- 이중지불 경쟁 모델에서 `p`, `q`, `z`를 정의할 수 있다.
- 왜 `q < p`일 때 공격자의 성공 확률이 감소하는지 설명할 수 있다.
- 왜 `q >= p`이면 모델상 최종 따라잡기 확률이 `1`이 되는지 설명할 수 있다.
- 왜 백서가 Poisson 근사를 쓰는지 설명할 수 있다.
- confirmation 수와 결정론적 finality를 구분할 수 있다.
- 공격자 해시파워 가정이 리스크 추정을 어떻게 좌우하는지 설명할 수 있다.
- 백서의 계산이 무엇을 증명하고 무엇을 증명하지 못하는지 설명할 수 있다.
- confirmation 정책을 리스크 관리 의사결정으로 해석할 수 있다.
- "6 confirmations" 관행을 보편적 보장으로 취급하지 않을 수 있다.

---

## 2. 핵심 질문

1. 섹션 11은 어떤 공격을 계산하는가?
2. `p`, `q`, `z`는 각각 무엇을 의미하는가?
3. 왜 정직 체인의 리드가 중요한가?
4. 왜 이 공격이 임의보행(random walk)으로 모델링되는가?
5. 왜 백서는 Poisson 분포를 도입하는가?
6. 공격자가 정직한 채굴자와 같거나 더 큰 블록 생산 확률을 가지면 무슨 일이 일어나는가?
7. 왜 공격자의 해시파워가 더 작을 때 확률은 지수적으로 감소하는가?
8. confirmation 수는 실제로 무엇을 측정하는가?
9. 표의 값이 의미 있으려면 어떤 가정이 성립해야 하는가?
10. 섹션 11은 고액 결제 수락에 대해 무엇을 시사하는가?
11. 이 계산에 포함되지 않는 리스크는 무엇인가?
12. 기관은 이 모델을 어떻게 활용하되, 과도하게 의존하지 않아야 하는가?

---

## 3. Executive Summary

백서 섹션 11은 수취인이 confirmation을 기다린 뒤에도 공격자가 따라잡을 수 있는 확률을 계산한다. 공격 모델은 이중지불 경쟁이다. 정직 체인은 수취인의 트랜잭션을 포함하고, 공격자는 그 트랜잭션을 제외하거나 충돌시키는 대체 체인을 비공개로 만들려 한다.[^ref-btc-wp]

모델은 다음과 같이 정의한다.

```text
p = probability honest nodes find the next block
q = probability attacker finds the next block
z = number of blocks by which the honest chain is ahead
```

`q < p`라면, 공격자가 `z`블록 뒤에서 언젠가 따라잡을 확률은 다음처럼 감소한다.

```text
(q / p)^z
```

`q >= p`라면, 백서는 공격자의 최종 따라잡기 확률을 `1`로 둔다.[^ref-btc-wp]

그 다음 섹션은 지불 confirmation 모델을 더 정교하게 만든다. 정직 체인이 `z`개의 블록을 찾는 동안 공격자도 일부 비공개 블록을 찾았을 수 있기 때문이다. 백서는 공격자의 숨겨진 진척을 기대값이 다음인 Poisson 분포로 모델링한다.

```text
lambda = z * (q / p)
```

결과적으로, 공격자가 정직한 채굴자보다 의미 있게 적은 해시파워를 가질 때 confirmation 수가 늘수록 공격 확률이 급격히 줄어드는 표가 도출된다.[^ref-btc-wp]

이것이 Bitcoin에서 확률적 finality의 수학적 기초다. confirmation은 역전을 불가능하게 만들지 않는다. 다만 공격자가 넘어야 하는 누적 작업량을 늘리고, 명시된 가정 아래 성공 확률을 줄인다.

기관 관점의 실무 결론은 다음과 같다.

```text
confirmation policy = value at risk + attacker model + network conditions + operational tolerance
```

백서는 확률 모델을 제공하지만, 모든 거래 금액, 상대방, 기관 워크플로에 적용되는 보편적 confirmation 임계값을 정해주지는 않는다.

---

## 4. 원래 설계

섹션 11은 앞선 작업증명과 네트워크 섹션을 이어받아, 수취인이 결제를 충분히 뒤집기 어렵다고 간주하기 전에 얼마나 기다려야 하는지를 묻는다.

백서의 가정은 다음과 같다.

1. 정직한 채굴자는 공개 체인을 연장하고 있다.
2. 공격자는 비공개로 대체 체인을 채굴하고 있다.
3. 수취인은 트랜잭션이 블록에 포함되고 그 뒤로 `z`개의 블록이 더 쌓일 때까지 기다린다.
4. 그 동안 공격자도 숨겨진 진척을 만들었을 수 있다.
5. 질문은 그 상태에서 공격자가 여전히 따라잡을 수 있는 확률이다.[^ref-btc-wp]

이것은 모든 Bitcoin 공격의 모델이 아니다. 특정한 이중지불 경쟁에 대한 모델이다.

이 섹션의 목적은 작업증명에서 나온 직관을 수치화하는 것이다.

```text
more confirmations
    -> larger honest-chain lead
    -> more work attacker must overcome
    -> lower success probability if q < p
```

---

## 5. 프로토콜 구조

### 경쟁 구조

이 공격 경쟁은 다음처럼 표현할 수 있다.

```text
honest chain:
    block containing payment + z confirmations

attacker chain:
    private branch excluding or conflicting with payment
```

공격자는 비공개 브랜치가 정직 체인을 따라잡고 추월해, 선택되는 체인이 되면 성공한다.

### Confirmation Count

confirmation 수는 깊이를 측정한다.

```text
1 confirmation:
    transaction is included in a block

6 confirmations:
    transaction's block plus five later blocks, depending on local counting convention
```

운영 인터페이스는 종종 포함된 블록 자체를 첫 confirmation으로 센다. 백서의 `z`는 트랜잭션이 포함된 블록이 수용된 이후 정직 체인이 앞서 있는 블록 수를 뜻한다. 분석가는 자신이 어떤 counting convention을 쓰는지 명확히 밝혀야 한다.

### Chainwork, Not Social Voting

이 확률 모델은 작업증명 기반 체인 선택을 가정한다. 노드는 신원 기준으로 투표하지 않는다. 유효한 블록을 검증하고, 가장 큰 작업량을 가진 유효 체인을 선호한다. Bitcoin Developer 문서도 유효한 경쟁 브랜치가 있을 때 재생성하기 가장 어려운 체인을 따라간다고 설명한다.[^ref-btc-dev-blockchain]

### Full Node와 SPV 맥락

이 계산은 full node와 SPV 사용자 모두에게 중요하지만, 두 모델이 가지는 증거 수준은 다르다. 풀노드는 블록을 직접 검증한다. SPV 클라이언트는 헤더와 머클 포함 증명에 의존한다. Bitcoin Developer 문서는 이 운영 모드를 구분하며, 제네시스부터 검증하는 full node가 가장 강한 모델이라고 말한다.[^ref-btc-dev-operating]

---

## 6. 기술적 메커니즘

### 임의보행 모델

백서는 공격자의 따라잡기 시도를 gambler's ruin 문제에 비유한다. 정직한 채굴자가 다음 블록을 찾을 확률이 `p`, 공격자가 찾을 확률이 `q`라면, 새 블록이 나올 때마다 두 체인 사이의 거리는 바뀐다.

정직한 채굴자가 다음 블록을 찾으면:

```text
attacker falls one block further behind
```

공격자가 다음 블록을 찾으면:

```text
attacker gets one block closer
```

`q < p`일 때 이 과정은 공격자에게 음의 drift를 가진다. 공격자가 더 뒤처질수록 최종 따라잡기 가능성은 작아진다.

### 고정 격차에서의 따라잡기 확률

`z`블록 뒤처진 상태에서:

```text
q_z = 1                  if p <= q
q_z = (q / p)^z          if p > q
```

이 공식이 답하는 질문은 다음과 같다.

```text
Given that the attacker is z blocks behind right now,
what is the probability the attacker ever catches up?
```

### 공격자의 숨겨진 진척

수취인은 정직 네트워크가 `z`개의 블록을 찾는 동안 공격자가 정확히 몇 개의 비공개 블록을 찾았는지 알 수 없다. 그래서 백서는 그 기간 동안의 공격자 진척을 기대값이 다음인 Poisson 확률변수로 모델링한다.

```text
lambda = z * (q / p)
```

최종 확률은 공격자가 얻었을 수 있는 여러 진척 경우를 모두 합산하고, 각 경우에서 남은 격차를 다시 따라잡을 확률로 가중한다.[^ref-btc-wp]

### 왜 Poisson이 등장하는가

Poisson 근사는 정직 체인이 `z`개의 블록을 찾는 데 대략 기대 시간이 걸렸고, 공격자는 독립적으로 더 낮은 기대 속도로 블록을 찾는다는 가정 때문에 쓰인다. 이것은 결정론적 스케줄을 측정하는 것이 아니라, 기다리는 동안 공격자가 얼마나 진척했는지의 분포를 추정하는 것이다.

---

## 7. 수학적 또는 경제적 모델

### 변수 정의

| Symbol | Meaning |
|---|---|
| `p` | honest miners' probability of finding the next block |
| `q` | attacker's probability of finding the next block |
| `z` | honest chain lead in blocks |
| `lambda` | expected number of attacker blocks while honest miners find `z` blocks |

이 모델은 단순화된 2자 경쟁에서 다음을 가정한다.

```text
p + q = 1
```

### 예시: 고정 격차

다음과 같이 가정하자.

```text
q = 0.10
p = 0.90
z = 6
```

그러면 고정 격차 기준 따라잡기 확률은:

```text
(q / p)^z = (0.10 / 0.90)^6
          = (1 / 9)^6
          ~= 0.00000188
```

이 값은 최종 결제 confirmation 확률과 동일하지 않다. 공격자가 정직 체인의 6블록 진행 동안 일부 숨겨진 블록을 찾았을 수 있기 때문이다.

### 예시: Poisson 기대 진척

동일하게:

```text
q = 0.10
p = 0.90
z = 6
```

백서의 Poisson 기대값은:

```text
lambda = z * (q / p)
       = 6 * (0.10 / 0.90)
       ~= 0.6667
```

즉, 정직한 채굴자가 6블록을 찾는 동안 공격자는 평균적으로 약 2/3블록 정도의 비공개 진척을 만들 것으로 기대된다는 뜻이다.

### 백서의 확률 예시

백서는 `q = 0.10`일 때 `z = 5`에서 공격 확률이 `0.0009137`, `z = 6`에서 `0.0002428`이라고 제시한다. `q = 0.30`일 때는 `z = 5`에서 `0.1773523`, `z = 10`에서 `0.0416605`라고 제시한다.[^ref-btc-wp]

Bitcoin Developer의 payment-processing 문서도 `q = 0.30` 케이스에 대해 유사한 표를 재현하며, confirmation은 이중지불 리스크를 없애는 것이 아니라 줄여준다고 설명한다.[^ref-btc-dev-payment]

### Confirmation 정책 해석

이 모델이 시사하는 것은 다음과 같다.

```text
higher q
    -> more confirmations needed for same risk threshold

higher transaction value
    -> lower acceptable attack probability

weaker network visibility
    -> stronger operational controls needed
```

이 공식은 비즈니스 정책을 정해주지 않는다. 가정 아래의 확률 추정만 제공한다.

---

## 8. 보안 가정

### 필요한 가정

섹션 11은 다음 가정에 의존한다.

1. 정직한 채굴자가 공격자보다 더 큰 블록 생산 확률을 가진다.
2. 수취인은 정직한 공개 체인을 보고 있다.
3. 공격자는 무효 블록을 유효하게 만들 수 없다.
4. 블록 발견은 모델이 가정한 확률 과정에 따른다.
5. 공격자의 목표는 자신의 결제를 유효한 경쟁 역사로 되돌리는 것이다.
6. 노드는 가장 큰 작업량의 유효 브랜치를 따른다.

### 모델이 제외하는 것

이 계산은 다음을 포함하지 않는다.

- 수취인에 대한 Eclipse 공격
- Selfish Mining 수익 동학
- fee-sniping 유인
- 네트워크 전파 비대칭
- 해시파워 임대 시장 제약
- 거래소 출금 통제
- 법적 또는 평판 억제 효과
- 채굴자 카르텔 거버넌스
- 실제 전기/하드웨어 법정통화 비용
- 공격 성공 시 BTC 가격 영향

이 요소들은 실제 리스크 관리에 중요할 수 있지만, 백서 섹션 11 계산의 범위 밖이다.

### 과반 공격자 경계

`q >= p`라면 모델은 공격자의 최종 따라잡기 확률을 `1`로 둔다.[^ref-btc-wp]

그렇다고 과반 공격자가 무엇이든 할 수 있다는 뜻은 아니다. 이 말은 오직 이 경쟁 모델에서 결국 따라잡을 수 있다는 뜻이다. 풀노드는 여전히 무효 블록과 무권한 지출을 거부한다.

---

## 9. Bitcoin Core 구현

### Section 11이라는 합의 함수는 없다

Bitcoin Core는 백서 섹션 11의 확률 계산을 합의 규칙으로 구현하지 않는다. 노드는 "6 confirmations"이나 추정된 공격자 확률을 기준으로 블록을 수락/거부하지 않는다.

합의 검증이 다루는 것은 다음이다.

- 블록 유효성
- 작업증명 유효성
- 컨텍스트 기반 난이도 규칙
- 트랜잭션 유효성
- UTXO 지출
- 작업량 기준 체인 선택

confirmation 수는 검증된 체인 상태 위에 얹힌 wallet, application, business-policy 개념이다.

### Full Node 보안 맥락

Bitcoin Developer 문서는 full node가 제네시스부터 블록체인을 검증한다고 설명하며, full node를 속이려면 더 큰 난이도를 가진 완전한 대체 역사를 제시해야 하고, confirmation이 쌓일수록 그 비용이 커진다고 말한다.[^ref-btc-dev-operating]

이 직관은 섹션 11과 맞닿아 있지만, 구현상 임계값과 혼동하면 안 된다.

---

## 10. 온체인 함의

### 관측 가능한 사실

분석가는 다음을 관측할 수 있다.

- 트랜잭션을 포함한 블록
- 그 뒤에 쌓인 블록 수
- 브랜치 대체 또는 reorganization
- 가능한 경우 경쟁 브랜치의 존재
- 유효 헤더로부터 도출한 누적 작업량
- reorg 후 제거된 트랜잭션 여부

### 추론 가능한 것

분석가는 다음을 추론할 수 있다.

- confirmation이 늘수록 줄어드는 이중지불 리스크
- 누적 작업량이 늘수록 높아지는 정산 신뢰
- 깊은 reorg 시점의 높은 리스크
- 충돌 지출이 승리했을 때의 공격 가능성
- 가치 대비 너무 약한 confirmation 정책

### 알 수 없는 것

체인 데이터만으로는 다음을 알기 어렵다.

- 공격자의 실제 해시파워
- 공개되기 전 private fork 존재 여부
- 공격 의도
- 오프체인 정산 시점
- 수취인이 Eclipse되었는지
- 거래소가 명시한 confirmation threshold 전에 크레딧했는지

---

## 11. 기관 관점에서의 해석

### Confirmation Policy

기관은 confirmation을 확률적 리스크 증거로 다뤄야 한다.

```text
confirmation count
    -> depth under current best chain
    -> work needed to reverse
    -> modeled lower probability if q < p
```

정책은 다음을 함께 고려해야 한다.

- 거래 금액
- 상대방 신뢰도
- 자산 인도의 비가역성
- 네트워크 상태
- reorg 모니터링
- 노드 독립성
- 도난 자금의 유동성
- 정산 지연 능력

### 리스크 티어

| Use Case | SPV/Low Confirmation Tolerance | Full Node / Higher Confirmation Need |
|---|---|---|
| Small retail payment | May tolerate lower depth | Optional depending on risk |
| Exchange deposit | Usually insufficient | Strongly preferred |
| Custody movement | Not appropriate | Required |
| Internal accounting | Depends on materiality | Recommended |
| High-value OTC settlement | Not appropriate | Required with monitoring |

### 리서치 규율

섹션 11을 인용할 때 분석가는 반드시 다음을 밝혀야 한다.

```text
attacker hashpower assumption q
confirmation depth z
whether the model is whitepaper probability or operational policy
what risks are outside the model
```

이 가정이 없다면, "6 confirmations are safe" 같은 문장은 기관 리서치에서 지나치게 모호하다.

---

## 12. 흔한 오해

### Misinterpretation 1: Six confirmations are final.

틀렸다. 6 confirmations은 흔한 관행일 뿐이다. 섹션 11은 확률 감소를 모델링하지, 결정론적 finality를 주지 않는다.

### Misinterpretation 2: The table values apply without assumptions.

틀렸다. 값은 공격자 해시파워, 정직 네트워크 가시성, 모델 구조에 크게 의존한다.

### Misinterpretation 3: A majority attacker can create invalid coins.

틀렸다. 섹션 11은 유효한 경쟁 체인을 따라잡는 문제를 다룬다. 무효 블록은 풀노드 검증 아래 여전히 무효다.

### Misinterpretation 4: Confirmation depth measures exact energy spent.

과장이다. confirmation depth는 블록 수 기반 프록시다. 더 정밀한 작업량 비교는 cumulative chainwork를 사용한다.

### Misinterpretation 5: The Poisson model proves attacks cannot happen.

틀렸다. 이 모델은 가정 아래 확률을 추정한다. 낮은 확률은 불가능을 뜻하지 않는다.

### Misinterpretation 6: Zero-confirmation payments have no risk if fees are high.

틀렸다. Bitcoin Developer의 payment-processing 문서는 미확정 트랜잭션은 별도 리스크 분석 없이는 일반적으로 신뢰하면 안 된다고 경고한다.[^ref-btc-dev-payment]

---

## 13. 연구 질문

1. confirmation 임계값은 거래 금액에 따라 어떻게 달라져야 하는가?
2. 기관은 공격자 해시파워 `q`를 어떻게 추정해야 하는가?
3. 실제 Bitcoin 메인넷에서 1블록을 넘는 reorg는 얼마나 자주 발생하는가?
4. 네트워크 이상 상황에서는 confirmation 정책을 어떻게 조정해야 하는가?
5. SPV 기반 confirmation 증거는 full-node 증거 대비 얼마나 할인해야 하는가?
6. cumulative chainwork는 단순 confirmation count보다 어떻게 개선된 척도인가?
7. 섹션 11 모델 밖의 리스크를 줄이려면 어떤 운영 통제가 필요한가?
8. 리스크 보고서에서 private fork 불확실성은 어떻게 표현해야 하는가?
9. 거래소 입금 정책은 충돌하는 mempool 트랜잭션을 어떻게 다뤄야 하는가?
10. 우발적 reorg와 의도적 이중지불을 구분하려면 어떤 데이터가 필요한가?

---

## 14. Practical Exercises

1. 백서 모델에서 `p`, `q`, `z`를 정의하라.
2. `q = 0.20`, `p = 0.80`, `z = 4`일 때 `(q / p)^z`를 계산하라.
3. `q = 0.30`, `p = 0.70`, `z = 10`일 때 `lambda = z * (q / p)`를 계산하라.
4. 왜 고정 격차 확률이 최종 결제 confirmation 확률과 같지 않은지 설명하라.
5. 왜 6 confirmations을 받은 트랜잭션도 원칙적으로는 뒤집힐 수 있는지 설명하라.
6. 고액 거래소 입금에 대한 짧은 confirmation 정책을 작성하고, 그 가정을 적어라.

---

## 15. 증거 분류

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Whitepaper Section 11 probability model and example results | A |
| REF-BTC-DEV-BLOCKCHAIN-001 | Official developer documentation | Chain selection, forks, and most-difficult-chain explanation | A |
| REF-BTC-DEV-PAYMENT-PROCESSING-001 | Official developer documentation | Confirmation risk guidance and reproduced probability examples | A |
| REF-BTC-DEV-OPERATING-001 | Official developer documentation | Full-node security model and confirmation context | A |

### Claim Ledger

| Claim ID | Claim | Classification | Evidence | Source |
|---|---|---|---|---|
| C001 | Section 11 models an attacker trying to catch up from behind after a recipient waits for confirmations. | FACT | A | REF-BTC-WP-001 |
| C002 | The whitepaper defines `p` as honest block probability and `q` as attacker block probability. | FACT | A | REF-BTC-WP-001 |
| C003 | If `q < p`, catch-up probability from `z` blocks behind is `(q / p)^z`. | FACT | A | REF-BTC-WP-001 |
| C004 | If `q >= p`, the model gives eventual catch-up probability as `1`. | FACT | A | REF-BTC-WP-001 |
| C005 | The whitepaper uses `lambda = z * (q / p)` for the attacker's expected hidden progress. | FACT | A | REF-BTC-WP-001 |
| C006 | Confirmation count reduces modeled double-spend risk but does not create deterministic finality. | FACT | A | REF-BTC-WP-001; REF-BTC-DEV-PAYMENT-PROCESSING-001 |
| C007 | Valid competing branches are resolved by following the most difficult chain to recreate. | FACT | A | REF-BTC-DEV-BLOCKCHAIN-001 |
| C008 | Full nodes provide stronger security than SPV by validating the chain from genesis. | FACT | A | REF-BTC-DEV-OPERATING-001 |
| C009 | Institutional confirmation thresholds should vary by value at risk and operating context. | INTERPRETATION | B | REF-BTC-WP-001; REF-BTC-DEV-PAYMENT-PROCESSING-001 |
| C010 | Section 11 does not model every real-world attack or operational control. | INTERPRETATION | B | REF-BTC-WP-001 |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| ESTIMATE | Calculation dependent on stated assumptions |
| HEURISTIC | Practical operating rule requiring caveats |
| UNKNOWN | Evidence is insufficient |

---

## 16. 지식 그래프

```text
BITCOIN-012 Calculations
|
+-- interprets: Whitepaper Section 11
|
+-- attack_model
|   +-- honest_probability: p
|   +-- attacker_probability: q
|   +-- honest_lead: z
|   +-- private_branch: attacker progress
|
+-- formulas
|   +-- fixed_deficit: (q / p)^z when q < p
|   +-- majority_boundary: probability 1 when q >= p
|   +-- poisson_mean: lambda = z * (q / p)
|
+-- settlement
|   +-- confirmations reduce modeled risk
|   +-- no finite confirmation count is absolute finality
|   +-- policy depends on value at risk
|
+-- leads_to: BITCOIN-013 Conclusion
```

---

## 17. 참고문헌

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Section 11, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-blockchain]: Bitcoin Developer Documentation, "Block Chain," forks, proof-of-work, and most-difficult-chain behavior, https://developer.bitcoin.org/devguide/block_chain.html, accessed 2026-08-04.

[^ref-btc-dev-payment]: Bitcoin Developer Documentation, "Payment Processing," confirmation guidance and double-spend risk discussion, https://developer.bitcoin.org/devguide/payment_processing.html, accessed 2026-08-04.

[^ref-btc-dev-operating]: Bitcoin Developer Documentation, "Operating Modes," full-node and SPV security model, https://developer.bitcoin.org/devguide/operating_modes.html, accessed 2026-08-04.

---

## 18. 교차 참조

### Parent

- Bitcoin Whitepaper

### Previous

- BITCOIN-011 — Whitepaper Section 10 — Privacy

### Next

- BITCOIN-013 — Whitepaper Section 12 — Conclusion

### Related

- BITCOIN-005 — Whitepaper Section 4 — Proof of Work
- BITCOIN-009 — Whitepaper Section 8 — Simplified Payment Verification
- BITCOIN-023 — Chain Reorganization
- POW-010 — Chain Selection
- POW-011 — Cumulative Chainwork
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- 섹션 11을 공격자의 따라잡기 모델로 한정했다.
- `p`, `q`, `z`, 고정 격차 확률, Poisson 기대 진척을 분리했다.
- confirmation 수와 결정론적 finality를 분리했다.
- 과반 해시파워의 함의를 invalid-block acceptance로 과도하게 일반화하지 않았다.

### Evidence Review

Passed.

- 백서의 공식과 확률 예시는 Section 11을 직접 인용한다.
- confirmation 및 payment-processing 가이드는 공식 Bitcoin Developer 문서를 인용한다.
- full-node와 SPV 맥락은 공식 operating-mode 문서를 인용한다.
- 기관 정책 관련 문장은 해석으로 표시했다.

### Editorial Review

Passed.

- Markdown 헤딩은 프로젝트 딥다이브 구조를 따른다.
- 메타데이터는 완전하다.
- 표와 코드 펜스는 닫혀 있다.
- confirmations, `p`, `q`, `z`, catch-up probability, Poisson, probabilistic finality 용어를 일관되게 사용했다.

### Adversarial Review

Passed.

- 6 confirmations을 final로 취급하지 않았다.
- 이 모델이 모든 공격을 다룬다고 주장하지 않았다.
- 과반 공격자가 무효 트랜잭션을 만들 수 있다고 암시하지 않았다.
- 확률 모델과 비즈니스 정책을 분리했다.

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
