---
knowledge_id: BITCOIN-014
title: UTXO Model
subtitle: Unspent Outputs, Outpoints, Chainstate, Transaction Validation, Fees, and On-Chain Analysis
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 95 min
estimated_study: 260 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - UTXO
  - Transactions
  - Validation
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-003
  - BITCOIN-010
  - BITCOIN-013
  - POW-009
related_topics:
  - Transaction Inputs
  - Transaction Outputs
  - Outpoints
  - Chainstate
  - CCoinsView
  - Fee Calculation
  - Double Spend Prevention
  - Coinbase Maturity
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-TRANSACTIONS-001
  - REF-BTC-DEV-BLOCKCHAIN-001
  - REF-BTC-CORE-TRANSACTION-001
  - REF-BTC-CORE-AMOUNT-001
  - REF-BTC-CORE-COINS-001
  - REF-BTC-CORE-TX-CHECK-001
  - REF-BTC-CORE-TX-VERIFY-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-VALIDATION-H-001
tags:
  - bitcoin
  - internals
  - utxo
  - transaction-validation
  - chainstate
  - coinsview
  - outpoint
---

# UTXO Model
> Bitcoin Internals  
> Research Unit: BITCOIN-014

---

## Research Brief

```yaml
knowledge_id: BITCOIN-014
title: UTXO Model
research_question: >
  How does Bitcoin represent spendable value as unspent transaction outputs,
  how do full nodes update that set during block validation, and what does
  the UTXO model imply for fees, double-spend prevention, privacy, and
  institutional on-chain analysis?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-003
  - BITCOIN-010
  - BITCOIN-013
  - POW-009
parent: Bitcoin Internals
previous: BITCOIN-013
next: BITCOIN-015
related_topics:
  - Transaction Structure
  - Coinbase Transaction
  - Chainstate
  - Transaction Fees
  - Mempool
  - Script Validation
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Original Design
  - Protocol Structure
  - Technical Mechanics
  - Mathematical or Economic Model
  - Security Assumptions
  - Bitcoin Core Implementation
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Complete script execution semantics
  - Full mempool policy
  - Wallet coin-selection algorithms
  - Complete database-engine internals
  - Non-Bitcoin account-model comparison beyond brief contrast
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Define a UTXO precisely.
- Explain the difference between a transaction output and an unspent transaction output.
- Explain what an outpoint is.
- Explain how a transaction consumes old UTXOs and creates new UTXOs.
- Explain why Bitcoin does not use mutable account balances at consensus level.
- Explain how input values determine transaction fees.
- Explain how the UTXO model prevents double spends in one active chain.
- Explain why coinbase outputs have special maturity rules.
- Identify how Bitcoin Core represents and queries the UTXO set.
- Distinguish consensus UTXO facts from wallet balances, address clusters, and analytics heuristics.

---

## 2. Key Questions

1. What is a UTXO?
2. How is a UTXO identified?
3. What is the difference between `txid:vout` and an address?
4. How does a transaction spend previous outputs?
5. How does a transaction create new outputs?
6. Why does a full node need a UTXO set?
7. How are transaction fees computed from UTXO values?
8. What makes a double spend invalid in one active chain?
9. How are coinbase-created outputs different?
10. What does Bitcoin Core store in `Coin` and `CCoinsView`?
11. What can an analyst infer from UTXO data?
12. What cannot be inferred from UTXO data alone?

---

## 3. Executive Summary

Bitcoin's UTXO model represents spendable value as discrete unspent transaction outputs. A transaction consumes one or more previous outputs and creates one or more new outputs. The current set of unspent outputs is the state that full nodes use to validate future spends.[^ref-btc-dev-transactions]

The core rule is:

```text
inputs reference existing unspent outputs
    -> transaction proves spending authority
    -> old outputs are marked spent
    -> new outputs become spendable
