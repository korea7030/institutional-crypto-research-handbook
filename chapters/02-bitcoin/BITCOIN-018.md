---
knowledge_id: BITCOIN-018
title: Transaction Fees
subtitle: Fee Calculation, Feerate, Weight, Virtual Size, Relay Policy, Fee Estimation, RBF, CPFP, and Miner Incentives
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 105 min
estimated_study: 300 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Transaction Fees
  - Fee Market
  - Mempool Policy
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-014
  - BITCOIN-015
  - BITCOIN-017
related_topics:
  - UTXO Model
  - Transaction Validation
  - Mempool
  - Block Space
  - RBF
  - CPFP
  - Fee Estimation
  - Miner Revenue
primary_sources:
  - REF-BTC-WP-001
  - REF-BIP-0125
  - REF-BIP-0141
  - REF-BTC-CORE-29-RELEASE-001
  - REF-BTC-CORE-31-RELEASE-001
  - REF-BTC-CORE-TX-VERIFY-001
  - REF-BTC-CORE-CONSENSUS-VALIDATION-001
  - REF-BTC-CORE-FEERATE-001
  - REF-BTC-CORE-FEE-ESTIMATOR-001
  - REF-BTC-CORE-POLICY-001
  - REF-BTC-CORE-RBF-001
  - REF-BTC-CORE-RPC-FEE-001
tags:
  - bitcoin
  - internals
  - transaction-fees
  - feerate
  - weight
  - vsize
  - mempool
  - rbf
  - cpfp
  - fee-estimation
---

# Transaction Fees
> Bitcoin Internals  
> Research Unit: BITCOIN-018

---

## Research Brief

```yaml
knowledge_id: BITCOIN-018
title: Transaction Fees
research_question: >
  How are Bitcoin transaction fees computed, how do weight and virtual size
  convert fees into fee rates, how do mempool and mining policies use fee
  rates, and how should institutions reason about fee estimation, RBF, CPFP,
  and miner incentives without confusing policy with consensus?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-014
  - BITCOIN-015
  - BITCOIN-017
parent: Bitcoin Internals
previous: BITCOIN-017
next: BITCOIN-019
related_topics:
  - UTXO Model
  - Transactions
  - Mempool
  - Block Template Construction
  - Mining Incentives
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
  - Full wallet coin-selection algorithm design
  - Mining pool private fee-market agreements
  - Detailed Lightning fee policy
  - Non-Bitcoin fee markets
  - Real-time fee recommendation for a live payment
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Explain why Bitcoin transaction fees are implicit rather than a serialized transaction field.
- Compute transaction fees from input and output values.
- Define fee rate in sat/vB and BTC/kvB contexts.
- Explain transaction weight and virtual size under SegWit.
- Distinguish consensus validity from fee-based relay and mining policy.
- Explain how RBF and CPFP change fee incentives for unconfirmed transactions.
- Explain how Bitcoin Core fee estimation uses observed confirmations.
- Identify fee-related Bitcoin Core source areas.
- Explain why fee estimation is probabilistic and local.
- Evaluate fee management risks for institutional operations.

---

## 2. Key Questions

1. Where is the transaction fee stored?
2. How do nodes compute a transaction fee?
3. What is the difference between absolute fee and fee rate?
4. What is transaction weight?
5. What is virtual size?
6. Why do witness bytes affect fees differently from non-witness bytes?
7. What fee thresholds are consensus rules?
8. What fee thresholds are relay or mining policy?
9. How does RBF fee bumping work?
10. How does CPFP fee bumping work?
11. How does Bitcoin Core estimate fees?
12. What should institutions avoid inferring from mempool fee data?

---

## 3. Executive Summary

Bitcoin transaction fees are not explicit fields. For a non-coinbase transaction, the fee is the difference between the total value of the UTXOs spent and the total value of new outputs created:

```text
fee = sum(input UTXO values) - sum(output values)
```

Bitcoin Core's `CheckTxInputs` checks input availability and amounts, verifies that input value is not less than output value, and returns the fee as a computed value.[^ref-btc-core-tx-verify]

Fee rate converts that absolute fee into a price for block space:

```text
fee_rate = fee / virtual_size
```

SegWit defines transaction weight as base transaction size times 3 plus total transaction size, and virtual transaction size as weight divided by 4, rounded up.[^ref-bip-0141] Bitcoin Core implements transaction weight through `GetTransactionWeight` using serialization with and without witness data.[^ref-btc-core-consensus-validation]

Fees operate across three boundaries:

| Boundary | Fee role |
|---|---|
| Consensus | Ordinary transactions must not create value; block subsidy plus fees limits coinbase value |
| Relay policy | Nodes may reject transactions below local relay or mempool thresholds |
| Mining policy | Miners generally prefer higher expected fee rate packages or chunks |

Bitcoin Core 29.1 lowered default `-minrelaytxfee` and `-incrementalrelayfee` to 100 satoshis per kvB, while noting that the mempool minimum fee can still rise under high volume.[^ref-btc-core-29-release] Bitcoin Core 31.0 updated the fee estimator's minimum fee-rate bucket to 0.1 sat/vB and removed deprecated static wallet fee settings such as `-paytxfee` and `settxfee`.[^ref-btc-core-31-release]

For institutional analysis, fees are strong evidence of block-space bidding and operational urgency only with caveats. A high fee may indicate urgency, poor estimation, consolidation during congestion, RBF/CPFP rescue, or policy constraints. It does not by itself prove intent.

---

## 4. Protocol Structure

### Fee as Implied Difference

A Bitcoin transaction consumes UTXOs with known values and creates outputs with specified values. For non-coinbase transactions:

```text
total_input_value >= total_output_value
fee = total_input_value - total_output_value
```

If outputs exceed inputs, the transaction creates unauthorized value and is invalid. Bitcoin Core enforces this in UTXO-context validation.[^ref-btc-core-tx-verify]

### Coinbase Collection

Miners collect fees through the coinbase transaction. The block's coinbase output value is constrained by:

```text
coinbase_claim <= block_subsidy + sum(transaction_fees_in_block)
```

The whitepaper describes incentives as a combination of new coins and transaction fees, and notes that fees can support incentives after a predetermined number of coins enter circulation.[^ref-btc-wp]

### Absolute Fee vs Fee Rate

Absolute fee:

```text
10,000 sats
```

Fee rate:

```text
20 sat/vB
```

Absolute fee determines how many satoshis the miner can collect. Fee rate determines how attractive the transaction is per unit of scarce block capacity.

### Weight and Virtual Size

SegWit changed how transaction size is priced by introducing weight:

```text
transaction_weight = base_size * 3 + total_size
virtual_size = ceil(transaction_weight / 4)
```

BIP141 defines base size as serialization without witness data and total size as serialization including witness data.[^ref-bip-0141]

This means witness data contributes less weight than non-witness data, but it is not free.

---

## 5. Technical Mechanics

### Fee Calculation Example

```text
inputs:
  UTXO A = 0.08000000 BTC
  UTXO B = 0.02000000 BTC

