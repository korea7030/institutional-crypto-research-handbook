---
knowledge_id: BITCOIN-012
title: Whitepaper Section 11 — Calculations
subtitle: Attacker Catch-Up Probability, Confirmations, Poisson Approximation, and Probabilistic Settlement
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 95 min
estimated_study: 260 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Probability
  - Security
  - Proof of Work
parent:
  - Bitcoin Whitepaper
prerequisites:
  - BITCOIN-005
  - BITCOIN-009
  - BITCOIN-010
  - POW-010
  - POW-011
  - POW-014
related_topics:
  - Double Spend
  - Confirmations
  - Probabilistic Finality
  - Poisson Distribution
  - Chainwork
  - Settlement Risk
  - Attacker Hashrate
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-BLOCKCHAIN-001
  - REF-BTC-DEV-PAYMENT-PROCESSING-001
  - REF-BTC-DEV-OPERATING-001
tags:
  - bitcoin
  - whitepaper
  - calculations
  - confirmations
  - double-spend
  - probability
  - settlement-risk
---

# Whitepaper Section 11 — Calculations
> Deep Dive Series  
> Research Unit: BITCOIN-012

---

## Research Brief

```yaml
knowledge_id: BITCOIN-012
title: Whitepaper Section 11 — Calculations
research_question: >
  How does the Bitcoin Whitepaper model the probability that an attacker can
  catch up from behind, and how should institutions interpret confirmation
  depth as probabilistic settlement evidence rather than deterministic finality?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-005
  - BITCOIN-009
  - BITCOIN-010
  - POW-010
  - POW-011
  - POW-014
parent: Bitcoin Whitepaper
previous: BITCOIN-011
next: BITCOIN-013
related_topics:
  - Proof of Work
  - Chain Selection
  - Chainwork
  - Double-Spend Probability
  - Confirmations
  - Settlement Risk
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
  - Full stochastic-process proof
  - Modern empirical hashrate estimation
  - Fee-sniping and selfish-mining models
  - Exchange-specific confirmation policies
  - Real-time attack-cost pricing
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Explain what attacker catch-up probability means in the whitepaper.
- Define `p`, `q`, and `z` in the double-spend race model.
- Explain why the attacker's probability declines when `q < p`.
- Explain why the model gives eventual catch-up probability `1` when `q >= p`.
- Explain why the whitepaper uses a Poisson approximation.
- Distinguish confirmation count from deterministic finality.
- Explain how attacker hashpower assumptions drive risk estimates.
- Explain what the whitepaper's calculations do and do not prove.
- Interpret confirmation policy as a risk-management decision.
- Avoid treating the "six confirmations" convention as a universal guarantee.

---

## 2. Key Questions

1. What attack is Section 11 calculating?
2. What do `p`, `q`, and `z` represent?
3. Why does the honest chain's lead matter?
4. Why is the attack modeled as a random walk?
5. Why does the whitepaper introduce the Poisson distribution?
6. What happens if the attacker has at least as much block-production probability as honest miners?
7. Why does probability fall exponentially when the attacker has less hashpower?
8. What does a confirmation count actually measure?
9. What assumptions must hold for the table values to be meaningful?
10. What does Section 11 imply for high-value payment acceptance?
11. What risks are not included in the calculation?
12. How should institutions use, but not overuse, this model?

---

## 3. Executive Summary

Whitepaper Section 11 calculates the probability that an attacker can catch up after a recipient waits for confirmations. The attack model is a double-spend race: the honest chain includes the recipient's transaction while the attacker privately tries to build an alternative chain that excludes or conflicts with that transaction.[^ref-btc-wp]

The model defines:

```text
p = probability honest nodes find the next block
q = probability attacker finds the next block
z = number of blocks by which the honest chain is ahead
```

If `q < p`, the probability that the attacker ever catches up from `z` blocks behind declines as:

```text
(q / p)^z
```

If `q >= p`, the whitepaper gives the attacker's eventual catch-up probability as `1`.[^ref-btc-wp]

The section then improves the payment-confirmation model by recognizing that while the honest chain finds `z` blocks, the attacker may have found some private blocks. The whitepaper models the attacker's hidden progress with a Poisson distribution whose expected value is:

```text
lambda = z * (q / p)
```

The result is a table showing that attack probability falls rapidly with confirmations when the attacker controls materially less hashpower than honest miners.[^ref-btc-wp]

This is the mathematical foundation for probabilistic finality in Bitcoin. Confirmations do not make reversal impossible. They increase the cumulative work an attacker must overcome and reduce the modeled probability of success under stated assumptions.

For institutions, the practical conclusion is:

```text
confirmation policy = value at risk + attacker model + network conditions + operational tolerance
```

The whitepaper provides the probability model. It does not decide a universal confirmation threshold for every transaction size, counterparty, or institutional workflow.

---

## 4. Original Design

Section 11 follows the earlier proof-of-work and network sections. It asks how long a recipient should wait before considering a payment sufficiently hard to reverse.

The whitepaper assumes:

1. Honest miners are extending the public chain.
2. The attacker is privately mining an alternative chain.
3. The recipient waits until the transaction is included in a block and `z` more blocks are added after it.
4. The attacker may have made hidden progress during that time.
5. The question is the probability that the attacker can still catch up.[^ref-btc-wp]

This is not a model of every Bitcoin attack. It is a model of a specific double-spend race.

The section's purpose is to quantify the intuition from proof of work:

```text
more confirmations
    -> larger honest-chain lead
    -> more work attacker must overcome
    -> lower success probability if q < p
