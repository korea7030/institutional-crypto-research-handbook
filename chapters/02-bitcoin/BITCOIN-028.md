---
knowledge_id: BITCOIN-028
title: Security Budget
subtitle: Miner Compensation, Subsidy-and-Fee Revenue, Attack-Cost Proxies, Confirmation Security, and the Limits of Simple Revenue Metrics
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 125 min
estimated_study: 360 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Economics
  - Security
  - Mining
parent:
  - Bitcoin Economics
prerequisites:
  - BITCOIN-007
  - BITCOIN-020
  - BITCOIN-025
  - BITCOIN-026
  - BITCOIN-027
  - POW-011
  - POW-014
related_topics:
  - Block Subsidy
  - Fee Market
  - Hashrate
  - Attack Cost
  - Confirmation Security
  - Chain Reorganization
  - Mining Economics
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-BLOCKCHAIN-001
  - REF-BTC-DEV-OPERATING-MODES-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-BLOCKASSEMBLER-001
  - REF-BTC-CORE-CHAINPARAMS-001
  - REF-BTC-CORE-AMOUNT-001
tags:
  - bitcoin
  - economics
  - security-budget
  - mining
  - subsidy
  - fees
  - hashrate
  - confirmations
---

# Security Budget
> Bitcoin Economics  
> Research Unit: BITCOIN-028

---

## Research Brief

