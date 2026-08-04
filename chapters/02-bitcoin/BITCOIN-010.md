---
knowledge_id: BITCOIN-010
title: Whitepaper Section 9 — Combining and Splitting Value
subtitle: UTXO Inputs, Multiple Outputs, Change, Fees, Fan-Out, and Transaction Construction
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 85 min
estimated_study: 230 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Transactions
  - UTXO
  - Wallets
parent:
  - Bitcoin Whitepaper
prerequisites:
  - BITCOIN-003
  - BITCOIN-007
  - BITCOIN-009
  - POW-009
related_topics:
  - UTXO Model
  - Transaction Inputs
  - Transaction Outputs
  - Change Output
  - Transaction Fees
  - Fan-Out
  - Privacy
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-TRANSACTIONS-001
  - REF-BTC-CORE-TRANSACTION-001
  - REF-BTC-CORE-TX-CHECK-001
  - REF-BTC-CORE-TX-VERIFY-001
  - REF-BTC-CORE-AMOUNT-001
tags:
  - bitcoin
  - whitepaper
  - combining-and-splitting-value
  - utxo
  - transactions
  - inputs
  - outputs
  - change
  - fees
---

# Whitepaper Section 9 — Combining and Splitting Value
> Deep Dive Series  
> Research Unit: BITCOIN-010

---

## Research Brief

```yaml
knowledge_id: BITCOIN-010
title: Whitepaper Section 9 — Combining and Splitting Value
research_question: >
  How does Bitcoin represent practical payments using transactions with
  multiple inputs and outputs, and what are the validation, fee, privacy,
  and on-chain-analysis implications of combining and splitting value?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-003
  - BITCOIN-007
  - BITCOIN-009
  - POW-009
parent: Bitcoin Whitepaper
previous: BITCOIN-009
next: BITCOIN-011
related_topics:
  - UTXO Model
  - Transaction Structure
  - Change Output
  - Transaction Fees
  - Multi-Input Heuristic
  - Fan-Out
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
  - Complete wallet coin-selection algorithms
  - Full Bitcoin privacy chapter
  - Script template taxonomy
  - Mempool package policy
  - Lightning channel construction
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Explain why Bitcoin transactions support multiple inputs and outputs.
- Distinguish an input from an output in the UTXO model.
- Explain how one large previous output can be split into payment and change.
- Explain how multiple smaller outputs can be combined into a larger payment.
- Calculate transaction fees as input value minus output value.
- Explain why change is a normal output, not a special consensus field.
- Explain why fan-out does not require carrying a complete standalone copy of transaction history.
- Distinguish transaction validity from wallet construction choices.
- Explain the privacy risk introduced by multi-input transactions.
- Identify which transaction-construction claims are facts, heuristics, or interpretations.

---

## 2. Key Questions

1. Why would handling coins individually be unwieldy?
2. What does it mean to combine value in Bitcoin?
3. What does it mean to split value in Bitcoin?
4. How does a transaction spend previous outputs?
5. What is a change output?
6. Is change marked on-chain as a special output type?
7. How are transaction fees represented?
8. Why does input value need to be at least output value?
9. What is fan-out?
10. Why does fan-out not require copying the entire transaction history?
11. What privacy heuristic is created by multi-input transactions?
12. What can analysts infer from transaction structure, and what remains uncertain?

---

## 3. Executive Summary

Whitepaper Section 9 explains why Bitcoin transactions can contain multiple inputs and multiple outputs. Handling each small unit of value as a separate transaction would be impractical. Bitcoin instead lets a transaction combine value from previous outputs and split value into new outputs.[^ref-btc-wp]

The core structure is:

```text
previous outputs consumed as inputs
    -> transaction
    -> new outputs created
