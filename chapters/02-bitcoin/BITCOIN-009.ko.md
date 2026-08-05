---
knowledge_id: BITCOIN-009
title: 백서 섹션 8 — 간이 결제 검증(Simplified Payment Verification)
subtitle: 헤더, 머클 브랜치, 라이트 클라이언트, Bloom Filter, Compact Block Filter, 그리고 SPV 보안 한계
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 90 min
estimated_study: 240 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Light Clients
  - Verification
  - P2P Network
parent:
  - Bitcoin Whitepaper
prerequisites:
  - BITCOIN-005
  - BITCOIN-006
  - BITCOIN-008
  - POW-010
  - POW-014
related_topics:
  - Simplified Payment Verification
  - SPV
  - Block Headers
  - Merkle Branch
  - Bloom Filters
  - BIP37
  - BIP157
  - BIP158
  - Light Clients
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-OPERATING-001
  - REF-BTC-DEV-P2P-REF-001
  - REF-BIP-0037
  - REF-BIP-0157
  - REF-BIP-0158
  - REF-BTC-CORE-MERKLEBLOCK-001
  - REF-BTC-CORE-BLOCKFILTERINDEX-001
  - REF-BTC-CORE-0190-001
  - REF-BTC-CORE-RPC-GETBLOCKFILTER-001
tags:
  - bitcoin
  - whitepaper
  - spv
  - light-client
  - merkle-branch
  - bip37
  - bip157
  - bip158
---

# 백서 섹션 8 — 간이 결제 검증(Simplified Payment Verification)
> Deep Dive Series  
> Research Unit: BITCOIN-009

---

## Research Brief

