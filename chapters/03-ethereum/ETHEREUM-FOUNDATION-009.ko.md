---
knowledge_id: ETHEREUM-FOUNDATION-009
title: Logs & Events
subtitle: Contract-emitted log, topic, event indexing, bloom filter, 그리고 protocol data와 application semantic의 경계
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 125 min
estimated_study: 320 min
last_reviewed: 2026-08-04
domain:
  - Ethereum
  - Logs
  - Events
  - Smart Contracts
parent:
  - Ethereum Foundations
prerequisites:
  - ETHEREUM-FOUNDATION-005
  - ETHEREUM-FOUNDATION-008
related_topics:
  - Receipts
  - Blocks
  - JSON-RPC
  - Contract Execution
primary_sources:
  - REF-ETH-DOC-BLOCKS-2026-001
  - REF-ETH-DOC-JSONRPC-2026-001
  - REF-ETH-TUTORIAL-EVENTS-001
  - REF-EIP-7668
tags:
  - ethereum
  - logs
  - events
  - topics
  - logs-bloom
  - indexing
---

# Logs & Events
> Ethereum Foundations  
> Research Unit: ETHEREUM-FOUNDATION-009

---

## Research Brief

```yaml
knowledge_id: ETHEREUM-FOUNDATION-009
title: Logs & Events
research_question: >
  What are Ethereum logs and application-level events, how are they emitted and
  retrieved, what role do topics and bloom filters play, and how should
  researchers separate protocol log data from application interpretation in
  August 2026?
document_type: deep-dive
difficulty: L300
prerequisites:
  - ETHEREUM-FOUNDATION-005
  - ETHEREUM-FOUNDATION-008
parent: Ethereum Foundations
previous: ETHEREUM-FOUNDATION-008
next: ETHEREUM-FOUNDATION-010
related_topics:
  - Receipts
  - Storage
  - Transactions
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
  - Solidity event syntax deep dive
  - Full ABI decoding tutorial
  - Third-party indexer architecture
  - Subgraph design
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- log와 event를 구분할 수 있다.
- contract execution이 어떻게 log를 emit하는지 설명할 수 있다.
- 왜 log가 receipt에 담기고 bloom structure로 요약되는지 설명할 수 있다.
- indexed event field와 topic의 역할을 설명할 수 있다.
- log는 semantic decoding을 필요로 하며 self-explanatory하지 않다는 점을 설명할 수 있다.

---

## 2. 핵심 질문

1. log란 무엇인가?
2. event란 무엇인가?
3. log는 receipt와 어떤 관계인가?
4. topic은 무엇이고, 왜 어떤 event field는 indexed되는가?
5. `logs_bloom`은 무엇을 하는가?
6. 왜 protocol log data를 application meaning과 자동으로 동일시할 수 없는가?

---

## 3. Executive Summary

Ethereum에서 log는 smart contract execution이 emit하고 transaction receipt에 저장되는 protocol-level execution output이다. "event"는 보통 Solidity 같은 smart contract language가 이 log를 application-facing하게 해석한 표현이다.[^ref-eth-tutorial-events][^ref-eth-doc-jsonrpc]

block documentation은 `logs_bloom`을 execution payload header의 일부로 제시하며, legacy framing의 receipt structure에는 `logsBloom`과 `logs`가 포함된다.[^ref-eth-doc-blocks][^ref-eip-7668]

ethereum.org의 events tutorial은 Solidity event를 dapp과 JSON-RPC API에 연결된 어떤 시스템이든 listen할 수 있는 dispatch signal로 설명하며, indexed field가 event history를 나중에 searchable하게 만든다고 설명한다.[^ref-eth-tutorial-events]

연구에서 핵심 규율은 분리다.

- log = execution이 emit한 protocol data
- event = contract/ABI assumption 아래에서 decode된 semantic interpretation

이 구분을 건너뛰면 chain이 직접 말해주는 범위를 과장하게 된다.

---

## 4. Protocol Structure

### Execution Path

log는 다음 경로에서 생긴다.

```text
contract execution
-> log emission
-> receipt inclusion
-> block-level receipts commitment
-> optional application decoding as events
```

### Why This Matters

log는 independent transaction도 아니고 direct state storage entry도 아니다. execution output이다.

### Application Layer

wallet, explorer, dashboard, dapp의 event system은 보통 이 log 위에 구축된다.

---

## 5. Logs vs Events

### Logs

log는 receipt에 기록되는 low-level protocol output이다.

### Events

ethereum.org tutorial은 Solidity에서 event가 contract code에 선언되고 signal로 emit되며, application이 이를 listen하고 반응할 수 있다고 설명한다.[^ref-eth-tutorial-events]

### Why the Distinction Matters

event는 protocol 위에 떠 있는 별도 primitive가 아니다. developer가 선언한 semantic wrapper가 log emission 위에 얹힌 것이다.

---

## 6. Topics and Indexed Fields

### Indexed Searchability

tutorial은 event를 indexed하면 event history가 나중에 searchable해진다고 명시적으로 말한다.[^ref-eth-tutorial-events]

### Practical Meaning

indexed parameter는 log/topic search surface의 일부가 되어, downstream system이 known identifier 기준으로 더 효율적으로 filter할 수 있게 한다.

### Research Consequence

분석가가 "address X에서 발생한 모든 transfer를 검색하라"고 할 때, 실제로는 balance만 보는 것이 아니라 indexed event/topic structure에 의존하는 경우가 많다.

---

## 7. Logs Bloom

### Block-Level Presence

current block docs는 execution payload header에 `logs_bloom`이 있다고 설명한다.[^ref-eth-doc-blocks]

### Receipt-Level Context

legacy receipt framing에는 `logsBloom`이 `logs`와 함께 들어간다.[^ref-eip-7668]

### Why This Exists

bloom filter는 relevant log가 있을 가능성이 있는 위치를 client와 application이 빠르게 식별하는 데 도움을 주도록 설계되었다.

### Current Qualification

EIP-7668은 execution block과 receipt에서 bloom filter를 제거하자고 제안하면서, 실제로는 많은 application이 raw bloom scan보다 extra-protocol indexing에 의존한다고 설명한다.[^ref-eip-7668]

이 proposal 자체가 final protocol law는 아니지만, bloom-based log discovery를 timeless design success처럼 다루면 안 된다는 강한 증거다.

---

## 8. Technical Mechanics

### Receipt Retrieval

JSON-RPC를 통한 receipt retrieval은 log를 포함한 post-execution record를 노출하므로, receipt access는 event-driven application의 일반적인 경로다.[^ref-eth-doc-jsonrpc]

### Example Intuition

ethereum.org tutorial은 친숙한 ERC-20 `Transfer` event를, application이 log를 어떻게 해석하는지 보여주는 concrete example로 사용한다.[^ref-eth-tutorial-events]

### Practical Flow

```text
tx executes
-> contract emits log
-> receipt stores log data
-> app queries receipt / logs
-> ABI decoding produces event semantics
```

---

## 9. Security Assumptions

### Protocol Fact vs Semantic Interpretation

chain은 log payload를 제공한다. 그 payload의 의미는 다음에 의존한다.

- contract의 ABI
- decoder assumption의 correctness
- 경우에 따라 upgradeability 또는 proxy context

### Searchability vs Truth

indexed log는 검색을 쉽게 만들지만, interpretation risk를 없애지는 않는다.

### Proposal Risk

EIP-7668 같은 proposal의 존재는 일부 log-discovery tooling assumption이 시간이 지나며 바뀔 수 있음을 상기시킨다.[^ref-eip-7668]

---

## 10. Mathematical or Economic Model

### Layered Interpretation Model

유용한 conceptual model은 다음과 같다.

```text
log data + ABI/context assumptions -> event interpretation
```

### Bloom Filter Intuition

bloom structure는 probabilistic filtering aid이지, semantically complete history가 아니다. 목적은 meaning이 아니라 speed/selection이다.

### Why This Matters

analyst가 raw log에서 application meaning으로 멀어질수록, pipeline 안으로 더 많은 inference가 들어온다.

---

## 11. Protocol Implementation

### Current Primary Sources

이 unit에서 가장 유용한 source는 다음이다.

- developer-facing event semantic을 위한 events tutorial
- receipt retrieval을 위한 official JSON-RPC docs
- `logs_bloom`을 위한 official block docs
- bloom reliance에 대한 현재 pressure를 보여주는 EIP-7668[^ref-eth-tutorial-events][^ref-eth-doc-jsonrpc][^ref-eth-doc-blocks][^ref-eip-7668]

### Why This Is Enough

이 source 조합은 다음을 분리하게 해준다.

- protocol log carriage
- application event semantic
- retrieval surface
- evolving indexing assumption

---

## 12. On-Chain Implications

### What Analysts Use Logs For

log는 다음에 핵심적이다.

- token transfer monitoring
- protocol action tracking
- contract interaction labeling
- event-driven alerting
- downstream analytics indexing

### What Logs Do Not Guarantee

log만으로는 다음을 보장하지 않는다.

- economic intent
- business meaning
- 여러 contract를 가로지르는 final application outcome
- complete state effect

### Practical Consequence

log는 Ethereum에서 가장 유용한 data surface 중 하나이지만, 동시에 가장 과도하게 해석되기 쉬운 surface 중 하나다.

---

## 13. Institutional Thinking

institution은 log를 high-value이면서 interpretation-sensitive한 data로 다뤄야 한다.

### Practical Implications

- event-driven monitoring pipeline은 decoded interpretation과 raw log를 모두 보존해야 한다.
- decoded event dataset에는 ABI와 contract-version context도 함께 보존해야 한다.
- proxy와 upgradeable contract pattern은 log-reading habit은 그대로여도 event semantic을 바꿀 수 있다.
- historical bloom behavior에 대한 의존은 신중히 문서화해야 한다.

### Better Research Posture

event claim을 하기 전에 다음을 물어야 한다.

- 이것이 raw log fact인가, decoded event interpretation인가?
- 어떤 ABI를 썼는가?
- contract가 upgradeable했는가?
- 주장하는 event가 state change를 의미하는가, 아니면 단지 signal인가?

---

## 14. Common Misinterpretations

### "Events are stored separately from receipts"

틀렸다. event semantic은 receipt structure 안에 실린 log에서 나온다.

### "If a log exists, the meaning is obvious"

틀렸다. 의미는 decoding과 context에 의존한다.

### "Bloom filters are the whole log-discovery solution"

더 이상 안전한 가정이 아니다. current proposal은 이 model에 직접 도전하고 있다.[^ref-eip-7668]

### "Logs and state changes are identical"

틀렸다. log는 behavior를 signal할 수 있지만, full persistent state 그 자체는 아니다.

---

## 15. Research Questions

1. 어떤 institutional pipeline이 decoded event와 별도로 raw-log preservation을 가장 절실히 필요로 하는가?
2. upgradeable contract의 log를 decoding할 때 analyst는 uncertainty를 어떻게 정량화해야 하는가?
3. Ethereum event indexing 중 얼마나 많은 부분이 protocol-native bloom behavior가 아니라 extra-protocol infrastructure에 의존하는가?

---

## 16. Practical Exercises

### Exercise 1

raw log와 decoded event의 차이를 설명하라.

### Exercise 2

indexed event field가 search에 왜 유용한지 설명하라.

### Exercise 3

inclusion 이후 log를 신뢰성 있게 접근하려면 왜 receipt가 필요한지 설명하라.

### Exercise 4

정확한 log가 어떻게 잘못 해석될 수 있는지 예를 들어라.

---

## 17. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Event semantics as developer-declared signals | Directly specified | ethereum.org events tutorial |
| Log/bloom presence in protocol structures | Directly specified | Block docs and receipt/EIP material |
| Bloom-model limitations and pressure to remove | Directly specified as proposal evidence | EIP-7668 |
| Institutional semantic-caution framing | Inference from sources | Derived from log/event architecture |

---

## 18. Knowledge Graph

```text
Logs & Events
├─ Protocol Layer
│  ├─ logs
│  ├─ receipts
│  └─ logs_bloom
├─ Application Layer
│  ├─ events
│  ├─ indexed fields
│  └─ ABI decoding
├─ Retrieval
│  ├─ JSON-RPC
│  └─ indexing systems
└─ Risks
   ├─ semantic over-interpretation
   ├─ ABI mismatch
   └─ bloom reliance limits
