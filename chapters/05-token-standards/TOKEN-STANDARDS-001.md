---
knowledge_id: TOKEN-STANDARDS-001
title: ERC-20
subtitle: Fungible Token Interface, Allowance Semantics, and the Base Standard for Onchain Asset Representation
version: 1.0.0
status: Reviewed
difficulty: L200
estimated_reading: 95 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - Token Standards
  - Ethereum
  - Assets
parent:
  - Token Standards
prerequisites:
  - SMART-CONTRACTS-001
  - SMART-CONTRACTS-003
related_topics:
  - ERC-721
  - ERC-1155
  - Stablecoins
primary_sources:
  - REF-EIP-20-2015-001
  - REF-ETH-ERC20-2026-001
  - REF-OZ-ERC20-2026-001
tags:
  - token-standards
  - erc20
  - fungible-tokens
---

# ERC-20
> Token Standards  
> Research Unit: TOKEN-STANDARDS-001

---

## Research Brief

```yaml
knowledge_id: TOKEN-STANDARDS-001
title: ERC-20
research_question: >
  What problem does ERC-20 solve, what exact interface and event model does it
  standardize, and what are the operational and security consequences of using
  the allowance-based fungible token model in Ethereum applications?
document_type: foundation
difficulty: L200
prerequisites:
  - SMART-CONTRACTS-001
  - SMART-CONTRACTS-003
parent: Token Standards
previous:
next: TOKEN-STANDARDS-002
related_topics:
  - ERC-721
  - ERC-1155
  - Stablecoins
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Standard Structure
  - Interface Mechanics
  - Operational Model
  - Security Considerations
  - Implementation Notes
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Full Solidity implementation tutorial
  - Stablecoin-specific reserve design
  - Token valuation
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Define ERC-20 as an interface standard rather than as a particular asset.
- Explain the role of `balanceOf`, `transfer`, `approve`, `allowance`, and `transferFrom`.
- Distinguish direct token transfer from delegated spending.
- Explain why ERC-20 interoperability depends on events as well as functions.
- Identify the main risks in allowance handling and token integration.

---

## 2. Key Questions

1. What exactly does ERC-20 standardize?
2. Why was a fungible token interface needed on Ethereum?
3. How does the allowance model work?
4. What do wallets, exchanges, and DeFi protocols assume from ERC-20 behavior?
5. Where do practical integration failures usually appear?

---

## 3. Executive Summary

ERC-20 is a standards-track Ethereum token interface that defines a common API for fungible tokens.[^ref-eip20]

The standard exists so that wallets, exchanges, smart contracts, and other applications can interact with different token contracts through the same basic method and event surface.[^ref-eip20]

At minimum, ERC-20 defines:

- total supply queries,
- per-account balance queries,
- direct transfers,
- delegated transfers through allowances,
- and standardized `Transfer` and `Approval` events.[^ref-eip20]

The economic point is simple: ERC-20 turns an application-specific smart contract balance table into a reusable asset interface. That abstraction made stablecoins, governance tokens, utility tokens, vault shares, and many DeFi position wrappers composable across Ethereum.

The operational caveat is equally important. ERC-20 standardizes the interface, not universal business logic. Transfer restrictions, fee-on-transfer mechanics, rebasing behavior, minting authority, pausability, and blacklist policy all live above the standard. Researchers therefore need to separate:

- ERC-20 consensus-neutral interface facts,
- issuer policy and tokenomics,
- application assumptions,
- and implementation-specific behavior.

---

## 4. Standard Structure

### 4.1 What ERC-20 Is

EIP-20 describes ERC-20 as a standard API for tokens within smart contracts.[^ref-eip20]

The standard does not create a native protocol object inside Ethereum. An ERC-20 token remains ordinary contract state managed by contract code. The standard only specifies how outside actors should interact with that state.

### 4.2 Why It Matters

The EIP states that a standard interface allows tokens on Ethereum to be reused by other applications, including wallets and decentralized exchanges.[^ref-eip20]

That statement is the core institutional insight. ERC-20 reduced integration cost by giving infrastructure providers one canonical interaction surface instead of one custom adapter per asset.

### 4.3 Fungibility

ERC-20 is a fungible token model. Fungibility means units are intended to be interchangeable at the protocol interface level. If Alice has 100 units and Bob has 100 units, the standard treats each unit as equivalent for transfer and accounting purposes.

That differs from NFT standards, where each asset identifier is tracked separately.

---

## 5. Interface Mechanics

### 5.1 Core Read Functions

The standard defines `totalSupply()` and `balanceOf(address)` as view functions for total issuance and per-account balances.[^ref-eip20]

These functions are the base accounting layer for wallets, explorers, and protocols that need current balances.

### 5.2 Direct Transfer

`transfer(address,uint256)` moves tokens from the caller to a recipient and must emit a `Transfer` event.[^ref-eip20]

EIP-20 also specifies that zero-value transfers must be treated as normal transfers and must emit the event.[^ref-eip20]

That requirement matters because many offchain systems reconstruct token movement from logs rather than from internal storage inspection.

### 5.3 Delegated Spending

The delegated spending model uses three functions:

- `approve(spender, value)`
- `allowance(owner, spender)`
- `transferFrom(from, to, value)`[^ref-eip20]

The flow is:

1. Token owner grants a spending limit to a spender.
2. Spender or spender-controlled contract later uses `transferFrom`.
3. The token contract checks whether the spend is authorized.

This model became the backbone of DEX routing, vault deposits, lending collateral movement, and subscription-like token pulls.

### 5.4 Events

ERC-20 defines two core events:

- `Transfer`
- `Approval`[^ref-eip20]

The EIP states that token creation should emit `Transfer` with `from = 0x0`.[^ref-eip20]

That means mint events are represented within the same log vocabulary as ordinary transfers, which simplifies indexer logic.

---

## 6. Operational Model

### 6.1 State Model

Conceptually, a minimal ERC-20 contract tracks:

- a total supply variable,
- a mapping from address to balance,
- and a nested mapping for allowances.

The standard does not force a specific storage layout, but almost every implementation reduces to those accounting objects.

### 6.2 Allowance as a Capability

An allowance is a delegated spending capability, not a transfer itself. Granting allowance does not move funds. It only authorizes future movement up to a limit if the spender chooses to exercise that right.

This distinction matters in analytics. Approval activity and transfer activity are different event classes and represent different user intents.

### 6.3 Interface vs Asset Behavior

A token can be ERC-20 compliant while still applying additional business rules such as:

- minting by privileged roles,
- transfers disabled while paused,
- blacklisting,
- fee deduction,
- or rebasing supply logic.

Therefore, "ERC-20 token" does not imply a neutral or simple monetary instrument. It implies only that the contract exposes the standard interface surface.

---

## 7. Mathematical or Economic Model

### 7.1 Fungible Balance Accounting

For account set `A`, balances `b(a)`, and total supply `S`:

`S = sum(b(a))` across all token-holding addresses, subject to mint and burn state transitions.

Direct transfer between addresses `x` and `y` for amount `v` preserves supply:

- `b'(x) = b(x) - v`
- `b'(y) = b(y) + v`
- `S' = S`

