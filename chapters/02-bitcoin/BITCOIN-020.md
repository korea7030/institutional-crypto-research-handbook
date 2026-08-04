---
knowledge_id: BITCOIN-020
title: Mining
subtitle: Block Template Construction, Proof-of-Work Search, Coinbase Rewards, Pools, Shares, Propagation, and Miner Incentives
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 110 min
estimated_study: 330 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Mining
  - Proof of Work
  - Block Construction
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-017
  - BITCOIN-018
  - POW-009
  - POW-012
  - POW-013
related_topics:
  - Block Templates
  - Coinbase Transaction
  - Proof-of-Work Search
  - Difficulty Target
  - Mining Pools
  - Shares
  - Transaction Selection
  - Block Propagation
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-MINING-001
  - REF-BIP-0022
  - REF-BIP-0023
  - REF-BTC-CORE-MINER-001
  - REF-BTC-CORE-MINING-IFACE-001
  - REF-BTC-CORE-MINING-RPC-001
  - REF-BTC-CORE-31-RELEASE-001
  - REF-BTC-CORE-POW-001
  - REF-BTC-CORE-VALIDATION-001
tags:
  - bitcoin
  - internals
  - mining
  - block-template
  - coinbase
  - proof-of-work
  - difficulty
  - mining-pools
  - getblocktemplate
---

# Mining
> Bitcoin Internals  
> Research Unit: BITCOIN-020

---

## Research Brief

```yaml
knowledge_id: BITCOIN-020
title: Mining
research_question: >
  How do Bitcoin miners construct candidate blocks, search for valid
  proof-of-work, coordinate through solo or pooled mining workflows, collect
  subsidy and fees, and interact with node policy, mempool state, and
  consensus validation?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-017
  - BITCOIN-018
  - POW-009
  - POW-012
  - POW-013
parent: Bitcoin Internals
previous: BITCOIN-019
next: BITCOIN-021
related_topics:
  - Mempool
  - Transaction Fees
  - Coinbase Transaction
  - Difficulty Adjustment
  - Proof-of-Work Validation
  - Mining Pools
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
  - ASIC electrical engineering
  - Mining facility power procurement
  - Pool payout formula accounting
  - Real-time miner identity attribution
  - Full Stratum v2 specification
  - Tax or regulatory treatment of mining revenue
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Explain what miners do in Bitcoin.
- Distinguish block construction from proof-of-work search.
- Explain how transaction selection, coinbase construction, Merkle root commitment, and header hashing fit together.
- Explain why mining search uses nonce, extra nonce, time, version, and transaction-set changes.
- Distinguish solo mining, pooled mining, pool operators, hashers, and shares.
- Explain why pool shares are not Bitcoin blocks.
- Explain how miners collect subsidy and transaction fees.
- Explain why mining policy can differ from node relay policy.
- Identify Bitcoin Core mining source areas and RPCs.
- Avoid treating coinbase labels, pool tags, or hashrate estimates as perfect identity evidence.

---

## 2. Key Questions

1. What is Bitcoin mining?
2. What is a candidate block?
3. What is a block template?
4. How does a miner choose transactions?
5. How does the coinbase transaction pay the miner?
6. Why does changing coinbase data change the Merkle root?
7. Why is the 32-bit header nonce not enough for modern mining?
8. What is the difference between network target and pool share target?
9. What is solo mining?
10. What is pooled mining?
11. What does Bitcoin Core's mining code construct?
12. What does mining reveal on-chain and what remains uncertain?

---

## 3. Executive Summary

Bitcoin mining is the process of constructing candidate blocks and searching for a block header hash below the current proof-of-work target. A valid block must satisfy both proof-of-work and full block validity rules. A header hash below target is necessary, but not sufficient, because the block's transactions, coinbase value, Merkle commitments, timestamps, difficulty bits, and contextual rules must also be valid.[^ref-btc-core-pow][^ref-btc-core-validation]

The mining workflow is:

```text
observe chain tip
select transactions
construct coinbase transaction
compute Merkle root
build block header
iterate header-affecting fields
find hash below target
submit and propagate full block
```

Bitcoin Developer documentation describes solo mining and pooled mining. Solo miners keep all rewards but face high variance. Pooled miners combine hashpower and receive smaller, more frequent payouts correlated with contributed work.[^ref-btc-dev-mining]

Bitcoin Core does not mine with ASICs itself in normal production deployments. Its mining-relevant code constructs block templates, validates submissions, and exposes mining interfaces. `BlockAssembler` is documented as generating a new block without valid proof-of-work, and `CreateNewBlock` constructs a new block template.[^ref-btc-core-miner]

Mining is economically driven by expected revenue:

```text
expected miner revenue = block subsidy + transaction fees - operating costs
```

Miners generally prefer high fee-rate transaction sets, but exact behavior can vary due to template policy, private orderflow, pool rules, propagation risk, and operational constraints.

For analysts, mining data supports observations about blocks, coinbase outputs, transaction selection, fee revenue, and rough pool attribution. It does not prove the real-world identity of every hasher or the full commercial arrangement behind a pool.

---

## 4. Protocol Structure

### Mining as Block Production

Mining adds new blocks to the block chain, making transaction history expensive to modify. The whitepaper frames proof-of-work as a mechanism where the longest chain represents the greatest proof-of-work investment, and incentives are provided through newly issued coins and transaction fees.[^ref-btc-wp]

The miner's output is a block:

```text
block
  header
  transactions