```yaml
knowledge_id: BITCOIN-028
title: Security Budget
research_question: >
  What does "security budget" mean in Bitcoin, how should analysts relate miner
  compensation from subsidy and fees to confirmation security and attack cost,
  and where are the boundaries between protocol facts, economic proxies, and
  claims that the network is or is not sufficiently secure?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-007
  - BITCOIN-020
  - BITCOIN-025
  - BITCOIN-026
  - BITCOIN-027
  - POW-011
  - POW-014
parent: Bitcoin Economics
previous: BITCOIN-027
next: BITCOIN-029
related_topics:
  - Miner Revenue
  - Reorg Risk
  - Double-Spend Resistance
  - Cumulative Work
  - Fee Share
  - Hashprice
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
  - Precise real-time energy market modeling
  - Full industrial-mining cost accounting
  - Nation-state strategic scenarios
  - Altchain security-budget comparisons
  - Deterministic valuation models
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Define Bitcoin's security budget as miner compensation available to support honest proof-of-work participation.
- Distinguish nominal BTC-denominated reward from real economic attack resistance.
- Explain why subsidy plus fees is a useful but incomplete proxy for security.
- Explain how confirmations and cumulative work relate to replacement cost.
- Explain why fee share and total security budget are different metrics.
- Explain why attack cost depends on more than just one block's reward.
- Separate protocol-determined quantities from market-determined quantities in security analysis.

---

## 2. Key Questions

1. What is the Bitcoin security budget?
2. Why is miner compensation central to security?
3. Is the security budget just subsidy plus fees?
4. Why does a higher security budget not mechanically imply perfect security?
5. Why does a high fee share not automatically imply adequate security?
6. How do confirmations relate to the economic cost of attack?
7. What can consensus guarantee, and what must markets supply?
8. What should institutions measure when evaluating Bitcoin security conditions?

---

## 3. Executive Summary

In Bitcoin analysis, "security budget" usually means the compensation available to miners from valid blocks, typically measured as block subsidy plus transaction fees on the active chain. This is a useful starting point because honest proof-of-work security depends on sustained miner participation, and miner participation depends on compensation relative to cost.[^ref-btc-wp][^ref-btc-core-validation][^ref-btc-core-blockassembler]

However, security budget is not the same thing as guaranteed security. The protocol can determine nominal block reward quantities, but it cannot determine:

- miner cost structure,
- hardware availability,
- electricity price,
- hash-rate centralization,
- BTC market value,
- attacker motivation,
- or whether compensation is sufficient at a given moment.

So the correct framing is:

- subsidy and fees are protocol-visible revenue components,
- cumulative work and confirmations are security evidence,
- attack cost is an economic inference,
- security sufficiency is a judgment, not a consensus field.

As subsidy declines across eras, fees become more important to the budget. But a rising fee share alone does not prove that the network is well funded. Analysts need both relative and absolute measures.

---

## 4. Protocol Structure

### Security Budget as Miner Compensation

At the simplest level:

```text
security_budget_per_block = subsidy + transaction_fees
```

This is not a line of code named "security budget." It is an analytical identity built from protocol-visible miner revenue components.[^ref-btc-core-validation][^ref-btc-core-blockassembler]

### Why It Matters

Bitcoin security comes from honest miners extending the valid chain with proof of work. If honest mining is economically well supported, replacing recent history becomes more expensive in expectation. If compensation falls relative to cost, the network may still function, but the security margin may narrow.

### Active-Chain Qualification

Only rewards on the active chain count toward realized miner compensation. A stale block may have been valid when found, but its subsidy and fees do not become spendable main-chain revenue if the block loses the fork race.

---

## 5. Revenue Components

### Subsidy

The subsidy is newly issued BTC created according to the height-based issuance schedule enforced by consensus.[^ref-btc-core-validation][^ref-btc-core-chainparams]

### Fees

Fees are existing BTC transferred from transaction spenders to the miner through the coinbase claim ceiling. They do not create new supply, but they do increase miner compensation.[^ref-btc-dev-blockchain][^ref-btc-core-validation]

### Block Reward

The coinbase claim ceiling for a valid block is:

```text
max_coinbase_value = subsidy + included_fees
```

So block reward is the protocol revenue opportunity for a mined block, while security budget is the broader analytical use of that revenue opportunity across time and across the active chain.

---

## 6. Technical Mechanics

### Coinbase Reward Enforcement

Bitcoin Core exposes `GetBlockSubsidy` for the subsidy component, and block validation enforces that coinbase value cannot exceed subsidy plus fees.[^ref-btc-core-validation]

### Fee Realization in Block Assembly

Bitcoin Core's block assembly tracks accumulated fees while constructing a candidate block. `BlockAssembler` includes `nFees` while adding selected transactions to the template, showing how fee revenue enters miner economics operationally.[^ref-btc-core-blockassembler]

### Amount Bounds

`CAmount`, `COIN`, and `MoneyRange` define monetary ranges and integer accounting primitives used across validation logic. These amount primitives matter for correct reward accounting, but they do not by themselves decide whether security is adequate.[^ref-btc-core-amount]

### Confirmation Security Link

The whitepaper and operating-modes documentation both connect security to the work built on top of a transaction's block. More confirmations mean more cumulative work for an attacker to overtake, making reversal more expensive in expectation.[^ref-btc-wp][^ref-btc-dev-operating-modes]

---

## 7. Security Assumptions and Failure Modes

### Revenue Is a Proxy, Not a Guarantee

A high nominal security budget improves incentives for honest mining, but security depends on:

- total and distributed hash rate,
- miner concentration,
- liquidity and financing of potential attackers,
- short-term opportunity costs,
- direct or indirect censorship incentives,
- and the depth of the transaction being attacked.

### Subsidy Decline

As halving reduces subsidy, the system leans more on fees for compensation. That is a structural transition, not a proof of weakness by itself. The relevant question is whether total compensation remains high enough relative to attack opportunity.

### Centralization Risk

Even if aggregate miner revenue looks large, security can still weaken if hash power becomes overly concentrated or strategically aligned. Revenue quantity and revenue distribution are different risk dimensions.

### Fee Volatility

A fee-dominated security budget may be more volatile than a subsidy-dominated one. Volatile compensation can create periods of stronger and weaker effective security even if long-run averages look acceptable.

---

## 8. Mathematical or Economic Model

### Per-Block Budget

Let:

- `S` = subsidy,
- `F` = included transaction fees.

Then:

```text
B_block = S + F
```

where `B_block` is the nominal per-block security-budget proxy.

### Interval Budget

For an interval of accepted blocks:

```text
B_interval = sum(S_i + F_i)
```

This is useful for daily, weekly, or era-based comparisons.

### Fee Share

```text
fee_share = F / (S + F)
```

Fee share measures composition, not adequacy. A 60 percent fee share on a small total reward can still imply weak total compensation.

### Confirmation and Replacement Cost

If a target transaction sits under cumulative public work `W_pub` after its inclusion block, then a successful replacement attack generally requires a valid alternative branch whose cumulative work overtakes that public branch from a relevant fork point:

```text
W_alt > W_pub
```

This is why confirmations are evidence of rising replacement cost. But work difference is still not a full attack-cost model, because costs and opportunities are external economic variables.

---

## 9. Validation Boundaries

### What Consensus Knows

Consensus can determine:

- valid subsidy at height,
- fees in a confirmed block,
- whether coinbase overclaims,
- cumulative work on known branches,
- confirmation depth on the active chain.

### What Consensus Does Not Know

Consensus does not know:

- electricity prices,
- ASIC depreciation,
- miner debt burdens,
- attacker financing,
- regulatory coercion,
- or whether a particular compensation level is "enough."

### Implication

Security-budget analysis is necessarily a hybrid of protocol facts and economic interpretation.

---

## 10. Bitcoin Core Implementation

### `validation`

`validation.h` exposes `GetBlockSubsidy`, which defines the subsidy component of miner compensation under consensus rules.[^ref-btc-core-validation]

### `BlockAssembler`

`node::BlockAssembler` maintains `nFees` while constructing a candidate block, reflecting the fee component of miner revenue in block production logic.[^ref-btc-core-blockassembler]

### `kernel/chainparams`

Chain parameters determine the subsidy-halving interval and therefore the long-run subsidy path that shapes the security-budget transition from subsidy-heavy to fee-heavier regimes.[^ref-btc-core-chainparams]

### `consensus/amount.h`

`CAmount`, `COIN`, and range checking are the monetary accounting substrate used when reasoning about valid rewards and amounts.[^ref-btc-core-amount]

---

## 11. On-Chain Implications

### Directly Measurable

From active-chain data, analysts can directly compute:

- subsidy per block,
- fees per block,
- block reward totals,
- fee share,
- daily or epochal aggregate miner revenue in BTC,
- confirmation depth and chainwork progress.

### Not Directly Measurable On-Chain

On-chain data usually cannot directly measure:

- miner operating costs,
- hedging strategy,
- effective profit margins,
- attacker capital access,
- off-chain side payments,
- miner concentration by beneficial ownership.

### Reporting Standard

A serious security-budget report should distinguish:

- BTC-denominated reward,
- fiat-denominated reward,
- fee share,
- realized vs projected budget,
- chainwork or confirmation evidence,
- and attack-cost interpretation confidence.

---

## 12. Institutional Thinking

Institutions should avoid treating "security budget" as a magic scalar.

### Practical Implications

- Use subsidy, fees, and fee share together, not interchangeably.
- Report both composition and absolute magnitude of miner compensation.
- Tie confirmation policy to work and risk tolerance, not folklore alone.
- Monitor reorgs, stale rates, and miner concentration alongside revenue metrics.
- State clearly when a conclusion depends on off-chain cost assumptions.

### Better Security Framing

A disciplined internal model often needs at least three layers:

- protocol layer: subsidy, fees, confirmations, chainwork;
- market layer: BTC price, fee demand, hash-rate response;
- adversarial layer: concentration, attack incentives, replacement opportunity.

---

## 13. Common Misinterpretations

### "Security budget equals security"

False. It is a proxy for miner compensation, not a complete security proof.

### "High fee share means Bitcoin is safe without subsidy"

False. Fee share is relative composition, not total economic sufficiency.

### "Low subsidy means immediate insecurity"

False. Security depends on total compensation and attack cost, not subsidy in isolation.

### "Six confirmations is a protocol guarantee"

False. It is a practical convention grounded in cost intuition, not a consensus constant.[^ref-btc-dev-operating-modes]

### "On-chain revenue metrics fully reveal attack cost"

False. Attack cost depends on many off-chain variables.

---

## 14. Research Questions

1. Which combination of BTC-denominated reward, fiat reward, and chainwork metrics best explains observed reorg resistance?
2. How much does fee volatility increase short-horizon security-budget uncertainty?
3. How should institutions incorporate miner concentration into security-budget dashboards?
4. How should confirmation policy change under fee-dominated versus subsidy-dominated compensation regimes?
5. What leading indicators best signal that nominal miner compensation is decoupling from effective attack cost?

---

## 15. Practical Exercises

### Exercise 1

Compute daily miner compensation from a recent block sample and split it into subsidy and fees.

### Exercise 2

Compare two periods with similar fee share but different total BTC-denominated reward and explain why their security implications may differ.

### Exercise 3

For a target transaction depth, describe how cumulative work and confirmation count inform replacement-cost reasoning without claiming exact attack probability.

### Exercise 4

Draft a security-budget dashboard schema that separates protocol facts from inferred economic variables.

---

## 16. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Subsidy and fee reward components | Directly specified | Whitepaper, dev docs, and Core validation / block assembly |
| Confirmation and cumulative-work security logic | Directly specified | Whitepaper and operating-modes guide |
| Security-budget-as-proxy framing | Inference from sources | Standard analytical interpretation of miner compensation |
| Sufficiency and attack-cost claims | Economic inference | Require off-chain assumptions beyond consensus |

---

## 17. Knowledge Graph

```text
Security Budget
├─ Revenue Components
│  ├─ subsidy
│  ├─ fees
│  └─ block reward
├─ Security Evidence
│  ├─ confirmations
│  ├─ cumulative work
│  └─ active-chain persistence
├─ Economic Interpretation
│  ├─ fiat value
│  ├─ miner costs
│  ├─ fee share
│  └─ attack cost
├─ Risks
│  ├─ subsidy decline
│  ├─ fee volatility
│  ├─ concentration
│  └─ reorg exposure
└─ Implementation
   ├─ GetBlockSubsidy
   ├─ BlockAssembler.nFees
   ├─ chainparams
   └─ amount accounting
