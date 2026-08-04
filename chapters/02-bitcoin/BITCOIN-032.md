---
knowledge_id: BITCOIN-032
title: Lightning Network
subtitle: Off-Chain Payment Channels, Commitment Transactions, HTLC Routing, Revocation, and the Boundary Between Base-Layer Enforcement and Layer-2 State
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 145 min
estimated_study: 420 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Layer 2
  - Lightning
  - Payments
parent:
  - Modern Bitcoin
prerequisites:
  - BITCOIN-009
  - BITCOIN-016
  - BITCOIN-022
  - BITCOIN-030
  - BITCOIN-031
related_topics:
  - Payment Channels
  - HTLC
  - Commitment Transaction
  - Routing
  - CSV
  - CLTV
  - Watchtowers
primary_sources:
  - REF-LN-PAPER-001
  - REF-BOLT-000-INTRO-001
  - REF-BOLT-002-PEER-001
  - REF-BOLT-003-TX-001
  - REF-BOLT-004-ROUTING-001
  - REF-BOLT-007-GOSSIP-001
  - REF-BIP-0112
  - REF-BTC-CORE-BIPS-001
tags:
  - bitcoin
  - lightning
  - layer2
  - payment-channels
  - htlc
  - routing
  - commitment-transactions
  - revocation
---

# Lightning Network
> Modern Bitcoin  
> Research Unit: BITCOIN-032

---

## Research Brief

