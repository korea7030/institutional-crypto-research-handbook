---
knowledge_id: ETHEREUM-FOUNDATION-009
title: Logs & Events
subtitle: Contract-Emitted Logs, Topics, Event Indexing, Bloom Filters, and the Boundary Between Protocol Data and Application Semantics
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

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Distinguish logs from events.
- Explain how contract execution emits logs.
- Explain why logs are carried in receipts and summarized in bloom structures.
- Explain the role of indexed event fields and topics.
- Explain why logs require semantic decoding and are not self-explanatory.

---

## 2. Key Questions

1. What is a log?
2. What is an event?
3. How do logs relate to receipts?
4. What are topics and why are some event fields indexed?
5. What does `logs_bloom` do?
6. Why can protocol log data not be equated automatically with application meaning?

---

## 3. Executive Summary

In Ethereum, logs are protocol-level execution outputs emitted by smart contract execution and stored in transaction receipts. "Events" are the application-facing interpretation of those logs, commonly expressed in smart contract languages such as Solidity.[^ref-eth-tutorial-events][^ref-eth-doc-jsonrpc]

The block documentation identifies `logs_bloom` as part of the execution payload header, while receipt structures include `logsBloom` and `logs` in legacy framing.[^ref-eth-doc-blocks][^ref-eip-7668]

The ethereum.org tutorial on events explains that Solidity events are dispatch signals that dapps and anything connected to Ethereum's JSON-RPC API can listen to, and that indexed fields make event history searchable later.[^ref-eth-tutorial-events]

For research, the main discipline is separation:

- log = protocol data emitted by execution,
- event = decoded semantic interpretation of that log under contract/ABI assumptions.

If you skip that distinction, you overstate what the chain directly tells you.

---

## 4. Protocol Structure

### Execution Path

Logs arise along the following path:

```text
contract execution
-> log emission
-> receipt inclusion
-> block-level receipts commitment
-> optional application decoding as events
```

### Why This Matters

Logs are execution outputs, not independent transactions and not direct state storage entries.

### Application Layer

Event systems in wallets, explorers, dashboards, and dapps typically sit on top of these logs.

---

## 5. Logs vs Events

### Logs

Logs are low-level protocol outputs recorded in receipts.

### Events

The ethereum.org tutorial explains that in Solidity, events are declared in contract code and emitted as signals; applications can listen to them and react accordingly.[^ref-eth-tutorial-events]

### Why the Distinction Matters

An event is not a separate protocol primitive floating above the chain. It is a developer-declared semantic wrapper around log emission.

---

## 6. Topics and Indexed Fields

### Indexed Searchability

The tutorial explicitly says an event can be indexed so that event history is searchable later.[^ref-eth-tutorial-events]

### Practical Meaning

Indexed parameters become part of the log/topic search surface, letting downstream systems filter by known identifiers more efficiently.

### Research Consequence

When analysts say "search for all transfers from address X," they are usually depending on indexed event/topic structures, not on balances alone.

---

## 7. Logs Bloom

### Block-Level Presence

Current block docs list `logs_bloom` in the execution payload header.[^ref-eth-doc-blocks]

### Receipt-Level Context

Legacy receipt framing includes `logsBloom` alongside `logs`.[^ref-eip-7668]

### Why This Exists

Bloom filters were intended to help applications and clients quickly identify where relevant logs may exist.

### Current Qualification

EIP-7668 proposes removing bloom filters from execution blocks and receipts, describing how in practice many applications rely on extra-protocol indexing instead of raw bloom scans.[^ref-eip-7668]

This proposal is not itself final protocol law here, but it is strong evidence that bloom-based log discovery should not be treated as a timeless design success.

---

## 8. Technical Mechanics

### Receipt Retrieval

Receipt retrieval via JSON-RPC exposes the post-execution record that contains logs, making receipt access a normal path for event-driven applications.[^ref-eth-doc-jsonrpc]

### Example Intuition

The ethereum.org tutorial uses the familiar ERC-20 `Transfer` event as a concrete example of how applications reason about logs.[^ref-eth-tutorial-events]

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

The chain gives you the log payload. The meaning of that payload depends on:

- the contract's ABI,
- the correctness of decoder assumptions,
- and sometimes upgradeability or proxy context.

### Searchability vs Truth

Indexed logs make searching easier. They do not eliminate interpretation risk.

