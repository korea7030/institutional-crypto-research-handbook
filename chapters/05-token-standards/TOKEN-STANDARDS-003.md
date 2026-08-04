---
knowledge_id: TOKEN-STANDARDS-003
title: ERC-1155
subtitle: Multi-Token Contracts, Batch Transfers, and the Fungibility-Agnostic Asset Model
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 105 min
estimated_study: 250 min
last_reviewed: 2026-08-04
domain:
  - Token Standards
  - Ethereum
  - Assets
parent:
  - Token Standards
prerequisites:
  - TOKEN-STANDARDS-001
  - TOKEN-STANDARDS-002
related_topics:
  - Stablecoins
  - Wrapped Assets
  - Utility Tokens
primary_sources:
  - REF-EIP-1155-2019-001
  - REF-ETH-ERC1155-2026-001
  - REF-OZ-ERC1155-2026-001
tags:
  - token-standards
  - erc1155
  - multi-token
---

# ERC-1155
> Token Standards  
> Research Unit: TOKEN-STANDARDS-003

---

## Research Brief

```yaml
knowledge_id: TOKEN-STANDARDS-003
title: ERC-1155
research_question: >
  How does ERC-1155 generalize token representation beyond ERC-20 and ERC-721,
  what benefits come from multi-token and batch semantics, and what analytical
  cautions arise when fungible and non-fungible assets coexist inside one
  contract?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-001
  - TOKEN-STANDARDS-002
parent: Token Standards
previous: TOKEN-STANDARDS-002
next: TOKEN-STANDARDS-004
related_topics:
  - Stablecoins
  - Wrapped Assets
  - Utility Tokens
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Standard Structure
  - Interface Mechanics
  - Asset Model
  - Security Considerations
  - Implementation Notes
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Full game-economy design
  - Royalty extensions
  - Crosschain asset mirrors
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Define ERC-1155 as a multi-token standard.
- Explain how one contract can manage multiple token types.
- Distinguish batch transfers from single-asset transfers.
- Explain how approval semantics differ from ERC-20 allowances.
- Identify the analytical tradeoffs of a fungibility-agnostic contract model.

---

## 2. Key Questions

1. Why was ERC-1155 introduced after ERC-20 and ERC-721?
2. What does it mean for one token ID to represent its own asset type?
3. Why are batch operations operationally important?
4. How do approvals and safe transfers work?
5. What new indexing and interpretation challenges appear?

---

## 3. Executive Summary

ERC-1155 is a standards-track interface for contracts that manage multiple token types, where a single deployed contract may contain fungible, non-fungible, or semi-fungible assets.[^ref-eip1155][^ref-eth-erc1155]

The EIP explains that previous standards often required separate contracts for separate token types or collections, which duplicated bytecode and limited functionality.[^ref-eip1155]

ERC-1155 changes the asset model. Instead of one contract per fungible token or one contract per NFT collection, a single contract can define many token IDs, and each ID can represent its own token type with its own supply and metadata characteristics.[^ref-eip1155]

The standard also introduces batch operations, allowing multiple assets to be transferred or queried in one call.[^ref-eth-erc1155]

The result is a more general asset container that is particularly useful where many asset classes coexist, such as gaming inventories, complex collectibles, or systems that mix fungible and non-fungible positions.

---

## 4. Standard Structure

### 4.1 What ERC-1155 Is

EIP-1155 defines a standard interface for contracts that manage multiple token types and states that a single contract may include fungible tokens, non-fungible tokens, or other configurations such as semi-fungible tokens.[^ref-eip1155]

This is the key conceptual break from earlier standards. ERC-1155 is not just "another token type." It is a more general container model for many token types under one contract address.

### 4.2 Why It Was Needed

The EIP notes that ERC-20 and ERC-721 often require separate contract deployments per token type or collection, creating redundant bytecode and limiting certain functionality.[^ref-eip1155]

ERC-1155 addresses this by using token IDs inside one contract as separate asset classes.

---

## 5. Interface Mechanics

### 5.1 Token IDs as Asset Types

EIP-1155 states that each token ID can represent a configurable token type with its own metadata, supply, and other attributes.[^ref-eip1155]

This means the `_id` argument is not just a serial number. It is the selector for which asset type is being referenced in a transaction.

### 5.2 Batch Operations

Ethereum.org summarizes key ERC-1155 features as:

- batch transfer,
- batch balance query,
- operator approval,
- receive hooks,
- NFT support,
- and safe transfer rules.[^ref-eth-erc1155]

The batch model matters because many related asset movements can be processed in one call rather than spread across many transactions.

### 5.3 Events

ERC-1155 defines:

- `TransferSingle`
- `TransferBatch`
- `ApprovalForAll`
- `URI`[^ref-eip1155]

The EIP states that minting uses the zero address as `_from`, and burning uses the zero address as `_to` in transfer events.[^ref-eip1155]

### 5.4 Safe Transfers

The EIP requires safe transfer behavior and specifies that transfers must revert if the recipient is the zero address, if balances are insufficient, or if other errors occur.[^ref-eip1155]

This makes transfer semantics more explicitly delivery-aware than plain ERC-20 behavior.

### 5.5 Approval Model

ERC-1155 uses operator-style approval rather than amount-based allowance. Ethereum.org notes that instead of approving specific amounts, a holder approves or unapproves an operator through `setApprovalForAll`.[^ref-eth-erc1155]

That makes ERC-1155 approval logic structurally closer to ERC-721 than to ERC-20.

---

## 6. Asset Model

### 6.1 One Contract, Many Asset Classes

Suppose contract `C` supports token ID set `I`.

For each `i in I`, define:

- supply `S_i`
- metadata reference `M_i`
- balance function `b_i(a)` for address `a`

Then the contract is not one asset. It is a family of assets:

`AssetFamily(C) = {i_1, i_2, ..., i_n}`

with shared code and often shared operator permissions.

### 6.2 Fungibility-Agnostic Design

If a token ID has large divisible supply, it may behave like a fungible asset.

If a token ID has supply 1, Ethereum.org notes it can be treated as an NFT.[^ref-eth-erc1155]

If a token ID begins as fungible and later narrows into more unique claim states, the design may operate as a semi-fungible model in application terms.

### 6.3 Batch State Transition

For transfer batch from `x` to `y` across ID list `I = [i1, i2, ..., in]` and values `V = [v1, v2, ..., vn]`:

for each `k`,

- `b'ik(x) = bik(x) - vk`
- `b'ik(y) = bik(y) + vk`