```

Normally, a transaction may spend one larger previous output and create two outputs: one payment output and one change output returning the remainder to the sender. Or it may combine multiple smaller previous outputs as inputs to fund a larger payment.[^ref-btc-wp]

Modern Bitcoin expresses this through the UTXO model. A non-coinbase input references a previous outpoint, which identifies a transaction ID and an output index. An output specifies a value in satoshis and a locking script.[^ref-btc-dev-transactions][^ref-btc-core-transaction]

The fee is not a separate consensus field. It is the difference between total input value and total output value:

```text
fee = sum(inputs) - sum(outputs)
```

Bitcoin Core's `CheckTxInputs` checks that all inputs are available, that input value is not below output value, and then computes the transaction fee as input value minus output value.[^ref-btc-core-tx-verify]

The main analytical caveat is privacy. Combining multiple inputs can suggest common control, because one transaction usually needs authorization to spend all inputs. But this is a heuristic, not a universal fact. Collaborative transactions, CoinJoin-like constructions, custodial batching, and wallet behavior can break simple ownership assumptions.

---

## 4. Original Design

Whitepaper Section 9 states that although coins could be handled individually, creating a separate transaction for every cent in a transfer would be unwieldy. To support practical payments, transactions contain multiple inputs and outputs.[^ref-btc-wp]

The whitepaper gives two normal patterns:

| Pattern | Description |
|---|---|
| Single larger input | Spend a larger previous transaction output and return change |
| Multiple smaller inputs | Combine smaller amounts to fund a payment |

It also states that fan-out is not a problem because there is no need to extract a complete standalone copy of a transaction's history.[^ref-btc-wp]

This section is short, but it is foundational. It is the bridge between the conceptual chain of signatures in Section 2 and the practical UTXO transaction model used by Bitcoin.

---

## 5. Literal Interpretation

### "Combining Value"

Combining value means using multiple previous outputs as inputs to one new transaction.

Example:

```text
Input A: 0.40 BTC
Input B: 0.35 BTC
Input C: 0.30 BTC
Total:   1.05 BTC
```

The transaction can use those inputs together to fund outputs whose total value is less than or equal to `1.05 BTC`.

### "Splitting Value"

Splitting value means creating multiple outputs from the value consumed by the inputs.

Example:

```text
Input total:     1.00 BTC
Payment output: 0.70 BTC
Change output:  0.29 BTC
Fee:            0.01 BTC
```

The payment and change outputs are both ordinary outputs. Consensus does not label one as "payment" and the other as "change."

### "At Most Two Outputs"

The whitepaper says normally there will be at most two outputs: one payment and one change.[^ref-btc-wp] This is an original-design simplification, not a current consensus limit.

Modern Bitcoin transactions can have more than two outputs, subject to transaction and block limits. Bitcoin Developer documentation describes transactions as having a variable number of outputs, and Bitcoin Core's transaction model stores outputs in the `vout` vector.[^ref-btc-dev-transactions][^ref-btc-core-transaction]

### "Fan-Out"

Fan-out means a transaction depends on several earlier transactions, and those earlier transactions depend on many more. The whitepaper says this is not a problem because a complete standalone copy of the full history is not needed.[^ref-btc-wp]

In modern terms, validation checks the current UTXO view. A node needs the previous outputs being spent and sufficient validation state, not a bundled copy of every ancestor transaction inside the spending transaction.

---

## 6. Protocol Structure

### Transaction Components

Bitcoin Developer documentation describes a raw transaction as:

| Component | Role |
|---|---|
| Version | Transaction version number |
| Input count | Number of inputs |
| Inputs | Previous outputs being spent |
| Output count | Number of outputs |
| Outputs | New spendable outputs |
| Lock time | Optional finality constraint |

Each non-coinbase input references a previous outpoint. Each output contains a value and a locking script.[^ref-btc-dev-transactions]

### Input and Output Objects

Bitcoin Core models these concepts in `src/primitives/transaction.h`:

| Object | Role |
|---|---|
| `COutPoint` | Transaction hash plus output index |
| `CTxIn` | Input containing a previous outpoint, scriptSig, sequence, and witness |
| `CTxOut` | Output containing value and scriptPubKey |
| `CTransaction` | Transaction containing `vin`, `vout`, version, and lock time |

These are implementation structures, not independent consensus prose. They are primary implementation evidence for how Bitcoin Core represents transaction data.[^ref-btc-core-transaction]

### Value Conservation

For non-coinbase transactions:

```text
sum(input values) >= sum(output values)
```

The difference is the transaction fee. If output value exceeds input value, the transaction attempts to create unauthorized value and is invalid.[^ref-btc-core-tx-verify]

Coinbase transactions are different. They create permitted subsidy and claim fees under block reward rules, covered in BITCOIN-007 and POW-009.

---

## 7. Technical Mechanics

### Combining Inputs

Suppose a wallet has three UTXOs:

```text
UTXO A = 0.20 BTC
UTXO B = 0.30 BTC
UTXO C = 0.55 BTC
```

To pay `0.90 BTC` plus a `0.01 BTC` fee, the wallet may combine all three:

```text
inputs = 0.20 + 0.30 + 0.55 = 1.05 BTC
outputs = 0.90 + 0.14 = 1.04 BTC
fee = 1.05 - 1.04 = 0.01 BTC
```

The `0.14 BTC` output is economically change if it returns to the sender. On-chain, it is still just an output.

### Splitting Outputs

One input can create several outputs:

```text
input = 1.00 BTC