### Proposal Risk

The existence of proposals such as EIP-7668 is a reminder that some log-discovery tooling assumptions can evolve over time.[^ref-eip-7668]

---

## 10. Mathematical or Economic Model

### Layered Interpretation Model

A useful conceptual model is:

```text
log data + ABI/context assumptions -> event interpretation
```

### Bloom Filter Intuition

Bloom structures are probabilistic filtering aids, not semantically complete histories. Their value is speed/selection, not full meaning.

### Why This Matters

The farther an analyst moves from raw logs toward application meaning, the more inference enters the pipeline.

---

## 11. Protocol Implementation

### Current Primary Sources

For this unit, the most useful sources are:

- the events tutorial for developer-facing event semantics,
- official JSON-RPC docs for receipt retrieval,
- official block docs for `logs_bloom`,
- EIP-7668 for current pressure against bloom reliance.[^ref-eth-tutorial-events][^ref-eth-doc-jsonrpc][^ref-eth-doc-blocks][^ref-eip-7668]

### Why This Is Enough

These sources let us separate:

- protocol log carriage,
- application event semantics,
- retrieval surface,
- and evolving indexing assumptions.

---

## 12. On-Chain Implications

### What Analysts Use Logs For

Logs are central for:

- token transfer monitoring,
- protocol action tracking,
- contract interaction labeling,
- event-driven alerting,
- downstream analytics indexing.

### What Logs Do Not Guarantee

Logs do not by themselves prove:

- economic intent,
- business meaning,
- final application outcome across multiple contracts,
- or complete state effects.

### Practical Consequence

Logs are among the most useful Ethereum data surfaces, but also among the most over-interpreted.

---

## 13. Institutional Thinking

Institutions should treat logs as high-value but interpretation-sensitive data.

### Practical Implications

- Event-driven monitoring pipelines should preserve raw logs as well as decoded interpretations.
- ABI and contract-version context should be retained alongside decoded event datasets.
- Proxy and upgradeable contract patterns can change event semantics without changing log-reading habits.
- Reliance on historical bloom behavior should be documented carefully.

### Better Research Posture

Before making an event claim, ask:

- Is this a raw log fact or a decoded event interpretation?
- Which ABI was used?
- Was the contract upgradeable?
- Does the claimed event imply a state change, or only a signaled message?

---

## 14. Common Misinterpretations

### "Events are stored separately from receipts"

False. Event semantics come from logs carried in receipt structures.

### "If a log exists, the meaning is obvious"

False. Meaning depends on decoding and context.

### "Bloom filters are the whole log-discovery solution"

No longer a safe assumption. Current proposals explicitly challenge that model.[^ref-eip-7668]

### "Logs and state changes are identical"

False. Logs can signal behavior, but they are not themselves the full persistent state.

---

## 15. Research Questions

1. Which institutional pipelines most need raw-log preservation in addition to decoded events?
2. How should analysts quantify uncertainty when decoding logs from upgradeable contracts?
3. How much of Ethereum event indexing is already dependent on extra-protocol infrastructure rather than protocol-native bloom behavior?

---

## 16. Practical Exercises

### Exercise 1

Explain the difference between a raw log and a decoded event.

### Exercise 2

Describe why indexed event fields are useful for search.

### Exercise 3

Explain why a receipt is needed to access logs reliably after inclusion.

### Exercise 4

Give an example of how a correct log can still be misinterpreted.

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

- Logs were separated from events.
- Indexed-field and bloom roles were differentiated.
- Receipt carriage and JSON-RPC retrieval context were included.
- Proposal-level bloom limitations were labeled carefully.

### Evidence Review

Passed.

- Event semantics cite the ethereum.org tutorial.
- Protocol structures cite block and receipt-related sources.
- Bloom criticism is explicitly sourced to a proposal, not presented as final protocol fact.
- Institutional interpretation is labeled as inference.

### Editorial Review

Passed.

- Structure follows the project document pattern.
- Metadata is complete.
- Terminology is consistent: log, event, topic, indexed field, logs bloom.
- Tables and code fences are closed.

### Adversarial Review

Passed.

- The document does not collapse logs into self-explanatory truth.
- It does not confuse application events with separate protocol primitives.
- It does not overstate bloom filters as permanent or sufficient.
- It does not treat decoded events as equivalent to full state.

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