```

---

## 5. Protocol Structure

### The Race

The attack race can be represented as:

```text
honest chain:
    block containing payment + z confirmations

attacker chain:
    private branch excluding or conflicting with payment
```

The attacker succeeds if the private branch catches up and overtakes the honest branch with enough work to become the selected chain.

### Confirmation Count

A confirmation count is a depth measure:

```text
1 confirmation:
    transaction is included in a block

6 confirmations:
    transaction's block plus five later blocks, depending on local counting convention
```

Operational interfaces sometimes count the containing block as the first confirmation. Section 11's `z` is the honest-chain lead after the transaction block is accepted. Analysts must state the convention they are using.

### Chainwork, Not Social Voting

The probability model assumes proof-of-work chain selection. Nodes do not vote by identity. They validate blocks and prefer the valid chain with the most work. Bitcoin Developer documentation describes peers following the most difficult chain to recreate when valid competing branches exist.[^ref-btc-dev-blockchain]

### Full Node and SPV Context

The calculation matters for both full-node and SPV users, but their evidence differs. Full nodes validate blocks directly. SPV clients rely on headers and Merkle inclusion proofs. Bitcoin Developer documentation distinguishes these operating modes and states that full nodes provide the strongest model by validating from genesis.[^ref-btc-dev-operating]

---

## 6. Technical Mechanics

### Random Walk Model

The whitepaper compares the attacker's attempt to catch up with a gambler's ruin problem. If honest miners have probability `p` of finding the next block and the attacker has probability `q`, then each new block changes the distance between chains.

If honest miners find the next block:

```text
attacker falls one block further behind
```

If the attacker finds the next block:

```text
attacker gets one block closer
```

When `q < p`, the process has negative drift for the attacker. The further behind the attacker is, the less likely eventual catch-up becomes.

### Catch-Up Probability from a Fixed Deficit

For a deficit of `z` blocks:

```text
q_z = 1                  if p <= q
q_z = (q / p)^z          if p > q
```

This formula answers:

```text
Given that the attacker is z blocks behind right now,
what is the probability the attacker ever catches up?
```

### Hidden Attacker Progress

A recipient cannot know exactly how many private blocks the attacker found while the honest network found `z` blocks. The whitepaper therefore models the attacker's progress during that time as a Poisson random variable with expected value:

```text
lambda = z * (q / p)
```

The final probability sums over possible attacker progress values and weighs each case by the probability of catching up from the remaining deficit.[^ref-btc-wp]

### Why Poisson Appears

The Poisson approximation is used because the honest chain took approximately the expected time to find `z` blocks, and the attacker is independently finding blocks at a lower expected rate. The model is not measuring a deterministic schedule. It is estimating the distribution of attacker progress during the waiting period.

---

## 7. Mathematical or Economic Model

### Variable Definitions

| Symbol | Meaning |
|---|---|
| `p` | honest miners' probability of finding the next block |
| `q` | attacker's probability of finding the next block |
| `z` | honest chain lead in blocks |
| `lambda` | expected number of attacker blocks while honest miners find `z` blocks |

The model assumes:

```text
p + q = 1
```

for the simplified two-party race.

### Example: Fixed Deficit

Suppose:

```text
q = 0.10
p = 0.90
z = 6
```

Then the fixed-deficit catch-up probability is:

```text
(q / p)^z = (0.10 / 0.90)^6
          = (1 / 9)^6
          ~= 0.00000188
