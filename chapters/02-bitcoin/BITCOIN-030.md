---
knowledge_id: BITCOIN-030
title: SegWit
subtitle: Witness Segregation, Malleability Mitigation, txid and wtxid, Block Weight, Witness Commitment, and Validation Boundaries
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 135 min
estimated_study: 390 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Consensus
  - Transactions
  - SegWit
parent:
  - Modern Bitcoin
prerequisites:
  - BITCOIN-009
  - BITCOIN-015
  - BITCOIN-016
  - BITCOIN-021
  - BITCOIN-022
related_topics:
  - Transaction Malleability
  - Witness Commitment
  - txid
  - wtxid
  - P2WPKH
  - P2WSH
  - Block Weight
  - Bech32
primary_sources:
  - REF-BTC-WP-001
  - REF-BIP-0141
  - REF-BIP-0143
  - REF-BIP-0144
  - REF-BTC-DEV-TRANSACTIONS-001
  - REF-BTC-DEV-BLOCKCHAIN-001
  - REF-BTC-DEV-P2P-001
  - REF-BTC-CORE-CONSENSUS-VALIDATION-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-BIPS-001
tags:
  - bitcoin
  - segwit
  - consensus
  - transactions
  - witness
  - malleability
  - block-weight
  - p2wsh
---

# SegWit
> Modern Bitcoin  
> Research Unit: BITCOIN-030

---

## Research Brief

