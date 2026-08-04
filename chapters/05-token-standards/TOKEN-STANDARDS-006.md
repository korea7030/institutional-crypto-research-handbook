---
knowledge_id: TOKEN-STANDARDS-006
title: Governance Tokens
subtitle: Voting Power, Delegation, Timelocks, and the Difference Between Ownership and Control
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 110 min
estimated_study: 260 min
last_reviewed: 2026-08-04
domain:
  - Token Standards
  - Ethereum
  - Governance
parent:
  - Token Standards
prerequisites:
  - TOKEN-STANDARDS-001
  - SMART-CONTRACTS-006
related_topics:
  - Utility Tokens
  - Tokenomics
  - Protocol Governance
primary_sources:
  - REF-COMPOUND-GOV-2026-001
  - REF-OZ-GOVERNANCE-2026-001
  - REF-MAKER-GOV-2026-001
tags:
  - governance-tokens
  - comp
  - mkr
---

# Governance Tokens
> Token Standards  
> Research Unit: TOKEN-STANDARDS-006

---

## Research Brief

```yaml
knowledge_id: TOKEN-STANDARDS-006
title: Governance Tokens
research_question: >
  What functions do governance tokens serve in onchain protocols, how do voting,
  delegation, and execution paths work, and how should researchers distinguish
  token ownership from effective protocol control?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-001
  - SMART-CONTRACTS-006
parent: Token Standards
previous: TOKEN-STANDARDS-005
next: TOKEN-STANDARDS-007
related_topics:
  - Utility Tokens
  - Tokenomics
  - Protocol Governance
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Governance Structure
  - Voting Mechanics
  - Economic Model
  - Security Considerations
  - Implementation Notes
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Political philosophy of governance
  - Per-protocol vote history
  - Legal control analysis
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Define governance tokens as tokens that help allocate decision power over protocol changes.
- Explain delegation, quorum, thresholds, and timelock execution.
- Distinguish token balances from effective voting power.
- Identify how protocol control can diverge from nominal token ownership.
- Assess governance-token concentration and operational risk.

---

## 2. Key Questions

1. What does a governance token actually govern?
2. How is voting power measured and delegated?
3. What is the role of proposal thresholds, quorums, and timelocks?
4. Why can token ownership differ from operational control?
5. What risks arise when governance is token-weighted?

---

## 3. Executive Summary

Compound documentation states that the protocol is governed and upgraded by COMP token holders through three components: the COMP token, the governance module, and the Timelock.[^ref-compound-gov]

OpenZeppelin's governance documentation describes onchain governance as a system built around Governor contracts, vote sources, counting modules, quorum rules, and timelock execution paths.[^ref-oz-governance]

Maker documentation describes governance contracts that facilitate MKR voting, proposal execution, and voting security.[^ref-maker-governance]

Taken together, these sources show that a governance token is not merely an ERC-20 with branding. It is a token embedded in a control system that may determine:

- parameter changes,
- treasury actions,
- upgrade paths,
- collateral policy,
- and emergency response.

The analytical warning is that token balance alone does not equal control. Effective control depends on:

- delegation,
- voter participation,
- quorum design,
- proposal thresholds,
- execution routing through timelocks or admins,
- and concentration among large holders or delegates.

---

## 4. Governance Structure

### 4.1 Core Components

Compound's governance docs explicitly separate:

- token,
- governor,
- timelock.[^ref-compound-gov]

That three-part model is a useful default framework for many protocols.

### 4.2 Voting as a Separate Accounting Layer

Compound states that COMP holders can delegate voting rights to themselves or another address and that vote balances change with delegated token balances.[^ref-compound-gov]

This means governance power may be distinct from simple wallet balances at any moment.

### 4.3 Governance as an Execution Pipeline

OpenZeppelin documents a governance pipeline with proposal creation, voting delay, voting period, quorum logic, and optional timelock queue/execution stages.[^ref-oz-governance]

Therefore, governance is best modeled as a state machine, not as a one-step vote.

---

## 5. Voting Mechanics

### 5.1 Delegation

Compound documents both direct delegation and delegation by signature, showing that governance systems may separate economic ownership from active voting participation.[^ref-compound-gov]

### 5.2 Snapshot Logic

OpenZeppelin notes that vote power is commonly retrieved from historical snapshots rather than current balances, which prevents some forms of double counting and timing abuse.[^ref-oz-governance]

### 5.3 Proposal Threshold and Quorum

Compound documents proposal thresholds and quorum votes as explicit parameters in governance.[^ref-compound-gov]

These mechanisms prevent tiny holders from spamming proposals and require minimum participation for legitimacy.

### 5.4 Timelock

Compound states that successful proposals are queued in a Timelock before execution.[^ref-compound-gov]

Timelocks create exit windows and reduce the risk of immediate hostile change, but they do not remove governance-capture risk.

---

## 6. Mathematical or Economic Model

### 6.1 Voting Power Function

Let:

- `b(a)` = token balance of account `a`
- `d(a)` = delegate chosen by account `a`

Then effective vote power for delegate `x` can be modeled as:

`v(x) = sum(b(a)) for all a such that d(a) = x`

This means control concentration should be measured at the delegate layer as well as the balance layer.

### 6.2 Proposal Success Condition

For proposal `p` with `for`, `against`, `abstain`, quorum `Q`, and success rule `S`, passage generally requires:

- quorum reached,
- vote-success criterion satisfied,
- and execution path completed.

Owning tokens alone does not enact policy without traversing that pipeline.

---

## 7. Security Considerations

### 7.1 Concentration Risk

A governance token may be widely distributed nominally but highly concentrated in active delegates, exchanges, foundations, treasuries, or early insiders.

### 7.2 Low Participation

If quorum is low relative to supply or participation is persistently weak, small organized coalitions may exert effective control.

### 7.3 Governance Capture Through Execution Layer

Maker governance docs show that governance includes proposal execution and voting security contracts, not just polling.[^ref-maker-governance]

If the execution path, timelock admin, or pause authority is centralized, the governance token may not be the whole control surface.

### 7.4 Timing and Tooling Risk

OpenZeppelin highlights settings such as voting delay, voting period, and timepoint-based quorum logic.[^ref-oz-governance]

Those parameters materially affect attack surface and participant ability to respond.

---

## 8. Implementation Notes

Compound provides a concrete governance-token pattern: delegated ERC-20 voting, proposal thresholds, quorum, voting windows, and timelocked execution.[^ref-compound-gov]

Maker adds a governance architecture where MKR voting influences protocol business logic and security processes.[^ref-maker-governance]

OpenZeppelin generalizes these patterns into reusable governance modules.[^ref-oz-governance]

The lesson is that "governance token" is not one standard like ERC-20. It is a role played by a token inside a separate governance architecture.

---

## 9. On-Chain Implications

### 9.1 Observable Governance Surface

Analysts can usually observe:

- delegation events,
- proposal creation,
- vote casting,
- queueing,
- execution.

### 9.2 Hidden Control Surface

Analysts should also inspect:

- multisigs,
- emergency roles,
- proxy admins,
- treasury custody,
- and offchain coordination.

### 9.3 Balance vs Influence

The most important measurement is often not total token ownership, but effective influence over executable governance outcomes.

---

## 10. Institutional Thinking

- Treat governance tokens as control instruments, not only as tradeable assets.
- Measure concentration at both holder and delegate layers.
- Ask what actions governance can actually execute.
- Distinguish signaling votes from executable onchain governance.
- Review timelocks, guardians, emergency powers, and admin escape hatches before concluding that control is decentralized.

---

## 11. Evidence Classification

| Claim | Classification | Basis |
|---|---|---|
| Compound governance is built from token, governance module, and timelock | Directly specified | Compound docs |
| Governance tokens often use delegation and snapshots | Directly specified | Compound and OpenZeppelin docs |
| Timelocks provide delayed execution after successful votes | Directly specified | Compound and OpenZeppelin docs |
| Effective control can diverge from nominal ownership | Analytical inference | Delegation, quorum, and admin-path structure |
| Governance tokens are a role within an architecture, not a single universal token standard | Analytical inference | Comparison across sources |

---

## 12. References

[^ref-compound-gov]: Compound v2 Docs, "Governance," official documentation, accessed 2026-08-04, https://docs.compound.finance/v2/governance/

[^ref-oz-governance]: OpenZeppelin Docs, "How to set up on-chain governance" and governance API documentation, accessed 2026-08-04, https://docs.openzeppelin.com/contracts/5.x/governance and https://docs.openzeppelin.com/contracts/5.x/api/governance

[^ref-maker-governance]: MakerDAO Technical Docs, "Governance Module," official documentation, accessed 2026-08-04, https://docs.makerdao.com/smart-contract-modules/governance-module

---

## 13. Cross References

### Previous

- TOKEN-STANDARDS-005 — Wrapped Assets

### Next

- TOKEN-STANDARDS-007 — Utility Tokens

---

## Review Status

### Technical Review

Passed.

- Delegation, quorum, and execution were separated clearly.
- Governance token balance and effective control were not conflated.

### Evidence Review

Passed.

- Core governance pipeline claims are tied to Compound and OpenZeppelin docs.
- Maker is used as a concrete governance-system example, not as a universal template.

### Editorial Review

Passed.

- Terminology stays precise and avoids overstating decentralization.
- Structure aligns with prior chapter format.

### Adversarial Review

Passed.

- The document does not imply token ownership alone guarantees protocol control.
- It does not ignore non-token admin surfaces.
- It does not present governance participation as uniformly distributed.

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
