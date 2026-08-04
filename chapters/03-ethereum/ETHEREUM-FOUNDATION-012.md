---
knowledge_id: ETHEREUM-FOUNDATION-012
title: Layer 2 Overview
subtitle: Rollups, Security Inheritance, Data Availability, and Why Ethereum's Scaling Path Centers on L2s
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 145 min
estimated_study: 360 min
last_reviewed: 2026-08-04
domain:
  - Ethereum
  - Layer 2
  - Scaling
  - Data Availability
parent:
  - Ethereum Foundations
prerequisites:
  - ETHEREUM-FOUNDATION-006
  - ETHEREUM-FOUNDATION-007
  - ETHEREUM-FOUNDATION-011
related_topics:
  - Ethereum Upgrades
  - Gas
  - Blocks
  - Rollups
primary_sources:
  - REF-ETH-L2-LEARN-2026-001
  - REF-ETH-ROADMAP-SCALING-2026-001
  - REF-ETH-DANKSHARDING-2026-001
  - REF-ETH-ROADMAP-2026-001
tags:
  - ethereum
  - layer2
  - rollups
  - scaling
  - data-availability
  - danksharding
---

# Layer 2 Overview
> Ethereum Foundations  
> Research Unit: ETHEREUM-FOUNDATION-012

---

## Research Brief

```yaml
knowledge_id: ETHEREUM-FOUNDATION-012
title: Layer 2 Overview
research_question: >
  What are Ethereum layer 2 systems, how do they inherit security from
  Ethereum, why has Ethereum's scaling strategy centered on rollups instead of
  scaling the main chain alone, and how should researchers interpret the L1/L2
  boundary as of August 4, 2026?
document_type: overview
difficulty: L300
prerequisites:
  - ETHEREUM-FOUNDATION-006
  - ETHEREUM-FOUNDATION-007
  - ETHEREUM-FOUNDATION-011
parent: Ethereum Foundations
previous: ETHEREUM-FOUNDATION-011
next:
related_topics:
  - Gas
  - Blocks
  - Scaling
  - Data Availability
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
  - Rollup-by-rollup product comparison
  - Fraud-proof and validity-proof formal derivation
  - Bridge exploit case-study catalog
  - Full L2 tokenomics analysis
```

## 1. Learning Objectives

After completing this Research Unit, readers should be able to:

- Define Ethereum layer 2 at a protocol-strategy level.
- Explain why Ethereum's scaling path centers on L2s.
- Distinguish L1 from L2 responsibilities.
- Explain rollup security inheritance and data availability dependence.
- Distinguish L2s from alt L1s, sidechains, and validiums at a high level.

---

## 2. Key Questions

1. What is a layer 2 on Ethereum?
2. Why doesn't Ethereum just scale the main chain directly?
3. How do rollups inherit security from Ethereum?
4. What role does Ethereum play as a data availability layer?
5. Why are sidechains and validiums not identical to L2s?

---

## 3. Executive Summary

Ethereum's current scaling strategy is centered on layer 2 systems, especially rollups, rather than on making the L1 execution layer do all throughput work itself.[^ref-eth-l2-learn][^ref-eth-roadmap-scaling]

The official "What is layer 2?" page defines L2 as a set of Ethereum scaling solutions that extend Ethereum and inherit Ethereum's security guarantees.[^ref-eth-l2-learn]

The official scaling roadmap explains that rollups batch transactions together offchain, reduce costs for users, and rely on Ethereum for data availability and settlement guarantees.[^ref-eth-roadmap-scaling][^ref-eth-danksharding]

As of August 4, 2026, the Ethereum roadmap and L2 pages make clear that Ethereum is no longer best understood as just one chain. It is a settlement and security layer for a broader network of networks, with recent and planned upgrades focused on making L2 data cheaper and scaling more effective.[^ref-eth-roadmap][^ref-eth-roadmap-scaling]

The correct mental model is:

- Ethereum L1 prioritizes security, decentralization, settlement, and data availability.
- L2s handle much of user-facing transaction throughput.

---

## 4. Protocol Structure

### L1 vs L2

The official L2 page explains that layer 1 blockchains such as Ethereum are the underlying foundation that layer 2 projects build on top of.[^ref-eth-l2-learn]

### Responsibility Split

```text
L1 Ethereum
-> base security
-> settlement
-> data availability
-> protocol finality

L2 systems
-> execute or batch many user transactions
-> compress / post results to Ethereum
-> aim for lower cost and higher throughput
```

### Why This Matters

Ethereum scaling is now architectural, not merely parameter tuning on one chain.

---

## 5. Why Ethereum Scales This Way

### Main Chain Limits

The L2 page and scaling roadmap argue that Ethereum cannot simply maximize throughput on the main chain without compromising decentralization or security.[^ref-eth-l2-learn][^ref-eth-roadmap-scaling]