```yaml
knowledge_id: BITCOIN-032
title: Lightning Network
research_question: >
  How does the Lightning Network move payments off-chain through bidirectional
  payment channels, what roles do commitment transactions, revocation,
  HTLCs, routing, and gossip play, and how should analysts separate Lightning's
  off-chain state from the on-chain enforcement and observability limits of the
  Bitcoin base layer?
document_type: deep-dive
difficulty: L400
prerequisites:
  - BITCOIN-009
  - BITCOIN-016
  - BITCOIN-022
  - BITCOIN-030
  - BITCOIN-031
parent: Modern Bitcoin
previous: BITCOIN-031
next: BITCOIN-033
related_topics:
  - Payment Channels
  - HTLCs
  - Routing Gossip
  - On-chain Enforcement
  - Watchtowers
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
  - Full BOLT-by-BOLT implementation tutorial
  - Detailed Lightning liquidity management strategies
  - Full watchtower protocol landscape
  - AMP / trampoline / blinded-path deep dive
  - Alt-L2 comparisons
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Explain Lightning as a network of off-chain Bitcoin payment channels.
- Distinguish on-chain channel funding and settlement from off-chain balance updates.
- Explain what commitment transactions do.
- Explain why revocation and delay mechanisms matter for channel security.
- Explain HTLCs as the mechanism that lets payments be routed trustlessly across channels.
- Distinguish private channel state from public network-gossip information.
- Explain what is and is not visible from base-layer chain data.

---

## 2. Key Questions

1. What is the Lightning Network?
2. Why does Lightning need Bitcoin's base layer at all?
3. What is a payment channel?
4. What is a commitment transaction?
5. What is a revoked commitment transaction and why is it dangerous to broadcast?
6. What do HTLCs do in routed payments?
7. What are channel gossip and `short_channel_id` used for?
8. What can on-chain analysts observe about Lightning, and what remains off-chain?

---

## 3. Executive Summary

The Lightning Network is a layer-2 protocol for moving many Bitcoin payments off-chain through a network of payment channels, relying on the Bitcoin blockchain only for channel funding, dispute enforcement, and final settlement when necessary.[^ref-ln-paper][^ref-bolt-000]

Two participants open a channel by locking bitcoin in a jointly controlled on-chain output. They then exchange updated off-chain commitment states that reallocate the channel balance without broadcasting each update to the blockchain. If cooperation holds, many payments can occur with only occasional on-chain settlement.[^ref-ln-paper][^ref-bolt-000][^ref-bolt-003]

Lightning's core security trick is that every participant always retains an on-chain enforceable exit path. Commitment transactions, revocation logic, `to_self_delay`, and HTLC scripts make it costly or dangerous to cheat by publishing outdated state. BIP112's `CHECKSEQUENCEVERIFY` is one of the base-layer primitives explicitly cited as enabling this style of channel design.[^ref-bip-0112][^ref-bolt-003]

For routed payments, Lightning uses HTLCs so that an intermediary only forwards value if it can safely reclaim value from the incoming hop. The network layer then uses peer messaging, onion routing, and gossip to discover channels and move payments across multiple hops without requiring every payment to hit the Bitcoin blockchain.[^ref-bolt-002][^ref-bolt-004][^ref-bolt-007]

For analysts, Lightning is a sharp observability break. The blockchain can show channel opens, closes, and some enforcement events. It does not show every off-chain rebalance, attempted route, invoice, failure, or intermediate channel state.

---

## 4. Protocol Structure

### Layer Split

Lightning is best understood as two coupled layers:

| Layer | What Happens There |
|---|---|
| Bitcoin base layer | funding outputs, unilateral closes, mutual closes, HTLC enforcement, penalties, final settlement |
| Lightning layer | channel state updates, HTLC forwarding, routing, gossip, invoices, peer messaging |

### Channel Lifecycle

At a high level:

```text
open channel on-chain
-> exchange off-chain states
-> forward and settle off-chain payments
-> close cooperatively or force-close on-chain
```

### Why Base Layer Still Matters

Lightning does not replace Bitcoin finality. It outsources frequent state changes off-chain, but falls back to Bitcoin consensus when parties disagree or when settlement must be enforced on-chain.[^ref-bolt-000]

---

## 5. Payment Channels

### Funding Output

A Lightning channel begins with a funding transaction that locks bitcoin into a jointly controlled output. BOLT #3 defines the funding transaction output and the on-chain transaction/script formats required for the channel participants to agree on enforceable state.[^ref-bolt-003]

### Commitment Transactions

Each side holds a commitment transaction representing the latest enforceable split of channel funds. These transactions are not all broadcast; they are pre-signed exit states that can be used if cooperation fails.[^ref-bolt-000][^ref-bolt-003]

### State Updates

As new payments occur, peers negotiate updated commitment states and revoke prior ones. BOLT #2 describes the message flow, including `commitment_signed` and `revoke_and_ack`, by which updates become irrevocably committed between peers.[^ref-bolt-002]

### Revocation

Old commitment transactions become dangerous for the party who tries to broadcast them later. BOLT #0 and BOLT #3 describe revoked commitment transactions and penalty spending paths that allow the other party to punish cheating.[^ref-bolt-000][^ref-bolt-003]

---

## 6. HTLCs and Routed Payments

### HTLC Purpose

Hash Time-Locked Contracts let a payment be offered conditionally: the receiver (or a downstream hop) must reveal a preimage before a timeout, otherwise the sender can recover the funds. BIP112 explicitly uses HTLCs and bidirectional payment channels as motivation and application context for `CHECKSEQUENCEVERIFY`.[^ref-bip-0112]

### Routed Safety Condition

BOLT #2 requires that a node forwarding an HTLC must not create a position where the outgoing payment can be claimed while the corresponding incoming payment cannot be reclaimed. This is why CLTV deltas, payment hashes, and commitment synchronization matter.[^ref-bolt-002]

### Onion Routing

BOLT #4 defines the onion-routing packet format used to carry per-hop forwarding instructions. The routing packet commits to the `payment_hash` as associated data, helping prevent replay of the same onion against a different payment hash.[^ref-bolt-004][^ref-bolt-002]

### End Result

HTLCs turn pairwise channels into a payment network:

```text
A pays B conditionally
B pays C conditionally
C reveals preimage
-> C gets paid
-> B safely claims upstream
-> A's payment completes
```

without B needing to trust C or A needing to trust B with custody of the whole route payment.

---

## 7. Commitment and Penalty Mechanics

### `to_self_delay`

BOLT #3 specifies that outputs returning funds to the owner of a commitment transaction are delayed by `to_self_delay` blocks so the counterparty has time to react if a revoked commitment transaction is published.[^ref-bolt-003]

### Penalty Path

If a revoked commitment transaction is broadcast, the counterparty can use revocation-related keys to spend the outputs immediately. This is the practical enforcement engine that makes outdated-state publication expensive and generally irrational.[^ref-bolt-000][^ref-bolt-003]

### Why This Is Layer-2 Security, Not Base-Layer Magic

Bitcoin does not "know" about Lightning balances. It only validates the transactions and scripts broadcast on-chain. Lightning security arises from how those transactions and scripts are constructed ahead of time and how parties monitor the chain for breaches.

---

## 8. Gossip, Routing, and Public Topology

### Public Channel Discovery

BOLT #7 defines node and channel discovery, including:

- `channel_announcement`,
- `channel_update`,
- `node_announcement`,
- and `short_channel_id` based on funding transaction location.[^ref-bolt-007]

### `short_channel_id`

For public channels, `short_channel_id` encodes:

- block height,
- transaction index in block,
- output index in transaction.[^ref-bolt-007]

This creates a public bridge between Lightning topology and a confirmed funding outpoint on the base chain.

### Limits of Public Topology

Gossip shows public channels and public routing information. It does not reveal:

- private channel state,
- every routed payment,
- private channels not announced to the network,
- or the full real-time liquidity of a channel.

---

## 9. Technical Mechanics

### Channel Management Messages

BOLT #2 structures peer channel management into phases such as establishment, normal operation, and closing, with message families for HTLC updates, commitments, revocations, and fee updates.[^ref-bolt-002]

### On-Chain Transaction Format

BOLT #3 specifies:

- funding outputs,
- commitment transactions,
- HTLC-timeout and HTLC-success transactions,
- closing transaction variants,
- fee calculation and payment rules.[^ref-bolt-003]

### SegWit Dependence

The Lightning paper explicitly notes that a new sighash behavior that addresses malleability is needed for trustless chained off-chain updates, which is why SegWit is foundational to deployable Lightning designs.[^ref-ln-paper]

### Outsourced Monitoring

BOLT #3 explains key-derivation and revocation structure partly in terms of enabling outsourced watching for revoked transactions. This is the conceptual foundation behind watchtower-style services, even though watchtower protocol details live outside the base BOLT set used here.[^ref-bolt-003]

---

## 10. Validation Boundaries

### What Bitcoin Validates

Bitcoin validates:

- funding transactions,
- closing transactions,
- HTLC-enforcement transactions,
- timelocks and script conditions actually broadcast on-chain.

### What Lightning Maintains Off-Chain

Lightning maintains off-chain:

- latest channel balances,
- pending HTLC sets,
- routing attempts,
- invoice state,
- peer negotiation state.

### Consequence

An on-chain observer sees only boundary events:

- open,
- close,
- force-close,
- penalty or timeout style enforcement,
- maybe public channel linkage through gossip-derived identifiers.

Everything else is mostly off-chain state.

---

## 11. Security Assumptions and Failure Modes

### Honest Monitoring Requirement

Lightning assumes channel participants, or delegated watchers, monitor the chain often enough to react before delay windows expire. If a party misses a revoked-state publication and the punishment window closes, it may lose funds.

### Liquidity Constraint

Lightning is not simply "free instant Bitcoin." Payments are constrained by channel balances, routing availability, fee policies, CLTV windows, and topology knowledge.

### Routing Failure

A valid invoice or path discovery does not guarantee delivery. HTLCs can fail because of insufficient liquidity, fee changes, policy differences, timeouts, or peer unavailability.

### Public vs Private Channels

Public gossip improves route discovery, but channels may remain private or partially private. This improves privacy in some cases while reducing observability and route knowledge.

---

## 12. Mathematical or Economic Model

### Channel State Model

For a simple two-party channel with total capacity `C`:

```text
balance_A + balance_B = C
```

Off-chain updates reallocate the split while keeping total capacity fixed unless and until a new funding/splice event occurs.

### Forwarding Fee Intuition

Lightning forwarding is not free by default. BOLT #7 describes channel fee parameters such as:

```text
fee_base_msat
fee_proportional_millionths
```

so forwarded amount plus fee requirements depend on channel policy.[^ref-bolt-007]

### Settlement-Efficiency Intuition

If `N` transfers occur within a channel and only open and close hit the blockchain, then on-chain footprint grows far more slowly than payment count. This is Lightning's core scaling intuition, though actual liquidity, routing, and rebalancing costs limit the idealized picture.

---

## 13. Bitcoin Core Implementation

### Bitcoin Core Boundary

Bitcoin Core does not implement the Lightning protocol itself, but it implements many of the base-layer capabilities Lightning depends on. The implemented-BIP index records support for relevant primitives such as:

- BIP65 `CHECKLOCKTIMEVERIFY`,
- BIP68 relative lock-time,
- BIP112 `CHECKSEQUENCEVERIFY`,
- SegWit support needed for deployable Lightning-era transaction safety.[^ref-btc-core-bips]

### Why This Still Matters

Even without native Lightning-node logic in Bitcoin Core, the base client remains part of Lightning's foundation because Lightning channels ultimately settle and enforce through Bitcoin transactions, scripts, and timelocks validated by the base layer.

---

## 14. On-Chain Implications

### What Can Be Observed

Base-layer analysts can often observe:

- funding outputs,
- channel opens and closes,
- some public-channel identifiers,
- force-close behavior,
- timeout or penalty-like enforcement patterns.

### What Cannot Be Reliably Observed

Base-layer analysts usually cannot observe:

- every off-chain payment,
- intermediate channel balances,
- private forwarding history,
- failed route attempts,
- invoice semantics,
- real-time usable channel liquidity.

### Caution on Attribution

A public funding outpoint does not imply stable channel usefulness, current liquidity, or successful routing performance. Public topology is only a partial view of Lightning's effective payment graph.

---

## 15. Institutional Thinking

Institutions should analyze Lightning as a settlement-thinning network, not as a replacement for Bitcoin consensus.

### Practical Implications

- Channel exposure is operational exposure: monitoring, liquidity, and key-management discipline matter.
- On-chain analytics for Lightning need explicit uncertainty labels because most state is off-chain.
- Treasury and payment teams should distinguish channel capacity from immediately usable outbound or inbound liquidity.
- Incident playbooks should cover force-closes, stale-state risk, and watchtower dependence.

---

## 16. Common Misinterpretations

### "Lightning transactions are invisible Bitcoin transactions"

Overstated. Many off-chain state changes are not on-chain, but channel opens, closes, and enforcement still leave base-layer traces.

### "A channel's capacity equals currently spendable liquidity in either direction"

False. Capacity is total locked value, not identical to present outbound or inbound spendability for one side.

### "Lightning removes trust entirely"

False. It reduces custodial dependence and uses enforceable contracts, but participants still rely on monitoring, implementation correctness, routing assumptions, and time-sensitive response windows.

### "Bitcoin Core includes Lightning"

False. Bitcoin Core provides relevant base-layer primitives, but Lightning protocol logic lives in separate implementations.

### "All Lightning channels are publicly visible"

False. Public gossip covers announced channels; private channels and most channel-state evolution remain outside that view.

---

## 17. Research Questions

1. How much of economically relevant Lightning activity remains invisible to chain-only analysis?
2. Which public metrics best proxy for network usability when channel liquidity is mostly hidden?
3. How should institutions quantify stale-state monitoring risk and watchtower dependence?
4. How do force-close patterns vary across congestion regimes and fee spikes?

---

## 18. Practical Exercises

### Exercise 1

Explain the difference between a funding transaction, a commitment transaction, and a closing transaction.

### Exercise 2

Describe why `to_self_delay` exists and what risk it mitigates.

### Exercise 3

Given a public `short_channel_id`, identify which funding outpoint it references.

### Exercise 4

List five important Lightning facts that cannot be recovered from on-chain data alone.

---

## 19. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Channel, commitment, HTLC, gossip, and public discovery mechanics | Directly specified | Lightning paper and BOLT specs |
| CSV role in Lightning-style contracts | Directly specified | BIP112 |
| SegWit as deployability foundation | Directly specified plus inference | Lightning paper motivation and Bitcoin base-layer context |
| Off-chain observability limits | Inference from sources | Derived from what BOLTs place off-chain vs on-chain |

---

## 20. Knowledge Graph

```text
Lightning Network
├─ Base Layer Anchors
│  ├─ funding transaction
│  ├─ timelocks
│  ├─ CSV / CLTV
│  └─ final settlement
├─ Channel State
│  ├─ commitment transactions
│  ├─ revocation
│  ├─ to_self_delay
│  └─ force close
├─ Routed Payments
│  ├─ HTLCs
│  ├─ payment hash / preimage
│  ├─ onion routing
│  └─ forwarding fees
├─ Network Topology
│  ├─ channel_announcement
│  ├─ channel_update
│  ├─ node_announcement
│  └─ short_channel_id
└─ Limits
   ├─ off-chain opacity
   ├─ liquidity constraints
   ├─ monitoring requirement
   └─ route failure risk