```yaml
knowledge_id: BITCOIN-009
title: Whitepaper Section 8 — Simplified Payment Verification
research_question: >
  How can a Bitcoin user verify that a transaction is included in a
  proof-of-work chain without running a full node, and what security,
  privacy, and availability assumptions does this simplified model add?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-005
  - BITCOIN-006
  - BITCOIN-008
parent: Bitcoin Whitepaper
previous: BITCOIN-008
next: BITCOIN-010
related_topics:
  - Block Headers
  - Merkle Proofs
  - Chainwork
  - Confirmation Depth
  - Bloom Filters
  - Compact Block Filters
  - Weak Client Security
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
  - Complete wallet design
  - Mobile-wallet UX
  - Lightning watchtower design
  - Private information retrieval protocols
  - Full compact-filter implementation walkthrough
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- 백서의 모델에서 SPV를 정의할 수 있다.
- 블록 헤더 체인이 무엇을 증명하는지 설명할 수 있다.
- 머클 브랜치가 무엇을 증명하는지 설명할 수 있다.
- 왜 SPV가 전체 트랜잭션 유효성이 아니라 포함 여부를 증명하는지 설명할 수 있다.
- SPV 클라이언트, 풀노드, 서버 신뢰형 클라이언트를 구분할 수 있다.
- 왜 confirmation 깊이가 결정론적 최종성이 아니라 공격 비용의 대리값인지 설명할 수 있다.
- 순진한 SPV가 가지는 누락(omission), Eclipse, Sybil, 프라이버시 리스크를 식별할 수 있다.
- BIP37 Bloom filter 기반 SPV가 어떻게 동작하는지 높은 수준에서 설명할 수 있다.
- 왜 Bitcoin Core가 v0.19.0에서 공개 BIP37 제공을 기본 비활성화했는지 설명할 수 있다.
- BIP157/158 compact block filter가 라이트 클라이언트의 데이터 흐름을 어떻게 바꾸는지 설명할 수 있다.
- 어떤 경우 기관 워크플로가 반드시 풀노드 검증을 요구해야 하는지 판단할 수 있다.

---

## 2. 핵심 질문

1. 백서가 말하는 "풀 네트워크 노드 없이 결제를 검증"한다는 것은 무엇인가?
2. SPV 클라이언트는 어떤 데이터를 보관하는가?
3. 머클 브랜치란 무엇인가?
4. 머클 브랜치는 무엇을 증명하는가?
5. 머클 브랜치가 증명하지 못하는 것은 무엇인가?
6. 왜 SPV 클라이언트도 가장 긴 혹은 가장 큰 작업량의 헤더 체인이 필요한가?
7. 왜 공격자가 네트워크를 압도하면 SPV가 더 취약해지는가?
8. 풀노드 검증과 SPV 검증의 차이는 무엇인가?
9. 풀노드는 omission을 통해 SPV 클라이언트를 어떻게 속일 수 있는가?
10. 트랜잭션 발견 과정은 어떤 프라이버시 문제를 만드는가?
11. BIP37 Bloom filter는 저대역폭 SPV를 어떻게 지원하는가?
12. BIP157/158 compact filter는 BIP37과 어떻게 다른가?

---

## 3. Executive Summary

백서 섹션 8은 Simplified Payment Verification, 즉 SPV를 설명한다. SPV 사용자는 가장 긴 작업증명 체인의 블록 헤더를 보관하고, 특정 트랜잭션을 타임스탬프한 블록에 연결하는 머클 브랜치를 얻는다.[^ref-btc-wp]

SPV가 검증하는 것은 풀노드 검증보다 훨씬 좁은 주장이다.

```text
This transaction is committed by this block header,
and this block header is buried under later proof-of-work headers.
```

SPV는 다음을 검증하지 않는다.

- 블록 안의 모든 트랜잭션
- 모든 스크립트
- 모든 UTXO 지출
- 블록의 코인베이스 금액
- 다른 곳에 충돌 트랜잭션이 없는지
- 피어가 관련 데이터를 모두 제공했는지

백서는 SPV 사용자가 트랜잭션을 스스로 검사할 수 없다고 명시한다. 대신 사용자는 그 트랜잭션을 체인 내 특정 위치에 연결하고, 이후 블록들이 쌓이는 것을 네트워크가 그 트랜잭션을 받아들였다는 증거로 본다.[^ref-btc-wp]

Bitcoin Developer 문서도 같은 구분을 한다. 풀노드는 제네시스부터 블록을 내려받아 검증한다. SPV 클라이언트는 헤더를 내려받고 필요할 때 관련 트랜잭션을 요청한다. 머클 루트와 머클 브랜치가 블록 내 포함 여부를 증명할 수는 있지만, 이것이 트랜잭션 유효성을 보장하지는 않는다고 문서는 설명한다.[^ref-btc-dev-operating]

현대 Bitcoin에는 여러 라이트 클라이언트 탐색 메커니즘이 존재했다.

| Model | Main Mechanism | Main Tradeoff |
|---|---|---|
| Whitepaper SPV | 헤더 + 머클 브랜치 | 전체 검증 없이 포함 증명 |
| BIP37 | 클라이언트가 bloom filter를 보내고, 피어가 `merkleblock`과 매칭 tx 반환 | 저대역폭이지만 프라이버시/DoS 약점 |
| BIP157/158 | 클라이언트가 compact block filter를 내려받고 관련 블록만 fetch | 피어 측 프라이버시 모델 개선, 하지만 여전히 전체 검증은 아님 |

기관 관점에서 SPV만으로는 보통 고액 정산에 충분하지 않다. 제약된 기기나 저가치 모니터링에는 적합할 수 있지만, 고액 커스터디, 거래소 입금, 컴플라이언스 분석은 독립적으로 검증하는 풀노드를 사용해야 한다.

---

## 4. 원문 해석

백서 섹션 8은 사용자가 풀 네트워크 노드를 실행하지 않고도 다음 절차로 결제를 검증할 수 있다고 말한다.

1. 가장 긴 작업증명 체인의 블록 헤더 사본을 보관한다.
2. 해당 트랜잭션을 타임스탬프한 블록에 연결하는 머클 브랜치를 얻는다.
3. 이후 블록들이 그 블록 위에 쌓이는 것을 관찰한다.[^ref-btc-wp]

이 섹션은 동시에 한계를 명시한다. 사용자는 트랜잭션을 스스로 검사할 수 없다. 사용자는 네트워크 노드가 그것을 받아들였다고 보며, 이후 블록들이 네트워크 수용을 추가로 확인해 준다고 보는 것이다.[^ref-btc-wp]

백서는 신뢰 경계도 설명한다.

- 정직한 노드가 네트워크를 지배하는 한 SPV는 신뢰할 수 있다.
- 공격자가 네트워크를 압도하면 더 취약해진다.
- 네트워크 노드는 트랜잭션을 직접 검증할 수 있다.
- 공격자가 네트워크를 압도하는 동안에는 조작된 트랜잭션으로 이 단순화된 방법을 속일 수 있다.
- 빈번히 결제를 받는 사업자는 더 독립적인 보안과 빠른 검증을 위해 자신의 노드를 실행하는 편이 좋을 가능성이 높다.[^ref-btc-wp]

이것은 강한 경고다. 섹션 8은 SPV가 전체 검증과 동등하다고 말하지 않는다.

---

## 5. 문자적 해석

### "Verify payments without running a full network node"

이는 신뢰 가정이 전혀 없다는 뜻이 아니라, 검증을 줄인다는 뜻이다. SPV 클라이언트는 모든 블록과 트랜잭션을 내려받고 검증하는 일을 피한다. 대신 작업증명 헤더와 트랜잭션 포함 여부를 검증할 수 있을 만큼의 정보는 여전히 필요하다.

### "Copy of the block headers"

클라이언트가 헤더를 보관하는 이유는 헤더가 다음을 담고 있기 때문이다.

- 이전 블록 해시
- 머클 루트
- 작업증명 목표값
- nonce
- 타임스탬프와 버전

헤더는 클라이언트가 작업증명 체인의 성장을 추적하게 해준다. 하지만 트랜잭션 본문은 들어 있지 않다.

### "Longest proof-of-work chain"

현대 용어로는 이 표현을 "가장 큰 작업량을 가진 유효한 헤더 체인"으로 읽어야 한다. 난이도가 변할 수 있으므로, 단순한 헤더 개수만으로는 충분하지 않다. 누적 Chainwork는 POW-011에서 다뤘다.

### "Merkle branch linking the transaction to the block"

머클 브랜치는 트랜잭션 해시로부터 머클 루트를 재계산하는 데 필요한 형제 해시들의 집합이다. 재계산한 루트가 헤더의 머클 루트와 같다면, 해당 트랜잭션은 그 블록의 트랜잭션 트리에 커밋되어 있다는 뜻이다.

### "He can't check the transaction for himself"

이 문장이 핵심이다. SPV는 모든 스크립트나 UTXO 지출을 검증하지 않는다. 그것이 검증하는 것은 다시 쓰기 비용이 커 보이는 체인 안에 포함되었다는 사실이다.

---

## 6. 프로토콜 구조

### SPV 데이터 흐름

```text
SPV client
    |
    |-- downloads block headers
    |-- checks header linkage and proof-of-work
    |-- identifies a transaction of interest
    |-- obtains Merkle branch for that transaction
    |-- verifies branch against block header Merkle root
    |-- counts confirmations / accumulated work above that block
