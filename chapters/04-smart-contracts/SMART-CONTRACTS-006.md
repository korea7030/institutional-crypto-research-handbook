---
knowledge_id: SMART-CONTRACTS-006
title: Upgradeability
subtitle: Immutability Tension, Virtual Upgrades, Storage Layout Constraints, and Governance Tradeoffs
version: 1.0.0
status: Reviewed
difficulty: L350
estimated_reading: 135 min
estimated_study: 340 min
last_reviewed: 2026-08-04
domain:
  - Smart Contracts
  - Upgradeability
  - Ethereum
parent:
  - Smart Contracts
prerequisites:
  - SMART-CONTRACTS-002
  - SMART-CONTRACTS-005
related_topics:
  - Proxy Patterns
  - Security
primary_sources:
  - REF-OZ-UPGRADING-001
  - REF-OZ-PROXY-PATTERN-001
tags:
  - smart-contracts
  - upgradeability
  - proxies
  - storage-layout
---

# Upgradeability
> Smart Contracts  
> Research Unit: SMART-CONTRACTS-006

---

## Research Brief

```yaml
knowledge_id: SMART-CONTRACTS-006
title: Upgradeability
research_question: >
  How do Ethereum developers reconcile code immutability with the practical need
  to patch and improve live systems, and what technical and governance tradeoffs
  arise when using upgradeable contract patterns?
document_type: deep-dive
difficulty: L350
prerequisites:
  - SMART-CONTRACTS-002
  - SMART-CONTRACTS-005
parent: Smart Contracts
previous: SMART-CONTRACTS-005
next: SMART-CONTRACTS-007
related_topics:
  - Proxy Patterns
  - Security
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
  - Tool-by-tool walkthrough
  - Governance process design catalog
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Explain why upgradeability exists despite contract immutability.
- Describe virtual upgrade logic at a high level.
- Explain storage layout constraints.
- Explain why upgradeability introduces governance and security tradeoffs.

---

## 2. Key Questions

1. Why make contracts upgradeable?
2. How do virtual upgrades work conceptually?
3. Why is storage layout a critical constraint?
4. What new risks does upgradeability create?

---

## 3. Executive Summary

OpenZeppelin's upgrade docs explain the basic tension clearly: smart contracts are immutable by default, but some systems need to fix bugs, add features, or change rules without forcing users to migrate to a new address.[^ref-oz-upgrading]

Their upgrade model keeps the user-facing address stable while changing the implementation logic behind that address.[^ref-oz-upgrading][^ref-oz-proxy-pattern]

This solves one operational problem but creates others:

- storage layout must remain compatible,
- upgrade authority becomes a governance and security concern,
- "immutability" becomes conditional rather than absolute.

---

## 4. Protocol Structure

```text
user-facing access point stays
-> implementation logic may change
-> state must remain coherent
-> governance decides when change occurs
```

---

## 5. Why Upgradeability Exists

OpenZeppelin notes that without upgradeability, bug fixes may require deploying a new contract, migrating state, updating integrations, and convincing users to move.[^ref-oz-upgrading]

That is operationally expensive and risky.

---

## 6. Technical Mechanics

The OpenZeppelin guide summarizes an upgrade as:

1. deploy a new implementation contract,
2. send a transaction that updates the proxy's implementation reference.[^ref-oz-upgrading]

This preserves address, state, and balance at the user-facing entry point.

---

## 7. Storage Layout Constraints

OpenZeppelin explicitly warns that you cannot arbitrarily change storage layout during upgrades: removing variables, changing types, or inserting variables before existing ones can break state interpretation.[^ref-oz-upgrading]

This is one of the central technical constraints of upgradeability.

---

## 8. Security Assumptions

Upgradeability adds governance risk and key-management risk. Whoever can authorize upgrades has substantial power over live contract behavior.

It also adds implementation risk because a bad upgrade can damage a previously safe system.

---

## 9. Mathematical or Economic Model

Conceptually:

```text
immutability benefit - flexibility benefit = core tradeoff tension
```

Upgradeability moves the system along that tradeoff curve toward flexibility.

---

## 10. Protocol Implementation

Primary sources here are OpenZeppelin's upgrading guide and proxy-pattern explanation.[^ref-oz-upgrading][^ref-oz-proxy-pattern]

---

## 11. On-Chain Implications

Upgradeable systems can preserve address continuity while materially changing behavior. Analysts therefore cannot assume "same address" means "same rules forever."

---

## 12. Institutional Thinking

Institutions should evaluate upgradeability as both a technical design and a governance power structure.

### Practical Implications

- Stable address does not imply immutable logic.
- Upgrade authority should be documented and monitored.
- Storage-layout safety is a core due-diligence topic.

---

## 13. Common Misinterpretations

### "Upgradeable means mutable in the casual software sense"

Too loose. The mechanism is indirect and constrained.

### "Upgradeable always means safer"

False. It can reduce bug-fix friction while adding governance and implementation risk.

---

## 14. Research Questions

1. Which governance controls best reduce upgrade abuse risk?
2. How should institutions classify upgradeable vs non-upgradeable contract exposure?

---

## 15. Practical Exercises

### Exercise 1

Explain why preserving address and preserving logic are different things.

### Exercise 2

Describe why storage layout matters during upgrades.

---

## 16. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Immutability tension and virtual upgrade process | Directly specified | OpenZeppelin docs |
| Storage layout constraints | Directly specified | OpenZeppelin docs |
| Governance-tradeoff framing | Inference from sources | Derived from upgrade authority implications |

---

## 17. Knowledge Graph

```text
Upgradeability
├─ Immutability Tension
├─ Stable Address
├─ Replaceable Logic
├─ Storage Layout Constraints
└─ Governance Authority
```

---

## 18. References

### Primary Sources

[^ref-oz-upgrading]: OpenZeppelin Docs, "Upgrading smart contracts," accessed 2026-08-04, https://docs.openzeppelin.com/contracts/5.x/learn/upgrading-smart-contracts

[^ref-oz-proxy-pattern]: OpenZeppelin Docs, "Proxy Upgrade Pattern," accessed 2026-08-04, https://docs.openzeppelin.com/upgrades-plugins/proxies

---

## 19. Cross References

### Previous

- SMART-CONTRACTS-005 — Security Considerations

### Next

- SMART-CONTRACTS-007 — Proxy Patterns

---

## Review Status

### Technical Review

Passed.

- Upgradeability mechanics and storage constraints were covered.
- Governance tradeoffs were included.

### Evidence Review

Passed.

- Core claims cite OpenZeppelin docs.

### Editorial Review

Passed.

- Structure follows project format.

### Adversarial Review

Passed.

- The document does not treat stable addresses as proof of immutable logic.
- It does not overstate upgradeability as costless.

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
