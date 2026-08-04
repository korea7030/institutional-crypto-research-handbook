---
knowledge_id: BITCOIN-024
title: Chain Reorganization
subtitle: Active-Chain Replacement, Fork Points, Disconnect-and-Connect Mechanics, Confirmation Reversal, and Analytical Risk
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 115 min
estimated_study: 330 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Consensus
  - Chain Selection
  - Reorganization
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-014
  - BITCOIN-017
  - BITCOIN-021
  - BITCOIN-022
  - BITCOIN-023
  - POW-010
  - POW-011
  - POW-013
related_topics:
  - Fork Point
  - Active Chain
  - Chainwork
  - Stale Blocks
  - Mempool Reinsertion
  - UTXO Rollback
  - Double Spend Risk
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-BLOCKCHAIN-001
  - REF-BTC-DEV-OPERATING-MODES-001
  - REF-BTC-CORE-CHAIN-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-TXMEMPOOL-001
  - REF-BTC-CORE-VALIDATIONINTERFACE-001
  - REF-BTC-CORE-CONSENSUS-VALIDATION-001
tags:
  - bitcoin
  - internals
  - consensus
  - reorg
  - chain-selection
  - chainwork
  - mempool
  - utxo
---

# Chain Reorganization
> Bitcoin Internals  
> Research Unit: BITCOIN-024

---

## Research Brief

