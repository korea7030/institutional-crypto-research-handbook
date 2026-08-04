---
knowledge_id: BITCOIN-033
title: Bitcoin Core
subtitle: Reference Implementation, Validation Engine, Policy Node, Wallet Stack, and the Boundary Between Consensus and Local Operation
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 155 min
estimated_study: 450 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Bitcoin Core
  - Node Software
  - Validation
parent:
  - Modern Bitcoin
prerequisites:
  - BITCOIN-014
  - BITCOIN-017
  - BITCOIN-021
  - BITCOIN-022
  - POW-013
related_topics:
  - Validation
  - Chainstate
  - Mempool
  - RPC
  - Wallet
  - AssumeUTXO
primary_sources:
  - REF-BTC-CORE-FILES-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-VALIDATIONINTERFACE-001
  - REF-BTC-CORE-NET-PROCESSING-001
  - REF-BTC-CORE-TXMEMPOOL-001
  - REF-BTC-CORE-ASSUMEUTXO-001
  - REF-BTC-CORE-31-RELEASE-001
tags:
  - bitcoin
  - bitcoin-core
  - node
  - validation
  - chainstate
  - mempool
  - rpc
  - wallet
---

# Bitcoin Core
> Modern Bitcoin  
> Research Unit: BITCOIN-033

---

## Research Brief

