---
knowledge_id: BITCOIN-020
title: Mining
subtitle: 블록 템플릿 구성, 작업증명 탐색, 코인베이스 보상, 풀, share, 전파, 그리고 채굴자 인센티브
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 110 min
estimated_study: 330 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Mining
  - Proof of Work
  - Block Construction
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-017
  - BITCOIN-018
  - POW-009
  - POW-012
  - POW-013
related_topics:
  - Block Templates
  - Coinbase Transaction
  - Proof-of-Work Search
  - Difficulty Target
  - Mining Pools
  - Shares
  - Transaction Selection
  - Block Propagation
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-MINING-001
  - REF-BIP-0022
  - REF-BIP-0023
  - REF-BTC-CORE-MINER-001
  - REF-BTC-CORE-MINING-IFACE-001
  - REF-BTC-CORE-MINING-RPC-001
  - REF-BTC-CORE-31-RELEASE-001
  - REF-BTC-CORE-POW-001
  - REF-BTC-CORE-VALIDATION-001
tags:
  - bitcoin
  - internals
  - mining
  - block-template
  - coinbase
  - proof-of-work
  - difficulty
  - mining-pools
  - getblocktemplate
---

# Mining
> Bitcoin Internals  
> Research Unit: BITCOIN-020

---

## Research Brief

```yaml
knowledge_id: BITCOIN-020
title: Mining
research_question: >
  How do Bitcoin miners construct candidate blocks, search for valid
  proof-of-work, coordinate through solo or pooled mining workflows, collect
  subsidy and fees, and interact with node policy, mempool state, and
  consensus validation?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-017
  - BITCOIN-018
  - POW-009
  - POW-012
  - POW-013
parent: Bitcoin Internals
previous: BITCOIN-019
next: BITCOIN-021
related_topics:
  - Mempool
  - Transaction Fees
  - Coinbase Transaction
  - Difficulty Adjustment
  - Proof-of-Work Validation
  - Mining Pools
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
  - ASIC electrical engineering
  - Mining facility power procurement
  - Pool payout formula accounting
  - Real-time miner identity attribution
  - Full Stratum v2 specification
  - Tax or regulatory treatment of mining revenue
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Bitcoin에서 채굴자가 무엇을 하는지 설명할 수 있다.
- block construction과 작업증명 탐색을 구분할 수 있다.
- transaction selection, coinbase construction, Merkle root commitment, header hashing이 어떻게 연결되는지 설명할 수 있다.
- mining search에서 nonce, extra nonce, time, version, transaction-set change를 왜 사용하는지 설명할 수 있다.
- solo mining, pooled mining, pool operator, hasher, share를 구분할 수 있다.
- pool share가 Bitcoin block이 아닌 이유를 설명할 수 있다.
- 채굴자가 block subsidy와 transaction fee를 어떻게 수취하는지 설명할 수 있다.
- mining policy가 node relay policy와 다를 수 있는 이유를 설명할 수 있다.
- Bitcoin Core의 mining 관련 source area와 RPC를 식별할 수 있다.
- coinbase label, pool tag, hashrate estimate를 완전한 신원 증거로 오해하지 않을 수 있다.

---

## 2. 핵심 질문

1. Bitcoin mining이란 무엇인가?
2. candidate block이란 무엇인가?
3. block template이란 무엇인가?
4. 채굴자는 transaction을 어떻게 선택하는가?
5. coinbase transaction은 채굴자에게 어떻게 보상하는가?
6. coinbase data를 바꾸면 왜 Merkle root가 바뀌는가?
7. 왜 32-bit header nonce만으로는 현대 채굴에 충분하지 않은가?
8. network target과 pool share target의 차이는 무엇인가?
9. solo mining이란 무엇인가?
10. pooled mining이란 무엇인가?
11. Bitcoin Core의 mining code는 무엇을 구성하는가?
12. mining이 온체인에서 드러내는 것은 무엇이고, 여전히 불확실한 것은 무엇인가?

---

## 3. Executive Summary

Bitcoin mining은 candidate block을 구성하고, 현재 작업증명 목표값(Target)보다 작은 block header hash를 찾는 과정이다. 유효한 block은 작업증명뿐 아니라 전체 block validity 규칙도 만족해야 한다. target 이하의 header hash는 필요조건일 뿐 충분조건이 아니다. block의 transaction, coinbase value, Merkle commitment, timestamp, difficulty bits, contextual rule까지 모두 유효해야 하기 때문이다.[^ref-btc-core-pow] [^ref-btc-core-validation]

mining workflow는 다음과 같다.

```text
observe chain tip
select transactions
construct coinbase transaction
compute Merkle root
build block header
iterate header-affecting fields
find hash below target
submit and propagate full block
```

Bitcoin Developer 문서는 solo mining과 pooled mining을 설명한다. solo miner는 모든 보상을 가져가지만 분산이 매우 크다. pooled miner는 해시파워를 합쳐서, 제공한 work에 비례한 더 작고 더 빈번한 payout을 받는다.[^ref-btc-dev-mining]

Bitcoin Core는 일반적인 production 환경에서 직접 ASIC으로 채굴하지는 않는다. 대신 mining 관련 code는 block template을 구성하고, submission을 검증하며, mining interface를 노출한다. `BlockAssembler`는 valid proof-of-work 없이 새로운 block을 생성하는 것으로 문서화되어 있으며, `CreateNewBlock`은 새로운 block template을 만든다.[^ref-btc-core-miner]

채굴은 경제적 유인에 의해 움직인다.

```text
expected miner revenue = block subsidy + transaction fees - operating costs
```

채굴자는 보통 희소한 block resource당 높은 fee rate의 transaction set을 선호하지만, 정확한 동작은 template policy, private orderflow, pool rule, propagation risk, 운영 제약에 따라 달라질 수 있다.

분석 관점에서 mining data는 block, coinbase output, transaction selection, fee revenue, 대략적인 pool attribution에 대한 관측을 뒷받침한다. 그러나 모든 hasher의 실세계 신원이나 pool 뒤의 상업적 계약 구조 전체를 증명하지는 못한다.

---

## 4. 프로토콜 구조

### 블록 생산으로서의 채굴

채굴은 새로운 block을 blockchain에 추가해 transaction history를 수정하는 비용을 높인다. 백서는 가장 긴 체인이 가장 큰 작업증명을 대표하며, 인센티브는 신규 발행 코인과 transaction fee를 통해 제공된다고 설명한다.[^ref-btc-wp]

채굴자의 산출물은 block이다.

```text
block
  header
  transactions