```

The header commits to:

- version;
- previous block hash;
- transaction Merkle root;
- time;
- compact difficulty target `nBits`;
- nonce.

### Candidate Block

A candidate block is a proposed block before proof-of-work is found.

It includes:

- a previous-block reference;
- a coinbase transaction;
- selected non-coinbase transactions;
- a valid transaction Merkle root;
- header fields suitable for hashing;
- a coinbase value within allowed subsidy plus fees.

### Block Template

A block template is data used by mining software to construct candidate blocks. BIP22 defines `getblocktemplate` as a JSON-RPC method for smart miners and proxies where the entire block structure is provided and can optionally be customized and assembled by the miner.[^ref-bip-0022]

Bitcoin Core RPC documentation says `getblocktemplate` returns data needed to construct a block to work on and references BIPs 22, 23, 9, and 145.[^ref-btc-core-mining-rpc]

### Coinbase Transaction

The coinbase transaction is the first transaction in a block. It creates the miner's spendable reward after maturity and claims:

```text
subsidy + included transaction fees
```

The coinbase transaction also affects mining search because changing coinbase data changes the coinbase transaction hash, which changes the Merkle root, which changes the block header.

### Proof-of-Work Search

Mining search tests candidate headers:

```text
double_sha256(block_header) <= target
```

The search is probabilistic. Each hash attempt is an independent trial over the header search space, assuming SHA-256 behaves as expected.

---

## 5. Technical Mechanics

### Template Construction

Template construction generally includes:

1. Read the active chain tip.
2. Set next block height.
3. Choose a block version.
4. Determine required difficulty bits.
5. Select transactions from mempool or other sources.
6. Build coinbase transaction.
7. Compute Merkle root.
8. Set time and nonce fields.
9. Return work to mining software or ASIC controller.

Bitcoin Core's `BlockAssembler::CreateNewBlock` begins by resetting block counters, creating a `CBlockTemplate`, adding a dummy coinbase as the first transaction, selecting transactions, and later updating the block with coinbase and Merkle commitment data.[^ref-btc-core-miner]

### Transaction Selection

Miners choose transactions under mining policy. The common incentive is to maximize fee revenue per scarce block resource, subject to:

- consensus validity;
- block weight;
- signature operation limits;
- transaction dependencies;
- package or cluster fee-rate ordering;
- pool policy;
- private orderflow;
- template freshness.

Bitcoin Core 31.0 cluster mempool orders transactions by expected mining feerate using chunks for block template construction, eviction, relay announcements, and replacement validation.[^ref-btc-core-31-release]

### Header Search Fields

The block header has a 32-bit nonce. Modern ASICs can exhaust this space quickly, so mining systems vary other header-affecting data:

| Field or input | Effect |
|---|---|
| Header nonce | Direct header field |
| Coinbase extra nonce | Changes coinbase txid, Merkle root, and header |
| Transaction set/order | Changes Merkle root |
| Timestamp | Changes header time |
| Version bits | May change header version within permitted rules |

Bitcoin Developer documentation explains that if all nonce values fail, mining software can update the block header by changing extra nonce data in the coinbase field, producing a new Merkle root.[^ref-btc-dev-mining]

### Block Submission

After a valid hash is found, mining software submits the full block. BIP22 defines `submitblock` for submitting potential blocks or shares, returning `null` when accepted or a rejection reason otherwise.[^ref-bip-0022]

Bitcoin Core exposes mining RPCs including `getblocktemplate`, `submitblock`, `submitheader`, `getmininginfo`, `getnetworkhashps`, and transaction prioritization RPCs.[^ref-btc-core-mining-rpc]

### Block Propagation

A found block has value only if it propagates and is accepted before competing blocks overtake it. Slow propagation increases stale-block risk.

Propagation risk affects:

- miner revenue;
- pool connectivity;
- transaction selection;
- compact block and relay infrastructure;
- incentive to mine on the latest tip quickly.

---

## 6. Solo Mining and Pooled Mining

### Solo Mining

Solo mining means the miner attempts to find blocks independently and keeps the full subsidy and fees for found blocks. It has high variance: a small miner may wait far longer than expected before finding a block.

Bitcoin Developer documentation describes solo miners using `bitcoind` to receive transactions, polling `getblocktemplate`, constructing a block, sending the 80-byte header to ASIC hardware, and submitting a full block if successful.[^ref-btc-dev-mining]

### Pooled Mining

Pooled mining combines hashpower. The pool operator coordinates templates and payout accounting; participants submit partial proofs called shares.

Benefits:

- lower payout variance for individual hashers;
- centralized transaction selection and template management;
- easier operations for individual hash owners.

Risks:

- pool operator centralization;
- template censorship by pool operator;
- hasher dependence on pool infrastructure;
- payout accounting disputes;
- pool-level regulatory or operational chokepoints.

### Shares

A share is proof that a miner performed work meeting a pool-defined target. The pool target is easier than the network target.

```text
share target > network target
share hash may prove work to pool
share hash usually does not create a valid Bitcoin block
```

Only a header hash below the network target can create a valid Bitcoin block.

### Getblocktemplate and Pooled Mining

BIP23 extends BIP22 for pooled mining, adding optional support such as pool extensions, block proposals, mutations, and submission abbreviations.[^ref-bip-0023]

Developer documentation notes that Stratum is widely used as an alternative to `getblocktemplate`, giving miners minimal data needed to construct block headers and reducing bandwidth for pool-miner coordination.[^ref-btc-dev-mining]

Stratum details are outside this unit's scope; the core point is that pooled mining can separate hash ownership from transaction-template control.

---

## 7. Mathematical or Economic Model

### Probability of Finding a Block

If a miner controls fraction `h` of network hash rate, then over a short period with stable network conditions:

```text
expected fraction of blocks found ~= h
```

This is expectation, not guarantee. Actual block discovery follows high-variance probabilistic search.

### Expected Time

If total network expected block interval is 10 minutes and miner share is `h`:

```text
expected solo block interval ~= 10 minutes / h
```

Example:

```text
h = 0.001 = 0.1%
expected interval ~= 10 / 0.001 = 10,000 minutes
                 ~= 6.94 days