```yaml
knowledge_id: BITCOIN-033
title: Bitcoin Core
research_question: >
  What is Bitcoin Core as the dominant Bitcoin full-node implementation, how do
  its validation, networking, mempool, wallet, and RPC subsystems interact, and
  why must analysts separate consensus rules from local node policy,
  implementation details, and operational configuration?
document_type: deep-dive
difficulty: L400
prerequisites:
  - BITCOIN-014
  - BITCOIN-017
  - BITCOIN-021
  - BITCOIN-022
  - POW-013
parent: Modern Bitcoin
previous: BITCOIN-032
next: BITCOIN-034
related_topics:
  - Validation
  - Chainstate
  - Mempool
  - Wallet
  - RPC
  - Indexes
  - AssumeUTXO
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
  - Full contributor guide for building Bitcoin Core from source
  - Exhaustive RPC catalog
  - Detailed wallet descriptor tutorial
  - Line-by-line source walkthrough of every subsystem
  - Non-Core client comparison survey
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Explain what Bitcoin Core is and what it is not.
- Distinguish consensus validation from local relay and mining policy.
- Identify the major Bitcoin Core subsystems: validation, chainstate, mempool, networking, wallet, RPC, and indexes.
- Explain why Bitcoin Core is implementation evidence, not protocol authority by decree.
- Explain how configuration choices such as pruning, indexes, and wallet usage change node behavior without changing Bitcoin consensus.
- Interpret mempool, wallet, and RPC outputs as node-local views rather than global truths.

---

## 2. Key Questions

1. What is Bitcoin Core?
2. Why is Bitcoin Core often treated as the reference implementation?
3. Which parts of Bitcoin behavior are consensus rules and which parts are local policy?
4. How do validation, chainstate, mempool, and networking interact?
5. What role do wallet, RPC, and indexes play?
6. How do pruning and AssumeUTXO affect operations and analysis?
7. What can analysts infer from a Bitcoin Core node, and what remains only local implementation state?

---

## 3. Executive Summary

Bitcoin Core is the most widely used full-node implementation in Bitcoin and functions as the ecosystem's dominant operational reference for block and transaction validation, peer-to-peer networking, mempool behavior, wallet management, and RPC interfaces.[^ref-btc-core-files][^ref-btc-core-validation]

It is not the Bitcoin protocol itself. Consensus is defined by the rules that nodes enforce over blocks and transactions, not by social deference to a specific repository. But because Bitcoin Core is the primary implementation used by many operators, exchanges, researchers, and infrastructure providers, its source tree is a critical primary source for understanding how modern Bitcoin behaves in practice.[^ref-btc-core-validation][^ref-btc-core-net-processing]

Bitcoin Core combines several distinct roles:

- full-node validation engine,
- peer-to-peer relay participant,
- mempool and mining-policy engine,
- wallet and key-management software,
- RPC and data-serving surface,
- optional indexing and operational tooling layer.[^ref-btc-core-files][^ref-btc-core-txmempool]

For analysis, the central discipline is separation. A Bitcoin Core node exposes a local viewpoint shaped by software version, configuration, peer set, indexes, pruning mode, wallet state, and policy parameters. Analysts must not confuse that local viewpoint with global network truth.[^ref-btc-core-net-processing][^ref-btc-core-31-release]

---

## 4. Protocol Structure

### Bitcoin Core as a Layered System

At a high level, Bitcoin Core can be understood as a stack:

```text
user / operator
-> CLI, GUI, RPC, config
-> wallet and indexes
-> mempool and policy
-> validation and chainstate
-> p2p networking
-> disk, database, and data directory
```

### Core Roles

| Role | What Bitcoin Core Does |
|---|---|
| Consensus validation | verifies blocks, transactions, scripts, subsidy constraints, and context rules |
| Chainstate maintenance | updates and queries the active UTXO set and block index |
| Relay node | exchanges blocks, headers, transactions, and peer metadata |
| Policy node | applies local mempool, standardness, anti-DoS, and mining-template rules |
| Wallet stack | manages keys, descriptors, balances, and signing |
| RPC server | exposes operational and programmatic interfaces |
| Indexing / storage | stores block files, indexes, chain metadata, and optional derived views |

### What Bitcoin Core Is Not

Bitcoin Core is not:

- a central coordinator,
- the only valid Bitcoin implementation,
- a guarantee that every local policy choice is universal,
- or a perfect mirror of global network state.

---

## 5. Major Components

### Binaries and Operator Surface

Bitcoin Core's source-tree overview documents the main binaries, including `bitcoind`, `bitcoin-qt`, `bitcoin-cli`, and `bitcoin-wallet`.[^ref-btc-core-files]

At an operational level:

- `bitcoind` is the daemon most institutions and infrastructure operators run.
- `bitcoin-cli` is the common control surface for RPC interaction.
- `bitcoin-qt` packages the node with a desktop GUI.
- `bitcoin-wallet` supports wallet-file management tasks.[^ref-btc-core-files]

### Validation and Chainstate

`validation.cpp` and `validation.h` are the central validation path for accepting blocks, maintaining the active chain, connecting and disconnecting blocks, and coordinating chainstate transitions.[^ref-btc-core-validation][^ref-btc-core-validation-h]

### Networking

`net_processing` is the relay and peer-state core. This is where Bitcoin Core handles message-driven synchronization, object announcements, transaction requests, peer-specific logic, and many anti-abuse decisions.[^ref-btc-core-net-processing]

### Mempool and Policy

`CTxMemPool` maintains the node's local pool of candidate transactions and associated metadata used for admission, eviction, package reasoning, and block construction.[^ref-btc-core-txmempool]

### Wallet and RPC

Bitcoin Core also ships wallet and RPC surfaces. These do not define Bitcoin consensus, but they heavily shape how operators, applications, and institutions interact with the system in practice.[^ref-btc-core-files]

---

## 6. Why Bitcoin Core Matters

### Dominant Implementation Evidence

Bitcoin Core matters because a large share of the ecosystem runs it or builds assumptions around it. That makes Core a primary implementation source for:

- current relay behavior,
- validation pathways,
- mempool structure,
- wallet defaults,
- and operational conventions.

### But Not Protocol Sovereignty

The protocol is not valid because Core says so. Rather, Core remains relevant because it must continue producing consensus-compatible results that other nodes accept. A rule change embedded in one implementation only matters if the network economically and operationally converges on it.

### Practical Consequence

When analysts say "Bitcoin does X," they often actually mean one of three different things:

1. the consensus rules require X,
2. Bitcoin Core currently implements X,
3. many operators conventionally run Bitcoin Core with settings that make X common.

Those statements are not interchangeable.

---

## 7. Consensus vs Policy

### Consensus

Consensus answers whether a block or transaction is valid for Bitcoin.

Examples:

- proof-of-work validity,
- block-weight limits,
- script correctness,
- subsidy and amount-range constraints,
- coinbase maturity,
- witness commitment rules.[^ref-btc-core-validation]

### Policy

Policy answers whether a node wants to relay, store, request, or mine an otherwise potentially valid object.

Examples:

- mempool feerate thresholds,
- standardness filters,
- orphan limits,
- request scheduling,
- eviction behavior,
- local block template construction.[^ref-btc-core-net-processing][^ref-btc-core-txmempool][^ref-btc-core-31-release]

### Why the Distinction Matters

A transaction can be consensus-valid yet missing from many mempools. A transaction can also be present in one mempool and never become mined. Policy is local and operational; consensus is global and final.

---

## 8. Validation Path and Chainstate

### Active-Chain Maintenance

Bitcoin Core tracks a block index and updates active-chain state as new headers and blocks are validated. `validation.cpp` contains the connection and disconnection paths that move chainstate forward or backward during normal extension and reorganization handling.[^ref-btc-core-validation][^ref-btc-core-validation-h]

### UTXO-Set Transition

Each accepted block applies state transitions to the coins view. Inputs consume existing UTXOs, outputs create new UTXOs, and the resulting state becomes the basis for validating future transactions and blocks.[^ref-btc-core-validation]

### Multiple Chainstates

Bitcoin Core's AssumeUTXO design documentation explains that modern synchronization can involve multiple chainstates, including a fully validated chainstate and a background-validation path for a snapshot-based fast start.[^ref-btc-core-assumeutxo]

This is an implementation architecture point, not a consensus split. The node still aims to converge on the same valid chainstate; it is changing synchronization workflow, not the underlying Bitcoin rules.

---

## 9. Networking and Relay

### Peer Message Processing

`net_processing` coordinates how a node reacts to incoming and outgoing peer events, including synchronization progress, inventory announcements, transaction and block requests, and peer-specific state tracking.[^ref-btc-core-net-processing]

### Object Requests and Anti-Abuse Logic

Bitcoin Core does not naively request every announced object from every peer. Request scheduling, announcement tracking, and anti-stall logic exist because bandwidth, latency, and adversarial behavior matter operationally.[^ref-btc-core-net-processing]

### Local Network View

A node's peer set determines what it sees and when it sees it. That affects:

- first-seen transaction timing,
- mempool composition,
- stale or delayed block receipt,
- topology bias,
- eclipse resilience.

Networking is therefore part of data quality, not just transport plumbing.

---

## 10. Mempool and Mining-Adjacent Policy

### What the Mempool Is

Bitcoin Core's mempool is a local set of unconfirmed transactions the node is willing to keep in memory and consider for relay or block construction. It is not a canonical network-wide pool.[^ref-btc-core-txmempool]

### Cluster-Aware Modern Behavior

Bitcoin Core 31.0 release notes describe a cluster mempool redesign in which transaction ordering, eviction, and replacement reasoning became more package-aware through chunk and feerate-diagram concepts.[^ref-btc-core-31-release]

This matters analytically because mempool observations are partly shaped by implementation version. Two otherwise similar nodes can diverge because one runs a newer policy engine.

### Mining Boundary

Bitcoin Core can construct block templates and apply mining-relevant policy, but template construction remains distinct from consensus. A miner can choose to include a valid transaction that another node would not have kept in its mempool.

---

## 11. Wallet, RPC, and Data Interfaces

### Wallet as an Optional Layer

Bitcoin Core's wallet is part of the distribution, but wallet usage is not required to run a validating node. This matters for institutions that separate validation infrastructure from custody or signing environments.[^ref-btc-core-files]

### RPC as the Operational Contract

For many production systems, the practical interface to Bitcoin Core is RPC. Applications often do not link against internal validation code directly; they interrogate Core through RPC results, notifications, and logs.

### Node-Local Semantics

RPC output is shaped by local node state:

- pruning can remove historical block data availability,
- wallet enablement changes balance and signing functions,
- indexes change queryability,
- peer state changes what the node currently knows,
- mempool policy changes pending-transaction visibility.

An RPC answer is therefore a local answer from that node at that moment.

---

## 12. Pruning, Indexes, and Storage Modes

### Pruning

A pruned node can fully validate the chain while discarding old block files after they are no longer needed for normal operation. This changes archival capability, not consensus validity.

### Indexes

Indexes provide additional derived lookup surfaces. They improve queryability and analysis ergonomics, but they are not required for a node to participate in Bitcoin consensus.

### Data Availability vs Validation Ability

This distinction matters:

```text
can validate != can serve every historical object quickly
can answer an RPC query != can prove a network-wide fact
```

Operational storage choices shape what data a Core node can return to analysts and downstream systems.

---

## 13. AssumeUTXO and Modern Synchronization

### Goal

AssumeUTXO is a synchronization architecture designed to let a node begin from a UTXO snapshot while validating the historical chain in the background.[^ref-btc-core-assumeutxo]

### Why It Exists

The design addresses long initial synchronization times and the practical need to make new nodes usable sooner without discarding eventual full validation goals.

### Analytical Importance

This feature reinforces a broader point: operational state and consensus state are related but not identical. A node may be operationally useful before every historical validation step has completed, depending on its synchronization phase and configuration.[^ref-btc-core-assumeutxo]

Institutions reading Core behavior need to track which node mode they are observing.

---

## 14. Technical Mechanics

### Simplified Subsystem Flow

```text
peer announces tx or block
-> net_processing evaluates request/relay path
-> object reaches validation path
-> consensus and contextual checks run
-> chainstate or mempool updates
-> validationinterface subscribers notified
-> RPC / wallet / indexes observe resulting state
```

### Validation Notifications

`CValidationInterface` marks the boundary where validation outcomes are exposed to subscribers. This is important for wallets, indexes, and other components that need to react to chain or mempool changes without redefining validation themselves.[^ref-btc-core-validationinterface]

### Implementation Stability vs Evolution

The exact internal decomposition of Core can evolve across releases, but the key architectural boundaries remain stable enough for research:

- networking receives and coordinates,
- validation decides,
- chainstate persists consensus results,
- mempool stores local pending state,
- wallet and RPC present node-local interfaces.

---

## 15. Security Assumptions and Failure Modes

### Software Correctness Matters

Bitcoin Core is security-critical software. Validation bugs, wallet bugs, persistence bugs, or relay bugs can have severe network or operator consequences.

### Configuration Risk

Operational misconfiguration can create misleading local views or degraded security:

- inadequate peer diversity,
- unexpected pruning,
- wallet exposure on validation hosts,
- permissive RPC access,
- inaccurate assumptions about index coverage.

### Version and Deployment Risk

Behavior can change across versions, especially in policy-heavy areas such as mempool admission, replacement, and relay. Institutions should pin software versions and document upgrade assumptions.[^ref-btc-core-31-release]

### Local View Risk

A single Core node can be:

- partially synchronized,
- eclipsed or topology-biased,
- policy-divergent,
- missing indexes,
- or running with wallet disabled.

None of those states rewrite Bitcoin consensus, but all of them change what the operator sees.

---

## 16. Mathematical or Economic Model

### Validation Cost Intuition

A simple way to reason about node work is:

```text
total operational load
= block validation load
+ transaction relay load
+ mempool maintenance load
+ disk and index maintenance load
+ wallet / RPC query load
```

This is not a protocol formula. It is an engineering decomposition of node resource demand.

### Local Mempool Non-Identity

If `M_i` denotes the mempool visible to node `i`, then in practice:

```text
M_a != M_b
```

is common because peer topology, fee filters, orphan state, software version, and policy settings differ across nodes.

### Operational Economics

Institutions selecting Core deployment profiles implicitly trade off:

- storage vs queryability,
- sync speed vs historical completeness,
- wallet convenience vs attack surface,
- one-node simplicity vs multi-node observability.

These are operator economics, not consensus economics.

---

## 17. Bitcoin Core Implementation

### Source-Tree Orientation

Bitcoin Core's `doc/files.md` describes the source-tree organization and main user-facing binaries. It is the correct starting point for implementation orientation because it tells readers where major subsystems live without pretending the repository is a flat codebase.[^ref-btc-core-files]

### `validation.cpp` and `validation.h`

These files are the main chain-validation and active-state coordination surface. They include block acceptance, reorg-sensitive state changes, chainstate handling, and many contextual validation pathways.[^ref-btc-core-validation][^ref-btc-core-validation-h]

### `net_processing`

This module coordinates peer-driven object handling, announcement logic, synchronization behavior, and anti-abuse responses.[^ref-btc-core-net-processing]

### `txmempool.h`

`CTxMemPool` and related structures represent the node-local pending-transaction set and metadata used for relay and mining-adjacent policy behavior.[^ref-btc-core-txmempool]

### `validationinterface.h`

`CValidationInterface` exposes notifications to subscribers when validation-relevant events occur. This preserves separation between the validation engine and downstream consumers.[^ref-btc-core-validationinterface]

### `doc/design/assumeutxo.md`

This design note is important because it documents a current Core architecture feature rather than an abstract future concept. It shows how Core separates operational sync convenience from eventual validation completeness.[^ref-btc-core-assumeutxo]

---

## 18. On-Chain Implications

### What On-Chain Data Reflects

The blockchain records accepted consensus outcomes:

- confirmed transactions,
- confirmed blocks,
- reorg history as eventually observed,
- spent and unspent outputs implied by accepted chain history.

### What It Does Not Reflect

The blockchain does not record most local Bitcoin Core state:

- mempool contents,
- orphan pool contents,
- peer request choices,
- wallet labels,
- RPC clients,
- pruning configuration,
- background sync phase.

### Consequence for Analysts

Chain analysis alone cannot reconstruct the full behavior of a Core node. A large fraction of node behavior is off-chain implementation state.

---

## 19. Institutional Thinking

Institutions should treat Bitcoin Core as both infrastructure software and a measurement instrument.

### Practical Implications

- A validation node and an analytics node may need different configurations.
- One Core node is not enough for strong mempool or propagation claims.
- Wallet-enabled Core instances should usually be separated from broader exposure surfaces unless there is a specific reason not to.
- Version upgrades should be analyzed as data-model changes in addition to software maintenance events.
- RPC consumers should log node version, pruning mode, wallet mode, and index coverage alongside analytical outputs.

### Better Research Posture

Serious Bitcoin research built on Core should:

- identify which claims are consensus claims,
- identify which claims are Core-implementation claims,
- identify which claims are local-configuration claims,
- and identify which claims are merely inferences from one node's observation window.

---

## 20. Common Misinterpretations

### "Bitcoin Core is Bitcoin"

Too strong. Bitcoin Core is the dominant implementation, not the protocol by metaphysical identity.

### "If Core relays it, Bitcoin accepts it"

False. Relay policy and consensus validity are different.

### "If my Core node shows it, the network shows it"

False. The node exposes a local, topology-dependent, policy-dependent view.

### "Pruned nodes are not real full nodes"

False in the consensus sense. Pruned nodes can fully validate, even though they do not retain all historical block data for archival serving.

### "Wallet and node are the same thing"

False. Bitcoin Core can run as a validating node without being used as the institution's custody or signing system.

### "RPC output is objective ground truth"

False. It is a serialized view from one configured node at one time.

---

## 21. Research Questions

1. How much do mempool and relay observations diverge across Bitcoin Core versions during fee spikes?
2. How should institutions classify analytical claims derived from pruned versus archival nodes?
3. What operational controls most reduce misinterpretation of node-local RPC outputs in research pipelines?
4. How materially does AssumeUTXO change institutional node-deployment strategy over time?
5. Which Core configuration fields should always be captured alongside exported analytics?

---

## 22. Practical Exercises

### Exercise 1

Map the path from an announced transaction to either mempool admission or rejection, distinguishing network handling, policy checks, and consensus-relevant validation.

### Exercise 2

Compare the outputs of two nodes with different pruning and index settings and identify which questions each node can answer reliably.

### Exercise 3

Explain why a consensus-valid transaction can be absent from one node's mempool while being present in another's.

### Exercise 4

Describe how `CValidationInterface` helps downstream components observe validation events without turning those components into validators.

---

## 23. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Main binaries and source-tree organization | Directly specified | `doc/files.md` |
| Validation, chainstate, and notification boundaries | Directly specified | `validation.cpp`, `validation.h`, `validationinterface.h` |
| Peer message handling and relay coordination | Directly specified | `net_processing.cpp` |
| Mempool local-state behavior | Directly specified | `txmempool.h` and release notes |
| AssumeUTXO architecture | Directly specified | Core design documentation |
| Measurement-bias and local-view implications | Inference from sources | Derived from node-local policy, peer topology, and storage/config modes |

---

## 24. Knowledge Graph

```text
Bitcoin Core
├─ Operator Surface
│  ├─ bitcoind
│  ├─ bitcoin-cli
│  ├─ bitcoin-qt
│  └─ bitcoin-wallet
├─ Consensus Engine
│  ├─ validation.cpp
│  ├─ validation.h
│  ├─ chainstate
│  └─ UTXO updates
├─ Network Layer
│  ├─ net_processing
│  ├─ peer state
│  ├─ requests
│  └─ relay behavior
├─ Local Policy
│  ├─ mempool
│  ├─ standardness
│  ├─ eviction
│  └─ mining templates
├─ Interfaces
│  ├─ RPC
│  ├─ wallet
│  ├─ indexes
│  └─ validationinterface
└─ Operational Modes
   ├─ pruning
   ├─ archival
   ├─ AssumeUTXO
   └─ version-specific behavior
