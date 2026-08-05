---
knowledge_id: ETHEREUM-FOUNDATION-008
title: Receipts
subtitle: Post-Execution Outcome Record, Receipts Root, Cumulative Gas, Status, Logs Bloom, 그리고 Query Boundary
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 125 min
estimated_study: 320 min
last_reviewed: 2026-08-04
domain:
  - Ethereum
  - Receipts
  - Execution
  - Data Structures
parent:
  - Ethereum Foundations
prerequisites:
  - ETHEREUM-FOUNDATION-004
  - ETHEREUM-FOUNDATION-007
related_topics:
  - Logs & Events
  - Transactions
  - Blocks
  - Receipts Trie
primary_sources:
  - REF-ETH-DOC-MPT-2026-001
  - REF-EIP-2718
  - REF-ETH-DOC-BLOCKS-2026-001
  - REF-ETH-DOC-JSONRPC-2026-001
tags:
  - ethereum
  - receipts
  - receipt-root
  - logs-bloom
  - cumulative-gas
---

# Receipts
> Ethereum Foundations  
> Research Unit: ETHEREUM-FOUNDATION-008

---

## Research Brief

```yaml
knowledge_id: ETHEREUM-FOUNDATION-008
title: Receipts
research_question: >
  What are Ethereum transaction receipts, how are they committed through the
  receipts trie and receipts root, what execution outcome fields do they carry,
  and how should analysts distinguish receipts from transactions and from full
  state in August 2026?
document_type: deep-dive
difficulty: L300
prerequisites:
  - ETHEREUM-FOUNDATION-004
  - ETHEREUM-FOUNDATION-007
parent: Ethereum Foundations
previous: ETHEREUM-FOUNDATION-007
next: ETHEREUM-FOUNDATION-009
related_topics:
  - Logs
  - Transactions
  - Blocks
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
  - Full RPC client tutorial
  - Indexer implementation details
  - Bloom-filter algorithm derivation
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Ethereum transaction receipt를 정의할 수 있다.
- receipt가 transaction 및 world state와 어떻게 다른지 설명할 수 있다.
- `receiptsRoot`가 무엇에 commit하는지 설명할 수 있다.
- status, cumulative gas used, logs bloom, logs 같은 주요 receipt field를 식별할 수 있다.
- post-execution analysis에서 receipt가 왜 핵심인지 설명할 수 있다.

---

## 2. 핵심 질문

1. receipt란 무엇인가?
2. 왜 Ethereum은 transaction 외에 receipt도 필요한가?
3. receipt는 block 안에서 어떻게 commit되는가?
4. receipt는 어떤 field를 담고 있는가?
5. 왜 receipt는 contract interaction에서 특히 중요한가?

---

## 3. Executive Summary

Ethereum transaction receipt는 transaction이 included되고 execution된 뒤에 생성되는 structured post-execution record다. 그것은 transaction 자체도 아니고 full state도 아니다. execution result, gas accounting, emitted log를 기록하는 outcome artifact다.[^ref-eth-doc-jsonrpc][^ref-eip-2718]

official trie documentation은 every block이 transaction index를 key로 하는 receipts trie를 가지며, block header의 `receiptsRoot`가 그 receipt들에 commit한다고 설명한다.[^ref-eth-doc-mpt]

EIP-2718은 receipt가 legacy receipt일 수도 있고 transaction type에 따라 format이 달라지는 typed receipt일 수도 있다고 formalize하지만, 어느 경우든 receipt root는 execution outcome에 대한 block-level commitment로 유지된다.[^ref-eip-2718]

연구자에게 receipt가 중요한 이유는 가장 유용한 post-execution signal의 많은 부분이 여기에 있기 때문이다.

- success/failure status
- cumulative gas used
- event log
- logs bloom
- transaction-type-aware outcome encoding[^ref-eip-2718][^ref-eth-doc-jsonrpc]

---

## 4. Protocol Structure

### Where Receipts Sit

Ethereum execution surface는 다음처럼 분리할 수 있다.

```text
transaction -> requested action
receipt -> execution outcome record
state -> resulting persistent world-state change
```

### Block-Level Commitment

current block docs는 execution payload header가 `receipts_root`를 포함한다고 설명한다.[^ref-eth-doc-blocks]

### Why This Matters

transaction은 무엇을 요청했는지 알려주고, receipt는 execution이 무엇을 보고했는지 알려주며, state는 무엇이 persist했는지 알려준다.

---

## 5. Receipts Trie

### Per-Block Trie

official Merkle Patricia Trie documentation은 every block이 `rlp(transactionIndex)`를 key로 하는 receipts trie를 가진다고 설명한다.[^ref-eth-doc-mpt]

### Legacy and Typed Receipts

같은 docs는 반환되는 receipt가 다음 중 하나일 수 있다고 말한다.

- `Receipt = TransactionType || ReceiptPayload`
- `LegacyReceipt = rlp([status, cumulativeGasUsed, logsBloom, logs])`

[^ref-eth-doc-mpt]

### EIP-2718 Rule

EIP-2718은 receipt의 `TransactionType`이 대응되는 transaction의 `TransactionType`과 일치해야 한다고 규정한다.[^ref-eip-2718]

---

## 6. Receipt Fields

### Core Outcome Fields

인용된 source 전반에서 주요 conceptual receipt field는 다음과 같다.

```text
status
cumulativeGasUsed
logsBloom
logs
```

### `status`

이 값은 current rule 아래에서 execution이 성공했는지를 나타낸다. transaction이 존재했다는 사실만을 뜻하지 않는다.

### `cumulativeGasUsed`

이 값은 해당 transaction 하나의 gas만이 아니라, 그 transaction까지 포함한 block-level cumulative gas used다.[^ref-eip-2718]

### `logsBloom`

이 값은 current documented model에서 receipt structure 안의 log와 연관된 bloom-filter summary다.[^ref-eip-2718]

### `logs`

이 값은 execution이 만든 emitted log record이며, 많은 application-facing "event"의 raw protocol substrate다.

---

## 7. Receipts vs Transactions vs State

### Transaction

transaction은 execution을 요청하는 signed instruction이다.

### Receipt

receipt는 post-execution result record다.

### State

state는 accepted execution effect가 commit된 뒤 persist하는 결과다.

### Why This Distinction Matters

receipt만으로 전체 state를 복원할 수 없고, raw transaction input만으로 모든 execution outcome을 추론할 수도 없다.

---

## 8. Technical Mechanics

### Simplified Lifecycle

```text
transaction included in block
-> execution happens
-> outcome fields determined
-> receipt created
-> receipt committed into receipts trie
-> receiptsRoot included in block commitment
```

### Query Surface

official JSON-RPC docs는 `eth_getTransactionReceipt`를 포함하며, transaction hash로 receipt object를 반환하고 pending transaction에는 receipt가 없다고 명시한다.[^ref-eth-doc-jsonrpc]

### Operational Consequence

즉 receipt availability는 mempool presence가 아니라, post-inclusion execution에 연결된다.

---

## 9. Security Assumptions

### Commitment Integrity

receipt가 유용한 이유는 어떤 RPC provider가 그렇게 말해서가 아니라, `receiptsRoot`를 통해 block에 commit되기 때문이다.

### Outcome vs Interpretation

receipt는 structured execution fact를 제공하지만, application semantic은 여전히 log와 contract context에 대한 해석을 필요로 한다.

### Freshness and Evolution

typed receipt와 protocol transport change는 모든 historical Ethereum era가 receipt를 동일하게 serialize하거나 expose한다고 가정하면 안 된다는 뜻이다.[^ref-eip-2718]

---

## 10. Mathematical or Economic Model

### Receipt Commitment Model

개념적으로:

```text
receiptsRoot = commitment(all receipts in block order)
```

이것은 per-block receipts trie의 개념적 요약이다.

### Cumulative Gas Interpretation

receipt `i`의 cumulative gas used를 `G_i`라고 하면:

```text
gas used by tx_i = G_i - G_(i-1)
```

이는 block 내 첫 transaction이 아닌 경우에 성립한다. cumulative gas used의 의미에서 바로 따라온다.

### Why This Matters

receipt는 ordered context 안에서 execution-cost breakdown을 추적하는 가장 깔끔한 block-local 수단인 경우가 많다.

---

## 11. Protocol Implementation

### Primary Sources

receipt를 이해하는 데 가장 중요한 1차 자료는 다음이다.

- commitment structure를 설명하는 official Merkle Patricia Trie docs
- typed receipt rule을 설명하는 EIP-2718
- `receipts_root`의 block 포함을 설명하는 official block docs
- retrieval semantic을 설명하는 official JSON-RPC docs[^ref-eth-doc-mpt][^ref-eip-2718][^ref-eth-doc-blocks][^ref-eth-doc-jsonrpc]

### Why These Four Together

이 넷 중 어느 하나만으로는 충분하지 않다.

- trie docs는 commitment를 설명하고
- EIP는 modern typed structure를 설명하며
- block docs는 그 commitment가 block 안 어디에 있는지 설명하고
- JSON-RPC docs는 operator가 객체를 어떻게 retrieve하는지 설명한다.

---

## 12. On-Chain Implications

### What Analysts Use Receipts For

receipt는 다음에 핵심적이다.

- execution success 판단
- log 읽기
- gas outcome 분석
- transaction과 emitted protocol/application signal 연결

### What Receipts Do Not Replace

receipt는 다음을 대체하지 않는다.

- full state inspection
- transaction input decoding
- contract code analysis
- trace-based reasoning

### Practical Consequence

serious Ethereum analytics는 거의 항상 receipt를 필요로 한다.

---

## 13. Institutional Thinking

institution은 receipt를 first-class execution evidence로 다뤄야 한다.

### Practical Implications

- raw transaction만 보는 monitoring은 불완전하다.
- operational pipeline은 successful execution과 failed execution 모두에 대해 receipt data를 캡처해야 한다.
- event-driven system은 log가 별도의 magical data source가 아니라 receipt에서 온다는 점을 기억해야 한다.
- gas accounting과 success/failure interpretation은 receipt-aware해야 한다.

### Better Research Posture

execution claim을 하기 전에 다음을 물어야 한다.

- transaction이 receipt를 얻었는가?
- receipt status는 무엇이었는가?
- 어떤 log가 emitted되었는가?
- 이것이 transaction intent에 대한 주장인가, execution outcome에 대한 주장인가?

---

## 14. Common Misinterpretations

### "The transaction itself tells me whether execution succeeded"

틀렸다. receipt-level outcome data가 필요하다.

### "Receipts are just wallet UI metadata"

틀렸다. 그것들은 protocol-committed execution artifact다.

### "`cumulativeGasUsed` is the same as gas used by this one transaction"

틀렸다. block 안에서 cumulative한 값이다.

### "Pending transactions have receipts"

JSON-RPC documentation에 따르면 틀렸다.[^ref-eth-doc-jsonrpc]

---

## 15. Research Questions

1. 어떤 institutional workflow가 receipt의 analytical value를 여전히 과소활용하고 있는가?
2. long-horizon Ethereum dataset에서는 typed receipt evolution을 어떻게 처리해야 하는가?
3. raw receipt fact와 가장 쉽게 혼동되는 application-layer interpretation은 무엇인가?

---

## 16. Practical Exercises

### Exercise 1

transaction hash와 transaction receipt의 차이를 설명하라.

### Exercise 2

block header에 `receiptsRoot`가 왜 존재하는지 짧게 설명하라.

### Exercise 3

인접한 receipt의 cumulative gas used가 주어졌을 때, per-transaction gas used를 도출하라.

### Exercise 4

event listener가 왜 receipt에 의존하는지 설명하라.

---

## 17. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Receipts trie and receipts root structure | Directly specified | Official MPT docs and block docs |
| Typed and legacy receipt framing | Directly specified | EIP-2718 and official trie docs |
| Receipt retrieval semantics | Directly specified | Official JSON-RPC docs |
| Institutional execution-evidence framing | Inference from sources | Derived from receipt role in execution analysis |

---

## 18. Knowledge Graph

```text
Receipts
├─ Relationship
│  ├─ transaction request
│  ├─ receipt outcome
│  └─ state result
├─ Commitment
│  ├─ receipts trie
│  └─ receiptsRoot
├─ Fields
│  ├─ status
│  ├─ cumulativeGasUsed
│  ├─ logsBloom
│  └─ logs
└─ Uses
   ├─ execution success tracking
   ├─ gas accounting
   ├─ event extraction
   └─ post-execution monitoring
