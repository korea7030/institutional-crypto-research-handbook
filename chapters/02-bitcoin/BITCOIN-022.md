---
knowledge_id: BITCOIN-022
title: Nodes and Network Propagation
subtitle: Peer Discovery, Handshake, Inventory Relay, Headers-First Synchronization, Transaction Relay, and Propagation Security
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 110 min
estimated_study: 320 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Networking
  - Nodes
  - Propagation
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-017
  - BITCOIN-021
  - POW-011
  - POW-013
related_topics:
  - P2P Network
  - Headers-First Sync
  - Transaction Relay
  - Compact Blocks
  - SPV
  - AddrMan
  - Eclipse Attacks
  - Mempool Policy
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-P2P-001
  - REF-BIP-0037
  - REF-BIP-0130
  - REF-BIP-0133
  - REF-BIP-0152
  - REF-BIP-0155
  - REF-BIP-0159
  - REF-BIP-0324
  - REF-BIP-0339
  - REF-BTC-CORE-NET-001
  - REF-BTC-CORE-PROTOCOL-001
  - REF-BTC-CORE-NET-PROCESSING-001
  - REF-BTC-CORE-TXREQUEST-001
  - REF-BTC-CORE-TXORPHANAGE-001
  - REF-BTC-CORE-VALIDATIONINTERFACE-001
tags:
  - bitcoin
  - internals
  - networking
  - nodes
  - p2p
  - propagation
  - relay
  - headers-first-sync
---

# Nodes and Network Propagation
> Bitcoin Internals  
> Research Unit: BITCOIN-022

---

## Research Brief

