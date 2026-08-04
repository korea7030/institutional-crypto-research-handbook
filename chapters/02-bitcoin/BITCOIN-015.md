---
knowledge_id: BITCOIN-015
title: Transactions in Depth
subtitle: Transaction Serialization, Inputs, Outputs, Identifiers, Locktime, Witness, Fees, and Validation Boundaries
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 105 min
estimated_study: 300 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Transactions
  - Validation
  - Serialization
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-003
  - BITCOIN-010
  - BITCOIN-014
  - POW-009
related_topics:
  - Transaction Inputs
  - Transaction Outputs
  - Transaction Identifiers
  - Segregated Witness
  - Locktime
  - Sequence Locks
  - Fees
  - Mempool Policy
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-TRANSACTIONS-001
  - REF-BIP-0068
  - REF-BIP-0125
  - REF-BIP-0141
  - REF-BIP-0143
  - REF-BIP-0341
  - REF-BTC-CORE-TRANSACTION-001
  - REF-BTC-CORE-TXID-001
  - REF-BTC-CORE-TX-CHECK-001
  - REF-BTC-CORE-TX-VERIFY-001
  - REF-BTC-CORE-INTERPRETER-001
  - REF-BTC-CORE-RBF-001
tags:
  - bitcoin
  - internals
  - transactions
  - txid
  - wtxid
  - locktime
  - sequence
  - segwit
  - fee
---

# Transactions in Depth
> Bitcoin Internals  
> Research Unit: BITCOIN-015

---

## Research Brief