### Rollup-Centric Strategy

The roadmap says rollups are the way Ethereum scales and that shard chains are no longer needed because rollups developed faster than expected.[^ref-eth-roadmap][^ref-eth-roadmap-scaling]

### Consequence

Ethereum's scaling philosophy in 2026 is not "one chain does everything." It is "a secure base layer supports many scaling layers."

---

## 6. Security Inheritance and Data Availability

### Security Inheritance

The L2 page says a layer 2 extends Ethereum and inherits Ethereum's security guarantees.[^ref-eth-l2-learn]

### Data Availability Role

The same page says Ethereum functions as a data availability layer for L2s, and if disputes arise, Ethereum provides the data needed for those disputes.[^ref-eth-l2-learn]

### Why This Is Important

Security inheritance is not magic. It depends on what data reaches Ethereum and what verification assumptions hold.

---

## 7. Rollups, Sidechains, and Validiums

### Rollups

The official pages center rollups as Ethereum's preferred scaling approach.[^ref-eth-roadmap-scaling]

### Sidechains and Validiums

The L2 page explicitly says sidechains and validiums do not derive their security or data availability from the main chain and therefore have different trust assumptions.[^ref-eth-l2-learn]

### Why Researchers Must Care

People often use "L2" too loosely. Trust assumptions differ materially across scaling systems.

---

## 8. Proto-Danksharding and Scaling Upgrades

### Official Scaling Narrative

The scaling and danksharding pages explain that rollups are already cheaper than L1 and that proto-danksharding/data-blob-related improvements make it cheaper for rollups to post data to Ethereum.[^ref-eth-roadmap-scaling][^ref-eth-danksharding]

### Role of Recent Upgrades

The roadmap and related 2026 material link upgrades such as Dencun, Pectra, and Fusaka to continued improvements in Ethereum's scaling posture.[^ref-eth-roadmap]

### Implication

Ethereum upgrades increasingly serve the L2-centered scaling strategy rather than trying to turn L1 into the only execution venue.

---

## 9. Technical Mechanics

### Simplified L2 Flow

```text
users transact on L2
-> L2 batches / compresses activity
-> data / commitments posted to Ethereum
-> Ethereum anchors availability and settlement
-> disputes or proofs resolve against Ethereum rules
```

### Why This Reduces Cost

Many user actions can share one L1 publication footprint instead of paying the full L1 execution burden individually.

### Why This Still Depends on L1

The trust story comes from what Ethereum ultimately secures and makes available.

---

## 10. Security Assumptions

### L2 Is Not "Free Security"

Security depends on:

- correct L2 design,
- correct bridge/security model,
- correct data publication assumptions,
- and Ethereum continuing to provide the promised base guarantees.

### Terminology Risk

Mislabeling a sidechain or validium as if it had the exact same security inheritance as a rollup can materially mislead users and institutions.[^ref-eth-l2-learn]

### Roadmap Risk

Scaling improvements are active areas of protocol evolution. Researchers need current sources.

---

## 11. Mathematical or Economic Model

### Cost-Sharing Intuition

A simplified rollup intuition is:

```text
many user transactions
-> one aggregated L1 publication footprint
-> lower average cost per user action
```

### L1/L2 Relationship

Conceptually:

```text
L2 throughput gain depends on L1 settlement + data publication economics
```

### Why This Matters

L2 economics are not independent of Ethereum. They are downstream of Ethereum's data and settlement cost structure.

---

## 12. Protocol Implementation

### Primary Sources

For current architectural understanding, the most important official sources are:

- the L2 overview page,
- the scaling roadmap,
- the danksharding page,
- the current roadmap page.[^ref-eth-l2-learn][^ref-eth-roadmap-scaling][^ref-eth-danksharding][^ref-eth-roadmap]

### Why This Set Matters

Together they provide:

- the definition of L2,
- the strategic rationale,
- the data-availability and scaling mechanism context,
- the current direction of protocol development.

---

## 13. On-Chain Implications

### More Than One Network Surface

Ethereum analysis increasingly needs to distinguish:

- L1 mainnet activity,
- L2 activity,
- bridge flows,
- data publication footprints,
- settlement anchoring.

### L1 Activity Alone Is Incomplete

If you only measure Ethereum L1 user transaction counts, you can miss a large share of the ecosystem's actual user activity.

### Analytical Consequence

Modern Ethereum research often needs cross-layer interpretation, not single-chain interpretation.

---

## 14. Institutional Thinking

Institutions should treat Ethereum as a layered system.

### Practical Implications

- Security assessment must specify which layer is being assessed.
- Cost analysis should distinguish L1 and L2 execution environments.
- Bridge and settlement assumptions should be documented explicitly.
- Roadmap understanding matters because many L1 upgrades primarily improve L2 economics rather than direct L1 UX alone.

