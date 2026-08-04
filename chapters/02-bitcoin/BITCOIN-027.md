---
knowledge_id: BITCOIN-027
title: Fee Market
subtitle: Blockspace Scarcity, Feerate Competition, Mempool Policy, Miner Selection, and the Boundary Between Consensus and Market Outcomes
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 120 min
estimated_study: 350 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Economics
  - Fees
  - Mempool
parent:
  - Bitcoin Economics
prerequisites:
  - BITCOIN-017
  - BITCOIN-018
  - BITCOIN-020
  - BITCOIN-025
  - BITCOIN-026
related_topics:
  - Feerate
  - Blockspace
  - Mempool Policy
  - Block Assembly
  - Fee Estimation
  - Security Budget
  - CPFP
  - RBF
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-TRANSACTIONS-001
  - REF-BTC-CORE-FEERATE-001
  - REF-BTC-CORE-POLICY-001
  - REF-BTC-CORE-TXMEMPOOL-001
  - REF-BTC-CORE-BLOCKASSEMBLER-001
  - REF-BTC-CORE-FEE-ESTIMATOR-001
  - REF-BTC-CORE-31-RELEASE-001
tags:
  - bitcoin
  - economics
  - fees
  - fee-market
  - mempool
  - blockspace
  - feerate
  - mining
---

# Fee Market
> Bitcoin Economics  
> Research Unit: BITCOIN-027

---

## Research Brief