```

header는 다음에 커밋한다.

- version
- previous block hash
- transaction Merkle root
- time
- compact difficulty target `nBits`
- nonce

### Candidate Block

candidate block은 작업증명을 찾기 전의 제안된 block이다.

여기에는 다음이 포함된다.

- previous-block reference
- coinbase transaction
- 선택된 non-coinbase transaction
- 유효한 transaction Merkle root
- hashing에 적합한 header field
- 허용된 subsidy + fee 이내의 coinbase value

### Block Template

block template은 mining software가 candidate block을 구성하는 데 쓰는 데이터다. BIP22는 `getblocktemplate`을 smart miner와 proxy를 위한 JSON-RPC method로 정의하며, 전체 block structure를 제공하고 필요에 따라 miner가 이를 조정하고 조립할 수 있게 한다.[^ref-bip-0022]

Bitcoin Core RPC 문서는 `getblocktemplate`이 block construction에 필요한 데이터를 반환하며 BIP 22, 23, 9, 145를 참조한다고 설명한다.[^ref-btc-core-mining-rpc]

### Coinbase Transaction

코인베이스 트랜잭션은 block의 첫 번째 transaction이다. maturity 이후 spendable reward를 만들며 다음을 청구한다.

```text
subsidy + included transaction fees
```

coinbase transaction은 mining search에도 영향을 준다. coinbase data가 바뀌면 coinbase transaction hash가 바뀌고, Merkle root가 바뀌며, 결국 block header가 바뀌기 때문이다.

### 작업증명 탐색

채굴은 candidate header를 다음 조건으로 시험한다.

```text
double_sha256(block_header) <= target
```

이 탐색은 확률적이다. SHA-256이 기대대로 동작한다고 가정하면 각 hash attempt는 header search space 위의 독립 시행으로 볼 수 있다.

---

## 5. Technical Mechanics

### Template 구성

template 구성은 일반적으로 다음을 포함한다.

1. active chain tip 읽기
2. next block height 설정
3. block version 선택
4. 필요한 difficulty bits 결정
5. mempool 또는 다른 source에서 transaction 선택
6. coinbase transaction 생성
7. Merkle root 계산
8. time과 nonce field 설정
9. mining software 또는 ASIC controller에 work 반환

Bitcoin Core의 `BlockAssembler::CreateNewBlock`은 block counter를 초기화하고, `CBlockTemplate`을 만들고, 첫 transaction으로 dummy coinbase를 추가하고, transaction을 선택한 뒤, 나중에 coinbase와 Merkle commitment data로 block을 갱신한다.[^ref-btc-core-miner]

### Transaction Selection

채굴자는 mining policy에 따라 transaction을 선택한다. 일반적인 유인은 희소한 block resource당 fee revenue를 최대화하는 것이며, 다음 제약을 받는다.

- consensus validity
- block weight
- signature operation limit
- transaction dependency
- package 또는 cluster fee-rate ordering
- pool policy
- private orderflow
- template freshness

Bitcoin Core 31.0 cluster mempool은 block template construction, eviction, relay announcement, replacement validation에서 chunk를 사용해 expected mining feerate 기준으로 transaction을 정렬한다.[^ref-btc-core-31-release]

### Header Search Field

block header에는 32-bit nonce가 있다. 현대 ASIC은 이 공간을 빠르게 소진하므로, mining system은 다른 header-affecting data도 함께 바꾼다.

| Field or input | Effect |
|---|---|
| Header nonce | 직접적인 header field |
| Coinbase extra nonce | coinbase txid, Merkle root, header를 변경 |
| Transaction set/order | Merkle root 변경 |
| Timestamp | header time 변경 |
| Version bits | 허용 규칙 안에서 header version 변경 가능 |

Bitcoin Developer 문서는 모든 nonce 값이 실패하면 mining software가 coinbase field의 extra nonce data를 바꿔 새로운 Merkle root를 만들고 block header를 갱신할 수 있다고 설명한다.[^ref-btc-dev-mining]

### Block Submission

유효한 hash를 찾으면 mining software는 full block을 제출한다. BIP22는 potential block 또는 share 제출을 위한 `submitblock`을 정의하며, 수락되면 `null`, 아니면 rejection reason을 반환한다.[^ref-bip-0022]

Bitcoin Core는 `getblocktemplate`, `submitblock`, `submitheader`, `getmininginfo`, `getnetworkhashps`, transaction prioritization RPC 등 mining RPC를 제공한다.[^ref-btc-core-mining-rpc]

### Block Propagation

찾아낸 block은 충분히 빨리 전파되어 수락되어야만 가치가 있다. 전파가 느리면 stale-block risk가 증가한다.

propagation risk는 다음에 영향을 준다.

- miner revenue
- pool connectivity
- transaction selection
- compact block 및 relay infrastructure
- 최신 tip 위에서 빠르게 채굴하려는 유인

---

## 6. Solo Mining과 Pooled Mining

### Solo Mining

solo mining은 채굴자가 독립적으로 block을 찾으려 하고, 찾은 block의 full subsidy와 fee를 모두 가져가는 방식이다. 분산이 크기 때문에 작은 채굴자는 기대보다 훨씬 오래 기다릴 수 있다.

Bitcoin Developer 문서는 solo miner가 `bitcoind`로 transaction을 수신하고, `getblocktemplate`을 polling하고, block을 구성하고, 80-byte header를 ASIC hardware에 보내고, 성공 시 full block을 제출하는 방식을 설명한다.[^ref-btc-dev-mining]

### Pooled Mining

pooled mining은 해시파워를 합친다. pool operator는 template과 payout accounting을 조정하고, 참여자는 share라 불리는 partial proof를 제출한다.

장점:

- 개별 hasher의 payout variance 감소
- 중앙화된 transaction selection과 template 관리
- 개별 hash owner에게 더 쉬운 운영

리스크:

- pool operator 중앙화
- pool operator의 template censorship
- hasher의 pool infrastructure 의존
- payout accounting 분쟁
- pool 수준의 규제 또는 운영 chokepoint

### Share

share는 miner가 pool이 정한 target을 만족하는 work를 수행했다는 증거다. pool target은 network target보다 더 쉽다.

```text
share target > network target
share hash may prove work to pool
share hash usually does not create a valid Bitcoin block
```

network target 이하인 header hash만 유효한 Bitcoin block을 만들 수 있다.

### Getblocktemplate와 Pooled Mining

BIP23은 BIP22를 pooled mining 방향으로 확장하며, pool extension, block proposal, mutation, submission abbreviation 같은 optional support를 추가한다.[^ref-bip-0023]

Developer 문서는 Stratum이 `getblocktemplate`의 대안으로 널리 사용되며, miner에게 block header 구성을 위한 최소 데이터만 제공해 pool-miner coordination의 bandwidth를 줄인다고 설명한다.[^ref-btc-dev-mining]

Stratum 세부 내용은 이 문서 범위를 벗어난다. 핵심은 pooled mining이 hash ownership과 transaction-template control을 분리할 수 있다는 점이다.

---

## 7. Mathematical or Economic Model

### 블록 발견 확률

miner가 네트워크 해시레이트의 비율 `h`를 가진다고 하자. 짧은 기간 동안 network condition이 안정적이라면:

```text
expected fraction of blocks found ~= h
```

이는 기대값이지 보장이 아니다. 실제 block discovery는 큰 분산을 가진 확률적 탐색을 따른다.

### 기대 시간

네트워크의 기대 block interval이 10분이고 miner 비중이 `h`라면:

```text
expected solo block interval ~= 10 minutes / h
```

예:

```text
h = 0.001 = 0.1%
expected interval ~= 10 / 0.001 = 10,000 minutes
                 ~= 6.94 days