Minting changes supply:

- `b'(y) = b(y) + v`
- `S' = S + v`

Burning changes supply:

- `b'(x) = b(x) - v`
- `S' = S - v`

The ERC-20 standard does not itself define mint or burn functions, but its event guidance explicitly acknowledges token creation through `Transfer` from the zero address.[^ref-eip20]

### 7.2 Allowance Constraint

For owner `o`, spender `s`, and approved limit `L(o,s)`, a valid delegated transfer of value `v` requires:

`v <= L(o,s)`

and sufficient owner balance.

In many implementations:

`L'(o,s) = L(o,s) - v`

after successful `transferFrom`, unless the contract intentionally implements special infinite-approval semantics.

---

## 8. Security Considerations

### 8.1 Allowance Race Condition

EIP-20 explicitly notes a known risk when changing an allowance from one nonzero value to another nonzero value, and recommends user interfaces first set the allowance to zero before setting a new value.[^ref-eip20]

This is an interface-level warning, not a mandatory contract rule. The standard preserved backward compatibility and left the mitigation largely to wallets and applications.[^ref-eip20]

### 8.2 Token Reception Assumptions

Ethereum.org notes a known issue: plain ERC-20 transfers do not notify receiving contracts in a mandatory way.[^ref-eth-erc20]

That means a contract can receive ERC-20 tokens without executing any receipt hook, creating integration risk if the receiving application expected callback-driven accounting.

### 8.3 Non-Standard Implementations

In production, many tokens deviate from idealized assumptions:

- some do not return `bool` as expected,
- some charge transfer fees,
- some rebase balances,
- some centralize mint and freeze powers.

