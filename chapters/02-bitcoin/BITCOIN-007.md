---
knowledge_id: BITCOIN-007
title: Whitepaper Section 6 — Incentive
subtitle: Coinbase Reward, Subsidy Schedule, Transaction Fees, Miner Economics, and Security Budget
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 80 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Incentives
  - Mining
  - Monetary Policy
parent:
  - Bitcoin Whitepaper
prerequisites:
  - BITCOIN-005
  - BITCOIN-006
  - POW-009
  - POW-014
related_topics:
  - Coinbase Transaction
  - Block Subsidy
  - Transaction Fees
  - Security Budget
  - Miner Incentives
  - Monetary Issuance
  - Fee Market
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-TRANSACTIONS-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-MINER-001
  - REF-BTC-CORE-CONSENSUS-001
  - REF-BTC-CORE-AMOUNT-001
  - REF-BTC-CORE-CHAINPARAMS-001
tags:
  - bitcoin
  - whitepaper
  - incentive
  - mining
  - coinbase
  - subsidy
  - fees
  - security-budget
---

# Whitepaper Section 6 — Incentive
> Deep Dive Series  
> Research Unit: BITCOIN-007

---

## Research Brief

```yaml
knowledge_id: BITCOIN-007
title: Whitepaper Section 6 — Incentive
research_question: >
  How does Bitcoin use the coinbase transaction, block subsidy, and
  transaction fees to incentivize miners, distribute new bitcoin, and
  fund proof-of-work security over time?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-005
  - BITCOIN-006
  - POW-009
parent: Bitcoin Whitepaper
previous: BITCOIN-006
next: BITCOIN-008
related_topics:
  - Coinbase Transaction
  - Block Subsidy
  - Transaction Fees
  - Coinbase Maturity
  - Security Budget
  - Fee Market
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
  - Mining pool payout accounting
  - ASIC manufacturing economics
  - Complete fee-estimation algorithms
  - Long-term BTC price forecasting
  - Non-Bitcoin reward designs
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Explain the incentive role of the first transaction in a block.
- Distinguish block subsidy from transaction fees.
- Explain why the subsidy is newly issued bitcoin while fees are transferred value.
- Explain why miners cannot choose arbitrary block rewards.
- Trace how Bitcoin Core computes the subsidy schedule.
- Explain why a coinbase transaction can underclaim but cannot overclaim the permitted reward.
- Explain why coinbase outputs must mature before being spent.
- Explain how declining subsidy shifts more security-budget pressure onto fees.
- Separate consensus reward rules from mining-pool payout policies.
- Analyze miner revenue from on-chain data without overclaiming miner identity or profitability.

---

## 2. Key Questions

1. What does the whitepaper mean by "the first transaction in a block"?
2. How does the coinbase transaction incentivize miners?
3. What is the difference between subsidy and transaction fees?
4. Why is new issuance analogous to gold mining in the whitepaper?
5. How does Bitcoin Core calculate the block subsidy?
6. How does Bitcoin Core enforce the maximum coinbase reward?
7. Why can a miner claim less than the maximum reward?
8. Why do coinbase outputs have a maturity period?
9. What changes as the subsidy declines?
10. Does fee revenue alone guarantee future security?
11. What can analysts infer from coinbase outputs?
12. What cannot be inferred from coinbase outputs alone?

---

## 3. Executive Summary

Whitepaper Section 6 introduces Bitcoin's incentive mechanism. The first transaction in a block is a special transaction that creates new coins payable to the block creator. This rewards miners for supporting the network and distributes new bitcoin without a central issuer.[^ref-btc-wp]

The section also introduces transaction fees as a second revenue source. If a transaction's input value exceeds its output value, the difference is a fee that can be claimed by the miner of the block containing that transaction.[^ref-btc-wp]

Modern Bitcoin expresses this incentive through the coinbase transaction. The maximum permitted coinbase output value is:

```text
maximum coinbase claim = block subsidy at height + sum(included transaction fees)
```

Bitcoin Core computes the subsidy with `GetBlockSubsidy`, using a starting subsidy of `50 * COIN`, the chain's subsidy halving interval, and a right shift by the number of completed halvings. Mainnet sets `nSubsidyHalvingInterval = 210000`.[^ref-btc-core-validation][^ref-btc-core-chainparams]

Bitcoin Core enforces the reward limit during block connection. After fees are calculated, it rejects a block with `bad-cb-amount` if the coinbase output value exceeds `nFees + GetBlockSubsidy(height, consensusParams)`.[^ref-btc-core-validation]

The incentive design has two separate effects:

1. **Issuance:** the subsidy creates new bitcoin according to consensus rules.
2. **Security budget:** subsidy plus fees are the revenue available to miners whose blocks are accepted into the active chain.

As the subsidy declines over time, fees become more important to miner revenue. This does not mean fees automatically become sufficient. Fee-based security depends on block-space demand, fee-market behavior, miner cost structure, BTC price, transaction patterns, and attack incentives. Those are economic variables, not direct consensus guarantees.

---

## 4. Original Source

Whitepaper Section 6 makes four core claims:

1. The first transaction in a block is special.
2. That transaction starts a new coin owned by the block creator.
3. This provides an incentive to support the network and initially distribute coins.
4. Transaction fees can eventually fund the incentive after predetermined issuance ends.[^ref-btc-wp]

The whitepaper also states an incentive argument: an attacker with more CPU power than honest nodes must choose between using that power to defraud people or using it to generate new coins. The text argues that honest mining should be more profitable than undermining the system and the attacker's own wealth.[^ref-btc-wp]

That argument is economic, not purely cryptographic. Proof of Work makes block creation costly. The reward makes honest block creation valuable. Bitcoin's security model needs both.

---

## 5. Literal Interpretation

### "By convention, the first transaction in a block is a special transaction"

The whitepaper describes the special first transaction that modern Bitcoin calls the coinbase transaction. Bitcoin Developer documentation defines the first transaction in a block as the coinbase transaction and describes the coinbase input as having no previous outpoint.[^ref-btc-dev-transactions]

This is not an ordinary payment. A normal transaction spends previous outputs. A coinbase transaction creates permitted outputs under consensus reward rules.

### "Starts a new coin owned by the creator of the block"

The whitepaper's phrase is conceptual. In modern UTXO terms, the coinbase transaction creates one or more new UTXOs. Those outputs are not valid because the miner says so. They are valid only if the block satisfies consensus rules and the coinbase value does not exceed the permitted subsidy plus fees.

### "The steady addition of a constant amount of new coins"

The whitepaper presents the reward as a steady issuance mechanism. Bitcoin Core's current rules implement a declining subsidy schedule rather than an unchanging subsidy forever: the initial subsidy is 50 BTC, and it halves every 210,000 blocks on mainnet.[^ref-btc-core-validation][^ref-btc-core-chainparams]

### "The incentive can also be funded with transaction fees"

A transaction fee is the difference between input value and output value. That difference is not assigned to any normal output. It becomes available to the miner through the coinbase reward limit.

---

## 6. Technical Interpretation

### Coinbase Reward Components

The block reward has two components:

| Component | Source | Creates New BTC? | Consensus Limit |
|---|---|---:|---|
| Block subsidy | Protocol issuance schedule | Yes | `GetBlockSubsidy(height, params)` |
| Transaction fees | Inputs minus outputs of included transactions | No | Sum of valid included fees |

The subsidy increases total issued supply. Fees redistribute existing bitcoin from transaction spenders to the miner.

### Maximum Claim Rule

For block height `h`:

```text
subsidy = GetBlockSubsidy(h, consensusParams)
fees = sum(input_value(tx) - output_value(tx)) for included non-coinbase txs
max_coinbase_value = subsidy + fees
```

If:

```text
coinbase_output_value > max_coinbase_value
```

then the block is invalid.

If:

```text
coinbase_output_value <= max_coinbase_value
```

then the amount rule is satisfied, assuming all other consensus rules also pass.

This means underclaiming is allowed. A miner can leave part of the permitted reward unclaimed. Overclaiming is not allowed.

### Coinbase Maturity

Bitcoin also delays spendability of coinbase outputs. Bitcoin Core defines `COINBASE_MATURITY = 100`, and its documentation states that coinbase transaction outputs can only be spent after that number of new blocks.[^ref-btc-core-consensus]

Coinbase maturity matters because reorgs can remove a block and therefore remove its coinbase output. A maturity period reduces the risk of spending rewards from blocks that later disappear from the active chain.

---

## 7. Protocol Structure

### Incentive Pipeline

```text
Miner selects valid transactions
        |
        v