```

분포의 변동성은 여전히 크다. expected time은 일정표가 아니다.

### Miner Revenue

찾은 block 하나에 대해:

```text
gross_revenue = subsidy + transaction_fees
net_revenue = gross_revenue - operating_costs
```

operating cost에는 전기, hardware depreciation, hosting, firmware/software operation, cooling, financing, pool fee가 포함된다.

### Pool Variance 감소

pool은 개별 기여자에게 revenue를 분산시켜 payout variance를 줄인다. 개념적으로는 다음과 같다.

```text
individual solo mining:
    high variance, full block reward when successful

pooled mining:
    lower variance, payout proportional to contributed shares
```

payout formula의 상세는 pool마다 다르며 이 문서 범위를 벗어난다.

### Fee Revenue와 Transaction Selection

miner의 transaction selection은 expected revenue 최대화를 지향한다.

```text
select transaction package if marginal_fee / marginal_weight is attractive
```

dependency가 있기 때문에 높은 fee의 child가 낮은 fee의 parent까지 포함할 유인을 만들 수 있다. 이는 CPFP와 cluster/chunk mempool logic으로 이어진다.

---

## 8. 보안 가정

### 채굴이 보호하는 것

채굴은 chain history를 다시 쓰는 비용을 높임으로써 Bitcoin을 보호한다. 더 많은 누적 작업량(Chainwork) 아래 묻힌 transaction은 되돌리기 더 비싸진다.

채굴은 다음에 기여한다.

- transaction ordering
- block production
- 누적 작업량을 통한 Sybil-resistant chain selection
- reorganization 시도에 대한 경제적 비용

### 채굴이 보호하지 않는 것

채굴은 다음을 보호하지 않는다.

- stolen private key
- exchange account compromise
- wallet malware
- bad custody procedure
- off-chain fraud
- incorrect address labeling
- invalid local mempool assumption

miner는 invalid block을 단순히 더 많이 hash한다고 valid하게 만들 수 없다. full node는 여전히 합의 규칙을 강제한다.[^ref-btc-core-validation]

### 채굴 중앙화 리스크

채굴은 여러 축에서 중앙화될 수 있다.

| Dimension | Risk |
|---|---|
| Pool template control | transaction censorship 또는 policy concentration |
| ASIC manufacturing | hardware supply bottleneck |
| Energy access | geographic and political concentration |
| Firmware/software | operational monoculture |
| Network connectivity | large actor의 propagation advantage |

이러한 리스크가 곧바로 합의 실패를 의미하는 것은 아니지만, censorship resistance, revenue distribution, attack surface에는 영향을 준다.

---

## 9. Bitcoin Core 구현

### Block Assembler

Bitcoin Core의 `node::BlockAssembler`는 valid proof-of-work 없이 새로운 block을 생성하는 것으로 문서화되어 있다. `CreateNewBlock` method는 새로운 block template을 구성한다.[^ref-btc-core-miner]

block assembler가 추적하는 중요한 상태는 다음과 같다.

- block weight
- signature operation cost
- number of transactions
- accumulated fees
- next height
- mempool pointer
- chainstate reference
- block creation options

### Coinbase와 Merkle 업데이트

Bitcoin Core mining source는 coinbase 및 Merkle-root update logic을 포함한다. `miner.cpp`에서 block construction은 coinbase-related commitment를 업데이트하고 `BlockMerkleRoot`를 사용해 `hashMerkleRoot`를 다시 계산한다.[^ref-btc-core-miner]

이 구현은 다음 개념 규칙과 일치한다.

```text
coinbase changes -> coinbase txid changes -> Merkle root changes -> header hash changes
```

### Mining Interface

Bitcoin Core는 interface를 통해 mining 기능을 노출한다. `interfaces/mining.h`에는 block template을 구성하는 `Mining::createNewBlock`과 관련 block creation/check/wait type이 포함된다.[^ref-btc-core-mining-iface]

Bitcoin Core 31.0 release notes는 IPC mining interface가 최신 `mining.capnp` schema를 요구하고, `Mining.createNewBlock`이 default cooldown behavior를 가지며, `BlockTemplate.getCoinbaseTx()`가 구조화된 `CoinbaseTx`를 반환한다고 설명한다.[^ref-btc-core-31-release]

### Mining RPC

Bitcoin Core RPC mining command는 다음을 포함한다.

- `getblocktemplate`
- `submitblock`
- `submitheader`
- `getmininginfo`
- `getnetworkhashps`
- `prioritisetransaction`
- `getprioritisedtransactions`

Bitcoin Core Doxygen의 `src/rpc/mining.cpp`는 이 mining RPC function들을 나열한다.[^ref-btc-core-mining-rpc]

### 작업증명 검증

Bitcoin Core의 작업증명 검증은 mining template construction 바깥에 있다. 채굴된 block은 validation code에서 검증된다. `CheckProofOfWork`와 contextual header validation은 block hash와 difficulty target이 chain context에 맞는지 확인한다.[^ref-btc-core-pow]

이 분리는 중요하다.

```text
miner constructs candidate
ASIC searches for hash
node validates full block
network accepts only valid blocks
```

---

## 10. Consensus, Policy, and Convention

### Consensus

채굴 관련 합의 규칙은 다음을 포함한다.

- block header가 작업증명 target을 만족해야 한다.
- `nBits`는 height와 parent context에 맞게 유효해야 한다.
- coinbase는 첫 번째 transaction이어야 한다.
- coinbase value는 subsidy + fee를 초과할 수 없다.
- block은 weight, sigops, transaction validity rule을 만족해야 한다.
- block timestamp는 합의 제약을 만족해야 한다.

### Mining Policy

mining policy는 다음을 포함한다.

- transaction selection
- fee-rate threshold
- template mutation choice
- block reserved weight
- private transaction inclusion
- nonstandard transaction inclusion
- pool-specific censorship or filtering

mining policy는 서로 달라도 consensus-valid block을 만들 수 있다.

### Pool Convention

pool convention은 다음을 포함한다.

- share difficulty
- payout formula
- miner worker naming
- coinbase tag
- job assignment format
- stale-share handling
- payout batching

이들은 Bitcoin 합의 규칙이 아니다.

---

## 11. 온체인 함의

### Strong Evidence

mining data는 다음을 강하게 뒷받침한다.

- block height와 timestamp
- block header field
- coinbase transaction 내용
- subsidy와 fee claim
- block에 포함된 transaction set
- block weight와 fee revenue
- 작업증명 target과 header hash
- block이 active chain의 일부가 되었는지 여부

### Weak Evidence

mining data는 다음에는 약한 근거만 제공한다.

- real-world miner identity
- 특정 시점의 exact pool hashrate
- pool 또는 hasher 중 누가 transaction을 골랐는지
- censorship intent
- private transaction source
- electricity cost 또는 profitability
- internal payout allocation

coinbase tag와 payout address는 유용한 근거지만, spoof, 공유, 변경, 중개 구조로 인해 해석에 주의가 필요하다.

### Stale Block과 Orphaned Block

같은 높이에서 두 유효 block이 경쟁하면, 더 많은 work를 쌓은 다른 branch 때문에 하나가 stale이 될 수 있다. stale block은 propagation race, network topology, 단순한 운을 시사할 수 있다.

분석가는 다음을 구분해야 한다.

- 경쟁에서 진 유효 block
- node가 거부한 invalid block
- never broadcast된 private candidate
- network target보다 높은 pool share

---

## 12. Institutional Thinking

### Investor and Research View

기관이 mining을 분석할 때는 다음을 추적해야 한다.

- hashprice와 fee share
- subsidy schedule
- difficulty adjustment
- estimated hash rate
- pool concentration
- block propagation quality
- transaction-selection behavior
- geographic and energy exposure
- 필요한 경우 public miner treasury와 debt structure

### Transaction Risk View

채굴은 다음을 통해 transaction settlement에 영향을 준다.

- fee 기반 inclusion probability
- block interval variance
- reorg risk
- censorship risk
- private relay와 direct-to-miner submission

mempool에 보인다는 사실은 settlement가 아니다. confirmation은 miner inclusion과 block acceptance에 달려 있다.

### Custody and Treasury View

treasury team이 mining을 이해해야 하는 이유:

- fee는 miner에게 지불된다.
- congestion은 withdrawal cost를 바꾼다.
- CPFP/RBF는 mining selection과 상호작용한다.
- confirmation policy는 확률적이다.
- coinbase output에는 maturity rule이 있다.
- mining pool attribution은 compliance monitoring에 영향을 줄 수 있다.

### Mining Operator View

mining operator는 다음을 관리해야 한다.

- template source reliability
- full-node validation
- pool connectivity
- stale rate
- firmware와 ASIC fleet operation
- payout address security
- regulatory and energy constraints
- mined BTC의 treasury management

---

## 13. Common Misinterpretations

### "채굴자만이 Bitcoin을 검증한다"

아니다. 채굴자는 block을 제안한다. full node가 block을 검증한다. validating node가 거부하는 invalid block을 hashpower만으로 consensus에 넣을 수는 없다.

### "채굴은 복잡한 수학 퍼즐을 푸는 것이다"

실제로는 candidate block header를 반복적으로 hashing해서 target 이하의 hash를 찾는 과정이다. shortcut이 있는 유일한 퍼즐을 푸는 것이 아니라 확률적 탐색이다.

### "Nonce가 전체 탐색 공간이다"

아니다. 채굴자는 coinbase extra nonce, Merkle root, timestamp, version field, transaction selection도 함께 바꾼다.

### "Pool Share는 Bitcoin Block이다"

아니다. share는 더 쉬운 target 아래에서 pool에 work를 증명할 뿐이다. network target도 만족하는 share만 유효한 Bitcoin block이 될 수 있다.

### "Coinbase Tag는 채굴자 신원을 증명한다"

아니다. coinbase tag는 유용하지만 암호학적 신원 증명이 아니다.

### "가장 수수료가 높은 transaction은 항상 포함된다"

아니다. dependency, policy, private transaction, template timing, miner preference 때문에 selection은 달라질 수 있다.

---

## 14. Research Questions

1. transaction selection power는 pool operator와 individual hasher 사이에 어떻게 분배되는가?
2. cluster mempool은 block template fee efficiency를 얼마나 바꾸는가?
3. pool attribution에 coinbase tag는 시간에 따라 얼마나 신뢰할 수 있는가?
4. 어떤 stale-rate pattern이 propagation advantage 또는 network issue를 시사하는가?
5. 높은 수요 구간 전후로 miner revenue의 fee share는 어떻게 변하는가?
6. private transaction inclusion을 시사하는 observable evidence는 무엇인가?
7. 기관은 pool, ASIC, energy, jurisdiction 차원에서 mining centralization을 어떻게 측정해야 하는가?

---

## 15. Practical Exercises

### Exercise 1: Block Inspection

최근 block 하나를 골라 다음을 기록하라.

- block hash
- previous block hash
- `nBits`
- nonce
- timestamp
- transaction count
- coinbase transaction
- total fees
- block weight

### Exercise 2: Coinbase Analysis

같은 block에 대해:

1. coinbase transaction을 decode한다.
2. coinbase output을 식별한다.
3. claimed value를 subsidy + fee와 비교한다.
4. coinbase tag 또는 pool marker를 기록한다.
5. attribution confidence를 표시한다.

### Exercise 3: Template Field

동기화된 node가 있다면 다음을 실행하라.

```bash
bitcoin-cli getblocktemplate '{"rules":["segwit"]}'
```

다음을 기록하라.

- previous block hash
- height
- bits
- target
- coinbase value
- transaction count
- weight limit
- mutable field

### Exercise 4: Share vs Block

왜 pool share는 payout accounting에는 유효하지만 Bitcoin block으로는 유효하지 않을 수 있는지 설명하라.

```text
share_target > network_target
hash <= share_target
hash > network_target
```

---

## 16. Evidence Classification

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Proof-of-work, incentives, chain history cost | A |
| REF-BTC-DEV-MINING-001 | Official developer documentation | Solo mining, pooled mining, getblocktemplate, Stratum overview, nonce and extra nonce | A |
| REF-BIP-0022 | API/RPC BIP | `getblocktemplate` and `submitblock` fundamentals | A |
| REF-BIP-0023 | API/RPC BIP | Pooled mining extensions to `getblocktemplate` | A |
| REF-BTC-CORE-MINER-001 | Primary implementation source | `BlockAssembler`, `CreateNewBlock`, coinbase/Merkle update logic | A |
| REF-BTC-CORE-MINING-IFACE-001 | Primary implementation source | Mining interface and block template creation APIs | A |
| REF-BTC-CORE-MINING-RPC-001 | Primary implementation/RPC source | Mining RPCs including `getblocktemplate`, `submitblock`, `submitheader` | A |
| REF-BTC-CORE-31-RELEASE-001 | Release documentation | Cluster mempool template effects and IPC mining interface changes | A |
| REF-BTC-CORE-POW-001 | Primary implementation source | Proof-of-work validation functions | A |
| REF-BTC-CORE-VALIDATION-001 | Primary implementation source | Full block validation boundary | A |

### Claim Ledger

| Claim | Classification | Evidence |
|---|---|---|
| mining은 candidate block을 구성하고 target 이하의 header hash를 찾는다 | FACT | Whitepaper, developer mining docs, Bitcoin Core PoW source |
| `BlockAssembler`는 valid proof-of-work 없이 block template을 만든다 | FACT | Bitcoin Core `BlockAssembler` documentation |
| coinbase 변경은 Merkle root를 바꾸고 mining search 공간을 확장할 수 있다 | FACT | Developer mining docs, Bitcoin Core miner source |
| solo mining은 pooled mining보다 payout variance가 높다 | FACT | Bitcoin Developer mining guide |
| pool share는 반드시 유효한 Bitcoin block이 아니다 | FACT | Developer mining guide, BIP23 target/share model |
| `getblocktemplate`은 block construction data를 전달한다 | FACT | BIP22 and Bitcoin Core RPC docs |
| mining policy는 relay policy와 다를 수 있다 | FACT | Bitcoin Core policy/mining boundary and BIP22 rationale |
| hashpower만 있으면 invalid block도 valid하게 만들 수 있다 | COUNTERCLAIM | Rejected; full nodes enforce validation |
| coinbase tag는 real-world miner identity를 증명한다 | COUNTERCLAIM | Rejected; tags are non-consensus metadata |
| miner transaction selection은 항상 public mempool fee order를 정확히 따른다 | COUNTERCLAIM | Rejected; private orderflow, policy, timing can differ |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| POLICY | Mining, pool, relay, or operational convention rather than consensus |
| HEURISTIC | Practical inference with known counterexamples |
| UNKNOWN | Evidence is insufficient |

---

## 17. Knowledge Graph

```text
BITCOIN-020 Mining
|
+-- builds_on: BITCOIN-017 Mempool
+-- builds_on: BITCOIN-018 Transaction Fees
+-- builds_on: POW-009 Coinbase Transaction
+-- builds_on: POW-012 Difficulty Adjustment
+-- builds_on: POW-013 Bitcoin Core PoW Validation
|
+-- mining workflow
|   +-- select_transactions
|   +-- build_coinbase
|   +-- compute_merkle_root
|   +-- hash_header
|   +-- submit_block
|
+-- economic incentive
|   +-- subsidy
|   +-- transaction_fees
|   +-- operating_costs
|
+-- coordination
|   +-- solo_mining
|   +-- pooled_mining
|   +-- shares
|   +-- getblocktemplate
|
+-- Bitcoin Core
|   +-- BlockAssembler
|   +-- CreateNewBlock
|   +-- mining RPCs
|   +-- mining interface
|   +-- validation boundary
|
+-- analysis
    +-- facts: block contents, fees, coinbase
    +-- heuristics: miner identity, pool attribution
    +-- risks: stale blocks, censorship, centralization