```yaml
knowledge_id: BITCOIN-015
title: Transactions in Depth
research_question: >
  What exactly is a Bitcoin transaction, how are its fields serialized and
  identified, how do inputs, outputs, locktime, sequence numbers, witness
  data, fees, and validation boundaries interact, and what should on-chain
  analysts avoid over-inferring from transaction data?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-003
  - BITCOIN-010
  - BITCOIN-014
parent: Bitcoin Internals
previous: BITCOIN-014
next: BITCOIN-016
related_topics:
  - UTXO Model
  - Script
  - Transaction Fees
  - Mempool
  - Segregated Witness
  - Taproot
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Protocol Structure
  - Serialization
  - Technical Mechanics
  - Mathematical or Economic Model
  - Security Assumptions
  - Bitcoin Core Implementation
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Complete Script opcode semantics
  - Wallet coin selection algorithms
  - Full mempool package relay behavior
  - Signature cryptography proofs
  - Non-Bitcoin transaction models except brief contrast
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Describe the fields of a Bitcoin transaction.
- Distinguish transaction inputs from transaction outputs.
- Explain how an input references a previous output through an outpoint.
- Explain the difference between `txid` and `wtxid`.
- Explain how Segregated Witness changed transaction serialization without changing the legacy `txid` definition.
- Explain how fees are derived from input and output values.
- Explain the role of `nVersion`, `nLockTime`, and input `nSequence`.
- Separate consensus validity from relay policy and wallet behavior.
- Identify the Bitcoin Core source areas that represent and validate transactions.
- Explain what transaction data can and cannot prove for on-chain analysis.

---

## 2. Key Questions

1. What is a Bitcoin transaction at the data-structure level?
2. Which fields are serialized into a legacy transaction?
3. Which additional fields exist for witness serialization?
4. What does an input actually spend?
5. What does an output actually lock?
6. Why is an address not a transaction field in consensus serialization?
7. How are `txid` and `wtxid` computed?
8. How do locktime and sequence interact with transaction finality?
9. How are fees computed?
10. Which checks are context-free?
11. Which checks require UTXO context?
12. Which transaction properties are policy rather than consensus?

---

## 3. Executive Summary

A Bitcoin transaction is a serialized state-transition instruction. It consumes existing UTXOs through inputs and creates new outputs with amounts and locking scripts. The transaction itself does not contain account balances. It contains references to previous outputs and new output definitions.[^ref-btc-dev-transactions]

The basic legacy transaction structure is:

```text
nVersion
txins
txouts
nLockTime
```

Segregated Witness adds marker, flag, and witness fields to the serialization used for witness-bearing transactions. BIP141 keeps the legacy `txid` definition as the double SHA256 of the non-witness serialization, while defining `wtxid` as the double SHA256 of the witness serialization.[^ref-bip-0141]

Transaction validation has layers. `CheckTransaction` performs context-independent checks such as non-empty inputs and outputs, output value range checks, duplicate input detection, null previous-outpoint rules, and coinbase script-size limits.[^ref-btc-core-tx-check] `CheckTxInputs` performs UTXO-context checks, including input availability, coinbase maturity, input/output value consistency, and fee calculation.[^ref-btc-core-tx-verify] Script validation then verifies whether each input satisfies the previous output's spending condition.[^ref-btc-core-interpreter]

Fees are not an explicit field. For non-coinbase transactions:

```text
fee = sum(input UTXO values) - sum(output values)
```

For analysts, transactions provide strong evidence about outpoints, values, script types, confirmation status, and graph topology. They do not by themselves prove real-world ownership, intent, customer identity, change status, or economic purpose.

---

## 4. Protocol Structure

### Transaction as State Transition

Bitcoin's transaction model follows the whitepaper's chain-of-signatures design: ownership transfer is represented by referencing earlier transaction data and authorizing the next transfer.[^ref-btc-wp]

At protocol level, a non-coinbase transaction says:

```text
spend these previous outputs
create these new outputs
optionally constrain finality by locktime and sequence
provide authorization material
```

The transaction is valid only if the referenced previous outputs exist, are unspent in the relevant view, carry sufficient value, and are spent with valid authorization.

### Core Fields

The standard transaction format documented by Bitcoin developer documentation includes version, input count, inputs, output count, outputs, and locktime.[^ref-btc-dev-transactions]

| Field | Meaning | Consensus relevance |
|---|---|---|
| `nVersion` | Transaction version | Enables version-gated semantics such as BIP68 sequence locks |
| Input count | CompactSize count of inputs | Must be non-empty for valid transactions |
| Inputs | Previous-output references plus spending data | Identify consumed UTXOs and provide authorization data |
| Output count | CompactSize count of outputs | Must be non-empty |
| Outputs | Values plus locking scripts | Define new spendable outputs |
| `nLockTime` | Earliest block height or time for finality | Used by finality checks when sequence permits |

### Inputs

A non-coinbase input contains:

| Component | Meaning |
|---|---|
| Previous outpoint | `txid` plus output index of the UTXO being spent |
| ScriptSig | Legacy unlocking data or redeem data |
| Sequence | Finality, relative-locktime, and policy signaling field |
| Witness | SegWit spending data, serialized outside legacy `txid` data |

The previous outpoint is the crucial link. It does not name an address. It names a specific output of a specific prior transaction.

### Outputs

An output contains:

| Component | Meaning |
|---|---|
| Value | Number of satoshis locked by the output |
| ScriptPubKey | Locking script defining spending conditions |

Bitcoin developer documentation describes a transaction output as spending a number of satoshis under conditions provided by the pubkey script.[^ref-btc-dev-transactions]

An address is usually a wallet/user-interface encoding of a destination or script template. It is not itself a consensus field in the transaction serialization.

### Coinbase Transactions

A coinbase transaction is the first transaction in a block. Its input has no previous outpoint and uses a null hash plus `0xffffffff` index. The coinbase creates block subsidy plus collected transaction fees, subject to consensus limits and maturity rules.[^ref-btc-dev-transactions]

Coinbase transactions must be analyzed separately from ordinary transactions because they create newly spendable value after maturity instead of consuming prior UTXOs.

---

## 5. Serialization and Identifiers

### Legacy Serialization

Legacy transaction serialization commits to:

```text
[nVersion][txins][txouts][nLockTime]
```

The `txid` is the transaction identifier derived from this serialization. BIP141 explicitly preserves this legacy `txid` definition when introducing witness data.[^ref-bip-0141]

### Witness Serialization

For witness transactions, BIP141 defines a serialization form:

```text
[nVersion][marker][flag][txins][txouts][witness][nLockTime]
```

The marker is `0x00`, the flag is nonzero with `0x01` currently used, and each input has an associated witness field. BIP141 states that witness data is not script.[^ref-bip-0141]

### `txid` and `wtxid`

The distinction is:

| Identifier | Hashes witness data? | Main use |
|---|---:|---|
| `txid` | No | Legacy transaction identity and outpoint references |
| `wtxid` | Yes | Witness-aware relay, block witness commitment, and malleability-resistant tracking |

Bitcoin Core's `transaction_identifier` template represents the canonical transaction identifier types `txid` and `wtxid`.[^ref-btc-core-txid]

For legacy non-witness transactions, `txid` and `wtxid` are effectively the same digest because there is no witness serialization difference. For witness transactions, changing witness data changes `wtxid` but not `txid`, assuming the non-witness serialization remains unchanged.

### Transaction Malleability Boundary

Before SegWit, signature data was part of the data hashed into the `txid`. This meant some changes to valid signature encodings or scriptSig data could alter the transaction identifier without changing the intended spend. SegWit moved witness data outside the legacy `txid` calculation, reducing this class of malleability for SegWit spends.

This does not mean all transaction semantics are malleability-free in every possible sense. It means the identifier used for outpoint references no longer commits to witness data for SegWit transactions.

---

## 6. Technical Mechanics

### Transaction Construction

Wallet construction usually follows this flow:

```text
select UTXOs
choose recipients and amounts
estimate fee
create outputs, including change if needed
set version, sequence, and locktime fields
produce signatures or witness data
broadcast or submit transaction
```

Most of these steps are wallet behavior. Consensus validation only evaluates the resulting transaction against protocol rules.

### Context-Free Checks

Context-free checks can be performed without knowing the current UTXO set. Bitcoin Core's `CheckTransaction` covers this category.[^ref-btc-core-tx-check]

Examples include:

- inputs must not be empty;
- outputs must not be empty;
- output values must not be negative;
- output values must remain in range;
- duplicate inputs are rejected;
- coinbase script size must satisfy consensus bounds;
- non-coinbase transactions may not use null previous outpoints.

These checks reject malformed transactions early but do not prove spendability.

### UTXO-Context Checks

UTXO-context checks require the current view of unspent outputs. Bitcoin Core's `CheckTxInputs` verifies input availability, coinbase maturity, value consistency, and fee calculation.[^ref-btc-core-tx-verify]

These checks answer questions such as:

- Does each referenced previous output exist and remain unspent?
- Is the input value sufficient to cover the output value?
- Is a coinbase output old enough to spend?
- What fee does the transaction pay?

### Script Checks

Script validation checks whether an input satisfies the previous output's locking conditions. Bitcoin Core's script interpreter implements script evaluation and verification paths such as `VerifyScript`.[^ref-btc-core-interpreter]

This Research Unit treats script as a validation boundary. Script language, standard script templates, witness programs, and Taproot script-path behavior are handled in BITCOIN-016 and later units.

### Finality Checks

Absolute finality uses `nLockTime`. A transaction can be constrained so it is not final until a specified block height or time. Bitcoin Core documents `IsFinalTx` as checking whether a transaction is final and can be included in a block at a specified height and time.[^ref-btc-core-tx-verify]

Relative finality uses input sequence numbers under BIP68. BIP68 gives consensus-enforced relative-locktime meaning to sequence numbers for version 2 or higher transactions when the disable flag is not set.[^ref-bip-0068]

These finality mechanisms are consensus relevant when their activation conditions apply.

### Replaceability Policy

Sequence numbers also interact with mempool policy. BIP125 defines opt-in full replace-by-fee signaling: a transaction signals replaceability if any input sequence number is less than `0xffffffff - 1`, and descendants can inherit replaceability while ancestors remain unconfirmed.[^ref-bip-0125]

This is relay and mempool policy, not a rule that determines whether a transaction can appear in a valid block. Bitcoin Core has RBF policy code such as `IsRBFOptIn` in `src/policy/rbf.cpp`.[^ref-btc-core-rbf]

---

## 7. Mathematical or Economic Model

### Fee Formula

For a non-coinbase transaction:

```text
I = sum(input UTXO values)
O = sum(output values)
F = I - O
```

Validity requires:

```text
I >= O
F >= 0
```

`F` is the transaction fee. It is not serialized as a field. It is implied by the difference between input values and output values.

### Fee Rate

Miners and mempool policy commonly evaluate transactions by fee rate:

```text
fee_rate = fee / virtual_size
```

SegWit introduced weight-based accounting. This made witness bytes discounted relative to non-witness bytes while preserving block validation limits under the SegWit rules.[^ref-bip-0141]

Fee rate is an economic and policy concept. A low-fee transaction can still be consensus-valid, but it may relay poorly or confirm slowly.

### Example

```text
inputs:
  0.03000000 BTC
  0.02000000 BTC