```

This is not the final payment-confirmation probability because the attacker may have found hidden blocks during the honest chain's six-block progress.

### Example: Poisson Expected Progress

For:

```text
q = 0.10
p = 0.90
z = 6
```

The whitepaper's Poisson expected value is:

```text
lambda = z * (q / p)
       = 6 * (0.10 / 0.90)
       ~= 0.6667
```

This means that while honest miners find six blocks, the attacker is expected to find about two-thirds of a block under the model.

### Whitepaper Probability Examples

The whitepaper reports that for `q = 0.10`, the attack probability at `z = 5` is `0.0009137` and at `z = 6` is `0.0002428`. For `q = 0.30`, it reports `0.1773523` at `z = 5` and `0.0416605` at `z = 10`.[^ref-btc-wp]

Bitcoin Developer payment-processing documentation reproduces a similar table for the `q = 0.30` case and frames confirmations as reducing double-spend risk rather than eliminating it.[^ref-btc-dev-payment]

### Confirmation Policy Interpretation

The model implies:

```text
higher q
    -> more confirmations needed for same risk threshold

higher transaction value
    -> lower acceptable attack probability

weaker network visibility
    -> stronger operational controls needed
```

The formula does not choose business policy. It gives a probability estimate under assumptions.

---

## 8. Security Assumptions

### Required Assumptions

Section 11 depends on these assumptions:

1. Honest miners control more block-production probability than the attacker.
2. The recipient sees the honest public chain.
3. The attacker cannot make invalid blocks valid.
4. Block discovery follows the modeled probabilistic process.
5. The attacker's goal is to reverse the attacker's own payment through a competing valid history.
6. Nodes follow the valid branch with the most work.

### What the Model Excludes

The calculation does not include:

- eclipse attacks against the recipient;
- selfish-mining revenue dynamics;
- fee-sniping incentives;
- network propagation asymmetry;
- rented-hashpower market constraints;
- exchange withdrawal controls;
- legal or reputational deterrence;
- miner cartel governance;
- actual fiat cost of electricity and hardware;
- BTC price impact from a successful attack.

Those factors may matter in real risk management, but they are outside the whitepaper's Section 11 calculation.

### Majority Attacker Boundary

If `q >= p`, the model says the attacker's eventual catch-up probability is `1`.[^ref-btc-wp]

This does not mean a majority attacker can do anything. It means the attacker can eventually catch up in this race model. Full nodes still reject invalid blocks and unauthorized spends.

---

## 9. Bitcoin Core Implementation

### No Consensus Function Named Section 11

Bitcoin Core does not implement the whitepaper's Section 11 probability calculation as a consensus rule. Nodes do not accept or reject blocks based on "six confirmations" or on an estimated attacker probability.

Consensus validation handles:

- block validity;
- proof-of-work validity;
- contextual difficulty rules;
- transaction validity;
- UTXO spending;
- chain selection by work.

Confirmation count is a wallet, application, and business-policy concept built on top of validated chain state.

### Full Node Security Context

Bitcoin Developer documentation describes full nodes as validating the block chain from genesis and notes that fooling a full node requires presenting a complete alternative history with greater difficulty, which becomes expensive after confirmations.[^ref-btc-dev-operating]

This matches Section 11's intuition but should not be confused with an implementation threshold.

---

## 10. On-Chain Implications

### Observable Facts

Analysts can observe:

- the block containing a transaction;
- the number of blocks after it;
- branch replacements or reorganizations;
- competing branch evidence if available;
- cumulative work if derived from valid headers;
- whether a transaction was removed after a reorg.

### Inferences

Analysts may infer:

- reduced double-spend risk after more confirmations;
- higher settlement confidence after more cumulative work;
- elevated risk during deep reorgs;
- possible attack behavior when a conflicting spend wins;
- confirmation policies that are too weak for value at risk.

### Unknowns

Chain data alone may not reveal:

- the attacker's actual hashpower;
- whether a private fork exists before release;
- attacker intent;
- off-chain settlement timing;
- whether the recipient was eclipsed;
- whether an exchange credited before its stated confirmation threshold.

---

## 11. Institutional Thinking

### Confirmation Policy

Institutions should treat confirmations as probabilistic risk evidence:

```text
confirmation count
    -> depth under current best chain
    -> work needed to reverse
    -> modeled lower probability if q < p
