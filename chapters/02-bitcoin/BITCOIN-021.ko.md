---
knowledge_id: BITCOIN-021
title: Blocks and Block Headers
subtitle: 직렬화된 블록, 80바이트 헤더, 머클 루트, 위트니스 커밋먼트, weight 한도, 그리고 검증 경계
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 105 min
estimated_study: 300 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Blocks
  - Block Headers
  - Consensus
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-015
  - BITCOIN-016
  - BITCOIN-020
  - POW-011
  - POW-013
related_topics:
  - Block Header
  - Merkle Root
  - Witness Commitment
  - Proof of Work
  - Block Weight
  - Coinbase Transaction
  - Chainwork
  - SPV
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-BLOCKCHAIN-001
  - REF-BIP-0141
  - REF-BTC-CORE-BLOCK-001
  - REF-BTC-CORE-MERKLE-001
  - REF-BTC-CORE-CONSENSUS-VALIDATION-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-CHAIN-001
  - REF-BTC-CORE-POW-001
tags:
  - bitcoin
  - internals
  - blocks
  - block-headers
  - merkle-root
  - witness-commitment
  - proof-of-work
  - validation
---

# Blocks and Block Headers
> Bitcoin Internals  
> Research Unit: BITCOIN-021

---

## Research Brief

```yaml
knowledge_id: BITCOIN-021
title: Blocks and Block Headers
research_question: >
  How are Bitcoin blocks and 80-byte block headers structured, what do the
  header fields commit to, how do Merkle roots and witness commitments bind
  transactions to blocks, and how does Bitcoin Core separate header checks,
  block checks, proof-of-work validation, and contextual validation?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-015
  - BITCOIN-016
  - BITCOIN-020
  - POW-011
  - POW-013
parent: Bitcoin Internals
previous: BITCOIN-020
next: BITCOIN-022
related_topics:
  - Transactions
  - Mining
  - Merkle Trees
  - Proof of Work
  - Chain Selection
  - SPV
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
  - Full P2P block relay protocol
  - Full AssumeUTXO or pruning internals
  - Complete compact block relay mechanics
  - Full SPV client implementation
  - Mining hardware engineering
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Bitcoin block의 직렬화 구조를 설명할 수 있다.
- 80-byte block header의 여섯 필드를 나열하고 설명할 수 있다.
- block hash가 header의 double-SHA256 hash인 이유를 설명할 수 있다.
- previous-block hash가 block을 chain으로 연결하는 방식을 설명할 수 있다.
- transaction Merkle root가 transaction ordering과 content에 어떻게 커밋하는지 설명할 수 있다.
- SegWit witness commitment가 legacy transaction Merkle root와 어떻게 다른지 설명할 수 있다.
- block height가 왜 직렬화된 header field가 아닌지 설명할 수 있다.
- header validation과 full block validation을 구분할 수 있다.
- Bitcoin Core의 block, Merkle, weight, validation source area를 식별할 수 있다.
- timestamp, version bits, coinbase metadata를 과도하게 해석하지 않을 수 있다.

---

## 2. 핵심 질문

1. Bitcoin block이란 무엇인가?
2. block header란 무엇인가?
3. 왜 header는 80 byte인가?
4. 작업증명을 위해 어떤 header field가 hashing되는가?
5. `hashPrevBlock`은 무엇에 커밋하는가?
6. `hashMerkleRoot`는 무엇에 커밋하는가?
7. `nBits`는 무엇을 인코딩하는가?
8. `nNonce`는 어떤 역할을 하는가?
9. block height는 어디에 저장되는가?
10. 왜 SegWit는 witness commitment가 필요한가?
11. full transaction 없이 header만으로 무엇을 증명할 수 있는가?
12. Bitcoin Core는 header와 block을 어떻게 검증하는가?

---

## 3. Executive Summary

Bitcoin block은 80-byte header와 transaction list로 구성된다. header는 채굴자가 작업증명을 위해 hash하는 compact commitment다. 백서는 체인을 해시로 연결된 timestamped block의 집합으로 제시하고, Bitcoin Developer 문서는 header field를 version, previous block header hash, Merkle root hash, time, `nBits`, nonce로 설명한다.[^ref-btc-wp] [^ref-btc-dev-blockchain]

header 구조는 다음과 같다.

```text
4 bytes   nVersion
32 bytes  hashPrevBlock
32 bytes  hashMerkleRoot
4 bytes   nTime
4 bytes   nBits
4 bytes   nNonce
= 80 bytes
```

block hash는 전체 block의 hash가 아니라 serialized header의 hash다. Bitcoin Core의 `CBlockHeader::GetHash()`는 header object를 hashing하고, `CBlock`은 `CBlockHeader`를 확장해 transaction vector `vtx`를 더한다.[^ref-btc-core-block]

transaction Merkle root는 순서가 있는 transaction list를 header에 커밋한다. Bitcoin Core의 `BlockMerkleRoot`는 leaf를 transaction `txid`에서 만들고 Merkle root를 계산한다.[^ref-btc-core-merkle] SegWit는 witness Merkle root와 witness reserved value를 coinbase transaction에 넣는 별도의 witness commitment를 추가해 `wtxid`에 커밋한다.[^ref-bip-0141]

header validation과 full block validation은 다르다. 유효한 작업증명 header라도 transaction, coinbase value, Merkle root, witness commitment, contextual rule이 틀리면 invalid block에 속할 수 있다. Bitcoin Core는 `CheckBlockHeader`, `CheckMerkleRoot`, `CheckBlock`, `ContextualCheckBlockHeader`, `ContextualCheckBlock` 같은 검사를 분리한다.[^ref-btc-core-validation]

분석가에게 header는 chain linkage, 작업증명, time claim, target encoding, transaction commitment에 대한 compact evidence를 제공한다. 그러나 이것만으로 모든 transaction validity나 real-world miner identity를 드러내지는 않는다.

---

## 4. 프로토콜 구조

### Serialized Block

serialized block은 다음을 포함한다.

```text
block_header
transaction_count
transactions
```

Bitcoin Developer 문서는 serialized block이 80-byte block header, CompactSize transaction count, 그리고 Merkle tree에 사용된 순서대로 배치된 raw transaction으로 구성된다고 설명한다.[^ref-btc-dev-blockchain]

첫 transaction은 반드시 coinbase transaction이어야 한다. non-coinbase transaction은 기존 output을 소비한다. block의 transaction set 전체가 유효해야 한다.

### Block Header Field

| Field | Size | Meaning |
|---|---:|---|
| `nVersion` | 4 bytes | block validation rule 및 deployment signaling용 version field |
| `hashPrevBlock` | 32 bytes | previous block header의 hash |
| `hashMerkleRoot` | 32 bytes | 순서가 있는 transaction `txid`의 Merkle root |
| `nTime` | 4 bytes | miner가 선언한 Unix epoch timestamp |
| `nBits` | 4 bytes | 작업증명 target의 compact encoding |
| `nNonce` | 4 bytes | miner가 바꾸는 header search field |

Bitcoin Core의 `CBlockHeader`는 정확히 이 field를 이 순서대로 저장하고 직렬화한다.[^ref-btc-core-block]

### Block Hash

block hash는 다음과 같다.

```text
block_hash = double_sha256(serialized_block_header)
```

transaction list 전체가 직접 hash되는 것은 아니므로, transaction commitment는 `hashMerkleRoot`를 통해 block hash에 반영된다.

### Previous-Block Link

`hashPrevBlock`은 각 block을 parent에 연결한다. 과거 block을 바꾸면 그 block의 header hash가 바뀌고, child가 기대하는 `hashPrevBlock`도 바뀌므로, 그 지점부터 작업증명을 다시 하지 않으면 chain이 끊어진다.

이것이 Bitcoin의 append-only chain of work의 구조적 기반이다.

### Block Height

block height는 직렬화된 block-header field가 아니다. height는 genesis부터 parent link를 따라가며 결정되는 contextual value다.

Bitcoin Core의 `CBlockIndex`는 header와 chain context에서 파생된 chain-index metadata를 저장하며, 여기에는 header에서 복사한 field와 parent linkage도 포함된다.[^ref-btc-core-chain]

현대 block에서는 BIP34에 따라 coinbase transaction에 block height가 들어가야 하지만, 이는 transaction data이지 header field가 아니다.

---

## 5. Merkle Root와 Commitment

### Transaction Merkle Root

transaction Merkle root는 모든 transaction `txid`에 순서대로 커밋한다.

```text
txids -> pairwise double-SHA256 hashing -> Merkle root -> block header
```

Bitcoin Developer 문서는 transaction ID를 짝지어 결합하고 double-SHA256한 뒤 상위 row를 만들고, 어떤 level에서 hash 개수가 홀수이면 마지막 hash를 복제한다고 설명한다.[^ref-btc-dev-blockchain]

### Mutation Caveat

Bitcoin Core의 `merkle.cpp`는 CVE-2012-2459와 관련된 historical duplicate-txid Merkle-tree flaw를 경고한다. 구현은 row 끝에서 동일한 hash가 짝지어지는 경우를 감지하고, 그러한 mutation case를 invalid로 처리한다.[^ref-btc-core-merkle]

분석상 중요한 점:

```text
Merkle root equality alone is not a full block-validity proof
```

이는 Bitcoin의 정확한 Merkle algorithm과 validation check 맥락에서 평가되어야 한다.

### Witness Commitment

SegWit는 witness transaction ID와 witness commitment를 도입했다. BIP141는 witness commitment를 coinbase transaction의 `OP_RETURN` output으로 정의하며, commitment hash는 다음과 같다.

```text
Double-SHA256(witness root hash | witness reserved value)
```

coinbase transaction의 `wtxid`는 witness root 계산에서 zero로 취급된다. 여러 coinbase output이 commitment pattern에 맞으면 가장 높은 output index가 사용된다. witness data가 있는 transaction이 하나도 없으면 witness commitment는 optional이다.[^ref-bip-0141]

### 왜 Header Merkle Root를 바꾸지 않았는가

SegWit는 legacy transaction Merkle root와 `txid` commitment를 유지하면서, coinbase를 통해 별도의 witness commitment를 추가했다. 이로써 80-byte block-header 구조를 바꾸지 않고 witness data에 커밋할 수 있었다.

이 구분은 다음에 중요하다.

- `txid` 기반 SPV-style proof
- full node의 witness validation
- transaction malleability 분석
- block parsing과 historical compatibility

---

## 6. Technical Mechanics

### Header Hashing과 작업증명

채굴은 80-byte header를 반복 hashing한다.

```text
hash = SHA256(SHA256(header))
valid if hash <= target
```

`nBits`는 target을 인코딩한다. Bitcoin Core는 target을 파생한 뒤 header hash와 비교해 작업증명을 검증한다.[^ref-btc-core-pow]

### Time Field

block timestamp는 miner가 선언한다. Bitcoin Developer 문서는 이 값이 이전 11개 block의 median time보다 커야 하며, full node는 자기 시계 기준 2시간보다 미래인 header를 거부한다고 설명한다.[^ref-btc-dev-blockchain]

timestamp 관련 주의:

```text
nTime is not a precise wall-clock timestamp
```

이는 합의 제약을 받는 miner declaration이다.

### Version Field

`nVersion`은 역사적으로 block version이었고, 나중에는 soft fork deployment/signaling surface가 되었다. deployment context를 확인하지 않고 version bits만으로 miner identity나 policy preference를 넓게 추론해서는 안 된다.

### Nonce와 Search Space

`nNonce`는 32-bit뿐이다. 채굴자가 nonce를 모두 소진하면 다른 header-affecting field를 바꿀 수 있다.

- coinbase extra nonce
- transaction ordering
- selected transaction
- timestamp
- 허용 범위 안의 version bits

이러한 변경은 Merkle root나 header를 바꾸어 새로운 hash trial을 만든다.

### Block Weight

SegWit는 weight accounting을 도입했다. Bitcoin Core의 `GetBlockWeight`는 witness 포함/미포함 serialization을 바탕으로 block weight를 계산하며, 개념적으로는 다음과 같다.

```text
weight = stripped_size * 3 + total_size
```

이는 `src/consensus/validation.h`에 transaction weight 계산과 함께 정의되어 있다.[^ref-btc-core-consensus-validation]

---

## 7. Validation Boundaries

### Header-Only Check

header check는 다음을 평가할 수 있다.

- header structure
- proof-of-work hash against target
- compact target validity
- parent가 알려져 있다면 previous-block linkage
- contextual difficulty transition
- timestamp constraint

header에는 full transaction data가 없기 때문에, header validation만으로는 transaction validity를 증명할 수 없다.

### Block-Level Check

block check는 다음을 평가할 수 있다.

- transaction count
- coinbase placement
- Merkle root
- witness commitment
- block weight
- transaction-level validity
- script validity
- UTXO spend
- coinbase value

Bitcoin Core의 `CheckBlock`은 context-independent block validation으로 문서화되어 있으며, contextual check는 별도로 처리된다.[^ref-btc-core-validation]

### Contextual Check

contextual check는 parent와 chain state에 의존한다.

- expected difficulty bits
- median-time-past
- height-dependent deployment
- coinbase height rule
- height/time relative finality
- connecting 시점의 UTXO availability

이 때문에 chain context 없이 block byte만으로 full validity를 판단할 수는 없다.

### Header가 유효하다고 해서 Block도 유효한 것은 아니다

block은 header hash가 유효해도 다음 이유로 실패할 수 있다.

- Merkle root가 transaction과 일치하지 않음
- witness commitment가 invalid
- coinbase가 reward를 과다 청구
- transaction이 없는 output이나 이미 spent된 output을 소비
- script 실패
- block이 weight limit 초과
- 해당 height의 difficulty bits가 잘못됨

---

## 8. Mathematical or Economic Model

### Header Commitment Model

```text
H = hash(version, prev_hash, merkle_root, time, bits, nonce)
```

header는 직접적으로 다음에 커밋한다.

- parent block
- ordered transaction `txid` Merkle root
- miner-declared time
- 작업증명 target encoding
- nonce

간접적으로는 다음에 커밋한다.

- `txid`를 통한 transaction content
- coinbase witness commitment를 통한 witness data
- `prev_hash`를 통한 chain history

### Chain Work

각 유효 block은 자신의 target으로 결정되는 work를 기여한다. chain selection은 단순 block count가 아니라 누적된 work를 사용한다.

개념적으로는:

```text
chainwork = sum(work represented by each valid block target)
```

정확한 chainwork mechanics는 POW-011에서 다룬다.

### Merkle Proof Size

transaction 수가 `n`인 block에서 하나의 transaction에 대한 Merkle proof는 대략 다음 수의 sibling hash를 요구한다.

```text
ceil(log2(n)) sibling hashes
```

이 logarithmic proof size 때문에 block header와 Merkle path는 simplified verification에 유용하지만, SPV는 모든 합의 규칙을 검증하지는 못한다.

### Mutation Detection

Bitcoin의 Merkle algorithm은 홀수 level에서 마지막 hash를 복제한다. Bitcoin Core의 mutation detection은 알려진 duplicate-tail ambiguity가 유효한 Merkle root mutation으로 수용되는 것을 막는다.[^ref-btc-core-merkle]

---

## 9. Bitcoin Core 구현

### Block Primitive

Bitcoin Core의 `src/primitives/block.h`는 다음을 정의한다.

| Type | Role |
|---|---|
| `CBlockHeader` | header field와 header hash interface |
| `CBlock` | header + transaction vector `vtx` |
| `CBlockLocator` | peer synchronization용 locator structure |

`CBlockHeader`는 `nVersion`, `hashPrevBlock`, `hashMerkleRoot`, `nTime`, `nBits`, `nNonce`를 포함한다. `CBlock`은 `CBlockHeader`를 상속하고 `vtx`와 validation-cache flag를 추가한다.[^ref-btc-core-block]

### Merkle Function

Bitcoin Core의 `src/consensus/merkle.cpp`는 다음을 정의한다.

| Function | Role |
|---|---|
| `ComputeMerkleRoot` | hash leaf에서 Merkle root를 계산하고 mutation을 탐지 |
| `BlockMerkleRoot` | `txid`로부터 transaction Merkle root 계산 |
| `BlockWitnessMerkleRoot` | `wtxid`로부터 witness Merkle root 계산 |
| `TransactionMerklePath` | transaction까지의 Merkle path 계산 |

이 함수들은 block validity가 정확한 root calculation에 의존하기 때문에 consensus-sensitive하다.[^ref-btc-core-merkle]

### Weight Function

Bitcoin Core의 `src/consensus/validation.h`는 `GetTransactionWeight`, `GetBlockWeight`, `GetTransactionInputWeight`를 정의하며, witness/non-witness serialization을 사용해 SegWit weight accounting을 구현한다.[^ref-btc-core-consensus-validation]

### Validation Function

Bitcoin Core validation 문서는 다음 function을 나열한다.

| Function | Role |
|---|---|
| `CheckBlockHeader` | header validity check |
| `CheckMerkleRoot` | transaction Merkle root check |
| `CheckWitnessMalleation` | witness commitment/malleation check |
| `CheckBlock` | context-independent block check |
| `ContextualCheckBlockHeader` | context-dependent header check |
| `ContextualCheckBlock` | context-dependent block check |
| `TestBlockValidity` | current tip 기준으로 transaction을 포함한 block validity 검증 |

이 경계 구분은 header proof-of-work를 full block validity와 혼동하지 않게 한다.[^ref-btc-core-validation]

### Block Index

Bitcoin Core의 `CBlockIndex`는 알려진 block header와 chain selection을 위한 metadata를 저장하며, header field, parent pointer, status, file position, work-related state를 포함한다.[^ref-btc-core-chain]

이는 block data와 validation context에서 파생된 node-local index state다. block 내부에 직렬화되는 것은 아니다.

---

## 10. Consensus, Policy, and Presentation

### Consensus

합의 규칙은 block이 active chain의 일부가 될 수 있는지 결정한다.

예:

- 작업증명 target 만족
- 올바른 difficulty bits
- 유효한 Merkle root
- 필요한 경우 유효한 witness commitment
- 유효한 transaction list
- 유효한 coinbase transaction과 reward
- limit 이내의 block weight
- timestamp constraint

### Policy

정책은 block이 존재하기 전 채굴자가 어떤 transaction을 포함할지에 영향을 준다. 일단 유효한 block이 발견되면 full node는 자신의 mempool이 해당 transaction을 모두 relay했는지가 아니라 합의 규칙을 본다.

### Presentation

explorer와 RPC는 다음을 표시한다.

- block hash
- height
- confirmations
- version
- Merkle root
- time
- bits
- nonce
- transaction count
- size와 weight

height와 confirmation은 contextual display value이지 serialized header field가 아니다.

---

## 11. 온체인 함의

### Strong Evidence

block과 header data는 다음을 강하게 뒷받침한다.

- parent-child chain linkage
- 작업증명 target claim
- header hash
- transaction inclusion commitment
- 합의 제약 범위 안의 timestamp claim
- coinbase transaction 내용
- block weight
- validated node에서 관측된 active-chain inclusion

### Weak Evidence

block data는 다음에는 약한 근거만 제공한다.

- exact creation time
- real-world miner identity
- transaction이 public mempool에서 왔는지 private submission인지
- miner intent
- pool operator와 hasher 중 누가 어떤 행동을 했는지
- transaction ordering의 경제적 동기

### SPV 경계

header와 Merkle proof는 transaction이 작업증명이 있는 block header에 커밋되었다는 것을 보여 줄 수 있다. 그러나 이것만으로 full block validity, UTXO validity, script correctness까지 독립적으로 증명하지는 못한다.

이것이 simplified payment verification의 핵심 한계다.

---

## 12. Institutional Thinking

### Settlement

기관은 settlement를 평가할 때 다음을 봐야 한다.

- active-chain inclusion
- confirmation depth
- cumulative chainwork
- reorg risk
- full node가 검증한 transaction validity
- counterparty와 operational context

header count만 보는 것은 validated chainwork context보다 약하다.

### Data Engineering

block indexing system은 다음을 저장하는 것이 좋다.

- raw header field
- block hash
- contextual index로서의 height
- transaction count
- transaction ID
- 존재하는 경우 witness commitment
- block weight와 size
- trusted node의 validation status
- reorg 중 chain status

### Monitoring

운영 모니터는 다음을 경고해야 한다.

- 같은 height의 competing block
- 예상 밖 reorg
- 비정상 timestamp
- coinbase overclaim rejection
- fee-share가 높은 block
- 누락되었거나 잘못된 witness commitment
- block propagation delay

### Analytics

분석가는 block metadata를 조심해서 다뤄야 한다. timestamp는 miner declaration이다. coinbase tag는 identity signature가 아니다. block 안에서의 transaction 위치는 dependency, fee strategy, private submission, template construction logic을 반영할 수 있다.

---

## 13. Common Misinterpretations

### "Block Hash는 전체 Block을 직접 Hash한다"

아니다. block hash는 header의 hash다. transaction은 Merkle root를 통해, witness data는 SegWit witness commitment를 통해 커밋된다.

### "Block Height는 Header에 있다"

아니다. height는 contextual하다. 현대 coinbase transaction에는 height가 들어가지만, header에는 없다.

### "유효한 Header면 유효한 Block이다"

아니다. header가 작업증명을 만족해도 full block은 transaction, Merkle, witness, contextual validation에서 실패할 수 있다.

### "Timestamp는 정확하다"

아니다. 이는 합의 제약을 받는 miner-declared value다.

### "Merkle Proof면 Transaction Validity까지 증명된다"

아니다. Merkle proof는 header commitment 아래의 inclusion을 보여 줄 뿐, 모든 validation rule을 증명하지는 않는다.

---

## 14. Research Questions

1. block timestamp는 실제 network arrival time과 얼마나 자주 유의미하게 차이 나는가?
2. cluster mempool 이후 block template transaction-order inference는 얼마나 신뢰할 수 있는가?
3. malformed witness commitment 시도를 잡아내는 monitoring은 무엇인가?
4. 기관은 confirmation depth와 chainwork를 어떻게 함께 표현해야 하는가?
5. stale block은 fee 및 miner attribution 분석에 어떤 영향을 주는가?
6. reorg history를 정확히 재구성하려면 어떤 metadata가 필요한가?
7. invalid-block 또는 eclipse scenario에서 SPV 가정은 어떻게 실패하는가?

---

## 15. Practical Exercises

### Exercise 1: Header Decode

최근 block 하나를 골라 다음을 기록하라.

- `version`
- `previousblockhash`
- `merkleroot`
- `time`
- `bits`
- `nonce`
- block hash

어떤 field가 header에 직렬화되고, 어떤 값이 contextual한지 설명하라.

### Exercise 2: Merkle Root 재계산

block의 ordered transaction ID를 사용해:

1. `txid`를 block order대로 놓는다.
2. 짝지어 double-SHA256한다.
3. 홀수 level에서는 마지막 hash를 복제한다.
4. 하나의 root가 남을 때까지 반복한다.
5. header Merkle root와 비교한다.

### Exercise 3: Witness Commitment

SegWit block에 대해:

1. coinbase transaction을 찾는다.
2. witness commitment output을 찾는다.
3. commitment prefix를 식별한다.
4. witness commitment가 왜 block header에 직접 저장되지 않는지 설명한다.

### Exercise 4: Header vs Block Validation

각 검사를 분류하라.

| Check | Header | Full block | Contextual |
|---|---:|---:|---:|
| Header hash below target | Yes | No | Maybe |
| Merkle root matches transaction list | No | Yes | No |
| Coinbase value does not overclaim | No | Yes | Yes |
| `nBits` correct for height | Yes | No | Yes |
| Transaction script validation | No | Yes | Yes |
| Block height display | No | No | Yes |

---

## 16. Evidence Classification

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Blocks, proof-of-work chain, Merkle hash tree concept | A |
| REF-BTC-DEV-BLOCKCHAIN-001 | Official developer documentation | Block header fields, serialized block format, Merkle tree, `nBits`, timestamp rules | A |
| REF-BIP-0141 | Consensus BIP | Witness commitment, witness Merkle root, block weight and SegWit commitments | A |
| REF-BTC-CORE-BLOCK-001 | Primary implementation source | `CBlockHeader`, `CBlock`, header serialization fields, `GetHash` | A |
| REF-BTC-CORE-MERKLE-001 | Primary implementation source | `ComputeMerkleRoot`, `BlockMerkleRoot`, `BlockWitnessMerkleRoot`, mutation detection | A |
| REF-BTC-CORE-CONSENSUS-VALIDATION-001 | Primary implementation source | `GetBlockWeight`, transaction/block weight accounting | A |
| REF-BTC-CORE-VALIDATION-001 | Primary implementation source | `CheckBlockHeader`, `CheckBlock`, Merkle/witness/contextual checks | A |
| REF-BTC-CORE-CHAIN-001 | Primary implementation source | `CBlockIndex`, contextual chain-index metadata | A |
| REF-BTC-CORE-POW-001 | Primary implementation source | Proof-of-work target/hash validation | A |

### Claim Ledger

| Claim | Classification | Evidence |
|---|---|---|
| Bitcoin block header는 6개 field로 구성된 80 byte다 | FACT | Bitcoin Developer docs, Bitcoin Core `block.h` |
| `CBlock`은 `CBlockHeader`를 확장해 transaction vector `vtx`를 더한다 | FACT | Bitcoin Core `block.h` |
| block hash는 serialized header의 hash다 | FACT | Bitcoin Core `CBlockHeader::GetHash` |
| transaction Merkle root는 ordered transaction ID를 header에 커밋한다 | FACT | Bitcoin Developer docs, Bitcoin Core `merkle.cpp` |
| SegWit witness data는 coinbase witness commitment를 통해 커밋된다 | FACT | BIP141 |
| block height는 contextual하며 header field가 아니다 | FACT | Header format, Bitcoin Core `CBlockIndex` |
| valid proof-of-work header가 full block validity를 증명한다 | COUNTERCLAIM | Rejected; full validation checks more than header PoW |
| timestamp는 exact wall-clock creation time이다 | COUNTERCLAIM | Rejected; timestamp is miner-declared under constraints |
| Merkle proof만으로 UTXO/script validity가 증명된다 | COUNTERCLAIM | Rejected; commitment inclusion only, not full validation |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| POLICY | Node, miner, or relay convention rather than consensus |
| HEURISTIC | Practical inference with known counterexamples |
| UNKNOWN | Evidence is insufficient |

---

## 17. Knowledge Graph

```text
BITCOIN-021 Blocks and Block Headers
|
+-- builds_on: BITCOIN-020 Mining
+-- builds_on: POW-011 Cumulative Chainwork
+-- builds_on: POW-013 Bitcoin Core PoW Validation
|
+-- block
|   +-- contains: header
|   +-- contains: transactions
|
+-- header
|   +-- nVersion
|   +-- hashPrevBlock
|   +-- hashMerkleRoot
|   +-- nTime
|   +-- nBits
|   +-- nNonce
|
+-- commitments
|   +-- txid_merkle_root
|   +-- witness_commitment
|   +-- previous_block_hash
|
+-- Bitcoin Core
|   +-- CBlockHeader
|   +-- CBlock
|   +-- ComputeMerkleRoot
|   +-- CheckBlockHeader
|   +-- CheckBlock
|   +-- CBlockIndex
|
+-- analysis
    +-- facts: header fields, block hash, Merkle root
    +-- caveats: timestamp, miner identity, SPV limits
