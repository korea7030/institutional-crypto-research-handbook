---
knowledge_id: BITCOIN-006
title: Whitepaper Section 5 — Network
subtitle: Transaction Relay, Block Propagation, Validation, Fork Resolution, and Network Observability
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 80 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - P2P Network
  - Consensus
  - Block Propagation
parent:
  - Bitcoin Whitepaper
prerequisites:
  - BITCOIN-004
  - BITCOIN-005
related_topics:
  - Transaction Relay
  - Mempool
  - Block Broadcasting
  - Headers-First Sync
  - Compact Blocks
  - Fork Resolution
  - Orphan Blocks
  - Stale Blocks
  - Node Validation
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-P2P-GUIDE-001
  - REF-BTC-DEV-P2P-REF-001
  - REF-BTC-CORE-NETPROCESSING-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BIP-0130
  - REF-BIP-0152
tags:
  - bitcoin
  - whitepaper
  - network
  - p2p
  - block-relay
  - validation
---

# Whitepaper Section 5 — Network
> Deep Dive Series  
> Research Unit: BITCOIN-006

---

## Research Brief

```yaml
knowledge_id: BITCOIN-006
title: Whitepaper Section 5 — Network
research_question: >
  How does Bitcoin's peer-to-peer network relay transactions and blocks,
  let independently validating nodes converge on a greatest-work history,
  and remain tolerant of message loss, latency, and temporary competing blocks?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-004
  - BITCOIN-005
parent: Bitcoin Whitepaper
previous: BITCOIN-005
next: BITCOIN-007
related_topics:
  - Transaction Relay
  - Block Propagation
  - Headers-First Sync
  - Compact Blocks
  - Fork Resolution
  - Mempool
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Original Design
  - Protocol Structure
  - Technical Mechanics
  - Security Assumptions
  - Bitcoin Core Implementation
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Full address-manager design
  - Tor and transport-layer privacy engineering
  - Erlay and package-relay design
  - Mining pool protocols
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Reconstruct the six-step network process described in the Bitcoin Whitepaper.
- Separate transaction relay from block confirmation.
- Explain why the mempool is not a consensus object.
- Explain how nodes validate received blocks before building on them.
- Distinguish temporary competing blocks from consensus failure.
- Explain why missing messages do not necessarily break eventual synchronization.
- Describe the roles of `inv`, `getdata`, `getheaders`, `headers`, `block`, and compact block messages.
- Distinguish legacy block relay, direct headers announcements, and compact block relay.
- Identify which network events are visible on-chain and which require off-chain measurement.

## 2. Key Questions

1. What does the whitepaper mean by "new transactions are broadcast to all nodes"?
2. Does every node receive every transaction before it is mined?
3. What does a node do when it receives a new block?
4. Why do nodes build on a block only after accepting it?
5. How does the network handle two valid blocks found close together?
6. What is the difference between a stale block and an orphan block?
7. How do modern nodes request data from peers?
8. Why did Bitcoin move from simple block relay toward headers-first and compact block relay?
9. Which network-layer observations can an on-chain analyst verify from the blockchain alone?
10. Why is propagation latency economically relevant to miners?

## 3. Executive Summary

Section 5 of the Bitcoin Whitepaper describes the operational network loop that connects transactions, mining, block validation, and convergence. Its six steps are: broadcast new transactions, collect transactions into a block, search for Proof of Work, broadcast the completed block, accept the block only if its transactions are valid and unspent, and express acceptance by mining on top of that block.[^ref-btc-wp]

The network section is often read as if Bitcoin requires every message to reach every node. It does not. The whitepaper explicitly states that new transaction broadcasts do not need to reach all nodes, as long as they reach many nodes and eventually enter a block. It also states that block broadcasts tolerate dropped messages because nodes can request missing blocks after learning that they missed one.[^ref-btc-wp]

Modern Bitcoin implements this broad design through a peer-to-peer protocol over TCP, using structured messages such as `inv`, `getdata`, `getheaders`, `headers`, and `block`.[^ref-btc-dev-p2p-ref] Since the whitepaper, block relay has evolved. Bitcoin Core has used headers-first synchronization, BIP130 `sendheaders`, and BIP152 compact block relay to reduce bandwidth, avoid some orphan-block problems, and improve block relay efficiency.[^ref-btc-dev-p2p-guide][^ref-bip-0130][^ref-bip-0152]

The key institutional distinction is:

```text
Network relay spreads candidate data.
Consensus validation decides whether local nodes accept it.
Proof of Work orders accepted blocks by cumulative work.
```

## 4. Definition and Scope

The Bitcoin network is a peer-to-peer communication layer through which nodes exchange transactions, block headers, blocks, peer addresses, and synchronization requests. It is not a central server, a global mempool, a guaranteed broadcast bus, or a formal consensus vote by IP address.

This document focuses on the network process described in Whitepaper Section 5 and the modern protocol mechanisms that implement the same broad goals. It uses Bitcoin Core and Bitcoin Developer documentation as implementation evidence, but it does not attempt to document every message type or every peer-management heuristic.

## 5. Historical Background

The whitepaper's network section follows directly from Proof of Work. Section 4 explains how work makes history expensive to rewrite; Section 5 explains how peers exchange transactions and blocks so that independent nodes can converge on the same worked history.[^ref-btc-wp]

The original model is deliberately simple. It describes broadcasting transactions and blocks, accepting blocks that pass validation, and resolving temporary disagreement by extending one branch with more Proof of Work.[^ref-btc-wp]

Modern Bitcoin preserves that logic but uses more detailed relay mechanisms. Bitcoin Developer documentation states that peer-to-peer communication occurs over TCP and describes a message protocol with headers, command names, payload sizes, and checksums.[^ref-btc-dev-p2p-ref]

## 6. Original Design

The whitepaper gives the network process as six steps:

```text
1. New transactions are broadcast to all nodes.
2. Each node collects new transactions into a block.
3. Each node works on finding Proof of Work for its block.
4. When a node finds Proof of Work, it broadcasts the block to all nodes.
5. Nodes accept the block only if all transactions are valid and not already spent.
6. Nodes express acceptance by working on the next block using the accepted block hash as the previous hash.
```

This sequence is not a guarantee that every node sees identical data at the same time. It is a convergence process under latency, message loss, and temporary disagreement.

The whitepaper also describes how simultaneous block discovery is handled. If two valid blocks are broadcast close together, different nodes may receive different blocks first. Nodes may work on the block they received first while keeping the competing branch. The tie is resolved when another Proof-of-Work block extends one branch, after which nodes on the other branch switch to the longer worked branch.[^ref-btc-wp]

## 7. Protocol Structure

Modern Bitcoin P2P messages have a common envelope containing:

| Field | Purpose |
|---|---|
| Start string | Identifies the Bitcoin network, such as mainnet or testnet. |
| Command name | Identifies the message type, such as `inv`, `getdata`, or `headers`. |
| Payload size | States the byte length of the payload. |
| Checksum | Detects payload corruption. |

Bitcoin Developer documentation describes this message-header format and notes that Bitcoin P2P communication occurs over TCP.[^ref-btc-dev-p2p-ref]

Important data-relay messages include:

| Message | Role |
|---|---|
| `inv` | Advertises known transactions or blocks by inventory hash. |
| `getdata` | Requests advertised objects. |
| `getheaders` | Requests block headers after a locator point. |
| `headers` | Sends up to 2,000 block headers in response to header requests. |
| `block` | Sends a serialized block. |
| `sendheaders` | Signals preference for header-based block announcements. |
| `cmpctblock` | Sends a compact representation of a block under BIP152. |

An inventory entry identifies an object type and object hash. For blocks, the inventory hash is the block-header hash.[^ref-btc-dev-p2p-ref]

## 8. Technical Mechanics

### Transaction Relay

Transaction relay is a network and policy process. Nodes may receive transactions, check them against local policy and consensus constraints, store acceptable unconfirmed transactions in a mempool, and advertise them to peers. However, the mempool is local node state. It is not globally synchronized and is not part of consensus.

This distinction matters because Whitepaper Step 1 is a simplified broadcast model. In practice, transactions may fail to propagate uniformly because of policy differences, fee filters, topology, timing, bandwidth, replacement behavior, or local configuration.

### Block Relay

When a miner finds a block, it can send the block directly or announce it so peers request the data. Bitcoin Developer documentation describes standard block relay using `inv`, peer requests using `getdata` or `getheaders`, and full block transfer using `block` messages.[^ref-btc-dev-p2p-guide][^ref-btc-dev-p2p-ref]

Modern block relay often uses headers-first logic. A node can learn and validate headers before requesting full block bodies. This reduces several blocks-first synchronization problems, including some orphan-block handling and unnecessary downloads.[^ref-btc-dev-p2p-guide]

### Compact Block Relay

BIP152 specifies compact block relay. Instead of sending every full transaction immediately, a peer can send a block header, short transaction identifiers, and selected prefilled transactions. The receiver reconstructs the block using transactions it already has, requesting missing transactions when necessary.[^ref-bip-0152]

Compact blocks rely on the empirical expectation that many peers already share much of the same mempool content. This is a network-efficiency assumption, not a consensus rule.

### Validation Before Acceptance

A node accepting a block is not merely receiving a message. Bitcoin Core's incoming-block path calls `CheckBlock`, then `AcceptBlock`, and then attempts active-chain activation through `ActivateBestChain` in `ProcessNewBlock`.[^ref-btc-core-validation]

The whitepaper's Step 5 is therefore still the key rule: nodes accept a block only if it satisfies validity requirements. A received block that fails validation is not made valid by being widely relayed.

## 9. Mathematical or Economic Model

Network propagation affects miner economics because stale-block risk depends partly on latency.

Let:

- `R` be expected block reward plus fees for a found block.
- `s` be the probability that a miner's valid block becomes stale because another competing branch wins.
- `E` be expected realized revenue for that found block.

A simplified expected-value model is:

```text
E = R * (1 - s)
```

This is not a consensus equation. It is an economic approximation.

The model assumes that the block is otherwise valid, that stale probability can be estimated, and that payout is zero if the block is not included in the active chain. Real mining economics also depend on pool rules, transaction fees, propagation topology, block size, orphan/stale definitions, and variance.

## 10. Security Assumptions

Bitcoin's network design assumes that honest nodes can learn about valid blocks quickly enough to converge on a greatest-work chain.

It does not assume:

- every node receives every transaction,
- every peer is honest,
- every node has the same mempool,
- message delivery is instant,
- IP identity is scarce,
- transaction relay policy is identical across nodes.

It does assume that nodes independently validate blocks instead of accepting a block merely because a peer sent it. This is why the network layer and validation layer must be analyzed separately.

## 11. Failure Modes and Attack Surface

### Eclipse and Isolation Risk

A node that is isolated from honest peers can receive a distorted view of transactions and blocks. The blockchain cannot reveal a node's private network view after the fact.

### Stale Blocks

A stale block is a valid block with a known parent that is not part of the active best chain. Stale blocks can arise naturally when miners find competing blocks at similar times.

### Orphan Blocks

Bitcoin Developer documentation describes orphan blocks as blocks whose previous block header hash refers to a parent the node has not seen. This is different from a stale block, which has a known parent but is not in the best chain.[^ref-btc-dev-p2p-guide]

### Invalid Relay

Peers can relay invalid blocks or transactions. Relay does not imply acceptance. Nodes must validate.

### Latency Advantage

Miners with faster propagation may reduce stale risk. This can create economic pressure toward optimized relay networks, geographic concentration, or private connectivity. The magnitude requires measurement beyond the blockchain alone.

## 12. Bitcoin Core or Protocol Implementation

Bitcoin Core's P2P processing is implemented primarily in `src/net_processing.cpp`. The `PeerManagerImpl::ProcessMessage` path handles incoming peer messages and dispatches behavior based on message type.[^ref-btc-core-netprocessing]

For block-related announcements, Bitcoin Core processes inventory and can respond by sending `getheaders` when a peer appears to know a block beyond the local known header chain.[^ref-btc-core-netprocessing]

For data requests, Bitcoin Core handles `getdata` messages and enforces inventory-size limits before responding.[^ref-btc-core-netprocessing]

For validated block download management, Bitcoin Core contains stalling and timeout logic. This is not a consensus rule, but it is important implementation behavior for resisting slow or unhelpful peers.[^ref-btc-core-netprocessing]

Bitcoin Core's validation path is separate. `ChainstateManager::ProcessNewBlock` processes an incoming block, runs `CheckBlock`, stores sufficiently validated blocks through `AcceptBlock`, and then attempts to activate the best chain.[^ref-btc-core-validation]

## 13. Related Standards and BIPs

### BIP130 — sendheaders

BIP130 defines the `sendheaders` message. It lets a node indicate that it prefers new block announcements by `headers` message rather than only by `inv`. The BIP motivates this as a way to reduce extra round trips after headers-first synchronization.[^ref-bip-0130]

### BIP152 — Compact Block Relay

BIP152 defines compact block relay. Its motivation is bandwidth reduction during block relay, because many transactions in a new block may already be available to peers before the block is relayed.[^ref-bip-0152]

These BIPs modify peer-service behavior. They do not change the fundamental consensus requirement that blocks must validate.

## 14. Modern Context

The whitepaper's phrase "broadcast to all nodes" should be interpreted as a design-level propagation goal, not a literal guarantee. Modern Bitcoin nodes relay through selected peers, request missing data, maintain local mempools, and apply anti-DoS and bandwidth controls.

Headers-first synchronization means a node can verify a chain of block headers before downloading every full block. This improves initial block download behavior compared with older blocks-first methods.[^ref-btc-dev-p2p-guide]

Compact block relay reduces redundant transaction transfer when peers already share mempool content. It is most useful when mempools overlap. If a receiver lacks many transactions, it must request missing transactions and reconstruction may require extra round trips.[^ref-bip-0152]

The high-level whitepaper loop remains recognizable, but the operational details are now more layered:

```text
Peer discovery and connection
↓
Transaction relay and local mempool policy
↓
Block-header announcement
↓
Block or compact-block reconstruction
↓
Independent validation
↓
Best-chain activation
↓
Further relay
```

## 15. On-Chain Implications

### Directly Observable

An on-chain analyst can directly observe:

- which block became part of the active chain,
- block height and parent linkage,
- block timestamp,
- transaction inclusion,
- transaction ordering inside a block,
- coinbase transaction content,
- chain reorganizations visible in the final node's chain history if archival competing blocks are available from the observer.

### Estimated

An analyst may estimate:

- propagation delay using independently collected network observations,
- stale-block frequency using relay-network or archival stale-block datasets,
- miner latency advantage from stale outcomes and timing data,
- mempool-to-confirmation behavior from a specific observer's mempool logs.

### Unobservable From Chain Alone

An analyst cannot infer from the final blockchain alone:

- which nodes saw a transaction before confirmation,
- exact propagation path,
- exact first-seen time across the network,
- whether a transaction was censored or merely failed local policy,
- every stale block that existed but was not observed by the analyst,
- a miner's private template-selection process.

The final chain records accepted history. It does not record the full network message graph that produced that history.

## 16. Institutional Thinking

### Observation

A transaction appears in a block at height `H`.

### Weak Interpretation

"The transaction was broadcast to the whole network and accepted by all nodes before block `H`."

### Institutional Decomposition

An institutional researcher should ask:

- Which observer first saw the transaction?
- Was it present in multiple independent mempools?
- Did relay policy affect propagation?
- Did the miner include it from public relay, private submission, or direct payment channel?
- Were competing blocks or reorgs observed around height `H`?
- Is the claim based on final chain data or network telemetry?

### Current Conclusion

The blockchain proves that the transaction was included in the accepted block history observed by the node. It does not prove universal pre-confirmation relay or intent by all nodes.

### Confidence

FACT — ECL-A for block inclusion.  
INTERPRETATION — ECL-B or ECL-C for propagation and intent claims, depending on independent network telemetry.

## 17. Analyst Notes

Treat mempool data as observer-specific. A mempool snapshot from one node is evidence about that node's local view, not the entire Bitcoin network.

Separate "not confirmed" from "not propagated." A transaction may be widely propagated but not mined because of fee rate, policy, miner selection, conflicts, or timing.

Separate "stale block" from "invalid block." A stale block can be valid but not part of the active chain. An invalid block violates consensus rules.

Use exact terms when describing relay:

- `inv` advertises.
- `getdata` requests.
- `headers` sends headers.
- `block` sends full block data.
- `cmpctblock` sends compact block data.

## 18. Common Misinterpretations

### Misinterpretation

> Bitcoin requires every transaction to reach every node before it can be mined.

### Assessment

Incorrect.

### Correction

The whitepaper states that transaction broadcasts do not need to reach all nodes. They need to reach enough nodes to be included in a block.[^ref-btc-wp]

### Misinterpretation

> The mempool is a single global queue.

### Assessment

Incorrect.

### Correction

Each node maintains local mempool state according to timing, policy, fee filters, and configuration.

### Misinterpretation

> A block is accepted because most peers relayed it.

### Assessment

Incorrect.

### Correction

Relay spreads data. Nodes accept blocks only after validation.

### Misinterpretation

> A temporary fork means Bitcoin consensus failed.

### Assessment

Incomplete.

### Correction

Temporary competing valid blocks are expected in a distributed network. Convergence occurs when one branch gains more cumulative work.

### Misinterpretation

> Compact blocks changed Bitcoin consensus.

### Assessment

Incorrect.

### Correction

BIP152 changes block relay efficiency. It does not change block validity rules.

## 19. Counter Evidence and Limitations

The whitepaper is intentionally abstract and does not specify the modern P2P message protocol. It should not be used as the only source for current relay behavior.

Bitcoin Developer documentation says its P2P section describes the protocol but is not itself a specification.[^ref-btc-dev-p2p-ref] For current behavior, implementation references should be checked.

Bitcoin Core behavior is not identical to all possible Bitcoin implementations. Consensus rules constrain valid chains, but relay policy and peer-management choices may differ across software.

Network observations are measurement-dependent. A researcher who observes no transaction in one mempool cannot conclude that no node saw it.

Compact block relay depends on mempool overlap. When peers do not share the needed transactions, reconstruction requires requesting missing data, reducing the latency benefit.[^ref-bip-0152]

## 20. Research Questions

1. How much mempool divergence exists across geographically distributed observers during fee spikes?
2. How do compact block reconstruction failures vary with mempool policy differences?
3. Which measurements best distinguish miner censorship from fee-market non-inclusion?
4. How often do stale blocks occur under different block-size and fee-pressure regimes?
5. What network telemetry is required to support claims about first-seen transaction timing?

## 21. Challenge

1. Reconstruct the whitepaper's six network steps without looking.
2. Explain why transaction relay is not consensus.
3. Distinguish `inv`, `getdata`, `headers`, and `block`.
4. Explain why a node may know a block header before downloading the full block.
5. Give one on-chain fact and one off-chain inference related to transaction propagation.

## 22. Evidence Classification

| Claim ID | Claim | Classification | ECL | Primary Sources |
|---|---|---|---|---|
| C001 | Whitepaper Section 5 defines a six-step network process for transaction broadcast, block construction, Proof of Work, block broadcast, validation, and extension. | FACT | A | REF-BTC-WP-001 |
| C002 | The whitepaper states that transaction broadcasts do not need to reach all nodes. | FACT | A | REF-BTC-WP-001 |
| C003 | Bitcoin P2P communication occurs over TCP and uses structured protocol messages. | FACT | A | REF-BTC-DEV-P2P-REF-001 |
| C004 | `inv` advertises objects and `getdata` requests advertised objects. | FACT | A | REF-BTC-DEV-P2P-REF-001 |
| C005 | Headers-first and direct header announcements reduce some round trips and orphan-block handling problems. | FACT | A | REF-BTC-DEV-P2P-GUIDE-001, REF-BIP-0130 |
| C006 | Compact block relay reduces bandwidth by sending compact block information when peers likely already have transactions. | FACT | A | REF-BIP-0152 |
| C007 | Bitcoin Core separates peer-message processing from block validation and best-chain activation. | FACT | A | REF-BTC-CORE-NETPROCESSING-001, REF-BTC-CORE-VALIDATION-001 |
| C008 | Final-chain data alone cannot prove transaction propagation path or universal pre-confirmation visibility. | INTERPRETATION | B | REF-BTC-WP-001, REF-BTC-DEV-P2P-REF-001 |

## 23. Knowledge Graph

```text
BITCOIN-005 Proof of Work
│
├── provides: costly block proposal
├── constrains: valid block headers
└── requires: network relay for propagation
        │
        ▼