```

### 각 객체가 증명하는 것

| Object | Proves | Does Not Prove |
|---|---|---|
| Header chain | 작업증명이 주장된 헤더들의 연속성 | 전체 블록 유효성 |
| Merkle branch | 블록의 커밋된 트랜잭션 트리 내 포함 여부 | 트랜잭션 유효성 |
| Confirmation depth | 그 블록 위에 쌓인 작업량 | 절대적 최종성 |
| Peer response | 그 피어가 보여준 정보 | 완전성 |
| Bloom/compact filter match | 관련 가능성 | 소유권, 유효성, 최종성 |

### 풀노드 vs SPV 클라이언트

| Capability | Full Node | SPV Client |
|---|---:|---:|
| Downloads block headers | Yes | Yes |
| Validates proof of work | Yes | Yes |
| Downloads full blocks | Yes | Usually no |
| Validates every transaction | Yes | No |
| Maintains UTXO set | Yes | Usually no |
| Detects invalid block contents directly | Yes | No |
| Verifies transaction inclusion | Yes | Yes, with proof |
| Depends on peers for transaction discovery | Less | More |

Bitcoin Developer 문서는 풀노드를 가장 안전한 모델로 분류하고, SPV는 헤더를 내려받은 뒤 필요시 풀노드에 관련 트랜잭션을 요청하는 대안으로 설명한다.[^ref-btc-dev-operating]

---

## 7. 기술적 메커니즘

### 머클 브랜치 검증

다음이 주어졌다고 하자.

```text
txid
merkle_branch = [sibling_0, sibling_1, ... sibling_n]
position bits
header_merkle_root
```

검증자는 다음을 수행한다.

1. 트랜잭션 ID에서 시작한다.
2. 각 형제 해시와 올바른 좌/우 순서로 해시한다.
3. 머클 루트를 다시 계산한다.
4. 그 결과를 블록 헤더의 머클 루트와 비교한다.

루트가 일치하면, 그 트랜잭션은 해당 헤더가 커밋한 트랜잭션 트리에 포함되어 있다. 이것이 inclusion proof다.

### Confirmation 깊이

포함 여부가 증명된 뒤, SPV 클라이언트는 그 블록 위에 몇 개의 블록이 더 쌓였는지를 본다. 백서는 이후 블록들이 그것을 더 확인해 준다고 말한다.[^ref-btc-wp]

의미는 다음과 같다.

```text
more blocks above transaction block
    -> more accumulated work above it
    -> higher cost to replace it
