---
knowledge_id: DEFI-009
title: Restaking
subtitle: 스테이킹된 보안의 재사용, AVS 리스크 확장, 그리고 Ethereum 위의 새로운 shared-security layer
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
  Restaking이란 무엇이며, 이미 stake된 자본을 추가 서비스에 어떻게 재사용하고,
  왜 shared-security 설계는 자본 효율성과 상관 리스크 확대를 동시에 만들 수
  있는가?
document_type: foundation
difficulty: L400
prerequisites:
  - DEFI-008
parent: DeFi
previous: DEFI-008
next: DEFI-010
```

## 1. Learning Objectives

- restaking을 정확히 정의할 수 있다.
- Ethereum base consensus 바깥으로 보안이 어떻게 확장되는지 설명할 수 있다.
- native restaking과 LST 기반 restaking을 구분할 수 있다.
- correlated slashing과 shared-security risk를 식별할 수 있다.

## 2. Executive Summary

EigenLayer의 공식 포럼 화이트페이퍼 공지문은 restaking을 base Ethereum validation을 넘어서는 추가 slashing condition과 operator participation을 중심으로 구현 중인 핵심 프로토콜 아이디어로 설명한다.[^ref-eigen-ann]

EigenLayer의 공식 자료는 restaking을 Ethereum에 stake된 자본 및 관련 자산이 추가 서비스 보안까지 담당하게 만드는 방식으로 제시한다.[^ref-eigen-res]

핵심적인 개념 변화는 동일한 경제적 stake가 둘 이상의 security domain에서 재사용된다는 점이다.

## 3. Mechanism

restaking은 일반적으로, 이미 Ethereum staking 또는 staking derivative에 묶여 있는 자본이 추가 서비스까지 뒷받침하도록 slashing 또는 delegated security 조건을 opt-in 방식으로 확장하는 구조를 말한다.

## 4. Benefits

- shared security bootstrap,
- 더 빠른 network/service launch,
- 참여자에게 추가 yield 기회 제공.

## 5. Risks

- correlated slashing,
- operator concentration,
- 불투명한 AVS risk,
- LST와 restaking layer가 중첩되며 생기는 stacked dependency.

## 6. Institutional Thinking

- restaking은 공짜 추가 수익이 아니다. 그것은 downside가 복합될 수 있는 security reuse다.
- 핵심 질문은 단순히 얼마나 많은 stake가 서비스를 뒷받침하는가가 아니라, 어떤 failure domain과 어떤 slashing condition 아래 그 보안이 성립하는가다.

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
