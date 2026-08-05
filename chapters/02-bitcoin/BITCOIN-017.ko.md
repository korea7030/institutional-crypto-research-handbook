---
knowledge_id: BITCOIN-017
title: Mempool
subtitle: 로컬 트랜잭션 풀, relay policy, 수수료 시장, RBF, package acceptance, 그리고 cluster mempool
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 105 min
estimated_study: 300 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Mempool
  - Transaction Relay
  - Policy
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-014
  - BITCOIN-015
  - BITCOIN-016
related_topics:
  - Transaction Validation
  - Relay Policy
  - Replace-by-Fee
  - Child Pays For Parent
  - Package Relay
  - Cluster Mempool
  - Fee Estimation
  - Block Template Construction
primary_sources:
  - REF-BIP-0125
  - REF-BTC-CORE-26-RELEASE-001
  - REF-BTC-CORE-31-RELEASE-001
  - REF-BTC-CORE-TXMEMPOOL-001
  - REF-BTC-CORE-MEMPOOL-ENTRY-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-RBF-001
  - REF-BTC-CORE-POLICY-001
  - REF-BTC-CORE-RPC-MEMPOOL-001
  - REF-BTC-CORE-RPC-CONSISTENCY-001
tags:
  - bitcoin
  - internals
  - mempool
  - relay-policy
  - rbf
  - cpfp
  - package-relay
  - cluster-mempool
  - fee-market
---

# Mempool
> Bitcoin Internals  
> Research Unit: BITCOIN-017

---

## Research Brief

