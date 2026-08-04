---
knowledge_id: CASE-STUDIES-004
title: Terra Collapse
subtitle: Algorithmic Stablecoin Failure, Reflexive Bank-Run Dynamics, and Confidence Destruction
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 100 min
estimated_study: 230 min
last_reviewed: 2026-08-04
domain:
  - Case Studies
  - Stablecoins
  - Systemic Risk
parent:
  - Case Studies
prerequisites:
  - TOKEN-STANDARDS-004
  - DEFI-012
related_topics:
  - Stablecoin Supply
  - DeFi Risks
primary_sources:
  - REF-SEC-TERRA-2023-001
  - REF-TERRA-REVIVE-2022-001
tags:
  - case-study
  - terra
  - ust
---

# Terra Collapse
> Case Studies  
> Research Unit: CASE-STUDIES-004

---

## Research Brief

```yaml
knowledge_id: CASE-STUDIES-004
title: Terra Collapse
research_question: >
  What failed in the Terra ecosystem in May 2022, and how should researchers
  understand the collapse as a reflexive stablecoin-confidence breakdown rather
  than as a simple one-token price crash?
document_type: case-study
difficulty: L400
prerequisites:
  - TOKEN-STANDARDS-004
  - DEFI-012
parent: Case Studies
previous: CASE-STUDIES-003
next: CASE-STUDIES-005
```

## 1. Observation

In May 2022, UST lost its dollar peg and LUNA entered a reflexive collapse, destroying the core confidence structure of the Terra ecosystem.

## 2. Key Dates

- May 2022: the SEC later alleged that UST depegged from the U.S. dollar and that UST and related tokens fell close to zero.[^ref-sec-terra]
- May 16, 2022: Terraform's "Terra Ecosystem Revival Plan 2" was posted to coordinate a new network launch.[^ref-terra-revive]

## 3. Event Structure

The system depended on confidence in redemption logic and endogenous demand. When confidence broke, the stabilization mechanism amplified stress instead of absorbing it.

This was a classic reflexive failure:

- peg weakness undermined trust,
- trust loss drove exits,
- exits worsened the mechanism's ability to stabilize.

## 4. Why It Mattered

Terra was a case study in why algorithmic or reflexive stablecoin designs can fail discontinuously. It also demonstrated how yield narratives and ecosystem growth can mask fragility in the monetary core.

## 5. Lessons

- Stablecoin credibility is a system property, not a token-label property.
- Reflexive stabilization can work until confidence suddenly stops compounding in the right direction.
- Ecosystem TVL and user growth are not substitutes for monetary resilience.

## 6. Institutional Thinking

- Terra should be analyzed as a confidence-run case, not just as a "bad tokenomics" case.
- The decisive variable was not only mechanism design but whether the mechanism could survive coordinated exit behavior.

## 7. References

[^ref-sec-terra]: SEC, "Terraform Labs PTE Ltd and Do Hyeong Kwon," litigation release and complaint summary, February 16, 2023, https://www.sec.gov/enforcement-litigation/litigation-releases/lr-25692

[^ref-terra-revive]: Terra Research Forum, "Terra Ecosystem Revival Plan 2 [PASSED GOV]," May 16, 2022, https://classic-agora.terra.money/t/terra-ecosystem-revival-plan-2-passed-gov/18498

## 8. Cross References

- Previous: CASE-STUDIES-003 — DeFi Summer
- Next: CASE-STUDIES-005 — FTX Collapse

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