The batch operation is essentially a vectorized collection of single-asset state transitions processed in one call.

---

## 7. Security Considerations

### 7.1 Broad Operator Power

Because approval is typically operator-wide rather than amount-specific, a compromised or malicious operator can affect all eligible token IDs under that approval scope.

### 7.2 Mixed-Asset Interpretation Risk

When fungible and non-fungible assets live under one contract, naive analytics can misread contract-level activity. A transfer spike may reflect:

- many token types moving simultaneously,
- one batch settlement,
- or one application workflow with heterogeneous assets.

Contract-level aggregation alone is often too coarse.

### 7.3 Metadata Dependency

The `URI` event and metadata scheme support discoverability, but the permanence and integrity of metadata still depend on how the application manages the referenced resources.

---

## 8. Implementation Notes

OpenZeppelin describes ERC-1155 as a fungibility-agnostic and gas-efficient token contract model that draws ideas from prior standards.[^ref-oz-erc1155]

That captures the operational value proposition well:

- fewer deployments,
- more general asset modeling,
- and efficient grouped operations.

But efficiency depends on actual application design. If the contract introduces complex per-ID rules, access control layers, or metadata indirection, overall operational simplicity may still be limited.

---

## 9. On-Chain Implications

### 9.1 Indexing Complexity

ERC-1155 analytics generally require a composite key at minimum:

- contract address,
- token ID,
- holder address

Event consumers also need to parse both single and batch transfer logs correctly.

### 9.2 Gas and Workflow Efficiency

Batch operations can reduce the number of transactions and event streams needed for related state changes, especially in inventory-style systems.

### 9.3 Contract-Centric Risk

Because many asset types can sit inside one contract, a bug, pause, proxy upgrade, or compromised admin path may affect a much broader asset surface than in one-contract-per-asset architectures.

---

## 10. Institutional Thinking

- Treat ERC-1155 contracts as multi-asset systems, not as single assets.
- Build dashboards and risk views at the `contract + tokenId` level, not just at the contract level.
- Operator approval monitoring is critical because the approval surface may span many asset classes.
- Shared-code efficiency can increase shared failure domains. Reduced deployment count does not reduce governance or contract risk by itself.
- When a project describes assets as fungible, semi-fungible, and collectible within one environment, ERC-1155 often explains the technical representation layer beneath that product story.

---

## 11. Evidence Classification

| Claim | Classification | Basis |
|---|---|---|
| ERC-1155 manages multiple token types in one contract | Directly specified | EIP-1155 summary and abstract |
| Each token ID can define its own type and attributes | Directly specified | EIP-1155 motivation |
| Batch transfer and batch balance are key features | Secondary explanatory summary | ethereum.org documentation |
| ERC-1155 uses operator approvals rather than ERC-20-style numeric allowances | Directly specified and summarized | EIP-1155 plus ethereum.org |
| ERC-1155 is fungibility-agnostic and gas-efficient in design intent | Implementation summary | OpenZeppelin documentation |
| Contract-level aggregation can mislead analysis | Analytical inference | Multi-asset architecture |

---

## 12. References

[^ref-eip1155]: Ethereum Improvement Proposals, "ERC-1155: Multi Token Standard," created 2019-06-17, accessed 2026-08-04, https://eips.ethereum.org/EIPS/eip-1155

[^ref-eth-erc1155]: ethereum.org, "ERC-1155 Multi-Token Standard," official documentation, last updated 2026-07-30, accessed 2026-08-04, https://ethereum.org/developers/docs/standards/tokens/erc-1155/

[^ref-oz-erc1155]: OpenZeppelin Docs, "ERC-1155," OpenZeppelin Contracts 5.x documentation, accessed 2026-08-04, https://docs.openzeppelin.com/contracts/5.x/erc1155

---

## 13. Cross References

### Previous

- TOKEN-STANDARDS-002 — ERC-721

### Next

- TOKEN-STANDARDS-004 — Stablecoins

---

## Review Status

### Technical Review

Passed.

- Multi-token architecture and token-ID semantics were separated clearly.
- Batch mechanics were described as vectorized state transitions.
- Approval semantics were distinguished from ERC-20 allowances.

### Evidence Review

Passed.

- Multi-token and per-ID claims are grounded in EIP-1155.
- Batch-feature summaries are tied to ethereum.org.
- Implementation framing is tied to OpenZeppelin and not overstated as protocol law.

### Editorial Review

Passed.

- Document structure is aligned with prior modules.
- Terminology around fungible, non-fungible, and semi-fungible use was kept precise.
- Mathematical notation is constrained to the conceptual model.

### Adversarial Review

Passed.

- The document does not imply every ERC-1155 deployment is simpler than ERC-20 or ERC-721 in practice.
- It does not flatten all token IDs into one asset.
- It does not assume metadata permanence from URI support alone.

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