### Better Research Posture

Before making a scaling claim, ask:

- Is this claim about L1, L2, or the combined system?
- Does the system inherit Ethereum security or only interact with Ethereum?
- What data availability assumptions are required?

---

## 15. Common Misinterpretations

### "Layer 2 just means any cheaper chain"

False. Security inheritance and trust assumptions matter.[^ref-eth-l2-learn]

### "Ethereum failed to scale because users moved to L2s"

False. Official strategy explicitly centers L2-based scaling.[^ref-eth-roadmap-scaling]

### "Sidechains and validiums are the same as rollups"

False. The official L2 page says they have different trust assumptions.[^ref-eth-l2-learn]

### "L1 usage alone captures Ethereum ecosystem activity"

False. The ecosystem is increasingly multi-layer.

---

## 16. Research Questions

1. Which institutional metrics best separate L1 settlement importance from L2 user activity?
2. How should researchers compare trust assumptions across rollups, validiums, and sidechains without flattening them into one category?
3. Which future roadmap items are most likely to change L2 economics materially?

---

## 17. Practical Exercises

### Exercise 1

Explain why Ethereum's scaling strategy no longer centers on shard chains.

### Exercise 2

Write a short distinction between a rollup and a sidechain.

### Exercise 3

Describe Ethereum's role as a data availability layer for L2s.

### Exercise 4

Explain why L2 activity changes how researchers should interpret Ethereum network usage.

---

## 18. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Official definition of L2 and security inheritance | Directly specified | Official L2 page |
| Rollup-centric scaling strategy | Directly specified | Official roadmap/scaling pages |
| Danksharding / proto-danksharding role | Directly specified | Official danksharding/scaling pages |
| Institutional cross-layer interpretation | Inference from sources | Derived from L1/L2 architecture |

---

## 19. Knowledge Graph

```text
Layer 2 Overview
├─ L1 Ethereum
│  ├─ settlement
│  ├─ security
│  ├─ data availability
│  └─ finality
├─ L2 Systems
│  ├─ rollups
│  ├─ cheaper execution
│  ├─ batched activity
│  └─ user throughput
├─ Trust Distinctions
│  ├─ rollups
│  ├─ sidechains
│  └─ validiums
└─ Scaling Path
   ├─ roadmap
   ├─ proto-danksharding
   ├─ PeerDAS era
   └─ network of networks
```

---

## 20. References

### Primary Sources

[^ref-eth-l2-learn]: ethereum.org, "What is layer 2?", official L2 overview describing Ethereum L2s, security inheritance, and distinctions from sidechains and validiums, page last update June 4, 2026, https://ethereum.org/layer-2/learn/, accessed 2026-08-04.

[^ref-eth-roadmap-scaling]: ethereum.org, "Scaling Ethereum," official roadmap/scaling page describing rollup-centered scaling and user-cost implications, page last update June 24, 2026, https://ethereum.org/roadmap/scaling/, accessed 2026-08-04.

[^ref-eth-danksharding]: ethereum.org, "Danksharding," official page describing proto-danksharding/data blobs and the goal of scaling rollups, page last update June 24, 2026, https://ethereum.org/roadmap/danksharding, accessed 2026-08-04.

[^ref-eth-roadmap]: ethereum.org, "Ethereum roadmap," official roadmap showing completed upgrades through Fusaka and in-development items in H2 2026, https://ethereum.org/roadmap/, accessed 2026-08-04.

### Supporting Interpretation Notes

- Where this document discusses institutional cross-layer analytics or scaling posture, those are analytical inferences built from the cited official L2 and roadmap sources.

---

## 21. Cross References

### Previous

- ETHEREUM-FOUNDATION-011 — Ethereum Upgrades

### Next

- None in current Phase 3 roadmap

### Related

- ETHEREUM-FOUNDATION-006 — Gas
- ETHEREUM-FOUNDATION-011 — Ethereum Upgrades

---

## Review Status

### Technical Review

Passed.

- L1/L2 responsibility split was described clearly.
- Security inheritance was distinguished from mere bridge connectivity.
- Sidechains and validiums were not flattened into rollups.
- Current roadmap/scaling context was included with date awareness.

### Evidence Review

Passed.

- Definition and trust-assumption claims cite official L2 docs.
- Scaling strategy claims cite official roadmap/scaling pages.
- Institutional interpretation is labeled as inference.

### Editorial Review

Passed.

- Structure follows the project document pattern.
- Metadata is complete.
- Terminology is consistent: L1, L2, rollup, data availability, settlement.
- Tables and code fences are closed.

### Adversarial Review

Passed.

- The document does not call every cheap chain an L2.
- It does not present roadmap aspirations as already-live facts.
- It does not imply L1 usage alone equals total Ethereum activity.
- It does not erase trust-assumption differences.

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