```

The distribution remains highly variable. Expected time is not a schedule.

### Miner Revenue

For a found block:

```text
gross_revenue = subsidy + transaction_fees
net_revenue = gross_revenue - operating_costs
```

Operating costs include electricity, hardware depreciation, hosting, firmware/software operations, cooling, financing, and pool fees.

### Pool Variance Reduction

Pools reduce individual payout variance by distributing revenue across contributors. Conceptually:

```text
individual solo mining:
    high variance, full block reward when successful

pooled mining:
    lower variance, payout proportional to contributed shares
```

Payout formulas differ by pool and are outside this unit's scope.

### Fee Revenue and Transaction Selection

Miner transaction selection aims to maximize expected revenue:

```text
select transaction package if marginal_fee / marginal_weight is attractive
```

Dependencies mean a high-fee child may make a low-fee parent worth including. This links mining to CPFP and cluster/chunk mempool logic.

---

## 8. Security Assumptions

### What Mining Secures

Mining secures Bitcoin by making chain history costly to rewrite. A transaction buried under more cumulative proof-of-work becomes more expensive to reverse.

Mining contributes to:

- transaction ordering;
- block production;
- Sybil-resistant chain selection through accumulated work;
- economic cost for reorganization attempts.

### What Mining Does Not Secure

Mining does not protect against:

- stolen private keys;
- exchange account compromise;
- wallet malware;
- bad custody procedures;
- off-chain fraud;
- incorrect address labeling;
- invalid local mempool assumptions.

A miner cannot make an invalid block valid merely by hashing it. Full nodes still enforce consensus rules.[^ref-btc-core-validation]

### Mining Centralization Risks

Mining can centralize across several dimensions:

| Dimension | Risk |
|---|---|
| Pool template control | Transaction censorship or policy concentration |
| ASIC manufacturing | Hardware supply bottlenecks |
| Energy access | Geographic and political concentration |
| Firmware/software | Operational monoculture |
| Network connectivity | Propagation advantage for large actors |

These risks do not automatically imply consensus failure, but they affect censorship resistance, revenue distribution, and attack surface.

---

## 9. Bitcoin Core Implementation

### Block Assembler

Bitcoin Core's `node::BlockAssembler` is documented as generating a new block without valid proof-of-work. Its `CreateNewBlock` method constructs a new block template.[^ref-btc-core-miner]

Important state tracked by the block assembler includes:

- block weight;
- signature operation cost;
- number of transactions;
- accumulated fees;
- next height;
- mempool pointer;
- chainstate reference;
- block creation options.

### Coinbase and Merkle Updates

Bitcoin Core mining source includes coinbase and Merkle-root update logic. In `miner.cpp`, block construction updates coinbase-related commitments and recomputes `hashMerkleRoot` using `BlockMerkleRoot`.[^ref-btc-core-miner]

This implementation matches the conceptual rule:

```text
coinbase changes -> coinbase txid changes -> Merkle root changes -> header hash changes
```

### Mining Interface

Bitcoin Core exposes mining functionality through interfaces. `interfaces/mining.h` includes `Mining::createNewBlock`, which constructs a new block template, and related block creation/check/wait types.[^ref-btc-core-mining-iface]

Bitcoin Core 31.0 release notes state that the IPC mining interface requires the latest `mining.capnp` schema, `Mining.createNewBlock` has default cooldown behavior, and `BlockTemplate.getCoinbaseTx()` returns a structured `CoinbaseTx`.[^ref-btc-core-31-release]

### Mining RPCs

Bitcoin Core RPC mining commands include:

- `getblocktemplate`;
- `submitblock`;
- `submitheader`;
- `getmininginfo`;
- `getnetworkhashps`;
- `prioritisetransaction`;
- `getprioritisedtransactions`.

Bitcoin Core Doxygen for `src/rpc/mining.cpp` lists these mining RPC functions.[^ref-btc-core-mining-rpc]

### Proof-of-Work Validation

Bitcoin Core proof-of-work validation lives outside mining template construction. A mined block is checked by validation code. `CheckProofOfWork` and contextual header validation ensure the block hash and difficulty target are valid for the chain context.[^ref-btc-core-pow]

This separation matters:

```text
miner constructs candidate
ASIC searches for hash
node validates full block
network accepts only valid blocks
```

---

## 10. Consensus, Policy, and Convention

### Consensus

Consensus mining-related rules include:

- block header must satisfy proof-of-work target;
- `nBits` must be valid for height and parent context;
- coinbase must be first transaction;
- coinbase value cannot exceed subsidy plus fees;
- block must satisfy weight, sigops, and transaction validity rules;
- block timestamp must satisfy consensus constraints.

### Mining Policy

Mining policy includes:

- transaction selection;
- fee-rate thresholds;
- template mutation choices;
- block reserved weight;
- private transaction inclusion;
- nonstandard transaction inclusion;
- pool-specific censorship or filtering.

Mining policy can vary between miners while still producing consensus-valid blocks.

### Pool Convention

Pool conventions include:

- share difficulty;
- payout formula;
- miner worker naming;
- coinbase tags;
- job assignment format;
- stale-share handling;
- payout batching.

These are not Bitcoin consensus rules.

---

## 11. On-Chain Implications

### Strong Evidence

Mining data strongly supports:

- block height and timestamp;
- block header fields;
- coinbase transaction contents;
- subsidy and fee claim;
- transaction set included in the block;
- block weight and fee revenue;
- proof-of-work target and header hash;
- whether a block became part of the active chain.

### Weak Evidence

Mining data weakly supports:

- real-world miner identity;
- exact pool hashrate at a moment;
- whether pool or hasher selected transactions;
- censorship intent;
- private transaction source;
- electricity cost or profitability;
- internal payout allocation.

Coinbase tags and payout addresses are useful evidence but can be spoofed, shared, changed, or structured through intermediaries.

### Stale and Orphaned Blocks

If two valid blocks compete at the same height, one may become stale when another branch accumulates more work. Stale blocks can indicate propagation races, network topology, or bad luck.

An analyst should distinguish:

- valid block that lost a race;
- invalid block rejected by nodes;
- private candidate never broadcast;
- pool share below network target.

---

## 12. Institutional Thinking

### Investor and Research View

Institutions analyzing mining should track:

- hashprice and fee share;
- subsidy schedule;
- difficulty adjustments;
- estimated hash rate;
- pool concentration;
- block propagation quality;
- transaction-selection behavior;
- geographic and energy exposure;
- public miner treasury and debt structure where relevant.

### Transaction Risk View

Mining affects transaction settlement through:

- fee-based inclusion probability;
- block interval variance;
- reorg risk;
- censorship risk;
- private relay and direct-to-miner submission.

Mempool appearance is not settlement. Confirmation depends on miner inclusion and block acceptance.

### Custody and Treasury View

Treasury teams should understand mining because:

- fees are paid to miners;
- congestion changes withdrawal costs;
- CPFP/RBF interact with mining selection;
- confirmation policies are probabilistic;
- coinbase outputs have maturity rules;
- mining pool attribution may affect compliance monitoring.

### Mining Operator View

Mining operators must manage:

- template source reliability;
- full-node validation;
- pool connectivity;
- stale rate;
- firmware and ASIC fleet operation;
- payout address security;
- regulatory and energy constraints;
- treasury management of mined BTC.

---

## 13. Common Misinterpretations

### "Miners Validate Bitcoin Alone"

No. Miners propose blocks. Full nodes validate blocks. Hashpower cannot force invalid blocks into consensus if validating nodes reject them.

### "Mining Means Solving a Complex Math Puzzle"

The practical process is repeated hashing of candidate block headers until a hash is below target. It is probabilistic search, not solving a unique puzzle with a shortcut.

### "The Nonce Is the Whole Search Space"

No. Miners also vary coinbase extra nonce, Merkle root, timestamp, version fields, and transaction selection.

### "Pool Shares Are Bitcoin Blocks"

No. Shares prove work to the pool under an easier target. Only shares also below the network target can become valid Bitcoin blocks.

### "Coinbase Tag Proves Miner Identity"

No. Coinbase tags are useful but not cryptographic identity proofs.

### "Highest Fee Transactions Are Always Included"

No. Dependencies, policy, private transactions, template timing, and miner preferences can change selection.

---

## 14. Research Questions

1. How much transaction selection power is held by pool operators versus individual hashers?
2. How does cluster mempool change block template fee efficiency?
3. How reliable are coinbase tags for pool attribution over time?
4. What stale-rate patterns suggest propagation advantage or network issues?
5. How does fee share of miner revenue change around high-demand periods?
6. What observable evidence indicates private transaction inclusion?
7. How should institutions measure mining centralization across pool, ASIC, energy, and jurisdiction dimensions?

---

## 15. Practical Exercises

### Exercise 1: Inspect a Block

Choose a recent block and record:

- block hash;
- previous block hash;
- `nBits`;
- nonce;
- timestamp;
- transaction count;
- coinbase transaction;
- total fees;
- block weight.

### Exercise 2: Coinbase Analysis

For the same block:

1. Decode the coinbase transaction.
2. Identify coinbase outputs.
3. Compare claimed value to subsidy plus fees.
4. Record any coinbase tag or pool marker.
5. Label attribution confidence.

### Exercise 3: Template Fields

Run on a synced node if available:

```bash
bitcoin-cli getblocktemplate '{"rules":["segwit"]}'
```

Record:

- previous block hash;
- height;
- bits;
- target;
- coinbase value;
- transaction count;
- weight limit;
- mutable fields.

### Exercise 4: Share vs Block

Explain why a pool share can be valid for payout accounting but not valid as a Bitcoin block.

Use:

```text
share_target > network_target
hash <= share_target
hash > network_target
```

---

## 16. Evidence Classification

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Proof-of-work, incentives, chain history cost | A |
| REF-BTC-DEV-MINING-001 | Official developer documentation | Solo mining, pooled mining, getblocktemplate, Stratum overview, nonce and extra nonce | A |
| REF-BIP-0022 | API/RPC BIP | `getblocktemplate` and `submitblock` fundamentals | A |
| REF-BIP-0023 | API/RPC BIP | Pooled mining extensions to `getblocktemplate` | A |
| REF-BTC-CORE-MINER-001 | Primary implementation source | `BlockAssembler`, `CreateNewBlock`, coinbase/Merkle update logic | A |
| REF-BTC-CORE-MINING-IFACE-001 | Primary implementation source | Mining interface and block template creation APIs | A |
| REF-BTC-CORE-MINING-RPC-001 | Primary implementation/RPC source | Mining RPCs including `getblocktemplate`, `submitblock`, `submitheader` | A |
| REF-BTC-CORE-31-RELEASE-001 | Release documentation | Cluster mempool template effects and IPC mining interface changes | A |
| REF-BTC-CORE-POW-001 | Primary implementation source | Proof-of-work validation functions | A |
| REF-BTC-CORE-VALIDATION-001 | Primary implementation source | Full block validation boundary | A |

### Claim Ledger

| Claim | Classification | Evidence |
|---|---|---|
| Mining constructs candidate blocks and searches for a header hash below target | FACT | Whitepaper, developer mining docs, Bitcoin Core PoW source |
| `BlockAssembler` creates block templates without valid proof-of-work | FACT | Bitcoin Core `BlockAssembler` documentation |
| Coinbase changes can change the Merkle root and expand mining search | FACT | Developer mining docs, Bitcoin Core miner source |
| Solo mining has higher payout variance than pooled mining | FACT | Bitcoin Developer mining guide |
| Pool shares are not necessarily valid Bitcoin blocks | FACT | Developer mining guide, BIP23 target/share model |
| `getblocktemplate` communicates block construction data | FACT | BIP22 and Bitcoin Core RPC docs |
| Mining policy can differ from relay policy | FACT | Bitcoin Core policy/mining boundary and BIP22 rationale |
| Hashpower alone can make invalid blocks valid | COUNTERCLAIM | Rejected; full nodes enforce validation |
| Coinbase tags prove real-world miner identity | COUNTERCLAIM | Rejected; tags are non-consensus metadata |
| Miner transaction selection always follows public mempool fee order exactly | COUNTERCLAIM | Rejected; private orderflow, policy, and timing can differ |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| POLICY | Mining, pool, relay, or operational convention rather than consensus |
| HEURISTIC | Practical inference with known counterexamples |
| UNKNOWN | Evidence is insufficient |

---

## 17. Knowledge Graph

```text
BITCOIN-020 Mining
|
+-- builds_on: BITCOIN-017 Mempool
+-- builds_on: BITCOIN-018 Transaction Fees
+-- builds_on: POW-009 Coinbase Transaction
+-- builds_on: POW-012 Difficulty Adjustment
+-- builds_on: POW-013 Bitcoin Core PoW Validation
|
+-- mining workflow
|   +-- select_transactions
|   +-- build_coinbase
|   +-- compute_merkle_root
|   +-- hash_header
|   +-- submit_block
|
+-- economic incentive
|   +-- subsidy
|   +-- transaction_fees
|   +-- operating_costs
|
+-- coordination
|   +-- solo_mining
|   +-- pooled_mining
|   +-- shares
|   +-- getblocktemplate
|
+-- Bitcoin Core
|   +-- BlockAssembler
|   +-- CreateNewBlock
|   +-- mining RPCs
|   +-- mining interface
|   +-- validation boundary
|
+-- analysis
    +-- facts: block contents, fees, coinbase
    +-- heuristics: miner identity, pool attribution
    +-- risks: stale blocks, censorship, centralization
