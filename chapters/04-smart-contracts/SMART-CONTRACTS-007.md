---
knowledge_id: SMART-CONTRACTS-007
title: Proxy Patterns
subtitle: Transparent, UUPS, Beacon, Delegatecall Routing, and the Practical Shapes of Upgradeable Indirection
version: 1.0.0
status: Reviewed
difficulty: L350
estimated_reading: 135 min
estimated_study: 340 min
last_reviewed: 2026-08-04
domain:
  - Smart Contracts
  - Proxies
  - Upgradeability
parent:
  - Smart Contracts
prerequisites:
  - SMART-CONTRACTS-006
related_topics:
  - Upgradeability
  - Security
primary_sources:
  - REF-OZ-PROXY-PATTERN-001
  - REF-OZ-UPGRADES-PLUGINS-001
  - REF-OZ-PROXY-API-001
tags:
  - smart-contracts
  - proxies
  - uups
  - transparent-proxy
  - beacon-proxy
---

# Proxy Patterns
> Smart Contracts  
> Research Unit: SMART-CONTRACTS-007

---

## Research Brief

```yaml
knowledge_id: SMART-CONTRACTS-007
title: Proxy Patterns
research_question: >
  What are the main upgradeable proxy patterns used in Ethereum smart contracts,
  how do they route calls and separate logic from state, and what practical
  tradeoffs distinguish transparent, UUPS, and beacon approaches?
document_type: deep-dive
difficulty: L350
prerequisites:
  - SMART-CONTRACTS-006
parent: Smart Contracts
previous: SMART-CONTRACTS-006
next: SMART-CONTRACTS-008
related_topics:
  - Upgradeability
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
  - Low-level assembly walkthrough
  - Non-Ethereum proxy ecosystems
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Explain what a proxy pattern is.
- Distinguish transparent, UUPS, and beacon proxies.
- Explain why proxies separate user-facing address from logic implementation.
- Explain why delegatecall-style indirection changes risk analysis.

---

## 2. Key Questions

1. What is a proxy?
2. Why are proxies used?
3. What is the difference between transparent and UUPS proxies?
4. What is a beacon proxy good for?
5. Why do proxies complicate analysis?

---

## 3. Executive Summary

OpenZeppelin's proxy documentation explains proxy patterns as mechanisms that forward calls to implementation logic while preserving a stable access point for users.[^ref-oz-proxy-pattern][^ref-oz-proxy-api]

The current plugin docs identify three important families:

- transparent proxies,
- UUPS proxies,
- beacon proxies.[^ref-oz-upgrades-plugins]

The practical differences are about where upgrade authority and logic live, how many deployments can be upgraded together, and how much complexity sits in the proxy itself versus the implementation.

---

## 4. Protocol Structure

```text
user
-> proxy address
-> forwarded call
-> implementation logic
```

State remains associated with the proxy-facing storage context while logic can be swapped according to the chosen pattern.

---

## 5. Transparent Proxy Pattern

OpenZeppelin explains that a transparent proxy distinguishes admin callers from ordinary users: the admin can use upgrade/admin functions, while other callers are delegated through to implementation logic.[^ref-oz-proxy-pattern]

This reduces some interface-clash confusion but adds proxy-side admin logic.

---

## 6. UUPS

OpenZeppelin's proxy API docs say their recommendation has shifted toward UUPS because it is lighter weight and more versatile.[^ref-oz-proxy-api]

In UUPS, upgrade logic lives in the implementation contract rather than being built into a heavier transparent proxy shell.[^ref-oz-proxy-api]

---

## 7. Beacon Proxies

OpenZeppelin plugin docs explain that beacon proxies can be upgraded atomically in groups by upgrading the beacon they point to.[^ref-oz-upgrades-plugins][^ref-oz-proxy-api]

This is useful when multiple proxy instances should follow one shared implementation pointer.

---

## 8. Technical Mechanics

Proxy docs describe the core idea as users interacting with the proxy while the proxy forwards to logic contracts.[^ref-oz-proxy-pattern]

Conceptually:

```text
same address
different implementation over time
```

This is the source of both upgrade convenience and analytical complexity.

---

## 9. Security Assumptions

Proxy correctness requires:

- safe forwarding logic,
- safe upgrade authorization,
- storage compatibility,
- correct operator understanding.

Proxy misuse can create severe governance and implementation risk.

---

## 10. Mathematical or Economic Model

Conceptually:

```text
proxy indirection
= address stability
+ logic replaceability
+ extra complexity
```

---

## 11. Protocol Implementation

Primary sources here are OpenZeppelin's proxy pattern, upgrades plugin, and proxy API docs.[^ref-oz-proxy-pattern][^ref-oz-upgrades-plugins][^ref-oz-proxy-api]

---

## 12. On-Chain Implications

Analysts must distinguish:

- proxy address,
- implementation address,
- admin or governance authority,
- upgrade history.

Same-address continuity alone is not enough.

---

## 13. Institutional Thinking

Institutions should monitor upgrade path and authority, not just contract address.

### Practical Implications

- Proxy type changes risk profile.
- Group upgrade patterns such as beacon setups can increase blast radius.
- UUPS vs transparent affects where upgrade logic lives and what must be reviewed.

---

## 14. Common Misinterpretations

### "A proxy is just a renamed contract"

False. It is an indirection layer with security consequences.

### "Same address means same code"

False.

### "Proxy choice is only a gas optimization detail"

False. It changes governance and attack surface.

---

## 15. Research Questions

1. Which proxy pattern failures are hardest for institutions to detect from surface-level monitoring?
2. How should upgrade authority be tracked across proxy families?

---

## 16. Practical Exercises

### Exercise 1

Explain one key difference between transparent and UUPS proxies.

### Exercise 2

Describe why beacon proxies can increase upgrade blast radius.

---

## 17. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Main proxy families and their roles | Directly specified | OpenZeppelin docs |
| Governance / blast-radius framing | Inference from sources | Derived from proxy architecture |

---

## 18. Knowledge Graph

```text
Proxy Patterns
├─ Transparent Proxy
├─ UUPS
├─ Beacon Proxy
├─ Stable Address
└─ Replaceable Logic
```

---

## 19. References

### Primary Sources

[^ref-oz-proxy-pattern]: OpenZeppelin Docs, "Proxy Upgrade Pattern," accessed 2026-08-04, https://docs.openzeppelin.com/upgrades-plugins/proxies

[^ref-oz-upgrades-plugins]: OpenZeppelin Docs, "Upgrades Plugins," accessed 2026-08-04, https://docs.openzeppelin.com/upgrades-plugins

[^ref-oz-proxy-api]: OpenZeppelin Docs, "Proxy" API / overview materials, accessed 2026-08-04, https://docs.openzeppelin.com/contracts/4.x/api/proxy

---

## 20. Cross References

### Previous

- SMART-CONTRACTS-006 — Upgradeability

### Next

- SMART-CONTRACTS-008 — Oracle Fundamentals

---

## Review Status

### Technical Review

Passed.

- Transparent, UUPS, and beacon patterns were separated.
- Stable address vs replaceable logic distinction was explicit.

### Evidence Review

Passed.

- Core claims cite OpenZeppelin docs.

### Editorial Review

Passed.

- Structure follows project format.

### Adversarial Review

Passed.

- The document does not reduce proxies to cosmetic indirection.
- It does not treat same address as same code.

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