```

하지만 이것이 절대적 의미의 최종성을 뜻하는 것은 아니다.

### BIP37 Bloom-filter SPV

BIP37은 피어 서비스 차원에서 connection bloom filtering을 추가한다. 클라이언트는 bloom filter를 로드하여, 피어가 전달하는 트랜잭션과 필터된 블록을 그 필터에 맞게 걸러서 보내도록 할 수 있다.[^ref-bip-0037]

P2P 참고 문서는 `MSG_FILTERED_BLOCK`, `filterload`, `filteradd`, `filterclear`, `merkleblock`을 설명한다. `merkleblock` 응답에는 블록 헤더와, 매칭된 트랜잭션을 헤더의 머클 루트에 연결하는 데 필요한 머클 트리 일부가 들어가며, 매칭된 트랜잭션 자체는 별도로 `tx` 메시지로 전송된다.[^ref-btc-dev-p2p-ref]

Bitcoin Core의 `CMerkleBlock`은 필터된 노드에 블록 헤더와 partial Merkle tree를 릴레이하는 데 사용된다. `CPartialMerkleTree`는 트랜잭션 ID의 부분집합을 표현하며, 매칭된 txid와 머클 루트를 인증된 방식으로 복원할 수 있게 한다.[^ref-btc-core-merkleblock]

### BIP37의 트레이드오프

BIP37은 대역폭을 줄여주지만 대가가 있다.

- false positive는 대역폭과 프라이버시를 교환한다.
- 너무 정밀한 필터는 지갑 관련 관심사를 피어에 노출한다.
- 피어는 관련 데이터를 생략할 수 있다.
- bloom filter 요청을 서비스하는 것은 풀노드에 자원 부담을 줄 수 있다.

Bitcoin Core v0.19.0은 기본 `-peerbloomfilters` 값을 false로 바꾸어, 공개 BIP37 bloom-filter 서비스와 `merkleblock` 응답을 기본적으로 비활성화했다. 릴리스 노트는 DoS 벡터를 이유로 들며, 대안 사용 또는 명시적 활성화를 권장한다.[^ref-btc-core-0190]

### BIP157/158 Compact Block Filters

BIP157은 client-side block filtering을 정의한다. 즉, 지갑별 bloom filter를 피어에게 보내는 대신 라이트 클라이언트가 블록별 compact filter를 받아 로컬에서 어떤 블록이 관심 데이터를 포함할 가능성이 있는지 판단한다.[^ref-bip-0157]

BIP158은 Golomb-Rice coded set을 사용하는 compact filter 구조를 정의하고, 초기 `basic` filter type을 명세한다.[^ref-bip-0158]

이는 프라이버시 방향을 바꾼다.

```text
BIP37: client tells peer a probabilistic wallet filter
BIP157/158: peer serves block filters; client checks locally
```

BIP157도 클라이언트를 풀 검증 노드로 바꾸지는 않는다. 발견 방식과 피어 교차검증 특성은 개선하지만, 클라이언트는 여전히 모든 블록의 모든 트랜잭션을 검증하지 않는다.

Bitcoin Core는 BIP157 네트워크 요청을 제공하기 위해 블록 필터와 해시, 블록 범위별 헤더를 저장하는 `BlockFilterIndex`를 포함한다.[^ref-btc-core-blockfilterindex] Bitcoin Developer RPC 문서에도 `getblockfilter`가 있으며, 이는 블록의 BIP157 content filter와 filter header를 반환한다.[^ref-btc-core-rpc-getblockfilter]

---

## 8. 수학적 또는 경제적 모델

### 대역폭 모델

다음을 정의하자.

```text
H = 80 bytes per header
N = number of blocks
P = size of Merkle proof or filter data
T = size of relevant transaction data
```

SPV 스타일 클라이언트는 대략 다음을 내려받는다.

```text
headers + relevant proofs + relevant transactions
= H * N + P + T
```

전체 아카이벌 동기화는 다음을 내려받는다.

```text
full blocks + undo/index/state data
```

SPV는 헤더에 대해서는 체인 길이에 비례하고, 트랜잭션 데이터에 대해서는 지갑 관련성에 비례한다. 전체 검증은 전체 블록 데이터와 검증 상태에 비례한다.

### 포함 증명 크기

트랜잭션이 `n`개인 블록에서 단일 트랜잭션에 대한 머클 브랜치는 대략 다음만큼의 형제 해시를 필요로 한다.

```text
ceil(log2(n)) sibling hashes
```

각 형제 해시는 32바이트라고 하면, 인코딩 오버헤드를 무시할 때:

```text
proof_size ~= 32 * ceil(log2(n)) bytes
```

예시:

```text
n = 4096 transactions
ceil(log2(4096)) = 12
proof_size ~= 32 * 12 = 384 bytes
```

이는 전체 블록을 내려받는 것보다 훨씬 작지만, 어디까지나 포함 여부만 증명한다.

### 공격 비용 프록시

SPV 보안은 트랜잭션 위에 쌓인 누적 작업량을 대체 비용의 대리값으로 사용한다.

```text
attack_cost_proxy = cumulative work required to replace the branch
```

이 프록시가 의미 있으려면 다음이 성립해야 한다.

- 클라이언트가 진짜 가장 큰 작업량 체인을 보고 있어야 한다.
- 공격자가 클라이언트의 피어 시야를 장악하지 않아야 한다.
- 관련 시간 구간에서 정직한 채굴자가 우세해야 한다.
- 포함된 트랜잭션이 풀노드 규칙 아래 유효해야 한다.

---

## 9. 보안 가정

### 필요한 가정

SPV는 다음에 의존한다.

1. 클라이언트가 가장 큰 작업량의 헤더 체인을 얻는다.
2. 머클 브랜치가 해당 트랜잭션과 헤더에 대해 올바르다.
3. 적어도 일부 연결된 피어는 정직하거나, 불일치를 감지할 수 있다.
4. 정직한 채굴자가 충분한 작업량을 지배하여 조작된 체인을 만드는 비용이 높다.
5. 비록 SPV 클라이언트는 모든 규칙을 검사하지 않더라도, 해당 트랜잭션이 풀노드 규칙 아래 유효했다.

### 약점 1: 무효 트랜잭션 포함

백서는 SPV가 트랜잭션을 스스로 검사할 수 없다고 말한다.[^ref-btc-wp] 공격자가 네트워크를 압도하거나 클라이언트를 고립시킬 수 있다면, 클라이언트는 풀노드라면 거부했을 조작 체인과 트랜잭션을 보여받을 수 있다.

### 약점 2: 누락

SPV 포함 증명은 비대칭적이다. 증명은 어떤 트랜잭션이 포함되었음을 보여줄 수는 있지만, 피어는 관련 트랜잭션을 생략하거나 "없다"고 말할 수도 있다. Bitcoin Developer 문서는 omission을 SPV의 잠재적 약점으로 명시한다.[^ref-btc-dev-operating]

### 약점 3: Eclipse 및 Sybil 리스크

라이트 클라이언트가 적대적 피어에만 연결되면, 헤더, 필터, 트랜잭션 알림에 대해 왜곡된 시야를 받을 수 있다. 그래서 피어 다양성이 중요하다.

### 약점 4: 프라이버시 유출

BIP37 Bloom filter는 필터 자체와 반복 질의를 통해 정보를 유출한다. false positive 비율을 높이면 모호성은 커지지만 대역폭이 늘고, false positive 비율을 낮추면 대역폭은 절약되지만 지갑 관심사가 더 정확히 노출된다.[^ref-bip-0037][^ref-btc-dev-operating]

### 약점 5: SPV처럼 보이지만 사실은 서버 신뢰

일부 지갑이나 애플리케이션은 헤더와 머클 증명을 직접 검증하지 않고 서버 API에 의존한다. 이는 백서의 SPV 모델이 아니다. 완전히 다른 신뢰 모델이다.

---

## 10. Bitcoin Core 구현

### Bitcoin Core는 기본적으로 풀노드다

Bitcoin Core의 기본 운영 모델은 전체 검증이다. 제네시스부터 블록을 내려받아 검증하고 chainstate를 유지한다. 따라서 섹션 8은 "Bitcoin Core의 일반 지갑 모드"가 아니라, 다른 소프트웨어가 구현할 수 있는 더 가벼운 클라이언트 모델이다.

### BIP37 지원과 `merkleblock`

Bitcoin Core는 역사적으로 BIP37을 구현했지만, 공개 bloom-filter 서비스는 v0.19.0 이후 `-peerbloomfilters=false`로 기본 비활성화되었다.[^ref-btc-core-0190]

관련 구현 구성요소는 다음과 같다.

| Component | Role |
|---|---|
| `CMerkleBlock` | 필터된 노드용 헤더 + partial Merkle tree |
| `CPartialMerkleTree` | 매칭된 txid와 머클 루트를 인코딩/추출 |
| `filterload` / `filteradd` / `filterclear` | BIP37 bloom-filter P2P 메시지 |
| `MSG_FILTERED_BLOCK` | 필터된 블록 요청 / `merkleblock` 응답 |

Bitcoin Core의 `merkleblock.cpp`는 전체 블록과 bloom filter 또는 txid 집합으로부터 `CMerkleBlock`을 만들고, 전체 txid 목록과 매칭 마스크로 `CPartialMerkleTree`를 구성한다.[^ref-btc-core-merkleblock]

### BIP157/158 지원

Bitcoin Core는 block filter index를 통해 compact block filter를 지원한다. `BlockFilterIndex`는 블록 필터, 해시, 헤더를 저장/조회하며 BIP157 네트워크 요청 서비스에 사용된다.[^ref-btc-core-blockfilterindex]

Bitcoin Core v0.19.0 릴리스 노트는 `-blockfilterindex`를 도입했으며, 로컬 사용자가 `getblockfilter` RPC를 통해 BIP158 필터를 얻을 수 있다고 설명한다. 다만 그 버전에서는 P2P를 통한 filter 서비스는 아직 제공하지 않았다.[^ref-btc-core-0190]

Bitcoin Developer RPC 문서는 `getblockfilter`가 블록의 BIP157 content filter를 가져오고, hex 인코딩된 필터와 filter header를 반환한다고 설명한다.[^ref-btc-core-rpc-getblockfilter]

---

## 11. 온체인 함의

### SPV가 보여줄 수 있는 것

SPV 클라이언트는 다음을 보여줄 수 있다.

- 트랜잭션 ID
- 블록 헤더
- 트랜잭션을 헤더에 연결하는 머클 증명
- 헤더 체인 내에서 그 블록의 겉보기 깊이
- 그 깊이 뒤에 있는 대략적인 작업증명 커밋먼트

### SPV만으로는 보여줄 수 없는 것

SPV만으로는 다음을 보여줄 수 없다.

- 완전한 UTXO 유효성
- 모든 입력 스크립트 통과 여부
- 블록 안에 inflation bug가 없는지
- 블록 다른 곳에 무효 트랜잭션이 없는지
- 관련 트랜잭션이 생략되지 않았는지
- 실제 세계의 finality
- 채굴자나 상대방의 의도

### 분석가 활용

SPV 증거는 다음에 유용하다.

- 저자원 환경의 트랜잭션 확인 체크
- 임베디드 기기
- 모바일 지갑 UX
- 특정 txid가 특정 헤더에 커밋되었음을 입증
- 서버 응답 교차검증

SPV 증거만으로 충분하지 않은 경우는 다음과 같다.

- 고액 거래소 입금 finality
- 규제 또는 포렌식 회계
- 완전한 수수료 분석
- UTXO 세트 재구성
- 모든 합의 규칙 아래 트랜잭션 유효성 증명
- 무효 블록의 독립적 탐지

---

## 12. 기관 관점에서의 해석

### SPV가 허용될 수 있는 경우

SPV 스타일 검증은 다음과 같은 경우 허용될 수 있다.

- 거래 금액이 낮고,
- 기기가 제약되어 있으며,
- 풀노드 연결이 불가능하고,
- 사용자가 최종 정산이 아니라 결제 표시 여부를 확인하려 하고,
- 여러 독립 피어에 질의하며,
- 애플리케이션이 omission과 지연 리스크를 감수할 수 있을 때

### 풀노드가 필요한 경우

기관은 다음에 대해 풀노드 검증을 요구해야 한다.

- 커스터디 입금 크레딧
- 거래소 정산
- 자금 이동 확인
- 감사 수준의 트랜잭션 기록
- 의심 거래 조사
- 고액 상거래 정산
- 보안 사고 대응

백서 자체도 빈번히 결제를 받는 사업자는 더 독립적인 보안과 더 빠른 검증을 위해 자체 노드를 운영하고 싶어할 것이라고 말한다.[^ref-btc-wp]

### 운영 통제

라이트 클라이언트 워크플로에서는 다음이 필요하다.

- 헤더와 작업증명을 로컬에서 검증하고,
- 여러 피어에 연결하며,
- 피어 간 불일치를 감지하고,
- 가능하면 지갑 관심사를 정확히 노출하는 방식보다 compact filter 설계를 선호하고,
- 데이터 미제공을 부재의 증거가 아니라 미결정 상태로 취급하며,
- 고액 이벤트는 풀노드 검증으로 승격시키고,
- 리서치 방법론에 신뢰 모델을 명시해야 한다.

---

## 13. 흔한 오해

### Misinterpretation 1: SPV is the same security as a full node.

틀렸다. SPV는 작업증명 헤더 체인에 포함되었다는 사실을 검증한다. 모든 트랜잭션과 블록 규칙을 검증하지는 않는다.

### Misinterpretation 2: A Merkle proof proves a transaction is valid.

틀렸다. 머클 증명은 커밋된 트랜잭션 트리에 포함되었다는 사실을 증명한다. 유효성은 전체 검증이 필요하다.

### Misinterpretation 3: More confirmations remove all risk.

틀렸다. 더 많은 confirmation은 모델 가정 아래 대체 비용을 올릴 뿐이다. 결정론적 최종성을 만드는 것은 아니다.

### Misinterpretation 4: BIP37 bloom filters are private.

과장이다. Bloom filter는 false positive를 추가할 수 있지만, 여전히 지갑 관심사 정보를 유출하며, Bitcoin Core에서 공개 서비스가 기본 비활성화된 것도 자원 및 프라이버시 문제 때문이다.[^ref-bip-0037][^ref-btc-core-0190]

### Misinterpretation 5: BIP157/158 makes light clients full validators.

틀렸다. Compact filter는 트랜잭션 발견과 피어 교차검증을 개선할 뿐, 클라이언트를 전체 트랜잭션 검증 노드로 바꾸지 않는다.

### Misinterpretation 6: A wallet using a server API is automatically SPV.

틀렸다. 백서 SPV는 로컬 헤더 검증과 머클 브랜치 검증을 요구한다. 서버 신뢰형 지갑은 다른 신뢰 모델이다.

---

## 14. 연구 질문

1. 모바일 지갑 중 헤더와 머클 증명을 로컬 검증하는 비율과 서버 신뢰형 비율은 각각 얼마인가?
2. 여전히 BIP37 bloom-filter 피어를 제공하는 공개 노드는 얼마나 되는가?
3. compact filter 기반 지갑 워크플로에는 어떤 프라이버시 유출이 여전히 남아 있는가?
4. 기관은 조사 과정에서 라이트 클라이언트 증거를 어떻게 문서화해야 하는가?
5. Eclipse 리스크 아래에서 라이트 클라이언트에 충분한 피어 다양성 전략은 무엇인가?
6. 지갑 소프트웨어는 omission이나 불일치하는 filter 응답을 어떻게 처리해야 하는가?
7. SPV 전용 검증에 적절한 confirmation 임계값은 무엇인가?
8. 일반적인 지갑 사용 패턴에서 compact filter는 BIP37 대비 대역폭을 얼마나 바꾸는가?
9. compact filter를 쓰더라도 어떤 지갑 워크플로는 전체 블록이 필요한가?
10. 감사자는 SPV 증거와 풀노드 검증 증거를 어떻게 구분해야 하는가?

---

## 15. Practical Exercises

1. 트랜잭션 ID, 머클 브랜치, 블록 헤더 머클 루트가 주어졌을 때 검증 절차를 설명하라.
2. 머클 증명이 왜 입력이 미사용 상태였다는 사실까지는 증명하지 못하는지 설명하라.
3. 지갑 관심사 데이터 관점에서 BIP37과 BIP157/158을 비교하라.
4. SPV 클라이언트가 한 개 피어에만 연결하는 것이 왜 약한지 설명하라.
5. 다음 주장을 풀노드 증거인지 SPV 증거인지 분류하라.
   - "The transaction is included in block X."
   - "All scripts in block X are valid."
   - "Block X has six descendants."
   - "No transaction paying this address exists."
6. SPV가 허용되는 경우와 풀노드 검증이 필요한 경우를 구분하는 짧은 정산 정책을 작성하라.

---

## 16. 증거 분류

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Whitepaper Section 8 SPV model and caveats | A |
| REF-BTC-DEV-OPERATING-001 | Official developer documentation | Full-node vs SPV operating modes and weaknesses | A |
| REF-BTC-DEV-P2P-REF-001 | Official developer documentation | `merkleblock`, `filterload`, `filteradd`, `filterclear`, `MSG_FILTERED_BLOCK` | A |
| REF-BIP-0037 | BIP | Bloom-filter peer-service specification for SPV clients | A |
| REF-BIP-0157 | BIP | Client-side block filtering protocol | A |
| REF-BIP-0158 | BIP | Compact block filter structure | A |
| REF-BTC-CORE-MERKLEBLOCK-001 | Primary implementation source | `CMerkleBlock` and `CPartialMerkleTree` | A |
| REF-BTC-CORE-BLOCKFILTERINDEX-001 | Primary implementation source | `BlockFilterIndex` for BIP157 filters | A |
| REF-BTC-CORE-0190-001 | Release documentation | BIP37 serving disabled by default; `-blockfilterindex` introduced | A |
| REF-BTC-CORE-RPC-GETBLOCKFILTER-001 | Official RPC documentation | `getblockfilter` RPC result fields | B |

### Claim Ledger

| Claim ID | Claim | Classification | Evidence | Source |
|---|---|---|---|---|
| C001 | Whitepaper SPV requires block headers and a Merkle branch linking the transaction to a block. | FACT | A | REF-BTC-WP-001 |
| C002 | The whitepaper states SPV users cannot check the transaction for themselves. | FACT | A | REF-BTC-WP-001 |
| C003 | SPV proves inclusion but does not guarantee transaction validity. | FACT | A | REF-BTC-WP-001; REF-BTC-DEV-OPERATING-001 |
| C004 | Full nodes provide the strongest security model by downloading and validating blocks. | FACT | A | REF-BTC-DEV-OPERATING-001 |
| C005 | BIP37 defines bloom-filter support for low-bandwidth SPV clients. | FACT | A | REF-BIP-0037 |
| C006 | A `merkleblock` message contains a block header and Merkle tree data needed to connect matches to the Merkle root. | FACT | A | REF-BTC-DEV-P2P-REF-001 |
| C007 | Bitcoin Core uses `CMerkleBlock` and `CPartialMerkleTree` for filtered-node Merkle block behavior. | FACT | A | REF-BTC-CORE-MERKLEBLOCK-001 |
| C008 | Bitcoin Core disabled public BIP37 serving by default in v0.19.0. | FACT | A | REF-BTC-CORE-0190-001 |
| C009 | BIP157/158 compact filters shift transaction discovery toward client-side filtering. | FACT | A | REF-BIP-0157; REF-BIP-0158 |
| C010 | Institutional high-value settlement should use full-node validation rather than SPV alone. | INTERPRETATION | B | REF-BTC-WP-001; REF-BTC-DEV-OPERATING-001 |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| HEURISTIC | Practical operating rule requiring context |
| OPEN | Unresolved or implementation-dependent question |

---

## 17. 지식 그래프

```text
BITCOIN-009 Simplified Payment Verification
|
+-- interprets: Whitepaper Section 8
|
+-- requires: block headers
|   +-- prove: proof-of-work chain
|   +-- do_not_prove: full block validity
|
+-- requires: Merkle branch
|   +-- proves: transaction inclusion
|   +-- do_not_prove: transaction validity
|
+-- light_client_models
|   +-- BIP37: bloom filter + merkleblock
|   +-- BIP157/158: compact block filters
|
+-- risks
|   +-- omission
|   +-- eclipse/Sybil
|   +-- privacy leakage
|   +-- invalid fabricated chain under overpowering attacker
|
+-- institution_policy
    +-- low value: possible SPV use
    +-- high value: full-node validation