```

---

## 18. References

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Sections 4-6, proof-of-work and incentive design, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-mining]: Bitcoin Developer Documentation, "Mining," solo mining, pooled mining, getblocktemplate, Stratum overview, nonce and extra nonce behavior, https://developer.bitcoin.org/devguide/mining.html, accessed 2026-08-04.

[^ref-bip-0022]: Luke Dashjr, "BIP 22: getblocktemplate - Fundamentals," 2012-02-28, https://bips.dev/22/ and https://github.com/bitcoin/bips/blob/master/bip-0022.mediawiki, accessed 2026-08-04.

[^ref-bip-0023]: Luke Dashjr, "BIP 23: getblocktemplate - Pooled Mining," 2012-02-28, https://bips.xyz/23, accessed 2026-08-04.

[^ref-btc-core-miner]: Bitcoin Core Contributors, `src/node/miner.h` and `src/node/miner.cpp`, `BlockAssembler`, `CreateNewBlock`, coinbase and Merkle-root update logic, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/classnode_1_1_block_assembler.html and https://doxygen.bitcoincore.org/miner_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-mining-iface]: Bitcoin Core Contributors, `src/interfaces/mining.h`, mining interface and block template APIs, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/interfaces_2mining_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-mining-rpc]: Bitcoin Core Contributors, `src/rpc/mining.cpp` and Bitcoin Core mining RPC documentation, mining RPCs, `getblocktemplate`, `submitblock`, and `submitheader`, Bitcoin Core Doxygen 31.99.0 documentation and RPC docs, https://doxygen.bitcoincore.org/rpc_2mining_8cpp.html and https://bitcoincore.org/en/doc/26.0.0/rpc/mining/getblocktemplate/, accessed 2026-08-04.

[^ref-btc-core-31-release]: Bitcoin Core Contributors, "Bitcoin Core 31.0 Release Notes," cluster mempool and IPC mining interface changes, https://bitcoin.org/en/releases/31.0/ and https://bitcoincore.org/en/releases/31.0/, accessed 2026-08-04.

[^ref-btc-core-pow]: Bitcoin Core Contributors, `src/pow.cpp` and `src/pow.h`, proof-of-work target and hash validation, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/pow_8cpp_source.html and https://doxygen.bitcoincore.org/pow_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-validation]: Bitcoin Core Contributors, `src/validation.cpp`, block and header validation boundaries, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/validation_8cpp.html, accessed 2026-08-04.

---

## 19. Cross References

### Parent

- Bitcoin Internals

### Previous

- BITCOIN-019 — Wallets and Key Management

### Next

- BITCOIN-021 — Blocks and Block Headers

### Related

- BITCOIN-007 — Whitepaper Section 6 — Incentive
- BITCOIN-017 — Mempool
- BITCOIN-018 — Transaction Fees
- POW-009 — Coinbase Transaction and Mining Commitments
- POW-011 — Cumulative Chainwork
- POW-012 — Difficulty Adjustment
- POW-013 — Bitcoin Core Proof-of-Work Validation
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- Block construction was separated from proof-of-work search.
- Solo mining, pooled mining, pool shares, coinbase rewards, transaction selection, and block submission were distinguished.
- Network target and pool share target were separated.
- Bitcoin Core `BlockAssembler`, mining interfaces, mining RPCs, PoW validation, and full block validation were referenced at the correct boundaries.
- Current Bitcoin Core 31.0 mining IPC and cluster mempool template-selection context was included without treating it as consensus.

### Evidence Review

Passed.

- Mining workflow claims cite Bitcoin Developer documentation, BIP22/BIP23, and Bitcoin Core source documentation.
- Proof-of-work and incentive claims cite the whitepaper and Bitcoin Core PoW validation references.
- Current implementation claims cite Bitcoin Core Doxygen and 31.0 release notes.
- Pool identity, coinbase tag, and transaction-selection claims are labeled with caveats where evidence is heuristic.

### Editorial Review

Passed.

- Markdown headings follow the project deep-dive structure.
- Metadata is complete.
- Tables and code fences are closed.
- Terminology is consistent: mining, candidate block, block template, coinbase, nonce, extra nonce, share, pool, target.

### Adversarial Review

Passed.

- The document does not claim miners alone define consensus.
- It does not treat shares as Bitcoin blocks.
- It does not treat coinbase tags as verified miner identity.
- It does not claim transaction selection always follows public mempool fee ordering exactly.
- It does not conflate mining policy, pool convention, and consensus rules.

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