```

---

## 21. References

### Primary Sources

[^ref-ln-paper]: Joseph Poon and Thaddeus Dryja, "The Bitcoin Lightning Network: Scalable Off-Chain Instant Payments," January 14, 2016. https://nakamotoinstitute.org/library/lightning-network/

[^ref-bolt-000]: Lightning BOLTs, BOLT #0 Introduction and Index. https://github.com/lightning/bolts/blob/master/00-introduction.md

[^ref-bolt-002]: Lightning BOLTs, BOLT #2 Peer Protocol for Channel Management. https://github.com/lightning/bolts/blob/master/02-peer-protocol.md

[^ref-bolt-003]: Lightning BOLTs, BOLT #3 Bitcoin Transaction and Script Formats. https://github.com/lightning/bolts/blob/master/03-transactions.md

[^ref-bolt-004]: Lightning BOLTs, BOLT #4 Onion Routing Protocol. https://github.com/lightning/bolts/blob/master/04-onion-routing.md

[^ref-bolt-007]: Lightning BOLTs, BOLT #7 P2P Node and Channel Discovery. https://github.com/lightning/bolts/blob/master/07-routing-gossip.md

[^ref-bip-0112]: BIP112, "CHECKSEQUENCEVERIFY," including Lightning-related motivation and HTLC / channel examples. https://github.com/bitcoin/bips/blob/master/bip-0112.mediawiki

[^ref-btc-core-bips]: Bitcoin Core `doc/bips.md`, implemented BIP index including BIP65, BIP68, BIP112, SegWit, and related primitives. https://github.com/bitcoin/bitcoin/blob/master/doc/bips.md

### Supporting Interpretation Notes

- Where this document discusses observability limits, institutional liquidity uncertainty, or payment-graph incompleteness, those statements are inferences from Lightning's off-chain design rather than explicit BOLT claims about analysts.

---

## 22. Cross References

### Previous

- BITCOIN-031 — Taproot

### Next

- BITCOIN-033 — Bitcoin Core

### Related

- BITCOIN-009 — Whitepaper Section 8 — Simplified Payment Verification
- BITCOIN-016 — Script & ScriptPubKey
- BITCOIN-022 — Nodes and Network Propagation
- BITCOIN-030 — SegWit
- BITCOIN-031 — Taproot

---

## Review Status

### Technical Review

Passed.

- Channel funding, commitment state, HTLC forwarding, revocation, and gossip were separated.
- Base-layer enforcement and off-chain state were explicitly distinguished.
- Lightning's dependence on Bitcoin primitives was described without implying Lightning is part of Bitcoin Core.
- Observability limits were included to prevent chain-only overinterpretation.

### Evidence Review

Passed.

- Lightning paper and BOLT specs support channel, HTLC, and routing claims.
- BIP112 supports CSV and Lightning-style contract motivation.
- Bitcoin Core `doc/bips.md` supports the base-layer implementation context.
- Analytical statements about hidden state and liquidity limits are labeled as inference.

### Editorial Review

Passed.

- Structure follows the project deep-dive format.
- Metadata is complete.
- Terminology is consistent: funding transaction, commitment transaction, revoked commitment, HTLC, `to_self_delay`, `short_channel_id`.
- Tables and code fences are closed.

### Adversarial Review

Passed.

- The document does not claim Lightning state is fully visible on-chain.
- It does not confuse total capacity with directional liquidity.
- It does not claim Lightning eliminates monitoring or all trust assumptions.
- It does not imply Bitcoin Core natively implements Lightning.
- It does not overstate what public gossip proves about actual payment success.

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