```yaml
knowledge_id: BITCOIN-027
title: Fee Market
research_question: >
  How does Bitcoin's fee market emerge from scarce blockspace, transaction
  demand, mempool policy, miner transaction selection, and feerate competition,
  and how should analysts separate consensus-valid fee payment from local relay
  policy, estimation heuristics, and broader security-budget interpretation?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-017
  - BITCOIN-018
  - BITCOIN-020
  - BITCOIN-025
  - BITCOIN-026
parent: Bitcoin Economics
previous: BITCOIN-026
next: BITCOIN-028
related_topics:
  - Block Assembly
  - Feerate Estimation
  - Mempool Eviction
  - CPFP
  - RBF
  - Security Budget
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
  - Non-Bitcoin fee markets
  - Price prediction using fee metrics
  - Full wallet UX guidance
  - Exhaustive package-relay policy history
  - Detailed miner treasury strategy analysis
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Define Bitcoin's fee market as competition for scarce blockspace rather than a fixed protocol price list.
- Distinguish fee amount from feerate.
- Explain why miner selection is usually driven by expected fee density under block constraints.
- Distinguish consensus-valid fees from local mempool and relay policy.
- Explain why CPFP and RBF matter for fee-market behavior.
- Explain why fee estimation is probabilistic and node-local.
- Explain how declining subsidy increases the analytical importance of fees without making fee sufficiency a protocol guarantee.

---

## 2. Key Questions

1. What is the Bitcoin fee market?
2. Why do users compete on feerate rather than just absolute fee amount?
3. What creates blockspace scarcity?
4. How do mempool policy and miner incentives interact?
5. Why are local mempool observations not the global fee market?
6. What roles do CPFP and RBF play in fee discovery and repricing?
7. How does fee estimation work in practice?
8. Why can high-fee transactions still wait?
9. What does a high fee share imply, and what does it not imply?

---

## 3. Executive Summary

Bitcoin's fee market is the process through which users compete for scarce blockspace by offering transaction fees, typically evaluated in feerate terms relative to transaction size or package size. The protocol does not publish a global required fee schedule. Instead, fees emerge from demand for inclusion, miner selection incentives, relay policy, and block-capacity constraints.[^ref-btc-wp][^ref-btc-core-feerate][^ref-btc-core-policy]

At the transaction level, a fee is simply the difference between total input value and total output value. At the market level, the more useful variable is usually feerate, because miners fill constrained blocks and therefore care about fee earned per unit of scarce blockspace rather than absolute fee alone.[^ref-btc-dev-transactions][^ref-btc-core-blockassembler]

Modern Bitcoin Core behavior sharpens this market structure:

- `CFeeRate` represents fee rates in policy code.[^ref-btc-core-feerate]
- mempool and relay policy impose local acceptance and eviction thresholds.[^ref-btc-core-policy][^ref-btc-core-txmempool]
- block assembly selects transactions based on expected mining feerate under block constraints.[^ref-btc-core-blockassembler]
- fee estimation infers likely inclusion thresholds from recent confirmation outcomes rather than reading any protocol-level oracle.[^ref-btc-core-fee-estimator]
- Bitcoin Core 31.0's cluster mempool redesign makes ordering, relay, eviction, and replacement more package-aware through chunks and feerate-diagram reasoning.[^ref-btc-core-31-release]

For analysts, the fee market must be treated as a partially observed, policy-mediated auction for blockspace, not as a single global number visible from one node.

---

## 4. Protocol Structure

### Fee as Transaction Arithmetic

A transaction fee is:

```text
fee = sum(inputs) - sum(outputs)
```

The fee is implicit. There is no dedicated fee field in the transaction format.[^ref-btc-dev-transactions]

### Fee Market as Blockspace Competition

The fee market begins when too many candidate transactions want inclusion relative to available blockspace. The binding scarcity is not "number of transactions" in the abstract. It is block weight, related validation constraints, and miner block-construction choices.

### Feerate vs Absolute Fee

Miners generally optimize for fee density under space constraints, so:

| Metric | Why It Matters |
|---|---|
| Absolute fee | Total revenue from a specific transaction |
| Feerate | Revenue per unit of scarce blockspace |
| Package feerate | Relevant when parent/child transactions must be mined together |

This is why a smaller transaction with a lower absolute fee can outrank a larger transaction with a higher absolute fee if its feerate is better.

---

## 5. Market Mechanics

### Blockspace Scarcity

Every block has finite weight capacity. Miners constructing candidate blocks cannot include every mempool transaction, so they rank opportunities. The fee market is the resulting prioritization process under scarcity.[^ref-btc-core-policy][^ref-btc-core-blockassembler]

### User Bidding

Users effectively "bid" for inclusion by choosing transaction structure and fee level. The bid is imperfect because:

- they do not know future competing demand with certainty,
- mempools differ across nodes,
- confirmation targets vary,
- package topology matters,
- miner behavior is not perfectly uniform.

### Miner Selection

Miners care about gross expected revenue subject to block limits, finality constraints, and dependency ordering. In modern Core, block assembly processes chunks selected by expected mining feerate, and block creation enforces a minimum feerate threshold for transactions included by mining code.[^ref-btc-core-blockassembler][^ref-btc-core-policy]

### CPFP and RBF

The fee market is not only about initial bids.

- CPFP lets a high-fee child improve effective package economics for a low-fee parent.
- RBF lets a replacement transaction improve the economic offer or otherwise alter the mempool state subject to policy rules.

These mechanisms matter because they let fee bids be updated after first broadcast rather than fixed forever.

---

## 6. Technical Mechanics

### `CFeeRate`

Bitcoin Core represents fee rate with `CFeeRate`, defined as satoshis per virtual byte in policy code.[^ref-btc-core-feerate]

Conceptually:

```text
feerate = fee / vsize
```

where `vsize` is the transaction's virtual size.

### Block Minimum Mining Fee

Policy code defines `DEFAULT_BLOCK_MIN_TX_FEE`, the default for `-blockmintxfee`, which sets the minimum feerate for transactions included by mining code in locally constructed blocks.[^ref-btc-core-policy]

This is not a consensus rule. A block containing a lower-fee transaction can still be valid if all consensus rules are satisfied.

### Rolling and Local Mempool Floors

`CTxMemPool` maintains a rolling minimum fee concept and mempool sequence state as part of admission and eviction behavior.[^ref-btc-core-txmempool]

This means a node under mempool pressure may demand higher effective feerates for admission than a lightly loaded node, even though both nodes share the same consensus rules.

### Cluster Mempool and Chunks

Bitcoin Core 31.0 reimplemented the mempool with a cluster-based design. Release notes describe:

- cluster size limits replacing prior ancestor/descendant count logic,
- ordering based on expected mining feerate of chunks,
- usage of that ordering for block template creation, eviction, relay announcements, and replacement validation,
- and stricter RBF acceptance based on improving the mempool's feerate diagram.[^ref-btc-core-31-release]

This is a policy and implementation development, not a consensus change.

### Fee Estimation

`CBlockPolicyEstimator` estimates the feerate needed for confirmation within target block counts by tracking how transactions in feerate buckets perform over time.[^ref-btc-core-fee-estimator]

The estimator is empirical and probabilistic:

- it depends on recent history,
- it reflects one node's observations,
- and it can be invalidated by structural shifts in demand or policy.

---

## 7. Validation Boundaries

### Fees Are Mostly Not Consensus-Specified Prices

Consensus requires that a transaction not create money from nowhere and that block rewards not exceed subsidy plus fees. But consensus does not prescribe a universal market-clearing feerate.

### Relay Policy Is Not the Fee Market Itself

A node's local acceptance threshold is one filter on observed demand, not the whole market. Different nodes can:

- use different local settings,
- experience different mempool pressure,
- observe different peer transaction sets,
- and derive different fee estimates.

### Miner Policy Is Not Consensus

Mining code may skip low-fee transactions according to local policy or block-template economics, but other miners may choose differently. A low-fee transaction can be mined if a miner includes it in a valid block.

---

## 8. Security Assumptions and Failure Modes

### Fee Share vs Security

As subsidy declines, fee revenue becomes more important to total miner compensation. But a higher fee share alone does not prove the security budget is sufficient. Analysts need to consider:

- absolute fee revenue,
- BTC price,
- miner costs,
- hash-rate response,
- attack economics.

### Congestion and Underbidding

When blockspace demand spikes, underpriced transactions may remain pending for extended periods or be evicted from some mempools. This is not a consensus failure. It is market rationing under scarce capacity.

### Node-View Fragmentation

Because mempools are local and policy-mediated, fee-market data gathered from one node may miss:

- private transaction flow,
- node-specific eviction behavior,
- package relationships not visible yet,
- miner-specific direct submission channels.

### Estimation Failure

Fee estimation can perform badly during regime changes:

- sudden congestion,
- rapid demand collapse,
- large policy transitions,
- atypical miner inclusion patterns.

Forecasting from recent history is useful, but it is not an oracle.

---

## 9. Mathematical or Economic Model

### Basic Feerate Identity

For a transaction:

```text
feerate = fee / vsize
```

For a package:

```text
package_feerate = total_package_fee / total_package_vsize
```

This is why a low-fee parent plus a high-fee child can still be attractive together.

### Capacity-Constrained Selection

If a block has remaining effective capacity `C`, miners face a constrained optimization problem:

```text
maximize total fees
subject to block weight, sigops, finality, and dependency constraints
```

This is not a perfect auction, but it is economically close to a capacity-constrained selection problem.

### Fee Share

Let:

- `S` = subsidy,
- `F` = fees,
- `R` = total miner revenue.

Then:

```text
R = S + F
fee_share = F / (S + F)
```

As `S` declines across eras, equal absolute fees imply larger fee share. That does not necessarily mean greater real security if total revenue is still low in fiat or attack-cost terms.

---

## 10. Bitcoin Core Implementation

### `policy/feerate.h`

`CFeeRate` provides the fee-rate representation used across policy and wallet-related code.[^ref-btc-core-feerate]

### `policy/policy.h`

`policy.h` defines defaults relevant to block construction and standardness-related policy, including `DEFAULT_BLOCK_MIN_TX_FEE`.[^ref-btc-core-policy]

### `txmempool.h`

`CTxMemPool` maintains local pool state, rolling fee concepts, and builder interfaces used in block construction.[^ref-btc-core-txmempool]

### `node::BlockAssembler`

`BlockAssembler` constructs block templates and, in current Core, adds transactions based on chunk feerate subject to block limits and transaction-level checks.[^ref-btc-core-blockassembler]

### `CBlockPolicyEstimator`

The estimator tracks bucketed confirmation outcomes to provide fee-rate estimates for target confirmation horizons.[^ref-btc-core-fee-estimator]

### Bitcoin Core 31.0 Release Notes

The 31.0 release notes are relevant because they document the current cluster mempool design and its use in ordering, eviction, relay, and replacement validation.[^ref-btc-core-31-release]

---

## 11. On-Chain Implications

### Directly Measurable

Analysts can directly measure:

- fee paid per confirmed transaction,
- block fee totals,
- fee share of block reward,
- confirmation delays for confirmed transactions,
- block fullness and weight usage.

### Not Directly Observable from Chain Alone

Chain data alone usually cannot reveal:

- the full mempool state miners observed,
- the exact fee alternatives that lost the inclusion race,
- local eviction thresholds,
- direct miner transaction submission outside public relay,
- the reason a transaction waited.

### Why This Matters

A transaction confirmed with a certain fee does not prove that same fee would have worked at broadcast time, on another node, or for another transaction topology.

---

## 12. Institutional Thinking

Institutions should treat the Bitcoin fee market as a live operating environment, not just a historical statistic.

### Practical Implications

- Fee policy must be target-based and adaptive, not static.
- Mempool observations should come from multiple high-quality vantage points where possible.
- Settlement and treasury analytics should separate gross fee revenue from newly issued subsidy.
- Security-budget commentary should report both fee share and absolute fee revenue.
- Operational systems should account for CPFP, RBF, and reorg-aware confirmation tracking.

---

## 13. Common Misinterpretations

### "Bitcoin has a fixed required transaction fee"

False. Bitcoin has no universal protocol-mandated market fee schedule.

### "Highest absolute fee always wins"

False. Under blockspace constraints, feerate and package economics usually matter more.

### "My node's mempool is the market"

False. It is one local, policy-mediated view of a distributed market.

### "A low-fee transaction is invalid"

False. It may be valid but unattractive for relay or mining under current conditions.

### "High fee share proves secure long-run mining economics"

False. Security depends on total compensation relative to attack cost, not fee share alone.

---

## 14. Research Questions

1. How much cross-node variance exists in first-seen fee-market conditions during congestion?
2. How materially did cluster mempool change empirical block-template selection and eviction behavior?
3. Which fee-market metrics best predict confirmation delay across different demand regimes?
4. How should institutions combine fee share, absolute fee revenue, and hash-cost estimates in security analysis?
5. What proportion of economically relevant fee flow bypasses ordinary public relay paths?

---

## 15. Practical Exercises

### Exercise 1

Given several transactions with different fees and virtual sizes, rank them by feerate and explain why that ranking matters for inclusion.

### Exercise 2

Build a simple package-feerate example showing how CPFP can improve a low-fee parent's inclusion prospects.

### Exercise 3

Using recent blocks, compute:

- total fees per block,
- median confirmed feerate,
- fee share of block reward,
- and a rough congestion proxy from block fullness.

### Exercise 4

Compare a node's fee estimate for multiple confirmation targets with actual subsequent inclusion outcomes and note the forecast error.

---

## 16. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Fee arithmetic and feerate primitives | Directly specified | Developer docs and `CFeeRate` |
| Block-construction and policy defaults | Directly specified | `policy.h`, `BlockAssembler`, `txmempool.h` |
| Fee estimation behavior | Directly specified | `CBlockPolicyEstimator` |
| Cluster mempool ordering and replacement behavior | Directly specified | Bitcoin Core 31.0 release notes |
| Security-budget interpretation | Inference from sources | Economic analysis based on fee and subsidy structure |

---

## 17. Knowledge Graph

```text
Fee Market
├─ Inputs
│  ├─ transaction demand
│  ├─ blockspace scarcity
│  ├─ mempool policy
│  └─ miner incentives
├─ Pricing
│  ├─ fee
│  ├─ feerate
│  ├─ package feerate
│  └─ estimation
├─ Mechanisms
│  ├─ CPFP
│  ├─ RBF
│  ├─ eviction
│  └─ block assembly
├─ Implementation
│  ├─ CFeeRate
│  ├─ CTxMemPool
│  ├─ BlockAssembler
│  ├─ fee estimator
│  └─ cluster mempool
└─ Risks
   ├─ node-view bias
   ├─ estimation error
   ├─ congestion
   └─ security-budget overstatement
