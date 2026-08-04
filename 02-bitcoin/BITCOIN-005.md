---
knowledge_id: BITCOIN-005
title: Whitepaper Section 4 — Proof of Work
subtitle: Distributed Timestamping, Hash Search, Difficulty Adjustment, and Chain Work
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 75 min
estimated_study: 210 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Consensus
  - Proof of Work
  - Distributed Systems
parent:
  - Bitcoin Whitepaper
prerequisites:
  - BITCOIN-003
  - BITCOIN-004
related_topics:
  - Timestamp Server
  - Block Headers
  - Difficulty Adjustment
  - Chain Work
  - Mining
  - Double Spending
  - Network Consensus
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-BLOCKCHAIN-001
  - REF-BTC-DEV-MINING-001
  - REF-BTC-CORE-POW-001
  - REF-BTC-CORE-CHAINPARAMS-001
  - REF-BIP-0094
tags:
  - bitcoin
  - proof-of-work
  - whitepaper
  - difficulty
  - chainwork
  - mining
---

# Whitepaper Section 4 — Proof of Work
> Deep Dive Series  
> Research Unit: BITCOIN-005

---

## Research Brief

```yaml
knowledge_id: BITCOIN-005
title: Whitepaper Section 4 — Proof of Work
research_question: >
  How does Bitcoin's proof-of-work mechanism transform the timestamp server
  from a published hash chain into a permissionless ordering system whose
  history is expensive to rewrite and whose work target is adjusted over time?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-003
  - BITCOIN-004
parent: Bitcoin Whitepaper
previous: BITCOIN-004
next: BITCOIN-006
related_topics:
  - Block Headers
  - Mining
  - Difficulty Adjustment
  - Chain Work
  - Double Spending
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Original Design
  - Protocol Structure
  - Technical Mechanics
  - Mathematical Model
  - Security Assumptions
  - Bitcoin Core Implementation
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Full mining-pool protocol design
  - Detailed ASIC hardware engineering
  - Section 11 attacker probability calculations
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Explain why Proof of Work is introduced after the timestamp server.
- Define the target, nonce, `nBits`, block hash, and chain work.
- Distinguish original whitepaper language from current Bitcoin implementation.
- Explain why Proof of Work is costly to produce but cheap to verify.
- Explain why modifying a past block requires redoing work for that block and later blocks.
- Distinguish "longest chain" as whitepaper wording from the modern greatest-work selection model.
- Describe how Bitcoin's mainnet difficulty target is adjusted over 2016-block intervals.
- Identify which Proof-of-Work variables are directly observable on-chain.
- Avoid common misconceptions about mining, validation, and majority hash power.

## 2. Key Questions

1. Why is a newspaper-style timestamp server insufficient for a peer-to-peer network?
2. What exactly is being hashed in Bitcoin Proof of Work?
3. Does Proof of Work prove that transactions are valid?
4. Why can a node verify Proof of Work with one header hash?
5. What does `nBits` encode?
6. Why does a lower target mean higher difficulty?
7. How does changing the nonce differ from changing the coinbase transaction?
8. Why is "one-CPU-one-vote" historically important but technically outdated?
9. Why is "longest chain" incomplete terminology in modern Bitcoin?
10. Which Proof-of-Work facts can an on-chain analyst observe directly?

## 3. Executive Summary

Section 4 of the Bitcoin Whitepaper adds Proof of Work to the timestamp server introduced in Section 3. The timestamp server establishes a hash-linked order of events; Proof of Work makes rewriting that order computationally expensive in an open peer-to-peer network.[^ref-btc-wp]

Bitcoin Proof of Work is a constrained hash search over the block header. A valid block header hash must be numerically less than or equal to the target encoded in the header's `nBits` field. The block header contains the previous block hash, Merkle root, time, `nBits`, nonce, and version, so the proof commits to a specific parent block and a specific set of transactions through the Merkle root.[^ref-btc-dev-blockchain]

Proof of Work does not make an invalid block valid. Current Bitcoin Core separately checks the header's work claim and the block's other consensus rules. In `pow.cpp`, Bitcoin Core derives a target from `nBits`, rejects invalid target encodings, and rejects headers whose hash is above the target.[^ref-btc-core-pow] That means mining is a block-proposal mechanism, not unilateral rule-making.

The whitepaper describes majority decision-making using the phrase "longest chain." For current technical writing, the safer concept is greatest cumulative Proof of Work, because equal-height branches can differ in accumulated work when difficulty changes. The whitepaper's core idea remains: rewriting old history requires redoing the work for the modified block and then catching up with the work added after it.[^ref-btc-wp]

## 4. Definition and Scope

Proof of Work is a mechanism that requires a block producer to find data whose cryptographic hash satisfies a network target. In Bitcoin, the relevant data is the 80-byte block header, and the success condition is:

```text
block_header_hash <= target
```

The target is not chosen by the miner. It is encoded in `nBits` and constrained by consensus rules and network parameters.[^ref-btc-dev-blockchain][^ref-btc-core-pow]

This document studies Proof of Work as introduced in Section 4 of the whitepaper and as implemented in modern Bitcoin consensus. It does not fully cover mining-pool payout protocols, ASIC architecture, fee selection algorithms, or the Section 11 probability model.

## 5. Historical Background

The whitepaper explicitly connects Bitcoin's Proof of Work to Adam Back's Hashcash. The design problem is not merely to make block creation expensive. The design problem is to make an open timestamp network resistant to cheap identity creation.[^ref-btc-wp]

Without Proof of Work, a peer-to-peer timestamp server would need some other way to decide whose ordering history counts. If voting were based on network identity, an attacker who could create many identities could distort the vote. The whitepaper frames Proof of Work as a substitute for identity-weighted voting by making influence proportional to expended computational work rather than IP addresses.[^ref-btc-wp]

The whitepaper uses the phrase "CPU power" because it was written in 2008. Modern Bitcoin mining is dominated by specialized ASIC hardware, so "CPU" should be read as historical design language, not a statement about current mining hardware.[^ref-btc-dev-mining]

## 6. Original Design

Section 4 introduces five core ideas.

First, a distributed timestamp server needs a Proof-of-Work system instead of newspaper or Usenet publication. This shifts timestamp publication from an external medium to a peer-to-peer network mechanism.[^ref-btc-wp]

Second, the work consists of scanning for a value whose hash has a required property. The whitepaper describes this as finding a hash that begins with a required number of zero bits, and states that average work increases exponentially as more zero bits are required.[^ref-btc-wp]

Third, Bitcoin implements the search by incrementing a nonce in the block until the block hash satisfies the required condition. In modern protocol terms, the nonce is one block-header field miners vary; miners can also change other header-affecting data such as the coinbase transaction and Merkle root when the 32-bit nonce space is exhausted.[^ref-btc-wp][^ref-btc-dev-mining]

Fourth, the work protects history because changing a past block changes its hash. Since later blocks commit to earlier block headers through the previous-block-hash field, changing one old block requires redoing the work for that block and every later block on the replacement branch.[^ref-btc-wp][^ref-btc-dev-blockchain]

Fifth, the whitepaper describes the majority decision as represented by the chain with the greatest Proof-of-Work effort. Its adjacent shorthand is "longest chain," but the load-bearing property is accumulated work, not visual length.[^ref-btc-wp]

## 7. Protocol Structure

Bitcoin Proof of Work is attached to the block header.

The block header contains:

| Field | Size | Proof-of-Work relevance |
|---|---:|---|
| `version` | 4 bytes | Signals block version and validation rule context. |
| `previous block header hash` | 32 bytes | Commits the candidate block to one parent header. |
| `merkle root hash` | 32 bytes | Commits the header to the ordered transaction set. |
| `time` | 4 bytes | Miner-provided timestamp subject to consensus limits. |
| `nBits` | 4 bytes | Compact encoding of the target threshold. |
| `nonce` | 4 bytes | Header field miners vary during search. |

Bitcoin Developer documentation describes the serialized header as 80 bytes and states that it is hashed as part of the Proof-of-Work algorithm.[^ref-btc-dev-blockchain]

The previous-block-hash field links the candidate block to a specific parent. The Merkle root links the header to the block's transactions. Therefore, a valid header hash proves work over a commitment to both a parent and a transaction set; it does not independently prove that every transaction is valid.[^ref-btc-dev-blockchain]

## 8. Technical Mechanics

The Proof-of-Work loop can be modeled as:

```text
candidate_header = serialize(version, prev_hash, merkle_root, time, nBits, nonce)
hash = SHA256(SHA256(candidate_header))

