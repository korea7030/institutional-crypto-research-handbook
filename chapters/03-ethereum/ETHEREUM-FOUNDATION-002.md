---
knowledge_id: ETHEREUM-FOUNDATION-002
title: Account Model
subtitle: Externally Owned Accounts, Contract Accounts, Nonces, Storage Roots, and the Modern Boundaries of Ethereum Identity
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 125 min
estimated_study: 320 min
last_reviewed: 2026-08-04
domain:
  - Ethereum
  - Accounts
  - EVM
  - State
parent:
  - Ethereum Foundations
prerequisites:
  - ETHEREUM-FOUNDATION-001
  - BITCOIN-014
related_topics:
  - World State
  - Transactions
  - Nonce
  - Contract Code
  - Validator Keys
primary_sources:
  - REF-ETH-WP-001
  - REF-ETH-DOC-ACCOUNTS-2026-001
  - REF-ETH-DOC-INTRO-2026-001
  - REF-EIP-7702
  - REF-ETH-EXEC-SPECS-README-2026-001
tags:
  - ethereum
  - accounts
  - eoa
  - contract-account
  - nonce
  - codehash
---

# Account Model
> Ethereum Foundations  
> Research Unit: ETHEREUM-FOUNDATION-002

---

## Research Brief

```yaml
knowledge_id: ETHEREUM-FOUNDATION-002
title: Account Model
research_question: >
  How does Ethereum represent ownership and executable entities through its
  account model, what fields define an account, how do externally owned and
  contract-controlled accounts differ, and which classical account-model
  assumptions now require qualification in August 2026?
document_type: deep-dive
difficulty: L300
prerequisites:
  - ETHEREUM-FOUNDATION-001
  - BITCOIN-014
parent: Ethereum Foundations
previous: ETHEREUM-FOUNDATION-001
next: ETHEREUM-FOUNDATION-003
related_topics:
  - World State
  - Transactions
  - Contract Creation
  - Nonce
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Protocol Structure
  - Technical Mechanics
  - Mathematical or Economic Model
  - Security Assumptions
  - Protocol Implementation
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Full wallet UX survey
  - Smart contract language details
  - Address checksum tutorial
  - Exhaustive account-abstraction design space
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Define Ethereum's account model and contrast it with Bitcoin's UTXO model.
- Explain the fields that make up an Ethereum account.
- Distinguish externally owned accounts from contract accounts.
- Explain why nonce exists and why it matters for replay protection and ordering.
- Explain why the classical EOA versus contract distinction now needs qualification in modern Ethereum.

---

## 2. Key Questions

1. What is an Ethereum account?
2. Why did Ethereum choose an account model instead of UTXOs?
3. What fields are stored in an account?
4. How do EOAs and contract accounts differ?
5. What does nonce do?
6. Why is the traditional "EOAs have no code" rule no longer a sufficient complete summary in 2026?

---

## 3. Executive Summary

Ethereum's account model is the basic object model of Ethereum state. The original whitepaper says the Ethereum state is made up of objects called accounts, each with a 20-byte address and four fields: nonce, balance, contract code if present, and storage.[^ref-eth-wp]

Current Ethereum documentation expresses the same idea in more operational terms: an Ethereum account is an entity with an ETH balance that can send messages on Ethereum, and accounts can be user-controlled or deployed as smart contracts.[^ref-eth-doc-accounts]

The classic distinction is:

- externally owned account (EOA): controlled by private keys,
- contract account: controlled by code.[^ref-eth-doc-accounts]

That distinction is still useful, but in 2026 it is no longer the whole story. EIP-7702 and the execution-spec releases that include it modify older invariants around EOAs and code, so researchers should avoid presenting the old mental model as timeless protocol truth.[^ref-eip-7702][^ref-eth-exec-specs]

The deeper point is that Ethereum uses accounts as the directly addressed objects of state transition. This is one of the foundational ways Ethereum differs from Bitcoin.

---

## 4. Protocol Structure

### Account-Centric Design

In Ethereum, state is organized around accounts rather than around a set of independent spendable outputs.

An account is an addressable object that can hold:

- ETH balance,
- replay/order state through nonce,
- executable code,
- persistent storage commitments.

### Two Classical Types

Current Ethereum documentation describes two account types:[^ref-eth-doc-accounts]

| Type | Controlled By | Can Initiate Transactions? |
|---|---|---|
| Externally owned account | private keys | yes |
| Contract account | contract code | not as an originator in the classical model |

### Why This Matters

This means Ethereum identity, value holding, and executable logic all live inside the same state object family.

---

## 5. Historical Context

### Original Whitepaper Framing

The Ethereum whitepaper introduced accounts as the base state objects because Ethereum wanted direct transfers of value and information between accounts rather than a UTXO graph of consumed and newly created outputs.[^ref-eth-wp]

### Why This Was Attractive

The account model simplifies some application-level reasoning:

- balances live in accounts,
- contracts live at addresses,
- repeated interactions reuse the same logical object,
- stateful applications can persist data under their own addresses.

### Trade-Off

This simplicity of application addressing comes with a more complex shared-state machine and richer execution semantics.

---

## 6. Account Fields

### Four Core Fields

The current Ethereum accounts documentation and the whitepaper both describe four core fields:[^ref-eth-doc-accounts][^ref-eth-wp]

```text
nonce
balance
storageRoot
codeHash
```

### `nonce`

Current docs define nonce as a counter indicating the number of transactions sent from an EOA or the number of contracts created by a contract account.[^ref-eth-doc-accounts]

Its function is to prevent replay and provide per-account sequencing.

### `balance`

`balance` is the amount of wei owned by the address.[^ref-eth-doc-accounts]

### `codeHash`

`codeHash` points to the EVM code associated with the account. For EOAs, the docs say this is the hash of an empty string in the classical model.[^ref-eth-doc-accounts]

### `storageRoot`

`storageRoot` is the root of the Merkle Patricia Trie that encodes the account's persistent storage.[^ref-eth-doc-accounts]

---

## 7. EOAs and Contract Accounts

### Externally Owned Accounts

EOAs are controlled by cryptographic keys. Current docs say they can initiate transactions and are created at no network storage cost.[^ref-eth-doc-accounts]

### Contract Accounts

Contract accounts are controlled by their deployed code. They can react to transactions and message calls, hold balances, and maintain storage.[^ref-eth-doc-accounts][^ref-eth-doc-intro]

### Contract Accounts Do Not "Sign"

A contract account does not own a private key in the ordinary sense. The network executes its code when triggered by valid transaction flow.

### Classical Summary

The traditional concise summary is:

```text
EOA signs and initiates
contract executes and reacts
```

This is still a useful baseline, but it is no longer an exhaustive statement of all modern behavior.

---

## 8. Nonce and Ordering

### Why Nonce Exists

Without nonce, signed transactions could be replayed.

### Security Function

Only one transaction with a given nonce can execute for a given account, which current docs explicitly tie to replay protection.[^ref-eth-doc-accounts]

### Operational Meaning

Nonce also creates per-account ordering semantics. It is not a global transaction index for the chain. It is local to the account.

### Contract-Side Meaning

For contract accounts, nonce historically counts contract creations from that account.[^ref-eth-doc-accounts]

---

## 9. Modern Qualification: EIP-7702

### Why the Old Model Needs Qualification

EIP-7702 breaks several older invariants. The EIP explicitly states that once an account has been delegated, an account balance can decrease without that account originating the transaction in the old sense, an EOA nonce may increase after execution has begun, and `tx.origin == msg.sender` no longer only occurs in the topmost frame.[^ref-eip-7702]

### Currentness

The execution-specs repository lists Prague, released on May 7, 2025, and includes EIP-7702 among the included EIPs.[^ref-eth-exec-specs]

### Practical Research Consequence

This means "EOAs have no code and therefore behave in exactly one simple way" is no longer a safe universal summary for August 4, 2026.

### What Still Survives

The account model still distinguishes between key-based authority and code-based behavior, but some older invariants around that distinction have softened.

---

## 10. Technical Mechanics

### Account Addressing

Accounts are directly addressed objects in global state. Current docs say an account address is 20 bytes, typically shown as a 42-character hex string with `0x` prefix.[^ref-eth-doc-accounts]

### State Interaction

Transactions update the state of accounts:

- debiting balances,
- incrementing nonces,
- executing code,
- updating storage,
- emitting logs through contract execution paths.

### Persistence

Unlike transient execution memory, account fields and contract storage belong to the persistent state that survives across blocks.

---

## 11. Security Assumptions

### Key Security

For EOAs, security depends fundamentally on private-key control.

### Code Security

For contract accounts, security depends on code correctness, deployment assumptions, and message-flow safety.

### Invariant Risk

Researchers and auditors often rely on simplified assumptions such as "EOAs cannot do X." Modern protocol evolution means these assumptions should be checked against current rules instead of treated as eternal truths.[^ref-eip-7702]

---

## 12. Mathematical or Economic Model

### Minimal Account State Model

A conceptual account object can be expressed as:

```text
account = (nonce, balance, storageRoot, codeHash)
```

This is a structural model, not an implementation detail invented here; it reflects the four-field description in Ethereum primary sources.[^ref-eth-wp][^ref-eth-doc-accounts]

### Replay Constraint

For a given origin account in the classical transaction model:

```text
valid next nonce = current nonce
```

and after successful processing:

```text
new nonce = old nonce + 1
```

This simplified expression captures the ordering intuition, though modern delegated-code behavior can complicate older assumptions about when and how nonce changes arise.[^ref-eip-7702]

---

## 13. Protocol Implementation

### Current Documentation

The clearest current operational description of account fields and types is in the official Ethereum accounts documentation.[^ref-eth-doc-accounts]

### Whitepaper Role

The whitepaper is still useful because it shows that the account model is not an accidental implementation detail. It is foundational to Ethereum's original design.[^ref-eth-wp]

### Spec Evolution

Because protocol rules evolve, current implementation/specification state must be checked through modern execution specs and adopted EIPs instead of only through historical summaries.[^ref-eip-7702][^ref-eth-exec-specs]

---

## 14. On-Chain Implications

### What Analysts Observe

On-chain analysts can observe:

- account balances,
- nonce progression,
- contract deployments,
- code-bearing addresses,
- storage effects indirectly through state queries and traces.

### What Requires Care

Naive "EOA vs contract" heuristics can become less reliable as protocol behavior evolves.

### Comparison with Bitcoin

An Ethereum address is not analogous to a Bitcoin UTXO. It is closer to a persistent state object.

---

## 15. Institutional Thinking

Institutions should treat Ethereum accounts as stateful control objects, not merely as balance containers.

### Practical Implications

- Key-management policy is necessary but not sufficient.
- Contract-risk analysis is account-risk analysis.
- Address labeling should distinguish user-controlled accounts from code-bearing or delegated-behavior accounts carefully.
- Historical assumptions about EOAs should be version-aware.

### Better Research Posture

When classifying account behavior, institutions should ask:

- Which protocol era is this analysis assuming?
- Is this statement about classical EOAs, contract accounts, or modern delegated code?
- Is the claim protocol-level or just a legacy heuristic?

---

## 16. Common Misinterpretations

### "Ethereum accounts are just addresses with balances"

False. They also encode ordering state and may encode storage and code commitments.

### "Contract accounts are just wallets with scripts"

Too weak. Contract accounts are persistent programmable state objects.

### "EOAs never have code-related behavior concerns"

No longer safe as a universal current statement after EIP-7702.[^ref-eip-7702]

### "Nonce is a chain-wide sequence number"

False. Nonce is per account.

---

## 17. Research Questions

1. How should modern analytics pipelines update classical EOA heuristics after EIP-7702?
2. Which institutional controls best separate key risk from code risk in account classification?
3. How should researchers present historical Ethereum account-model explanations without misleading readers about current behavior?

---

## 18. Practical Exercises

### Exercise 1

Explain the difference between `balance` and `storageRoot`.

### Exercise 2

Write a one-paragraph comparison between a Bitcoin UTXO and an Ethereum account.

### Exercise 3

List three reasons nonce exists.

### Exercise 4

Explain why EIP-7702 forces caution when using older EOA summaries.

---

## 19. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Four-field account structure | Directly specified | Whitepaper and official accounts docs |
| Classical EOA / contract distinction | Directly specified | Official accounts docs |
| Modern qualification of old account invariants | Directly specified | EIP-7702 and execution-spec release context |
| Institutional caution about heuristics | Inference from sources | Derived from current protocol evolution |

---

## 20. Knowledge Graph

```text
Account Model
├─ Account Fields
│  ├─ nonce
│  ├─ balance
│  ├─ storageRoot
│  └─ codeHash
├─ Classical Types
│  ├─ externally owned account
│  └─ contract account
├─ Security Surfaces
│  ├─ private keys
│  ├─ contract code
│  └─ replay protection
├─ Modern Qualification
│  ├─ EIP-7702
│  └─ invariant softening
└─ Related Concepts
   ├─ world state
   ├─ transactions
   └─ state transition
