---
knowledge_id: BITCOIN-003
title: Whitepaper Section 2 — Transactions
version: 1.0.0
status: Draft
difficulty: L200
estimated_reading: 45 min
estimated_study: 120 min
author: Institutional Crypto Research Handbook
prerequisites:
  - BITCOIN-002
related_topics:
  - UTXO
  - Digital Signatures
  - Public Key Cryptography
primary_sources:
  - Bitcoin: A Peer-to-Peer Electronic Cash System (2008)
tags:
  - bitcoin
  - transactions
  - digital-signature
  - utxo
---

# Whitepaper Section 2 — Transactions

> Research Unit: BITCOIN-003

---

# Learning Objectives

After completing this Research Unit, readers should be able to:

- Explain how Bitcoin defines ownership.
- Understand the role of digital signatures.
- Explain the "chain of digital signatures."
- Understand why digital signatures alone cannot prevent Double Spending.
- Prepare for the UTXO model introduced later in this handbook.

---

# Section Overview

Section 2 introduces the concept of a Bitcoin transaction.

Contrary to common intuition,

Bitcoin does **not** move "coins" between accounts.

Instead,

Bitcoin transfers the **right to spend** a previous transaction output.

Ownership is represented cryptographically,

not by account balances.

---

# Original Text (Summary)

The Whitepaper explains that an electronic coin is represented as:

> A chain of digital signatures.

Each owner transfers ownership by:

1. Signing the previous transaction.
2. Including the recipient's public key.
3. Broadcasting the transaction to the network.

This creates an unbroken chain of ownership.

---

# Core Concept

Bitcoin does not maintain bank accounts.

Instead,

ownership is proven through cryptographic history.

A simplified example:

```
Alice

↓

Bob

↓

Charlie

↓

David
```

Each transfer references the previous transfer.

The chain itself becomes the ownership record.

---

# Why Use Digital Signatures?

## FACT (ECL-A)

Digital signatures provide three important properties.

### Authentication

The network can verify who authorized the transaction.

---

### Integrity

Any modification invalidates the signature.

---

### Non-Repudiation

The signer cannot later deny having authorized the transaction.

---

Digital signatures therefore prove authorization.

They do **not** determine transaction ordering.

---

# Chain of Digital Signatures

Suppose Alice owns a transaction output.

She transfers it to Bob.

Conceptually,

the transaction contains:

```
Previous Transaction

↓

Recipient Public Key

↓

Alice's Digital Signature
```

Bob later transfers ownership to Charlie.

```
Alice

↓

Bob

↓

Charlie
```

Every transfer references the previous one.

Ownership is therefore represented by history.

---

# Why Public Keys Matter

Bitcoin does not identify users.

It identifies cryptographic keys.

Ownership means:

> Possession of the private key capable of producing a valid signature.

This is fundamentally different from traditional banking.

Banks identify customers.

Bitcoin identifies authorization.

---

# A Common Misunderstanding

Many beginners imagine Bitcoin as digital coins stored in a wallet.

This is inaccurate.

Wallets do not literally contain bitcoins.

Wallets contain:

- Private keys
- Public keys
- Metadata

The blockchain contains transaction history.

Ownership is reconstructed from that history.

---

# Why Digital Signatures Are Not Enough

Suppose Alice signs two transactions.

```
Transaction A

↓

Bob
```

and simultaneously

```
Transaction B

↓

Charlie
```

Both signatures are valid.

Which transaction should the network accept?

Digital signatures cannot answer this question.

Consensus is still required.

This is precisely why Section 3 introduces the Timestamp Server.

---

# Relationship to the UTXO Model

Although the Whitepaper has not yet introduced the term UTXO,

Section 2 lays the conceptual foundation.

Each transaction consumes previous outputs

and creates new outputs.

Later implementations formalized this model as the

**Unspent Transaction Output (UTXO)** model.

Understanding this section makes the UTXO model intuitive rather than confusing.

---

# Historical Context (2008)

In 2008,

most electronic payment systems were account-based.

Examples included:

- Banks
- Credit cards
- PayPal

Bitcoin proposed a radically different approach.

Instead of updating balances,

it recorded ownership transfers.

This distinction remains one of Bitcoin's defining characteristics.

---

# Modern Context (2026)

Today,

Bitcoin Core,

hardware wallets,

block explorers,

and on-chain analytics all rely upon the transaction model introduced in this section.

Although Bitcoin has evolved through upgrades such as SegWit and Taproot,

the underlying ownership model remains unchanged.

The chain of transaction history continues to define ownership.

---

# Implementation Notes

Bitcoin Core stores transactions using:

- Inputs
- Outputs
- Scripts
- Locking conditions
- Unlocking conditions

These implementation details extend the simplified explanation presented in the Whitepaper.

Readers should therefore distinguish between:

- Whitepaper concepts
- Bitcoin Core implementation

Both describe the same system at different levels of abstraction.

---

# Institutional Thinking

Professional researchers ask:

Observation

Ownership is represented by transaction history.

↓

Question

How does the network determine which history is valid?

↓

Evidence

Digital signatures prove authorization,

but not ordering.

↓

Research Question

Which mechanism establishes global ordering?

This naturally leads to the Timestamp Server described in the next section.

---

# Common Misinterpretations

## Misinterpretation 1

"Bitcoin stores balances."

Incorrect.

Bitcoin stores transactions.

Balances are derived from transaction outputs.

---

## Misinterpretation 2

"A wallet contains bitcoins."

Incorrect.

A wallet contains private keys.

The blockchain contains transaction history.

---

## Misinterpretation 3

"Digital signatures prevent Double Spending."

Incorrect.

Digital signatures prove authorization.

Consensus determines which authorized transaction becomes part of the accepted history.

---

# Key Takeaways

- Bitcoin ownership is represented by transaction history.
- Transactions form a chain of digital signatures.
- Digital signatures prove authorization.
- Consensus determines transaction ordering.
- Section 2 establishes the conceptual foundation for the UTXO model.

---

# Research Questions

1. Why did Bitcoin avoid account balances?
2. Why are transaction outputs preferable for decentralized verification?
3. How does the UTXO model emerge from this section?
4. Which responsibilities belong to digital signatures?
5. Which responsibilities belong to consensus?

---

# References

## Primary Source

1. Satoshi Nakamoto.
   *Bitcoin: A Peer-to-Peer Electronic Cash System.*
   Section 2 — Transactions.

---

# Cross References

Previous

- BITCOIN-002 — Whitepaper Section 1: Introduction

Next

- BITCOIN-004 — Whitepaper Section 3: Timestamp Server

Related

- BITCOIN-014 — UTXO Model
- BITCOIN-015 — Transactions in Depth
- BLOCKCHAIN-FOUNDATION-001 — The Double Spending Problem

---

Status: Draft

Review: Pending