if hash <= target_from_nBits:
    candidate_header satisfies Proof of Work
else:
    modify nonce or other header-affecting data
    try again
```

Bitcoin Core's current Proof-of-Work check follows this structure at validation time. It derives a target from `nBits`, rejects negative, zero, overflowing, or above-limit targets, and then rejects a header if its hash is greater than the derived target.[^ref-btc-core-pow]

This explains the asymmetry of Proof of Work:

- Producing a valid hash is probabilistic and may require many trials.
- Verifying a presented header is deterministic and cheap.

The whitepaper expresses this asymmetry by stating that average work rises with the required zero bits while verification requires executing a single hash.[^ref-btc-wp]

## 9. Mathematical or Economic Model

Let:

- `H` be the block header hash interpreted as an unsigned 256-bit integer.
- `T` be the target threshold.
- `N = 2^256` be the number of possible 256-bit hash outputs.
- `p` be the probability that one independent hash attempt succeeds.

Under the simplifying assumption that SHA-256d outputs are uniformly distributed over 256-bit values, the success probability for one attempt is:

```text
p = (T + 1) / 2^256
```

The `+ 1` appears because valid hash values include zero and every integer up to and including `T`.

Toy example:

```text
hash space: 0..255
target: 15
valid outputs: 0..15
success probability: 16 / 256 = 6.25%
```

This toy example uses an 8-bit space for clarity. Bitcoin uses a 256-bit hash output, so real targets are vastly smaller relative to the total space.

Difficulty is inversely related to the target. Lower target values reduce the number of acceptable hashes and therefore increase expected work. Bitcoin Developer documentation distinguishes target from difficulty: the target is the threshold a header hash must meet, while difficulty measures work relative to the easiest target.[^ref-btc-dev-blockchain]

## 10. Security Assumptions

Proof of Work relies on several assumptions.

First, hash outputs must be practically unpredictable before computation. If miners could predict successful nonces cheaply, work would no longer represent costly search.

Second, the cost of rewriting history must grow as more blocks are built after a target block. The whitepaper's argument depends on later blocks chaining to earlier blocks and requiring the attacker to redo the modified block's work and the later work.[^ref-btc-wp]

Third, the honest network must control enough hash power to outpace attackers over time. The whitepaper states its security condition in terms of a majority of CPU power not cooperating to attack the network.[^ref-btc-wp] In modern terminology, this is better stated as majority hash power, not majority CPUs.

Fourth, economic and network assumptions matter. Hash power can be temporarily rented, redirected, censored, or coordinated. Proof of Work provides a cost mechanism, not a guarantee that miners are decentralized, honest, or economically independent.

## 11. Failure Modes and Attack Surface

### History Rewrite

An attacker can attempt to replace a confirmed transaction by privately mining an alternative branch. Proof of Work makes this expensive because the attacker must create a branch with more cumulative work than the branch accepted by the network.

### Temporary Forks

Two miners can find valid blocks at similar times. The whitepaper states that nodes may receive different competing blocks first and later converge when another Proof-of-Work block extends one branch.[^ref-btc-wp] This is not a consensus failure; it is an expected behavior in a distributed network.

### Majority Hash-Power Attack

If an attacker controls enough hash power, they can attempt reorganization, censorship, or selfish strategies. The available primary sources do not establish that Proof of Work prevents all attacks below 50 percent hash power. Section 11 of the whitepaper itself models nonzero attacker catch-up probability when the attacker has less hash power than honest miners.[^ref-btc-wp]

### Invalid Block With Valid Work

A valid header hash does not make invalid transactions acceptable. The whitepaper's Network section states that nodes accept a block only if transactions are valid and not already spent.[^ref-btc-wp] Current Bitcoin Core similarly separates Proof-of-Work checks from broader block and transaction validation.[^ref-btc-core-pow]

## 12. Bitcoin Core or Protocol Implementation

Bitcoin Core implements mainnet Proof-of-Work parameters in `src/kernel/chainparams.cpp`. On mainnet, the configured Proof-of-Work limit is:

```text
00000000ffffffffffffffffffffffffffffffffffffffffffffffffffffffff
```

The same source sets the target timespan to `14 * 24 * 60 * 60` seconds and target spacing to `10 * 60` seconds.[^ref-btc-core-chainparams]

Bitcoin Core's `GetNextWorkRequired` changes the required work only at difficulty-adjustment intervals, except for networks that allow special minimum-difficulty rules. For mainnet, `fPowAllowMinDifficultyBlocks` is false.[^ref-btc-core-pow][^ref-btc-core-chainparams]

Bitcoin Core's `CalculateNextWorkRequired` computes the observed timespan across the adjustment window, constrains it to one quarter and four times the target timespan, multiplies the previous target by the constrained ratio, caps it at the Proof-of-Work limit, and returns compact `nBits` encoding.[^ref-btc-core-pow]

Bitcoin Core's `DeriveTarget` and `CheckProofOfWorkImpl` are consensus-critical for validating a header's work claim. `DeriveTarget` rejects invalid target encodings, and `CheckProofOfWorkImpl` rejects the header when the header hash is above the target.[^ref-btc-core-pow]

## 13. Related Standards and BIPs

No BIP originally introduced Bitcoin mainnet Proof of Work; the mechanism predates the BIP process and is described in the 2008 whitepaper.

BIP94 is relevant for a narrow reason: it specifies Testnet 4 differences, including minimum-difficulty behavior and a difficulty-period rule intended to address testnet block storms and time-warp concerns.[^ref-bip-0094] These rules should not be generalized to Bitcoin mainnet.

## 14. Modern Context

Modern Bitcoin mining differs materially from the whitepaper's 2008 "CPU" framing. Mining software constructs block templates and gives mining hardware an 80-byte header plus a target. ASIC hardware then searches nonce space and returns successful work if found.[^ref-btc-dev-mining]

When the nonce field alone is insufficient, mining software can alter header-affecting data. A common method is changing extra nonce data in the coinbase transaction, which changes the coinbase transaction ID, the Merkle root, and therefore the block header.[^ref-btc-dev-mining]

The modern network also separates roles more sharply than simplified explanations suggest:

- Miners assemble and propose blocks.
- Full nodes independently validate block headers, Proof of Work, transactions, scripts, and UTXO state.
- Economic users decide which validated chain state they rely on.

This distinction matters because Proof of Work orders valid history; it does not define validity by itself.

## 15. On-Chain Implications

### Directly Observable

An on-chain analyst can directly observe or deterministically derive:

- block header hash,
- parent hash,
- Merkle root,
- timestamp,
- `nBits`,
- nonce,
- block height,
- competing stale blocks only if separately observed from network data or archival sources,
- cumulative work as calculated from headers.

### Estimated

An analyst can estimate:

- network hash rate,
- miner or pool share,
- expected block interval under a modeled hash-rate assumption,
- cost of rewriting a given depth under assumed hardware and electricity costs.

### Unobservable From Chain Alone

An analyst cannot directly observe:

- every failed hash attempt,
- exact miner electricity cost,
- physical hardware owner,
- private pool coordination,
- whether a miner was honest or strategically withholding a block.

The distinction is essential. `nBits` and block hashes are protocol evidence. Hash rate, miner cost, and entity attribution are model outputs.

## 16. Institutional Thinking

### Observation

A block contains an `nBits` value and a header hash. A validating node can check whether the header hash is less than or equal to the target derived from `nBits`.[^ref-btc-dev-blockchain][^ref-btc-core-pow]

### Weak Interpretation

"A high hash rate proves the network is decentralized."

### Institutional Decomposition

An institutional researcher should separate:

- total observed work,
- ownership concentration,
- pool coordination,
- geographic and regulatory exposure,
- hardware supply concentration,
- energy source exposure,
- node validation behavior.

### Current Conclusion

Proof of Work provides measurable costliness for block production and history replacement. It does not, by itself, prove miner independence, decentralization, or benign intent.

### Confidence

FACT — ECL-A for the protocol validation rule.  
INTERPRETATION — ECL-B for institutional decentralization assessment because it requires off-chain and entity-level evidence.

## 17. Analyst Notes

Use "target" for the numeric threshold and "difficulty" for the relative measure of expected work. Do not use the terms interchangeably.

Use "greatest cumulative work" when describing chain selection in modern technical writing. If quoting or explaining the whitepaper, clarify that "longest chain" is whitepaper shorthand tied to accumulated Proof-of-Work effort.

Do not infer miner identity from a coinbase tag without caveats. Coinbase tags are self-reported data and can be copied or omitted.

When estimating hash rate from block intervals, state the time window and assumptions. Short windows are noisy because block discovery is probabilistic.

## 18. Common Misinterpretations

### Misinterpretation

> Proof of Work validates transactions.

### Assessment

Incorrect.

### Correction

Proof of Work validates that a header hash satisfies a target. Transaction validity is checked by separate consensus rules.

### Misinterpretation

> The longest chain always means the chain with the most blocks.

### Assessment

Incomplete.

### Correction

The whitepaper uses "longest chain," but the security property is greatest accumulated Proof of Work. Block count is not the safest standalone description when difficulty can vary.

### Misinterpretation

> Miners vote by IP address.

### Assessment

Incorrect.

### Correction

The whitepaper explicitly rejects one-IP-address-one-vote as Sybil-vulnerable and replaces it with Proof-of-Work weighting.[^ref-btc-wp]

### Misinterpretation

> A 51 percent attack can create coins from nothing.

### Assessment

Incorrect.

### Correction

Hash power can help an attacker reorganize or censor blocks, but full nodes still reject blocks that violate issuance, transaction, or script rules.

## 19. Counter Evidence and Limitations

The whitepaper's "one-CPU-one-vote" language is historically accurate but not current operational reality. Modern mining uses ASICs and often pool coordination, so CPU-based wording should not be used without historical qualification.[^ref-btc-dev-mining]

The whitepaper's "longest chain" phrase can mislead readers if presented without qualification. The more precise modern concept is accumulated work.

Bitcoin Developer documentation notes an off-by-one issue in the historical difficulty-adjustment implementation, where retargeting uses timestamps from 2015 intervals across 2016 blocks.[^ref-btc-devguide-pow] This does not invalidate Proof of Work, but it matters for precise difficulty explanations.

Network-specific exceptions exist. Testnet rules may allow minimum-difficulty blocks, and BIP94 specifies Testnet 4 differences. These rules should not be imported into mainnet explanations.[^ref-bip-0094]

On-chain data does not reveal failed hashes, exact production cost, or miner intent. Any analysis that claims to know those variables from block data alone exceeds the evidence.

## 20. Research Questions

1. How should analysts estimate hash rate over different time windows without overstating precision?
2. How does mining-pool concentration differ from hardware ownership concentration?
3. Under what conditions can censorship be detected from block contents?
4. How does fee revenue volatility affect the security budget after subsidy declines?
5. How should analysts compare Proof-of-Work cost across assets with different algorithms and hardware markets?

## 21. Challenge

1. Explain why changing one transaction in an old block changes the Proof-of-Work problem for that block.
2. Given a target `T`, write the one-hash success probability and explain the `+ 1` term.
3. Explain why a valid Proof of Work does not make an invalid transaction valid.
4. Distinguish target, difficulty, `nBits`, nonce, and chain work.
5. Identify three Proof-of-Work variables that are directly observable on-chain and three that are not.

## 22. Evidence Classification

| Claim ID | Claim | Classification | ECL | Primary Sources |
|---|---|---|---|---|
| C001 | Section 4 introduces Proof of Work to implement a distributed timestamp server. | FACT | A | REF-BTC-WP-001 |
| C002 | Bitcoin Proof of Work is checked against a target derived from `nBits`. | FACT | A | REF-BTC-DEV-BLOCKCHAIN-001, REF-BTC-CORE-POW-001 |
| C003 | Verification is cheap relative to production because a candidate header can be checked with a hash and comparison. | FACT | A | REF-BTC-WP-001, REF-BTC-CORE-POW-001 |
| C004 | Rewriting an old block requires redoing that block's work and later branch work. | FACT | A | REF-BTC-WP-001, REF-BTC-DEV-BLOCKCHAIN-001 |
| C005 | "Greatest cumulative work" is the safer modern interpretation of whitepaper "longest chain." | INTERPRETATION | B | REF-BTC-WP-001, REF-BTC-DEV-BLOCKCHAIN-001 |
| C006 | Proof of Work does not independently establish transaction validity. | FACT | A | REF-BTC-WP-001, REF-BTC-CORE-POW-001 |
| C007 | Mainnet uses a 2016-block retarget interval with a two-week target timespan and ten-minute target spacing. | FACT | A | REF-BTC-CORE-CHAINPARAMS-001, REF-BTC-CORE-POW-001 |
| C008 | Hash rate, miner cost, and miner intent are not directly observable from on-chain block data alone. | INTERPRETATION | B | REF-BTC-DEV-BLOCKCHAIN-001, REF-BTC-DEV-MINING-001 |

## 23. Knowledge Graph

```text
BITCOIN-004 Timestamp Server
│
├── requires: hash commitments
├── requires: public transaction ordering
└── exposes: need for peer-to-peer ordering authority
        │
        ▼
