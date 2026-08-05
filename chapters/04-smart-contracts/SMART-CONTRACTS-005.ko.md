---
knowledge_id: SMART-CONTRACTS-005
title: Security Considerations
subtitle: Public Adversary, Reentrancy, Checks-Effects-Interactions, Testing Limit, 그리고 smart contract security가 하나의 discipline인 이유
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 145 min
estimated_study: 380 min
last_reviewed: 2026-08-04
domain:
  - Smart Contracts
  - Security
  - Ethereum
parent:
  - Smart Contracts
prerequisites:
  - SMART-CONTRACTS-001
  - SMART-CONTRACTS-002
  - SMART-CONTRACTS-003
related_topics:
  - Upgradeability
  - Proxy Patterns
primary_sources:
  - REF-SOLIDITY-SECURITY-001
  - REF-ETH-DOC-SC-TESTING-2026-001
tags:
  - smart-contracts
  - security
  - reentrancy
  - testing
---

# Security Considerations
> Smart Contracts  
> Research Unit: SMART-CONTRACTS-005

---

## Research Brief

```yaml
knowledge_id: SMART-CONTRACTS-005
title: Security Considerations
research_question: >
  What makes smart contract security unusually difficult, which canonical
  pitfalls and design patterns matter most, and how should researchers separate
  protocol-level guarantees from application-level safety assumptions?
document_type: deep-dive
difficulty: L400
prerequisites:
  - SMART-CONTRACTS-001
  - SMART-CONTRACTS-002
  - SMART-CONTRACTS-003
parent: Smart Contracts
previous: SMART-CONTRACTS-004
next: SMART-CONTRACTS-006
related_topics:
  - Upgradeability
  - Proxy Patterns
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
  - Full exploit catalog
  - Formal verification manual
  - Bug bounty process survey
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- 왜 smart contract security가 어려운지 설명할 수 있다.
- reentrancy 같은 canonical pitfall을 식별할 수 있다.
- checks-effects-interactions pattern을 설명할 수 있다.
- 왜 testing만으로는 충분하지 않은지 설명할 수 있다.
- protocol guarantee와 application safety를 구분할 수 있다.

---

## 2. 핵심 질문

1. 왜 smart contract security는 ordinary app security와 다른가?
2. 가장 중요한 canonical pitfall은 무엇인가?
3. checks-effects-interactions는 무엇을 의미하는가?
4. 왜 testing은 필요하지만 충분하지 않은가?

---

## 3. Executive Summary

Solidity security documentation은 smart contract security가 어려운 이유로, contract가 valuable asset을 다루고, publicly execute되며, malicious actor에게 계속 노출된다는 점을 강조한다.[^ref-solidity-security]

같은 documentation은 reentrancy 같은 canonical pitfall을 강조하고, checks-effects-interactions 같은 pattern을 권고한다.[^ref-solidity-security]

Ethereum의 smart-contract testing docs는 immutable public blockchain에서 mistake cost가 매우 크며, mainnet 전 testing은 luxury가 아니라 minimum requirement라고 다시 강조한다.[^ref-eth-sc-testing]

올바른 security posture는 layered하다.

- sound design
- safe coding pattern
- testing
- review
- operational governance
- external integration에 대한 cautious assumption

---

## 4. Protocol Structure

```text
security posture
= code safety
+ state safety
+ call-flow safety
+ operational safety
+ governance / upgrade safety
```

---

## 5. Canonical Pitfalls

### Reentrancy

Solidity security guidance는 reentrancy를 canonical pitfall로 든다. external call이 original function이 자신의 state transition을 안전하게 마무리하기 전에 다른 contract에 control을 넘겨줄 수 있다는 것이다.[^ref-solidity-security]

### Public Visibility and Value

contract는 기본적으로 public하고 adversarial하다. attacker는 code를 inspect하고 interaction을 simulate하며 edge case를 exploit할 수 있다.

### External Calls

다른 contract와의 모든 interaction은 security boundary가 될 수 있다.

---

## 6. Checks-Effects-Interactions

Solidity security guidance는 checks-effects-interactions pattern을 방어적 접근으로 제시한다.[^ref-solidity-security]

핵심 아이디어는 다음과 같다.

```text
checks
-> effects
-> external interactions
```

이는 certain reentrancy-style failure에 대한 노출을 줄인다.

---

## 7. Testing Limits

testing은 필수지만, 모든 bug의 부재를 증명하지는 못한다. official Ethereum docs는 서로 다른 방법이 서로 다른 failure class를 잡기 때문에 layered testing approach를 권한다.[^ref-eth-sc-testing]

---

## 8. Security Assumptions

smart contract safety는 protocol correctness 이상에 의존한다. application logic, deployment assumption, external integration, governance control에도 의존한다.

---

## 9. Mathematical or Economic Model

개념적으로:

```text
exploitability
= reachable bug
+ economic incentive
+ executable path
```

이것은 analytical framing이지 protocol formula가 아니다.

---

## 10. Protocol Implementation

이 주제의 primary source는 Solidity security considerations와 Ethereum smart contract testing docs다.[^ref-solidity-security][^ref-eth-sc-testing]

---

## 11. On-Chain Implications

많은 exploit은 malicious execution path가 실제로 사용된 뒤에야 visible해진다. onchain evidence는 종종 preventative가 아니라 retrospective다.

---

## 12. Institutional Thinking

institution은 smart contract security를 one-time code review가 아니라 continuous risk management로 다뤄야 한다.

### Practical Implications

- testing은 필요하지만 충분하지 않다.
- external integration은 attack surface를 넓힌다.
- upgrade path와 governance control은 risk를 줄이기도 하고 늘리기도 한다.

---

## 13. Common Misinterpretations

### "If it compiles and tests pass, it is safe"

틀렸다.

### "Protocol security means application security"

틀렸다.

### "Reentrancy is the only serious risk"

틀렸다. canonical할 뿐 exhaustive하지 않다.

---

## 14. Research Questions

1. 어떤 failure class가 institutional due diligence에서 체계적으로 과소평가되고 있는가?
2. operational control과 code quality는 어떤 비중으로 평가해야 하는가?

---

## 15. Practical Exercises

### Exercise 1

checks-effects-interactions를 자신의 말로 설명하라.

### Exercise 2

immutable deployment가 왜 security bar를 높이는지 설명하라.

---

## 16. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Security pitfalls and CEI pattern | Directly specified | Solidity security docs |
| Testing minimum requirement | Directly specified | Ethereum testing docs |
| Ongoing risk-management framing | Inference from sources | Derived from immutability and adversarial execution |

---

## 17. Knowledge Graph

```text
Security Considerations
├─ Reentrancy
├─ External Calls
├─ Checks-Effects-Interactions
├─ Testing Limits
└─ Operational Governance
```

---

## 18. References

### Primary Sources

[^ref-solidity-security]: Solidity documentation, "Security Considerations," accessed 2026-08-04, https://docs.soliditylang.org/en/latest/security-considerations.html

[^ref-eth-sc-testing]: ethereum.org, "Testing smart contracts," accessed 2026-08-04, https://ethereum.org/developers/docs/smart-contracts/testing/

---

## 19. Cross References

### Previous

- SMART-CONTRACTS-004 — Events

### Next

- SMART-CONTRACTS-006 — Upgradeability

---

## Review Status

### Technical Review

Passed.

- canonical pitfall과 CEI pattern을 포함했다.
- testing을 minimum requirement로 다뤘고, complete solution로 과장하지 않았다.

### Evidence Review

Passed.

- core claim은 Solidity security docs와 Ethereum testing docs를 인용한다.

### Editorial Review

Passed.

- 구조는 프로젝트 format을 따른다.

### Adversarial Review

Passed.

- 문서는 testing을 과장하지 않고, 모든 risk를 하나의 bug class로 환원하지 않는다.

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