Transaction fees are implied by inputs minus outputs
        |
        v
Miner constructs coinbase transaction
        |
        v
Coinbase may claim subsidy + included fees
        |
        v
Miner searches for valid Proof of Work
        |
        v
Block is broadcast
        |
        v
Nodes validate PoW, block structure, transactions, scripts, and reward amount
        |
        v
If accepted into active chain, coinbase output exists but remains immature
        |
        v
After maturity, coinbase output can be spent
```

### Consensus vs Policy vs Business Practice

| Category | Example | Consensus? |
|---|---|---:|
| Coinbase must be first transaction | `CheckBlock` enforces first transaction shape | Yes |
| Coinbase reward cannot exceed subsidy plus fees | `ConnectBlock` rejects `bad-cb-amount` | Yes |
| Coinbase maturity is 100 blocks | `COINBASE_MATURITY` | Yes |
| Miner chooses which transactions to include | Block construction behavior | No, within validity limits |
| Miner chooses payout address or script | Coinbase output construction | No, within validity limits |
| Pool distributes revenue to workers | Pool accounting | No |
| Exchange waits for N confirmations | Business policy | No |

### Incentive and Chain Selection

A miner earns the block reward only if its block survives in the active chain. A valid block that becomes stale normally does not pay spendable main-chain subsidy or fees.

This links the incentive section to the network section:

- fast block relay reduces stale risk;
- validation by peers determines whether others build on the block;
- cumulative work determines which valid branch wins;
- coinbase maturity delays final spendability of the reward.

---

## 8. Mathematical or Economic Model

### Subsidy Schedule

Bitcoin Core computes:

```text
halvings = height / nSubsidyHalvingInterval
if halvings >= 64:
    subsidy = 0