```

This differs from an account model. Bitcoin consensus does not maintain a mutable balance field for each user. Wallet balances are derived from the set of UTXOs controlled by the wallet's keys or scripts.

An output is not automatically a UTXO forever. It is a UTXO only while it remains unspent in the active chain. Once spent, it is removed from the spendable set. If a reorganization disconnects a block, previously spent outputs may be restored and outputs created by disconnected transactions may be removed.

Bitcoin Core models this through transaction primitives, `Coin`, `CCoinsView`, `CCoinsViewCache`, and validation functions. Doxygen describes `CCoinsView` as a view on the open transaction output dataset, with methods such as `GetCoin`, `HaveCoin`, and `BatchWrite`.[^ref-btc-core-coins] Bitcoin Core's `CheckTxInputs` checks that inputs are available, enforces coinbase maturity, verifies input value is not below output value, and computes fees.[^ref-btc-core-tx-verify]

For analysts, UTXO data is fundamental evidence. It supports fee calculation, spend tracing, consolidation analysis, change heuristics, age analysis, and supply-state analysis. But ownership, identity, intent, and wallet control remain interpretations unless supported by additional evidence.

---

## 4. Original Design

The Bitcoin whitepaper introduces coins as a chain of digital signatures. Each owner transfers value by signing a hash of the previous transaction and the next owner's public key. This creates a transaction chain, but the paper also notes that a payee needs a way to verify that previous owners did not double spend the coin.[^ref-btc-wp]

The term "UTXO" is not central in the whitepaper prose, but the model follows from Sections 2 and 9:

- transactions reference previous transaction outputs;
- value can be combined from multiple inputs;
- value can be split into multiple outputs;
- unspent outputs represent spendable state.[^ref-btc-wp]

Modern Bitcoin documentation and implementation use UTXO language to describe this active spendable set.

---

## 5. Protocol Structure

### UTXO Definition

A UTXO is:

```text
a transaction output
that exists in the active chain
and has not yet been spent
```

It is identified by an outpoint:

```text
outpoint = transaction id + output index
```

Bitcoin Core represents this with `COutPoint`, while transaction inputs use `CTxIn` to reference previous outpoints and transaction outputs use `CTxOut` to hold value and locking script data.[^ref-btc-core-transaction]

### Transaction State Transition

A non-coinbase transaction performs a state transition:

```text
before:
    UTXO set contains old outputs A, B, C

transaction:
    spends A and B
    creates D and E

after:
    A and B removed
    C remains
    D and E added
