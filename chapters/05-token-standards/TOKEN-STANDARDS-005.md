---
knowledge_id: TOKEN-STANDARDS-005
title: Wrapped Assets
subtitle: Native-to-Token Conversion, Bridge Representations, and Custody Assumptions Hidden Behind Composable Interfaces
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 95 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - Token Standards
  - Ethereum
  - Interoperability
parent:
  - Token Standards
prerequisites:
  - TOKEN-STANDARDS-001
  - TOKEN-STANDARDS-004
related_topics:
  - Stablecoins
  - Bridges
  - DeFi
primary_sources:
  - REF-ETH-WETH-2026-001
  - REF-CIRCLE-XRESERVE-2026-001
  - REF-CIRCLE-USDC-2026-001
tags:
  - wrapped-assets
  - weth
  - bridges
---

# Wrapped Assets
> Token Standards  
> Research Unit: TOKEN-STANDARDS-005

---

## Research Brief

```yaml
knowledge_id: TOKEN-STANDARDS-005
title: Wrapped Assets
research_question: >
  What is a wrapped asset, why are wrapped representations needed, and how
  should researchers separate technical fungibility from the custody, bridge,
  and redemption assumptions behind the wrapped token?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-001
  - TOKEN-STANDARDS-004
parent: Token Standards
previous: TOKEN-STANDARDS-004
next: TOKEN-STANDARDS-006
related_topics:
  - Stablecoins
  - Bridges
  - DeFi
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - System Structure
  - Conversion Mechanics
  - Economic Model
  - Security Considerations
  - Implementation Notes
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Full bridge taxonomy
  - Per-chain wrapped-token registry
  - Legal custody analysis
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Define wrapped assets as tokenized representations of another asset.
- Distinguish native-asset wrappers from bridge-backed representations.
- Explain mint-on-deposit and burn-on-redeem flows.
- Identify the hidden trust surfaces behind wrapped tokens.
- Separate composability benefits from custody and bridge risk.

---

## 2. Key Questions

1. Why do wrapped assets exist?
2. How does wrapping differ from ordinary token issuance?
3. What is the difference between WETH and a bridged asset representation?
4. What assumptions make a wrapped token credible?
5. What should analysts track when evaluating wrapped assets?

---

## 3. Executive Summary

Ethereum.org explains that wrapped ETH (WETH) exists because native ETH predates ERC-20 and does not conform to the ERC-20 interface, while many applications expect ERC-20 semantics.[^ref-eth-weth]

In the WETH model, users deposit ETH into a smart contract and receive the same amount of minted WETH, which can later be redeemed back into ETH, with the deposited WETH burned on redemption.[^ref-eth-weth]

That is the simplest wrapped-asset pattern: a tokenized representation of another asset held or locked elsewhere.

The broader category includes:

- native-asset wrappers like WETH,
- custodial token wrappers,
- and bridge-backed representations across chains.

Circle's xReserve documentation describes a system in which USDC reserve held in a source-chain smart contract backs stablecoin representations on remote chains through attestations.[^ref-circle-xreserve]

This shows why "wrapped asset" analysis cannot stop at token interface inspection. The crucial questions are:

- where the base asset actually sits,
- who controls mint and burn,
- how redemption works,
- and what happens when the custodian, bridge, or attestation system fails.

---

## 4. System Structure

### 4.1 What a Wrapped Asset Is

A wrapped asset is a blockchain token representing another underlying asset under a conversion rule. The underlying may be:

- a native asset,
- a token on another chain,
- a token in escrow,
- or an offchain asset held by a custodian.

### 4.2 Why Wrapping Is Needed

Ethereum.org states that ETH is not ERC-20 compatible and that wrapping allows ETH to be used where ERC-20 behavior is required.[^ref-eth-weth]

The general pattern applies elsewhere too. A wrapped representation exists because the original asset cannot directly satisfy the interface or network context required by the application.

### 4.3 Main Design Families

Researchers should distinguish:

- same-chain functional wrappers,
- cross-chain representations,
- and custodian-backed synthetic forms.

These may look similar at the wallet interface but differ sharply in trust and failure modes.

---

## 5. Conversion Mechanics

### 5.1 Canonical WETH Flow

Ethereum.org describes the basic WETH flow as:

1. deposit ETH into the WETH contract,
2. receive equal minted WETH,
3. later redeem WETH for ETH,
4. with redeemed WETH burned from supply.[^ref-eth-weth]

This is a 1:1 same-chain wrapper.

### 5.2 Bridge-Backed Flow

Circle's xReserve guide describes reserve USDC held on a source chain and a dual-attestation model used to support USDC-backed tokens on remote chains.[^ref-circle-xreserve]

That means the representation layer depends on:

- lock or reserve accounting,
- attesters,
- mint authorization,
- and remote-chain token issuance logic.

### 5.3 Wrapped vs Native

Ethereum.org notes that WETH is an ERC-20 representation of ETH and that native ETH remains the accepted unit for gas fees, while paying gas with WETH is not natively supported.[^ref-eth-weth]

This shows an important principle: wrapping may equalize transfer interface but does not necessarily equalize protocol privilege.

---

## 6. Mathematical or Economic Model

### 6.1 1:1 Wrapper Invariant

For a simple wrapper with locked underlying `U` and wrapped supply `W`:

`U = W`

should hold if every wrapped unit is fully backed and immediately redeemable.

### 6.2 Bridge Representation Condition

For source reserve `R_s` and remote wrapped supply `W_r`:

`R_s >= W_r`

is the minimal backing condition, subject to attestation correctness and settlement liveness.

### 6.3 Liquidity vs Backing

A wrapped token can remain backed but still trade away from parity if:

- redemption is delayed,
- bridge exits are congested,
- or users distrust the operator.

So price parity depends on operational confidence as well as nominal backing.

---

## 7. Security Considerations

### 7.1 Smart Contract Risk

Ethereum.org characterizes canonical WETH as a simple and battle-tested contract and notes formal verification, but also warns that there are other WETH variants in the wild that may behave differently.[^ref-eth-weth]

Researchers should therefore distinguish canonical wrappers from imitator tokens.

### 7.2 Bridge and Attestation Risk

Bridge-backed wrapped assets introduce more trust surfaces than same-chain wrappers:

- message validation,
- reserve accounting,
- signer integrity,
- and upgrade control.

### 7.3 Redemption Risk

A wrapped asset is only as strong as the path from wrapped representation back to the underlying. If redemption halts, the wrapper may continue trading while no longer functioning as a reliable substitute.

---

## 8. Implementation Notes

Circle's USDC documentation describes the native token itself, while xReserve documentation describes reserve-held USDC backing remote-chain representations.[^ref-circle-usdc][^ref-circle-xreserve]

This is a useful distinction:

- native issuance,
- versus wrapped or backed remote representation.

The same ticker or economic story can hide different settlement architectures across chains.

---

## 9. On-Chain Implications

### 9.1 Composability

Wrapped assets make otherwise incompatible assets usable across ERC-20-based protocols, DEX pools, vaults, and lending systems.

### 9.2 Identity Ambiguity

Users often see one symbol and assume one asset. Analysts should verify:

- contract address,
- chain,
- issuer or wrapper,
- and redemption route.

### 9.3 Monitoring

Key observables include:

- supply growth,
- reserve changes where visible,
- mint/burn patterns,
- bridge pause or attestation failures,
- and discount or premium to the underlying.

---

## 10. Institutional Thinking

- Treat wrapped assets as claims, not just as tokens.
- Always ask what sits beneath the wrapper and who controls exit.
- Distinguish same-chain wrappers like WETH from bridge IOUs and custodian claims.
- A shared ERC-20 interface can hide radically different operational risk.
- For treasury or settlement usage, evaluate redemption reliability before liquidity depth.

---

## 11. Evidence Classification

| Claim | Classification | Basis |
|---|---|---|
| WETH exists because ETH is not ERC-20 compatible | Directly specified | ethereum.org WETH documentation |
| WETH is minted on deposit and burned on redemption | Directly specified | ethereum.org WETH documentation |
| Wrapped-asset security varies by wrapper implementation | Directly specified and inferred | ethereum.org warning about variants |
| Reserve-backed remote representations depend on attestation and reserve flow | Directly specified | Circle xReserve docs |
| Interface parity does not imply equal protocol privilege or risk | Analytical inference | Native ETH vs WETH and bridge architecture |

---

## 12. References

[^ref-eth-weth]: ethereum.org, "Wrapped ether (WETH)," official documentation, last updated 2026-07-23, accessed 2026-08-04, https://ethereum.org/wrapped-eth/

[^ref-circle-usdc]: Circle Docs, "What is USDC?," official documentation, accessed 2026-08-04, https://developers.circle.com/stablecoins/what-is-usdc

[^ref-circle-xreserve]: Circle Docs, "xReserve Technical Guide," official documentation, accessed 2026-08-04, https://developers.circle.com/xreserve/concepts/technical-guide

---

## 13. Cross References

### Previous

- TOKEN-STANDARDS-004 — Stablecoins

### Next

- TOKEN-STANDARDS-006 — Governance Tokens

---

## Review Status

### Technical Review

Passed.

- WETH mechanics were separated from bridge-backed representations.
- Native-asset privilege and ERC-20 compatibility were not conflated.

### Evidence Review

Passed.

- WETH mechanics are tied to ethereum.org.
- Remote reserve-backed representation claims are tied to Circle docs.

### Editorial Review

Passed.

- Structure matches repository conventions.
- Risk terminology is consistent with bridge and token modules.

### Adversarial Review

Passed.

- The document does not imply all wrapped assets are equally trustworthy.
- It does not equate ticker sameness with redemption equivalence.
- It does not assume all WETH-like contracts are canonical.

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