```

---

## 18. 참고문헌

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Sections 4-6, proof-of-work and incentive design, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.
[^ref-btc-dev-mining]: Bitcoin Developer Documentation, "Mining," solo mining, pooled mining, getblocktemplate, Stratum overview, nonce and extra nonce behavior, https://developer.bitcoin.org/devguide/mining.html, accessed 2026-08-04.
[^ref-bip-0022]: Luke Dashjr, "BIP 22: getblocktemplate - Fundamentals," 2012-02-28, https://bips.dev/22/ and https://github.com/bitcoin/bips/blob/master/bip-0022.mediawiki, accessed 2026-08-04.
[^ref-bip-0023]: Luke Dashjr, "BIP 23: getblocktemplate - Pooled Mining," 2012-02-28, https://bips.xyz/23, accessed 2026-08-04.
[^ref-btc-core-miner]: Bitcoin Core Contributors, `src/node/miner.h` and `src/node/miner.cpp`, `BlockAssembler`, `CreateNewBlock`, coinbase and Merkle-root update logic, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/classnode_1_1_block_assembler.html and https://doxygen.bitcoincore.org/miner_8cpp_source.html, accessed 2026-08-04.
[^ref-btc-core-mining-iface]: Bitcoin Core Contributors, `src/interfaces/mining.h`, mining interface and block template APIs, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/interfaces_2mining_8h_source.html, accessed 2026-08-04.
[^ref-btc-core-mining-rpc]: Bitcoin Core Contributors, `src/rpc/mining.cpp` and Bitcoin Core mining RPC documentation, mining RPCs, `getblocktemplate`, `submitblock`, and `submitheader`, Bitcoin Core Doxygen 31.99.0 documentation and RPC docs, https://doxygen.bitcoincore.org/rpc_2mining_8cpp.html and https://bitcoincore.org/en/doc/26.0.0/rpc/mining/getblocktemplate/, accessed 2026-08-04.
[^ref-btc-core-31-release]: Bitcoin Core Contributors, "Bitcoin Core 31.0 Release Notes," cluster mempool and IPC mining interface changes, https://bitcoin.org/en/releases/31.0/ and https://bitcoincore.org/en/releases/31.0/, accessed 2026-08-04.
[^ref-btc-core-pow]: Bitcoin Core Contributors, `src/pow.cpp` and `src/pow.h`, proof-of-work target and hash validation, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/pow_8cpp_source.html and https://doxygen.bitcoincore.org/pow_8h_source.html, accessed 2026-08-04.
[^ref-btc-core-validation]: Bitcoin Core Contributors, `src/validation.cpp`, block and header validation boundaries, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/validation_8cpp.html, accessed 2026-08-04.

---

## 19. 교차 참조

### Parent

- Bitcoin Internals

### Previous

- BITCOIN-019 — Wallets and Key Management

### Next

- BITCOIN-021 — Blocks and Block Headers

### Related

- BITCOIN-007 — Whitepaper Section 6 — Incentive
- BITCOIN-017 — Mempool
- BITCOIN-018 — Transaction Fees
- POW-009 — Coinbase Transaction and Mining Commitments
- POW-011 — Cumulative Chainwork
- POW-012 — Difficulty Adjustment
- POW-013 — Bitcoin Core Proof-of-Work Validation
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- block construction과 proof-of-work search를 분리했다.
- solo mining, pooled mining, pool share, coinbase reward, transaction selection, block submission을 구분했다.
- network target과 pool share target을 분리했다.
- Bitcoin Core `BlockAssembler`, mining interface, mining RPC, PoW validation, full block validation을 올바른 경계에 배치했다.
- 현재 Bitcoin Core 31.0 mining IPC 및 cluster mempool template-selection 맥락을 consensus와 혼동하지 않도록 포함했다.

### Evidence Review

Passed.

- mining workflow 관련 설명은 Bitcoin Developer 문서, BIP22/BIP23, Bitcoin Core source 문서에 연결했다.
- 작업증명과 incentive 관련 설명은 백서와 Bitcoin Core PoW validation reference에 연결했다.
- 현재 구현 관련 설명은 Bitcoin Core Doxygen과 31.0 release note에 연결했다.
- pool identity, coinbase tag, transaction-selection 관련 설명은 heuristic 한계를 표시했다.

### Editorial Review

Passed.

- Markdown heading은 프로젝트 deep-dive 구조를 따른다.
- metadata가 완전하다.
- table과 code fence가 닫혀 있다.
- 용어는 mining, candidate block, block template, coinbase, nonce, extra nonce, share, pool, target으로 일관된다.

### Adversarial Review

Passed.

- 채굴자만이 consensus를 정한다고 주장하지 않았다.
- share를 Bitcoin block과 동일시하지 않았다.
- coinbase tag를 검증된 miner identity로 취급하지 않았다.
- transaction selection이 항상 public mempool fee ordering을 정확히 따른다고 주장하지 않았다.
- mining policy, pool convention, consensus rule을 혼동하지 않았다.

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
