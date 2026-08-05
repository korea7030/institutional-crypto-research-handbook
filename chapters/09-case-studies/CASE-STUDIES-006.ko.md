---
knowledge_id: CASE-STUDIES-006
title: Ethereum Merge
subtitle: 합의 전환, execution continuity, 그리고 Ethereum 보안 서사의 재가격화
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 95 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - Case Studies
  - Ethereum
  - Protocol Upgrades
parent:
  - Case Studies
prerequisites:
  - ETHEREUM-FOUNDATION-007
  - DEFI-008
related_topics:
  - Staking
  - Layer 2 Expansion
primary_sources:
  - REF-EF-MERGE-2022-001
  - REF-EF-FINALIZED36-2022-001
tags:
  - case-study
  - merge
  - ethereum
---

# Ethereum Merge
> Case Studies  
> Research Unit: CASE-STUDIES-006

---

## Research Brief

```yaml
knowledge_id: CASE-STUDIES-006
title: Ethereum Merge
research_question: >
  왜 Merge는 역사적인 protocol transition이었으며, 연구자는 이를 application
  continuity를 보존하면서도 Ethereum의 security와 issuance narrative를 바꾼
  consensus change로 어떻게 평가해야 하는가?
document_type: case-study
difficulty: L300
prerequisites:
  - ETHEREUM-FOUNDATION-007
  - DEFI-008
parent: Case Studies
previous: CASE-STUDIES-005
next: CASE-STUDIES-007
```

## 1. Observation

Ethereum은 2022년 9월 application layer를 교체하지 않은 채 proof-of-work에서 proof-of-stake로 전환했다.

## 2. Key Dates

- 2022년 8월 12일: Ethereum Foundation의 "Finalized no. 36"은 merge sequence를 제시하고 2022년 9월 15일을 목표일로 잡았다.[^ref-ef-final36]
- 2022년 8월 24일: Ethereum Foundation은 Bellatrix를 2022년 9월 6일로 공지하고 mainnet merge 경로를 설명했다.[^ref-ef-merge]
- 2022년 9월 15일: Merge가 Ethereum mainnet에서 완료되었다.[^ref-ef-merge]

## 3. Event Structure

Merge는 합의 엔진을 바꾸면서도 다음을 보존했다.

- account,
- smart contract,
- balance,
- application state.

이는 settlement environment는 유지한 채 security process만 바꾼 드문 live-system migration이었다.

## 4. Why It Mattered

이 사건은 Ethereum을 분석하는 방식을 바꾸었다.

- miner economics에서 validator economics로,
- PoW issuance narrative에서 staking 및 burn narrative로,
- energy criticism에서 scaling-through-rollups narrative로.

## 5. Lessons

- client diversity와 testing discipline이 강하면 대규모 protocol transition도 점진적으로 실행할 수 있다.
- 사용자가 application-level discontinuity를 거의 느끼지 못하더라도 consensus change는 경제적으로 중요할 수 있다.
- Merge 이후 Ethereum의 정체성은 staking과 L2 scaling에 더 강하게 연결되었다.

## 6. Institutional Thinking

- Merge는 단지 기술적 업그레이드가 아니었다. 그것은 Ethereum의 investable framing을 바꾸었다.
- 연구자는 execution continuity와 monetary/security mechanic의 깊은 변화를 분리해서 봐야 한다.

## 7. References

[^ref-ef-final36]: Ethereum Foundation Blog, "Finalized no. 36," August 12, 2022, https://blog.ethereum.org/2022/08/12/finalized-no-36

[^ref-ef-merge]: Ethereum Foundation Blog, "Mainnet Merge Announcement," August 24, 2022, https://blog.ethereum.org/2022/08/24/mainnet-merge-announcement

## 8. Cross References

- Previous: CASE-STUDIES-005 — FTX Collapse
- Next: CASE-STUDIES-007 — Spot Bitcoin ETF Era

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