```

This is why Bitcoin state is not a list of account balances. It is a set of spendable outputs.

### Coinbase Exception

Coinbase transactions create new outputs without spending previous outputs. They are limited by subsidy plus fees and subject to maturity rules before spending. Bitcoin Core's transaction and validation paths treat coinbase behavior specially.[^ref-btc-core-tx-check][^ref-btc-core-tx-verify]

---

## 6. Technical Mechanics

### Spending an Output

To spend a UTXO, a transaction input must:

1. Reference the previous output by outpoint.
2. Provide unlocking data or witness data as required by the previous output's script.
3. Pass context-free transaction checks.
4. Pass UTXO availability and value checks.
5. Pass script validation under applicable flags.

This document focuses on UTXO availability and value accounting. Script validation is covered in later transaction and script documents.

### Fee Accounting

Fees are derived from UTXO values:

```text
fee = sum(input UTXO values) - sum(new output values)
```

Bitcoin Core's `CheckTxInputs` computes `nValueIn`, compares it with output value, rejects if input value is below output value, and returns the fee as the difference.[^ref-btc-core-tx-verify]

### Double-Spend Prevention

Within one active chain, an output can be spent once. If a transaction spends an outpoint, that output is removed from the UTXO set. A later transaction trying to spend the same outpoint will fail because the coin is no longer available in the view.

This is a local validation rule over chainstate, not a global account-locking service.

### Reorganization Behavior

Reorgs matter because the UTXO set is tied to the active chain. When a block is disconnected, its effects must be reversed:

- outputs created by disconnected transactions are removed;
- outputs spent by disconnected transactions are restored from undo data;
- the chainstate reflects the new active tip.

Bitcoin developer documentation describes the block chain as the ordered transaction history that full nodes validate, while Bitcoin Core exposes `DisconnectBlock` and `ConnectBlock` as block disconnection and connection operations on a `CCoinsViewCache` in validation interfaces.[^ref-btc-dev-blockchain][^ref-btc-core-validation-h]

---

## 7. Mathematical or Economic Model

### Balance as Derived State

In a wallet context:

```text
wallet balance = sum(UTXO values controlled by wallet keys/scripts)
```

This is a wallet-level calculation. It is not a consensus field.

### Transaction Validity Accounting

For a non-coinbase transaction:

```text
I = sum(input UTXO values)
O = sum(output values)
F = I - O
```

Consensus validity requires:

```text
I >= O
F >= 0
```

Amounts also need to be in range. Bitcoin Core defines `CAmount`, `COIN = 100000000`, `MAX_MONEY = 21000000 * COIN`, and `MoneyRange` for amount checks.[^ref-btc-core-amount]

### UTXO Fragmentation

A wallet can hold many small UTXOs:

```text
100 outputs * 0.001 BTC = 0.100 BTC wallet-controlled value
```

Spending many small UTXOs can increase transaction size and fee burden. Consolidating UTXOs can reduce future input count but can also reduce privacy by linking inputs together.

This is an economic and privacy tradeoff, not a consensus rule.

---

## 8. Security Assumptions

### What the UTXO Model Assumes

The UTXO model depends on:

1. Transaction IDs and output indexes uniquely identify previous outputs.
2. Full nodes maintain a correct active-chain UTXO set.
3. Inputs cannot spend outputs absent from the UTXO set.
4. Scripts authorize spending of referenced outputs.
5. Value cannot be created by ordinary transactions because inputs must cover outputs.
6. Reorg handling correctly reverses and reapplies UTXO state changes.

### What It Does Not Prove

UTXO validation does not prove:

- real-world ownership;
- transaction intent;
- wallet identity;
- exchange customer identity;
- whether an output is change;
- whether multiple inputs belong to one person;
- whether a transaction is economically meaningful beyond its state transition.

Those are analytical inferences.

---

## 9. Bitcoin Core Implementation

### Transaction Primitives

Bitcoin Core's `src/primitives/transaction.h` defines:

| Structure | Role |
|---|---|
| `COutPoint` | Identifies a previous output by transaction hash and output index |
| `CTxIn` | Transaction input referencing a previous outpoint |
| `CTxOut` | Transaction output holding value and locking script |
| `CTransaction` | Immutable transaction object containing `vin` and `vout` |

These structures are the basis for representing UTXO spends and creations.[^ref-btc-core-transaction]

### Coin and Coins Views

Bitcoin Core's `coins.h` defines `Coin` and the `CCoinsView` hierarchy. `CCoinsView` is documented as a view on the open transaction output dataset and exposes methods including `GetCoin`, `PeekCoin`, `HaveCoin`, `GetBestBlock`, and `BatchWrite`.[^ref-btc-core-coins]

The `CoinsViews` helper in `validation.h` constructs a layered view hierarchy. Doxygen describes the lowest level as a LevelDB database on disk and the top cache as holding as many coins in memory as permitted by cache settings.[^ref-btc-core-validation-h]

### `CheckTransaction`

`CheckTransaction` performs context-independent transaction checks. It rejects empty input or output vectors, invalid output values, duplicate inputs, non-coinbase null previous outpoints, and invalid coinbase script-size ranges.[^ref-btc-core-tx-check]

These checks do not require knowing whether referenced outputs are unspent.

### `CheckTxInputs`

`CheckTxInputs` performs UTXO-context checks. It verifies that inputs exist in the view, enforces coinbase maturity, checks input and output value consistency, and computes transaction fees.[^ref-btc-core-tx-verify]

This is the core validation boundary where the UTXO set becomes necessary.

### `ConnectBlock`

`ConnectBlock` applies valid block transactions to the UTXO view during block connection. It works with the chainstate's coins view and is part of the active-chain validation path.[^ref-btc-core-validation]

The exact internal organization can change across Bitcoin Core versions, but consensus-compatible implementations must produce the same valid state transition results.

---

## 10. On-Chain Implications

### Observable Facts

Analysts can directly observe:

- transaction outputs;
- output values;
- output scripts;
- input outpoints;
- whether an output has later been spent in the active chain;
- UTXO age by block height or time;
- input count and output count;
- consolidation and fan-out patterns.

### Derived Metrics

Common UTXO-derived metrics include:

| Metric | Description |
|---|---|
| UTXO count | Number of currently unspent outputs |
| Realized output age | Time since an output was created |
| Coin days destroyed | Value multiplied by age when spent |
| Consolidation ratio | Many inputs to fewer outputs |
| Fan-out ratio | Fewer inputs to many outputs |
| Fee rate context | Fee relative to transaction weight and input/output structure |

These metrics require careful methodology. Different data providers may define filters, entity adjustments, and dust handling differently.

### Inference Limits

UTXO data alone does not prove:

- who owns an output;
- whether an output is change;
- whether a spend is a sale;
- whether consolidation is by one user or a custodian;
- whether an output is lost;
- whether long dormancy means conviction.

These are interpretations and should be labeled accordingly.

---

## 11. Institutional Thinking

### Why UTXO Literacy Matters

Institutional Bitcoin research depends on UTXO literacy because UTXOs underlie:

- fee calculation;
- supply state;
- transaction tracing;
- wallet operations;
- custody controls;
- proof-of-reserves design;
- exchange flow interpretation;
- dormant supply metrics;
- realized-cap style metrics;
- privacy and clustering analysis.

### Controls for Research Workflows

Research systems should:

- compute fees from prior output values, not from visible outputs alone;
- distinguish unspent outputs from addresses;
- record the chain tip used for UTXO state;
- handle reorgs explicitly;
- separate consensus validity from mempool policy;
- label change and clustering as heuristic;
- avoid treating wallet balance as an on-chain field;
- verify high-stakes analyses against a full node or reproducible dataset.

### Custody and Operations

For custodians and institutions, UTXO management affects:

- withdrawal batching;
- fee exposure;
- consolidation timing;
- privacy leakage;
- signing complexity;
- dust management;
- proof-of-reserves output selection;
- recovery and accounting workflows.

UTXO management is therefore both a protocol concern and an operational discipline.

---

## 12. Common Misinterpretations

### Misinterpretation 1: Bitcoin stores account balances.

Incorrect. Bitcoin consensus tracks spendable outputs. Wallet balances are derived from controlled UTXOs.

### Misinterpretation 2: An address is a UTXO.

Incorrect. An address or script can receive many outputs. Each output is distinct and is identified by outpoint.

### Misinterpretation 3: A transaction spends an address.

Incorrect. A transaction input spends a previous output. Addresses are user-facing encodings or abstractions.

### Misinterpretation 4: Change outputs are consensus-labeled.

Incorrect. Change is a wallet behavior inferred by analysts. It is not marked by consensus.

### Misinterpretation 5: UTXO age proves investor intent.

Incorrect. Dormancy is observable. Intent is an interpretation.

### Misinterpretation 6: A reorg only changes headers.

Incorrect. If the active chain changes, the UTXO state must reflect disconnected and newly connected blocks.

---

## 13. Research Questions

1. How has Bitcoin's UTXO count changed across fee regimes?
2. How do custodians manage UTXO consolidation under high fees?
3. Which UTXO age metrics are robust to exchange and custodian behavior?
4. How should dust outputs be handled in institutional datasets?
5. How do script-type changes affect UTXO analysis?
6. How reliable are change heuristics across modern wallet types?
7. How should reorgs be represented in UTXO-derived metrics?
8. How does UTXO fragmentation affect transaction fee exposure?
9. What UTXO patterns distinguish exchanges, miners, custodians, and individual wallets?
10. How can full-node-derived UTXO state be reconciled with third-party analytics data?

---

## 14. Practical Exercises

1. Given three UTXOs of `0.10 BTC`, `0.25 BTC`, and `0.40 BTC`, construct a transaction paying `0.50 BTC` with a `0.002 BTC` fee. Compute change.
2. Explain why an address with five received outputs does not have one on-chain balance object.
3. Given a transaction input, identify the previous outpoint it spends.
4. Explain why a transaction with output value greater than input value is invalid unless it is a permitted coinbase claim.
5. Describe how a reorg can restore a previously spent UTXO.
6. Classify "this output is change" as fact, interpretation, heuristic, or unknown.

---

## 15. Evidence Classification

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Transaction chain, combining/splitting value, and double-spend problem | A |
| REF-BTC-DEV-TRANSACTIONS-001 | Official developer documentation | Transaction inputs, outputs, outpoints, and coinbase exception | A |
| REF-BTC-DEV-BLOCKCHAIN-001 | Official developer documentation | Validation, forks, and chain behavior context | A |
| REF-BTC-CORE-TRANSACTION-001 | Primary implementation source | `COutPoint`, `CTxIn`, `CTxOut`, `CTransaction` | A |
| REF-BTC-CORE-AMOUNT-001 | Primary implementation source | `CAmount`, `COIN`, `MAX_MONEY`, `MoneyRange` | A |
| REF-BTC-CORE-COINS-001 | Primary implementation source | `Coin`, `CCoinsView`, `CCoinsViewCache`, UTXO set access | A |
| REF-BTC-CORE-TX-CHECK-001 | Primary implementation source | Context-free transaction checks | A |
| REF-BTC-CORE-TX-VERIFY-001 | Primary implementation source | UTXO-dependent input validation and fee calculation | A |
| REF-BTC-CORE-VALIDATION-001 | Primary implementation source | `ConnectBlock` and block connection state transitions | A |
| REF-BTC-CORE-VALIDATION-H-001 | Primary implementation source | `ConnectBlock`, `DisconnectBlock`, and `CoinsViews` declarations | A |

### Claim Ledger

| Claim ID | Claim | Classification | Evidence | Source |
|---|---|---|---|---|
| C001 | Bitcoin transactions consume previous outputs and create new outputs. | FACT | A | REF-BTC-WP-001; REF-BTC-DEV-TRANSACTIONS-001 |
| C002 | A UTXO is a transaction output that remains unspent in the active chain. | FACT | A | REF-BTC-DEV-TRANSACTIONS-001; REF-BTC-CORE-COINS-001 |
| C003 | Bitcoin Core represents previous-output references with `COutPoint`. | FACT | A | REF-BTC-CORE-TRANSACTION-001 |
| C004 | `CCoinsView` is a view on the open transaction output dataset. | FACT | A | REF-BTC-CORE-COINS-001 |
| C005 | `CheckTransaction` performs context-free checks such as rejecting duplicate inputs and invalid output values. | FACT | A | REF-BTC-CORE-TX-CHECK-001 |
| C006 | `CheckTxInputs` checks UTXO availability and computes fees from input value minus output value. | FACT | A | REF-BTC-CORE-TX-VERIFY-001 |
| C007 | `ConnectBlock` applies block transaction effects to a coins view. | FACT | A | REF-BTC-CORE-VALIDATION-001; REF-BTC-CORE-VALIDATION-H-001 |
| C008 | Wallet balance is derived from controlled UTXOs rather than stored as a consensus account balance. | INTERPRETATION | B | REF-BTC-DEV-TRANSACTIONS-001; REF-BTC-CORE-COINS-001 |
| C009 | Change detection and ownership clustering are heuristic analyses, not consensus facts. | INTERPRETATION | B | REF-BTC-WP-001 |
| C010 | UTXO management affects institutional fee, privacy, and custody operations. | INTERPRETATION | B | REF-BTC-DEV-TRANSACTIONS-001 |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| HEURISTIC | Practical analysis rule requiring caveats |
| UNKNOWN | Evidence is insufficient |

---

## 16. Knowledge Graph

```text
BITCOIN-014 UTXO Model
|
+-- builds_on: BITCOIN-010 Combining and Splitting Value
+-- builds_on: POW-009 Coinbase Transaction
|
+-- UTXO
|   +-- identified_by: outpoint = txid + vout
|   +-- contains: value + locking script
|   +-- state: unspent or spent
|
+-- transaction
|   +-- consumes: existing UTXOs
|   +-- creates: new outputs
|   +-- fee: input values - output values
|
+-- Bitcoin Core
|   +-- transaction primitives
|   +-- Coin
|   +-- CCoinsView
|   +-- CheckTransaction
|   +-- CheckTxInputs
|   +-- ConnectBlock
|
+-- analysis
    +-- facts: outpoints, values, spend status
    +-- heuristics: change, ownership, clustering
    +-- operations: consolidation, batching, custody
