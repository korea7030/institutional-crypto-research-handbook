---
knowledge_id: BITCOIN-027
title: Fee Market
subtitle: blockspace scarcity, feerate 경쟁, mempool policy, miner selection, 그리고 합의와 시장 결과의 경계
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 120 min
estimated_study: 350 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Economics
  - Fees
  - Mempool
parent:
  - Bitcoin Economics
prerequisites:
  - BITCOIN-017
  - BITCOIN-018
  - BITCOIN-020
  - BITCOIN-025
  - BITCOIN-026
related_topics:
  - Feerate
  - Blockspace
  - Mempool Policy
  - Block Assembly
  - Fee Estimation
  - Security Budget
  - CPFP
  - RBF
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-TRANSACTIONS-001
  - REF-BTC-CORE-FEERATE-001
  - REF-BTC-CORE-POLICY-001
  - REF-BTC-CORE-TXMEMPOOL-001
  - REF-BTC-CORE-BLOCKASSEMBLER-001
  - REF-BTC-CORE-FEE-ESTIMATOR-001
  - REF-BTC-CORE-31-RELEASE-001
tags:
  - bitcoin
  - economics
  - fees
  - fee-market
  - mempool
  - blockspace
  - feerate
  - mining
---

# Fee Market
> Bitcoin Economics  
> Research Unit: BITCOIN-027

---

## Research Brief