outputs:
  recipient 1 = 0.20 BTC
  recipient 2 = 0.30 BTC
  change      = 0.49 BTC

fee = 0.01 BTC
```

This allows batching, merchant payments, exchange withdrawals, wallet change, and many other patterns. The protocol validates values and scripts; it does not know the business reason for each output.

### Fee Construction

Fees are implicit:

```text
fee = sum(previous outputs spent) - sum(new outputs created)
```

Bitcoin Core's `CheckTxInputs`:

1. Checks that all inputs are present in the UTXO view.
2. Rejects missing or spent inputs.
3. Enforces coinbase maturity where relevant.
4. Sums input values.
5. Checks amount ranges.
6. Rejects `nValueIn < value_out`.
7. Sets the transaction fee to `nValueIn - value_out`.[^ref-btc-core-tx-verify]

### Context-Free Checks

Bitcoin Core's `CheckTransaction` performs checks that do not depend on the UTXO set:

- input vector must not be empty;
- output vector must not be empty;
- transaction serialized size must not exceed block weight constraints in that check;
- output values must not be negative;
- individual and total output values must be within range;
- duplicate inputs are rejected;
- non-coinbase transactions must not use null previous outpoints;
- coinbase script length is constrained.[^ref-btc-core-tx-check]

These checks are separate from the UTXO-dependent fee and spendability checks.

### Change Output

Change is a wallet construction pattern:

```text
selected input value > payment amount + intended fee
```

The wallet creates a new output for the remainder. That output is often controlled by the sender. Consensus does not label change. Analysts infer change using heuristics such as script type, address reuse, output amount patterns, wallet behavior, and later spending.

---

## 8. Mathematical or Economic Model

### Basic Accounting

Let:

```text
I = sum(input values)
O = sum(output values)
F = transaction fee
```

Then:

```text
F = I - O
```

Validity requires:

```text
I >= O
F >= 0
```

Bitcoin Core also checks monetary ranges using `MoneyRange`, and `MAX_MONEY` is defined as `21000000 * COIN` with `COIN = 100000000` satoshis.[^ref-btc-core-amount]

### Change Calculation

For a simple one-payment transaction:

```text
change = selected_input_value - payment_amount - fee
```

Example:

```text
selected input value = 1.000 BTC
payment amount       = 0.700 BTC
fee                  = 0.002 BTC
change               = 0.298 BTC
```

The change output is optional. If the wallet does not create change, the unassigned value becomes additional fee.

### Fan-Out and Graph Growth

Transaction graphs can fan out:

```text
one transaction
    -> many outputs
    -> many later spends
    -> more outputs
```

They can also fan in:

```text
many previous outputs
    -> one transaction
```

The whitepaper's point is that a transaction does not need to carry a full copy of this ancestry. It references previous outputs by outpoint, and validation uses node state to check whether those outputs are available and spendable.

---

## 9. Security Assumptions

### What the Model Requires

Combining and splitting value depends on:

1. Outputs being uniquely referenced by transaction ID and output index.
2. Full nodes maintaining a correct view of unspent outputs.
3. Signatures and scripts authorizing spends of referenced outputs.
4. Value conservation being enforced by validation.
5. The network rejecting double spends of the same output in the same active chain.

### What It Does Not Require

It does not require:

- a central account balance;
- one transaction per coin unit;
- a complete transaction history embedded in each transaction;
- a protocol-level change-output label;
- a trusted party to calculate balances.

### Privacy Boundary

Section 9 enables practical payments, but it creates linkability surfaces. A multi-input transaction often suggests that the same party controlled all inputs because the transaction needed spending authorization for each input.

That statement is a heuristic, not a consensus fact. Collaborative transactions can intentionally combine inputs from multiple parties. This becomes central in Whitepaper Section 10 on privacy.

---

## 10. Bitcoin Core Implementation

### Transaction Data Structures

Bitcoin Core's transaction primitives provide the implementation model:

```text
CTransaction
    vin:  vector of CTxIn
    vout: vector of CTxOut