```

Policy should consider:

- transaction value;
- counterparty trust;
- asset release irreversibility;
- network conditions;
- reorg monitoring;
- node independence;
- liquidity of stolen funds;
- operational ability to delay settlement.

### Risk Tiers

| Use Case | SPV/Low Confirmation Tolerance | Full Node / Higher Confirmation Need |
|---|---|---|
| Small retail payment | May tolerate lower depth | Optional depending on risk |
| Exchange deposit | Usually insufficient | Strongly preferred |
| Custody movement | Not appropriate | Required |
| Internal accounting | Depends on materiality | Recommended |
| High-value OTC settlement | Not appropriate | Required with monitoring |

### Research Discipline

When citing Section 11, analysts should state:

```text
attacker hashpower assumption q
confirmation depth z
whether the model is whitepaper probability or operational policy
what risks are outside the model
```

Without those assumptions, a statement like "six confirmations are safe" is too vague for institutional research.

---

## 12. Common Misinterpretations

### Misinterpretation 1: Six confirmations are final.

Incorrect. Six confirmations are a common convention. Section 11 models decreasing probability, not deterministic finality.

### Misinterpretation 2: The table values apply without assumptions.

Incorrect. The values depend heavily on attacker hashpower, honest network visibility, and the model structure.

### Misinterpretation 3: A majority attacker can create invalid coins.

Incorrect. Section 11 concerns catching up with a valid competing chain. Invalid blocks remain invalid under full-node validation.

### Misinterpretation 4: Confirmation depth measures exact energy spent.

Overstated. Confirmation depth is a block-count proxy. More precise work comparison uses cumulative chainwork.

### Misinterpretation 5: The Poisson model proves attacks cannot happen.

Incorrect. It estimates probability under assumptions. Low probability is not impossibility.

### Misinterpretation 6: Zero-confirmation payments have no risk if fees are high.

Incorrect. Bitcoin Developer payment-processing documentation warns that unconfirmed transactions should generally not be trusted without risk analysis.[^ref-btc-dev-payment]

---

## 13. Research Questions

1. How should confirmation thresholds vary by transaction value?
2. How should institutions estimate plausible attacker hashpower `q`?
3. How often do real reorgs exceed one block on Bitcoin mainnet?
4. How should confirmation policy change during network anomalies?
5. How should SPV-based confirmation evidence be discounted relative to full-node evidence?
6. How does cumulative chainwork improve on simple confirmation count?
7. What operational controls reduce risk outside the Section 11 model?
8. How should private-fork uncertainty be represented in risk reports?
9. How should exchange deposit policies handle conflicting mempool transactions?
10. What data is needed to distinguish accidental reorg from deliberate double spend?

---

## 14. Practical Exercises

1. Define `p`, `q`, and `z` in the whitepaper's model.
2. Compute `(q / p)^z` for `q = 0.20`, `p = 0.80`, and `z = 4`.
3. Compute `lambda = z * (q / p)` for `q = 0.30`, `p = 0.70`, and `z = 10`.
4. Explain why the fixed-deficit probability is not the same as the final payment-confirmation probability.
5. Explain why a transaction with six confirmations can still be reversed in principle.
6. Write a short confirmation policy for a high-value exchange deposit and list assumptions.

---

## 15. Evidence Classification

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Whitepaper Section 11 probability model and example results | A |
| REF-BTC-DEV-BLOCKCHAIN-001 | Official developer documentation | Chain selection, forks, and most-difficult-chain explanation | A |
| REF-BTC-DEV-PAYMENT-PROCESSING-001 | Official developer documentation | Confirmation risk guidance and reproduced probability examples | A |
| REF-BTC-DEV-OPERATING-001 | Official developer documentation | Full-node security model and confirmation context | A |

### Claim Ledger

| Claim ID | Claim | Classification | Evidence | Source |
|---|---|---|---|---|
| C001 | Section 11 models an attacker trying to catch up from behind after a recipient waits for confirmations. | FACT | A | REF-BTC-WP-001 |
| C002 | The whitepaper defines `p` as honest block probability and `q` as attacker block probability. | FACT | A | REF-BTC-WP-001 |
| C003 | If `q < p`, catch-up probability from `z` blocks behind is `(q / p)^z`. | FACT | A | REF-BTC-WP-001 |
| C004 | If `q >= p`, the model gives eventual catch-up probability as `1`. | FACT | A | REF-BTC-WP-001 |
| C005 | The whitepaper uses `lambda = z * (q / p)` for the attacker's expected hidden progress. | FACT | A | REF-BTC-WP-001 |
| C006 | Confirmation count reduces modeled double-spend risk but does not create deterministic finality. | FACT | A | REF-BTC-WP-001; REF-BTC-DEV-PAYMENT-PROCESSING-001 |
| C007 | Valid competing branches are resolved by following the most difficult chain to recreate. | FACT | A | REF-BTC-DEV-BLOCKCHAIN-001 |
| C008 | Full nodes provide stronger security than SPV by validating the chain from genesis. | FACT | A | REF-BTC-DEV-OPERATING-001 |
| C009 | Institutional confirmation thresholds should vary by value at risk and operating context. | INTERPRETATION | B | REF-BTC-WP-001; REF-BTC-DEV-PAYMENT-PROCESSING-001 |
| C010 | Section 11 does not model every real-world attack or operational control. | INTERPRETATION | B | REF-BTC-WP-001 |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| ESTIMATE | Calculation dependent on stated assumptions |
| HEURISTIC | Practical operating rule requiring caveats |
| UNKNOWN | Evidence is insufficient |

---

## 16. Knowledge Graph

```text
BITCOIN-012 Calculations
|
+-- interprets: Whitepaper Section 11
|
+-- attack_model
|   +-- honest_probability: p
|   +-- attacker_probability: q
|   +-- honest_lead: z
|   +-- private_branch: attacker progress
|
+-- formulas
|   +-- fixed_deficit: (q / p)^z when q < p
|   +-- majority_boundary: probability 1 when q >= p
|   +-- poisson_mean: lambda = z * (q / p)
|
+-- settlement
|   +-- confirmations reduce modeled risk
|   +-- no finite confirmation count is absolute finality
|   +-- policy depends on value at risk
|
+-- leads_to: BITCOIN-013 Conclusion
```

---

## 17. References

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Section 11, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-blockchain]: Bitcoin Developer Documentation, "Block Chain," forks, proof-of-work, and most-difficult-chain behavior, https://developer.bitcoin.org/devguide/block_chain.html, accessed 2026-08-04.

[^ref-btc-dev-payment]: Bitcoin Developer Documentation, "Payment Processing," confirmation guidance and double-spend risk discussion, https://developer.bitcoin.org/devguide/payment_processing.html, accessed 2026-08-04.

[^ref-btc-dev-operating]: Bitcoin Developer Documentation, "Operating Modes," full-node and SPV security model, https://developer.bitcoin.org/devguide/operating_modes.html, accessed 2026-08-04.

---

## 18. Cross References

### Parent

- Bitcoin Whitepaper

### Previous

- BITCOIN-011 — Whitepaper Section 10 — Privacy

### Next

- BITCOIN-013 — Whitepaper Section 12 — Conclusion

### Related

- BITCOIN-005 — Whitepaper Section 4 — Proof of Work
- BITCOIN-009 — Whitepaper Section 8 — Simplified Payment Verification
- BITCOIN-023 — Chain Reorganization
- POW-010 — Chain Selection
- POW-011 — Cumulative Chainwork
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- Section 11 was scoped to the attacker catch-up model.
- `p`, `q`, `z`, fixed-deficit probability, and Poisson expected progress were separated.
- Confirmation count was separated from deterministic finality.
- Majority-hashrate implications were bounded to the catch-up model and not generalized to invalid-block acceptance.

### Evidence Review

Passed.

- Whitepaper formulas and probability examples cite Section 11 directly.
- Confirmation and payment-processing guidance cites official Bitcoin Developer documentation.
- Full-node and SPV context cites official operating-mode documentation.
- Institutional policy statements are labeled as interpretation.

### Editorial Review

Passed.

- Markdown headings follow the project deep-dive structure.
- Metadata is complete.
- Tables and code fences are closed.
- Terminology is consistent: confirmations, `p`, `q`, `z`, catch-up probability, Poisson, probabilistic finality.

### Adversarial Review

Passed.

- The document does not treat six confirmations as final.
- The document does not claim the model covers every attack.
- The document does not imply majority attackers can create invalid transactions.
- The document separates probability model from business policy.

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
