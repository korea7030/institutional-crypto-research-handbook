---
knowledge_id: BITCOIN-017
title: Mempool
subtitle: Local Transaction Pools, Relay Policy, Fee Markets, RBF, Package Acceptance, and Cluster Mempool
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 105 min
estimated_study: 300 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Mempool
  - Transaction Relay
  - Policy
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-014
  - BITCOIN-015
  - BITCOIN-016
related_topics:
  - Transaction Validation
  - Relay Policy
  - Replace-by-Fee
  - Child Pays For Parent
  - Package Relay
  - Cluster Mempool
  - Fee Estimation
  - Block Template Construction
primary_sources:
  - REF-BIP-0125
  - REF-BTC-CORE-26-RELEASE-001
  - REF-BTC-CORE-31-RELEASE-001
  - REF-BTC-CORE-TXMEMPOOL-001
  - REF-BTC-CORE-MEMPOOL-ENTRY-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-RBF-001
  - REF-BTC-CORE-POLICY-001
  - REF-BTC-CORE-RPC-MEMPOOL-001
  - REF-BTC-CORE-RPC-CONSISTENCY-001
tags:
  - bitcoin
  - internals
  - mempool
  - relay-policy
  - rbf
  - cpfp
  - package-relay
  - cluster-mempool
  - fee-market
---

# Mempool
> Bitcoin Internals  
> Research Unit: BITCOIN-017

---

## Research Brief

```yaml
knowledge_id: BITCOIN-017
title: Mempool
research_question: >
  What is Bitcoin's mempool, how do nodes accept, store, replace, evict,
  relay, and expose unconfirmed transactions, and how should institutional
  analysts distinguish local mempool observation from consensus settlement?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-014
  - BITCOIN-015
  - BITCOIN-016
parent: Bitcoin Internals
previous: BITCOIN-016
next: BITCOIN-018
related_topics:
  - UTXO Model
  - Transactions
  - Script
  - Transaction Fees
  - Mining
  - Reorganizations
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
  - Full P2P transaction relay protocol
  - Complete package relay specification
  - Miner-private transaction submission markets
  - Wallet fee-estimation implementation details
  - Non-Bitcoin mempool designs
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Define the mempool as a node-local set of unconfirmed transactions.
- Explain why there is no single global Bitcoin mempool.
- Distinguish consensus validity from mempool acceptance policy.
- Explain how local relay fee, mempool minimum fee, and eviction affect visibility.
- Explain parent, child, conflict, replacement, cluster, and chunk terminology.
- Explain RBF as unconfirmed transaction replacement policy.
- Explain CPFP as package-level fee incentive rather than consensus rule.
- Identify current Bitcoin Core mempool source areas.
- Interpret mempool RPC fields without treating them as global truth.
- Explain why mempool data is useful but fragile for institutional analysis.

---

## 2. Key Questions

1. What is the mempool?
2. Why do different nodes have different mempools?
3. What checks happen before a transaction enters a mempool?
4. What is the difference between consensus validity and relay policy?
5. What causes mempool eviction?
6. What does RBF replace?
7. How does CPFP change package incentives?
8. What changed with Bitcoin Core 31.0 cluster mempool?
9. How do RPCs expose mempool state?
10. What does mempool observation prove?
11. What does mempool observation not prove?
12. How should institutions use mempool data in settlement, risk, and fee workflows?

---

## 3. Executive Summary

The mempool is a node's local working set of unconfirmed transactions that the node considers valid according to the current best chain and acceptable under its local policy. It is not a consensus object and it is not globally identical across the network.

Bitcoin Core's `CTxMemPool` is documented as storing transactions valid according to the current best chain that may be included in the next block. Its interfaces are used by validation, net processing, RPC inspection, and block construction.[^ref-btc-core-txmempool]

Mempool acceptance is stricter and more local than block validity:

```text
consensus validity:
    can this transaction be included in a valid block?

mempool policy:
    will this node keep and relay this unconfirmed transaction now?