else:
    subsidy = 50 * COIN >> halvings
```

Mainnet sets:

```text
nSubsidyHalvingInterval = 210000
COIN = 100000000 satoshis
```

The first subsidy is therefore:

```text
50 * 100000000 = 5,000,000,000 satoshis
```

After four completed halvings, the subsidy is:

```text
50 BTC / 2^4 = 3.125 BTC
```

This is the current mainnet subsidy regime after height 840,000, assuming mainnet consensus parameters.

### Miner Expected Revenue

Let:

```text
alpha = miner share of effective network hash rate
S = block subsidy
F = transaction fees in the block
p_stale = probability the block becomes stale
```

A simplified expected per-block opportunity is:

```text
expected_reward_share ~= alpha * (S + F) * (1 - p_stale)
```

This is not a consensus formula. It is an economic approximation. Realized miner income depends on variance, pool payout rules, uptime, block propagation, transaction selection, fee volatility, hedging, electricity costs, financing, and hardware depreciation.

### Security Budget

For one accepted block:

```text
security_budget_per_block = subsidy + fees
```

For a time interval:

```text
security_budget_interval = sum(subsidy + fees for accepted blocks in interval)
```

This is a useful analyst measure because it estimates the revenue defending the chain through mining incentives. It is not the same as total mining cost. Miners may mine at a loss or profit depending on market conditions.

### Fee Transition

Whitepaper Section 6 says the incentive can transition entirely to transaction fees after predetermined coin issuance.[^ref-btc-wp]

This is a design claim and a long-term economic hypothesis:

- The consensus rules can reduce subsidy over time.
- The protocol can allow fees to compensate miners.
- The actual adequacy of fee revenue depends on future demand for block space and miner economics.

Do not treat "can transition" as proof that fee revenue will always be sufficient under every future demand regime.

---

## 9. Security Assumptions

### Incentive Alignment

The whitepaper's incentive argument depends on the idea that miners with large hashpower should prefer honest mining revenue to attacks that damage Bitcoin's value.[^ref-btc-wp]

This relies on several assumptions:

1. Mining rewards are economically meaningful.
2. Attack gains are lower than honest mining gains plus the value of preserving Bitcoin's credibility.
3. Miners have long-term exposure to Bitcoin or mining infrastructure.
4. Honest full nodes reject invalid blocks even if those blocks have costly Proof of Work.
5. The network converges on valid blocks quickly enough that honest mining is not excessively penalized by stale risk.

### Limits of the Incentive Argument

The incentive argument is not absolute:

- An attacker may value sabotage more than profit.
- A short-term attacker may not care about long-term BTC value.
- Borrowed or rented hashpower can change incentives.
- Mining pools can create principal-agent issues between hash owners and pool operators.
- Fee spikes can create temporary reorg incentives.
- Subsidy decline can change the relationship between security budget and attack cost.

These are economic and operational risks, not failures of the coinbase reward rule itself.

### Full Nodes as Constraint

Miners are paid only through valid blocks. A miner cannot compensate itself by exceeding the allowed coinbase value because full nodes reject the block. Bitcoin Core's `ConnectBlock` enforces this by comparing the coinbase output value against fees plus subsidy.[^ref-btc-core-validation]

This makes the incentive mechanism compatible with permissionless mining: anyone may try to mine, but everyone independently verifies the reward claim.

---

## 10. Bitcoin Core Implementation

### Subsidy Calculation

Bitcoin Core declares `GetBlockSubsidy` in `validation.h` and defines it in `validation.cpp`.[^ref-btc-core-validation]

The function:

1. Computes completed halvings as `nHeight / consensusParams.nSubsidyHalvingInterval`.
2. Returns zero if `halvings >= 64`.
3. Starts from `50 * COIN`.
4. Right-shifts by `halvings`.
5. Returns the resulting subsidy.[^ref-btc-core-validation]

Mainnet consensus parameters set `nSubsidyHalvingInterval = 210000`.[^ref-btc-core-chainparams]

### Coinbase Construction

Bitcoin Core's block assembler constructs a candidate coinbase transaction while creating a new block template. In `BlockAssembler::CreateNewBlock`, it creates a coinbase transaction and sets the coinbase output value to:

```text
nFees + GetBlockSubsidy(nHeight, chainparams.GetConsensus())
```

It also starts the coinbase scriptSig with the block height as required by BIP34.[^ref-btc-core-miner]

This is candidate block construction. It does not make the block valid by itself. The block still needs valid Proof of Work and must pass validation by peers.

### Coinbase Reward Validation

During block connection, Bitcoin Core calculates:

```text
blockReward = nFees + GetBlockSubsidy(pindex->nHeight, params.GetConsensus())
```

If the coinbase output value exceeds this value, the block is rejected with `bad-cb-amount`.[^ref-btc-core-validation]

### Amount Bounds

Bitcoin Core defines:

```text
COIN = 100000000
MAX_MONEY = 21000000 * COIN
```

and `MoneyRange` checks that amounts are not negative and do not exceed `MAX_MONEY`.[^ref-btc-core-amount]

This amount bound is not the same as the per-block subsidy schedule. It is a general validity bound for Bitcoin amounts. The per-block reward limit is separately enforced by subsidy plus fees.

### Coinbase Maturity

`COINBASE_MATURITY = 100` appears in `consensus.h`, and validation code uses coinbase maturity logic when evaluating whether coinbase outputs can be spent and when handling reorg-related mempool validity.[^ref-btc-core-consensus][^ref-btc-core-validation]

---

## 11. On-Chain Implications

### What Analysts Can Observe

From block and transaction data, analysts can observe:

- the coinbase transaction;
- the coinbase output value;
- the output scripts receiving the coinbase reward;
- the block height;
- the transaction list included in the block;
- the fee total, if spent inputs are known;
- whether the coinbase appears to underclaim;
- whether a coinbase output is mature enough to spend;
- coinbase metadata such as pool tags, with attribution caveats.

### What Analysts Cannot Directly Infer

From coinbase data alone, analysts cannot reliably infer:

- the real-world identity of the miner;
- the exact mining pool payout distribution;
- the miner's electricity cost;
- whether the miner was profitable;
- whether a pool tag is authentic;
- whether the block was found by the entity named in the tag;
- off-chain fee-sharing arrangements.

### Coinbase Value Check

A practical validation-oriented check:

```text
1. Get block height.
2. Compute subsidy for that height.
3. For every non-coinbase transaction:
   fee = sum(spent input values) - sum(outputs)