```yaml
knowledge_id: BITCOIN-022
title: Nodes and Network Propagation
research_question: >
  How do Bitcoin nodes discover peers, establish sessions, exchange addresses
  and inventories, synchronize headers and blocks, relay transactions, and
  defend propagation quality without confusing local relay policy with global
  consensus?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-017
  - BITCOIN-021
  - POW-011
  - POW-013
parent: Bitcoin Internals
previous: BITCOIN-021
next: BITCOIN-023
related_topics:
  - P2P Protocol
  - Transaction Relay
  - Block Relay
  - Headers-First Synchronization
  - Mempool Policy
  - Eclipse Resistance
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
  - Full transport cryptography implementation details
  - Lightning Network routing
  - Compact block reconstruction internals beyond analyst-relevant behavior
  - Tor operational guidance
  - Non-Bitcoin P2P network comparisons
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Distinguish node discovery, connection setup, synchronization, and relay.
- Explain the `version`/`verack` handshake and the role of service bits.
- Explain how `addr` and `addrv2` support peer discovery.
- Explain how `inv`, `getdata`, `headers`, `getheaders`, and `block` interact.
- Distinguish block relay from transaction relay.
- Explain why headers-first synchronization reduces bandwidth and trust assumptions.
- Explain how `sendheaders`, compact blocks, and `wtxid` relay improve propagation.
- Distinguish consensus validity from local mempool relay policy.
- Explain why propagation topology affects latency, stale risk, and privacy.
- Identify Bitcoin Core modules that implement peer state, relay, and orphan handling.

---

## 2. Key Questions

1. What kinds of nodes exist in Bitcoin's network model?
2. How do nodes discover and connect to peers?
3. What information is exchanged during the handshake?
4. How are new blocks announced?
5. How are transactions announced and requested?
6. What is the difference between `inv`-based relay and `headers`-based relay?
7. What problem do compact blocks solve?
8. Why does `wtxid` relay matter after SegWit?
9. What is the difference between relay policy and consensus?
10. How do orphan transactions arise during propagation?
11. What are the main propagation-layer attack surfaces?
12. What can on-chain analysts infer, and what can they not infer, from propagation behavior?

---

## 3. Executive Summary

Bitcoin is a peer-to-peer network of nodes that discover each other, negotiate capabilities, synchronize block headers and blocks, and relay transactions. The whitepaper's network section describes the baseline workflow as broadcasting transactions to all nodes, collecting them into blocks, and propagating accepted blocks so nodes can continue working on the next block.[^ref-btc-wp]

In modern Bitcoin, propagation is more structured than the whitepaper's short description. Nodes exchange a `version` message, acknowledge with `verack`, advertise reachable peers through `addr` or `addrv2`, synchronize chain knowledge with `getheaders` and `headers`, and request missing objects using `getdata` after receiving announcements such as `inv` or `headers`.[^ref-btc-dev-p2p]

Block propagation and transaction propagation have different operational goals. Block propagation is latency-sensitive because slow propagation raises stale-block risk for miners and slows global convergence on the most-work tip. Transaction propagation is more policy-sensitive because nodes may reject, delay, or avoid relaying transactions that fail local mempool or anti-abuse policy even when those transactions are not consensus-invalid.[^ref-btc-core-net-processing][^ref-btc-core-txrequest]

Modern relay also separates multiple concerns:

- Discovery: address relay and address management.
- Session setup: handshake, service negotiation, relay preferences.
- Chain sync: headers-first synchronization and block download.
- Unconfirmed transaction relay: announcements, requests, orphan handling, fee filters, and relay suppression.
- Efficiency and robustness: `sendheaders`, compact blocks, `wtxid` relay, and selective peer behavior.[^ref-bip-0130][^ref-bip-0152][^ref-bip-0339]

For analysts, propagation is critical because timing, topology, and node policy shape when transactions become visible, when blocks become dominant, and how much mempool data one vantage point can actually observe. A node's local view is never the network's full view.

---

## 4. Protocol Structure

### Node Roles

Bitcoin does not have a single authoritative node type. Instead, peers differ by what data they keep, what services they offer, and what policies they apply.

Analytically useful distinctions include:

- Full validating nodes: verify consensus rules and maintain enough chainstate to validate new blocks and transactions.
- Archival full nodes: validating nodes that retain historical block data rather than pruning old blocks.
- Pruned nodes: validating nodes that may discard older block files while keeping enough data to validate the active chain.
- Lightweight clients: may track headers and request proofs or filtered data rather than validating the full chain.
- Mining nodes or pool infrastructure: highly latency-sensitive peers that optimize block reception and announcement.

Service bits in the `version` message help peers advertise capabilities such as serving the full network history, serving limited recent blocks, or supporting newer transport or relay features.[^ref-btc-dev-p2p][^ref-bip-0159][^ref-bip-0324]

### Connection Lifecycle

At a high level, peer interaction follows this sequence:

```text
peer discovery
-> outbound connection attempt
-> version exchange
-> verack exchange
-> capability-dependent messages
-> headers / address / inventory / data exchange
-> steady-state relay
```

The `version` message communicates protocol version, service flags, timestamp, addresses, nonce, user agent, start height, and optionally the transaction relay preference flag defined by BIP37.[^ref-btc-dev-p2p][^ref-bip-0037]

### Message Families

Analytically, Bitcoin P2P messages fall into four broad families:

| Family | Example Messages | Primary Purpose |
|---|---|---|
| Session control | `version`, `verack`, `ping`, `pong` | Negotiate a usable connection and track liveness |
| Peer discovery | `addr`, `addrv2`, `getaddr`, `sendaddrv2` | Learn about reachable peers and transport formats |
| Chain synchronization | `getheaders`, `headers`, `getblocks`, `block`, `cmpctblock` | Learn chain state and download missing blocks |
| Data relay | `inv`, `getdata`, `tx`, `notfound`, `feefilter` | Announce and fetch transactions or blocks |

### Inventory-Based Relay

Many relay flows are inventory-based. A peer announces object hashes with `inv`. The receiving peer compares those hashes against local state and uses `getdata` to request unknown objects. The actual payload then arrives in a message such as `tx`, `block`, or `cmpctblock`.[^ref-btc-dev-p2p]

This design reduces redundant transfer because nodes do not send full objects to every peer unconditionally.

### Headers-First Synchronization

Headers-first synchronization changes block synchronization from "download many full blocks first" to "download and verify headers first, then fetch blocks that extend promising chains." The `getheaders` message requests up to 2,000 headers at a time, and each `headers` entry contains an 80-byte block header plus a zero transaction count byte.[^ref-btc-dev-p2p]

This means a node can cheaply evaluate proof of work, linkage, and chainwork progression before spending bandwidth on full block bodies.

---

## 5. Peer Discovery and Session Establishment

### Address Relay

Nodes that accept inbound connections advertise peer addresses with `addr` or, in newer format negotiation, `addrv2`. `getaddr` requests additional peer addresses. These messages support bootstrapping and network graph refresh rather than consensus.[^ref-btc-dev-p2p][^ref-bip-0155]

Address information is advisory, not authoritative. It is unauthenticated and may be stale, incomplete, or malicious.

### Address Management

Bitcoin Core uses `AddrMan`, a stochastic address manager, to track and score known peers rather than treating every observed address as equally valuable.[^ref-btc-core-net]

This matters operationally because peer selection influences:

- Exposure to eclipse risk.
- Geographic and topological diversity.
- Propagation latency.
- Reliability of future outbound connections.

### Handshake Semantics

The `version`/`verack` exchange establishes a session before ordinary traffic is processed. The `version` message can signal:

- Protocol compatibility.
- Service capabilities.
- Peer software identity via user agent.
- Reported best height.
- Whether the sender wants transaction announcements by default.[^ref-btc-dev-p2p][^ref-bip-0037]

This handshake is capability discovery, not trust establishment. A remote peer can lie about services, height, or identity.

### Liveness and Flow Control

`ping`/`pong` help monitor connection health and round-trip time. This affects peer eviction, request timing, and relay quality more than it affects consensus itself.[^ref-btc-dev-p2p]

---

## 6. Block and Header Propagation

### New Block Announcement Paths

Historically, new blocks were often announced with `inv`, after which the receiving node requested the full block with `getdata`. BIP130 introduced `sendheaders`, allowing a peer to prefer direct `headers` announcements for new blocks.[^ref-bip-0130][^ref-btc-dev-p2p]

The difference matters:

- `inv` announces only a hash and requires an extra round trip to learn the header.
- `headers` directly provides the new header, allowing immediate proof-of-work and parent-link checks.

For block races, even one round trip matters.

### Headers-First Chain Extension

When a node receives new headers, it can:

1. Verify each header's format and proof of work.
2. Verify that each header connects to a known parent or extends a candidate branch.
3. Update cumulative chainwork estimates.
4. Decide whether the branch is worth downloading in full.

This is why header propagation is the network's first convergence layer and full block download is the second.

### Full Block Fetch

After a promising header is accepted, the node requests the corresponding block body with `getdata`. Full block processing then moves from transport into validation logic: syntax checks, Merkle checks, witness commitment checks, transaction checks, and contextual checks.[^ref-btc-core-net-processing][^ref-btc-core-validationinterface]

### Compact Blocks

BIP152 compact block relay reduces the bandwidth and latency of new block transfer by sending a block header, short transaction identifiers, and only the transactions likely missing from the receiving node.[^ref-bip-0152][^ref-btc-dev-p2p]

The compact block model assumes the receiving node already has many candidate transactions in mempool. Instead of transferring the full block again, the sender transmits enough information to reconstruct it locally, requesting only the missing transactions if reconstruction fails.

This is operationally important for miners because lower block propagation delay reduces stale-block probability.

---

## 7. Transaction Propagation

### Basic Transaction Relay

Transaction relay usually begins with an announcement rather than a direct payload push:

```text
peer A learns tx
-> peer A announces tx hash
-> peer B decides whether to request
-> peer B sends getdata
-> peer A sends tx
-> peer B validates and maybe relays onward
```

This decouples network awareness from bandwidth-heavy transfer.

### `txid` vs `wtxid`

SegWit introduced witness data that does not affect the legacy `txid` but does affect the `wtxid`. BIP339 updates relay semantics so transactions can be announced and requested by `wtxid`, reducing ambiguity and making relay behavior align better with SegWit-era transaction identity.[^ref-bip-0339]

Analytically, this matters because a mempool observer reasoning only in legacy `txid` terms may miss relay-layer distinctions that matter for modern transaction transport.

### Fee Filters and Relay Suppression

BIP133 introduced `feefilter`, allowing a peer to tell its counterparty not to send announcements for transactions below a chosen feerate threshold.[^ref-bip-0133][^ref-btc-dev-p2p]

This does not change consensus validity. It changes what a peer chooses to hear about in steady-state transaction relay.

### Orphan Transactions

A transaction can arrive before one of its parents. In that case, the receiving node may classify it as an orphan candidate rather than immediately treating it as fully processable. Bitcoin Core's orphan handling limits memory use and peer abuse because an attacker can intentionally flood dependency-missing transactions.[^ref-btc-core-txorphanage]

Important distinction:

- Orphan in relay context: missing referenced inputs from the receiver's current view.
- Invalid in consensus context: structurally or semantically violates protocol rules.

These are not the same.

### Transaction Request Scheduling

Nodes should not request every announced transaction from every peer. Bitcoin Core uses transaction request management to coordinate which peer should be asked for which object, reducing redundant requests and helping defend against delay or flood patterns.[^ref-btc-core-txrequest][^ref-btc-core-net-processing]

---

## 8. Technical Mechanics

### Message Container

Bitcoin P2P messages share a common message-header container consisting of:

```text
start string
command name
payload size
checksum
payload
```

The start string identifies the network, the command identifies the message type, and the checksum is derived from the payload. This common framing applies across control, relay, and synchronization messages.[^ref-btc-dev-p2p]

### Inventories

An inventory entry is a 36-byte structure:

```text
4 bytes   type identifier
32 bytes  object hash
```

Inventory type distinguishes objects such as transactions, blocks, filtered blocks, compact blocks, and witness variants.[^ref-btc-dev-p2p]

### Headers Download Limits

The Developer Reference documents two practical sync asymmetries:

- `getblocks` replies contain at most 500 block hashes.
- `headers` replies can contain up to 2,000 block headers.[^ref-btc-dev-p2p]

This encourages headers-first synchronization as the efficient path for discovering the best chain.

### Service Bits and Capability Surfaces

Service bits are not mere metadata. They define what a peer claims it can support: full blocks, limited-history blocks, new transport, and similar features. Peers use these claims to decide who is suitable for specific requests, but the claims themselves remain untrusted until behavior confirms them.[^ref-btc-dev-p2p][^ref-bip-0159][^ref-bip-0324]

### Relay Preference Signaling

Relay-related messages express preferences, not universal obligations:

- `sendheaders`: prefer header announcements for new blocks.[^ref-bip-0130]
- `feefilter`: suppress low-fee transaction announcements to this peer.[^ref-bip-0133]
- `sendaddrv2`: prefer the newer address message format.[^ref-bip-0155]

These are local session controls. Different peers may choose different behaviors.

---

## 9. Validation Boundaries

### Transport, Relay, and Consensus Are Different Layers

A node can successfully receive a message from a peer without accepting its contents into mempool or chainstate. The decision tree is layered:

1. Transport layer: was the message well framed and parseable?
2. Relay layer: is this object worth requesting, storing, or forwarding?
3. Validation layer: does the object satisfy consensus and local policy?
4. Chainstate layer: does it become part of the active chain or only a side branch?

Confusing these layers leads to bad analysis.

### Header Acceptance Is Not Block Acceptance

A header can pass proof-of-work and linkage checks while its full block later fails on:

- Invalid transactions.
- Bad Merkle root.
- Bad witness commitment.
- Coinbase amount violation.
- Contextual rules such as height-dependent deployments.

So "the header propagated" does not mean "the block was globally valid."

### Mempool Admission Is Not Consensus

A transaction can be consensus-valid yet absent from many mempools because of:

- Minimum relay fee.
- Fee filter suppression.
- Package topology.
- Ancestor or descendant limits.
- Local anti-abuse policy.
- Temporary missing parents.

Conversely, a transaction seen in one mempool has not thereby gained any consensus status.

---

## 10. Security Assumptions and Failure Modes

### Latency Matters

Block propagation is a race against competing extensions of the same tip. If two miners find valid competing blocks near the same time, slower propagation increases the probability that one branch loses and becomes stale. Propagation quality therefore feeds directly into miner revenue variance and network convergence.

### Eclipse and Topology Manipulation

If an attacker can dominate a node's peer set, that node's view of mempool, headers, and blocks becomes distorted. This is the propagation-layer foundation of eclipse risk. The attack does not need to break proof of work. It only needs to control the victim's information environment long enough to delay or bias what the victim sees.

### Address Pollution

Because address relay is unauthenticated, malicious peers can advertise low-quality or attacker-controlled addresses. Address management and peer diversity exist partly to reduce the resulting concentration risk.

### Flooding and Resource Exhaustion

Propagation code must defend against:

- Address spam.
- Inventory floods.
- Orphan transaction floods.
- Redundant data requests.
- Slow-drip peers that waste outbound request slots.

Bitcoin Core's request scheduling, orphan limits, and peer-state tracking exist for these reasons.[^ref-btc-core-txrequest][^ref-btc-core-txorphanage][^ref-btc-core-net-processing]

### Privacy Leakage

First-seen timing, transaction announcement paths, bloom-filter usage, and peer-specific relay behavior can leak information about wallet ownership or network location. BIP37-style bloom filtering was a pragmatic SPV mechanism, but it has known privacy limitations.[^ref-bip-0037]

### Transport Upgrades

BIP324 specifies a version 2 encrypted transport protocol. Transport encryption improves resistance to some forms of passive inspection and message tampering at the connection layer, but it does not make peer announcements authoritative and does not remove topology-based attack surfaces.[^ref-bip-0324]

---

## 11. Mathematical or Economic Model

### Propagation Delay and Stale Risk

Let:

- `T_p` = propagation delay from first miner receipt to broad network awareness
- `lambda` = aggregate rate of competing block discoveries

Under a simple Poisson approximation, the probability that at least one competing block is found during propagation time is:

```text
P(competing block during propagation) = 1 - e^(-lambda * T_p)
```

This is only a rough model, but it captures the directional truth: slower propagation increases stale-block exposure.

### Relay Selectivity

If a node uses a local feerate floor `f_min`, then the set of announced transactions it receives is not:

```text
all consensus-valid unconfirmed transactions
```

It is closer to:

```text
transactions peers choose to announce
intersect
transactions meeting local relay preferences
intersect
transactions not lost to timing, topology, or outages
```

This is why mempool measurements from one node are biased samples rather than a canonical global pool.

### Headers-First Bandwidth Intuition

Downloading headers first is cheap relative to downloading full blocks:

```text
1 header = 80 bytes
2,000 headers = 160,000 bytes before framing overhead
```

That is small compared with full block bodies. So headers-first synchronization lets nodes cheaply evaluate candidate chain extensions before committing heavier bandwidth and validation work.

---

## 12. Bitcoin Core Implementation

### `protocol.h`

`protocol.h` defines message types, inventory semantics, service flags, and related protocol constants. This is the schema layer for peer-to-peer communication.[^ref-btc-core-protocol]

### `net.h` and `net.cpp`

`net.h` and `net.cpp` provide the network transport and peer connection substrate, including serialized network messages, node objects, and connection management.[^ref-btc-core-net]

At this layer, Core is still mostly dealing with:

- sockets and sessions,
- message queues,
- peer bookkeeping,
- address handling,
- transport abstractions.

### `net_processing`

`net_processing` is the relay and peer-state brain. This is where Bitcoin Core coordinates message handling, block and transaction announcements, request logic, peer misbehavior responses, and synchronization behavior.[^ref-btc-core-net-processing]

Analytically, this is where raw network traffic turns into operational decisions about:

- which peer to trust for a given object request,
- whether to announce now or later,
- whether a peer is stalling or flooding,
- whether to prefer headers, compact blocks, or other relay paths.

### `txrequest`

`txrequest` manages transaction request scheduling, making sure Core avoids naive duplicate requests and tracks per-peer announcement/request state.[^ref-btc-core-txrequest]

### `txorphanage`

`txorphanage` manages transactions that failed because inputs are currently missing from local view. It explicitly limits announcement counts, memory usage, and dependency-related processing cost, reflecting the fact that orphan handling is a DoS-sensitive network edge.[^ref-btc-core-txorphanage]

### `validationinterface`

`CValidationInterface` is not the network layer itself, but it is important because it exposes validation and mempool events to subscribers. It marks the boundary where network-received data has progressed into validation outcomes or chainstate updates.[^ref-btc-core-validationinterface]

---

## 13. Consensus, Policy, and Presentation

### Consensus Layer

Consensus determines whether a block or transaction is valid for the Bitcoin system.

Examples:

- block proof of work,
- transaction syntax under consensus rules,
- script validity,
- subsidy limits,
- witness commitment correctness.

### Policy Layer

Policy determines whether an otherwise acceptable object is worth relaying, storing, or mining from a specific node's perspective.

Examples:

- fee floors,
- mempool package limits,
- orphan retention limits,
- relay suppression,
- peer-specific announcement choices.

### Presentation Layer

Presentation is what an observer or API sees. A wallet, explorer, or analyst's node view depends on:

- which peers were connected,
- when they were connected,
- what filters or preferences were active,
- whether the node was pruned,
- whether the node was in initial block download.

Observed network state is therefore always vantage-point dependent.

---

## 14. On-Chain Implications

Propagation is mostly off-chain behavior, but it leaves indirect analytical footprints.

### Mempool Visibility Bias

A transaction's first appearance time in one dataset may reflect:

- the observer's topology,
- the observer's fee filters,
- peer-specific delay,
- orphan dependency timing,
- BIP339 or compact-relay era behavior.

It does not necessarily equal the time the transaction first entered "the Bitcoin network."

### Block Arrival Bias

A block's observed arrival time at a node is a local receipt time, not the objective creation time. Analysts comparing miner speed or geographic advantage need multiple measurement points before drawing conclusions.

### Fork and Reorg Sensitivity

Short-lived forks are propagation events before they become chain-history facts. Slow or asymmetric propagation can increase transient disagreement about the tip even when consensus eventually resolves cleanly to the most-work branch.

---

## 15. Institutional Thinking

Institutions that rely on mempool or propagation data should treat the network as a partially observed system.

### Practical Implications

- One-node mempool feeds are insufficient for strong market or compliance claims.
- Propagation latency matters for miner monitoring, exchange deposit risk, and high-frequency fee estimation.
- Network vantage quality is a core data-quality issue, not a minor infrastructure detail.
- Relay-policy changes in Bitcoin Core can materially change observed transaction populations without changing Bitcoin consensus.

### Better Research Posture

For serious propagation analysis, institutions usually need:

- multiple geographically distributed nodes,
- clear logging of peer composition and software versions,
- separation of local receipt time from estimated origin time,
- awareness of policy toggles such as pruning, blocksonly, and fee thresholds.

---

## 16. Common Misinterpretations

### "If my node did not see it, the network did not see it"

False. Your node has a partial, path-dependent view.

### "Relayed means valid"

False. A relayed object can later fail policy, full validation, or contextual checks.

### "Not in mempool means invalid"

False. The transaction may be below fee thresholds, missing parents, filtered, delayed, or excluded by local policy.

### "A peer's announced height or services can be trusted"

False. These fields are unauthenticated claims.

### "Transport encryption solves propagation attacks"

False. It can improve confidentiality and robustness at the wire level, but peer selection and topology attacks still matter.

---

## 17. Research Questions

1. How much variance exists across geographically distributed first-seen transaction timestamps?
2. How has BIP339 changed empirical relay convergence for SegWit-heavy transaction flows?
3. How often do compact block reconstructions fail under fee spikes or mempool fragmentation?
4. How much of short-lived fork incidence is explained by propagation asymmetry rather than hashpower distribution?
5. How much address-manager diversity is needed to materially reduce eclipse exposure in practice?

---

## 18. Practical Exercises

### Exercise 1

Capture `version`, `verack`, `inv`, `getdata`, `headers`, and `block` messages from a controlled test environment and map the sequence for both block sync and transaction relay.

### Exercise 2

Run multiple nodes with different fee filters and compare the set and timing of first-seen transactions over the same window.

### Exercise 3

Measure first-seen block arrival time across nodes in different regions and estimate the distribution of propagation delay.

### Exercise 4

Compare transaction visibility before and after enabling a block-only node profile to understand the difference between chain sync and mempool relay.

---

## 19. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| `version`/`verack`, `addr`, `inv`, `getdata`, `headers`, and `block` message roles | Directly specified | Bitcoin Developer Reference and Bitcoin Core protocol sources |
| `sendheaders`, `feefilter`, compact blocks, `addrv2`, limited-history serving, and v2 transport roles | Directly specified | Relevant BIPs |
| Request scheduling, orphan limits, and peer-state handling in Core | Directly specified | Bitcoin Core Doxygen sources |
| Relay-policy and vantage-point implications for analysts | Inference from sources | Derived from protocol behavior and node-local policy design |
| Latency and stale-risk relationship | Analytical model | Directionally correct simplification, not a full empirical model |

---

## 20. Knowledge Graph

```text
Nodes and Network Propagation
├─ Peer Discovery
│  ├─ addr
│  ├─ addrv2
│  ├─ getaddr
│  └─ AddrMan
├─ Session Establishment
│  ├─ version
│  ├─ verack
│  ├─ service bits
│  └─ ping/pong
├─ Block Propagation
│  ├─ inv
│  ├─ headers
│  ├─ getheaders
│  ├─ block
│  ├─ cmpctblock
│  └─ chainwork comparison
├─ Transaction Propagation
│  ├─ inv
│  ├─ getdata
│  ├─ tx
│  ├─ wtxid relay
│  ├─ feefilter
│  └─ orphan handling
├─ Implementation
│  ├─ protocol.h
│  ├─ net.h / net.cpp
│  ├─ net_processing
│  ├─ txrequest
│  ├─ txorphanage
│  └─ validationinterface
└─ Risks
   ├─ eclipse
   ├─ flooding
   ├─ stale risk
   └─ privacy leakage
