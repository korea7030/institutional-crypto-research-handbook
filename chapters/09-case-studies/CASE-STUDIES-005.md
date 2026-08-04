---
knowledge_id: CASE-STUDIES-005
title: FTX Collapse
subtitle: Custody Failure, Commingling, and the Breakdown of Trusted Intermediation
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 100 min
estimated_study: 230 min
last_reviewed: 2026-08-04
domain:
  - Case Studies
  - Centralized Exchanges
  - Systemic Risk
parent:
  - Case Studies
prerequisites:
  - DEFI-003
  - IRF-001
related_topics:
  - March 2020 Crash
  - Metric Limitations
primary_sources:
  - REF-SEC-FTX-2022-001
  - REF-CFTC-FTX-2022-001
  - REF-DOJ-FTX-2022-001
tags:
  - case-study
  - ftx
  - custody
---

# FTX Collapse
> Case Studies  
> Research Unit: CASE-STUDIES-005

---

## Research Brief

```yaml
knowledge_id: CASE-STUDIES-005
title: FTX Collapse
research_question: >
  What did the FTX collapse reveal about centralized-exchange counterparty risk,
  and why should researchers separate crypto-market volatility from custody and
  governance failures inside offchain intermediaries?
document_type: case-study
difficulty: L400
prerequisites:
  - DEFI-003
  - IRF-001
parent: Case Studies
previous: CASE-STUDIES-004
next: CASE-STUDIES-006
```

## 1. Observation

In November 2022, FTX failed as a centralized intermediary, and regulators later alleged large-scale misuse and commingling of customer assets.

## 2. Key Dates

- November 11, 2022: the conduct period described in later CFTC filings ran through November 11, 2022.[^ref-cftc-ftx]
- December 13, 2022: the SEC charged Sam Bankman-Fried with defrauding equity investors in FTX and alleged concealment of the diversion of customer funds to Alameda Research.[^ref-sec-ftx]
- December 13, 2022: the DOJ announced fraud, money laundering, and campaign finance charges.[^ref-doj-ftx]

## 3. Event Structure

This was not primarily a "crypto protocol" failure. It was a centralized-intermediary failure involving:

- custody misrepresentation,
- internal controls failure,
- asset commingling,
- and governance breakdown.

## 4. Why It Mattered

FTX broke one of the most important distinctions in crypto:

- protocol risk,
- versus intermediary risk.

The collapse reminded the market that crypto-native assets can still be held inside very traditional failure modes.

## 5. Lessons

- Self-custody and onchain transparency are not marketing slogans; they change failure surfaces.
- A firm can appear sophisticated while still failing basic segregation discipline.
- Counterparty risk can dominate token or protocol analysis in centralized venues.

## 6. Institutional Thinking

- FTX should not be read as proof that blockchains failed. It is evidence that opaque intermediaries remain a core crypto risk class.
- Researchers must distinguish exchange solvency and governance from market-direction calls.

## 7. References

[^ref-sec-ftx]: SEC, "SEC Charges Samuel Bankman-Fried with Defrauding Investors in Crypto Asset Trading Platform FTX," December 13, 2022, https://www.sec.gov/newsroom/press-releases/2022-219

[^ref-cftc-ftx]: CFTC, "CFTC Charges Alameda CEO and Alameda and FTX Co-Founder with Fraud in Action Against Sam Bankman-Fried and his Companies," December 21, 2022, https://www.cftc.gov/PressRoom/PressReleases/8644-22

[^ref-doj-ftx]: U.S. Department of Justice, "FTX Founder Indicted for Fraud, Money Laundering, and Campaign Finance Offenses," December 13, 2022, https://www.justice.gov/archives/opa/pr/ftx-founder-indicted-fraud-money-laundering-and-campaign-finance-offenses

## 8. Cross References

- Previous: CASE-STUDIES-004 — Terra Collapse
- Next: CASE-STUDIES-006 — Ethereum Merge

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