4. Sum all fees.
5. Compare coinbase output value with subsidy + fees.
6. If coinbase value is greater, the block is invalid.
7. If coinbase value is lower, the miner underclaimed.
```

Most analysts do not need to reimplement full validation for every workflow, but this model explains why coinbase value is not arbitrary issuance.

### Security Budget Metrics

Common analyst metrics:

| Metric | Formula | Use |
|---|---|---|
| Block reward | subsidy + fees | Per-block miner revenue ceiling |
| Fee share | fees / (subsidy + fees) | Fee-market importance |
| Daily security budget | sum(block rewards over day) | Miner revenue proxy |
| Annualized security budget | daily budget * 365 | Comparative security metric |
| Coinbase spend maturity | coinbase height + 100 | Spendability check |

These metrics should be denominated both in BTC and in fiat terms when assessing miner economics, because miner costs are often fiat-denominated while rewards are BTC-denominated.

---

## 12. Institutional Thinking

### Settlement and Miner Incentives

Institutions should understand that miner incentives are not abstract. They affect:

- confirmation reliability;
- fee requirements during congestion;
- reorg incentives after unusually high-fee blocks;
- mining centralization pressure;
- long-term security-budget debates;
- custody and exchange confirmation policies.

### Subsidy Decline

As subsidy declines, the network relies more heavily on transaction fees to compensate miners. This raises research questions:

- Will block-space demand produce sufficient fees?
- Will fee volatility increase short-term reorg incentives?
- Will miners become more sensitive to transaction selection?
- Will out-of-band payments to miners become more relevant?
- Will pool concentration affect transaction inclusion?

These are not settled by Section 6 alone. Section 6 establishes the mechanism; modern analysis must study fee-market data and miner behavior.

### Controls for Businesses

Institutional systems should:

- run independent full nodes;
- verify deposits against validated chain data;
- scale confirmation requirements by value and risk;
- monitor reorgs and stale-block conditions;
- monitor fee-market stress;
- avoid crediting high-value deposits solely from third-party API status;
- treat coinbase-based miner attribution as heuristic.

### Investment Research Lens

For Bitcoin research, Section 6 connects monetary policy to security:

```text
issuance schedule
    -> miner revenue
    -> hashpower incentive
    -> attack cost
    -> settlement confidence
    -> institutional usability