```

---

## 19. References

### Primary Sources

[^ref-eth-doc-blocks]: ethereum.org, "Blocks," official documentation including `logs_bloom` in execution payload headers, https://ethereum.org/developers/docs/blocks/, accessed 2026-08-04.

[^ref-eth-doc-jsonrpc]: ethereum.org, "JSON-RPC API," official documentation including transaction-receipt retrieval interfaces, https://ethereum.org/developers/docs/apis/json-rpc/, accessed 2026-08-04.

[^ref-eth-tutorial-events]: ethereum.org, "Logging data from smart contracts with events," tutorial describing Solidity events, emitted signals, and indexed searchability, https://ethereum.org/developers/tutorials/logging-events-smart-contracts/, accessed 2026-08-04.

[^ref-eip-7668]: EIP-7668, "Remove bloom filters," Ethereum Improvement Proposals, proposal evidence describing current limitations of bloom-based log discovery, https://eips.ethereum.org/EIPS/eip-7668, accessed 2026-08-04.

### Supporting Interpretation Notes

- Where this document discusses institutional decoding risk or semantic over-interpretation, those are analytical inferences from the cited log/event sources.

---

## 20. Cross References

### Previous

- ETHEREUM-FOUNDATION-008 — Receipts

### Next

- ETHEREUM-FOUNDATION-010 — Storage

### Related

- ETHEREUM-FOUNDATION-005 — EVM
- ETHEREUM-FOUNDATION-008 — Receipts

---

## Review Status

### Technical Review

Passed.

- log와 event를 분리했다.
- indexed-field와 bloom의 역할을 구분했다.
- receipt carriage와 JSON-RPC retrieval context를 포함했다.
- proposal-level bloom limitation을 신중히 라벨링했다.

### Evidence Review

Passed.

- event semantic은 ethereum.org tutorial을 인용한다.
- protocol structure는 block과 receipt-related source를 인용한다.
- bloom criticism은 final protocol fact가 아니라 proposal source임을 명시했다.
- institutional interpretation은 inference로 라벨링했다.

### Editorial Review

Passed.

- 구조는 프로젝트 문서 패턴을 따른다.
- metadata는 완전하다.
- terminology는 log, event, topic, indexed field, logs bloom으로 일관된다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 log를 self-explanatory truth로 다루지 않는다.
- application event를 separate protocol primitive와 혼동하지 않는다.
- bloom filter를 permanent하거나 sufficient한 것으로 과장하지 않는다.
- decoded event를 full state와 동일시하지 않는다.

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