```

---

## 25. References

### Primary Sources

[^ref-btc-core-files]: Bitcoin Core Contributors, `doc/files.md`, source-tree and binary overview, Bitcoin Core master branch, https://github.com/bitcoin/bitcoin/blob/master/doc/files.md, accessed 2026-08-04.

[^ref-btc-core-validation]: Bitcoin Core Contributors, `src/validation.cpp`, block validation, chainstate coordination, and active-chain processing, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/validation_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-validation-h]: Bitcoin Core Contributors, `src/validation.h`, validation interfaces, chainstate-related declarations, and block connection/disconnection surfaces, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/validation_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-validationinterface]: Bitcoin Core Contributors, `src/validationinterface.h`, `CValidationInterface` notification interface, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/validationinterface_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-net-processing]: Bitcoin Core Contributors, `src/net_processing.cpp`, peer message processing and synchronization logic, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/net__processing_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-txmempool]: Bitcoin Core Contributors, `src/txmempool.h`, `CTxMemPool` structures and node-local transaction pool metadata, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/txmempool_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-assumeutxo]: Bitcoin Core Contributors, `doc/design/assumeutxo.md`, snapshot-based chainstate synchronization design, Bitcoin Core master branch, https://github.com/bitcoin/bitcoin/blob/master/doc/design/assumeutxo.md, accessed 2026-08-04.

[^ref-btc-core-31-release]: Bitcoin Core Contributors, Bitcoin Core 31.0 release notes, mempool and policy-related implementation changes, https://bitcoincore.org/en/releases/31.0/, accessed 2026-08-04.

### Supporting Interpretation Notes

- Where this document discusses measurement bias, node-local truth, or institutional observability limits, those claims are analytical inferences from Bitcoin Core's documented architecture and configuration-dependent behavior rather than explicit consensus claims.

---

## 26. Cross References

### Previous

- BITCOIN-032 — Lightning Network

### Next

- BITCOIN-034

### Related

- BITCOIN-014 — UTXO Model
- BITCOIN-017 — Mempool
- BITCOIN-021 — Blocks and Block Headers
- BITCOIN-022 — Nodes and Network Propagation
- POW-013 — Bitcoin Core Proof-of-Work Validation

---

## Review Status

### Technical Review

Passed.

- Bitcoin Core's roles were separated into validation, relay, mempool policy, wallet, RPC, and storage/indexing.
- Consensus rules were explicitly distinguished from local policy and operational configuration.
- AssumeUTXO was described as a synchronization architecture feature, not a consensus change.
- Mempool and RPC outputs were treated as node-local state rather than protocol truth.

### Evidence Review

Passed.

- Source-tree, binary, and subsystem orientation is grounded in `doc/files.md`.
- Validation, networking, mempool, and notification claims are grounded in Bitcoin Core source documentation.
- Version-sensitive mempool claims are tied to the 31.0 release notes.
- Analytical claims about local-view bias are labeled as inference.

### Editorial Review

Passed.

- Structure follows the project deep-dive format.
- Metadata is complete.
- Terminology is consistent: consensus, policy, chainstate, mempool, pruning, RPC, wallet, AssumeUTXO.
- Tables and code fences are closed.

### Adversarial Review

Passed.

- The document does not equate Bitcoin Core with Bitcoin itself.
- It does not imply local relay behavior defines consensus.
- It does not treat one node's mempool or RPC output as global truth.
- It does not describe pruning as a loss of validation capability.
- It does not describe AssumeUTXO as bypassing eventual validation.

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
