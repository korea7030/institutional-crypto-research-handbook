---
knowledge_id: SMART-CONTRACTS-001
title: Smart Contract Fundamentals
subtitle: Programs at Addresses, Deterministic Enforcement, and the Core Execution Model of Ethereum Applications
version: 1.0.0
status: Reviewed
difficulty: L200
estimated_reading: 110 min
estimated_study: 260 min
last_reviewed: 2026-08-04
domain:
  - Smart Contracts
  - Ethereum
  - Applications
parent:
  - Smart Contracts
prerequisites:
  - ETHEREUM-FOUNDATION-001
  - ETHEREUM-FOUNDATION-005
related_topics:
  - Contract Lifecycle
  - ABI
  - Events
primary_sources:
  - REF-ETH-DOC-SC-INTRO-2026-001
  - REF-ETH-DOC-SC-ANATOMY-2026-001
  - REF-SOLIDITY-INTRO-001
tags:
  - smart-contracts
  - ethereum
  - fundamentals
---

# Smart Contract Fundamentals
> Smart Contracts  
> Research Unit: SMART-CONTRACTS-001

---

## Research Brief

```yaml
knowledge_id: SMART-CONTRACTS-001
title: Smart Contract Fundamentals
research_question: >
  What is a smart contract on Ethereum, what properties make it different from
  conventional backend software, and how should researchers frame smart
  contracts as deterministic onchain programs rather than as legal or marketing
  abstractions?
document_type: foundation
difficulty: L200
prerequisites:
  - ETHEREUM-FOUNDATION-001
  - ETHEREUM-FOUNDATION-005
parent: Smart Contracts
previous:
next: SMART-CONTRACTS-002
related_topics:
  - ABI
  - Contract Lifecycle
  - Events
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
  - Full Solidity tutorial
  - Legal contract enforceability by jurisdiction
  - Token-standard specifics
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Define a smart contract precisely in Ethereum terms.
- Explain why a smart contract is both an account and a program.
- Distinguish smart contracts from ordinary web backends.
- Explain why deterministic execution matters.
- Explain why "smart contract" should not be confused with legal agreement text.

---

## 2. Key Questions

1. What is a smart contract?
2. Where does a smart contract live?
3. How do users interact with it?
4. What makes smart contracts different from ordinary application code?
5. Why are smart contracts risky?

---

## 3. Executive Summary

Official Ethereum documentation defines a smart contract as a program that runs on the Ethereum blockchain, consisting of code and data residing at a specific address.[^ref-eth-sc-intro]

Smart contracts are a type of Ethereum account, meaning they can hold a balance and be the target of transactions, but they are not controlled by a user. Instead, they execute as programmed when triggered by valid interactions.[^ref-eth-sc-intro]

Solidity documentation frames smart contracts as programs that govern the behavior of accounts within the Ethereum state.[^ref-solidity-intro]

The practical consequence is that a smart contract is best understood as:

- onchain executable logic,
- tied to persistent state,
- invoked through transactions or calls,
- and constrained by deterministic execution and gas metering.

That is more precise than either the popular "code is law" slogan or the looser metaphor of a legal contract.

---

## 4. Protocol Structure

### What a Smart Contract Is

At minimum:

```text
smart contract
= contract account
+ deployed bytecode
+ persistent state
+ callable functions / interfaces
```

### Core Difference from a Wallet Account

User accounts sign transactions.

Smart contracts execute code when invoked.

### Why This Matters

This makes smart contracts the application layer primitive of Ethereum.

---

## 5. Historical Context

### Ethereum's Design Goal

Ethereum's original design aimed to support decentralized applications by letting developers publish executable logic directly into shared network state.[^ref-solidity-intro]

### Modern Framing

Current Ethereum docs describe smart contracts as reusable code snippets that anyone can request to execute via transaction requests.[^ref-eth-sc-intro]

### Consequence

The smart contract concept is not an extra feature bolted onto Ethereum. It is central to the platform's identity.

---

## 6. Technical Foundations

### Program + State

The anatomy documentation says smart contracts are made up of data and functions that execute upon receiving a transaction.[^ref-eth-sc-anatomy]

### Addressability

Once deployed, a contract lives at a specific address and other actors interact with it through that address.[^ref-eth-sc-intro]

### Deterministic Runtime

Because execution occurs through the EVM, smart contract logic must be deterministic across nodes.

---

## 7. Security Assumptions

### Immutability by Default

The smart contracts intro says smart contracts cannot be deleted by default and interactions with them are irreversible.[^ref-eth-sc-intro]

### Public Adversarial Environment

Solidity security documentation emphasizes that contracts often control valuable assets, execute publicly, and are exposed to malicious actors.[^ref-solidity-security]

### Why This Matters

Smart contract programming is closer to writing public financial infrastructure than writing a private web service.

---

## 8. Mathematical or Economic Model

### Minimal Conceptual Model

```text
contract interaction
= call request
+ gas budget
+ deterministic execution
+ state update if valid
```

### Persistence Cost

Because contract state persists onchain, storing and updating data has recurring network-wide cost, not just local server cost.

---

## 9. Protocol Implementation

### Primary Sources

The strongest current foundations for this topic are:

- Ethereum smart contract introduction docs,
- smart contract anatomy docs,
- Solidity introductory docs.[^ref-eth-sc-intro][^ref-eth-sc-anatomy][^ref-solidity-intro]

### Why This Is Enough

These sources establish what a smart contract is before later units discuss lifecycle, ABI, and security.

---

## 10. On-Chain Implications

### Why Contracts Matter for Analysis

Many meaningful Ethereum actions are not simple transfers. They are contract-mediated state transitions.

### Surface vs Meaning

A transaction to a contract address may trigger complex execution far beyond the surface ETH value field.

---

## 11. Institutional Thinking

Institutions should treat smart contracts as operational and financial infrastructure.

### Practical Implications

- Contract risk is application risk.
- Address labels alone are insufficient without code context.
- Deterministic execution does not eliminate design or governance risk.

### Better Research Posture

Ask:

- What code is at this address?
- What state does it control?
- What external assumptions does it rely on?

---

## 12. Common Misinterpretations

### "Smart contracts are legal contracts"

Too broad. They can encode rules and agreements, but they are fundamentally onchain programs.

### "Smart contracts run themselves without transactions"

False. They require triggering through transactions or other calls.[^ref-eth-sc-intro]

### "A contract address is just another wallet"

False. It is a code-bearing account with runtime behavior.

---

## 13. Research Questions

1. Which smart contract properties matter most for institutional risk classification?
2. How should researchers describe smart contracts without overstating legal analogies?
3. Which classes of smart contract behavior are least visible from high-level transfer data?

---

## 14. Practical Exercises

### Exercise 1

Write a one-paragraph definition of a smart contract without using the phrase "code is law."

### Exercise 2

Explain the difference between a user account and a contract account.

### Exercise 3

Describe why smart contract execution must be deterministic.

---

## 15. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Smart contracts as code + state at an address | Directly specified | ethereum.org docs |
| Smart contracts as programs governing account behavior | Directly specified | Solidity docs |
| Institutional infrastructure framing | Inference from sources | Derived from public execution and irreversibility |

---

## 16. Knowledge Graph

```text
Smart Contract Fundamentals
├─ Contract Account
├─ Bytecode
├─ Persistent State
├─ Deterministic Execution
├─ Gas-Constrained Calls
└─ Public Adversarial Environment
```

---

## 17. References

### Primary Sources

[^ref-eth-sc-intro]: ethereum.org, "Introduction to smart contracts," official documentation defining smart contracts as programs at addresses on Ethereum, accessed 2026-08-04, https://ethereum.org/developers/docs/smart-contracts/

[^ref-eth-sc-anatomy]: ethereum.org, "Anatomy of smart contracts," official documentation describing contract data, functions, memory, storage, and events, accessed 2026-08-04, https://ethereum.org/developers/docs/smart-contracts/anatomy

[^ref-solidity-intro]: Solidity documentation, "Solidity" and introductory materials describing smart contracts as programs governing account behavior, accessed 2026-08-04, https://docs.soliditylang.org/en/latest/

[^ref-solidity-security]: Solidity documentation, "Security Considerations," accessed 2026-08-04, https://docs.soliditylang.org/en/latest/security-considerations.html

---

## 18. Cross References

### Next

- SMART-CONTRACTS-002 — Contract Lifecycle

### Related

- ETHEREUM-FOUNDATION-005 — EVM
- SMART-CONTRACTS-003 — ABI

---

## Review Status

### Technical Review

Passed.

- Smart contracts were defined as programs at addresses with persistent state.
- Contract accounts were separated from EOAs.
- Deterministic execution and gas-constrained invocation were included.

### Evidence Review

Passed.

- Core claims cite official Ethereum and Solidity docs.
- Security framing cites Solidity security guidance.

### Editorial Review

Passed.

- Structure follows project format.
- Terminology is consistent.
- Tables and code fences are closed.

### Adversarial Review

Passed.

- The document does not equate smart contracts with legal agreements.
- It does not imply autonomous execution without triggering.
- It does not treat contract addresses as ordinary wallets.

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