outputs:
  0.04400000 BTC recipient
  0.00590000 BTC change

fee:
  0.05000000 - 0.04990000 = 0.00010000 BTC
```

In satoshis:

```text
5,000,000 sats - 4,990,000 sats = 10,000 sats
```

The transaction does not say "fee = 10,000 sats." Nodes derive that value from the UTXOs spent.

### Output Ordering

Bitcoin consensus does not require a recipient output to appear before a change output. Output ordering is wallet behavior. Analysts who infer change must treat output position as at most one weak signal among others.

---

## 8. Signature Commitment and Sighash Scope

### Legacy Sighash

Signatures commit to a transaction digest determined by the signature hash type and script rules. Legacy signature hashing has historical complexity and performance issues.

### SegWit Version 0

BIP143 defines the transaction digest algorithm for version 0 witness programs. It was designed to reduce redundant hashing and to commit the input value being spent, which helps offline signers avoid needing the entire previous transaction from a trusted source.[^ref-bip-0143]

This matters operationally because a signer must know what it is authorizing. If the value committed by the signature is wrong, the signature will not validate for the actual spend.

### Taproot

BIP341 defines Taproot as SegWit version 1, with Schnorr signatures and a common signature message function for Taproot spends.[^ref-bip-0341]

For this unit, the key point is architectural: different output types can use different signature message algorithms while preserving the same high-level transaction model of inputs consuming outputs and outputs defining future spending conditions.

---

## 9. Security Assumptions

### What Transaction Validation Enforces

Transaction validation enforces:

- structural well-formedness;
- no duplicate inputs within a transaction;
- valid amount ranges;
- availability of spent UTXOs;
- no ordinary inflation from outputs exceeding inputs;
- coinbase maturity;
- applicable finality rules;
- script satisfaction for each spend.

### What It Does Not Enforce

Transaction validation does not enforce:

- that a payment corresponds to an invoice;
- that an address belongs to a named entity;
- that an output is change;
- that multiple inputs belong to one person;
- that a transaction is economically rational;
- that a zero-confirmation transaction will not be replaced;
- that local mempool acceptance means global network acceptance.

These are application, wallet, policy, or analytical questions.

### Attack and Failure Modes

Important transaction-level risks include:

| Risk | Description | Boundary |
|---|---|---|
| Double spend attempt | Reusing an already-spent outpoint | Consensus rejects in one active chain |
| Unconfirmed replacement | Higher-fee conflict replaces a mempool transaction | Policy and miner behavior |
| Malleability | Identifier or witness-related tracking changes | Depends on transaction type |
| Fee underpayment | Transaction is valid but economically unattractive | Relay/mining policy |
| Change misclassification | Analyst labels output ownership incorrectly | Heuristic failure |
| Signing wrong transaction | Signer commits to unexpected outputs or values | Wallet/signing security |

---

## 10. Bitcoin Core Implementation

### Transaction Primitives

Bitcoin Core's `src/primitives/transaction.h` defines the main transaction structures.[^ref-btc-core-transaction]

| Structure | Role |
|---|---|
| `COutPoint` | Previous transaction hash plus output index |
| `CTxIn` | Input referencing a previous outpoint and carrying script/sequence data |
| `CTxOut` | Output value plus scriptPubKey |
| `CTransaction` | Immutable transaction object with inputs, outputs, version, and locktime |
| `CMutableTransaction` | Mutable construction form used before final transaction creation |

Bitcoin Core's `CTransaction` is the basic transaction object broadcast on the network and contained in blocks.[^ref-btc-core-transaction]

### Transaction Identifiers

Bitcoin Core separates identifier types through `transaction_identifier`, which represents canonical `txid` and `wtxid` types.[^ref-btc-core-txid]

This distinction is important for code review because a function expecting a witness-aware identifier should not silently accept a legacy identifier when witness data matters.

### Consensus Transaction Checks

Bitcoin Core's consensus transaction checks are split across files:

| Source area | Function | Role |
|---|---|---|
| `src/consensus/tx_check.cpp` | `CheckTransaction` | Context-free transaction validity |
| `src/consensus/tx_verify.cpp` | `IsFinalTx` | Absolute finality |
| `src/consensus/tx_verify.cpp` | `SequenceLocks` | BIP68 relative finality |
| `src/consensus/tx_verify.cpp` | `CheckTxInputs` | UTXO-context validity and fee computation |
| `src/script/interpreter.cpp` | `VerifyScript` | Script satisfaction |

The exact file organization may change over time, but these references reflect Bitcoin Core Doxygen documentation accessed on 2026-08-04.

### Policy Code

RBF is implemented in Bitcoin Core policy code, not consensus code. `src/policy/rbf.cpp` includes `IsRBFOptIn`, which checks explicit and inherited BIP125 replaceability in mempool context.[^ref-btc-core-rbf]

An institutional analyst should avoid treating RBF signaling as an invalidity marker. It is a replaceability and settlement-risk signal for unconfirmed transactions.

---

## 11. Consensus, Policy, and Wallet Behavior

### Consensus

Consensus rules determine whether a transaction can be included in a valid block.

Examples:

- transaction structure must be valid;
- outputs must be in valid money range;
- inputs must reference spendable UTXOs;
- scripts must validate;
- finality rules must be satisfied;
- coinbase value and maturity rules must be respected.

### Policy

Policy rules determine whether a node relays or mines a transaction by default.

Examples:

- dust policy;
- standard script forms;
- minimum relay fee;
- mempool replacement rules;
- ancestor and descendant limits;
- package acceptance rules.

A non-standard transaction can be consensus-valid but not relayed by many nodes.

### Wallet Behavior

Wallet behavior determines how transactions are constructed.

Examples:

- coin selection;
- change address selection;
- output ordering;
- fee estimation;
- RBF signaling choice;
- address reuse policy.

Wallet behavior shapes on-chain patterns but is not a direct consensus rule.

---

## 12. On-Chain Implications

### Strong Evidence

Transaction data strongly supports claims about:

- input outpoints;
- output values;
- output scripts;
- transaction identifiers;
- witness presence;
- block inclusion;
- confirmation depth;
- fee paid after UTXO values are known;
- script template classification when templates are clear.

### Weak or Heuristic Evidence

Transaction data alone weakly supports claims about:

- change output identification;
- wallet ownership;
- entity clustering;
- exchange deposit or withdrawal labeling;
- user intent;
- economic category of a payment;
- whether a consolidation was operational, tactical, or accidental.

These claims require caveats and ideally external corroboration.

### Analyst Workflow

A rigorous transaction analysis workflow is:

1. Parse transaction fields.
2. Identify inputs and previous outputs.
3. Confirm whether the transaction is coinbase or non-coinbase.
4. Compute input value, output value, and fee.
5. Classify output scripts.
6. Check confirmation status and reorg risk.
7. Identify policy signals such as opt-in RBF if unconfirmed.
8. Apply heuristics only after factual state is established.
9. Label confidence and counterexamples.

---

## 13. Institutional Thinking

### Custody

Custodians care about transaction structure because every withdrawal, consolidation, sweep, and cold-storage movement is a UTXO state transition. Controls should verify:

- destination outputs;
- change handling;
- fee reasonableness;
- signing scope;
- expected script type;
- input provenance;
- replacement policy if unconfirmed.

### Risk

Risk teams should distinguish final settlement from mempool visibility. An unconfirmed transaction is not equivalent to settled payment. RBF signaling, low fee rate, ancestor dependencies, and mempool divergence can all affect short-term settlement expectations without changing long-run consensus validity.

### Accounting

Accounting systems should not model Bitcoin as a single account balance at the protocol layer. A balance view is derived from UTXOs controlled by the institution and must account for pending spends, confirmations, change outputs, and reorg risk.

### Compliance Analytics

Compliance analysis should avoid overclaiming. A transaction graph can show flow of outputs, but the mapping from outputs to persons or institutions is probabilistic unless grounded in independent attribution data.

---

## 14. Common Misinterpretations

### "A Transaction Sends From an Address"

Not at consensus level. A transaction spends previous outputs. Addresses are user-facing encodings or descriptors of script destinations, not authoritative sender fields.

### "The Fee Is Written in the Transaction"

No. The fee is inferred from input values and output values.

### "Every Input Belongs to the Same Owner"

Often assumed in simple clustering, but not guaranteed. CoinJoin and collaborative transactions intentionally break this assumption.

### "RBF Means Fraud"

No. RBF is a policy mechanism for replacing unconfirmed transactions. It can be used for fee bumping, correction, or attempted double spend. Interpretation depends on context.

### "SegWit Removed All Malleability"

No. SegWit changes the identifier boundary for witness data and fixes important practical malleability issues for SegWit spends. Analysts still need to understand transaction type and signature scope.

### "A Valid Transaction Must Relay Everywhere"

No. Consensus validity and relay policy are different. A transaction can be consensus-valid but not accepted into many nodes' mempools.

---

## 15. Research Questions

1. How does transaction structure determine the boundaries of what can be proven from chain data?
2. How should an analyst classify confidence when identifying change outputs?
3. What operational signals distinguish consolidation from payment batching?
4. How do `txid` and `wtxid` affect transaction tracking systems?
5. How should institutions handle unconfirmed transactions that signal replaceability?
6. How do signature hash modes change the meaning of "the transaction was signed"?
7. What controls should hardware signing workflows use to prevent wrong-output or wrong-fee signing?
8. How do Taproot spends change script visibility for analysts?

---

## 16. Practical Exercises

### Exercise 1: Parse a Transaction

Choose a confirmed non-coinbase transaction and identify:

- version;
- input count;
- each previous outpoint;
- output count;
- each output value;
- each output script type;
- locktime;
- whether witness data is present.

### Exercise 2: Compute Fee

For the same transaction:

1. Retrieve the previous outputs spent by every input.
2. Sum input values.
3. Sum output values.
4. Compute the fee.
5. Compare the result to a block explorer.

### Exercise 3: Identify Validation Boundaries

Classify each statement:

| Statement | Consensus | Policy | Wallet behavior | Analytics heuristic |
|---|---:|---:|---:|---:|
| Output value must not be negative | Yes | No | No | No |
| Transaction signals opt-in RBF | No | Yes | Maybe | No |
| Second output is change | No | No | Maybe | Yes |
| Input spends a previous outpoint | Yes | No | No | No |
| Fee rate is attractive to miners | No | Maybe | No | Interpretation |

### Exercise 4: Compare `txid` and `wtxid`

Inspect a SegWit transaction and record:

- legacy `txid`;
- witness transaction id if available;
- whether witness data appears in the serialized form;
- why outpoints still refer to `txid`.

---

## 17. Evidence Classification

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Transaction-chain model and double-spend framing | A |
| REF-BTC-DEV-TRANSACTIONS-001 | Official developer documentation | Transaction fields, inputs, outputs, outpoints, coinbase input | A |
| REF-BIP-0068 | Consensus BIP | Relative locktime semantics for sequence numbers | A |
| REF-BIP-0125 | Policy BIP | Opt-in full RBF signaling and replacement policy | B |
| REF-BIP-0141 | Consensus BIP | SegWit serialization, `txid`, `wtxid`, witness data | A |
| REF-BIP-0143 | Consensus BIP | Version 0 witness signature digest algorithm | A |
| REF-BIP-0341 | Consensus BIP | Taproot and SegWit version 1 signature rules | A |
| REF-BTC-CORE-TRANSACTION-001 | Primary implementation source | `COutPoint`, `CTxIn`, `CTxOut`, `CTransaction`, `CMutableTransaction` | A |
| REF-BTC-CORE-TXID-001 | Primary implementation source | `transaction_identifier`, `txid`, `wtxid` typing | A |
| REF-BTC-CORE-TX-CHECK-001 | Primary implementation source | Context-free transaction checks | A |
| REF-BTC-CORE-TX-VERIFY-001 | Primary implementation source | Finality, sequence locks, input validation, fee calculation | A |
| REF-BTC-CORE-INTERPRETER-001 | Primary implementation source | Script verification boundary | A |
| REF-BTC-CORE-RBF-001 | Primary implementation source | RBF policy implementation | A |

### Claim Ledger

| Claim | Classification | Evidence |
|---|---|---|
| A transaction consumes previous outputs and creates new outputs | FACT | Whitepaper, developer documentation |
| Outpoints identify previous outputs by transaction id and index | FACT | Developer documentation, Bitcoin Core transaction primitives |
| `txid` excludes witness data under SegWit | FACT | BIP141 |
| `wtxid` includes witness data under SegWit | FACT | BIP141 |
| Fees are implied by input minus output value | FACT | Developer documentation, `CheckTxInputs` |
| `CheckTransaction` is context-free | FACT | Bitcoin Core `tx_check.cpp` |
| `CheckTxInputs` requires UTXO context | FACT | Bitcoin Core `tx_verify.cpp` |
| BIP68 gives sequence numbers relative-locktime meaning under defined conditions | FACT | BIP68 |
| BIP125 replaceability is mempool policy | FACT | BIP125, Bitcoin Core RBF policy source |
| Address labels and ownership clusters are not consensus facts | INTERPRETATION | Derived from transaction structure and UTXO model |
| Output ordering can identify change reliably by itself | COUNTERCLAIM | Rejected; output ordering is wallet behavior and weak heuristic |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| HEURISTIC | Practical rule with known counterexamples |
| POLICY | Relay, mempool, or mining convention rather than consensus |
| UNKNOWN | Evidence is insufficient |

---

## 18. Knowledge Graph

```text
BITCOIN-015 Transactions in Depth
|
+-- builds_on: BITCOIN-014 UTXO Model
+-- builds_on: BITCOIN-010 Combining and Splitting Value
|
+-- transaction
|   +-- fields: version, inputs, outputs, locktime
|   +-- segwit_fields: marker, flag, witness
|
+-- input
|   +-- references: outpoint
|   +-- carries: scriptSig, sequence, witness
|
+-- output
|   +-- contains: value
|   +-- locks_with: scriptPubKey
|
+-- identifiers
|   +-- txid: non-witness serialization hash
|   +-- wtxid: witness serialization hash
|
+-- validation
|   +-- context_free: CheckTransaction
|   +-- utxo_context: CheckTxInputs
|   +-- script_context: VerifyScript
|   +-- finality: IsFinalTx, SequenceLocks
|
+-- policy
|   +-- RBF: BIP125
|   +-- relay: standardness and fee policy
|
+-- analysis
    +-- facts: outpoints, outputs, fee, ids
    +-- heuristics: ownership, change, clustering