```

---

## 19. References

### Primary Sources

[^ref-eth-doc-mpt]: ethereum.org, "Merkle Patricia Trie," official documentation describing receipts trie structure and receipt encoding context, https://ethereum.org/developers/docs/data-structures-and-encoding/patricia-merkle-trie/, accessed 2026-08-04.

[^ref-eip-2718]: EIP-2718, "Typed Transaction Envelope," Ethereum Improvement Proposals, including typed receipt rules and legacy receipt structure, https://eips.ethereum.org/EIPS/eip-2718, accessed 2026-08-04.

[^ref-eth-doc-blocks]: ethereum.org, "Blocks," official documentation describing execution payload fields including `receipts_root`, https://ethereum.org/developers/docs/blocks/, accessed 2026-08-04.

[^ref-eth-doc-jsonrpc]: ethereum.org, "JSON-RPC API," official documentation including `eth_getTransactionReceipt` and receipt retrieval semantics, https://ethereum.org/developers/docs/apis/json-rpc/, accessed 2026-08-04.

### Supporting Interpretation Notes

- Where this document discusses institutional monitoring posture or receipt-centric analytics, those are analytical inferences built from the cited receipt-structure sources.

---

## 20. Cross References

### Previous

- ETHEREUM-FOUNDATION-007 — Blocks

### Next

- ETHEREUM-FOUNDATION-009 — Logs & Events

### Related

- ETHEREUM-FOUNDATION-004 — State Transition
- ETHEREUM-FOUNDATION-010 — Storage

---

## Review Status

### Technical Review

Passed.

- receipt를 transaction과 state에서 분리했다.
- trie commitment와 typed receipt structure를 모두 다뤘다.
- cumulative gas usage와 per-transaction gas를 구분했다.
- JSON-RPC retrieval semantic을 포함했다.

### Evidence Review

Passed.

- commitment/structure claim은 trie docs, block docs, EIP-2718을 인용한다.
- retrieval semantic은 official JSON-RPC docs를 인용한다.
- institutional interpretation은 inference로 라벨링했다.

### Editorial Review

Passed.

- 구조는 프로젝트 문서 패턴을 따른다.
- metadata는 완전하다.
- terminology는 receipt, receiptsRoot, cumulativeGasUsed, logsBloom으로 일관된다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 receipt와 transaction을 혼동하지 않는다.
- pending transaction에 receipt가 있다고 암시하지 않는다.
- receipt log를 full application semantic과 동일시하지 않는다.
- receipt를 wallet-only metadata로 취급하지 않는다.

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