outputs:
  recipient = 0.07000000 BTC
  change    = 0.02985000 BTC

fee:
  0.10000000 - 0.09985000 = 0.00015000 BTC
```

In satoshis:

```text
10,000,000 sats - 9,985,000 sats = 15,000 sats
```

If virtual size is 300 vB:

```text
fee_rate = 15,000 / 300 = 50 sat/vB
```

### No Fee Field

The transaction does not contain:

```text
fee: 15000
```

Nodes compute the fee from the UTXO set. This is why fee calculation requires knowing the previous outputs being spent.

### Fee Rate Units

Common units:

| Unit | Meaning |
|---|---|
| sat/vB | Satoshis per virtual byte; common user-facing unit |
| sat/kvB | Satoshis per 1,000 virtual bytes |
| BTC/kvB | Bitcoin Core RPC display unit in some fee RPCs |

Conversions:

```text
1 sat/vB = 1,000 sat/kvB
100 sat/kvB = 0.1 sat/vB
```

Bitcoin Core RPC fee outputs often use BTC/kvB, while wallets and fee dashboards often use sat/vB.

### Dust and Uneconomical Outputs

Dust policy discourages outputs whose value is too small relative to the cost of spending them. Dust is policy, not a consensus rule. Bitcoin Core exposes relay dust fee settings through node interfaces and policy code.[^ref-btc-core-policy]

Analytical rule:

```text
small output != invalid output
small output may be nonstandard under policy
```

### Fee Bumping With RBF

RBF changes an unconfirmed transaction by replacing it with a conflicting transaction that pays better under the node's replacement policy.

BIP125 defines opt-in replaceability signaling through input `nSequence` and replacement fee requirements.[^ref-bip-0125] Bitcoin Core implements RBF policy checks in `src/policy/rbf.cpp`, including explicit or inherited signaling and replacement fee checks.[^ref-btc-core-rbf]

### Fee Bumping With CPFP

CPFP adds a high-fee child transaction spending an output from a low-fee parent:

```text
parent fee rate low
child fee rate high
combined package fee rate acceptable
```

This works because the child cannot be mined unless the parent is also mined. Miners evaluate the combined incentive where package or chunk selection applies.

Bitcoin Core 31.0 release notes describe package relay changes allowing one-parent-one-child packages where the parent may be below `-minrelaytxfee`, even zero fee, when covered by the package context.[^ref-btc-core-31-release]

---

## 6. Fee Market Dynamics

### Block Space as Scarce Resource

Bitcoin block capacity is limited by consensus weight rules. When demand for block inclusion exceeds near-term block space, users bid through fee rates.

The relevant economic question for miners is not simply:

```text
which transaction pays the largest absolute fee?
```

It is closer to:

```text
which valid set of transactions maximizes fees per scarce block resource?
```

With dependencies, the set may be a parent-child package or cluster chunk rather than an isolated transaction.

### Mempool Minimum Fee

The mempool minimum fee is local and dynamic. When a node's mempool is full or under pressure, it may evict lower-fee transactions and raise the effective minimum needed for new mempool acceptance.

Bitcoin Core 29.1 release notes explicitly warn that the mempool minimum fee still changes in response to high volume, even after default relay fee changes.[^ref-btc-core-29-release]

### Miner Selection

Mining software typically selects transactions by fee rate, but real behavior can differ because of:

- dependency packages;
- cluster/chunk ordering;
- private transaction relay;
- pool policy;
- transaction prioritization;
- template construction settings;
- non-fee considerations such as operational constraints.

Bitcoin Core 31.0 cluster mempool orders transactions by expected mining feerate using chunks, and this ordering is used in block template construction, eviction, relay announcements, and replacement validation.[^ref-btc-core-31-release]

### Fee Estimation

Fee estimation is a forecast, not a guarantee. Bitcoin Core's `CBlockPolicyEstimator` estimates the feerate needed for inclusion within a target number of blocks by grouping transactions into feerate buckets and tracking how long transactions in those buckets take to be mined.[^ref-btc-core-fee-estimator]

The RPC `estimatesmartfee` estimates the approximate fee per kilobyte needed for a transaction to begin confirmation within a target number of blocks if possible, using virtual transaction size as defined by BIP141.[^ref-btc-core-rpc-fee]

Limitations:

- estimates depend on the node's observed mempool and blocks;
- estimates may be unavailable with insufficient data;
- demand can change suddenly;
- private orderflow can reduce public-mempool visibility;
- estimates do not guarantee inclusion.

---

## 7. Mathematical or Economic Model

### Base Formulas

```text
fee = input_value - output_value
weight = base_size * 3 + total_size
vsize = ceil(weight / 4)
feerate = fee / vsize
```

### Package Formulas

```text
package_fee = sum(transaction_fees)
package_vsize = sum(transaction_vsizes)
package_feerate = package_fee / package_vsize
```

### Change and Fee Tradeoff

Suppose a wallet has one 100,000 sat UTXO and wants to pay 70,000 sats.

If target fee is 2,000 sats:

```text
recipient = 70,000
change = 28,000
fee = 2,000
```

If the wallet omits change:

```text
recipient = 70,000
fee = 30,000
```

Omitting change can accidentally overpay fees unless it is deliberate. Analysts should verify output construction before interpreting fee size as urgency.

### Consolidation Cost

A consolidation transaction may have many inputs and few outputs:

```text
many small inputs -> larger vsize -> higher absolute fee at same sat/vB
```

A high absolute fee can be a normal consequence of large input count rather than urgent settlement demand.

### Security Budget

Miner revenue per block is:

```text
miner_revenue = subsidy + transaction_fees
```

As subsidy declines over halvings, transaction fees become more important to miner revenue. The whitepaper anticipates transaction fees as an incentive source after issuance declines.[^ref-btc-wp]

This does not imply fees must follow a specific path. It means fee revenue is part of Bitcoin's long-run security-budget discussion.

---

## 8. Security Assumptions

### What Fees Secure

Fees support:

- miner incentives to include transactions;
- block-space allocation during congestion;
- anti-spam filtering through relay policy;
- transaction replacement incentives;
- package inclusion incentives.

### What Fees Do Not Guarantee

Fees do not guarantee:

- next-block confirmation;
- global propagation;
- that no replacement will occur;
- that a miner will include the transaction;
- that a transaction is economically meaningful;
- that a high-fee transaction is legitimate.

### Fee-Related Failure Modes

| Failure mode | Description | Boundary |
|---|---|---|
| Underpayment | Fee rate too low for current mempool or miner policy | Policy/economic |
| Overpayment | Wallet pays much more than needed | Wallet/operator |
| Stuck parent | Parent has low fee and child depends on it | Package policy |
| RBF surprise | Unconfirmed transaction is replaced | Policy/risk |
| CPFP pinning | Structure makes fee bumping difficult or expensive | Policy/adversarial |
| Dust output | Output uneconomical or nonstandard to relay | Policy |
| Misread fee | Analyst confuses absolute fee with fee rate | Analytics |

---

## 9. Bitcoin Core Implementation

### Fee Calculation in Validation

Bitcoin Core computes transaction fee in UTXO-context validation. `Consensus::CheckTxInputs` takes a transaction, validation state, coins view cache, spend height, and output fee variable. It checks input validity and amounts and returns the computed fee.[^ref-btc-core-tx-verify]

This is why fee calculation belongs after previous outputs are known.

### Weight and Virtual Size

Bitcoin Core's `src/consensus/validation.h` defines `GetTransactionWeight`. The implementation uses serialization with and without witness data and is equivalent to:

```text
weight = stripped_size * 3 + total_size
```

It also defines block weight calculation and input weight calculation.[^ref-btc-core-consensus-validation]

### Fee Rate Object

Bitcoin Core represents fee rates with `CFeeRate` in policy code. Fee-rate objects are used across relay, mempool, wallet, and fee-estimation interfaces.[^ref-btc-core-feerate]

The implementation detail matters because users may encounter different units in RPCs, logs, and wallet settings.

### Fee Estimator

Bitcoin Core's `CBlockPolicyEstimator` tracks feerate buckets over short, medium, and long horizons. Doxygen describes it as estimating the feerate needed for inclusion within a number of blocks by observing how long transactions in similar feerate buckets take to be mined.[^ref-btc-core-fee-estimator]

This is empirical, not oracle-based.

### Policy Defaults and Version Caveat

Bitcoin Core 29.1 changed default `-minrelaytxfee` and `-incrementalrelayfee` to 100 satoshis per kvB.[^ref-btc-core-29-release] Bitcoin Core 31.0 updated the fee estimator minimum bucket to 0.1 sat/vB and removed static wallet fee settings `-paytxfee` and `settxfee`.[^ref-btc-core-31-release]

This document reflects Bitcoin Core 31.x behavior as of 2026-08-04. Fee policy is more changeable than consensus.

### RBF Policy Code

Bitcoin Core's `src/policy/rbf.cpp` checks opt-in replaceability and replacement fee requirements. The code includes `IsRBFOptIn` and fee checks such as ensuring replacement fees cover original fees and relay bandwidth costs.[^ref-btc-core-rbf]

### Mining Policy Code

Bitcoin Core policy headers include mining-related defaults such as `DEFAULT_BLOCK_MIN_TX_FEE`, the default for `-blockmintxfee`, which sets the minimum feerate for transactions in blocks created by the mining code.[^ref-btc-core-policy]

Mining policy is configurable and may differ among miners.

---

## 10. Consensus, Policy, and Wallet Behavior

### Consensus

Consensus fee-related rules include:

- ordinary transactions cannot create value;
- coinbase outputs cannot claim more than subsidy plus included transaction fees;
- block weight limits constrain total included transaction data.

Consensus does not require a transaction to pay a positive fee in all cases. Zero-fee or very-low-fee transactions can be consensus-valid if all other rules pass.

### Policy

Policy fee-related rules include:

- minimum relay fee;
- incremental relay fee;
- mempool minimum fee;
- dust threshold;
- RBF replacement fee requirements;
- package acceptance fee requirements;
- mining template minimum fee.

Policy determines relay and local mempool acceptance, not universal validity.

### Wallet Behavior

Wallet behavior includes:

- fee estimation;
- target confirmation selection;
- coin selection;
- change output management;
- RBF opt-in decisions;
- CPFP construction;
- batching and consolidation timing.

Wallet errors or operator settings can cause fee overpayment or underpayment even when the transaction is consensus-valid.

---

## 11. On-Chain Implications

### Strong Evidence

Confirmed transaction data strongly supports:

- absolute fee after input values are known;
- fee rate after vsize is computed;
- whether SegWit witness discount affects size accounting;
- whether a transaction used many inputs;
- whether a transaction was confirmed during a high-fee period;
- miner fee revenue for a block.

### Weak Evidence

Fee data weakly supports:

- payer urgency;
- wallet sophistication;
- exchange withdrawal policy;
- emergency movement;
- liquidation behavior;
- deliberate overpayment;
- miner preference beyond observed inclusion.

High fees can result from congestion, bad estimation, sweeping many inputs, RBF replacement, CPFP rescue, or policy constraints.

### Block-Level Analysis

For a block:

```text
block_fees = sum(fees of non-coinbase transactions)
miner_revenue = block_subsidy + block_fees
fee_share = block_fees / miner_revenue
```

This helps analyze fee pressure and security-budget composition, but it should not be extrapolated from one block to long-run economics.

---

## 12. Institutional Thinking

### Treasury Operations

Institutions should manage fees through:

- batching withdrawals when possible;
- consolidating UTXOs during low-fee periods;
- using RBF for controlled fee bumping;
- using CPFP for stuck dependencies;
- monitoring package context rather than isolated transaction fee rate only;
- setting policy for urgent vs normal settlement.

### Custody and Signing Controls

Signing systems should display:

- absolute fee;
- fee rate;
- input count and total input value;
- output amounts;
- change output;
- effective fee rate after package context if relevant;
- RBF signaling status.

Controls should prevent accidental no-change overpayment and wrong-fee signing.

### Accounting

Fees are expenses paid in BTC. Accounting systems should distinguish:

- transaction amount sent to recipient;
- change returned to institution;
- network fee;
- miner revenue recognized by block;
- pending fee estimate vs confirmed paid fee.

### Risk

Unconfirmed low-fee deposits may be delayed or replaced. High-fee withdrawals may still fail to confirm quickly during sudden congestion or if they depend on low-fee unconfirmed parents.

Operational dashboards should label fee-based confirmation estimates as forecasts.

---

## 13. Common Misinterpretations

### "The Fee Is Written in the Transaction"

No. It is computed from input and output values.

### "Higher Absolute Fee Always Means Better Confirmation Probability"

No. Fee rate and package context matter more than absolute fee alone.

### "Consensus Requires a Minimum Fee"

No. Minimum fee thresholds are generally relay and mining policy. Consensus prevents unauthorized value creation.

### "A Fee Estimate Is a Guarantee"

No. It is a probabilistic estimate from observed data and policy assumptions.

### "SegWit Makes Witness Data Free"

No. Witness data is discounted in weight accounting, not free.

### "High Fee Means Urgent Transfer"

Not necessarily. It may reflect large input count, bad estimation, RBF, CPFP, batching, or consolidation.

---

## 14. Research Questions

1. How do public mempool fee estimates differ across geographically distributed nodes?
2. How often do wallet fee estimates overpay relative to next-block clearing rates?
3. How does cluster mempool change CPFP and RBF analysis?
4. What fraction of exchange withdrawals use RBF signaling?
5. How much miner revenue comes from fees across different market regimes?
6. How do batching and consolidation affect long-run institutional fee cost?
7. What evidence separates urgent movement from large-input-count fee inflation?

---

## 15. Practical Exercises

### Exercise 1: Compute a Transaction Fee

Select a confirmed non-coinbase transaction.

1. List every input outpoint.
2. Retrieve each previous output value.
3. Sum input values.
4. Sum output values.
5. Compute fee.
6. Compute fee rate from virtual size.

### Exercise 2: Compare Absolute Fee and Fee Rate

Find two transactions:

- one with high absolute fee and large vsize;
- one with lower absolute fee but higher sat/vB.

Explain which one is more attractive per block-space unit.

### Exercise 3: RBF Fee Bump

Find an unconfirmed replacement pair or historical example.

Record:

- original fee;
- original vsize;
- replacement fee;
- replacement vsize;
- absolute fee increase;
- fee-rate increase;
- whether replacement policy was likely relevant.

### Exercise 4: CPFP Package

Find a parent-child pair where the child spends an unconfirmed parent output.

Compute:

```text
parent_feerate
child_feerate
combined_package_feerate
```

Explain why the child can change the parent's confirmation incentive.

---

## 16. Evidence Classification

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Fees as miner incentive after issuance declines | A |
| REF-BIP-0125 | Policy BIP | Opt-in RBF signaling and replacement fee rules | A |
| REF-BIP-0141 | Consensus BIP | Transaction weight and virtual size definitions | A |
| REF-BTC-CORE-29-RELEASE-001 | Release documentation | Default relay fee and incremental relay fee change to 100 sat/kvB | A |
| REF-BTC-CORE-31-RELEASE-001 | Release documentation | Fee estimator bucket update, package relay note, static wallet fee removal | A |
| REF-BTC-CORE-TX-VERIFY-001 | Primary implementation source | `CheckTxInputs` fee calculation | A |
| REF-BTC-CORE-CONSENSUS-VALIDATION-001 | Primary implementation source | `GetTransactionWeight` and weight formula | A |
| REF-BTC-CORE-FEERATE-001 | Primary implementation source | `CFeeRate` fee-rate representation | A |
| REF-BTC-CORE-FEE-ESTIMATOR-001 | Primary implementation source | `CBlockPolicyEstimator` algorithm | A |
| REF-BTC-CORE-POLICY-001 | Primary implementation source | Relay and mining policy fee defaults | A |
| REF-BTC-CORE-RBF-001 | Primary implementation source | RBF replacement policy checks | A |
| REF-BTC-CORE-RPC-FEE-001 | RPC documentation | `estimatesmartfee` behavior and units | B |

### Claim Ledger

| Claim | Classification | Evidence |
|---|---|---|
| Fees are computed as input value minus output value | FACT | Bitcoin Core `CheckTxInputs` |
| Fees are not serialized as a transaction field | FACT | Transaction structure plus `CheckTxInputs` fee computation |
| BIP141 defines transaction weight and virtual size | FACT | BIP141 |
| Bitcoin Core implements transaction weight using witness and non-witness serialization | FACT | Bitcoin Core `consensus/validation.h` |
| Minimum relay fee is policy, not consensus | FACT | Bitcoin Core release and policy documentation |
| Bitcoin Core 29.1 lowered default relay and incremental relay feerates to 100 sat/kvB | FACT | Bitcoin Core 29.1 release notes |
| Bitcoin Core 31.0 updated fee estimator minimum bucket to 0.1 sat/vB | FACT | Bitcoin Core 31.0 release notes |
| `estimatesmartfee` is probabilistic and based on observed data | FACT | RPC documentation and fee estimator source |
| CPFP changes incentive through combined package feerate | INTERPRETATION | Derived from fee-rate model and package relay documentation |
| High fee proves urgent intent | COUNTERCLAIM | Rejected; multiple causes can produce high fees |
| Fee estimate guarantees confirmation | COUNTERCLAIM | Rejected; estimates are probabilistic |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| POLICY | Relay, mempool, wallet, or mining convention rather than consensus |
| HEURISTIC | Practical rule with known counterexamples |
| UNKNOWN | Evidence is insufficient |

---

## 17. Knowledge Graph

```text
BITCOIN-018 Transaction Fees
|
+-- builds_on: BITCOIN-014 UTXO Model
+-- builds_on: BITCOIN-015 Transactions in Depth
+-- builds_on: BITCOIN-017 Mempool
|
+-- fee
|   +-- computed_as: inputs - outputs
|   +-- collected_by: coinbase
|
+-- size accounting
|   +-- weight: base_size * 3 + total_size
|   +-- vsize: ceil(weight / 4)
|
+-- fee rate
|   +-- unit: sat/vB
|   +-- used_by: relay policy, mining policy, fee estimation
|
+-- fee bumping
|   +-- RBF: replacement transaction
|   +-- CPFP: high-fee child package
|
+-- Bitcoin Core
|   +-- CheckTxInputs
|   +-- GetTransactionWeight
|   +-- CFeeRate
|   +-- CBlockPolicyEstimator
|   +-- estimatesmartfee
|
+-- analysis
    +-- facts: paid fee, vsize, fee rate
    +-- caveats: intent, urgency, miner preference