```

Policy may reject transactions that are consensus-valid. Examples include insufficient relay fee, nonstandard scripts, replacement rules, local capacity pressure, or cluster/package limits.

Bitcoin Core 31.0 reimplemented the mempool with cluster mempool. The release notes state that ancestor and descendant size/count limits are no longer enforced as before; instead, connected components called clusters are limited by default to 64 transactions and 101 kB virtual size, and transactions are ordered by mining feerate using chunks.[^ref-btc-core-31-release]

For analysts, mempool data is valuable for pending-flow monitoring, fee-market observation, replacement risk, and near-term settlement expectations. But it is probabilistic and local. Seeing a transaction in one node's mempool does not prove global propagation, miner acceptance, or future confirmation.

---

## 4. Protocol Structure

### Local State, Not Consensus State

The blockchain and UTXO set are consensus-relevant shared state. The mempool is local node state.

```text
confirmed chain:
    globally convergent through consensus

mempool:
    local, policy-dependent, time-sensitive
```

Two honest nodes can have different mempools because they may have:

- different peers;
- different arrival order;
- different policy settings;
- different fee thresholds;
- different memory pressure;
- different uptime;
- different chain-tip timing;
- different replacement configuration or version.

### Transaction Lifecycle

A typical unconfirmed transaction lifecycle is:

```text
wallet creates transaction
    -> node receives transaction
    -> node checks consensus and policy acceptance
    -> node stores transaction in mempool if accepted
    -> node relays transaction to peers if relay conditions apply
    -> miner may include transaction in block
    -> transaction leaves mempool when confirmed