```

---

## 17. References

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Sections 2 and 9, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-transactions]: Bitcoin Developer Documentation, "Transactions," inputs, outputs, outpoints, raw transaction format, and coinbase input exception, https://developer.bitcoin.org/reference/transactions.html, accessed 2026-08-04.

[^ref-btc-dev-blockchain]: Bitcoin Developer Documentation, "Block Chain," transaction data and chain validation context, https://developer.bitcoin.org/devguide/block_chain.html, accessed 2026-08-04.

[^ref-btc-core-transaction]: Bitcoin Core Contributors, `src/primitives/transaction.h`, `COutPoint`, `CTxIn`, `CTxOut`, and `CTransaction`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/primitives_2transaction_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-amount]: Bitcoin Core Contributors, `src/consensus/amount.h`, `CAmount`, `COIN`, `MAX_MONEY`, and `MoneyRange`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/amount_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-coins]: Bitcoin Core Contributors, `src/coins.h` and `src/coins.cpp`, `Coin`, `CCoinsView`, `CCoinsViewCache`, and UTXO set access methods, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/coins_8h_source.html and https://doxygen.bitcoincore.org/coins_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-tx-check]: Bitcoin Core Contributors, `src/consensus/tx_check.cpp`, `CheckTransaction`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/tx__check_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-tx-verify]: Bitcoin Core Contributors, `src/consensus/tx_verify.cpp`, `CheckTxInputs`, input availability, coinbase maturity, input/output value checks, and fee calculation, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/tx__verify_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-validation]: Bitcoin Core Contributors, `src/validation.cpp`, `ConnectBlock`, `UpdateCoins`, and chainstate block connection logic, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/validation_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-validation-h]: Bitcoin Core Contributors, `src/validation.h`, `ConnectBlock`, `DisconnectBlock`, and `CoinsViews`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/validation_8h_source.html, accessed 2026-08-04.

---

## 18. Cross References

### Parent

- Bitcoin Internals

### Previous

- BITCOIN-013 — Whitepaper Section 12 — Conclusion

### Next

- BITCOIN-015 — Transactions in Depth

### Related

- BITCOIN-003 — Whitepaper Section 2 — Transactions
- BITCOIN-007 — Whitepaper Section 6 — Incentive
- BITCOIN-010 — Whitepaper Section 9 — Combining and Splitting Value
- BITCOIN-015 — Transactions in Depth
- BITCOIN-016 — Script & ScriptPubKey
- BITCOIN-017 — Mempool
- BITCOIN-023 — Chain Reorganization
- POW-009 — Coinbase Transaction and Mining Commitments

---

## Review Status

### Technical Review

Passed.

- UTXO, transaction output, outpoint, wallet balance, and address were separated.
- Context-free transaction checks were separated from UTXO-dependent validation.
- `COutPoint`, `CTxIn`, `CTxOut`, `CTransaction`, `Coin`, `CCoinsView`, `CCoinsViewCache`, `CheckTransaction`, `CheckTxInputs`, `ConnectBlock`, and `DisconnectBlock` references were checked against Bitcoin Core source.
- Reorg effects on UTXO state were included without over-specifying implementation internals.

### Evidence Review

Passed.

- Whitepaper-derived transaction-chain claims cite the whitepaper.
- Transaction format and outpoint claims cite official Bitcoin Developer documentation.
- Current implementation claims cite Bitcoin Core Doxygen source references.
- Wallet balance, ownership, change, and clustering statements are labeled as interpretation or heuristic.

### Editorial Review

Passed.

- Markdown headings follow the project deep-dive structure.
- Metadata is complete.
- Tables and code fences are closed.
- Terminology is consistent: UTXO, outpoint, input, output, chainstate, coins view, wallet balance.

### Adversarial Review

Passed.

- The document does not describe Bitcoin as an account-balance ledger.
- It does not treat addresses as UTXOs.
- It does not treat change or ownership clustering as consensus facts.
- It distinguishes active-chain UTXO state from historical transaction data.

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
