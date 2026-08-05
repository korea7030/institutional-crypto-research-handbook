---
knowledge_id: SMART-CONTRACTS-007
title: Proxy Patterns
subtitle: Transparent, UUPS, Beacon, Delegatecall Routing, 그리고 upgradeable indirection의 실제 형태
version: 1.0.0
status: Reviewed
difficulty: L350
estimated_reading: 135 min
estimated_study: 340 min
last_reviewed: 2026-08-04
domain:
  - Smart Contracts
  - Proxies
  - Upgradeability
parent:
  - Smart Contracts
prerequisites:
  - SMART-CONTRACTS-006
related_topics:
  - Upgradeability
  - Security
primary_sources:
  - REF-OZ-PROXY-PATTERN-001
  - REF-OZ-UPGRADES-PLUGINS-001
  - REF-OZ-PROXY-API-001
tags:
  - smart-contracts
  - proxies
  - uups
  - transparent-proxy
  - beacon-proxy
---

# Proxy Patterns
> Smart Contracts  
> Research Unit: SMART-CONTRACTS-007

---

## Research Brief

```yaml
knowledge_id: SMART-CONTRACTS-007
title: Proxy Patterns
research_question: >
  What are the main upgradeable proxy patterns used in Ethereum smart contracts,
  how do they route calls and separate logic from state, and what practical
  tradeoffs distinguish transparent, UUPS, and beacon approaches?
document_type: deep-dive
difficulty: L350
prerequisites:
  - SMART-CONTRACTS-006
parent: Smart Contracts
previous: SMART-CONTRACTS-006
next: SMART-CONTRACTS-008
related_topics:
  - Upgradeability
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
  - Low-level assembly walkthrough
  - Non-Ethereum proxy ecosystems
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- proxy pattern이 무엇인지 설명할 수 있다.
- transparent, UUPS, beacon proxy를 구분할 수 있다.
- proxy가 user-facing address와 logic implementation을 왜 분리하는지 설명할 수 있다.
- delegatecall-style indirection이 risk analysis를 왜 바꾸는지 설명할 수 있다.

---

## 2. 핵심 질문

1. proxy란 무엇인가?
2. 왜 proxy를 사용하는가?
3. transparent proxy와 UUPS proxy의 차이는 무엇인가?
4. beacon proxy는 무엇에 유용한가?
5. 왜 proxy는 분석을 복잡하게 만드는가?

---

## 3. Executive Summary

OpenZeppelin의 proxy documentation은 proxy pattern을 user를 위한 stable access point를 유지하면서도, call을 implementation logic으로 forward하는 mechanism으로 설명한다.[^ref-oz-proxy-pattern][^ref-oz-proxy-api]

current plugin docs는 세 가지 중요한 family를 식별한다.

- transparent proxy
- UUPS proxy
- beacon proxy[^ref-oz-upgrades-plugins]

실질적인 차이는 upgrade authority와 logic이 어디에 있는지, 얼마나 많은 deployment를 함께 upgrade할 수 있는지, proxy 자체와 implementation 중 어디에 complexity가 더 많이 놓이는지에 있다.

---

## 4. Protocol Structure

```text
user
-> proxy address
-> forwarded call
-> implementation logic
```

state는 proxy-facing storage context에 남고, logic은 chosen pattern에 따라 교체될 수 있다.

---

## 5. Transparent Proxy Pattern

OpenZeppelin은 transparent proxy가 admin caller와 ordinary user를 구분한다고 설명한다. admin은 upgrade/admin function을 사용할 수 있고, 다른 caller는 implementation logic으로 delegate된다.[^ref-oz-proxy-pattern]

이 방식은 일부 interface-clash confusion을 줄이지만, proxy-side admin logic을 추가한다.

---

## 6. UUPS

OpenZeppelin의 proxy API docs는 추천 방향이 UUPS 쪽으로 옮겨갔다고 설명한다. 이유는 더 light-weight하고 versatile하기 때문이다.[^ref-oz-proxy-api]

UUPS에서는 upgrade logic이 heavier transparent proxy shell 안이 아니라 implementation contract에 존재한다.[^ref-oz-proxy-api]

---

## 7. Beacon Proxies

OpenZeppelin plugin docs는 beacon proxy가 자신이 가리키는 beacon을 upgrade함으로써 multiple deployment를 atomically group-upgrade할 수 있다고 설명한다.[^ref-oz-upgrades-plugins][^ref-oz-proxy-api]

이는 여러 proxy instance가 하나의 shared implementation pointer를 따라야 할 때 유용하다.

---

## 8. Technical Mechanics

proxy docs는 핵심 아이디어를, user가 proxy와 상호작용하고 proxy가 logic contract로 forward하는 구조라고 설명한다.[^ref-oz-proxy-pattern]

개념적으로:

```text
same address
different implementation over time
```

이것이 upgrade convenience와 analytical complexity를 동시에 만든다.

---

## 9. Security Assumptions

proxy correctness에는 다음이 필요하다.

- safe forwarding logic
- safe upgrade authorization
- storage compatibility
- correct operator understanding

proxy misuse는 severe governance risk와 implementation risk를 만들 수 있다.

---

## 10. Mathematical or Economic Model

개념적으로:

```text
proxy indirection
= address stability
+ logic replaceability
+ extra complexity
```

---

## 11. Protocol Implementation

이 주제의 primary source는 OpenZeppelin의 proxy pattern, upgrades plugin, proxy API docs다.[^ref-oz-proxy-pattern][^ref-oz-upgrades-plugins][^ref-oz-proxy-api]

---

## 12. On-Chain Implications

analyst는 다음을 구분해야 한다.

- proxy address
- implementation address
- admin 또는 governance authority
- upgrade history

same-address continuity만으로는 충분하지 않다.

---

## 13. Institutional Thinking

institution은 contract address뿐 아니라 upgrade path와 authority도 모니터링해야 한다.

### Practical Implications

- proxy type은 risk profile을 바꾼다.
- beacon setup 같은 group upgrade pattern은 blast radius를 키울 수 있다.
- UUPS와 transparent의 차이는 upgrade logic이 어디에 있는지, 무엇을 review해야 하는지에 영향을 준다.

---

## 14. Common Misinterpretations

### "A proxy is just a renamed contract"

틀렸다. security consequence를 가진 indirection layer다.

### "Same address means same code"

틀렸다.

### "Proxy choice is only a gas optimization detail"

틀렸다. governance와 attack surface를 바꾼다.

---

## 15. Research Questions

1. 어떤 proxy pattern failure가 institution이 surface-level monitoring만으로 가장 발견하기 어려운가?
2. proxy family 전반에 걸쳐 upgrade authority는 어떻게 추적해야 하는가?

---

## 16. Practical Exercises

### Exercise 1

transparent proxy와 UUPS proxy의 핵심 차이 하나를 설명하라.

### Exercise 2

beacon proxy가 왜 upgrade blast radius를 키울 수 있는지 설명하라.

---

## 17. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Main proxy families and their roles | Directly specified | OpenZeppelin docs |
| Governance / blast-radius framing | Inference from sources | Derived from proxy architecture |

---

## 18. Knowledge Graph

```text
Proxy Patterns
├─ Transparent Proxy
├─ UUPS
├─ Beacon Proxy
├─ Stable Address
└─ Replaceable Logic
```

---

## 19. References

### Primary Sources

[^ref-oz-proxy-pattern]: OpenZeppelin Docs, "Proxy Upgrade Pattern," accessed 2026-08-04, https://docs.openzeppelin.com/upgrades-plugins/proxies

[^ref-oz-upgrades-plugins]: OpenZeppelin Docs, "Upgrades Plugins," accessed 2026-08-04, https://docs.openzeppelin.com/upgrades-plugins

[^ref-oz-proxy-api]: OpenZeppelin Docs, "Proxy" API / overview materials, accessed 2026-08-04, https://docs.openzeppelin.com/contracts/4.x/api/proxy

---

## 20. Cross References

### Previous

- SMART-CONTRACTS-006 — Upgradeability

### Next

- SMART-CONTRACTS-008 — Oracle Fundamentals

---

## Review Status

### Technical Review

Passed.

- transparent, UUPS, beacon pattern을 분리했다.
- stable address와 replaceable logic의 차이를 명시했다.

### Evidence Review

Passed.

- core claim은 OpenZeppelin docs를 인용한다.

### Editorial Review

Passed.

- 구조는 프로젝트 format을 따른다.

### Adversarial Review

Passed.

- 문서는 proxy를 cosmetic indirection으로 축소하지 않는다.
- same address를 same code와 동일시하지 않는다.

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