```

Transactions can also leave the mempool without confirming.

Common removal reasons include:

- replaced by a conflicting transaction;
- evicted due to memory pressure;
- expired after time in mempool;
- conflicted by a block;
- invalidated by reorg-dependent state;
- removed with descendants or conflicts.

### Parent, Child, Cluster, and Chunk

Mempool relationships are defined by unconfirmed dependencies:

| Term | Meaning |
|---|---|
| Parent | In-mempool transaction whose output is spent by another in-mempool transaction |
| Child | In-mempool transaction spending an output from another in-mempool transaction |
| Conflict | Transaction spending the same input as another transaction |
| Cluster | Connected component of mempool transactions linked through parent/child relationships |
| Chunk | A group ordered by expected mining feerate within cluster mempool logic |

Bitcoin Core 31.0 release notes describe clusters as transactions connected by parent/child relationships and chunks as ordering units used for block construction, eviction, relay announcements, and replacement evaluation.[^ref-btc-core-31-release]

---

## 5. Acceptance Mechanics

### Mempool Acceptance

Bitcoin Core exposes `AcceptToMemoryPool` in `validation.cpp`. Doxygen describes it as trying to add a transaction to the mempool and returning a `MempoolAcceptResult` indicating acceptance or rejection with reason.[^ref-btc-core-validation]

At a high level, mempool acceptance checks:

```text
transaction structure
UTXO availability against chainstate plus mempool view
script validity under required flags
finality and sequence-lock state
standardness policy
fee and relay policy
conflicts and replacement policy
package or cluster limits
```

This is not a single consensus check. It is a pipeline combining consensus validation, local policy, and mempool-state checks.

### Chainstate Plus Mempool View

Unconfirmed transactions can depend on other unconfirmed transactions. A node therefore needs a view that can answer whether an input spends:

- a confirmed UTXO in chainstate;
- an unconfirmed output created by another mempool transaction;
- an unavailable or already-conflicted output.

Bitcoin Core defines `CCoinsViewMemPool` as a coins view backed by a base view and a mempool, with `GetCoin` access for outpoints and package-add support.[^ref-btc-core-txmempool]

### Standardness

Standardness policy reduces denial-of-service risk and encourages predictable relay behavior. Bitcoin Core policy code includes standard transaction and script checks such as `IsStandard` and `IsStandardTx`.[^ref-btc-core-policy]

Important distinction:

```text
nonstandard under policy != consensus-invalid
```

A miner can include some consensus-valid nonstandard transactions in a block, even if many nodes would not relay them through their mempools.

### Fee Thresholds

Mempool acceptance depends on local fee thresholds:

- minimum relay feerate;
- incremental relay fee;
- dynamic mempool minimum fee under capacity pressure;
- node-specific policy settings;
- package feerate where package acceptance applies.

Bitcoin Core 31.0 release notes state that the fee estimator minimum fee rate bucket was updated to 0.1 sat/vB to match the default `minrelaytxfee`.[^ref-btc-core-31-release]

Fee policy changes over time and can differ across nodes. Analysts should record software version and policy assumptions when using mempool data.

---

## 6. Replacement and Fee Bumping

### Replace-by-Fee

BIP125 defines opt-in full replace-by-fee signaling. A transaction signals replaceability if any input has an `nSequence` less than `0xffffffff - 1`. It can also inherit replaceability from unconfirmed ancestors while those ancestors remain unconfirmed.[^ref-bip-0125]

RBF means:

```text
an unconfirmed transaction may be replaced
by a conflicting transaction
if replacement policy is satisfied
```

It does not mean:

- the original transaction was invalid;
- fraud necessarily occurred;
- all nodes will make the same replacement decision;
- the replacement is guaranteed to confirm.

### Bitcoin Core RBF Implementation

Bitcoin Core's `src/policy/rbf.cpp` includes `IsRBFOptIn`, which checks explicit signaling and inherited signaling through mempool ancestors. It also includes fee checks such as replacement paying at least original fees and paying for relay bandwidth.[^ref-btc-core-rbf]

Bitcoin Core 31.0 release notes state that replacement validation now requires the resulting mempool's feerate diagram to be strictly better than before the replacement.[^ref-btc-core-31-release]

This is a current-policy point. Historical analysis must check the Bitcoin Core version and policy environment active at the time.

### Child Pays For Parent

CPFP is the opposite direction of fee bumping:

```text
low-fee parent + high-fee child = package with higher combined incentive
```

Miners care about the fee they can earn by including dependent transactions together. A child cannot be mined unless its unconfirmed parent is also mined first, so the package's combined fee rate matters.

CPFP is an incentive mechanism and mempool/mining policy issue. It is not a special consensus exception.

### Package Acceptance

Package acceptance allows nodes to evaluate related transactions together in some contexts. Bitcoin Core 26.0 introduced `submitpackage` for evaluating raw transactions as a package under consensus and mempool policy, with package CPFP support and limitations.[^ref-btc-core-26-release]

Bitcoin Core 31.0 further changed package and cluster behavior under cluster mempool.[^ref-btc-core-31-release]

Analysts should not assume every node or miner accepts the same package forms.

---

## 7. Eviction, Expiry, and Reorgs

### Eviction

Mempools are bounded by local resource policy. When a node's mempool is under pressure, lower-incentive transactions or chunks can be evicted so the node stays within configured limits.

Bitcoin Core 31.0 states that cluster mempool ordering is used for eviction when the mempool is full.[^ref-btc-core-31-release]

Eviction is local. If a transaction disappears from one node's mempool, it may still exist in another node's mempool.

### Expiry

Nodes may remove transactions that remain unconfirmed too long. Expiry avoids retaining stale transactions indefinitely.

Expiry does not prove the transaction became invalid. It means the local node no longer keeps it in its mempool.

### Block Connection

When a block is connected:

- transactions included in the block are removed from the mempool;
- transactions conflicting with block transactions are removed;
- descendants of removed transactions may also be affected;
- fee estimator and wallet views may update.

Bitcoin Core's `CTxMemPool` class reference notes interfaces to validation for updating the mempool after a new block is connected or after a reorg.[^ref-btc-core-txmempool]

### Reorgs

A reorg can return previously confirmed transactions to an unconfirmed state if they are not included in the new active chain. Nodes may try to re-add disconnected transactions to the mempool if still valid under the new chainstate and policy.

This is another reason mempool observation should be timestamped and tied to a specific node.

---

## 8. Mathematical or Economic Model

### Fee Rate

For a transaction:

```text
fee_rate = fee / virtual_size
```

For a package or chunk:

```text
package_fee_rate = sum(fees) / sum(virtual_sizes)
```

Fee rate is the price signal for scarce block space. It is not a consensus rule by itself.

### Parent and Child Example

```text
parent:
    fee = 500 sats
    vsize = 500 vB
    fee_rate = 1 sat/vB

child:
    fee = 9,500 sats
    vsize = 250 vB
    fee_rate = 38 sat/vB

combined:
    fee = 10,000 sats
    vsize = 750 vB
    fee_rate = 13.33 sat/vB
```

If miners evaluate parent and child together, the child can make the parent economically attractive.

### Replacement Example

```text
original:
    fee = 5,000 sats
    vsize = 250 vB
    fee_rate = 20 sat/vB

