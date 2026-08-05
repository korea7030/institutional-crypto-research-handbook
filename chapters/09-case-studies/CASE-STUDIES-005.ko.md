---
knowledge_id: CASE-STUDIES-005
title: FTX Collapse
subtitle: custody 실패, commingling, 그리고 trusted intermediation의 붕괴
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
  FTX 붕괴는 centralized-exchange counterparty risk에 대해 무엇을 드러냈으며,
  왜 연구자는 crypto-market volatility와 offchain intermediary 내부의
  custody 및 governance failure를 분리해야 하는가?
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

2022년 11월, FTX는 중앙화된 중개기관으로서 실패했고, 규제기관은 나중에 고객 자산의 대규모 오용과 commingling을 주장했다.

## 2. Key Dates

- 2022년 11월 11일: 이후 CFTC 문서에서 설명된 행위 기간은 2022년 11월 11일까지 이어졌다.[^ref-cftc-ftx]
- 2022년 12월 13일: SEC는 Sam Bankman-Fried를 FTX의 equity investor를 상대로 한 사기 혐의로 기소했고, 고객 자금이 Alameda Research로 전용된 사실을 은폐했다고 주장했다.[^ref-sec-ftx]
- 2022년 12월 13일: DOJ는 사기, 자금세탁, 선거자금 관련 혐의를 발표했다.[^ref-doj-ftx]

## 3. Event Structure

이 사건은 본질적으로 "crypto protocol" 실패가 아니었다. 그것은 다음을 포함하는 centralized-intermediary failure였다.

- custody misrepresentation,
- internal controls failure,
- asset commingling,
- governance breakdown.

## 4. Why It Mattered

FTX는 암호화폐 시장에서 가장 중요한 구분 중 하나를 무너뜨렸다.

- protocol risk,
- versus intermediary risk.

이 붕괴는 crypto-native asset도 매우 전통적인 실패 양식 안에 보관될 수 있음을 시장에 상기시켰다.

## 5. Lessons

- self-custody와 onchain transparency는 마케팅 문구가 아니다. 그것은 실패 surface를 바꾼다.
- 겉보기에 정교한 기업도 기본적인 segregation discipline에서 실패할 수 있다.
- centralized venue에서는 token 또는 protocol 분석보다 counterparty risk가 더 지배적일 수 있다.

## 6. Institutional Thinking

- FTX는 blockchain이 실패했다는 증거로 읽어서는 안 된다. 이는 opaque intermediary가 여전히 핵심 crypto risk class임을 보여주는 증거다.
- 연구자는 exchange solvency와 governance를 market-direction call과 구분해야 한다.

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
