---
knowledge_id: BITCOIN-030
title: SegWit
subtitle: witness 분리, malleability 완화, txid와 wtxid, block weight, witness commitment, 그리고 검증 경계
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 135 min
estimated_study: 390 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Consensus
  - Transactions
  - SegWit
parent:
  - Modern Bitcoin
prerequisites:
  - BITCOIN-009
  - BITCOIN-015
  - BITCOIN-016
  - BITCOIN-021
  - BITCOIN-022
related_topics:
  - Transaction Malleability
  - Witness Commitment
  - txid
  - wtxid
  - P2WPKH
  - P2WSH
  - Block Weight
  - Bech32
primary_sources:
  - REF-BTC-WP-001
  - REF-BIP-0141
  - REF-BIP-0143
  - REF-BIP-0144
  - REF-BTC-DEV-TRANSACTIONS-001
  - REF-BTC-DEV-BLOCKCHAIN-001
  - REF-BTC-DEV-P2P-001
  - REF-BTC-CORE-CONSENSUS-VALIDATION-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-BIPS-001
tags:
  - bitcoin
  - segwit
  - consensus
  - transactions
  - witness
  - malleability
  - block-weight
  - p2wsh
---

# SegWit
> Modern Bitcoin  
> Research Unit: BITCOIN-030

---

## Research Brief