```

---

## 18. References

### Primary Sources

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," especially Sections 6 and 11. https://bitcoin.org/bitcoin.pdf

[^ref-btc-dev-blockchain]: Bitcoin Developer Reference, "Block Chain," including subsidy and fee description. https://developer.bitcoin.org/reference/block_chain.html

[^ref-btc-dev-operating-modes]: Bitcoin Developer Guide, "Operating Modes," including full-node and SPV security discussion. https://developer.bitcoin.org/devguide/operating_modes.html

[^ref-btc-core-validation]: Bitcoin Core Doxygen, `validation.h`, including `GetBlockSubsidy`. https://doxygen.bitcoincore.org/validation_8h_source.html

[^ref-btc-core-blockassembler]: Bitcoin Core Doxygen, `node::BlockAssembler`, including fee accumulation via `nFees`. https://doxygen.bitcoincore.org/classnode_1_1_block_assembler.html

[^ref-btc-core-chainparams]: Bitcoin Core Doxygen, `kernel/chainparams.cpp`, including subsidy-halving parameters. https://doxygen.bitcoincore.org/kernel_2chainparams_8cpp_source.html

[^ref-btc-core-amount]: Bitcoin Core Doxygen, `consensus/amount.h`, including `CAmount`, `COIN`, and `MoneyRange`. https://doxygen.bitcoincore.org/amount_8h_source.html

### Supporting Interpretation Notes

- Where this document discusses sufficiency, attack cost, miner margin, or concentration-adjusted security, those claims are analytical inferences that combine protocol-visible quantities with off-chain economic assumptions.

---

## 19. Cross References

### Previous

- BITCOIN-027 — Fee Market

### Next

- BITCOIN-029 — Bitcoin Game Theory

### Related

- BITCOIN-007 — Whitepaper Section 6 — Incentive
- BITCOIN-012 — Whitepaper Section 11 — Calculations
- BITCOIN-020 — Mining
- BITCOIN-024 — Chain Reorganization
- BITCOIN-025 — Bitcoin Monetary Policy
- BITCOIN-026 — Halving
- BITCOIN-027 — Fee Market
- BITCOIN-029 — Bitcoin Game Theory
- POW-011 — Cumulative Chainwork
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- Subsidy, fees, block reward, confirmations, and attack-cost interpretation were separated.
- Security budget was treated as an analytical proxy rather than a protocol field.
- Confirmation security was connected to cumulative work without overstating exact attack probabilities.
- Implementation references were limited to reward-accounting and block-assembly surfaces relevant to miner compensation.

### Evidence Review

Passed.

- Whitepaper and developer documentation support the reward and confirmation-security foundations.
- Core validation and block-assembly references support subsidy and fee accounting claims.
- Economic sufficiency claims are explicitly labeled as interpretation.

### Editorial Review

Passed.

- Structure follows the project deep-dive format.
- Metadata is complete.
- Terminology is consistent: security budget, subsidy, fees, fee share, confirmations, cumulative work.
- Tables and code fences are closed.

### Adversarial Review

Passed.

- The document does not equate security budget with complete security.
- It does not claim fee share alone proves adequacy.
- It does not turn six confirmations into a consensus constant.
- It does not claim on-chain data alone reveals full attack cost.
- It does not ignore concentration risk.

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