```yaml
knowledge_id: BITCOIN-017
title: Mempool
research_question: >
  What is Bitcoin's mempool, how do nodes accept, store, replace, evict,
  relay, and expose unconfirmed transactions, and how should institutional
  analysts distinguish local mempool observation from consensus settlement?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-014
  - BITCOIN-015
  - BITCOIN-016
parent: Bitcoin Internals
previous: BITCOIN-016
next: BITCOIN-018
related_topics:
  - UTXO Model
  - Transactions
  - Script
  - Transaction Fees
  - Mining
  - Reorganizations
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
  - Full P2P transaction relay protocol
  - Complete package relay specification
  - Miner-private transaction submission markets
  - Wallet fee-estimation implementation details
  - Non-Bitcoin mempool designs
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- mempool을 노드 로컬의 미확정 transaction 집합으로 정의할 수 있다.
- 단일한 글로벌 Bitcoin mempool이 없다는 점을 설명할 수 있다.
- consensus validity와 mempool acceptance policy를 구분할 수 있다.
- local relay fee, mempool minimum fee, eviction이 가시성에 미치는 영향을 설명할 수 있다.
- parent, child, conflict, replacement, cluster, chunk 용어를 설명할 수 있다.
- RBF를 미확정 transaction 교체 정책으로 설명할 수 있다.
- CPFP를 package 수준 수수료 유인으로 설명할 수 있다.
- 현재 Bitcoin Core mempool 관련 소스 영역을 식별할 수 있다.
- mempool RPC 필드를 전역적 진실처럼 취급하지 않고 해석할 수 있다.
- mempool 데이터가 유용하지만 취약한 분석 신호라는 점을 설명할 수 있다.

---

## 2. 핵심 질문

1. mempool은 정확히 어떤 상태를 나타내는가?
2. 왜 모든 노드가 동일한 mempool을 공유하지 않는가?
3. 어떤 transaction이 로컬 mempool에 들어오는가?
4. consensus-valid와 relayable은 왜 다른가?
5. RBF와 CPFP는 미확정 상태에서 유인을 어떻게 바꾸는가?
6. cluster mempool은 예전 ancestor/descendant 모델과 무엇이 다른가?
7. mempool 관측으로 무엇을 알 수 있고 무엇은 알 수 없는가?
8. 기관 분석 시스템은 미확정 거래를 어떻게 다뤄야 하는가?

---

## 3. Executive Summary

mempool은 각 노드가 현재 최선의 체인 상태를 기준으로 받아들인 미확정 transaction들의 로컬 집합이다. 이는 consensus state가 아니라 pre-chain network state다. 따라서 "mempool에 있다"는 말은 전역 네트워크 합의가 아니라 특정 시점 특정 노드의 관측을 뜻한다.[^ref-btc-core-txmempool]

Bitcoin Core의 mempool 수용은 대략 다음 요소에 의해 좌우된다.

- consensus validity
- standardness / relay policy
- fee-related policy
- conflict and replacement rules
- local size limits and eviction
- current best chain context

BIP125는 opt-in RBF 정책을 정의하고, Bitcoin Core는 이를 기반으로 미확정 transaction 교체를 평가한다.[^ref-bip-0125] CPFP는 부모와 자식의 결합된 경제성을 통해 확인 유인을 바꾸는 메커니즘이며, 이는 consensus rule이 아니라 패키지 관점의 policy/market 현상이다.

Bitcoin Core 31.0의 cluster mempool은 트랜잭션 집합을 cluster와 chunk 관점에서 관리한다. 기본 cluster 제한은 64개 transaction과 101 kB virtual size이며, 이 구조는 블록 구성, eviction, relay announcement, replacement evaluation과 연결된다.[^ref-btc-core-31-release]

분석상 중요한 결론은 단순하다. mempool 데이터는 유익하지만 결제 확정과 동일하지 않다. 기관 시스템은 pending 상태를 settled 상태와 명확히 분리해야 한다.

---

## 4. 프로토콜 구조

### mempool의 의미

mempool은 블록에 아직 포함되지 않았지만 노드가 받아들인 transaction 집합이다. transaction은 블록에 포함되어 active chain에 남아야 비로소 on-chain state가 된다.

### 글로벌 상태가 아닌 이유

노드별 mempool 차이는 자연스럽다. 이유는 다음과 같다.

- 노드가 transaction을 받은 시점이 다르다.
- relay policy 설정이 다르다.
- 수수료 최소치가 다르다.
- eviction 압력이 다르다.
- chain tip이 잠시 다를 수 있다.
- private relay 또는 direct-to-miner submission이 존재할 수 있다.

### parent, child, conflict

| Term | Meaning |
|---|---|
| Parent | 다른 미확정 transaction이 그 출력을 소비하는 선행 transaction |
| Child | 미확정 부모 출력을 소비하는 후행 transaction |
| Conflict | 동일한 outpoint를 경쟁적으로 소비하는 transaction |
| Replacement | 기존 mempool entry를 다른 transaction이 대체하는 경우 |
| Cluster | 의존 관계로 연결된 transaction 집합 |
| Chunk | cluster 내부에서 정렬과 평가에 사용되는 단위 |

---

## 5. Technical Mechanics

### acceptance 경로

Bitcoin Core는 validation 계층에서 transaction을 검사한 뒤 `AcceptToMemoryPool` 계열 로직을 통해 mempool 수용 여부를 결정한다.[^ref-btc-core-validation]

여기에는 다음이 포함된다.

- 기본 transaction validity 검사
- UTXO context 검사
- script 검증
- standardness 검사
- fee / replacement / package 관련 policy 검사

### `CTxMemPool`과 인덱싱

`CTxMemPool`은 수용된 미확정 transaction과 관련 메타데이터를 유지한다. 문서화된 구현 설명에 따르면 txid, wtxid, entry time 등으로 조회할 수 있는 구조를 가진다.[^ref-btc-core-txmempool]

`mapNextTx`는 특정 outpoint를 현재 mempool 안에서 누가 지출하고 있는지 추적한다. 이 구조는 double-spend conflict 탐지와 parent-child 관계 파악에 중요하다.

### `CTxMemPoolEntry`

entry는 단순히 raw transaction만 저장하지 않는다. fee, vsize, entry height, lock points, ancestor/descendant 또는 cluster 관련 판단에 필요한 보조 정보가 함께 유지된다.[^ref-btc-core-mempool-entry]

### RBF

BIP125는 opt-in full RBF signaling과 replacement 조건을 설명한다.[^ref-bip-0125] Bitcoin Core는 `IsRBFOptIn`과 관련 replacement fee check로 이를 구현한다.[^ref-btc-core-rbf] 핵심은 다음과 같다.

- 기존 transaction이 replaceable로 간주되는가
- 새 transaction이 충분한 추가 수수료를 제공하는가
- 충돌 transaction과 그 관계가 정책 조건을 만족하는가

RBF는 사기와 동일어가 아니다. 더 빠른 확인을 위한 정상적 fee bumping에도 사용된다.

### CPFP와 package acceptance

CPFP는 낮은 수수료의 부모와 높은 수수료의 자식을 결합해 확인 유인을 높인다. Bitcoin Core 26.0은 `submitpackage` 도입으로 package 관점 수용을 노출했고, 31.0은 cluster mempool 문맥에서 관련 동작을 더 현재적인 방식으로 설명한다.[^ref-btc-core-26-release] [^ref-btc-core-31-release]

### eviction과 expiry

mempool은 무한하지 않다. 로컬 크기 제한과 동적 mempool minimum fee 상승에 따라 낮은 경제성을 가진 entry가 축출될 수 있다. 따라서 어떤 transaction이 한 시점에 보이지 않는다고 해서 존재하지 않는다고 결론 내릴 수 없다.[^ref-btc-core-policy]

---

## 6. Mathematical or Economic Model

### 로컬 시장으로서의 mempool

mempool은 블록 공간 수요의 로컬 스냅샷처럼 동작한다. 단순화하면 노드는 다음과 같은 판단 문제를 풀고 있다.

```text
accept if valid and policy-satisfying and economically worth keeping
```

### package 유인

CPFP 상황에서는 개별 transaction fee rate보다 결합된 package fee rate가 중요할 수 있다.

```text
package_fee = sum(transaction_fees)
package_vsize = sum(transaction_vsizes)
package_feerate = package_fee / package_vsize
```

### cluster 관점

cluster mempool에서는 독립적인 개별 tx보다 의존 관계를 가진 transaction 집합이 더 중요해진다. 따라서 "이 transaction의 fee rate"만 보고 채굴 우선순위를 단정하는 것은 점점 덜 정확해진다.

---

## 7. Security Assumptions

### local observation risk

단일 노드 mempool 관측은 본질적으로 부분 정보다. propagation 지연, 네트워크 분리, 정책 차이, private relay 때문에 보이는 것과 실제 네트워크 전체 상태가 다를 수 있다.

### settlement risk

미확정 transaction은 되돌릴 수 있고 교체될 수 있으며 영원히 확인되지 않을 수도 있다. 따라서 high-risk value transfer에서 mempool 관측만으로 자산을 확정 처리하면 위험하다.

### software-version dependence

정책과 구현은 버전에 따라 변한다. 예를 들어 31.0 cluster mempool 동작을 오래된 ancestor/descendant 제한 설명과 혼용하면 분석 오류가 생긴다.[^ref-btc-core-31-release]

---

## 8. Bitcoin Core 구현

### 핵심 소스 영역

| Area | Role |
|---|---|
| `txmempool.*` | `CTxMemPool`, 인덱스, dependency 관리 |
| `kernel/mempool_entry.*` | mempool entry metadata |
| `validation.cpp` | mempool acceptance path |
| `policy/rbf.cpp` | RBF replacement logic |
| `policy/policy.*` | standardness와 relay checks |

### `CCoinsViewMemPool`

Core는 base chainstate 위에 mempool 상태를 overlay해서 transaction 검사를 수행할 수 있다. 이는 미확정 부모를 참조하는 자식 transaction 처리에 중요하다.[^ref-btc-core-txmempool]

### RPC 관측

`getrawmempool true`는 개별 entry의 `vsize`, `weight`, `fees`, `depends`, `spentby`, `wtxid`, `bip125-replaceable`, `unbroadcast` 등을 노출한다.[^ref-btc-core-rpc-mempool] 이 정보는 강력하지만, 특정 호출 시점의 로컬 스냅샷이라는 점을 항상 명시해야 하며, RPC 일관성 설명도 mempool 상태를 전역 진실로 해석하면 안 된다고 경고한다.[^ref-btc-core-rpc-consistency]

---

## 9. Consensus, Policy, and Mining

### Consensus

consensus는 transaction이 유효한 블록에 포함될 수 있는지를 결정한다. mempool 존재 여부는 consensus 요건이 아니다.

### Policy

policy는 다음 같은 질문을 다룬다.

- 기본 노드가 relay할 것인가
- 로컬 mempool에 보관할 것인가
- replacement를 허용할 것인가
- package로 수용할 것인가

### Mining Policy

채굴 정책은 후보 블록에 어떤 transaction을 포함할지 결정한다. 일반적으로 relay policy와 비슷하지만 반드시 같지는 않다. miner는 다음과 같이 다르게 행동할 수 있다.

- private transaction submission 수용
- pool-specific minimum fee 적용
- out-of-band fee agreement 반영
- block-template customizations 적용
- nonstandard transaction 포함

mempool acceptance는 confirmation guarantee가 아니다.

---

## 10. On-Chain and Pre-Chain Implications

### Strong Evidence

mempool 데이터는 특정 시점 특정 노드에 대해 다음을 강하게 뒷받침한다.

- 해당 노드가 특정 `txid` 또는 `wtxid`를 mempool에 보유했다.
- RPC가 특정 fee, vsize, weight, dependency set을 보고했다.
- 그 노드 관점에서 BIP125 replaceable 여부가 판정되었다.
- 미확정 parent/child 관계가 존재했다.
- 로컬 mempool sequence가 변했다.

### Weak Evidence

mempool 데이터는 다음 주장에는 약한 근거만 제공한다.

- 전역 네트워크 전파 상태
- miner intent
- confirmation probability
- replacement 발생 여부
- pending payment의 최종성
- 내 노드에서 보이지 않는 transaction의 네트워크 전체 부재

### 아직 온체인이 아님

미확정 transaction은 온체인이 아니다. 그것은 pre-chain network state다. active chain에 남는 블록에 포함되어야만 on-chain state가 된다.

기관 리포팅에서는 이 구분이 핵심이다. pending deposit과 confirmed deposit은 같은 상태가 아니다.

---

## 11. Institutional Thinking

### Settlement Risk

기관은 단일 mempool에 보였다는 이유만으로 고위험 가치를 즉시 credit하면 안 된다. 최소한 다음을 평가해야 한다.

- confirmation depth
- RBF signaling
- conflicting transaction 존재 여부
- 현재 시장 대비 fee rate
- 미확정 ancestor 존재 여부
- package dependency
- 복수 노드 전파 관측
- counterparty risk

### Fee Management

mempool 모니터링은 다음 실무에 도움을 준다.

- withdrawal batching
- CPFP rescue
- RBF fee bumping
- consolidation timing
- 목표 confirmation 구간 fee 추정

다만 이러한 판단은 로컬 노드 편향과 빠르게 바뀌는 block-space 수요를 반영해야 한다.

### 운영 기록

시스템은 최소한 다음을 남기는 것이 좋다.

- transaction creation time
- node별 first-seen time
- node software version
- 알려진 policy settings
- fee rate와 package context
- conflicts와 replacements
- confirmation block 또는 removal reason

### Compliance

pending inflow/outflow 대시보드는 미확정 상태를 명시적으로 라벨링해야 한다. mempool event를 settled transfer처럼 보여 주면 안 된다.

---

## 12. Common Misinterpretations

### "mempool은 전역적으로 하나다"

아니다. 각 노드는 자신만의 mempool을 가진다.

### "mempool에 있으면 곧 확인된다"

아니다. 이는 한 노드가 로컬에서 받아들였다는 뜻일 뿐이며, 실제 확인은 miner selection, fee market, conflict, 이후 블록 전개에 달려 있다.

### "내 mempool에 없으면 broadcast되지 않은 것이다"

아니다. 다른 곳에는 존재할 수 있고, 네 노드 정책 기준 아래에 있을 수 있으며, 도착했다가 축출되었을 수도 있고, miner에게 직접 제출되었을 수도 있다.

### "RBF는 사기다"

아니다. RBF는 미확정 transaction 교체 정책이다. 정상적 fee bumping에도 쓰이고, 적대적 replacement에도 쓰일 수 있다. 문맥이 중요하다.

### "consensus-valid면 relayable이다"

아니다. consensus-valid transaction도 로컬 policy를 통과하지 못할 수 있다.

### "mempool fee rate는 고정된 시장 가격이다"

아니다. 그것은 로컬이며 시간에 따라 변하는 신호다.

---

## 13. Research Questions

1. fee spike 동안 잘 연결된 public node들 사이의 mempool 차이는 얼마나 큰가?
2. 로컬 mempool에서 사라졌다가 나중에 확인되는 transaction은 얼마나 자주 발생하는가?
3. RBF와 CPFP는 기관 출금 운영에 어떤 구조적 영향을 주는가?
4. 미확정 ancestor가 있을 때 pending balance를 어떻게 보고해야 하는가?
5. cluster mempool은 fee bumping과 block-template 분석을 어떻게 바꾸는가?
6. pending deposit을 low-risk로 간주하기 전에 어떤 증거가 필요한가?
7. private relay와 direct-to-miner submission은 public mempool 관측을 얼마나 약화시키는가?

---

## 14. Practical Exercises

### Exercise 1: 로컬 mempool 점검

다음을 실행하라.

```bash
bitcoin-cli getrawmempool true
```

세 transaction에 대해 다음을 기록하라.

- `vsize`
- `weight`
- `fees.base`
- `wtxid`
- `depends`
- `spentby`
- `bip125-replaceable`
- `unbroadcast`

### Exercise 2: fee rate 계산

하나의 mempool transaction에 대해:

```text
fee_rate = fees.base / vsize
```

현재 fee estimate와 최근 block inclusion 결과와 비교하라.

### Exercise 3: dependency 식별

`depends` 또는 `spentby`가 비어 있지 않은 transaction을 선택하라. 다음을 설명하라.

- 어떤 transaction이 parent인가
- 어떤 transaction이 child인가
- CPFP가 관련될 수 있는가
- child가 parent보다 먼저 확인될 수 없는 이유는 무엇인가

### Exercise 4: consensus vs policy

| Statement | Consensus | Policy | Local observation | Analytics |
|---|---:|---:|---:|---:|
| 내 노드 mempool에 transaction이 보인다 | No | Yes | Yes | Evidence |
| transaction이 유효한 블록에 들어갈 수 있다 | Yes | No | No | Fact if validated |
| transaction이 BIP125 replaceability를 signal한다 | No | Yes | Yes | Risk signal |
| transaction이 다음 블록에 확인될 것이다 | No | No | No | Forecast |
| transaction이 전역적으로 없다 | No | No | No | Usually unknowable |

---

## 15. Evidence Classification

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BIP-0125 | Policy BIP | Opt-in full RBF signaling and replacement rules | A |
| REF-BTC-CORE-31-RELEASE-001 | Release documentation | Cluster mempool, cluster limits, chunk ordering, current fee-estimator note | A |
| REF-BTC-CORE-TXMEMPOOL-001 | Primary implementation source | `CTxMemPool`, map indexes, `mapNextTx`, `CCoinsViewMemPool` | A |
| REF-BTC-CORE-MEMPOOL-ENTRY-001 | Primary implementation source | `CTxMemPoolEntry`, `LockPoints`, mempool transaction metadata | A |
| REF-BTC-CORE-VALIDATION-001 | Primary implementation source | `AcceptToMemoryPool` and mempool acceptance result | A |
| REF-BTC-CORE-RBF-001 | Primary implementation source | `IsRBFOptIn` and replacement fee checks | A |
| REF-BTC-CORE-POLICY-001 | Primary implementation source | Standardness and relay policy checks | A |
| REF-BTC-CORE-RPC-MEMPOOL-001 | RPC documentation | `getrawmempool` fields and mempool sequence output | A |
| REF-BTC-CORE-RPC-CONSISTENCY-001 | Core documentation | RPC consistency guarantees and mempool-state caveats | A |
| REF-BTC-CORE-26-RELEASE-001 | Release documentation | `submitpackage` and package CPFP introduction | B |

### Claim Ledger

| Claim | Classification | Evidence |
|---|---|---|
| mempool state는 로컬 노드 상태이며 consensus state가 아니다 | FACT | Core implementation and RPC consistency docs |
| `CTxMemPool`은 다음 블록에 포함될 수 있는 미확정 transaction을 저장한다 | FACT | Bitcoin Core `CTxMemPool` documentation |
| mempool entry는 txid, wtxid, entry time 등으로 인덱싱된다 | FACT | Bitcoin Core `txmempool.h` documentation |
| `mapNextTx`는 outpoint에서 mempool spender로 매핑한다 | FACT | Bitcoin Core `CTxMemPool` documentation |
| `AcceptToMemoryPool`은 transaction을 추가 시도하고 acceptance/rejection 결과를 반환한다 | FACT | Bitcoin Core `validation.cpp` Doxygen |
| BIP125 replaceability는 미확정 transaction에 대한 policy다 | FACT | BIP125 and Bitcoin Core RBF source |
| Bitcoin Core 31.0은 cluster mempool을 사용하며 예전 ancestor/descendant size/count 제한 설명과 구분되어야 한다 | FACT | Bitcoin Core 31.0 release notes |
| 한 mempool에 존재하는 transaction은 반드시 확인된다 | COUNTERCLAIM | Rejected; mempool observation is local and non-final |
| 내 mempool에 없다는 사실은 네트워크 전체 부재를 증명한다 | COUNTERCLAIM | Rejected; no single global mempool |
| CPFP는 dependent transactions를 함께 보게 함으로써 유인을 바꾼다 | INTERPRETATION | Derived from package feerate model and package docs |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | 직접적인 1차 자료로 뒷받침됨 |
| INTERPRETATION | 사실을 기반으로 한 분석적 종합 |
| POLICY | relay, mempool, mining 관행 |
| HEURISTIC | 실무적 규칙이지만 반례가 있음 |
| UNKNOWN | 근거가 부족함 |

---

## 16. Knowledge Graph

```text
BITCOIN-017 Mempool
|
+-- builds_on: BITCOIN-015 Transactions in Depth
+-- builds_on: BITCOIN-016 Script & ScriptPubKey
|
+-- mempool
|   +-- property: node-local
|   +-- stores: unconfirmed accepted transactions
|   +-- indexed_by: txid, wtxid, entry time
|
+-- acceptance
|   +-- consensus_checks
|   +-- policy_checks
|   +-- fee_checks
|   +-- replacement_checks
|
+-- relationships
|   +-- parent
|   +-- child
|   +-- conflict
|   +-- cluster
|   +-- chunk
|
+-- fee mechanics
|   +-- RBF
|   +-- CPFP
|   +-- package feerate
|
+-- Bitcoin Core
|   +-- CTxMemPool
|   +-- CTxMemPoolEntry
|   +-- AcceptToMemoryPool
|   +-- IsRBFOptIn
|
+-- analysis
    +-- facts: local first-seen, dependencies, fee rate
    +-- risks: replacement, eviction, non-confirmation
    +-- caveat: no global mempool
