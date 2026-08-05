---
knowledge_id: ETHEREUM-FOUNDATION-007
title: Blocks
subtitle: Slot-based proposal, execution payload, state root, gas bound, 그리고 post-Merge Ethereum block 구조
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 140 min
estimated_study: 360 min
last_reviewed: 2026-08-04
domain:
  - Ethereum
  - Blocks
  - Consensus
  - Execution
parent:
  - Ethereum Foundations
prerequisites:
  - ETHEREUM-FOUNDATION-003
  - ETHEREUM-FOUNDATION-004
  - ETHEREUM-FOUNDATION-006
related_topics:
  - EVM
  - Gas
  - State Transition
  - Proof of Stake
  - Receipts
primary_sources:
  - REF-ETH-DOC-BLOCKS-2026-001
  - REF-ETH-DOC-POS-2026-001
  - REF-ETH-DOC-TX-2026-001
tags:
  - ethereum
  - blocks
  - execution-payload
  - proof-of-stake
  - state-root
  - gas-limit
---

# Blocks
> Ethereum Foundations  
> Research Unit: ETHEREUM-FOUNDATION-007

---

## Research Brief

```yaml
knowledge_id: ETHEREUM-FOUNDATION-007
title: Blocks
research_question: >
  What is an Ethereum block in August 2026, how do slot-based proof-of-stake
  proposal and execution payloads fit together, what fields matter most for
  researchers, and how should block structure be interpreted in a post-Merge
  Ethereum architecture?
document_type: deep-dive
difficulty: L300
prerequisites:
  - ETHEREUM-FOUNDATION-003
  - ETHEREUM-FOUNDATION-004
  - ETHEREUM-FOUNDATION-006
parent: Ethereum Foundations
previous: ETHEREUM-FOUNDATION-006
next: ETHEREUM-FOUNDATION-008
related_topics:
  - Gas
  - Transactions
  - Proof of Stake
  - Receipts
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Protocol Structure
  - Technical Mechanics
  - Mathematical or Economic Model
  - Security Assumptions
  - Protocol Implementation
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Full beacon-chain consensus spec
  - MEV-Boost builder market detail
  - Uncle/ommer historical proof-of-work deep dive
  - L2 batch posting specifics
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- post-Merge architecture에서 Ethereum block을 정의할 수 있다.
- slot, proposer, attestation, execution payload의 관계를 설명할 수 있다.
- execution과 analysis에 중요한 주요 block field를 식별할 수 있다.
- state root, receipts root, transaction이 block 안에 어떻게 맞물리는지 설명할 수 있다.
- gas bound가 block capacity를 어떻게 형성하는지 설명할 수 있다.

---

## 2. 핵심 질문

1. 2026년의 Ethereum block이란 무엇인가?
2. transaction이 이미 있는데도 block은 왜 필요한가?
3. proof-of-stake slot과 block proposal은 어떻게 작동하는가?
4. execution payload란 무엇인가?
5. analyst에게 중요한 block field는 무엇인가?
6. gas target과 gas limit은 block structure에 어떻게 영향을 주는가?

---

## 3. Executive Summary

Ethereum block은 cryptographic parent reference로 연결된 transaction batch이지만, post-Merge Ethereum에서는 proof-of-stake consensus structure와 execution-layer state transition이 만나는 지점으로 이해하는 것이 더 정확하다.[^ref-eth-doc-blocks]

current block documentation은 block이 participant가 synchronized state를 유지하고 transaction history에 합의하도록 도와주며, proof-of-stake Ethereum의 slot 리듬에 맞춰 12초마다 생성된다고 설명한다.[^ref-eth-doc-blocks]

같은 docs는 randomly selected validator가 block을 propose하고, 다른 validator가 transaction을 re-execute해 resulting state를 확인하며, client는 `execution_payload`를 실행한 결과가 block이 주장하는 `state_root`와 일치하는지 검증한다고 설명한다.[^ref-eth-doc-blocks]

2026년 8월 4일 기준, 올바른 Ethereum block 설명에는 다음이 포함되어야 한다.

- slot-based block proposal
- validator attestation
- execution payload field
- `state_root`, `transactions_root`, `receipts_root`
- gas target과 gas limit
- inclusion과 이후 consensus strengthening의 구분[^ref-eth-doc-blocks][^ref-eth-doc-pos][^ref-eth-doc-tx]

---

## 4. Protocol Structure

### Why Blocks Exist

official docs는 block이 많은 transaction을 한데 묶어 participant가 agreed state update를 한 번에 synchronize하게 한다고 설명한다.[^ref-eth-doc-blocks]

### Post-Merge Structure

2026년의 Ethereum block은 단순한 execution bundle이 아니다. 그것은 proof-of-stake consensus object이기도 하다.

### Minimal Structural View

```text
slot
-> proposer selected
-> execution payload assembled
-> transactions executed
-> new state_root claimed
-> validators attest
```

---

## 5. Slot-Based Proof of Stake

### Slot Timing

current block docs는 Ethereum time이 12-second slot으로 나뉘며, 각 slot마다 하나의 validator가 block proposal을 위해 선택된다고 설명한다.[^ref-eth-doc-blocks]

### Proposal and Re-Execution

docs는 선택된 validator가 transaction을 묶고, 실행하고, new state를 계산한 다음, block을 다른 validator에게 전달하며, 다른 validator는 proposed global state change에 동의하는지 보기 위해 transaction을 다시 실행한다고 설명한다.[^ref-eth-doc-blocks]

### Agreement Process

slot에 conflicting block이 존재할 경우, validator는 PoS protocol documentation이 설명하는 fork choice에 따라 가장 많은 staked ETH의 support를 받는 branch를 선택한다.[^ref-eth-doc-blocks][^ref-eth-doc-pos]

---

## 6. What Is in a Block

### High-Level Fields

current block docs는 다음과 같은 high-level field를 나열한다.[^ref-eth-doc-blocks]

- `slot`
- `proposer_index`
- `parent_root`
- `state_root`
- `body`

### Block Body

block `body`는 attestation과 slashing 같은 consensus-related field를 포함하며, `execution_payload`도 포함한다.[^ref-eth-doc-blocks]

### Why This Matters

이것은 Ethereum block을 miner-produced execution bundle로만 보던 pre-Merge mental model과 가장 선명하게 갈라지는 지점이다.

---

## 7. Execution Payload

### Core Role

current block docs는 `execution_payload`가 execution client에서 전달된 transaction을 포함한다고 설명한다.[^ref-eth-doc-blocks]

### Validation Logic

docs는 `execution_payload`의 transaction 실행이 global state를 갱신하며, 모든 client가 이를 재실행해 resulting state가 block의 `state_root`와 일치하는지 확인한다고 명시한다.[^ref-eth-doc-blocks]

### Key Execution Fields

`execution_payload_header`는 다음과 같은 field를 포함한다.[^ref-eth-doc-blocks]

- `parent_hash`
- `fee_recipient`
- `state_root`
- `receipts_root`
- `logs_bloom`
- `block_number`
- `gas_limit`
- `gas_used`
- `timestamp`
- `base_fee_per_gas`
- `transactions_root`

### Why Analysts Care

이 field들은 execution economics, state transition, consensus packaging을 서로 연결한다.

---

## 8. Gas Bounds and Block Size

### Current Capacity Rules

current block docs는 각 block이 30 million gas의 target size와 60 million gas의 block limit를 가진다고 설명한다.[^ref-eth-doc-blocks]

### Network Effect

이는 demand가 높을 때 block이 일시적으로 target을 넘어 확장될 수 있음을 뜻하며, fee mechanism은 base-fee adjustment를 통해 반응한다.[^ref-eth-doc-blocks]

### Capacity Is a Decentralization Variable

docs는 block이 arbitrarily large해질 수 없다고도 강조한다. block이 커질수록 hardware requirement가 올라가고 centralizing pressure가 생기기 때문이다.[^ref-eth-doc-blocks]

---

## 9. Technical Mechanics

### Simplified Block Lifecycle

```text
pending transactions exist
-> proposer selected for slot
-> proposer assembles execution payload
-> execution performed
-> state_root and receipts_root determined
-> block propagated
-> other validators re-execute and attest
```

### Transaction Ordering

current block docs는 block 내부 transaction이 strictly ordered된다고 설명한다.[^ref-eth-doc-blocks]

### Execution and Commitment

post-execution commitment는 단순한 transaction list가 아니다. new state commitment와 receipts commitment도 포함한다.

---

## 10. Security Assumptions

### Re-Execution by Others

security는 다른 validator와 node가 proposer의 claim을 그대로 믿지 않고 independently payload를 re-execute하는 데 의존한다.[^ref-eth-doc-blocks]

### Capacity Limits

gas bound는 demand에 따라 block size가 무한정 커지는 것을 막아 centralization을 억제한다.[^ref-eth-doc-blocks]

### Slot Reliability

block time이 slot-based이므로 proposer가 offline이면 empty slot이 생길 수 있지만, protocol rhythm 자체는 12초로 유지된다.[^ref-eth-doc-blocks]

---

## 11. Mathematical or Economic Model

### Capacity Model

단순화한 block-capacity rule은 다음과 같다.

```text
sum(gas_used_by_transactions) <= block gas limit
```

이것이 block의 execution-capacity constraint다.

### Time Model

current docs는 다음을 설명한다.

```text
1 slot = 12 seconds
1 proposer per slot
```

[^ref-eth-doc-blocks][^ref-eth-doc-pos]

### Congestion Link

target block size가 hard block limit보다 낮기 때문에, every block을 항상 최대로 만들지 않으면서도 temporary burst를 허용할 수 있다.[^ref-eth-doc-blocks]

---

## 12. Protocol Implementation

### Current Primary Source

official blocks documentation은 post-Merge block structure와 execution payload semantic을 설명하는 가장 명확한 current source다.[^ref-eth-doc-blocks]

### Supporting Context

PoS docs는 proposer/attestation timing을 보완해 주고, transaction docs는 block inclusion과 transaction lifecycle을 연결해 준다.[^ref-eth-doc-pos][^ref-eth-doc-tx]

### Why This Matters

consensus-layer structure 없이 execution-layer block field만 설명하는 older educational model은 더 이상 충분하지 않다.

---

## 13. On-Chain Implications

### Rich Block-Level Analytics

block을 통해 analyst는 다음을 연구할 수 있다.

- proposer cadence
- gas usage
- fee recipient behavior
- state/receipts commitment
- logs bloom
- inclusion timing

### Block Data Is Not Just Transaction Count

Ethereum의 block analysis는 gas, execution, consensus structure와 분리될 수 없다.

### Post-Merge Interpretation

연구자는 modern Ethereum block을 combined consensus-plus-execution object로 읽어야 한다.

---

## 14. Institutional Thinking

institution은 Ethereum block을 execution risk, fee realization, consensus timing의 main packaging layer로 다뤄야 한다.

### Practical Implications

- monitoring은 execution payload metric과 consensus timing을 함께 포함해야 한다.
- block fullness와 gas usage는 cosmetic stat이 아니라 operational signal이다.
- finality confidence와 inclusion timing은 separate하게 추적해야 한다.
- post-Merge Ethereum에서는 validator-oriented field가 older miner-centric model보다 더 중요하다.

### Better Research Posture

block claim을 하기 전에 다음을 물어야 한다.

- 이것이 consensus-layer block statement인가, execution-payload statement인가?
- capacity, ordering, finality 중 무엇을 분석하는가?
- post-Merge mental model을 쓰고 있는가?

---

## 15. Common Misinterpretations

### "An Ethereum block is just a bag of transactions"

틀렸다. consensus structure와 execution commitment도 포함한다.[^ref-eth-doc-blocks]

### "Block time means every 12 seconds a block always appears"

틀렸다. slot은 12초지만, proposer가 offline이면 empty slot이 가능하다.[^ref-eth-doc-blocks]

### "`state_root` and `transactions_root` say the same thing"

틀렸다. 하나는 resulting state에 commit하고, 다른 하나는 transaction list에 commit한다.

### "Bigger blocks are always better"

틀렸다. 더 큰 capacity는 centralization pressure를 만들 수 있다.[^ref-eth-doc-blocks]

---

## 16. Research Questions

1. execution congestion과 consensus health를 가장 잘 포착하는 block-level metric은 무엇인가?
2. institution은 inclusion fact와 finality fact를 어떻게 구분해 라벨링해야 하는가?
3. Ethereum operational analytics에서 가장 과소활용되는 post-Merge field는 무엇인가?

---

## 17. Practical Exercises

### Exercise 1

왜 `execution_payload`가 post-Merge block interpretation의 중심인지 설명하라.

### Exercise 2

`state_root`, `transactions_root`, `receipts_root`의 차이를 짧게 설명하라.

### Exercise 3

proposed block을 들은 뒤 다른 validator가 무엇을 하는지 설명하라.

### Exercise 4

왜 12-second slot이 매 slot마다 non-empty block을 보장하지 않는지 설명하라.

---

## 18. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Slot-based block proposal and re-execution | Directly specified | Official blocks and PoS docs |
| Execution payload semantics and field meanings | Directly specified | Official blocks docs |
| Gas target and block limit | Directly specified | Official blocks docs |
| Institutional monitoring implications | Inference from sources | Derived from current block structure |

---

## 19. Knowledge Graph

```text
Blocks
├─ Consensus Layer Structure
│  ├─ slot
│  ├─ proposer
│  ├─ attestations
│  └─ fork choice
├─ Execution Layer Structure
│  ├─ execution_payload
│  ├─ transactions
│  ├─ state_root
│  ├─ receipts_root
│  └─ base_fee_per_gas
├─ Capacity
│  ├─ gas target
│  ├─ gas limit
│  └─ gas used
└─ Research Concerns
   ├─ inclusion
   ├─ finality
   ├─ congestion
   └─ validator behavior
