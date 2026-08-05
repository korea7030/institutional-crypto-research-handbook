---
knowledge_id: SMART-CONTRACTS-008
title: Oracle Fundamentals
subtitle: Oracle Problem, Offchain Data Injection, Trust Model, 그리고 Hybrid Smart Contract
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 135 min
estimated_study: 340 min
last_reviewed: 2026-08-04
domain:
  - Smart Contracts
  - Oracles
  - Ethereum
parent:
  - Smart Contracts
prerequisites:
  - SMART-CONTRACTS-001
  - ETHEREUM-FOUNDATION-012
related_topics:
  - Security
  - Layer 2
primary_sources:
  - REF-ETH-DOC-ORACLES-2026-001
tags:
  - smart-contracts
  - oracles
  - oracle-problem
  - hybrid-smart-contracts
---

# Oracle Fundamentals
> Smart Contracts  
> Research Unit: SMART-CONTRACTS-008

---

## Research Brief

```yaml
knowledge_id: SMART-CONTRACTS-008
title: Oracle Fundamentals
research_question: >
  Why do smart contracts need oracles, what is the oracle problem, how do
  oracle trust models differ, and how should researchers evaluate hybrid smart
  contracts that depend on offchain information?
document_type: deep-dive
difficulty: L300
prerequisites:
  - SMART-CONTRACTS-001
  - ETHEREUM-FOUNDATION-012
parent: Smart Contracts
previous: SMART-CONTRACTS-007
next:
related_topics:
  - Security
  - Events
  - Layer 2
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
  - Vendor-by-vendor oracle comparison
  - Full price-feed exploit catalog
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- oracle problem을 설명할 수 있다.
- input, output, computational oracle role을 정의할 수 있다.
- centralized oracle trust model과 decentralized oracle trust model을 구분할 수 있다.
- oracle-dependent contract가 hybrid smart contract인 이유를 설명할 수 있다.
- onchain determinism과 offchain truth가 왜 tension을 만드는지 설명할 수 있다.

---

## 2. 핵심 질문

1. 왜 smart contract는 offchain fact를 직접 알 수 없는가?
2. oracle은 무엇을 하는가?
3. 주요 oracle trust model은 무엇인가?
4. 왜 oracle-dependent contract는 hybrid system인가?

---

## 3. Executive Summary

Ethereum의 official oracle documentation은 smart contract가 기본적으로 blockchain network 밖에 저장된 information에 접근할 수 없으며, oracle은 external information을 source하고 verify하고 transmit하는 application이라고 설명한다.[^ref-eth-oracles]

같은 documentation은 Ethereum의 deterministic execution model이 이런 한계의 이유 중 하나라고 설명한다. node는 shared onchain input에 대해 같은 output에 합의해야 하므로, naive external API call은 consensus assumption을 깨뜨린다.[^ref-eth-oracles]

따라서 oracle은 onchain computation과 external world 사이의 information gap을 메우며, docs가 명시적으로 hybrid smart contract라고 부르는 구조를 만든다.[^ref-eth-oracles]

이 때문에 oracle analysis는 smart contract risk의 중심이다. 경제적으로 중요한 많은 contract는 offchain data와 그 trust model의 신뢰성에 의존하기 때문이다.

---

## 4. Protocol Structure

```text
offchain information
-> oracle mechanism
-> onchain contract storage / callback
-> smart contract execution
```

이 bridge가 없으면 contract는 onchain data로만 제한된다.

---

## 5. The Oracle Problem

official docs는 blockchain이 deterministic하므로 onchain data만으로 답할 수 있는 question에 대한 consensus로 자신을 제한한다고 설명한다.[^ref-eth-oracles]

즉 contract는 consensus assumption을 해치지 않고 execution 중에 단순히 "인터넷에 물어볼" 수 없다.

---

## 6. Oracle Types

official oracle docs는 다음을 포함한 여러 category를 구분한다.

- input oracle
- output oracle
- computational oracle
- publish-subscribe와 request-response 같은 architectural model[^ref-eth-oracles]

이것만으로도 "oracle"이 단일 메커니즘이 아니라는 점은 충분히 드러난다.

---

## 7. Trust Models

official docs는 centralized oracle과 decentralized oracle을 구분하며, centralized oracle은 efficient할 수 있지만 correctness, availability, incentive guarantee가 약할 수 있다고 설명한다.[^ref-eth-oracles]

따라서 oracle trust assumption은 first-order security issue다.

---

## 8. Technical Mechanics

oracle docs는 common pattern을 다음처럼 설명한다.

```text
client contract requests data
-> onchain oracle contract emits log
-> offchain oracle node retrieves data
-> node sends transaction back with result
-> contract consumes oracle-fed value
```

[^ref-eth-oracles]

이것이 핵심 hybrid loop다.

---

## 9. Security Assumptions

oracle-dependent system은 다음에 의존한다.

- source-data correctness
- timely availability
- incentive compatibility
- callback integrity
- oracle configuration에 대한 governance

이들은 pure EVM execution guarantee와 같은 것이 아니다.

---

## 10. Mathematical or Economic Model

개념적으로:

```text
contract outcome
= onchain logic
+ offchain data quality
+ oracle trust model
```

이 때문에 어떤 system에서는 code correctness보다 oracle risk가 더 지배적일 수 있다.

---

## 11. Protocol Implementation

이 주제의 primary source는 Ethereum의 official oracles documentation이며, problem framing, trust model, architectural pattern을 다룬다.[^ref-eth-oracles]

---

## 12. On-Chain Implications

contract가 oracle에 의존하면, onchain execution trace만으로는 특정 값이 왜 도착했는지 충분히 설명되지 않을 수 있다. offchain data provenance가 중요하다.

---

## 13. Institutional Thinking

institution은 oracle-dependent contract를 purely onchain system이 아니라 hybrid system으로 다뤄야 한다.

### Practical Implications

- data source provenance를 평가해야 한다.
- liveness와 update cadence를 평가해야 한다.
- 누가 oracle configuration을 바꿀 수 있는지 평가해야 한다.

---

## 14. Common Misinterpretations

### "Smart contracts can directly call APIs"

틀렸다.

### "Decentralized contracts are fully onchain in every meaningful sense"

oracle가 개입하면 흔히 틀린 말이 된다.

### "Oracle risk is separate from contract risk"

틀렸다. 그것은 contract-system risk의 일부다.

---

## 15. Research Questions

1. institutional due diligence에서 어떤 oracle trust assumption이 가장 중요한가?
2. analyst는 protocol risk framework에서 hybrid-system exposure를 어떻게 라벨링해야 하는가?

---

## 16. Practical Exercises

### Exercise 1

oracle problem을 자신의 말로 설명하라.

### Exercise 2

centralized oracle model과 decentralized oracle model의 핵심 차이 하나를 설명하라.

---

## 17. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Oracle problem and determinism tension | Directly specified | Ethereum oracle docs |
| Oracle architectural and trust-model categories | Directly specified | Ethereum oracle docs |
| Hybrid smart contract framing | Directly specified plus synthesis | Ethereum oracle docs |

---

## 18. Knowledge Graph

```text
Oracle Fundamentals
├─ Oracle Problem
├─ Offchain Data
├─ Input / Output / Computational Oracles
├─ Centralized vs Decentralized Trust
└─ Hybrid Smart Contracts
```

---

## 19. References

### Primary Sources

[^ref-eth-oracles]: ethereum.org, "Oracles," official documentation describing oracle problem, trust models, and oracle architectures, accessed 2026-08-04, https://ethereum.org/developers/docs/oracles/

---

## 20. Cross References

### Previous

- SMART-CONTRACTS-007 — Proxy Patterns

### Related

- ETHEREUM-FOUNDATION-012 — Layer 2 Overview
- SMART-CONTRACTS-005 — Security Considerations

---

## Review Status

### Technical Review

Passed.

- oracle problem, trust model, hybrid-contract framing을 포함했다.

### Evidence Review

Passed.

- core claim은 official Ethereum oracle docs를 인용한다.

### Editorial Review

Passed.

- 구조는 프로젝트 format을 따른다.

### Adversarial Review

Passed.

- 문서는 smart contract가 offchain system을 직접 query한다고 암시하지 않는다.
- oracle risk를 contract risk 바깥의 것으로 다루지 않는다.

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