These behaviors may still be application-relevant even if the token broadly presents an ERC-20 interface. Analysts therefore need to inspect both the standard conformance layer and the issuer logic layer.

---

## 9. Implementation Notes

OpenZeppelin documents ERC-20 as the standard interface for fungible tokens and provides a widely used implementation baseline.[^ref-oz-erc20]

That matters because ecosystem behavior often converges around reference implementations rather than around the abstract EIP alone. In practice, many integrators implicitly assume OpenZeppelin-like semantics for:

- revert behavior,
- allowance updates,
- metadata helpers,
- and mint/burn extensions.

Researchers should treat those assumptions as implementation conventions, not as mandatory ERC-20 consensus rules.

---

## 10. On-Chain Implications

### 10.1 Indexing

Most token analytics pipelines use `Transfer` and `Approval` logs as the first-layer observable surface. This is efficient but incomplete when a token has unusual logic, proxy upgrades, or off-standard side effects.

### 10.2 Composability

ERC-20 made token balances portable across applications. A DEX router, lending market, bridge, or vault can integrate any conforming token with much lower marginal engineering cost.

### 10.3 Policy Layer Above the Standard

Researchers must separate:

- standard interface compliance,
- contract ownership and privileged roles,
- token issuance policy,
- and surrounding legal or reserve claims.

For example, a stablecoin may use ERC-20 while also depending on offchain reserve attestations and issuer discretion. The ERC-20 layer alone cannot validate those external claims.

---

## 11. Institutional Thinking

- Treat ERC-20 as an interoperability grammar, not as an asset-quality certification.
- Separate transferability from trustlessness. Many ERC-20 assets remain centrally administered.
- Separate token balances from legal claims. The contract can represent units, but not automatically the enforceability of offchain rights.
- In due diligence, inspect admin powers, mint authority, pause controls, blacklist logic, upgradeability, and dependency on external systems.
- In analytics, distinguish approvals from actual flows. Approval spikes can indicate future routing or protocol migration, not completed asset movement.

---

## 12. Evidence Classification

| Claim | Classification | Basis |
|---|---|---|
| ERC-20 defines a standard API for tokens in smart contracts | Directly specified | EIP-20 abstract |
| ERC-20 improves wallet and exchange interoperability | Directly specified | EIP-20 motivation |
| Zero-value transfers must emit `Transfer` | Directly specified | EIP-20 specification |
| Allowance changes from nonzero to nonzero create known UX/security risk | Directly specified | EIP-20 note |
| OpenZeppelin semantics influence ecosystem expectations | Inference from implementation practice | Widely used reference implementation |
| ERC-20 interface compliance does not guarantee neutral issuer policy | Analytical inference | Standard scope is interface-only |

---

## 13. References

[^ref-eip20]: Ethereum Improvement Proposals, "ERC-20: Token Standard," created 2015-11-19, accessed 2026-08-04, https://eips.ethereum.org/EIPS/eip-20

[^ref-eth-erc20]: ethereum.org, "ERC-20 Token Standard," official documentation, last updated 2026-07-30, accessed 2026-08-04, https://ethereum.org/developers/docs/standards/tokens/erc-20/

[^ref-oz-erc20]: OpenZeppelin Docs, "ERC-20," OpenZeppelin Contracts 5.x documentation, accessed 2026-08-04, https://docs.openzeppelin.com/contracts/5.x/erc20

---

## 14. Cross References

### Previous

- SMART-CONTRACTS-008 — Oracle Fundamentals

### Next

- TOKEN-STANDARDS-002 — ERC-721

---

## Review Status

### Technical Review

Passed.

- Core ERC-20 functions, events, and allowance flow were separated clearly.
- Interface standard and implementation convention were not conflated.
- Mint and burn were treated as common patterns rather than mandatory base functions.

### Evidence Review

Passed.

- Core interface claims are tied to EIP-20.
- Known allowance warning is tied to the EIP note.
- Implementation observations are attributed to OpenZeppelin as convention, not as protocol law.

### Editorial Review

Passed.

- Section order follows project format.
- Vocabulary is consistent with prior Ethereum and Smart Contract modules.
- Tables and math notation are bounded and readable.

### Adversarial Review

Passed.

- The document does not treat all ERC-20 tokens as trust-minimized.
- It does not imply that interface compliance ensures sound tokenomics.
- It does not collapse approval activity into completed transfers.

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