```yaml
knowledge_id: BITCOIN-024
title: Chain Reorganization
research_question: >
  What is a Bitcoin chain reorganization, how does a node replace one active
  branch with another valid higher-work branch, how are UTXO and mempool state
  updated during the transition, and what does a reorg prove or fail to prove
  to analysts and institutions?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-014
  - BITCOIN-017
  - BITCOIN-021
  - BITCOIN-022
  - BITCOIN-023
  - POW-010
  - POW-011
  - POW-013
parent: Bitcoin Internals
previous: BITCOIN-023
next: BITCOIN-025
related_topics:
  - Fork Resolution
  - UTXO State
  - Mempool Consistency
  - Confirmation Risk
  - Double-Spend Analysis
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
  - Full attacker game theory
  - Exhaustive review of historical Bitcoin reorg incidents
  - Exchange-specific operational policies
  - Altchain finality models
  - Wallet UI design for reorg handling
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Define a reorg as an active-chain replacement event rather than merely a fork.
- Distinguish fork creation from fork resolution and from reorganization.
- Explain how a node finds the fork point between the current tip and a better branch.
- Explain why reorg decisions are based on valid higher cumulative work, not just header count.
- Explain how disconnected blocks affect UTXO state.
- Explain how disconnected transactions may return to mempool if still valid and policy-acceptable.
- Explain why a reorg can reverse confirmations without proving malicious intent.
- Identify the main Bitcoin Core functions involved in reorg handling.

---

## 2. Key Questions

1. What exactly is a Bitcoin chain reorganization?
2. When does a fork become a reorg?
3. What is disconnected from the old branch and connected from the new branch?
4. What happens to UTXOs created or spent on the losing branch?
5. What happens to transactions from disconnected blocks?
6. Why can a reorg reverse confirmations?
7. Why does a higher-work header chain alone not guarantee an immediate reorg?
8. How does Bitcoin Core preserve mempool consistency through a reorg?
9. What can on-chain data show about a reorg, and what can it not show?
10. When should analysts suspect adversarial behavior rather than ordinary propagation races?

---

## 3. Executive Summary

A chain reorganization occurs when a node replaces part of its current active chain with another branch that is both valid and has greater cumulative proof of work. It is not the same thing as a fork itself. A fork is the existence of competing branches after a common ancestor; a reorg is the event of switching the active branch.[^ref-btc-wp][^ref-btc-dev-blockchain]

The whitepaper's "longest chain" description implies this replacement logic: if two blocks are found at once, nodes may temporarily disagree, but they eventually extend one branch and abandon the other. In modern Bitcoin terms, that abandonment becomes an explicit active-chain switch once a higher-work valid branch is selected.[^ref-btc-wp]

Reorgs have state consequences beyond headers. A node must roll back chainstate changes from disconnected blocks and then apply chainstate changes from the replacement branch. That means:

- outputs created only on the losing branch can disappear from the active UTXO set,
- previously spent outputs can become spendable again,
- transactions from disconnected blocks may become unconfirmed and possibly re-enter mempool if still valid under the new active chain.[^ref-btc-core-validation][^ref-btc-core-txmempool]

For analysts, the key point is that a reorg is observable chain behavior, not automatically proof of intent. A one-block reorg can arise naturally from propagation races. A deeper or strategically timed reorg can indicate stronger adversarial hypotheses, but intent requires additional evidence beyond the branch replacement itself.[^ref-btc-dev-blockchain][^ref-btc-dev-operating-modes]

---

## 4. Protocol Structure

### Fork vs Reorg

Start with a common ancestor `H`:

```text
H
├─ A1 ─ A2
└─ B1 ─ B2 ─ B3
```

If a node is currently on branch `A` and later learns that branch `B` is the best valid higher-work branch, the node may switch from `A2` to `B3`. That switch is the reorganization.

### Minimal Definition

A reorg has three ingredients:

1. A fork point shared by the old and new branches.
2. A current active branch to be disconnected after that fork point.
3. A better valid branch to be connected after that fork point.

### Reorg Depth

Reorg depth is commonly described in blocks, meaning how many blocks are removed from the previously active branch. Analysts should also care about work difference, because equal depth across different difficulty regimes does not imply equal replacement cost.

---

## 5. High-Level Mechanics

### Fork Point Discovery

Bitcoin Core represents chain history through `CBlockIndex` parent links. To switch branches, the node first determines the last common ancestor between the current tip and the better candidate tip.[^ref-btc-core-chain]

### Disconnect Old Branch

Every block after the fork point on the losing branch must be disconnected from chainstate. This reverses the active effects of those blocks:

- spent outputs may be restored,
- outputs created only by the disconnected blocks may be removed,
- confirmations attached to transactions in those blocks disappear from the active chain view.

### Connect New Branch

The node then validates and connects blocks on the replacement branch from the fork point forward. Transactions in those blocks become confirmed in the new active history.

### Result

After the sequence completes, the node has one active chain again. The losing branch may still be known in block index data, but it is no longer the active history used for confirmations and spendability.

---

## 6. UTXO and Transaction Effects

### UTXO Rollback

The UTXO set always reflects the active chain, not every known block. A reorg therefore changes the spendable set.

If a disconnected block had:

- created output `X`, then `X` may disappear from the active UTXO set;
- spent output `Y`, then `Y` may become unspent again if no replacement-branch transaction also spends it.

This is why chainstate must be updated, not just headers.

### Confirmation Reversal

A transaction confirmed on the losing branch becomes unconfirmed from the active-chain perspective unless it is also confirmed on the winning branch.

This is the protocol basis for settlement reversal risk in reorgs.

### Conflicts and Double Spends

If the winning branch contains a conflicting transaction spending the same inputs differently, the losing-branch transaction does not simply become "pending again." It may be permanently excluded from the active history because the inputs are now spent by another transaction in the winning chain.

### Coinbase Special Case

Coinbase outputs are especially sensitive to reorgs because the entire block reward disappears from the active chain if the coinbase's block is disconnected. Coinbase maturity is one reason Bitcoin avoids immediate spendability of freshly mined rewards.

---

## 7. Mempool Effects

### Disconnected Transactions Are Not Automatically Lost

Transactions from disconnected blocks may be reconsidered for mempool admission if they remain valid and policy-compliant under the new active chain. Bitcoin Core explicitly includes mechanisms to update mempool consistency after a new block is connected or after a reorg.[^ref-btc-core-txmempool]

### Reinsertion Conditions

A disconnected transaction may return to mempool only if:

- its inputs are available under the new chainstate,
- it does not conflict with winning-branch confirmed transactions,
- it still passes local mempool policy,
- and related dependency conditions remain satisfied.

### Reorg Consistency

Bitcoin Core exposes validation and mempool callbacks such as `BlockConnected`, `BlockDisconnected`, and mempool-removal notifications, reflecting that reorg handling is not merely internal bookkeeping but a state transition with downstream consumers.[^ref-btc-core-validationinterface]

---

## 8. Technical Mechanics

### Most-Work Valid Chain

A node does not reorganize just because it sees an alternative header path. The candidate branch must survive validation and become the best valid branch by cumulative work. This point matters because header awareness, block validity, and active-chain replacement are separate steps.[^ref-btc-core-validation][^ref-btc-core-consensus-validation]

### Core Reorg Skeleton

At a high level:

```text
find best valid candidate tip
find last common ancestor with active tip
disconnect active blocks after fork point
connect candidate-branch blocks after fork point
update active tip
repair mempool consistency
emit validation callbacks
```

This sequence is conceptually simple but operationally sensitive because chainstate, mempool, wallet logic, and observers all depend on consistent ordering.

### Atomicity at the External Interface

Bitcoin Core tests explicitly care that callers do not observe arbitrary inconsistent intermediate mempool states during reorg handling. The implementation goal is not "no internal intermediate steps exist," but rather that externally visible state transitions remain coherent.[^ref-btc-core-validation][^ref-btc-core-txmempool]

### Reorg Depth vs Work Depth

Two reorgs of depth 2 are not necessarily similar. If difficulty, timestamp spacing, or branch work differ, the replacement effort and security implications differ. Work-aware reporting is more robust than depth-only reporting.

---

## 9. Validation Boundaries

### Header Competition Is Not Yet a Reorg

A node may know of a competing header chain without having all blocks or without having validated them enough to activate the branch. So:

- competing headers: possible candidate,
- valid full blocks: stronger candidate,
- active-chain switch: actual reorg.

### Reorg Does Not Equal Attack

A reorg is protocol behavior. Small reorgs can arise from ordinary propagation races. Calling every reorg an attack collapses necessary distinctions between:

- accidental near-simultaneous mining,
- natural topology delay,
- deliberate withholding,
- deliberate double-spend attempts.

### Confirmation Count Is Not Finality

Confirmations increase replacement cost but do not create absolute irreversibility. A reorg demonstrates this directly: previously confirmed transactions can disappear from the active history if a higher-work valid branch replaces them.

---

## 10. Security Assumptions and Failure Modes

### Natural Reorgs

One-block or other shallow reorgs can occur without adversarial coordination, especially when block discoveries are close in time and propagation is imperfect.

### Adversarial Reorgs

Deeper or strategically timed reorgs may support stronger concern about:

- double-spend attempts,
- selfish-mining style withholding,
- majority-hashrate attacks,
- eclipse-assisted victim-specific misperception.

But the public reorg alone usually cannot prove which motive applied.

### SPV Exposure

Operating-mode documentation notes that SPV clients depend on headers and cumulative work rather than full validation of all blocks.[^ref-btc-dev-operating-modes]

That means SPV clients are especially reliant on confirmation depth and honest majority assumptions when reasoning about reorg risk.

### Infrastructure Risk

Reorgs stress:

- exchange crediting logic,
- deposit finality assumptions,
- wallet transaction-state tracking,
- accounting systems that expect monotonic confirmation growth,
- analytics pipelines that do not model active-chain changes explicitly.

---

## 11. Mathematical or Economic Model

### Replacement-Cost Intuition

If a transaction sits `k` blocks deep, replacing it requires a competing branch that overtakes the public branch in cumulative work from before that transaction's block. Depth is a coarse proxy for this cost; cumulative work is a better one.

### Simple Confirmation Model

Let:

- `W_pub` = cumulative work added on the public branch after a target block,
- `W_alt` = cumulative work added on the competing branch after the same fork point.

A reorg to the alternative branch becomes possible when:

```text
W_alt > W_pub
```

subject to both branches being valid under the node's consensus rules.

### Economic Meaning

Reorgs matter economically because they can:

- reverse settlement assumptions,
- restore spendability of prior inputs,
- change fee attribution to miners,
- alter realized block rewards on the losing branch,
- force exchanges and custodians to delay crediting.

---

## 12. Bitcoin Core Implementation

### `chain.h`

`chain.h` provides the block-index structure and `LastCommonAncestor`, which is the structural primitive for identifying where a reorg begins.[^ref-btc-core-chain]

### `validation`

Bitcoin Core's validation layer includes `ActivateBestChain`, `ActivateBestChainStep`, `DisconnectBlock`, `ConnectBlock`, and `MaybeUpdateMempoolForReorg`. These names capture the core execution phases of a reorg: choose best valid branch, disconnect old state, connect new state, then reconcile mempool.[^ref-btc-core-validation]

### `txmempool`

`CTxMemPool` documentation explicitly notes support for updating the mempool to remain consistent with the best chain after a new block is connected or after a reorg, and it maintains structures useful for recovering transactions affected by chain changes.[^ref-btc-core-txmempool]

### `validationinterface`

`CValidationInterface` exposes callbacks such as `UpdatedBlockTip`, `BlockConnected`, `BlockDisconnected`, and mempool-related notifications. These make reorg state transitions visible to dependent subsystems such as wallets, indexes, or external integrations.[^ref-btc-core-validationinterface]

---

## 13. Consensus, Policy, and Presentation

### Consensus Layer

Consensus determines whether the replacement branch is valid and whether it has greater cumulative work.

### Policy Layer

Policy determines what disconnected transactions may re-enter mempool or be relayed again. A transaction can be consensus-valid yet fail re-entry because of local mempool policy or conflicts.

### Presentation Layer

Presentation is what users and systems observe:

- a transaction may show confirmed, then unconfirmed;
- a deposit may appear credited, then reversed;
- a block explorer may relabel a block as stale or orphaned after branch replacement.

The presentation change is downstream of the active-chain switch.

---

## 14. On-Chain Implications

### What Public Data Can Show

Once both branches are known publicly, analysts can reconstruct:

- fork point,
- losing-branch depth,
- winning-branch depth,
- conflicting transactions visible on-chain,
- replaced confirmations.

### What Public Data Usually Cannot Show

Public chain data usually cannot prove:

- whether a private fork existed before release,
- the attacker's intent,
- whether the event was accidental until more context is considered,
- the exact set of nodes that had already switched views at each moment.

### Reorg Evidence Requirements

Strong reorg analysis should report at least:

- old tip and new tip,
- fork point height,
- reorg depth in blocks,
- work difference if available,
- whether conflicting spends appeared,
- whether the event was public throughout or only visible after release.

---

## 15. Institutional Thinking

Institutions should treat reorg handling as a state-management problem, not just a security headline.

### Practical Implications

- Confirmation thresholds should reflect reorg tolerance, not superstition.
- Accounting systems need reversible settlement states.
- Mempool and chain observers should record active-chain transitions, not just first-seen events.
- Incident response should separate "public reorg observed" from "malicious intent established."

### Better Reporting

For high-quality internal reporting, use both:

- reorg depth in blocks,
- replacement work differential when reconstructible.

Depth alone is too weak for serious risk comparison.

---

## 16. Common Misinterpretations

### "A reorg only changes headers"

False. It changes active chainstate, which means UTXO state, confirmations, and potentially mempool contents.

### "If a tx was once confirmed, it remains final"

False. Confirmation is evidence of growing replacement cost, not absolute permanence.

### "Every reorg proves an attack"

False. Small reorgs can occur naturally from propagation races.

### "If the alternative branch has more headers, reorg must happen"

False. The branch must be valid and win by cumulative work, not merely by visible header count.

### "Disconnected transactions always go back to mempool"

False. They return only if still valid and policy-acceptable under the new active chain.

---

## 17. Research Questions

1. How often do shallow Bitcoin mainnet reorgs occur under current propagation conditions?
2. How much explanatory power does work differential add over raw depth in reorg risk dashboards?
3. How reliably can public observers distinguish accidental from strategic reorgs?
4. Which mempool datasets best preserve reorg-aware transaction histories?
5. How should institutional finality models incorporate topology-specific observation lag?

---

## 18. Practical Exercises

### Exercise 1

Reconstruct a public reorg from chain data and identify the fork point, losing branch, winning branch, and depth.

### Exercise 2

For a transaction removed by a reorg, classify whether it:

- re-entered mempool,
- was replaced by a conflicting spend,
- or disappeared because it became invalid under the new chain.

### Exercise 3

Using cumulative work data where available, compare two reorg events that had the same depth but different replacement-cost profiles.

### Exercise 4

Trace the high-level Core flow from candidate branch discovery to `ActivateBestChain` and post-reorg mempool reconciliation.

---

## 19. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Reorg as branch replacement after fork competition | Directly specified | Whitepaper and Developer Guide |
| Fork-point and best-chain mechanics | Directly specified | Bitcoin Core `chain` and `validation` references |
| Mempool consistency and disconnected-tx handling | Directly specified | `txmempool` and validation references |
| Analyst interpretation of intent | Inference from sources | Derived from observable protocol behavior, not explicit proof |
| Work-aware risk framing | Analytical model | Derived from chainwork and confirmation semantics |

---

## 20. Knowledge Graph

```text
Chain Reorganization
├─ Preconditions
│  ├─ fork point
│  ├─ competing branch
│  ├─ validity
│  └─ greater cumulative work
├─ Mechanics
│  ├─ last common ancestor
│  ├─ disconnect old blocks
│  ├─ connect new blocks
│  ├─ update tip
│  └─ reconcile mempool
├─ State Effects
│  ├─ UTXO rollback
│  ├─ confirmation reversal
│  ├─ stale blocks
│  └─ tx reinsertion or conflict
├─ Observation
│  ├─ public reorg
│  ├─ private-fork uncertainty
│  └─ on-chain evidence limits
└─ Risk
   ├─ settlement reversal
   ├─ double-spend exposure
   ├─ accounting inconsistency
   └─ SPV sensitivity