```

---

## 21. References

### Primary Sources

[^ref-eth-wp]: Vitalik Buterin, "Ethereum Whitepaper," official ethereum.org whitepaper page, including the account-model description, https://ethereum.org/whitepaper/, accessed 2026-08-04.

[^ref-eth-doc-accounts]: ethereum.org, "Ethereum accounts," official documentation describing account types and fields, published 2026, https://ethereum.org/developers/docs/accounts, accessed 2026-08-04.

[^ref-eth-doc-intro]: ethereum.org, "Technical intro to Ethereum," official documentation describing the EVM and shared state, page last updated April 22, 2026, https://ethereum.org/developers/docs/intro-to-ethereum/, accessed 2026-08-04.

[^ref-eip-7702]: EIP-7702, "Set Code for EOAs," Ethereum Improvement Proposals, including backward-compatibility notes about changed invariants, https://eips.ethereum.org/EIPS/eip-7702, accessed 2026-08-04.

[^ref-eth-exec-specs]: Ethereum execution-specs repository README, protocol releases table showing Prague released on May 7, 2025 and including EIP-7702, https://github.com/ethereum/execution-specs, accessed 2026-08-04.

### Supporting Interpretation Notes

- Where this document discusses heuristic risk or institutional classification risk, those statements are inferences from the cited current protocol sources rather than direct normative protocol text.

---

## 22. Cross References

### Previous

- ETHEREUM-FOUNDATION-001 — Ethereum Vision

### Next

- ETHEREUM-FOUNDATION-003 — World State

### Related

- BITCOIN-014 — UTXO Model
- ETHEREUM-FOUNDATION-004 — State Transition

---

## Review Status

### Technical Review

Passed.

- The account model was described through its four fields and classical account types.
- Nonce, storage, and code semantics were separated.
- EIP-7702 was included to prevent outdated EOA claims.
- Bitcoin UTXO comparisons were kept conceptual and limited.

### Evidence Review

Passed.

- Whitepaper and official accounts docs support the four-field model and classical type distinction.
- EIP-7702 and execution-specs support the current-state qualification.
- Interpretive caution is labeled as inference.

### Editorial Review

Passed.

- Structure follows the project document pattern.
- Metadata is complete.
- Terminology is consistent: EOA, contract account, nonce, codeHash, storageRoot.
- Tables and code fences are closed.

### Adversarial Review

Passed.

- The document does not treat 2015-era EOA rules as timeless.
- It does not reduce accounts to balances only.
- It does not confuse private-key control with contract-code control.
- It does not overstate the completeness of simple address heuristics.

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