```

---

## 19. References

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Sections 2 and 9, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-transactions]: Bitcoin Developer Documentation, "Transactions," transaction format, inputs, outputs, outpoints, locktime, and coinbase input, https://developer.bitcoin.org/reference/transactions.html, accessed 2026-08-04.

[^ref-bip-0068]: Mark Friedenbach, BtcDrak, Nicolas Dorier, and kinoshitajona, "BIP 68: Relative lock-time using consensus-enforced sequence numbers," 2015-05-28, https://bips.dev/68/, accessed 2026-08-04.

[^ref-bip-0125]: David A. Harding and Peter Todd, "BIP 125: Opt-in Full Replace-by-Fee Signaling," 2015-12-04, https://bips.dev/125/, accessed 2026-08-04.

[^ref-bip-0141]: Eric Lombrozo, Johnson Lau, and Pieter Wuille, "BIP 141: Segregated Witness (Consensus layer)," 2015-12-21, https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki, accessed 2026-08-04.

[^ref-bip-0143]: Johnson Lau and Pieter Wuille, "BIP 143: Transaction Signature Verification for Version 0 Witness Program," 2016-01-03, https://bips.dev/143/, accessed 2026-08-04.

[^ref-bip-0341]: Pieter Wuille, Jonas Nick, and Anthony Towns, "BIP 341: Taproot: SegWit version 1 spending rules," 2020-01-19, https://bips.xyz/341, accessed 2026-08-04.

[^ref-btc-core-transaction]: Bitcoin Core Contributors, `src/primitives/transaction.h`, `COutPoint`, `CTxIn`, `CTxOut`, `CTransaction`, and `CMutableTransaction`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/primitives_2transaction_8h.html, accessed 2026-08-04.

[^ref-btc-core-txid]: Bitcoin Core Contributors, `src/primitives/transaction_identifier.h`, `transaction_identifier` template for `txid` and `wtxid`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/classtransaction__identifier.html, accessed 2026-08-04.

[^ref-btc-core-tx-check]: Bitcoin Core Contributors, `src/consensus/tx_check.cpp`, `CheckTransaction`, Bitcoin Core Doxygen 31.99.0 source documentation, https://doxygen.bitcoincore.org/tx__check_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-tx-verify]: Bitcoin Core Contributors, `src/consensus/tx_verify.h` and `src/consensus/tx_verify.cpp`, `IsFinalTx`, `SequenceLocks`, `GetTransactionSigOpCost`, and `CheckTxInputs`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/tx__verify_8h.html, accessed 2026-08-04.

[^ref-btc-core-interpreter]: Bitcoin Core Contributors, `src/script/interpreter.cpp`, script evaluation and `VerifyScript`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/interpreter_8cpp.html, accessed 2026-08-04.

[^ref-btc-core-rbf]: Bitcoin Core Contributors, `src/policy/rbf.cpp`, `IsRBFOptIn` and RBF mempool policy logic, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/policy_2rbf_8cpp_source.html, accessed 2026-08-04.

---

## 20. Cross References

### Parent

- Bitcoin Internals

### Previous

- BITCOIN-014 — UTXO Model

### Next

- BITCOIN-016 — Script & ScriptPubKey

### Related

- BITCOIN-003 — Whitepaper Section 2 — Transactions
- BITCOIN-008 — Whitepaper Section 7 — Reclaiming Disk Space
- BITCOIN-009 — Whitepaper Section 8 — Simplified Payment Verification
- BITCOIN-010 — Whitepaper Section 9 — Combining and Splitting Value
- BITCOIN-014 — UTXO Model
- BITCOIN-016 — Script & ScriptPubKey
- BITCOIN-017 — Mempool
- BITCOIN-023 — Chain Reorganization
- POW-009 — Coinbase Transaction and Mining Commitments

---

## Review Status

### Technical Review

Passed.

- Transaction fields were separated from wallet concepts.
- `txid` and `wtxid` definitions were checked against BIP141 and Bitcoin Core identifier documentation.
- Context-free, UTXO-context, script, finality, and policy checks were separated.
- BIP68 sequence-lock semantics were distinguished from BIP125 RBF policy signaling.
- Fee calculation was expressed as an implied value difference, not a serialized field.

### Evidence Review

Passed.

- Transaction structure claims cite official Bitcoin Developer documentation.
- SegWit serialization claims cite BIP141.
- Signature digest claims cite BIP143 and BIP341.
- Implementation claims cite current Bitcoin Core Doxygen or source documentation.
- RBF claims are labeled as policy and cite BIP125 plus Bitcoin Core policy source.

### Editorial Review

Passed.

- Markdown headings follow the project deep-dive structure.
- Metadata is complete.
- Tables and code fences are closed.
- Terminology is consistent: transaction, input, output, outpoint, scriptSig, scriptPubKey, witness, `txid`, `wtxid`, fee.

### Adversarial Review

Passed.

- The document does not describe addresses as consensus sender fields.
- It does not treat RBF as fraud or consensus invalidity.
- It does not treat change detection or ownership clustering as fact.
- It does not claim SegWit eliminates every possible form of transaction ambiguity.
- It distinguishes consensus validity from mempool relay policy.

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