```

---

## 18. 참고문헌

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Sections 3-5, timestamp server, proof-of-work, and Merkle tree pruning context, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.
[^ref-btc-dev-blockchain]: Bitcoin Developer Documentation, "Block Chain," block headers, serialized blocks, Merkle trees, `nBits`, and timestamp constraints, https://developer.bitcoin.org/reference/block_chain.html, accessed 2026-08-04.
[^ref-bip-0141]: Eric Lombrozo, Johnson Lau, and Pieter Wuille, "BIP 141: Segregated Witness (Consensus layer)," witness commitment and block weight rules, 2015-12-21, https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki, accessed 2026-08-04.
[^ref-btc-core-block]: Bitcoin Core Contributors, `src/primitives/block.h` and `src/primitives/block.cpp`, `CBlockHeader`, `CBlock`, header serialization, and `GetHash`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/block_8h_source.html and https://doxygen.bitcoincore.org/class_c_block_header.html, accessed 2026-08-04.
[^ref-btc-core-merkle]: Bitcoin Core Contributors, `src/consensus/merkle.cpp`, `ComputeMerkleRoot`, `BlockMerkleRoot`, `BlockWitnessMerkleRoot`, and mutation detection, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/consensus_2merkle_8cpp_source.html, accessed 2026-08-04.
[^ref-btc-core-consensus-validation]: Bitcoin Core Contributors, `src/consensus/validation.h`, `GetTransactionWeight`, `GetBlockWeight`, and weight formula, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/consensus_2validation_8h_source.html, accessed 2026-08-04.
[^ref-btc-core-validation]: Bitcoin Core Contributors, `src/validation.cpp` and `src/validation.h`, `CheckBlockHeader`, `CheckMerkleRoot`, `CheckWitnessMalleation`, `CheckBlock`, contextual block checks, and `TestBlockValidity`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/validation_8cpp.html and https://doxygen.bitcoincore.org/validation_8h.html, accessed 2026-08-04.
[^ref-btc-core-chain]: Bitcoin Core Contributors, `src/chain.h`, `CBlockIndex`, block-index metadata and reconstructed headers, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/chain_8h_source.html, accessed 2026-08-04.
[^ref-btc-core-pow]: Bitcoin Core Contributors, `src/pow.cpp` and `src/pow.h`, proof-of-work target and hash validation, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/pow_8cpp_source.html and https://doxygen.bitcoincore.org/pow_8h_source.html, accessed 2026-08-04.

---

## 19. 교차 참조

### Parent

- Bitcoin Internals

### Previous

- BITCOIN-020 — Mining

### Next

- BITCOIN-022 — Nodes and Network Propagation

### Related

- BITCOIN-008 — Whitepaper Section 7 — Reclaiming Disk Space
- BITCOIN-009 — Whitepaper Section 8 — Simplified Payment Verification
- BITCOIN-015 — Transactions in Depth
- BITCOIN-016 — Script & ScriptPubKey
- BITCOIN-020 — Mining
- POW-009 — Coinbase Transaction and Mining Commitments
- POW-011 — Cumulative Chainwork
- POW-012 — Difficulty Adjustment
- POW-013 — Bitcoin Core Proof-of-Work Validation

---

## Review Status

### Technical Review

Passed.

- header field, serialized block structure, Merkle root, witness commitment, block weight, validation boundary를 분리했다.
- Bitcoin Core `CBlockHeader`, `CBlock`, Merkle function, weight function, validation function, `CBlockIndex` reference를 현재 Doxygen 기준으로 확인했다.
- header validation과 full block/contextual validation을 구분했다.
- SPV inclusion proof의 한계를 포함했다.

### Evidence Review

Passed.

- header와 serialized block 관련 설명은 Bitcoin Developer 문서와 Bitcoin Core primitive reference에 연결했다.
- Merkle root 관련 설명은 Bitcoin Developer 문서와 Bitcoin Core `merkle.cpp`에 연결했다.
- witness commitment는 BIP141에 연결했다.
- validation boundary는 Bitcoin Core validation 문서에 연결했다.
- timestamp precision, miner identity, SPV limit에 대한 분석 설명은 caveat를 포함했다.

### Editorial Review

Passed.

- Markdown heading은 프로젝트 deep-dive 구조를 따른다.
- metadata가 완전하다.
- table과 code fence가 닫혀 있다.
- 용어는 block, block header, block hash, Merkle root, witness commitment, `nBits`, `nNonce`, block weight로 일관된다.

### Adversarial Review

Passed.

- block hash가 전체 block을 직접 hash한다고 주장하지 않았다.
- block height를 header field로 취급하지 않았다.
- valid header를 full block validity의 증거로 취급하지 않았다.
- timestamp를 exact wall-clock time으로 취급하지 않았다.
- Merkle inclusion을 full transaction validation으로 취급하지 않았다.

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