```

---

## 21. References

### Primary Sources

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Sections 5 and 8. https://bitcoin.org/bitcoin.pdf

[^ref-btc-dev-p2p]: Bitcoin Developer Reference, "P2P Network." https://developer.bitcoin.org/reference/p2p_networking.html

[^ref-bip-0037]: BIP37, "Connection Bloom filtering." https://github.com/bitcoin/bips/blob/master/bip-0037.mediawiki

[^ref-bip-0130]: BIP130, "sendheaders message." https://github.com/bitcoin/bips/blob/master/bip-0130.mediawiki

[^ref-bip-0133]: BIP133, "feefilter message." https://github.com/bitcoin/bips/blob/master/bip-0133.mediawiki

[^ref-bip-0152]: BIP152, "Compact Block Relay." https://github.com/bitcoin/bips/blob/master/bip-0152.mediawiki

[^ref-bip-0155]: BIP155, "addrv2 message." https://github.com/bitcoin/bips/blob/master/bip-0155.mediawiki

[^ref-bip-0159]: BIP159, "NODE_NETWORK_LIMITED service bit." https://github.com/bitcoin/bips/blob/master/bip-0159.mediawiki

[^ref-bip-0324]: BIP324, "Version 2 P2P Encrypted Transport Protocol." https://github.com/bitcoin/bips/blob/master/bip-0324.mediawiki

[^ref-bip-0339]: BIP339, "WTXID-based transaction relay." https://github.com/bitcoin/bips/blob/master/bip-0339.mediawiki

[^ref-btc-core-net]: Bitcoin Core Doxygen, `net.h` and `net.cpp`. https://doxygen.bitcoincore.org/net_8cpp.html

[^ref-btc-core-protocol]: Bitcoin Core Doxygen, `protocol.h` and `NetMsgType`. https://doxygen.bitcoincore.org/namespace_net_msg_type.html

[^ref-btc-core-net-processing]: Bitcoin Core Doxygen, `net_processing.cpp` and `net_processing.h`, including `PeerManager` and peer message processing references. https://doxygen.bitcoincore.org/

[^ref-btc-core-txrequest]: Bitcoin Core Doxygen, `txrequest.h` and transaction request coordination references. https://doxygen.bitcoincore.org/

[^ref-btc-core-txorphanage]: Bitcoin Core Doxygen, `txorphanage.h` and orphan transaction management references. https://doxygen.bitcoincore.org/txorphanage_8h.html

[^ref-btc-core-validationinterface]: Bitcoin Core Doxygen, `validationinterface.h` and `CValidationInterface`. https://doxygen.bitcoincore.org/class_c_validation_interface.html

### Supporting Interpretation Notes

- Where this document discusses analyst bias, local view asymmetry, or institutional measurement limits, those statements are inferences from the documented message flow, relay preferences, and node-local policy architecture rather than explicit protocol claims.

---

## 22. Cross References

### Previous

- BITCOIN-021 — Blocks and Block Headers

### Next

- BITCOIN-023 — Forks

### Related

- BITCOIN-006 — Whitepaper Section 5 — Network
- BITCOIN-009 — Whitepaper Section 8 — Simplified Payment Verification
- BITCOIN-017 — Mempool
- BITCOIN-018 — Transaction Fees
- BITCOIN-021 — Blocks and Block Headers
- POW-011 — Cumulative Chainwork
- POW-013 — Bitcoin Core Proof-of-Work Validation
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- Peer discovery, handshake, header sync, block relay, and transaction relay were separated.
- Relay policy was distinguished from consensus validity.
- Modern relay mechanisms such as `sendheaders`, compact blocks, `addrv2`, `feefilter`, and `wtxid` relay were scoped as protocol or peer-service features rather than consensus rules.
- Bitcoin Core implementation references were limited to relevant networking and relay modules.

### Evidence Review

Passed.

- Baseline network workflow claims cite the whitepaper and Bitcoin Developer Reference.
- Message semantics cite the Bitcoin Developer Reference.
- Relay feature claims cite their respective BIPs.
- Core implementation claims cite Doxygen pages for `net`, `protocol`, `txorphanage`, and `validationinterface`.
- Analyst-facing interpretations are labeled as inference where appropriate.

### Editorial Review

Passed.

- Markdown structure follows the project deep-dive template.
- Metadata is complete.
- Terminology is consistent across node roles, relay, synchronization, and policy.
- Tables and code fences are closed.

### Adversarial Review

Passed.

- The document does not treat mempool visibility as a global truth.
- It does not treat peer claims as authenticated facts.
- It does not treat relay acceptance as consensus validity.
- It does not treat transport encryption as a complete defense against topology attacks.
- It does not claim the whitepaper fully specifies modern Bitcoin relay behavior.

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
