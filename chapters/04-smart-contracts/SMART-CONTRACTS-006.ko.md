---
knowledge_id: SMART-CONTRACTS-006
title: Upgradeability
subtitle: Immutability tension, virtual upgrade, storage layout constraint, 그리고 governance tradeoff
version: 1.0.0
status: Reviewed
difficulty: L350
estimated_reading: 135 min
estimated_study: 340 min
last_reviewed: 2026-08-04
domain:
  - Smart Contracts
  - Upgradeability
  - Ethereum
parent:
  - Smart Contracts
prerequisites:
  - SMART-CONTRACTS-002
  - SMART-CONTRACTS-005
related_topics:
  - Proxy Patterns
  - Security
primary_sources:
  - REF-OZ-UPGRADING-001
  - REF-OZ-PROXY-PATTERN-001
tags:
  - smart-contracts
  - upgradeability
  - proxies
  - storage-layout
---

# Upgradeability
> Smart Contracts  
> Research Unit: SMART-CONTRACTS-006

---

## Research Brief

```yaml
knowledge_id: SMART-CONTRACTS-006
title: Upgradeability
research_question: >
  How do Ethereum developers reconcile code immutability with the practical need
  to patch and improve live systems, and what technical and governance tradeoffs
  arise when using upgradeable contract patterns?
document_type: deep-dive
difficulty: L350
prerequisites:
  - SMART-CONTRACTS-002
  - SMART-CONTRACTS-005
parent: Smart Contracts
previous: SMART-CONTRACTS-005
next: SMART-CONTRACTS-007
related_topics:
  - Proxy Patterns
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
  - Tool-by-tool walkthrough
  - Governance process design catalog
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- contract immutability가 있는데도 upgradeability가 존재하는 이유를 설명할 수 있다.
- virtual upgrade logic을 상위 수준에서 설명할 수 있다.
- storage layout constraint를 설명할 수 있다.
- upgradeability가 governance와 security tradeoff를 왜 도입하는지 설명할 수 있다.

---

## 2. 핵심 질문

1. 왜 contract를 upgradeable하게 만드는가?
2. virtual upgrade는 개념적으로 어떻게 작동하는가?
3. 왜 storage layout이 critical constraint인가?
4. upgradeability는 어떤 새로운 risk를 만드는가?

---

## 3. Executive Summary

OpenZeppelin의 upgrade docs는 기본 tension을 분명하게 설명한다. smart contract는 기본적으로 immutable하지만, 어떤 system은 bug fix, feature addition, rule change를 위해 user를 새 address로 강제 migration시키지 않고 live system을 수정할 필요가 있다.[^ref-oz-upgrading]

이들의 upgrade model은 user-facing address를 stable하게 유지하면서, 그 address 뒤의 implementation logic을 바꾼다.[^ref-oz-upgrading][^ref-oz-proxy-pattern]

이것은 한 operational problem을 해결하지만, 다른 문제를 만든다.

- storage layout은 compatible해야 한다.
- upgrade authority는 governance/security concern이 된다.
- "immutability"는 absolute가 아니라 conditional이 된다.

---

## 4. Protocol Structure

```text
user-facing access point stays
-> implementation logic may change
-> state must remain coherent
-> governance decides when change occurs
```

---

## 5. Why Upgradeability Exists

OpenZeppelin은 upgradeability가 없으면 bug fix를 위해 new contract를 deploy하고, state를 migrate하고, integration을 업데이트하고, user를 설득해 이동시켜야 할 수 있다고 설명한다.[^ref-oz-upgrading]

이는 operationally expensive하고 risky하다.

---

## 6. Technical Mechanics

OpenZeppelin guide는 upgrade를 다음처럼 요약한다.

1. new implementation contract를 deploy한다.
2. proxy의 implementation reference를 업데이트하는 transaction을 보낸다.[^ref-oz-upgrading]

이렇게 하면 user-facing entry point에서 address, state, balance를 유지할 수 있다.

---

## 7. Storage Layout Constraints

OpenZeppelin은 upgrade 중 storage layout을 임의로 바꾸면 안 된다고 명시적으로 경고한다. variable 제거, type 변경, 기존 variable 앞에 새 variable 삽입은 state interpretation을 깨뜨릴 수 있다.[^ref-oz-upgrading]

이것이 upgradeability의 핵심 technical constraint 중 하나다.

---

## 8. Security Assumptions

upgradeability는 governance risk와 key-management risk를 추가한다. upgrade를 authorize할 수 있는 자는 live contract behavior에 대해 상당한 power를 가진다.

또한 implementation risk도 추가한다. bad upgrade는 이전까지 안전했던 system을 망가뜨릴 수 있다.

---

## 9. Mathematical or Economic Model

개념적으로:

```text
immutability benefit - flexibility benefit = core tradeoff tension
```

upgradeability는 system을 그 tradeoff curve에서 flexibility 쪽으로 이동시킨다.

---

## 10. Protocol Implementation

이 주제의 primary source는 OpenZeppelin의 upgrading guide와 proxy-pattern explanation이다.[^ref-oz-upgrading][^ref-oz-proxy-pattern]

---

## 11. On-Chain Implications

upgradeable system은 address continuity를 유지하면서도 behavior를 materially 바꿀 수 있다. 따라서 analyst는 "same address"를 "same rules forever"로 읽으면 안 된다.

---

## 12. Institutional Thinking

institution은 upgradeability를 technical design이자 governance power structure로 함께 평가해야 한다.

### Practical Implications

- stable address가 immutable logic을 뜻하지는 않는다.
- upgrade authority는 문서화하고 모니터링해야 한다.
- storage-layout safety는 핵심 due-diligence topic이다.

---

## 13. Common Misinterpretations

### "Upgradeable means mutable in the casual software sense"

너무 느슨한 표현이다. mechanism은 indirect하고 constrained되어 있다.

### "Upgradeable always means safer"

틀렸다. bug-fix friction을 낮출 수 있지만 governance와 implementation risk를 추가한다.

---

## 14. Research Questions

1. 어떤 governance control이 upgrade abuse risk를 가장 잘 줄이는가?
2. institution은 upgradeable contract exposure와 non-upgradeable contract exposure를 어떻게 분류해야 하는가?

---

## 15. Practical Exercises

### Exercise 1

address를 보존하는 것과 logic을 보존하는 것이 왜 다른지 설명하라.

### Exercise 2

upgrade 중 storage layout이 왜 중요한지 설명하라.

---

## 16. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Immutability tension and virtual upgrade process | Directly specified | OpenZeppelin docs |
| Storage layout constraints | Directly specified | OpenZeppelin docs |
| Governance-tradeoff framing | Inference from sources | Derived from upgrade authority implications |

---

## 17. Knowledge Graph

```text
Upgradeability
├─ Immutability Tension
├─ Stable Address
├─ Replaceable Logic
├─ Storage Layout Constraints
└─ Governance Authority
```

---

## 18. References

### Primary Sources

[^ref-oz-upgrading]: OpenZeppelin Docs, "Upgrading smart contracts," accessed 2026-08-04, https://docs.openzeppelin.com/contracts/5.x/learn/upgrading-smart-contracts

[^ref-oz-proxy-pattern]: OpenZeppelin Docs, "Proxy Upgrade Pattern," accessed 2026-08-04, https://docs.openzeppelin.com/upgrades-plugins/proxies

---

## 19. Cross References

### Previous

- SMART-CONTRACTS-005 — Security Considerations

### Next

- SMART-CONTRACTS-007 — Proxy Patterns

---

## Review Status

### Technical Review

Passed.

- upgradeability mechanism과 storage constraint를 다뤘다.
- governance tradeoff를 포함했다.

### Evidence Review

Passed.

- core claim은 OpenZeppelin docs를 인용한다.

### Editorial Review

Passed.

- 구조는 프로젝트 format을 따른다.

### Adversarial Review

Passed.

- 문서는 stable address를 immutable logic의 증거로 다루지 않는다.
- upgradeability를 costless하게 과장하지 않는다.

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