replacement:
    fee = 8,000 sats
    vsize = 250 vB
    fee_rate = 32 sat/vB
```

Whether the replacement is accepted depends on current node policy, conflicts, cluster effects, and relay fee requirements.

### Mempool Observation Bias

Let:

```text
M_i(t) = mempool of node i at time t
G(t) = unobservable global set of propagated unconfirmed transactions
```

No single `M_i(t)` equals `G(t)`. A mempool sample is a local observation:

```text
M_i(t) is evidence, not ground truth
```

This matters for fee estimation, pending-flow dashboards, and liquidation or settlement monitoring.

---

## 9. Bitcoin Core Implementation

### `CTxMemPool`

Bitcoin Core's `CTxMemPool` stores accepted unconfirmed transactions. Doxygen states that entries are stored in `mapTx`, a boost multi-index container sorted by transaction hash, witness transaction hash, and time in mempool. It also maintains `mapNextTx`, mapping an outpoint to the in-mempool transaction that spends it.[^ref-btc-core-txmempool]

These structures support:

- transaction lookup;
- conflict detection;
- reorg handling;
- relay inventory selection;
- RPC inspection;
- block template construction.

### `CTxMemPoolEntry`

Bitcoin Core's mempool entry data tracks transaction-related metadata such as fee, virtual size, entry height, and lock points. `mempool_entry.h` defines `LockPoints` for BIP68 relative-locktime context and transaction information structures.[^ref-btc-core-mempool-entry]

This metadata is not part of transaction consensus serialization. It is node-local accounting.

### `AcceptToMemoryPool`

`AcceptToMemoryPool` is the validation entry point for trying to add a transaction to the mempool. It takes active chainstate, transaction reference, accept time, limit-bypass setting, and test-accept setting, then returns a `MempoolAcceptResult`.[^ref-btc-core-validation]

Important modes:

| Mode | Meaning |
|---|---|
| normal acceptance | validate and submit if accepted |
| `test_accept` | run checks without adding to mempool |
| bypass limits | used for special internal cases, not ordinary relay behavior |

### Policy and RBF

Bitcoin Core policy code covers standardness and relay behavior. RBF-specific policy code is in `src/policy/rbf.cpp`, including `IsRBFOptIn` and fee checks for replacement.[^ref-btc-core-rbf]

### RPC Observability

The `getrawmempool` RPC returns transaction ids in the node's memory pool. In verbose mode it returns fields such as virtual size, weight, fee information, entry time, entry height, ancestor and descendant fields, `wtxid`, dependencies, spending children, BIP125 replaceability, and unbroadcast status.[^ref-btc-core-rpc-mempool]

Bitcoin Core's JSON-RPC interface documentation warns that mempool-related RPC state may not be up to date with the current mempool state, while the returned mempool state is internally consistent with the chain state at the time of the call.[^ref-btc-core-rpc-consistency]

### Current Version Caveat

This document reflects Bitcoin Core 31.x behavior as of 2026-08-04. Mempool policy changes more frequently than consensus rules. Future releases may change:

- default relay feerates;
- replacement policy;
- package relay;
- cluster limits;
- RPC fields;
- mining template selection algorithms.

---

## 10. Consensus, Policy, and Mining

### Consensus

Consensus decides whether a block is valid.

Examples:

- transaction inputs are valid;
- scripts pass mandatory rules;
- no double spend occurs within the active chain;
- block weight and coinbase value are valid.

### Mempool Policy

Mempool policy decides whether a node keeps and relays an unconfirmed transaction.

Examples:

- standardness;
- relay fee;
- replacement policy;
- cluster limits;
- package acceptance;
- mempool size limit.

### Mining Policy

Mining policy decides which transactions miners include in candidate blocks.

Mining policy often resembles relay policy because miners run full nodes, but it can differ through:

- private transaction submission;
- pool-specific minimum fee;
- out-of-band fee agreements;
- block-template customizations;
- nonstandard transaction inclusion.

Mempool acceptance is therefore not a guarantee of confirmation.

---

## 11. On-Chain and Pre-Chain Implications

### Strong Evidence

Mempool data strongly supports claims about a specific node at a specific time:

- the node had transaction `txid` or `wtxid` in its mempool;
- the node's RPC reported a given fee, vsize, weight, and dependency set;
- the node considered the transaction BIP125-replaceable or not under its view;
- the node saw unconfirmed parent/child relationships;
- the node's local mempool sequence changed.

### Weak Evidence

Mempool data weakly supports claims about:

- global network propagation;
- miner intent;
- confirmation probability;
- whether a transaction will be replaced;
- whether a pending payment should be treated as final;
- whether an unseen transaction does not exist elsewhere.

### Not On-Chain Yet

Unconfirmed transactions are not on-chain. They are pre-chain network state. A transaction becomes on-chain only when included in a block that remains in the active chain.

This distinction is central for institutional reporting. A pending deposit and a confirmed deposit are not the same state.

---

## 12. Institutional Thinking

### Settlement Risk

Institutions should not credit high-risk value solely because a transaction appears in one mempool. Factors to evaluate include:

- confirmation depth;
- RBF signaling;
- conflicting transactions;
- fee rate relative to current market;
- unconfirmed ancestors;
- package dependency;
- observed propagation across multiple nodes;
- counterparty risk.

### Fee Management

Mempool monitoring supports:

- withdrawal batching decisions;
- CPFP rescue decisions;
- RBF fee bumping;
- consolidation timing;
- target confirmation fee estimation.

But fee estimation should account for local-node bias and rapidly changing block-space demand.

### Operations

Operational systems should record:

- transaction creation time;
- first-seen time per node;
- node software version;
- mempool policy settings if known;
- fee rate and package context;
- conflicts and replacements;
- confirmation block or removal reason.

### Compliance

Compliance dashboards often track pending inflows and outflows. Those systems should label unconfirmed data explicitly and avoid presenting mempool events as settled transfers.

---

## 13. Common Misinterpretations

### "The Mempool Is Global"

No. Each node has its own mempool.

### "In the Mempool Means Confirmed Soon"

No. It means one node accepted the transaction locally. Confirmation depends on miner selection, fee market conditions, conflicts, and future blocks.

### "Not in My Mempool Means Not Broadcast"

No. The transaction may exist elsewhere, may be below your node's policy threshold, may have arrived and been evicted, or may be privately submitted to miners.

### "RBF Means the Transaction Is Fraudulent"

No. RBF is replacement policy for unconfirmed transactions. It can be used for fee bumping or for adversarial replacement. Context determines risk.

### "Consensus Valid Means Relayable"

No. A transaction can be consensus-valid but fail local mempool policy.

### "Mempool Fee Rate Is a Fixed Market Price"

No. It is a local, time-varying signal. Different nodes and miners may have different thresholds and transaction sets.

---

## 14. Research Questions

1. How different are mempools across well-connected public nodes during fee spikes?
2. How often do transactions disappear from local mempools and later confirm?
3. How do RBF and CPFP affect institutional withdrawal operations?
4. How should systems report pending balances when ancestors are unconfirmed?
5. How does cluster mempool change fee-bumping and block-template analysis?
6. What evidence is required before treating a pending deposit as low-risk?
7. How do private relay and direct-to-miner submission weaken public mempool observation?

---

## 15. Practical Exercises

### Exercise 1: Inspect Local Mempool

Run:

```bash
bitcoin-cli getrawmempool true
```

For three transactions, record:

- `vsize`;
- `weight`;
- `fees.base`;
- `wtxid`;
- `depends`;
- `spentby`;
- `bip125-replaceable`;
- `unbroadcast`.

### Exercise 2: Compute Fee Rate

For one mempool transaction:

```text
fee_rate = fees.base / vsize
```

Compare the result to current fee estimates and recent block inclusion.

### Exercise 3: Identify Dependencies

Choose a transaction with non-empty `depends` or `spentby`.

Explain:

- which transaction is parent;
- which transaction is child;
- whether CPFP may be relevant;
- why the child cannot confirm before the parent.

### Exercise 4: Consensus vs Policy

Classify each statement:

| Statement | Consensus | Policy | Local observation | Analytics |
|---|---:|---:|---:|---:|
| Transaction appears in my node's mempool | No | Yes | Yes | Evidence |
| Transaction can be in a valid block | Yes | No | No | Fact if validated |
| Transaction signals BIP125 replaceability | No | Yes | Yes | Risk signal |
| Transaction will confirm next block | No | No | No | Forecast |
| Transaction is absent globally | No | No | No | Usually unknowable |

---

## 16. Evidence Classification

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BIP-0125 | Policy BIP | Opt-in full RBF signaling and replacement rules | A |
| REF-BTC-CORE-31-RELEASE-001 | Release documentation | Cluster mempool, cluster limits, chunk ordering, current fee-estimator note | A |
| REF-BTC-CORE-TXMEMPOOL-001 | Primary implementation source | `CTxMemPool`, map indexes, `mapNextTx`, `CCoinsViewMemPool` | A |
| REF-BTC-CORE-MEMPOOL-ENTRY-001 | Primary implementation source | `CTxMemPoolEntry`, `LockPoints`, mempool transaction metadata | A |
| REF-BTC-CORE-VALIDATION-001 | Primary implementation source | `AcceptToMemoryPool` and mempool acceptance result | A |
| REF-BTC-CORE-RBF-001 | Primary implementation source | `IsRBFOptIn` and replacement fee checks | A |
| REF-BTC-CORE-POLICY-001 | Primary implementation source | Standardness and relay policy checks | A |
| REF-BTC-CORE-RPC-MEMPOOL-001 | RPC documentation | `getrawmempool` fields and mempool sequence output | A |
| REF-BTC-CORE-RPC-CONSISTENCY-001 | Core documentation | RPC consistency guarantees and mempool-state caveats | A |
| REF-BTC-CORE-26-RELEASE-001 | Release documentation | `submitpackage` and package CPFP introduction | B |

### Claim Ledger

| Claim | Classification | Evidence |
|---|---|---|
| Mempool state is local node state, not consensus state | FACT | Bitcoin Core implementation and RPC consistency docs |
| `CTxMemPool` stores transactions valid according to current best chain that may be included in the next block | FACT | Bitcoin Core `CTxMemPool` documentation |
| Mempool entries are indexed by txid, wtxid, and entry time | FACT | Bitcoin Core `txmempool.h` documentation |
| `mapNextTx` maps outpoints to in-mempool spenders | FACT | Bitcoin Core `CTxMemPool` documentation |
| `AcceptToMemoryPool` tries to add a transaction and returns acceptance or rejection result | FACT | Bitcoin Core `validation.cpp` Doxygen |
| BIP125 replaceability is policy for unconfirmed transactions | FACT | BIP125 and Bitcoin Core RBF source |
| Bitcoin Core 31.0 uses cluster mempool and no longer enforces old ancestor/descendant size/count limits | FACT | Bitcoin Core 31.0 release notes |
| A transaction in one mempool is guaranteed to confirm | COUNTERCLAIM | Rejected; mempool observation is local and non-final |
| Absence from one mempool proves absence from the network | COUNTERCLAIM | Rejected; no single global mempool |
| CPFP changes incentives by considering dependent transactions together | INTERPRETATION | Derived from package fee-rate model and package acceptance docs |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| POLICY | Relay, mempool, or mining convention rather than consensus |
| HEURISTIC | Practical analysis rule with counterexamples |
| UNKNOWN | Evidence is insufficient |

---

## 17. Knowledge Graph

```text
BITCOIN-017 Mempool
|
+-- builds_on: BITCOIN-015 Transactions in Depth
+-- builds_on: BITCOIN-016 Script & ScriptPubKey
|
+-- mempool
|   +-- property: node-local
|   +-- stores: unconfirmed accepted transactions
|   +-- indexed_by: txid, wtxid, entry time
|
+-- acceptance
|   +-- consensus_checks
|   +-- policy_checks
|   +-- fee_checks
|   +-- replacement_checks
|
+-- relationships
|   +-- parent
|   +-- child
|   +-- conflict
|   +-- cluster
|   +-- chunk
|
+-- fee mechanics
|   +-- RBF
|   +-- CPFP
|   +-- package feerate
|
+-- Bitcoin Core
|   +-- CTxMemPool
|   +-- CTxMemPoolEntry
|   +-- AcceptToMemoryPool
|   +-- IsRBFOptIn
|
+-- analysis
    +-- facts: local first-seen, dependencies, fee rate
    +-- risks: replacement, eviction, non-confirmation
    +-- caveat: no global mempool