```

---

## 21. References

### Primary Sources

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Sections 5 and 11. https://bitcoin.org/bitcoin.pdf

[^ref-btc-dev-blockchain]: Bitcoin Developer Guide, "Block Chain," including most-difficult-chain and fork behavior. https://developer.bitcoin.org/devguide/block_chain.html

[^ref-btc-dev-operating-modes]: Bitcoin Developer Guide, "Operating Modes," including full-node and SPV chain-validation discussion. https://developer.bitcoin.org/devguide/operating_modes.html

[^ref-btc-core-chain]: Bitcoin Core Doxygen, `chain.h`, including `CBlockIndex`, block proof helpers, and `LastCommonAncestor`. https://doxygen.bitcoincore.org/chain_8h_source.html

[^ref-btc-core-validation]: Bitcoin Core Doxygen, `validation.h`, including `ActivateBestChain`, `ActivateBestChainStep`, `DisconnectBlock`, `ConnectBlock`, and `MaybeUpdateMempoolForReorg`. https://doxygen.bitcoincore.org/validation_8h_source.html

[^ref-btc-core-txmempool]: Bitcoin Core Doxygen, `CTxMemPool` / `txmempool.h`, including mempool consistency after a new block or reorg. https://doxygen.bitcoincore.org/class_c_tx_mem_pool.html

[^ref-btc-core-validationinterface]: Bitcoin Core Doxygen, `validationinterface.h` and `CValidationInterface`, including `UpdatedBlockTip`, `BlockConnected`, and `BlockDisconnected`. https://doxygen.bitcoincore.org/validationinterface_8h_source.html

[^ref-btc-core-consensus-validation]: Bitcoin Core Doxygen, `consensus/validation.h`, including validation-state distinctions for transactions and blocks. https://doxygen.bitcoincore.org/consensus_2validation_8h.html

### Supporting Interpretation Notes

- Where this document discusses intent, institutional reporting standards, or work-aware risk framing, those statements are inferences from Bitcoin's chain-selection and validation architecture rather than explicit protocol claims.

---

## 22. Cross References

### Previous

- BITCOIN-023 — Forks

### Next

- BITCOIN-025 — Bitcoin Monetary Policy

### Related

- BITCOIN-007 — Whitepaper Section 6 — Incentive
- BITCOIN-014 — UTXO Model
- BITCOIN-017 — Mempool
- BITCOIN-021 — Blocks and Block Headers
- BITCOIN-022 — Nodes and Network Propagation
- BITCOIN-023 — Forks
- POW-010 — Longest Chain Rule and Fork Resolution
- POW-011 — Cumulative Chainwork
- POW-013 — Bitcoin Core Proof-of-Work Validation
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- Fork, branch replacement, stale blocks, and confirmation reversal were separated.
- UTXO rollback, disconnected transaction handling, and mempool reconciliation were included.
- Active-chain replacement was described in valid-higher-work terms rather than raw block count terms.
- Core references were limited to chain, validation, mempool, and validation-interface surfaces relevant to reorgs.

### Evidence Review

Passed.

- Whitepaper and Developer Guide support the branch-replacement model.
- Core implementation claims cite `chain.h`, `validation.h`, `txmempool.h`, and `validationinterface.h`.
- Mempool and chainstate side effects are tied to explicit Core documentation.
- Interpretation about attacker intent is labeled as inference.

### Editorial Review

Passed.

- Document structure follows the project deep-dive format.
- Metadata is complete.
- Terms are consistent: fork point, active chain, reorg depth, cumulative work, disconnected block, replacement branch.
- Tables and code fences are closed.

### Adversarial Review

Passed.

- The document does not claim every reorg is malicious.
- It does not reduce reorg decisions to header count alone.
- It does not claim disconnected transactions always return to mempool.
- It does not imply headers-only awareness guarantees branch activation.
- It does not confuse public observability with proof of private-fork absence.

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