BITCOIN-005 Proof of Work
│
├── commits_to: previous block hash
├── commits_to: Merkle root
├── implemented_by: block-header hash search
├── constrained_by: nBits target
├── adjusted_by: difficulty retarget
├── measured_by: cumulative chain work
└── limits: cheap identity creation
        │
        ▼
BITCOIN-006 Network
│
├── propagates: transactions
├── propagates: candidate blocks
└── validates: accepted chain extension
```

## 24. References

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Sections 3-5 and 11, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-blockchain]: Bitcoin Developer Documentation, "Block Chain — Block Headers" and "Target nBits," https://developer.bitcoin.org/reference/block_chain.html, accessed 2026-08-04.

[^ref-btc-dev-mining]: Bitcoin Developer Documentation, "Mining," https://developer.bitcoin.org/devguide/mining.html, accessed 2026-08-04.

[^ref-btc-devguide-pow]: Bitcoin Developer Documentation, "Block Chain — Proof Of Work," https://developer.bitcoin.org/devguide/block_chain.html, accessed 2026-08-04.

[^ref-btc-core-pow]: Bitcoin Core Contributors, `src/pow.cpp`, functions `GetNextWorkRequired`, `CalculateNextWorkRequired`, `DeriveTarget`, and `CheckProofOfWorkImpl`, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/pow_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-chainparams]: Bitcoin Core Contributors, `src/kernel/chainparams.cpp`, class `CMainParams`, Bitcoin Core master branch, https://raw.githubusercontent.com/bitcoin/bitcoin/master/src/kernel/chainparams.cpp, accessed 2026-08-04.

[^ref-bip-0094]: Fabian Jahr, "BIP94: Testnet 4," Bitcoin Improvement Proposals repository, https://github.com/bitcoin/bips/blob/master/bip-0094.mediawiki, accessed 2026-08-04.

## 25. Cross References

### Parent

- Bitcoin Whitepaper

### Previous

- BITCOIN-004 — Whitepaper Section 3 — Timestamp Server

### Next

- BITCOIN-006 — Whitepaper Section 5 — Network

### Related

- POW-001 — Proof of Work Foundations
- POW-002 — Hashcash and Costly Signals
- POW-003 — Bitcoin Block Headers
- POW-004 — Target and Difficulty
- POW-005 — Difficulty Adjustment
- POW-008 — Bitcoin Mining

## Review Status

### Technical Review

Passed.

- Protocol behavior checked against Bitcoin Whitepaper Section 4, Bitcoin Developer block-header documentation, and Bitcoin Core `pow.cpp`.
- Consensus and policy were separated.
- Mainnet and testnet-specific Proof-of-Work rules were separated.
- Mathematical success-probability example was checked with a finite toy hash space.

### Evidence Review

Passed.

- Material protocol claims cite primary or protocol-maintainer sources.
- Source-code references identify repository, file, relevant functions, branch/documentation version, and access date.
- Historical whitepaper claims are separated from modern implementation claims.
- Unknown or unobservable variables are not presented as facts.

### Editorial Review

Passed.

- Markdown headings follow the required deep-dive structure.
- Metadata is complete under the current authoring guide.
- Canonical terminology is used consistently: target, `nBits`, nonce, difficulty, cumulative work.

### Adversarial Review

Passed.

- Overstatement risks were reduced around "longest chain," "one-CPU-one-vote," 51 percent attacks, and miner identity.
- Network-specific exceptions were isolated.
- The document does not claim that Proof of Work proves transaction validity, miner honesty, or decentralization.

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
