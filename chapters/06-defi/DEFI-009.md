---
knowledge_id: DEFI-009
title: Restaking
subtitle: Reusing Staked Security, AVS Risk Extension, and the New Shared-Security Layer on Top of Ethereum
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 110 min
estimated_study: 270 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Security
  - Ethereum
parent:
  - DeFi
prerequisites:
  - DEFI-008
related_topics:
  - Staking
  - Bridges
  - DeFi Risks
primary_sources:
  - REF-EIGEN-WP-ANN-2023-001
  - REF-EIGEN-FORUM-RES-2023-001
tags:
  - restaking
  - eigenlayer
  - shared-security
---

# Restaking
> DeFi  
> Research Unit: DEFI-009

---

## Research Brief

```yaml
knowledge_id: DEFI-009
title: Restaking
research_question: >
  What is restaking, how does it reuse existing staked capital for additional
  services, and why can shared-security design create both capital efficiency
  and correlated-risk expansion?
document_type: foundation
difficulty: L400
prerequisites:
  - DEFI-008
parent: DeFi
previous: DEFI-008
next: DEFI-010
```

## 1. Learning Objectives

- Define restaking precisely.
- Explain how security is extended beyond Ethereum base consensus.
- Distinguish native restaking and LST-based restaking.
- Identify correlated slashing and shared-security risks.

## 2. Executive Summary

EigenLayer's official forum announcement for the whitepaper describes restaking as a core protocol idea under implementation, centered on additional slashing conditions and operator participation beyond base Ethereum validation.[^ref-eigen-ann]

Official EigenLayer resource material presents restaking as a way for Ethereum-staked capital and related assets to secure additional services.[^ref-eigen-res]

The main conceptual shift is that the same economic stake is reused for more than one security domain.

## 3. Mechanism

Restaking generally involves opt-in extension of slashing or delegated security conditions so that capital already committed to Ethereum staking or staking derivatives also backs additional services.

## 4. Benefits

- shared security bootstrap,
- faster network/service launch,
- additional yield opportunities for participants.

## 5. Risks

- correlated slashing,
- operator concentration,
- opaque AVS risk,
- stacked dependency on LST and restaking layers.

## 6. Institutional Thinking

- Restaking is not free extra yield; it is security reuse with potentially compounded downside.
- The key question is not only how much stake backs a service, but under what failure domains and slashing conditions.

## 7. References

[^ref-eigen-ann]: EigenLayer Forum, "Announcing the EigenLayer Whitepaper v1.0," official forum announcement, published 2023-02-21, accessed 2026-08-04, https://forum.eigenlayer.xyz/t/announcing-the-eigenlayer-whitepaper-v1-0/3432

[^ref-eigen-res]: EigenLayer Forum, "Learn about EigenLayer," official resource post, published 2023-04-18, accessed 2026-08-04, https://forum.eigenlayer.xyz/t/learn-about-eigenlayer/3418

## 8. Cross References

- Previous: DEFI-008 — Staking
- Next: DEFI-010 — Bridges

## Review Status

### Technical Review
Passed.

### Evidence Review
Passed.

### Editorial Review
Passed.

### Adversarial Review
Passed.

### Quality Gate

| Gate | Status |
|---|---|
| Research scope was followed | Pass |
| Required primary sources were reviewed | Pass |
| Material claims are cited | Pass |
| Fact and interpretation are separated | Pass |
| No unresolved critical review issue remains | Pass |