```yaml
knowledge_id: BITCOIN-030
title: SegWit
research_question: >
  What did Segregated Witness change in Bitcoin's transaction and block model,
  how does it mitigate involuntary transaction malleability, what is the
  difference between txid and wtxid, how do witness programs and witness
  commitments work, and how should analysts separate consensus changes from
  relay, serialization, and wallet-level consequences?
document_type: deep-dive
difficulty: L400
prerequisites:
  - BITCOIN-009
  - BITCOIN-015
  - BITCOIN-016
  - BITCOIN-021
  - BITCOIN-022
parent: Modern Bitcoin
previous: BITCOIN-029
next: BITCOIN-031
related_topics:
  - Malleability
  - Witness Programs
  - Block Weight
  - SPV
  - Transaction Relay
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
  - Full Lightning Network protocol design
  - Full Taproot design
  - Address UX details beyond analytical necessity
  - Exhaustive historical activation politics
  - Detailed wallet implementation tutorials
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Explain why SegWit was introduced.
- Distinguish transaction effects from witness data used only for validation.
- Distinguish `txid` from `wtxid`.
- Explain how SegWit mitigates involuntary transaction malleability.
- Explain the witness-commitment structure in the coinbase transaction.
- Explain native versus P2SH-nested witness programs.
- Explain block weight and virtual size.
- Separate consensus changes from relay and wallet-level consequences.

---

## 2. Key Questions

1. What problem was SegWit trying to solve?
2. What exactly is "witness" data?
3. Why do SegWit transactions have both `txid` and `wtxid`?
4. How does SegWit reduce malleability risk?
5. What is committed in the block header, and what is committed through coinbase witness commitment?
6. What are P2WPKH and P2WSH?
7. Why did SegWit introduce block weight instead of simply "raising block size"?
8. What changed in network relay and serialization?
9. What can non-upgraded nodes see and fail to validate?

---

## 3. Executive Summary

Segregated Witness, specified primarily in BIP141, introduces a new structure called witness data that is committed to blocks separately from the legacy transaction Merkle tree. Witness data contains information needed to validate spends, especially signatures and related script data, but it is not required to determine the transaction's effects on the UTXO set. This is also the layer where SegWit extends the original whitepaper-era distinction between publication proofs and full validation data.[^ref-bip-0141][^ref-btc-wp]

This separation solves several problems at once. Most importantly, it removes involuntary signature-data malleability from transaction identification for SegWit spends, because the legacy `txid` no longer commits to witness serialization. BIP141 therefore defines both a traditional `txid` and a witness-inclusive `wtxid`.[^ref-bip-0141]

SegWit also changes how block capacity is accounted for. Rather than using only the old serialized-size model, it introduces block weight and transaction weight, discounting witness bytes relative to base bytes in a way that remains soft-fork compatible for older nodes.[^ref-bip-0141][^ref-btc-core-consensus-validation]

At the block level, witness data is not placed directly in the 80-byte header. Instead, BIP141 requires a witness commitment in the coinbase transaction, committing the block's `wtxid` tree through a commitment hash. Bitcoin Core validation exposes this with weight helpers and witness-commitment checks such as `GetWitnessCommitmentIndex` and witness-malleation validation paths.[^ref-bip-0141][^ref-btc-core-consensus-validation][^ref-btc-core-validation]

For analysts, SegWit is not just a throughput story. It is a transaction-identity, serialization, validation, and commitment-system change with lasting effects on wallets, fee calculation, relay, and higher-layer protocols.

---

## 4. Protocol Structure

### Before SegWit

Legacy transaction identity used one serialization, matching the traditional raw transaction structure documented in developer references.[^ref-btc-dev-transactions]

```text
[nVersion][txins][txouts][nLockTime]
```

and signatures lived inside `scriptSig`, meaning signature-related mutations could change the transaction hash.[^ref-bip-0141]

### After SegWit

SegWit defines witness data separately. Each transaction can now have:

- a legacy `txid`, and
- a witness-inclusive `wtxid`.[^ref-bip-0141]

The `wtxid` uses serialization that includes marker, flag, and witness fields.

### Core Design Idea

SegWit separates:

- data needed to determine effects: spends and new outputs,
- data needed to validate authorization: witness.

This distinction is central to why SegWit improves malleability behavior and enables new commitment patterns.

---

## 5. Transaction Identity and Malleability

### `txid`

BIP141 keeps the old `txid` definition unchanged:

```text
txid = double_sha256([nVersion][txins][txouts][nLockTime])
```

### `wtxid`

BIP141 defines:

```text
wtxid = double_sha256([nVersion][marker][flag][txins][txouts][witness][nLockTime])
```

If all inputs are non-witness programs, `wtxid = txid`.[^ref-bip-0141]

### Why This Matters

In legacy transactions, mutations to signature-related data could change the transaction hash even if the economic effects were unchanged. For SegWit spends, witness mutations no longer alter the legacy `txid`, so involuntary malleability is greatly reduced for those spends.[^ref-bip-0141]

Important boundary:

- SegWit mitigates involuntary malleability for SegWit inputs.
- It does not make every possible higher-level transaction graph risk disappear.
- Legacy inputs remain legacy.

---

## 6. Witness Programs and Spending Forms

### Witness Program

BIP141 gives special meaning to a script form consisting of:

- a version byte (`OP_0` to `OP_16`), followed by
- a push of 2 to 40 bytes, the witness program.[^ref-bip-0141]

### Native Witness Program

If the `scriptPubKey` is directly a witness program:

- the `scriptSig` must be empty,
- witness data carries the spending arguments.[^ref-bip-0141]

### P2SH-Nested Witness Program

If the `scriptPubKey` is P2SH and the `redeemScript` is itself a witness program:

- the `scriptSig` must contain only that redeem script push,
- witness carries the actual unlocking data.[^ref-bip-0141]

This gave SegWit a compatibility bridge before native witness output types were broadly used.

### Version 0 Programs

For witness version 0:

- 20-byte program means P2WPKH,
- 32-byte program means P2WSH.[^ref-bip-0141]

P2WPKH moves the signature and pubkey to witness. P2WSH moves the script and witness stack there, with `SHA256(witnessScript)` committed in the program.

---

## 7. Block-Level Commitment Structure

### Header Merkle Root

The block header still commits to the legacy transaction Merkle root, derived from `txid`s, not `wtxid`s.[^ref-btc-dev-blockchain][^ref-bip-0141]

### Witness Commitment

BIP141 adds a new block rule requiring commitment to witness data through the coinbase transaction. The witness Merkle root uses `wtxid`s as leaves, treating coinbase `wtxid` as all-zeroes, and places the commitment in a coinbase `scriptPubKey` matching the defined pattern.[^ref-bip-0141]

The commitment structure includes:

```text
OP_RETURN
0x24 push length
0xaa21a9ed commitment header
32-byte commitment hash = Double-SHA256(witness_root_hash | witness_reserved_value)
```

and the coinbase input witness must contain a single 32-byte witness reserved value.[^ref-bip-0141]

### Why Not Put Witness Tree in the Header?

The witness commitment is nested through coinbase for soft-fork compatibility. Older nodes continue to see the old header and Merkle-root model, while upgraded nodes validate the added witness commitment.[^ref-bip-0141]

---

## 8. Weight, Size, and Capacity

### Block Weight

BIP141 replaces the old single-size limit with block weight:

```text
block_weight = base_size * 3 + total_size
```

with the consensus rule:

```text
block_weight <= 4,000,000
```

where base size is serialization without witness data, and total size includes witness serialization.[^ref-bip-0141]

### Transaction Weight and Virtual Size

BIP141 similarly defines transaction weight and virtual transaction size:

```text
tx_weight = base_tx_size * 3 + total_tx_size
vsize = ceil(tx_weight / 4)
```

Bitcoin Core implements the weight formula directly in `GetTransactionWeight` and `GetBlockWeight` using serialization with and without witness data.[^ref-btc-core-consensus-validation]

### Analytical Implication

SegWit does not mean "witness bytes are free." It means witness bytes are discounted relative to base bytes in the block-capacity accounting model.

---

## 9. Relay and Serialization Changes

### BIP144

BIP144 adds new transaction and block serialization rules for peers that relay witness-bearing data, including witness-aware inventory types and serialization expectations in the P2P layer.[^ref-bip-0144][^ref-btc-dev-p2p]

Examples include witness-related inventory identifiers such as `MSG_WITNESS_TX` and `MSG_WITNESS_BLOCK` in developer P2P documentation.[^ref-btc-dev-p2p]

### Why Relay Changed

Once witness data becomes distinct from legacy serialization, the network layer needs a way to request and serve transactions and blocks with or without witness serialization depending on peer capabilities and use case.

---

## 10. Signature Digest and Validation Semantics

### BIP143

BIP143 introduces a new transaction digest algorithm for signature verification in version 0 witness programs.[^ref-bip-0141][^ref-bip-0143]

This matters because SegWit is not only about moving bytes around. It also changes how signatures for witness version 0 spends commit to transaction data.

### Validation Surface

Bitcoin Core validation exposes witness-specific boundaries through:

- `GetWitnessCommitmentIndex`,
- witness-commitment constants,
- witness-malleation checks,
- and transaction validation states such as `TX_WITNESS_MUTATED` and `TX_WITNESS_STRIPPED`.[^ref-btc-core-consensus-validation][^ref-btc-core-validation]

These show that witness data is a first-class validation concern, not a wallet-only encoding detail.

---

## 11. Validation Boundaries

### Non-Upgraded Nodes

BIP141 backward-compatibility section explains that non-upgraded nodes do not see or validate witness data and interpret witness programs under old rules, with significant limitations.[^ref-bip-0141]

This is the essence of the soft-fork design:

- old nodes keep operating,
- upgraded nodes enforce additional rules,
- witness semantics are hidden behind structures that remain acceptable to old software.

### SegWit Is Not Just Bigger Blocks

Overstated interpretations should be avoided:

- SegWit is not merely a blocksize increase.
- It is not merely a new address format.
- It is not a total cure for every transaction dependency risk.

It is a combined consensus, serialization, validation, and commitment reform.

---

## 12. Security Assumptions and Failure Modes

### Malleability Reduction

SegWit fundamentally improves the transaction-malleability situation for SegWit spends because witness data is removed from `txid` calculation. This is important for unconfirmed transaction dependency chains and protocols layered on top of Bitcoin.[^ref-bip-0141]

### SPV and Proof Implications

BIP141 motivation also notes that optional transmission of signature data can reduce SPV proof size and improve bandwidth tradeoffs, though SPV remains weaker than full validation.[^ref-bip-0141]

### Policy vs Consensus

BIP141 also discusses relay and mining policies that accompanied the initial release, including witness-related usage cautions. Analysts should keep these separate from the consensus changes themselves.[^ref-bip-0141]

### Legacy Coexistence

Bitcoin did not become a purely witness-native system overnight. Nested forms, legacy forms, and later native-address adoption mean that transaction datasets can mix multiple eras and spend types. Analysts who treat "SegWit" as binary often lose important detail.

---

## 13. Mathematical or Economic Model

### Weight Formula

The core quantitative change is:

```text
weight = stripped_size * 3 + total_size
```

which Bitcoin Core notes is equivalent to:

```text
weight = stripped_size * 4 + witness_size
```

since `witness_size = total_size - stripped_size`.[^ref-btc-core-consensus-validation]

### Feerate Implication

Because transaction fees are commonly compared on a virtual-size basis after SegWit, the fee market increasingly prices transactions according to witness-discounted capacity use rather than raw total byte count alone. This is why SegWit matters directly to fee analytics.

### Malleability and Dependency Chains

If downstream transactions refer to parent transaction IDs, stabilizing parent `txid` behavior for SegWit spends improves the reliability of pre-confirmation dependency construction. That is one of the key economic-enablement changes for layered protocols, even before discussing any specific second-layer design.

---

## 14. Bitcoin Core Implementation

### `consensus/validation.h`

Bitcoin Core implements witness-related consensus helpers here, including:

- `GetTransactionWeight`,
- `GetBlockWeight`,
- `GetTransactionInputWeight`,
- `GetWitnessCommitmentIndex`,
- witness-commitment constants.[^ref-btc-core-consensus-validation]

### `validation.cpp`

Validation includes witness-specific logic such as `CheckWitnessMalleation`, showing that SegWit added explicit consensus validation pathways at the block level.[^ref-btc-core-validation]

### `doc/bips.md`

Bitcoin Core's implemented-BIP index records BIP144 SegWit-related support as present since 0.13.0.[^ref-btc-core-bips]

---

## 15. On-Chain Implications

### What Analysts Should Track

SegWit-aware analysis may need to distinguish:

- legacy vs witness-bearing inputs,
- `txid` vs `wtxid` contexts,
- native witness vs P2SH-nested witness,
- block weight vs raw size,
- witness commitment presence in blocks where expected.

### Common Dataset Failure

Older or poorly normalized datasets may:

- report only raw size and ignore weight,
- miss witness serialization distinctions,
- misclassify nested witness spends,
- or treat all post-2017 activity as uniformly "SegWit."

### Fee Interpretation

Post-SegWit fee analytics should use vsize- or weight-aware metrics rather than raw byte counts wherever the research question depends on miner inclusion economics.

---

## 16. Institutional Thinking

Institutions should treat SegWit as an infrastructure upgrade with lasting analytical consequences.

### Practical Implications

- Custody and settlement systems should distinguish `txid`-facing workflows from witness-inclusive transaction handling.
- Fee estimation and transaction-cost models should use virtual size.
- Audit and indexing systems should preserve witness-aware serialization where required.
- Malleability assumptions from legacy Bitcoin should not be projected unchanged onto SegWit spends.

---

## 17. Common Misinterpretations

### "SegWit just increased block size"

False. It changed transaction identity, commitment structure, validation boundaries, and capacity accounting.

### "SegWit removed the transaction Merkle root from the header"

False. The header still commits to the legacy `txid` Merkle root; witness data is committed separately through coinbase witness commitment.

### "`wtxid` replaced `txid` everywhere"

False. Both exist, and they serve different purposes.

### "SegWit made every Bitcoin transaction non-malleable"

False. It mitigates involuntary malleability for SegWit spends, not every conceivable transaction-graph risk and not legacy inputs.

### "Witness bytes don't count"

False. They count under weight accounting, just at a discount relative to base bytes.

---

## 18. Research Questions

1. How much analyst error still comes from raw-size rather than vsize-based fee measurement?
2. How quickly did nested witness versus native witness adoption diverge across wallet populations?
3. How much of post-SegWit transaction-chain reliability improvement is directly attributable to malleability mitigation?
4. Which datasets preserve witness-aware transaction identity best for institutional audit?

---

## 19. Practical Exercises

### Exercise 1

Explain the difference between `txid` and `wtxid` for a version 0 witness spend.

### Exercise 2

Given a block with witness-bearing transactions, identify which data is committed by the header Merkle root and which data is committed through the coinbase witness commitment.

### Exercise 3

Compute a transaction's fee density using virtual size rather than raw total bytes and explain why the difference matters.

### Exercise 4

Classify a spend as legacy, native P2WPKH, P2SH-nested P2WPKH, native P2WSH, or P2SH-nested P2WSH from its script structure.

---

## 20. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Witness structure, txid/wtxid, witness commitment, weight rules | Directly specified | BIP141 and Core consensus helpers |
| v0 witness digest semantics | Directly specified | BIP143 |
| Witness-aware relay serialization | Directly specified | BIP144 and P2P reference |
| Throughput and fee implications | Directly specified plus inference | Weight rules are specified; market effects are analytical |
| Malleability mitigation and higher-layer enablement | Directly specified plus inference | BIP141 motivation plus analytical consequence |

---

## 21. Knowledge Graph

```text
SegWit
├─ Motivation
│  ├─ malleability mitigation
│  ├─ witness segregation
│  ├─ script extensibility
│  └─ capacity accounting reform
├─ Identity
│  ├─ txid
│  └─ wtxid
├─ Script Forms
│  ├─ P2WPKH
│  ├─ P2WSH
│  ├─ native witness
│  └─ P2SH-nested witness
├─ Block Commitments
│  ├─ txid merkle root
│  ├─ witness merkle root
│  └─ coinbase witness commitment
├─ Capacity
│  ├─ block weight
│  ├─ tx weight
│  └─ vsize
└─ Implications
   ├─ fee analytics
   ├─ relay serialization
   ├─ wallet handling
   └─ off-chain protocol support