```

`COutPoint` identifies the previous output being spent through a transaction hash and output index. `CTxOut` stores the output value and locking script.[^ref-btc-core-transaction]

### `CheckTransaction`

`CheckTransaction` verifies transaction structure before UTXO context is considered. It rejects empty inputs, empty outputs, invalid output amounts, duplicate inputs, and non-coinbase null previous outpoints.[^ref-btc-core-tx-check]

This maps directly to Section 9's model. Multiple inputs and outputs are allowed, but they must be structurally valid.

### `CheckTxInputs`

`CheckTxInputs` is UTXO-context dependent. It checks whether inputs exist in the UTXO view, enforces coinbase maturity, sums input values, rejects transactions whose input value is below output value, and returns the fee.[^ref-btc-core-tx-verify]

This is the implementation-level enforcement of value conservation for ordinary transactions.

### Amount Rules

Bitcoin Core represents amounts as `CAmount`, a signed 64-bit integer count of satoshis. `MoneyRange` checks that a value is non-negative and not greater than `MAX_MONEY`.[^ref-btc-core-amount]

These amount checks reduce the risk of overflow and unauthorized value creation. They do not identify change or payment intent.

---

## 11. On-Chain Implications

### Observable Facts

From a decoded transaction, analysts can observe:

- number of inputs;
- previous outpoints referenced by inputs;
- number of outputs;
- output amounts;
- output script types;
- transaction fee if input values are known;
- transaction size and weight if full serialization is available;
- whether an input set combines multiple previous outputs.

### Inferences

Analysts may infer:

- likely payment output;
- likely change output;
- possible common input ownership;
- consolidation behavior;
- batching behavior;
- exchange or custodian withdrawal patterns;
- wallet behavior.

These are interpretations. They require confidence labels and caveats.

### Unknowns

Transaction structure alone usually cannot prove:

- real-world sender identity;
- real-world recipient identity;
- which output is change;
- whether inputs belong to one person or multiple collaborators;
- why a wallet selected particular inputs;
- whether a transaction is an exchange withdrawal, self-transfer, or payment without additional evidence.

---

## 12. Institutional Thinking

### Why This Section Matters

Section 9 is a small whitepaper section with large analytical consequences. It explains why Bitcoin does not behave like an account ledger. Users do not spend from a single mutable balance. They consume discrete previous outputs and create new outputs.

For institutions, this affects:

- deposit attribution;
- wallet clustering;
- transaction monitoring;
- fee accounting;
- UTXO management;
- custody operations;
- privacy risk;
- tax-lot and cost-basis workflows;
- forensic graph analysis.

### Operational Controls

Institutions should:

- compute fees from full input and output context;
- avoid assuming every two-output transaction has payment plus change;
- treat multi-input clustering as heuristic;
- track UTXO fragmentation and consolidation;
- document change-detection methodology;
- separate validated transaction facts from attribution claims;
- use full-node or independently verified data for high-value analysis.

### Research Discipline

Good transaction analysis starts with facts:

```text
input count
output count
input values
output values
scripts
confirmation context
```

Only after that should it move to interpretation:

```text
possible change
possible owner cluster
possible exchange behavior
possible consolidation
```

This separation prevents common analytical overreach.

---

## 13. Common Misinterpretations

### Misinterpretation 1: Bitcoin has account balances like a bank database.

Incorrect. Bitcoin uses spendable outputs. Wallet balances are computed from UTXOs controlled by keys or scripts.

### Misinterpretation 2: Change outputs are marked on-chain.

Incorrect. Change is inferred from transaction construction and later behavior. It is not a special consensus flag.

### Misinterpretation 3: Every two-output transaction has one payment and one change output.

Overstated. Many transactions follow that pattern, but batching, collaborative transactions, self-transfers, and wallet-specific behavior can produce different meanings.

### Misinterpretation 4: Multi-input transactions prove one owner.

Overstated. Multi-input transactions often suggest common control, but collaborative spending can break the heuristic.

### Misinterpretation 5: Fees are an explicit output paid to miners.

Incorrect. Fees are the unassigned difference between input value and output value. Miners claim aggregate fees through the block's coinbase transaction.

### Misinterpretation 6: Fan-out requires each transaction to contain its full history.

Incorrect. Transactions reference previous outputs. Nodes use validated state and prior block data, not a standalone transaction-history bundle inside each transaction.

---

## 14. Research Questions

1. How often do Bitcoin transactions use one input and two outputs?
2. How reliable are common change-output heuristics across script types?
3. How does batching alter output-count interpretation?
4. How do exchanges differ from self-custody wallets in input/output patterns?
5. How does UTXO consolidation respond to fee-market conditions?
6. What false positives arise from multi-input clustering?
7. How do collaborative transactions weaken ownership heuristics?
8. How should institutions score confidence in change detection?
9. How does output splitting affect future fee burden?
10. Which transaction graph metrics best identify operational wallet behavior?

---

## 15. Practical Exercises

1. A transaction spends inputs worth `0.4 BTC`, `0.3 BTC`, and `0.2 BTC`. It creates outputs worth `0.5 BTC` and `0.39 BTC`. Compute the fee.
2. Explain why the second output might be change but cannot be proven as change from value alone.
3. Decode a transaction and identify each input's previous outpoint.
4. Explain why a transaction with five inputs can still be valid.
5. Explain why a transaction with output value greater than input value is invalid unless it is a coinbase reward under block reward rules.
6. Give one example where a multi-input ownership heuristic can fail.

---

## 16. Evidence Classification

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Whitepaper Section 9 on combining and splitting value | A |
| REF-BTC-DEV-TRANSACTIONS-001 | Official developer documentation | Raw transaction format, inputs, outputs, outpoints, and value rules | A |
| REF-BTC-CORE-TRANSACTION-001 | Primary implementation source | `COutPoint`, `CTxIn`, `CTxOut`, and `CTransaction` structures | A |
| REF-BTC-CORE-TX-CHECK-001 | Primary implementation source | `CheckTransaction` context-free transaction checks | A |
| REF-BTC-CORE-TX-VERIFY-001 | Primary implementation source | `CheckTxInputs`, UTXO checks, and fee calculation | A |
| REF-BTC-CORE-AMOUNT-001 | Primary implementation source | `CAmount`, `COIN`, `MAX_MONEY`, and `MoneyRange` | A |

### Claim Ledger

| Claim ID | Claim | Classification | Evidence | Source |
|---|---|---|---|---|
| C001 | Whitepaper Section 9 states that transactions can contain multiple inputs and outputs to combine and split value. | FACT | A | REF-BTC-WP-001 |
| C002 | The whitepaper describes a normal pattern of one payment output and one change output. | FACT | A | REF-BTC-WP-001 |
| C003 | Modern Bitcoin transactions can contain variable numbers of inputs and outputs. | FACT | A | REF-BTC-DEV-TRANSACTIONS-001; REF-BTC-CORE-TRANSACTION-001 |
| C004 | Non-coinbase inputs reference previous outpoints. | FACT | A | REF-BTC-DEV-TRANSACTIONS-001; REF-BTC-CORE-TRANSACTION-001 |
| C005 | Bitcoin Core rejects duplicate transaction inputs. | FACT | A | REF-BTC-CORE-TX-CHECK-001 |
| C006 | Bitcoin Core rejects non-coinbase transactions with null previous outpoints. | FACT | A | REF-BTC-CORE-TX-CHECK-001 |
| C007 | Bitcoin Core rejects ordinary transactions whose input value is below output value. | FACT | A | REF-BTC-CORE-TX-VERIFY-001 |
| C008 | Bitcoin Core computes transaction fee as input value minus output value. | FACT | A | REF-BTC-CORE-TX-VERIFY-001 |
| C009 | Change output identification is heuristic rather than a consensus fact. | INTERPRETATION | B | REF-BTC-WP-001; REF-BTC-DEV-TRANSACTIONS-001 |
| C010 | Multi-input common ownership is a useful but fallible heuristic. | INTERPRETATION | B | REF-BTC-WP-001 |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| HEURISTIC | Practical analysis rule requiring caveats |
| UNKNOWN | Evidence is insufficient |

---

## 17. Knowledge Graph

```text
BITCOIN-010 Combining and Splitting Value
|
+-- interprets: Whitepaper Section 9
|
+-- transaction
|   +-- contains: inputs
|   +-- contains: outputs
|   +-- computes: fee = inputs - outputs
|
+-- combining
|   +-- uses: multiple previous outputs
|   +-- enables: larger payment from smaller UTXOs
|
+-- splitting
|   +-- creates: multiple new outputs
|   +-- enables: payment + change
|
+-- validation
|   +-- CheckTransaction
|   +-- CheckTxInputs
|   +-- MoneyRange
|
+-- analysis_risks
    +-- change detection is heuristic
    +-- multi-input ownership is heuristic
    +-- fan-out does not require full embedded history