```yaml
knowledge_id: BITCOIN-027
title: Fee Market
research_question: >
  How does Bitcoin's fee market emerge from scarce blockspace, transaction
  demand, mempool policy, miner transaction selection, and feerate competition,
  and how should analysts separate consensus-valid fee payment from local relay
  policy, estimation heuristics, and broader security-budget interpretation?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-017
  - BITCOIN-018
  - BITCOIN-020
  - BITCOIN-025
  - BITCOIN-026
parent: Bitcoin Economics
previous: BITCOIN-026
next: BITCOIN-028
related_topics:
  - Block Assembly
  - Feerate Estimation
  - Mempool Eviction
  - CPFP
  - RBF
  - Security Budget
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
  - Non-Bitcoin fee markets
  - Price prediction using fee metrics
  - Full wallet UX guidance
  - Exhaustive package-relay policy history
  - Detailed miner treasury strategy analysis
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Bitcoin fee market을 fixed protocol price list가 아니라 scarce blockspace를 둘러싼 경쟁으로 정의할 수 있다.
- fee amount와 feerate를 구분할 수 있다.
- miner selection이 보통 block constraint 아래의 expected fee density에 의해 움직이는 이유를 설명할 수 있다.
- consensus-valid fee와 local mempool/relay policy를 구분할 수 있다.
- CPFP와 RBF가 fee-market behavior에서 왜 중요한지 설명할 수 있다.
- fee estimation이 확률적이며 node-local한 이유를 설명할 수 있다.
- subsidy decline이 fee의 분석 중요성을 높이지만, fee sufficiency를 프로토콜 보장으로 만들지는 않는다는 점을 설명할 수 있다.

---

## 2. 핵심 질문

1. Bitcoin fee market이란 무엇인가?
2. 사용자는 왜 절대 fee amount보다 feerate로 경쟁하는가?
3. blockspace scarcity는 어디서 오는가?
4. mempool policy와 miner incentive는 어떻게 상호작용하는가?
5. local mempool 관측이 왜 global fee market이 아닌가?
6. CPFP와 RBF는 fee discovery와 repricing에서 어떤 역할을 하는가?
7. 실제로 fee estimation은 어떻게 동작하는가?
8. high-fee transaction도 왜 대기할 수 있는가?
9. 높은 fee share는 무엇을 시사하고, 무엇을 시사하지 않는가?

---

## 3. Executive Summary

Bitcoin의 fee market은 사용자가 scarce blockspace를 두고 transaction fee를 제시하며 경쟁하는 과정이다. 보통 이 경쟁은 transaction size 또는 package size 대비 feerate 기준으로 평가된다. 프로토콜은 전역적인 required fee schedule을 게시하지 않는다. fee는 inclusion demand, miner selection incentive, relay policy, block-capacity constraint의 상호작용에서 형성된다.[^ref-btc-wp] [^ref-btc-core-feerate] [^ref-btc-core-policy]

transaction level에서 fee는 total input value와 total output value의 차이일 뿐이다. 그러나 시장 수준에서는 feerate가 더 유용하다. miner는 제약된 block을 채워야 하므로 절대 fee보다는 희소한 blockspace 단위당 얼마를 버는지가 더 중요하기 때문이다.[^ref-btc-dev-transactions] [^ref-btc-core-blockassembler]

현대 Bitcoin Core 동작은 이 시장 구조를 더 선명하게 만든다.

- `CFeeRate`가 policy code에서 fee rate를 표현한다.[^ref-btc-core-feerate]
- mempool과 relay policy가 local acceptance/eviction threshold를 부과한다.[^ref-btc-core-policy] [^ref-btc-core-txmempool]
- block assembly는 block constraint 아래 expected mining feerate를 기준으로 transaction을 선택한다.[^ref-btc-core-blockassembler]
- fee estimation은 프로토콜 oracle을 읽는 것이 아니라 최근 confirmation outcome으로 likely inclusion threshold를 추정한다.[^ref-btc-core-fee-estimator]
- Bitcoin Core 31.0의 cluster mempool redesign은 chunk와 feerate-diagram reasoning을 통해 ordering, relay, eviction, replacement를 더 package-aware하게 만들었다.[^ref-btc-core-31-release]

분석가에게 fee market은 하나의 node에서 보이는 단일한 global number가 아니라, blockspace에 대한 부분 관측·정책 매개 경매로 다뤄져야 한다.

---

## 4. 프로토콜 구조

### Transaction Arithmetic로서의 Fee

transaction fee는 다음과 같다.

```text
fee = sum(inputs) - sum(outputs)
```

fee는 implicit하다. transaction format에 dedicated fee field는 없다.[^ref-btc-dev-transactions]

### Blockspace Competition으로서의 Fee Market

fee market은 inclusion을 원하는 candidate transaction 수가 available blockspace보다 많아질 때 시작된다. 여기서 binding scarcity는 추상적인 "transaction 수"가 아니라 block weight, 관련 validation constraint, miner의 block-construction choice다.

### Feerate vs Absolute Fee

miner는 대개 공간 제약 아래 fee density를 최적화하므로:

| Metric | Why It Matters |
|---|---|
| Absolute fee | 특정 transaction이 주는 총수익 |
| Feerate | 희소한 blockspace 단위당 수익 |
| Package feerate | parent/child를 함께 채굴해야 할 때 중요 |

그래서 절대 fee는 더 낮지만 feerate가 더 높은 작은 transaction이, 절대 fee는 더 크지만 더 큰 transaction보다 먼저 선택될 수 있다.

---

## 5. Market Mechanics

### Blockspace Scarcity

모든 block은 유한한 weight capacity를 가진다. candidate block을 구성하는 miner는 mempool의 모든 transaction을 포함할 수 없으므로 우선순위를 매긴다. fee market은 이 scarcity 아래에서 일어나는 prioritization process다.[^ref-btc-core-policy] [^ref-btc-core-blockassembler]

### User Bidding

사용자는 transaction structure와 fee level을 선택함으로써 사실상 inclusion에 "입찰"한다. 다만 이 bid는 완전한 정보 위에서 이루어지지 않는다.

- future competing demand를 확실히 알 수 없다.
- node마다 mempool이 다르다.
- confirmation target이 다르다.
- package topology가 중요하다.
- miner behavior는 완전히 균질하지 않다.

### Miner Selection

miner는 block limit, finality constraint, dependency ordering 아래 gross expected revenue를 신경 쓴다. 현대 Core에서는 block assembly가 expected mining feerate를 기준으로 chunk를 처리하며, mining code는 locally constructed block에 포함할 transaction의 minimum feerate threshold도 적용한다.[^ref-btc-core-blockassembler] [^ref-btc-core-policy]

### CPFP와 RBF

fee market은 initial bid만으로 결정되지 않는다.

- CPFP는 high-fee child를 통해 low-fee parent의 package economics를 개선한다.
- RBF는 policy rule 아래에서 replacement transaction으로 경제적 제안을 높이거나 mempool state를 재구성하게 한다.

이 메커니즘은 fee bid가 broadcast 이후에도 수정될 수 있음을 뜻한다.

---

## 6. Technical Mechanics

### `CFeeRate`

Bitcoin Core는 policy code에서 satoshis per virtual byte로 fee rate를 표현하는 `CFeeRate`를 사용한다.[^ref-btc-core-feerate]

개념적으로는:

```text
feerate = fee / vsize
```

여기서 `vsize`는 transaction의 virtual size다.

### Block Minimum Mining Fee

policy code는 locally constructed block에 mining code가 포함할 transaction의 minimum feerate를 설정하는 `DEFAULT_BLOCK_MIN_TX_FEE` 즉 `-blockmintxfee`의 기본값을 정의한다.[^ref-btc-core-policy]

이것은 consensus rule이 아니다. 다른 모든 consensus rule을 만족한다면 이보다 낮은 fee의 transaction을 포함한 block도 valid할 수 있다.

### Rolling and Local Mempool Floor

`CTxMemPool`은 admission과 eviction behavior의 일부로 rolling minimum fee 개념과 mempool sequence state를 유지한다.[^ref-btc-core-txmempool]

즉, mempool pressure를 받는 node는 lightly loaded node보다 더 높은 effective feerate를 admission 기준으로 요구할 수 있다. 둘 다 같은 consensus rule을 공유하더라도 그렇다.

### Cluster Mempool과 Chunk

Bitcoin Core 31.0은 mempool을 cluster-based design으로 재구현했다. release note는 다음을 설명한다.

- 이전 ancestor/descendant count logic을 대체하는 cluster size limit
- expected mining feerate 기반 chunk ordering
- block template creation, eviction, relay announcement, replacement validation에서 그 ordering을 사용
- mempool의 feerate diagram을 개선하는 방향으로 더 엄격해진 RBF acceptance[^ref-btc-core-31-release]

이는 policy/implementation 변화이지 consensus change가 아니다.

### Fee Estimation

`CBlockPolicyEstimator`는 feerate bucket별 confirmation outcome을 시간에 따라 추적해, 목표 block 수 안에 확인되기 위해 필요한 feerate를 추정한다.[^ref-btc-core-fee-estimator]

이 estimator는 empirical하고 probabilistic하다.

- recent history에 의존한다.
- 한 node의 observation을 반영한다.
- demand나 policy에 구조적 변화가 있으면 쉽게 빗나갈 수 있다.

---

## 7. Validation Boundaries

### Fee는 대부분 Consensus-Specified Price가 아니다

consensus는 transaction이 무에서 돈을 만들지 않아야 하고, block reward가 subsidy + fee를 넘지 않아야 한다고 요구한다. 하지만 universal market-clearing feerate를 prescribe하지는 않는다.

### Relay Policy는 Fee Market 자체가 아니다

node의 local acceptance threshold는 observed demand의 한 필터일 뿐 시장 전체가 아니다. 서로 다른 node는 다음이 다를 수 있다.

- local setting
- mempool pressure
- 관측하는 peer transaction set
- 도출하는 fee estimate

### Miner Policy는 Consensus가 아니다

mining code는 local policy나 block-template economics에 따라 low-fee transaction을 건너뛸 수 있다. 그러나 다른 miner는 다르게 선택할 수 있다. miner가 valid block에 포함한다면 low-fee transaction도 채굴될 수 있다.

---

## 8. Security Assumptions and Failure Modes

### Fee Share vs Security

subsidy가 줄어들수록 fee revenue는 total miner compensation에서 더 중요해진다. 그러나 높은 fee share만으로 security budget이 충분하다는 뜻은 아니다. 분석가는 다음을 함께 봐야 한다.

- absolute fee revenue
- BTC price
- miner cost
- hash-rate response
- attack economics

### Congestion과 Underbidding

blockspace demand가 급증하면 underpriced transaction은 오랫동안 pending으로 남거나 일부 mempool에서 축출될 수 있다. 이는 consensus failure가 아니라 scarce capacity 아래의 market rationing이다.

### Node-View Fragmentation

mempool은 local하고 policy-mediated이기 때문에, 한 node에서 수집한 fee-market data는 다음을 놓칠 수 있다.

- private transaction flow
- node-specific eviction behavior
- 아직 보이지 않는 package relationship
- miner-specific direct submission channel

### Estimation Failure

fee estimation은 regime change에서 성능이 나빠질 수 있다.

- sudden congestion
- rapid demand collapse
- large policy transition
- atypical miner inclusion pattern

recent history 기반 예측은 유용하지만 oracle은 아니다.

---

## 9. Mathematical or Economic Model

### Basic Feerate Identity

transaction 하나에 대해:

```text
feerate = fee / vsize
```

package에 대해:

```text
package_feerate = total_package_fee / total_package_vsize
```

이 때문에 low-fee parent와 high-fee child가 함께 보면 매력적인 조합이 될 수 있다.

### Capacity-Constrained Selection

block의 남은 effective capacity를 `C`라 하면, miner는 다음과 같은 constrained optimization 문제를 푼다.

```text
maximize total fees
subject to block weight, sigops, finality, and dependency constraints
```

완벽한 auction은 아니지만, 경제적으로는 capacity-constrained selection problem에 가깝다.

### Fee Share

다음을 두자.

- `S` = subsidy
- `F` = fee
- `R` = total miner revenue

```text
R = S + F
fee_share = F / (S + F)
```

era가 지날수록 `S`가 감소하므로, absolute fee가 같아도 fee share는 커진다. 그렇다고 total revenue가 여전히 fiat 기준 또는 attack-cost 기준으로 낮다면 real security가 커진다고 볼 수는 없다.

---

## 10. Bitcoin Core 구현

### `policy/feerate.h`

`CFeeRate`는 policy와 wallet-related code 전반에서 사용되는 fee-rate representation을 제공한다.[^ref-btc-core-feerate]

### `policy/policy.h`

`policy.h`는 `DEFAULT_BLOCK_MIN_TX_FEE`를 포함해 block construction과 standardness-related policy에 중요한 기본값을 정의한다.[^ref-btc-core-policy]

### `txmempool.h`

`CTxMemPool`은 local pool state, rolling fee concept, block construction에 쓰이는 builder interface를 유지한다.[^ref-btc-core-txmempool]

### `node::BlockAssembler`

`BlockAssembler`는 block template을 구성하며, 현재 Core에서는 block limit과 transaction-level check 아래 chunk feerate를 기준으로 transaction을 추가한다.[^ref-btc-core-blockassembler]

### `CBlockPolicyEstimator`

estimator는 bucketized confirmation outcome을 추적해 target confirmation horizon에 대한 fee-rate estimate를 제공한다.[^ref-btc-core-fee-estimator]

### Bitcoin Core 31.0 Release Note

31.0 release note는 current cluster mempool design과 ordering, eviction, relay, replacement validation에서의 사용을 문서화하기 때문에 중요하다.[^ref-btc-core-31-release]

---

## 11. 온체인 함의

### 직접 측정 가능한 것

분석가는 다음을 직접 측정할 수 있다.

- confirmed transaction당 paid fee
- block fee total
- block reward에서 fee share
- confirmed transaction의 confirmation delay
- block fullness와 weight usage

### Chain Alone으로 직접 관찰할 수 없는 것

chain data만으로는 보통 다음을 알 수 없다.

- miner가 관측한 full mempool state
- inclusion race에서 진 exact fee alternative
- local eviction threshold
- public relay 밖의 direct miner submission
- 특정 transaction이 기다린 정확한 이유

### 왜 중요한가

어떤 transaction이 특정 fee로 confirmed되었다고 해서, 그와 같은 fee가 broadcast 시점이나 다른 node, 다른 transaction topology에서도 충분했음을 증명하지는 않는다.

---

## 12. Institutional Thinking

기관은 Bitcoin fee market을 historical statistic이 아니라 live operating environment로 다뤄야 한다.

### Practical Implications

- fee policy는 static이 아니라 target-based이고 adaptive해야 한다.
- 가능하다면 mempool observation은 multiple high-quality vantage point에서 수집해야 한다.
- settlement와 treasury analytics는 gross fee revenue와 newly issued subsidy를 분리해야 한다.
- security-budget commentary는 fee share와 absolute fee revenue를 함께 보고해야 한다.
- operational system은 CPFP, RBF, reorg-aware confirmation tracking을 고려해야 한다.

---

## 13. Common Misinterpretations

### "Bitcoin에는 고정된 required transaction fee가 있다"

틀렸다. Bitcoin에는 universal protocol-mandated market fee schedule이 없다.

### "절대 fee가 가장 높으면 항상 이긴다"

틀렸다. blockspace constraint 아래에서는 보통 feerate와 package economics가 더 중요하다.

### "내 node의 mempool이 곧 시장이다"

틀렸다. 그것은 분산된 시장의 local하고 policy-mediated한 한 시야일 뿐이다.

### "Low-fee transaction은 invalid하다"

틀렸다. valid할 수 있지만 현재 조건에서 relay나 mining에 비경제적일 뿐이다.

### "High fee share는 secure long-run mining economics를 증명한다"

틀렸다. security는 fee share만이 아니라 attack cost 대비 total compensation에 달려 있다.

---

## 14. Research Questions

1. congestion 동안 first-seen fee-market condition의 cross-node variance는 얼마나 큰가?
2. cluster mempool은 empirical block-template selection과 eviction behavior를 얼마나 바꿨는가?
3. 어떤 fee-market metric이 서로 다른 demand regime에서 confirmation delay를 가장 잘 예측하는가?
4. 기관은 security analysis에서 fee share, absolute fee revenue, hash-cost estimate를 어떻게 결합해야 하는가?
5. 경제적으로 중요한 fee flow 중 ordinary public relay path를 우회하는 비중은 얼마인가?

---

## 15. Practical Exercises

### Exercise 1

fee와 virtual size가 다른 여러 transaction을 받아 feerate로 순위를 매기고, 왜 그 순위가 inclusion에 중요한지 설명하라.

### Exercise 2

CPFP가 low-fee parent의 inclusion prospect를 어떻게 개선하는지 보여 주는 simple package-feerate example을 만들어라.

### Exercise 3

최근 block을 사용해 다음을 계산하라.

- total fees per block
- median confirmed feerate
- fee share of block reward
- block fullness 기반 rough congestion proxy

### Exercise 4

여러 confirmation target에 대한 node fee estimate를 실제 이후 inclusion outcome과 비교하고 forecast error를 기록하라.

---

## 16. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Fee arithmetic and feerate primitives | Directly specified | Developer docs and `CFeeRate` |
| Block-construction and policy defaults | Directly specified | `policy.h`, `BlockAssembler`, `txmempool.h` |
| Fee estimation behavior | Directly specified | `CBlockPolicyEstimator` |
| Cluster mempool ordering and replacement behavior | Directly specified | Bitcoin Core 31.0 release notes |
| Security-budget interpretation | Inference from sources | Economic analysis based on fee and subsidy structure |

---

## 17. Knowledge Graph

```text
Fee Market
├─ Inputs
│  ├─ transaction demand
│  ├─ blockspace scarcity
│  ├─ mempool policy
│  └─ miner incentives
├─ Pricing
│  ├─ fee
│  ├─ feerate
│  ├─ package feerate
│  └─ estimation
├─ Mechanisms
│  ├─ CPFP
│  ├─ RBF
│  ├─ eviction
│  └─ block assembly
├─ Implementation
│  ├─ CFeeRate
│  ├─ CTxMemPool
│  ├─ BlockAssembler
│  ├─ fee estimator
│  └─ cluster mempool
└─ Risks
   ├─ node-view bias
   ├─ estimation error
   ├─ congestion
   └─ security-budget overstatement