```

---

## 18. References

### Primary Sources

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," especially Section 6. https://bitcoin.org/bitcoin.pdf

[^ref-btc-dev-transactions]: Bitcoin Developer Reference, "Transactions," including raw transaction structure and implicit fee arithmetic. https://developer.bitcoin.org/reference/transactions.html

[^ref-btc-core-feerate]: Bitcoin Core Doxygen, `policy/feerate.h` and `CFeeRate`. https://doxygen.bitcoincore.org/class_c_fee_rate.html

[^ref-btc-core-policy]: Bitcoin Core Doxygen, `policy/policy.h`, including `DEFAULT_BLOCK_MIN_TX_FEE`. https://doxygen.bitcoincore.org/policy_8h_source.html

[^ref-btc-core-txmempool]: Bitcoin Core Doxygen, `txmempool.h`, including rolling fee state and block-builder chunk interfaces. https://doxygen.bitcoincore.org/txmempool_8h_source.html

[^ref-btc-core-blockassembler]: Bitcoin Core Doxygen, `node::BlockAssembler` and `miner.cpp`, including chunk-based block assembly. https://doxygen.bitcoincore.org/classnode_1_1_block_assembler.html

[^ref-btc-core-fee-estimator]: Bitcoin Core Doxygen, `CBlockPolicyEstimator`. https://doxygen.bitcoincore.org/class_c_block_policy_estimator.html

[^ref-btc-core-31-release]: Bitcoin Core 31.0 release notes, mempool and fee-estimation changes. https://bitcoincore.org/en/releases/31.0/

### Supporting Interpretation Notes

- Where this document discusses security sufficiency, partial observability, or market-level inference from node-local data, those statements are analytical interpretations built on documented policy, mempool, and block-construction behavior rather than direct protocol guarantees.

---

## 19. Cross References

### Previous

- BITCOIN-026 — Halving

### Next

- BITCOIN-028 — Security Budget

### Related

- BITCOIN-017 — Mempool
- BITCOIN-018 — Transaction Fees
- BITCOIN-020 — Mining
- BITCOIN-025 — Bitcoin Monetary Policy
- BITCOIN-026 — Halving
- BITCOIN-028 — Security Budget
- BITCOIN-029 — Bitcoin Game Theory

---

## Review Status

### Technical Review

Passed.

- Fee arithmetic, feerate competition, policy filters, miner selection, and estimation were separated.
- Consensus-valid fees were distinguished from relay and mining policy.
- Current Core block assembly and mempool behavior were described with chunk- and cluster-aware terminology.
- Security-budget implications were kept analytical rather than treated as protocol facts.

### Evidence Review

Passed.

- Developer references support fee arithmetic.
- Core Doxygen supports fee-rate, policy, mempool, assembler, and estimator descriptions.
- Bitcoin Core 31.0 release notes support the cluster mempool behavior description.
- Interpretation about observability and security is labeled as inference.

### Editorial Review

Passed.

- Structure follows the project deep-dive format.
- Metadata is complete.
- Terminology is consistent: fee, feerate, package feerate, blockspace, estimator, fee share.
- Tables and code fences are closed.

### Adversarial Review

Passed.

- The document does not claim a universal fixed Bitcoin fee.
- It does not equate one node's mempool with the entire market.
- It does not reduce miner choice to absolute fee amount alone.
- It does not confuse relay rejection with consensus invalidity.
- It does not overstate what fee share alone proves about security.

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