```

---

## 18. References

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Section 9, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-transactions]: Bitcoin Developer Documentation, "Transactions," raw transaction format, transaction inputs and outputs, outpoints, and coinbase input exception, https://developer.bitcoin.org/reference/transactions.html, accessed 2026-08-04.

[^ref-btc-core-transaction]: Bitcoin Core Contributors, `src/primitives/transaction.h`, `COutPoint`, `CTxIn`, `CTxOut`, and `CTransaction`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/primitives_2transaction_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-tx-check]: Bitcoin Core Contributors, `src/consensus/tx_check.cpp`, `CheckTransaction`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/tx__check_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-tx-verify]: Bitcoin Core Contributors, `src/consensus/tx_verify.cpp`, `CheckTxInputs`, input availability, coinbase maturity, input/output value checks, and fee calculation, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/tx__verify_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-amount]: Bitcoin Core Contributors, `src/consensus/amount.h`, `CAmount`, `COIN`, `MAX_MONEY`, and `MoneyRange`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/amount_8h_source.html, accessed 2026-08-04.

---

## 19. Cross References

### Parent

- Bitcoin Whitepaper

### Previous

- BITCOIN-009 — Whitepaper Section 8 — Simplified Payment Verification

### Next

- BITCOIN-011 — Whitepaper Section 10 — Privacy

### Related

- BITCOIN-003 — Whitepaper Section 2 — Transactions
- BITCOIN-007 — Whitepaper Section 6 — Incentive
- BITCOIN-014 — UTXO Model
- BITCOIN-015 — Transactions in Depth
- BITCOIN-017 — Mempool
- BITCOIN-026 — Fee Market
- POW-009 — Coinbase Transaction and Mining Commitments

---

## Review Status

### Technical Review

Passed.

- Whitepaper Section 9 was separated from current transaction implementation details.
- Input combining, output splitting, change, fee calculation, and fan-out were separated.
- `COutPoint`, `CTxIn`, `CTxOut`, `CTransaction`, `CheckTransaction`, `CheckTxInputs`, and `MoneyRange` were checked against Bitcoin Core source.
- Change output and multi-input ownership were described as heuristics, not consensus labels.

### Evidence Review

Passed.

- Whitepaper Section 9 claims cite the whitepaper directly.
- Transaction format claims cite official Bitcoin Developer documentation.
- Current implementation claims cite Bitcoin Core source.
- On-chain attribution and change-detection claims are labeled as interpretation or heuristic.

### Editorial Review

Passed.

- Markdown headings follow the project deep-dive structure.
- Metadata is complete.
- Tables and code fences are closed.
- Terminology is consistent: input, output, outpoint, UTXO, change, fee, fan-out.

### Adversarial Review

Passed.

- The document does not imply change is explicitly labeled on-chain.
- The document does not treat multi-input ownership as proof.
- The document does not imply the whitepaper's "at most two outputs" phrase is a consensus limit.
- The document distinguishes ordinary transaction value conservation from coinbase reward creation.

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