```

---

## 18. 참고문헌

### Primary Sources

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," especially Section 6. https://bitcoin.org/bitcoin.pdf
[^ref-btc-dev-transactions]: Bitcoin Developer Reference, "Transactions," including raw transaction structure and implicit fee arithmetic. https://developer.bitcoin.org/reference/transactions.html
[^ref-btc-core-feerate]: Bitcoin Core Doxygen, `policy/feerate.h` and `CFeeRate`. https://doxygen.bitcoincore.org/class_c_fee_rate.html
[^ref-btc-core-policy]: Bitcoin Core Doxygen, `policy/policy.h`, including `DEFAULT_BLOCK_MIN_TX_FEE`. https://doxygen.bitcoincore.org/policy_8h_source.html
[^ref-btc-core-txmempool]: Bitcoin Core Doxygen, `txmempool.h`, including rolling fee state and block-builder chunk interfaces. https://doxygen.bitcoincore.org/txmempool_8h_source.html
[^ref-btc-core-blockassembler]: Bitcoin Core Doxygen, `node::BlockAssembler` and `miner.cpp`, including chunk-based block assembly. https://doxygen.bitcoincore.org/classnode_1_1_block_assembler.html
[^ref-btc-core-fee-estimator]: Bitcoin Core Doxygen, `CBlockPolicyEstimator`. https://doxygen.bitcoincore.org/class_c_block_policy_estimator.html
[^ref-btc-core-31-release]: Bitcoin Core 31.0 release notes, mempool and fee-estimation changes. https://bitcoincore.org/en/releases/31.0/

### Supporting Interpretation Notes

- Where this document discusses security sufficiency, partial observability, or market-level inference from node-local data, those statements are analytical interpretations built on documented policy, mempool, and block-construction behavior rather than direct protocol guarantees.

---

## 19. 교차 참조

### Previous

- BITCOIN-026 — Halving

### Next

- BITCOIN-028 — Security Budget

### Related

- BITCOIN-017 — Mempool
- BITCOIN-018 — Transaction Fees
- BITCOIN-020 — Mining
- BITCOIN-025 — Bitcoin Monetary Policy
- BITCOIN-026 — Halving
- BITCOIN-028 — Security Budget
- BITCOIN-029 — Bitcoin Game Theory

---

## Review Status

### Technical Review

Passed.

- fee arithmetic, feerate competition, policy filter, miner selection, estimation을 분리했다.
- consensus-valid fee와 relay/mining policy를 구분했다.
- current Core block assembly와 mempool behavior를 chunk/cluster-aware terminology로 설명했다.
- security-budget implication을 protocol fact가 아니라 analytical layer로 유지했다.

### Evidence Review

Passed.

- developer reference가 fee arithmetic를 뒷받침한다.
- Core Doxygen이 fee-rate, policy, mempool, assembler, estimator 설명을 뒷받침한다.
- Bitcoin Core 31.0 release note가 cluster mempool behavior 설명을 뒷받침한다.
- observability와 security 관련 해석은 inference로 표시했다.

### Editorial Review

Passed.

- 구조는 프로젝트 deep-dive format을 따른다.
- metadata가 완전하다.
- 용어는 fee, feerate, package feerate, blockspace, estimator, fee share로 일관된다.
- table과 code fence가 닫혀 있다.

### Adversarial Review

Passed.

- universal fixed Bitcoin fee가 있다고 주장하지 않았다.
- 한 node의 mempool을 시장 전체와 동일시하지 않았다.
- miner choice를 절대 fee amount 하나로 환원하지 않았다.
- relay rejection을 consensus invalidity와 혼동하지 않았다.
- fee share만으로 security가 증명된다고 과장하지 않았다.

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