```yaml
knowledge_id: BITCOIN-030
title: SegWit
research_question: >
  What did Segregated Witness change in Bitcoin's transaction and block model,
  how does it mitigate involuntary transaction malleability, what is the
  difference between txid and wtxid, how do witness programs and witness
  commitments work, and how should analysts separate consensus changes from
  relay, serialization, and wallet-level consequences?
document_type: deep-dive
difficulty: L400
prerequisites:
  - BITCOIN-009
  - BITCOIN-015
  - BITCOIN-016
  - BITCOIN-021
  - BITCOIN-022
parent: Modern Bitcoin
previous: BITCOIN-029
next: BITCOIN-031
related_topics:
  - Malleability
  - Witness Programs
  - Block Weight
  - SPV
  - Transaction Relay
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
  - Full Lightning Network protocol design
  - Full Taproot design
  - Address UX details beyond analytical necessity
  - Exhaustive historical activation politics
  - Detailed wallet implementation tutorials
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- SegWit가 왜 도입되었는지 설명할 수 있다.
- transaction effect와 validation에만 쓰이는 witness data를 구분할 수 있다.
- `txid`와 `wtxid`를 구분할 수 있다.
- SegWit가 involuntary transaction malleability를 어떻게 완화하는지 설명할 수 있다.
- coinbase transaction 안의 witness-commitment structure를 설명할 수 있다.
- native witness program과 P2SH-nested witness program을 설명할 수 있다.
- block weight와 virtual size를 설명할 수 있다.
- consensus change와 relay/wallet-level consequence를 분리할 수 있다.

---

## 2. 핵심 질문

1. SegWit는 어떤 문제를 해결하려고 했는가?
2. "witness" data는 정확히 무엇인가?
3. 왜 SegWit transaction에는 `txid`와 `wtxid`가 모두 있는가?
4. SegWit는 malleability risk를 어떻게 줄이는가?
5. block header에는 무엇이 커밋되고, coinbase witness commitment에는 무엇이 커밋되는가?
6. P2WPKH와 P2WSH는 무엇인가?
7. 왜 SegWit는 단순히 "block size 증가"가 아니라 block weight를 도입했는가?
8. network relay와 serialization에서는 무엇이 바뀌었는가?
9. non-upgraded node는 무엇을 볼 수 있고 무엇을 검증하지 못하는가?

---

## 3. Executive Summary

주로 BIP141에 정의된 Segregated Witness는 witness data라는 새로운 구조를 도입해, 이를 legacy transaction Merkle tree와 별도로 block에 커밋한다. witness data는 지출 검증에 필요한 정보, 특히 signature와 관련 script data를 담지만, transaction이 UTXO set에 미치는 효과를 결정하는 데는 필요하지 않다. 이는 publication proof와 full validation data를 구분하던 백서 시대의 아이디어를 한 단계 확장한 것이다.[^ref-bip-0141] [^ref-btc-wp]

이 분리는 여러 문제를 한 번에 푼다. 가장 중요한 점은, SegWit spend에 대해 legacy `txid`가 더 이상 witness serialization에 커밋하지 않기 때문에 involuntary signature-data malleability가 transaction identification을 바꾸지 못하게 된다는 것이다. 그래서 BIP141는 전통적인 `txid`와 witness-inclusive `wtxid`를 모두 정의한다.[^ref-bip-0141]

SegWit는 block capacity accounting도 바꾼다. 예전 serialized-size model만 쓰는 대신, older node와 soft-fork compatible하게 witness byte를 base byte보다 할인하는 block weight와 transaction weight를 도입한다.[^ref-bip-0141] [^ref-btc-core-consensus-validation]

block level에서 witness data는 80-byte header에 직접 들어가지 않는다. 대신 BIP141는 coinbase transaction에 witness commitment를 넣어 block의 `wtxid` tree를 commitment hash로 묶도록 요구한다. Bitcoin Core validation은 `GetWitnessCommitmentIndex`, weight helper, witness-malleation validation path를 통해 이를 드러낸다.[^ref-bip-0141] [^ref-btc-core-consensus-validation] [^ref-btc-core-validation]

분석가에게 SegWit는 단순 throughput 개선이 아니다. transaction identity, serialization, validation, commitment system을 바꾸는 업그레이드이며, wallet, fee calculation, relay, higher-layer protocol에 장기적인 영향을 준다.

---

## 4. 프로토콜 구조

### SegWit 이전

legacy transaction identity는 developer reference에 기록된 전통적 raw transaction structure와 동일한 serialization 하나만 사용했다.[^ref-btc-dev-transactions]

```text
[nVersion][txins][txouts][nLockTime]
```

signature는 `scriptSig` 안에 있었기 때문에 signature-related mutation이 transaction hash를 바꿀 수 있었다.[^ref-bip-0141]

### SegWit 이후

SegWit는 witness data를 별도로 정의한다. 이제 transaction은 다음 둘을 가질 수 있다.

- legacy `txid`
- witness-inclusive `wtxid`[^ref-bip-0141]

`wtxid`는 marker, flag, witness field를 포함한 serialization을 사용한다.

### 핵심 설계 아이디어

SegWit는 다음을 분리한다.

- effect를 결정하는 데이터: spend와 new output
- authorization을 검증하는 데이터: witness

이 구분이 SegWit가 malleability behavior를 개선하고 새로운 commitment pattern을 가능하게 하는 핵심 이유다.

---

## 5. Transaction Identity와 Malleability

### `txid`

BIP141는 old `txid` 정의를 그대로 유지한다.

```text
txid = double_sha256([nVersion][txins][txouts][nLockTime])
```

### `wtxid`

BIP141는 다음을 정의한다.

```text
wtxid = double_sha256([nVersion][marker][flag][txins][txouts][witness][nLockTime])
```

모든 입력이 non-witness program이면 `wtxid = txid`다.[^ref-bip-0141]

### 왜 중요한가

legacy transaction에서는 signature-related data mutation이 경제적 effect가 바뀌지 않아도 transaction hash를 바꿀 수 있었다. SegWit spend에서는 witness mutation이 legacy `txid`를 바꾸지 않으므로, 해당 spend에 대한 involuntary malleability가 크게 줄어든다.[^ref-bip-0141]

중요한 경계:

- SegWit는 SegWit input에 대한 involuntary malleability를 완화한다.
- 가능한 모든 higher-level transaction-graph risk를 없애는 것은 아니다.
- legacy input은 여전히 legacy다.

---

## 6. Witness Program과 Spending Form

### Witness Program

BIP141는 다음 script form에 특별한 의미를 부여한다.

- version byte (`OP_0` to `OP_16`)
- 그 뒤의 2~40 byte push, 즉 witness program[^ref-bip-0141]

### Native Witness Program

`scriptPubKey`가 직접 witness program이라면:

- `scriptSig`는 비어 있어야 하고
- witness data가 실제 spending argument를 담는다.[^ref-bip-0141]

### P2SH-Nested Witness Program

`scriptPubKey`가 P2SH이고 `redeemScript`가 witness program이면:

- `scriptSig`는 그 redeem script push만 포함해야 하고
- 실제 unlocking data는 witness에 들어간다.[^ref-bip-0141]

이 구조는 native witness output이 널리 쓰이기 전에 SegWit를 위한 compatibility bridge 역할을 했다.

### Version 0 Program

witness version 0에서:

- 20-byte program은 P2WPKH
- 32-byte program은 P2WSH를 의미한다.[^ref-bip-0141]

P2WPKH는 signature와 pubkey를 witness로 옮긴다. P2WSH는 script와 witness stack을 witness로 옮기고, program 안에는 `SHA256(witnessScript)`만 커밋한다.

---

## 7. Block-Level Commitment Structure

### Header Merkle Root

block header는 여전히 `wtxid`가 아니라 `txid`에서 계산한 legacy transaction Merkle root에 커밋한다.[^ref-btc-dev-blockchain] [^ref-bip-0141]

### Witness Commitment

BIP141는 coinbase transaction을 통해 witness data에 커밋하는 새로운 block rule을 추가한다. witness Merkle root는 leaf로 `wtxid`를 사용하고, coinbase `wtxid`는 all-zeroes로 취급하며, commitment는 정의된 pattern의 coinbase `scriptPubKey`에 넣는다.[^ref-bip-0141]

commitment structure는 다음을 포함한다.

```text
OP_RETURN
0x24 push length
0xaa21a9ed commitment header
32-byte commitment hash = Double-SHA256(witness_root_hash | witness_reserved_value)
```

그리고 coinbase input witness는 single 32-byte witness reserved value를 담아야 한다.[^ref-bip-0141]

### 왜 Witness Tree를 Header에 넣지 않았는가

witness commitment를 coinbase를 통해 중첩한 이유는 soft-fork compatibility 때문이다. old node는 기존 header와 Merkle-root model을 그대로 보고, upgraded node만 추가 witness commitment를 검증한다.[^ref-bip-0141]

---

## 8. Weight, Size, and Capacity

### Block Weight

BIP141는 예전 single-size limit을 block weight로 대체한다.

```text
block_weight = base_size * 3 + total_size
```

합의 규칙은:

```text
block_weight <= 4,000,000
```

여기서 base size는 witness data를 뺀 serialization 크기이고, total size는 witness를 포함한 전체 serialization이다.[^ref-bip-0141]

### Transaction Weight와 Virtual Size

BIP141는 transaction weight와 virtual transaction size도 정의한다.

```text
tx_weight = base_tx_size * 3 + total_tx_size
vsize = ceil(tx_weight / 4)
```

Bitcoin Core는 witness 포함/제외 serialization을 사용해 `GetTransactionWeight`와 `GetBlockWeight`에서 이 공식을 직접 구현한다.[^ref-btc-core-consensus-validation]

### 분석상 함의

SegWit가 "witness byte는 공짜"라는 뜻은 아니다. block-capacity accounting model에서 witness byte가 base byte보다 할인된다는 뜻이다.

---

## 9. Relay와 Serialization Change

### BIP144

BIP144는 witness-bearing data를 relay하는 peer를 위해 새로운 transaction/block serialization rule을 추가하며, witness-aware inventory type과 P2P layer의 serialization expectation도 정의한다.[^ref-bip-0144] [^ref-btc-dev-p2p]

예를 들어 developer P2P 문서에는 `MSG_WITNESS_TX`, `MSG_WITNESS_BLOCK` 같은 witness-related inventory identifier가 포함된다.[^ref-btc-dev-p2p]

### 왜 Relay가 바뀌었는가

witness data가 legacy serialization과 구분되면, network layer도 peer capability와 use case에 따라 witness를 포함하거나 제외한 transaction/block을 request하고 serve할 방법이 필요하다.

---

## 10. Signature Digest와 Validation Semantics

### BIP143

BIP143는 version 0 witness program을 위한 새로운 transaction digest algorithm을 도입한다.[^ref-bip-0141] [^ref-bip-0143]

즉, SegWit는 단지 byte를 옮기는 것이 아니라, witness version 0 spend에서 signature가 transaction data에 어떻게 커밋하는지도 바꾼다.

### Validation Surface

Bitcoin Core validation은 다음을 통해 witness-specific boundary를 노출한다.

- `GetWitnessCommitmentIndex`
- witness-commitment constant
- witness-malleation check
- `TX_WITNESS_MUTATED`, `TX_WITNESS_STRIPPED` 같은 validation state[^ref-btc-core-consensus-validation] [^ref-btc-core-validation]

이는 witness data가 wallet-only encoding detail이 아니라 first-class validation concern임을 보여 준다.

---

## 11. Validation Boundaries

### Non-Upgraded Node

BIP141의 backward-compatibility section은 non-upgraded node가 witness data를 보지도, 검증하지도 못하고, witness program을 old rule 아래에서 해석한다는 점과 그 한계를 설명한다.[^ref-bip-0141]

이것이 soft-fork 설계의 핵심이다.

- old node도 계속 동작한다.
- upgraded node는 추가 rule을 집행한다.
- witness semantics는 old software가 받아들일 수 있는 구조 뒤에 숨는다.

### SegWit는 단순 Bigger Block이 아니다

과장된 해석은 피해야 한다.

- SegWit는 단순한 blocksize increase가 아니다.
- 단순한 새 address format도 아니다.
- 모든 transaction dependency risk를 완전히 치유하는 것도 아니다.

이는 consensus, serialization, validation, commitment를 함께 바꾸는 reform이다.

---

## 12. Security Assumptions and Failure Modes

### Malleability Reduction

SegWit는 witness data를 `txid` 계산에서 제거함으로써 SegWit spend의 transaction-malleability 문제를 근본적으로 개선한다. 이는 unconfirmed transaction dependency chain과 그 위에 쌓이는 protocol에 중요하다.[^ref-bip-0141]

### SPV와 Proof Implication

BIP141의 motivation은 signature data의 optional transmission이 SPV proof size를 줄이고 bandwidth tradeoff를 개선할 수 있다고도 설명한다. 다만 SPV는 여전히 full validation보다 약하다.[^ref-bip-0141]

### Policy vs Consensus

BIP141는 초기 release와 함께 적용된 relay/mining policy도 논의한다. witness 관련 usage caution은 consensus change 자체와 분리해서 읽어야 한다.[^ref-bip-0141]

### Legacy Coexistence

Bitcoin이 하루아침에 purely witness-native system으로 바뀐 것은 아니다. nested form, legacy form, 이후의 native-address adoption이 섞여 있기 때문에 dataset에는 여러 era와 spend type이 공존한다. "SegWit"를 binary하게만 보면 중요한 detail을 잃기 쉽다.

---

## 13. Mathematical or Economic Model

### Weight Formula

핵심적인 정량 변화는 다음이다.

```text
weight = stripped_size * 3 + total_size
```

Bitcoin Core는 이것이 다음과 동치라고도 보여 준다.

```text
weight = stripped_size * 4 + witness_size
```

왜냐하면 `witness_size = total_size - stripped_size`이기 때문이다.[^ref-btc-core-consensus-validation]

### Feerate Implication

SegWit 이후 transaction fee는 보통 raw total byte count가 아니라 virtual-size 기준으로 비교되므로, fee market은 witness-discounted capacity use를 더 직접적으로 가격화하게 되었다. 이 때문에 SegWit는 fee analytics에도 직접 중요하다.

### Malleability와 Dependency Chain

downstream transaction이 parent transaction ID를 참조할 때, SegWit spend에서 parent `txid`가 더 안정적이라는 점은 pre-confirmation dependency construction의 신뢰성을 높인다. 특정 second-layer design을 논하기 전에도, 이것이 layered protocol을 가능하게 만든 핵심 경제적 변화 중 하나다.

---

## 14. Bitcoin Core 구현

### `consensus/validation.h`

Bitcoin Core는 여기서 다음 witness-related consensus helper를 구현한다.

- `GetTransactionWeight`
- `GetBlockWeight`
- `GetTransactionInputWeight`
- `GetWitnessCommitmentIndex`
- witness-commitment constant[^ref-btc-core-consensus-validation]

### `validation.cpp`

validation에는 `CheckWitnessMalleation` 같은 witness-specific logic이 포함되며, 이는 SegWit가 block level에 explicit consensus validation pathway를 추가했음을 보여 준다.[^ref-btc-core-validation]

### `doc/bips.md`

Bitcoin Core의 implemented-BIP index는 BIP144 SegWit-related support가 0.13.0부터 존재한다고 기록한다.[^ref-btc-core-bips]

---

## 15. 온체인 함의

### Analyst가 추적해야 할 것

SegWit-aware analysis는 다음을 구분할 필요가 있다.

- legacy input vs witness-bearing input
- `txid` context vs `wtxid` context
- native witness vs P2SH-nested witness
- block weight vs raw size
- 필요한 block에서 witness commitment 존재 여부

### Common Dataset Failure

오래되었거나 정규화가 부실한 dataset은 종종:

- raw size만 보고 weight를 무시하거나
- witness serialization distinction을 놓치거나
- nested witness spend를 잘못 분류하거나
- 2017년 이후 activity 전체를 균일한 "SegWit"으로 취급한다

### Fee Interpretation

SegWit 이후 fee analytics는 miner inclusion economics와 관련된 질문이라면 raw byte count보다 vsize/weight-aware metric을 사용해야 한다.

---

## 16. Institutional Thinking

기관은 SegWit를 장기적인 분석 함의를 가진 인프라 업그레이드로 봐야 한다.

### Practical Implications

- custody와 settlement system은 `txid` 중심 workflow와 witness-inclusive transaction handling을 구분해야 한다.
- fee estimation과 transaction-cost model은 virtual size를 사용해야 한다.
- audit와 indexing system은 필요한 경우 witness-aware serialization을 보존해야 한다.
- legacy Bitcoin의 malleability assumption을 SegWit spend에 그대로 투영하면 안 된다.

---

## 17. Common Misinterpretations

### "SegWit는 그냥 Block Size를 늘렸다"

틀렸다. transaction identity, commitment structure, validation boundary, capacity accounting을 모두 바꿨다.

### "SegWit가 Header의 Transaction Merkle Root를 없앴다"

틀렸다. header는 여전히 legacy `txid` Merkle root에 커밋하며, witness data는 coinbase witness commitment를 통해 별도로 커밋된다.

### "`wtxid`가 `txid`를 완전히 대체했다"

틀렸다. 둘 다 존재하며 목적도 다르다.

### "SegWit는 모든 Bitcoin Transaction을 Non-Malleable하게 만들었다"

틀렸다. SegWit spend에 대한 involuntary malleability를 완화할 뿐이며, 모든 conceivable transaction-graph risk나 legacy input까지 없애는 것은 아니다.

### "Witness Byte는 계산에 안 들어간다"

틀렸다. weight accounting 아래에서 계산에 들어가지만, base byte 대비 할인될 뿐이다.

---

## 18. Research Questions

1. raw-size 기반 fee measurement 때문에 여전히 얼마나 많은 analyst error가 발생하는가?
2. nested witness와 native witness adoption은 wallet population별로 얼마나 다르게 전개되었는가?
3. post-SegWit transaction-chain reliability 개선 중 얼마나 많은 부분이 malleability mitigation에 직접 기인하는가?
4. 어떤 dataset이 institutional audit을 위해 witness-aware transaction identity를 가장 잘 보존하는가?

---

## 19. Practical Exercises

### Exercise 1

version 0 witness spend에 대해 `txid`와 `wtxid`의 차이를 설명하라.

### Exercise 2

witness-bearing transaction이 들어 있는 block을 골라, header Merkle root가 커밋하는 데이터와 coinbase witness commitment가 커밋하는 데이터를 구분하라.

### Exercise 3

raw total byte가 아니라 virtual size로 transaction의 fee density를 계산하고, 왜 차이가 중요한지 설명하라.

### Exercise 4

script structure를 보고 spend를 legacy, native P2WPKH, P2SH-nested P2WPKH, native P2WSH, P2SH-nested P2WSH로 분류하라.

---

## 20. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Witness structure, txid/wtxid, witness commitment, weight rules | Directly specified | BIP141 and Core consensus helpers |
| v0 witness digest semantics | Directly specified | BIP143 |
| Witness-aware relay serialization | Directly specified | BIP144 and P2P reference |
| Throughput and fee implications | Directly specified plus inference | Weight rules are specified; market effects are analytical |
| Malleability mitigation and higher-layer enablement | Directly specified plus inference | BIP141 motivation plus analytical consequence |

---

## 21. Knowledge Graph

```text
SegWit
├─ Motivation
│  ├─ malleability mitigation
│  ├─ witness segregation
│  ├─ script extensibility
│  └─ capacity accounting reform
├─ Identity
│  ├─ txid
│  └─ wtxid
├─ Script Forms
│  ├─ P2WPKH
│  ├─ P2WSH
│  ├─ native witness
│  └─ P2SH-nested witness
├─ Block Commitments
│  ├─ txid merkle root
│  ├─ witness merkle root
│  └─ coinbase witness commitment
├─ Capacity
│  ├─ block weight
│  ├─ tx weight
│  └─ vsize
└─ Implications
   ├─ fee analytics
   ├─ relay serialization
   ├─ wallet handling
   └─ off-chain protocol support