```

---

## 18. References

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Section 6, incentive and transaction fee discussion, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-bip-0125]: David A. Harding and Peter Todd, "BIP 125: Opt-in Full Replace-by-Fee Signaling," 2015-12-04, https://bips.dev/125/, accessed 2026-08-04.

[^ref-bip-0141]: Eric Lombrozo, Johnson Lau, and Pieter Wuille, "BIP 141: Segregated Witness (Consensus layer)," transaction size calculations, 2015-12-21, https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki, accessed 2026-08-04.

[^ref-btc-core-29-release]: Bitcoin Core Contributors, "Bitcoin Core 29.1 Release Notes," mempool policy changes for default `-minrelaytxfee` and `-incrementalrelayfee`, https://bitcoincore.org/en/releases/29.1/, accessed 2026-08-04.

[^ref-btc-core-31-release]: Bitcoin Core Contributors, "Bitcoin Core 31.0 Release Notes," package relay, fee estimator, wallet fee-setting, and cluster mempool notes, https://bitcoin.org/en/releases/31.0/, accessed 2026-08-04.

[^ref-btc-core-tx-verify]: Bitcoin Core Contributors, `src/consensus/tx_verify.h` and `src/consensus/tx_verify.cpp`, `Consensus::CheckTxInputs`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/tx__verify_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-consensus-validation]: Bitcoin Core Contributors, `src/consensus/validation.h`, `GetTransactionWeight`, `GetBlockWeight`, and transaction input weight calculation, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/consensus_2validation_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-feerate]: Bitcoin Core Contributors, `src/policy/feerate.h` and `src/policy/feerate.cpp`, `CFeeRate`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/feerate_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-fee-estimator]: Bitcoin Core Contributors, `src/policy/fees/block_policy_estimator.h` and `src/policy/fees/block_policy_estimator.cpp`, `CBlockPolicyEstimator`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/class_c_block_policy_estimator.html and https://doxygen.bitcoincore.org/block__policy__estimator_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-policy]: Bitcoin Core Contributors, `src/policy/policy.h` and `src/policy/policy.cpp`, relay fee, dust, standardness, and mining policy defaults, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/policy_8h.html, accessed 2026-08-04.

[^ref-btc-core-rbf]: Bitcoin Core Contributors, `src/policy/rbf.cpp`, `IsRBFOptIn` and replacement fee checks, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/policy_2rbf_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-rpc-fee]: Bitcoin Core RPC documentation, "`estimatesmartfee` RPC," fee estimate behavior, confirmation target, virtual size note, and BTC/kvB output, https://bitcoincore.org/en/doc/26.0.0/rpc/util/estimatesmartfee/, accessed 2026-08-04.

---

## 19. Cross References

### Parent

- Bitcoin Internals

### Previous

- BITCOIN-017 — Mempool

### Next

- BITCOIN-019 — Wallets and Key Management

### Related

- BITCOIN-007 — Whitepaper Section 6 — Incentive
- BITCOIN-014 — UTXO Model
- BITCOIN-015 — Transactions in Depth
- BITCOIN-017 — Mempool
- BITCOIN-019 — Wallets and Key Management
- POW-009 — Coinbase Transaction and Mining Commitments
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- Fee calculation was tied to UTXO values and `CheckTxInputs`.
- Weight and vsize formulas were checked against BIP141 and Bitcoin Core `GetTransactionWeight`.
- Consensus, relay policy, mining policy, and wallet behavior were separated.
- RBF and CPFP were described as unconfirmed fee-management mechanisms, not consensus guarantees.
- Bitcoin Core 29.1 and 31.0 fee-policy changes were dated and treated as current-version policy, not timeless consensus.

### Evidence Review

Passed.

- Fee formula claims cite Bitcoin Core validation source.
- Weight/vsize claims cite BIP141 and Bitcoin Core consensus validation source.
- Current relay and fee-estimator claims cite Bitcoin Core release notes.
- Fee estimation claims cite Bitcoin Core estimator source and RPC documentation.
- Intent and urgency claims are labeled as interpretation or rejected overclaim.

### Editorial Review

Passed.

- Markdown headings follow the project deep-dive structure.
- Metadata is complete.
- Tables and code fences are closed.
- Terminology is consistent: fee, fee rate, weight, vsize, relay policy, mining policy, RBF, CPFP.

### Adversarial Review

Passed.

- The document does not claim fees are serialized transaction fields.
- It does not treat fee estimates as guarantees.
- It does not conflate minimum relay fee with consensus validity.
- It does not infer user intent from high fees alone.
- It distinguishes absolute fee from fee rate and package context.

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