```

---

## 18. References

[^ref-bip-0125]: David A. Harding and Peter Todd, "BIP 125: Opt-in Full Replace-by-Fee Signaling," 2015-12-04, https://bips.dev/125/, accessed 2026-08-04.

[^ref-btc-core-31-release]: Bitcoin Core Contributors, "Bitcoin Core 31.0 Release Notes," mempool, cluster mempool, replacement, RPC, and fee-estimation changes, https://bitcoin.org/en/releases/31.0/, accessed 2026-08-04.

[^ref-btc-core-26-release]: Bitcoin Core Contributors, "Bitcoin Core 26.0 Release Notes," `submitpackage` and package CPFP notes, https://bitcoincore.org/en/releases/26.0/, accessed 2026-08-04.

[^ref-btc-core-txmempool]: Bitcoin Core Contributors, `src/txmempool.h`, `CTxMemPool`, `mapTx`, `mapNextTx`, `ChangeSet`, and `CCoinsViewMemPool`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/txmempool_8h_source.html and https://doxygen.bitcoincore.org/class_c_tx_mem_pool.html, accessed 2026-08-04.

[^ref-btc-core-mempool-entry]: Bitcoin Core Contributors, `src/kernel/mempool_entry.h`, `CTxMemPoolEntry`, `LockPoints`, and transaction info structures, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/mempool__entry_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-validation]: Bitcoin Core Contributors, `src/validation.cpp`, `AcceptToMemoryPool`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/validation_8cpp.html, accessed 2026-08-04.

[^ref-btc-core-rbf]: Bitcoin Core Contributors, `src/policy/rbf.cpp`, `IsRBFOptIn` and replacement fee checks, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/policy_2rbf_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-policy]: Bitcoin Core Contributors, `src/policy/policy.cpp`, standardness and relay policy checks, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/policy_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-rpc-mempool]: Bitcoin Developer Documentation, "`getrawmempool` RPC," mempool transaction fields, `wtxid`, dependency fields, BIP125 replaceability, and unbroadcast status, https://developer.bitcoin.org/reference/rpc/getrawmempool.html, accessed 2026-08-04.

[^ref-btc-core-rpc-consistency]: Bitcoin Core Contributors, "JSON-RPC interface," RPC consistency guarantees and transaction pool caveats, https://github.com/bitcoin/bitcoin/blob/master/doc/JSON-RPC-interface.md, accessed 2026-08-04.

---

## 19. Cross References

### Parent

- Bitcoin Internals

### Previous

- BITCOIN-016 — Script & ScriptPubKey

### Next

- BITCOIN-018 — Transaction Fees

### Related

- BITCOIN-014 — UTXO Model
- BITCOIN-015 — Transactions in Depth
- BITCOIN-016 — Script & ScriptPubKey
- BITCOIN-018 — Transaction Fees
- BITCOIN-023 — Chain Reorganization
- POW-009 — Coinbase Transaction and Mining Commitments
- POW-013 — Bitcoin Core Proof-of-Work Validation

---

## Review Status

### Technical Review

Passed.

- Mempool state was separated from consensus state.
- Current Bitcoin Core 31.0 cluster mempool behavior was distinguished from older ancestor/descendant-limit descriptions.
- RBF, CPFP, package acceptance, eviction, expiry, and reorg handling were described as policy or local-node behavior where appropriate.
- Bitcoin Core implementation references were checked against current Doxygen and release documentation.

### Evidence Review

Passed.

- RBF claims cite BIP125 and Bitcoin Core RBF source.
- Current cluster mempool claims cite Bitcoin Core 31.0 release notes.
- `CTxMemPool`, `CTxMemPoolEntry`, `AcceptToMemoryPool`, and RPC claims cite primary implementation or official documentation.
- Analytical claims about confirmation probability and global propagation are labeled as interpretation or caveat.

### Editorial Review

Passed.

- Markdown headings follow the project deep-dive structure.
- Metadata is complete.
- Tables and code fences are closed.
- Terminology is consistent: mempool, policy, cluster, chunk, RBF, CPFP, package, eviction, expiry.

### Adversarial Review

Passed.

- The document does not claim a single global mempool exists.
- It does not treat mempool acceptance as confirmation.
- It does not treat RBF as fraud.
- It does not treat consensus-valid as automatically relayable.
- It does not use outdated ancestor/descendant limits as current Bitcoin Core policy.

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