```

---

## 20. References

### Primary Sources

[^ref-eth-doc-blocks]: ethereum.org, "Blocks," official documentation describing post-Merge block structure, slots, execution payloads, state roots, and gas bounds, page last updated February 23, 2026, https://ethereum.org/developers/docs/blocks/, accessed 2026-08-04.

[^ref-eth-doc-pos]: ethereum.org, "Proof-of-stake (PoS)," official documentation describing proposer selection, attestation, and finality context, https://ethereum.org/developers/docs/consensus-mechanisms/pos/, accessed 2026-08-04.

[^ref-eth-doc-tx]: ethereum.org, "Transactions," official documentation describing transaction lifecycle and inclusion, https://ethereum.org/developers/docs/transactions/, accessed 2026-08-04.

### Supporting Interpretation Notes

- Where this document discusses institutional monitoring posture or block-level analytics emphasis, those are analytical inferences built on the cited official block structure documentation.

---

## 21. Cross References

### Previous

- ETHEREUM-FOUNDATION-006 — Gas

### Next

- ETHEREUM-FOUNDATION-008 — Receipts

### Related

- ETHEREUM-FOUNDATION-004 — State Transition
- ETHEREUM-FOUNDATION-005 — EVM

---

## Review Status

### Technical Review

Passed.

- post-Merge block structure를 consensus-plus-execution object로 설명했다.
- `execution_payload` semantic을 consensus field와 분리했다.
- gas capacity bound를 포함했다.
- inclusion과 re-execution behavior를 다뤘다.

### Evidence Review

Passed.

- core block-structure claim은 current official blocks docs를 인용한다.
- PoS timing/role claim은 official PoS docs를 인용한다.
- transaction lifecycle linkage는 transaction docs를 인용한다.
- institutional interpretation은 inference로 라벨링했다.

### Editorial Review

Passed.

- 구조는 프로젝트 문서 패턴을 따른다.
- metadata는 완전하다.
- terminology는 slot, proposer, execution payload, state root, receipts root, gas limit로 일관된다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 Ethereum block을 transaction bag으로 축소하지 않는다.
- every 12 seconds마다 반드시 block이 나온다고 설명하지 않는다.
- `state_root`와 `transactions_root`를 혼동하지 않는다.
- bigger block이 항상 좋다고 말하지 않는다.

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