BITCOIN-006 Network
│
├── relays: transactions
├── relays: block headers
├── relays: full or compact blocks
├── depends_on: independent node validation
├── produces: temporary competing branches
├── resolves_by: cumulative work extension
└── exposes: observer-specific mempool evidence
        │
        ▼
BITCOIN-007 Incentive
│
├── rewards: accepted block production
├── penalizes: stale block risk
└── motivates: relay efficiency
```

## 24. References

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Section 5, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-p2p-ref]: Bitcoin Developer Documentation, "P2P Network," message headers and data messages, https://developer.bitcoin.org/reference/p2p_networking.html, accessed 2026-08-04.

[^ref-btc-dev-p2p-guide]: Bitcoin Developer Documentation, "P2P Network," block broadcasting, headers-first synchronization, and orphan blocks, https://developer.bitcoin.org/devguide/p2p_network.html, accessed 2026-08-04.

[^ref-btc-core-netprocessing]: Bitcoin Core Contributors, `src/net_processing.cpp`, functions `PeerManagerImpl::ProcessMessage`, `ProcessHeadersMessage`, inventory handling, `getdata` handling, and block download timeout logic, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/net__processing_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-validation]: Bitcoin Core Contributors, `src/validation.cpp`, functions `ChainstateManager::ProcessNewBlock`, `CheckBlock`, `AcceptBlock`, and `ActivateBestChain`, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/validation_8cpp_source.html, accessed 2026-08-04.

[^ref-bip-0130]: Suhas Daftuar, "BIP130: sendheaders message," Bitcoin Improvement Proposals, status Deployed, https://bips.dev/130/, accessed 2026-08-04.

[^ref-bip-0152]: Matt Corallo, "BIP152: Compact Block Relay," Bitcoin Improvement Proposals, status Deployed, https://bips.dev/152/, accessed 2026-08-04.

## 25. Cross References

### Parent

- Bitcoin Whitepaper

### Previous

- BITCOIN-005 — Whitepaper Section 4 — Proof of Work

### Next

- BITCOIN-007 — Whitepaper Section 6 — Incentive

### Related

- BITCOIN-017 — Mempool
- BITCOIN-018 — Blocks & Block Headers
- BITCOIN-021 — Nodes & Network Propagation
- BITCOIN-022 — Forks
- BITCOIN-023 — Chain Reorganization
- POW-008 — Bitcoin Mining

## Review Status

### Technical Review

Passed.

- Whitepaper Section 5 sequence was preserved.
- Transaction relay, block relay, validation, and chain selection were separated.
- Stale and orphan terminology was checked against Bitcoin Developer documentation.
- BIP130 and BIP152 were scoped as peer-service changes rather than consensus changes.

### Evidence Review

Passed.

- Material claims cite primary or protocol-maintainer sources.
- Current implementation claims are mapped to Bitcoin Core `net_processing.cpp` and `validation.cpp`.
- The document does not infer universal propagation from block inclusion.
- Source limitations are explicitly stated.

### Editorial Review

Passed.

- Markdown headings follow the required deep-dive structure.
- Metadata is complete under the authoring guide.
- Tables and code fences are closed.
- Terminology is consistent: relay, validation, mempool, stale block, orphan block, active chain.

### Adversarial Review

Passed.

- Overstatement risks were reduced around "broadcast to all nodes," global mempool assumptions, compact-block benefits, and propagation inference.
- Current Bitcoin Core behavior is not generalized to all implementations where relay policy may differ.
- Network telemetry limits are stated clearly.

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
