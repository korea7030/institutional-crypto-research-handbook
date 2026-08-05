---
knowledge_id: CASE-STUDIES-004
title: Terra Collapse
subtitle: 알고리즘형 stablecoin 실패, 반사적 뱅크런 동학, 그리고 신뢰의 붕괴
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
  2022년 5월 Terra ecosystem에서는 무엇이 실패했으며, 연구자는 이 붕괴를
  단순한 one-token price crash가 아니라 reflexive stablecoin-confidence
  breakdown으로 어떻게 이해해야 하는가?
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

2022년 5월 UST는 달러 peg를 잃었고, LUNA는 반사적 붕괴에 들어가면서 Terra ecosystem의 핵심 confidence structure를 파괴했다.

## 2. Key Dates

- 2022년 5월: SEC는 나중에 UST가 미 달러 peg에서 이탈했고 UST 및 관련 token이 거의 0에 가까운 수준까지 하락했다고 주장했다.[^ref-sec-terra]
- 2022년 5월 16일: 새로운 network launch를 조정하기 위한 Terraform의 "Terra Ecosystem Revival Plan 2"가 게시되었다.[^ref-terra-revive]

## 3. Event Structure

이 시스템은 redemption logic과 endogenous demand에 대한 confidence에 의존하고 있었다. confidence가 깨지자 stabilization mechanism은 스트레스를 흡수하는 대신 증폭했다.

이는 전형적인 reflexive failure였다.

- peg 약화가 trust를 훼손했고,
- trust 상실이 exit를 유발했으며,
- exit는 메커니즘의 안정화 능력을 더 약화시켰다.

## 4. Why It Mattered

Terra는 algorithmic 또는 reflexive stablecoin 설계가 왜 불연속적으로 실패할 수 있는지를 보여주는 사례였다. 동시에 yield narrative와 ecosystem growth가 monetary core의 취약성을 가릴 수 있다는 점도 보여주었다.

## 5. Lessons

- stablecoin credibility는 token label이 아니라 system property다.
- reflexive stabilization은 confidence가 갑자기 올바른 방향으로 더 이상 복리되지 않을 때까지는 작동할 수 있다.
- ecosystem TVL과 user growth는 monetary resilience의 대체재가 아니다.

## 6. Institutional Thinking

- Terra는 단순한 "bad tokenomics" 사례가 아니라 confidence-run 사례로 분석해야 한다.
- 결정적 변수는 메커니즘 설계 자체만이 아니라, 그 메커니즘이 coordinated exit behavior를 견딜 수 있었는가였다.

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