```

This chain is partly mechanical and partly economic. The subsidy schedule is consensus. The market value of that subsidy is not. Fee demand is emergent. Miner costs are external. Security-budget analysis must separate these layers.

---

## 13. Common Misinterpretations

### Misinterpretation 1: Miners create any amount they want.

Incorrect. Miners construct coinbase transactions, but full nodes reject blocks whose coinbase output value exceeds subsidy plus fees.[^ref-btc-core-validation]

### Misinterpretation 2: Transaction fees are newly created bitcoin.

Incorrect. Fees are existing bitcoin paid implicitly by transaction inputs exceeding transaction outputs. The subsidy is newly issued bitcoin.

### Misinterpretation 3: The coinbase reward is immediately spendable.

Incorrect. Coinbase outputs require 100-block maturity before they can be spent under Bitcoin Core consensus rules.[^ref-btc-core-consensus]

### Misinterpretation 4: A coinbase tag proves miner identity.

Incorrect. Coinbase metadata is self-reported or operationally inserted. It can help attribution but does not prove real-world identity without corroborating evidence.

### Misinterpretation 5: Fee transition is already solved by protocol design.

Overstated. The protocol permits a transition toward fee-funded mining incentives. Whether future fee revenue is sufficient is an economic question.

### Misinterpretation 6: Security budget equals miner cost.

Incorrect. Security budget is miner revenue available from accepted blocks. Miner cost depends on hardware, electricity, financing, operations, geography, and hedging.

---

## 14. Research Questions

1. How has the fee share of miner revenue changed across subsidy eras?
2. How volatile is fee revenue compared with subsidy revenue?
3. Do high-fee blocks increase short-term reorg incentives?
4. How concentrated are coinbase payout scripts among mining pools?
5. How reliable are coinbase tags compared with payout-address clustering?
6. How does subsidy decline affect small miner survival?
7. How should institutions set confirmation policies when fee volatility is high?
8. What fee-market conditions would materially change Bitcoin's security-budget profile?
9. How often do miners underclaim rewards, and why?
10. How does coinbase maturity affect reorg risk for miners and pools?

---

## 15. Practical Exercises

1. Given a block at height 840,001 with fees of `0.25 BTC`, compute the maximum valid coinbase output value.
2. Explain why a coinbase output from height `900,000` should not be spent in a block at height `900,050`.
3. Find a recent block and compute fee share:

```text
fee_share = fees / (subsidy + fees)
```

4. Compare coinbase tag attribution with payout-output clustering for the same block.
5. Explain why a stale block's coinbase transaction does not create spendable main-chain bitcoin.

---

## 16. Evidence Classification

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Whitepaper Section 6 incentive design | A |
| REF-BTC-DEV-TRANSACTIONS-001 | Official developer documentation | Coinbase input structure and transaction format | A |
| REF-BTC-CORE-VALIDATION-001 | Primary implementation source | `GetBlockSubsidy`, `ConnectBlock`, `bad-cb-amount` | A |
| REF-BTC-CORE-MINER-001 | Primary implementation source | `BlockAssembler::CreateNewBlock` coinbase construction | A |
| REF-BTC-CORE-CONSENSUS-001 | Primary implementation source | `COINBASE_MATURITY` consensus constant | A |
| REF-BTC-CORE-AMOUNT-001 | Primary implementation source | `COIN`, `MAX_MONEY`, `MoneyRange` | A |
| REF-BTC-CORE-CHAINPARAMS-001 | Primary implementation source | Mainnet `nSubsidyHalvingInterval` | A |

### Claim Ledger

| Claim ID | Claim | Classification | Evidence | Source |
|---|---|---|---|---|
| C001 | Whitepaper Section 6 defines the first transaction in a block as a special reward transaction. | FACT | A | REF-BTC-WP-001 |
| C002 | Transaction fees are the difference between transaction inputs and outputs. | FACT | A | REF-BTC-WP-001 |
| C003 | Bitcoin Core computes subsidy with `GetBlockSubsidy`. | FACT | A | REF-BTC-CORE-VALIDATION-001 |
| C004 | Mainnet sets `nSubsidyHalvingInterval` to 210,000. | FACT | A | REF-BTC-CORE-CHAINPARAMS-001 |
| C005 | Bitcoin Core rejects excessive coinbase output value as `bad-cb-amount`. | FACT | A | REF-BTC-CORE-VALIDATION-001 |
| C006 | Bitcoin Core's block assembler constructs candidate coinbase output value as fees plus subsidy. | FACT | A | REF-BTC-CORE-MINER-001 |
| C007 | Coinbase outputs require 100-block maturity. | FACT | A | REF-BTC-CORE-CONSENSUS-001 |
| C008 | Fees redistribute existing BTC while subsidy issues new BTC. | INTERPRETATION | A | REF-BTC-WP-001; REF-BTC-CORE-VALIDATION-001 |
| C009 | Future fee adequacy is an economic question rather than a direct consensus guarantee. | INTERPRETATION | B | REF-BTC-WP-001 |
| C010 | Coinbase metadata alone does not prove real-world miner identity. | INTERPRETATION | B | REF-BTC-DEV-TRANSACTIONS-001 |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| HEURISTIC | Practical research rule requiring context |
| OPEN | Unresolved or future-dependent question |

---

## 17. Knowledge Graph

```text
BITCOIN-007 Incentive
|
+-- interprets: Whitepaper Section 6
|
+-- uses: Coinbase Transaction
|   +-- first transaction in block
|   +-- creates permitted reward outputs
|   +-- matures after 100 blocks
|
+-- reward = Subsidy + Fees
|   +-- subsidy: new issuance
|   +-- fees: existing BTC transfer
|
+-- enforced_by: Bitcoin Core
|   +-- GetBlockSubsidy(height, params)
|   +-- ConnectBlock bad-cb-amount check
|   +-- COINBASE_MATURITY
|
+-- affects: Security Budget
|   +-- miner revenue
|   +-- hashpower incentive
|   +-- fee-market dependence
|
+-- leads_to: BITCOIN-008 Reclaiming Disk Space
```

---

## 18. References

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Section 6, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-transactions]: Bitcoin Developer Documentation, "Transactions — Coinbase Input: The Input Of The First Transaction In A Block," https://developer.bitcoin.org/reference/transactions.html, accessed 2026-08-04.

[^ref-btc-core-validation]: Bitcoin Core Contributors, `src/validation.cpp`, functions `GetBlockSubsidy`, `ConnectBlock`, and coinbase reward validation, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/validation_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-miner]: Bitcoin Core Contributors, `src/node/miner.cpp`, `BlockAssembler::CreateNewBlock` coinbase construction, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/miner_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-consensus]: Bitcoin Core Contributors, `src/consensus/consensus.h`, `COINBASE_MATURITY`, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/consensus_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-amount]: Bitcoin Core Contributors, `src/consensus/amount.h`, `COIN`, `MAX_MONEY`, and `MoneyRange`, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/amount_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-chainparams]: Bitcoin Core Contributors, `src/kernel/chainparams.cpp`, mainnet consensus parameters including `nSubsidyHalvingInterval`, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/kernel_2chainparams_8cpp_source.html, accessed 2026-08-04.

---

## 19. Cross References

### Parent

- Bitcoin Whitepaper

### Previous

- BITCOIN-006 — Whitepaper Section 5 — Network

### Next

- BITCOIN-008 — Whitepaper Section 7 — Reclaiming Disk Space

### Related

- BITCOIN-024 — Bitcoin Monetary Policy
- BITCOIN-026 — Fee Market
- BITCOIN-027 — Security Budget
- POW-008 — Bitcoin Mining
- POW-009 — Coinbase Transaction and Mining Commitments
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- Whitepaper Section 6 was separated from later implementation details.
- Subsidy, fees, coinbase construction, reward validation, and maturity were separated.
- `GetBlockSubsidy`, `BlockAssembler::CreateNewBlock`, `ConnectBlock`, `COINBASE_MATURITY`, `COIN`, and `MAX_MONEY` were checked against Bitcoin Core source.
- Underclaiming and overclaiming behavior were distinguished.

### Evidence Review

Passed.

- Whitepaper claims cite Section 6 directly.
- Current consensus and implementation claims cite Bitcoin Core source.
- Coinbase structure claims cite official Bitcoin Developer documentation.
- Economic statements about fee adequacy and security budget are labeled as interpretation.

### Editorial Review

Passed.

- Markdown headings follow the project deep-dive structure.
- Metadata is complete.
- Tables and code fences are closed.
- Terminology is consistent: subsidy, fees, coinbase, block reward, security budget, maturity.

### Adversarial Review

Passed.

- The document does not imply miners can create arbitrary bitcoin.
- The document does not imply fees are newly issued coins.
- The document does not treat fee-market sufficiency as guaranteed.
- Coinbase metadata attribution caveats are included.

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