```

---

## 22. 참고문헌

### Primary Sources

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," especially Sections 8 and 11 for original SPV and confirmation context. https://bitcoin.org/bitcoin.pdf
[^ref-bip-0141]: BIP141, "Segregated Witness (Consensus layer)." https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki
[^ref-bip-0143]: BIP143, "Transaction Signature Verification for Version 0 Witness Program." https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki
[^ref-bip-0144]: BIP144, "Segregated Witness (Peer Services)." https://github.com/bitcoin/bips/blob/master/bip-0144.mediawiki
[^ref-btc-dev-transactions]: Bitcoin Developer Reference, "Transactions." https://developer.bitcoin.org/reference/transactions.html
[^ref-btc-dev-blockchain]: Bitcoin Developer Reference, "Block Chain." https://developer.bitcoin.org/reference/block_chain.html
[^ref-btc-dev-p2p]: Bitcoin Developer Reference, "P2P Network," including witness-aware inventory types. https://developer.bitcoin.org/reference/p2p_networking.html
[^ref-btc-core-consensus-validation]: Bitcoin Core Doxygen, `consensus/validation.h`, including weight helpers and witness commitment helpers. https://doxygen.bitcoincore.org/consensus_2validation_8h_source.html
[^ref-btc-core-validation]: Bitcoin Core Doxygen, `validation.cpp`, including `CheckWitnessMalleation`. https://doxygen.bitcoincore.org/validation_8cpp.html
[^ref-btc-core-bips]: Bitcoin Core `doc/bips.md`, implemented BIP index including BIP144 SegWit support. https://github.com/bitcoin/bitcoin/blob/master/doc/bips.md

### Supporting Interpretation Notes

- Where this document discusses fee-market consequences, layered-protocol enablement, or institutional data-model implications, those statements are analytical inferences built on the SegWit consensus and serialization changes rather than stand-alone protocol commands.

---

## 23. 교차 참조

### Previous

- BITCOIN-029 — Bitcoin Game Theory

### Next

- BITCOIN-031 — Taproot

### Related

- BITCOIN-009 — Whitepaper Section 8 — Simplified Payment Verification
- BITCOIN-015 — Transactions in Depth
- BITCOIN-016 — Script & ScriptPubKey
- BITCOIN-018 — Transaction Fees
- BITCOIN-021 — Blocks and Block Headers
- BITCOIN-022 — Nodes and Network Propagation
- BITCOIN-031 — Taproot

---

## Review Status

### Technical Review

Passed.

- witness structure, identity change, capacity accounting, commitment structure, validation boundary를 분리했다.
- `txid`와 `wtxid`를 문서 전체에서 구분했다.
- header Merkle commitment와 coinbase witness commitment를 혼동하지 않았다.
- native와 nested witness form을 나눠 설명했다.

### Evidence Review

Passed.

- BIP141이 핵심 SegWit consensus/structure claim을 뒷받침한다.
- BIP143이 witness-v0 digest semantics를 뒷받침한다.
- BIP144와 P2P reference가 relay/serialization claim을 뒷받침한다.
- Core Doxygen이 weight와 witness-commitment implementation reference를 뒷받침한다.
- interpretive consequence는 inference로 표시했다.

### Editorial Review

Passed.

- 구조는 프로젝트 deep-dive format을 따른다.
- metadata가 완전하다.
- witness, txid, wtxid, witness program, block weight, vsize, witness commitment 용어가 일관된다.
- table과 code fence가 닫혀 있다.

### Adversarial Review

Passed.

- SegWit를 단순한 blocksize change로 축소하지 않았다.
- `wtxid`가 `txid`를 보편적으로 대체했다고 주장하지 않았다.
- witness byte가 공짜라고 하지 않았다.
- malleability fix를 SegWit scope 이상으로 과장하지 않았다.
- relay change와 consensus change를 혼동하지 않았다.

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