```

---

## 17. 참고문헌

[^ref-bip-0125]: David A. Harding and Peter Todd, "BIP 125: Opt-in Full Replace-by-Fee Signaling," 2015-12-04, https://bips.dev/125/, accessed 2026-08-04.
[^ref-btc-core-31-release]: Bitcoin Core Contributors, "Bitcoin Core 31.0 Release Notes," mempool, cluster mempool, replacement, RPC, and fee-estimation changes, https://bitcoin.org/en/releases/31.0/, accessed 2026-08-04.
[^ref-btc-core-26-release]: Bitcoin Core Contributors, "Bitcoin Core 26.0 Release Notes," `submitpackage` and package CPFP notes, https://bitcoincore.org/en/releases/26.0/, accessed 2026-08-04.
[^ref-btc-core-txmempool]: Bitcoin Core Contributors, `src/txmempool.h`, `CTxMemPool`, `mapTx`, `mapNextTx`, `ChangeSet`, and `CCoinsViewMemPool`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/txmempool_8h_source.html and https://doxygen.bitcoincore.org/class_c_tx_mem_pool.html, accessed 2026-08-04.
[^ref-btc-core-mempool-entry]: Bitcoin Core Contributors, `src/kernel/mempool_entry.h`, `CTxMemPoolEntry`, `LockPoints`, and transaction info structures, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/mempool__entry_8h_source.html, accessed 2026-08-04.
[^ref-btc-core-validation]: Bitcoin Core Contributors, `src/validation.cpp`, `AcceptToMemoryPool`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/validation_8cpp.html, accessed 2026-08-04.
[^ref-btc-core-rbf]: Bitcoin Core Contributors, `src/policy/rbf.cpp`, `IsRBFOptIn` and replacement fee checks, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/policy_2rbf_8cpp_source.html, accessed 2026-08-04.
[^ref-btc-core-policy]: Bitcoin Core Contributors, `src/policy/policy.cpp`, standardness and relay policy checks, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/policy_8cpp_source.html, accessed 2026-08-04.
[^ref-btc-core-rpc-mempool]: Bitcoin Developer Documentation, "`getrawmempool` RPC," mempool transaction fields, `wtxid`, dependency fields, BIP125 replaceability, and unbroadcast status, https://developer.bitcoin.org/reference/rpc/getrawmempool.html, accessed 2026-08-04.
[^ref-btc-core-rpc-consistency]: Bitcoin Core Contributors, "JSON-RPC interface," RPC consistency guarantees and transaction pool caveats, https://github.com/bitcoin/bitcoin/blob/master/doc/JSON-RPC-interface.md, accessed 2026-08-04.

---

## 18. 교차 참조

### Parent

- Bitcoin Internals

### Previous

- BITCOIN-016 — Script & ScriptPubKey

### Next

- BITCOIN-018 — Transaction Fees

### Related

- BITCOIN-014 — UTXO Model
- BITCOIN-015 — Transactions in Depth
- BITCOIN-016 — Script & ScriptPubKey
- BITCOIN-018 — Transaction Fees
- BITCOIN-023 — Chain Reorganization
- POW-009 — Coinbase Transaction and Mining Commitments
- POW-013 — Bitcoin Core Proof-of-Work Validation

---

## Review Status

### Technical Review

Passed.

- mempool state와 consensus state를 분리했다.
- Bitcoin Core 31.0 cluster mempool 동작을 예전 ancestor/descendant-limit 설명과 구분했다.
- RBF, CPFP, package acceptance, eviction, expiry, reorg handling을 policy 또는 local-node behavior로 분류했다.

### Evidence Review

Passed.

- RBF 관련 설명은 BIP125와 Core RBF source에 연결했다.
- cluster mempool 설명은 31.0 release notes에 연결했다.
- `CTxMemPool`, `CTxMemPoolEntry`, `AcceptToMemoryPool`, RPC 설명은 1차 구현 또는 공식 문서에 연결했다.

### Editorial Review

Passed.

- 기존 deep-dive 구조를 유지했다.
- 용어는 mempool, policy, cluster, chunk, RBF, CPFP, package로 일관화했다.

### Adversarial Review

Passed.

- 단일 글로벌 mempool이 존재한다고 주장하지 않았다.
- mempool acceptance를 confirmation과 동일시하지 않았다.
- RBF를 사기와 동일시하지 않았다.
- consensus-valid와 relayable을 혼동하지 않았다.

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