```

---

## 22. References

### Primary Sources

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," especially Sections 8 and 11 for original SPV and confirmation context. https://bitcoin.org/bitcoin.pdf

[^ref-bip-0141]: BIP141, "Segregated Witness (Consensus layer)." https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki

[^ref-bip-0143]: BIP143, "Transaction Signature Verification for Version 0 Witness Program." https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki

[^ref-bip-0144]: BIP144, "Segregated Witness (Peer Services)." https://github.com/bitcoin/bips/blob/master/bip-0144.mediawiki

[^ref-btc-dev-transactions]: Bitcoin Developer Reference, "Transactions." https://developer.bitcoin.org/reference/transactions.html

[^ref-btc-dev-blockchain]: Bitcoin Developer Reference, "Block Chain." https://developer.bitcoin.org/reference/block_chain.html

[^ref-btc-dev-p2p]: Bitcoin Developer Reference, "P2P Network," including witness-aware inventory types. https://developer.bitcoin.org/reference/p2p_networking.html

[^ref-btc-core-consensus-validation]: Bitcoin Core Doxygen, `consensus/validation.h`, including weight helpers and witness commitment helpers. https://doxygen.bitcoincore.org/consensus_2validation_8h_source.html

[^ref-btc-core-validation]: Bitcoin Core Doxygen, `validation.cpp`, including `CheckWitnessMalleation`. https://doxygen.bitcoincore.org/validation_8cpp.html

[^ref-btc-core-bips]: Bitcoin Core `doc/bips.md`, implemented BIP index including BIP144 SegWit support. https://github.com/bitcoin/bitcoin/blob/master/doc/bips.md

### Supporting Interpretation Notes

- Where this document discusses fee-market consequences, layered-protocol enablement, or institutional data-model implications, those statements are analytical inferences built on the SegWit consensus and serialization changes rather than stand-alone protocol commands.

---

## 23. Cross References

### Previous

- BITCOIN-029 — Bitcoin Game Theory

### Next

- BITCOIN-031 — Taproot

### Related

- BITCOIN-009 — Whitepaper Section 8 — Simplified Payment Verification
- BITCOIN-015 — Transactions in Depth
- BITCOIN-016 — Script & ScriptPubKey
- BITCOIN-018 — Transaction Fees
- BITCOIN-021 — Blocks and Block Headers
- BITCOIN-022 — Nodes and Network Propagation
- BITCOIN-031 — Taproot

---

## Review Status

### Technical Review

Passed.

- Witness structure, identity changes, capacity accounting, commitment structure, and validation boundaries were separated.
- `txid` and `wtxid` were kept distinct throughout.
- Header Merkle commitment and coinbase witness commitment were not conflated.
- Native and nested witness forms were described separately.

### Evidence Review

Passed.

- BIP141 supports the core SegWit consensus and structure claims.
- BIP143 supports witness-v0 digest semantics.
- BIP144 and P2P reference support relay and serialization claims.
- Core Doxygen supports weight and witness-commitment implementation references.
- Interpretive consequences are labeled as inference where appropriate.

### Editorial Review

Passed.

- Structure follows the project deep-dive format.
- Metadata is complete.
- Terminology is consistent: witness, txid, wtxid, witness program, block weight, vsize, witness commitment.
- Tables and code fences are closed.

### Adversarial Review

Passed.

- The document does not reduce SegWit to a simple blocksize change.
- It does not claim `wtxid` replaced `txid` universally.
- It does not claim witness bytes are free.
- It does not overstate malleability fixes beyond SegWit scope.
- It does not confuse relay changes with consensus changes.

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