```

---

## 18. 참고문헌

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Section 8, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-operating]: Bitcoin Developer Documentation, "Operating Modes — Full Node and Simplified Payment Verification," https://developer.bitcoin.org/devguide/operating_modes.html, accessed 2026-08-04.

[^ref-btc-dev-p2p-ref]: Bitcoin Developer Documentation, "P2P Network," `merkleblock`, bloom-filter messages, and `MSG_FILTERED_BLOCK`, https://developer.bitcoin.org/reference/p2p_networking.html, accessed 2026-08-04.

[^ref-bip-0037]: Mike Hearn and Matt Corallo, "BIP37: Connection Bloom filtering," Bitcoin Improvement Proposals, status Deployed, assigned 2012-10-24, https://bips.dev/37/, accessed 2026-08-04.

[^ref-bip-0157]: Olaoluwa Osuntokun, Alex Akselrod, and Jim Posen, "BIP157: Client Side Block Filtering," Bitcoin Improvement Proposals, status Deployed, assigned 2017-05-24, https://bips.dev/157/, accessed 2026-08-04.

[^ref-bip-0158]: Olaoluwa Osuntokun and Alex Akselrod, "BIP158: Compact Block Filters for Light Clients," Bitcoin Improvement Proposals, status Deployed, assigned 2017-05-24, https://bips.dev/158/, accessed 2026-08-04.

[^ref-btc-core-merkleblock]: Bitcoin Core Contributors, `src/merkleblock.cpp` and `src/merkleblock.h`, `CMerkleBlock` and `CPartialMerkleTree`, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/merkleblock_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-blockfilterindex]: Bitcoin Core Contributors, `src/index/blockfilterindex.cpp`, `BlockFilterIndex`, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/blockfilterindex_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-0190]: Bitcoin Core Contributors, "Bitcoin Core 0.19.0.1 Release Notes," BIP37 default change and `-blockfilterindex`, https://bitcoincore.org/en/releases/0.19.0.1/, accessed 2026-08-04.

[^ref-btc-core-rpc-getblockfilter]: Bitcoin Developer Documentation, "getblockfilter RPC," https://developer.bitcoin.org/reference/rpc/getblockfilter.html, accessed 2026-08-04.

---

## 19. 교차 참조

### Parent

- Bitcoin Whitepaper

### Previous

- BITCOIN-008 — Whitepaper Section 7 — Reclaiming Disk Space

### Next

- BITCOIN-010 — Whitepaper Section 9 — Combining and Splitting Value

### Related

- BITCOIN-005 — Whitepaper Section 4 — Proof of Work
- BITCOIN-006 — Whitepaper Section 5 — Network
- BITCOIN-011 — Whitepaper Section 10 — Privacy
- BITCOIN-012 — Whitepaper Section 11 — Calculations
- BITCOIN-018 — Blocks & Block Headers
- BITCOIN-021 — Nodes & Network Propagation
- POW-010 — Chain Selection
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- SPV 포함 증명과 전체 트랜잭션/블록 검증을 분리했다.
- 헤더 체인 검증, 머클 브랜치 검증, confirmation 깊이 해석을 구분했다.
- BIP37, BIP157, BIP158을 합의 변경이 아닌 라이트 클라이언트/피어 서비스 메커니즘으로 한정했다.
- Bitcoin Core `CMerkleBlock`, `CPartialMerkleTree`, `BlockFilterIndex`, v0.19.0 bloom-filter 기본 동작을 1차 자료와 대조했다.

### Evidence Review

Passed.

- 백서 섹션 8 관련 주장은 백서를 직접 인용한다.
- 풀노드 vs SPV 보안 관련 주장은 공식 Bitcoin Developer 문서를 인용한다.
- BIP37 및 compact filter 관련 주장은 BIP 문서를 인용한다.
- Bitcoin Core 구현 관련 주장은 Doxygen 소스 또는 Bitcoin Core 릴리스 문서를 인용한다.

### Editorial Review

Passed.

- Markdown 헤딩은 프로젝트 딥다이브 구조를 따른다.
- 메타데이터는 완전하다.
- 표와 코드 펜스는 닫혀 있다.
- SPV, light client, Merkle branch, header chain, bloom filter, compact filter 용어를 일관되게 사용했다.

### Adversarial Review

Passed.

- SPV를 풀노드 검증과 동등시하지 않았다.
- 머클 증명이 트랜잭션 유효성을 증명한다고 주장하지 않았다.
- omission, 프라이버시, Sybil, Eclipse 스타일 리스크를 포함했다.
- 백서 SPV와 서버 신뢰형 지갑 모델을 구분했다.

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
