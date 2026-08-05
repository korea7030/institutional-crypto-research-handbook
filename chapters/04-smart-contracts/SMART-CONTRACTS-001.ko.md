---
knowledge_id: SMART-CONTRACTS-001
title: Smart Contract Fundamentals
subtitle: 주소 위의 프로그램, deterministic enforcement, 그리고 Ethereum application의 핵심 execution model
version: 1.0.0
status: Reviewed
difficulty: L200
estimated_reading: 110 min
estimated_study: 260 min
last_reviewed: 2026-08-04
domain:
  - Smart Contracts
  - Ethereum
  - Applications
parent:
  - Smart Contracts
prerequisites:
  - ETHEREUM-FOUNDATION-001
  - ETHEREUM-FOUNDATION-005
related_topics:
  - Contract Lifecycle
  - ABI
  - Events
primary_sources:
  - REF-ETH-DOC-SC-INTRO-2026-001
  - REF-ETH-DOC-SC-ANATOMY-2026-001
  - REF-SOLIDITY-INTRO-001
tags:
  - smart-contracts
  - ethereum
  - fundamentals
---

# Smart Contract Fundamentals
> Smart Contracts  
> Research Unit: SMART-CONTRACTS-001

---

## Research Brief

```yaml
knowledge_id: SMART-CONTRACTS-001
title: Smart Contract Fundamentals
research_question: >
  What is a smart contract on Ethereum, what properties make it different from
  conventional backend software, and how should researchers frame smart
  contracts as deterministic onchain programs rather than as legal or marketing
  abstractions?
document_type: foundation
difficulty: L200
prerequisites:
  - ETHEREUM-FOUNDATION-001
  - ETHEREUM-FOUNDATION-005
parent: Smart Contracts
previous:
next: SMART-CONTRACTS-002
related_topics:
  - ABI
  - Contract Lifecycle
  - Events
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
  - Full Solidity tutorial
  - Legal contract enforceability by jurisdiction
  - Token-standard specifics
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Ethereum 맥락에서 smart contract를 정확히 정의할 수 있다.
- smart contract가 account이면서 program이라는 점을 설명할 수 있다.
- smart contract와 ordinary web backend를 구분할 수 있다.
- deterministic execution이 왜 중요한지 설명할 수 있다.
- "smart contract"를 legal agreement text와 혼동하면 안 되는 이유를 설명할 수 있다.

---

## 2. 핵심 질문

1. smart contract란 무엇인가?
2. smart contract는 어디에 존재하는가?
3. 사용자는 그것과 어떻게 상호작용하는가?
4. smart contract는 ordinary application code와 무엇이 다른가?
5. 왜 smart contract는 위험한가?

---

## 3. Executive Summary

official Ethereum documentation은 smart contract를 Ethereum blockchain 위에서 실행되는 program으로 정의하며, code와 data가 특정 address에 존재한다고 설명한다.[^ref-eth-sc-intro]

smart contract는 Ethereum account의 한 종류다. 따라서 balance를 가질 수 있고 transaction의 target이 될 수 있지만, user가 직접 제어하지는 않는다. 대신 valid interaction이 trigger될 때 programmed된 대로 실행된다.[^ref-eth-sc-intro]

Solidity documentation은 smart contract를 Ethereum state 안에서 account의 behavior를 govern하는 program으로 설명한다.[^ref-solidity-intro]

실무적으로 smart contract는 다음처럼 이해하는 것이 가장 정확하다.

- onchain executable logic
- persistent state에 결합된 program
- transaction이나 call로 invoke됨
- deterministic execution과 gas metering의 제약을 받음

이 설명이 popular slogan인 "code is law"나, legal contract에 대한 느슨한 비유보다 더 정확하다.

---

## 4. Protocol Structure

### What a Smart Contract Is

최소한 다음과 같이 볼 수 있다.

```text
smart contract
= contract account
+ deployed bytecode
+ persistent state
+ callable functions / interfaces
```

### Core Difference from a Wallet Account

user account는 transaction에 sign한다.

smart contract는 invoke될 때 code를 실행한다.

### Why This Matters

이 때문에 smart contract는 Ethereum의 application-layer primitive가 된다.

---

## 5. Historical Context

### Ethereum's Design Goal

Ethereum의 original design은 개발자가 executable logic을 shared network state 안에 직접 게시함으로써 decentralized application을 만들 수 있게 하려는 것이었다.[^ref-solidity-intro]

### Modern Framing

current Ethereum docs는 smart contract를, 누구나 transaction request를 통해 execution을 요청할 수 있는 reusable code snippet으로 설명한다.[^ref-eth-sc-intro]

### Consequence

smart contract 개념은 Ethereum에 나중에 덧붙은 기능이 아니라, platform identity의 중심이다.

---

## 6. Technical Foundations

### Program + State

anatomy documentation은 smart contract가 transaction을 받으면 실행되는 data와 function으로 구성된다고 설명한다.[^ref-eth-sc-anatomy]

### Addressability

한 번 deploy되면 contract는 특정 address에 존재하고, 다른 actor는 그 address를 통해 상호작용한다.[^ref-eth-sc-intro]

### Deterministic Runtime

execution은 EVM을 통해 이뤄지므로, smart contract logic은 node 간에 deterministic해야 한다.

---

## 7. Security Assumptions

### Immutability by Default

smart contracts intro는 smart contract가 기본적으로 delete될 수 없고, 그와의 interaction은 irreversible하다고 설명한다.[^ref-eth-sc-intro]

### Public Adversarial Environment

Solidity security documentation은 contract가 종종 valuable asset을 통제하고, publicly execute되며, malicious actor에 노출된다고 강조한다.[^ref-solidity-security]

### Why This Matters

smart contract programming은 private web service를 작성하는 것보다 public financial infrastructure를 작성하는 것에 가깝다.

---

## 8. Mathematical or Economic Model

### Minimal Conceptual Model

```text
contract interaction
= call request
+ gas budget
+ deterministic execution
+ state update if valid
```

### Persistence Cost

contract state는 onchain에 persist하므로, data 저장과 업데이트에는 local server cost가 아니라 network-wide cost가 수반된다.

---

## 9. Protocol Implementation

### Primary Sources

이 주제의 현재 기초를 가장 잘 설명하는 자료는 다음이다.

- Ethereum smart contract introduction docs
- smart contract anatomy docs
- Solidity introductory docs 및 security guidance[^ref-eth-sc-intro][^ref-eth-sc-anatomy][^ref-solidity-intro][^ref-solidity-security]

### Why These Matter

이 source 조합은 definition, structure, execution, security posture를 함께 제공한다.

---

## 10. On-Chain Implications

### What Analysts Can See

analyst는 다음을 관찰할 수 있다.

- deployment transaction
- contract address
- write interaction
- emitted log
- later migration 또는 replacement pattern

### Invisible Parts

직접 보기 어려운 것은 다음이다.

- offchain testing quality
- governance intent
- internal review quality
- operator runbook

---

## 11. Institutional Thinking

institution은 smart contract를 marketing abstraction이 아니라 programmable public infrastructure로 다뤄야 한다.

### Practical Implications

- contract exposure는 code exposure다.
- deterministic execution은 predictability를 주지만, bug가 public하게 exploit될 수도 있게 만든다.
- smart contract risk는 legal wording보다 execution logic과 state transition에 더 크게 좌우된다.

### Better Research Posture

contract claim을 하기 전에 다음을 물어야 한다.

- 이것이 code behavior에 대한 주장인가?
- state를 바꾸는가, 아니면 단순 signal만 emit하는가?
- 어떤 external dependency가 있는가?

---

## 12. Common Misinterpretations

### "Smart contracts are legal contracts"

너무 넓다. rule과 agreement를 encode할 수는 있지만, 근본적으로는 onchain program이다.

### "Smart contracts run themselves without transactions"

틀렸다. transaction이나 다른 call을 통한 triggering이 필요하다.[^ref-eth-sc-intro]

### "A contract address is just another wallet"

틀렸다. 그것은 runtime behavior를 가진 code-bearing account다.

---

## 13. Research Questions

1. 어떤 smart contract property가 institutional risk classification에 가장 중요한가?
2. 연구자는 legal analogy를 과장하지 않으면서 smart contract를 어떻게 설명해야 하는가?
3. 어떤 smart contract behavior class가 high-level transfer data만으로는 가장 잘 보이지 않는가?

---

## 14. Practical Exercises

### Exercise 1

"code is law"라는 표현을 쓰지 않고 smart contract를 한 단락으로 정의하라.

### Exercise 2

user account와 contract account의 차이를 설명하라.

### Exercise 3

왜 smart contract execution이 deterministic해야 하는지 설명하라.

---

## 15. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Smart contracts as code + state at an address | Directly specified | ethereum.org docs |
| Smart contracts as programs governing account behavior | Directly specified | Solidity docs |
| Institutional infrastructure framing | Inference from sources | Derived from public execution and irreversibility |

---

## 16. Knowledge Graph

```text
Smart Contract Fundamentals
├─ Contract Account
├─ Bytecode
├─ Persistent State
├─ Deterministic Execution
├─ Gas-Constrained Calls
└─ Public Adversarial Environment
```

---

## 17. References

### Primary Sources

[^ref-eth-sc-intro]: ethereum.org, "Introduction to smart contracts," official documentation defining smart contracts as programs at addresses on Ethereum, accessed 2026-08-04, https://ethereum.org/developers/docs/smart-contracts/

[^ref-eth-sc-anatomy]: ethereum.org, "Anatomy of smart contracts," official documentation describing contract data, functions, memory, storage, and events, accessed 2026-08-04, https://ethereum.org/developers/docs/smart-contracts/anatomy

[^ref-solidity-intro]: Solidity documentation, "Solidity" and introductory materials describing smart contracts as programs governing account behavior, accessed 2026-08-04, https://docs.soliditylang.org/en/latest/

[^ref-solidity-security]: Solidity documentation, "Security Considerations," accessed 2026-08-04, https://docs.soliditylang.org/en/latest/security-considerations.html

---

## 18. Cross References

### Next

- SMART-CONTRACTS-002 — Contract Lifecycle

### Related

- ETHEREUM-FOUNDATION-005 — EVM
- SMART-CONTRACTS-003 — ABI

---

## Review Status

### Technical Review

Passed.

- smart contract를 address 위의 program과 persistent state로 정의했다.
- contract account와 EOA를 구분했다.
- deterministic execution과 gas-constrained invocation을 포함했다.

### Evidence Review

Passed.

- core claim은 official Ethereum/ Solidity docs를 인용한다.
- security framing은 Solidity security guidance를 인용한다.

### Editorial Review

Passed.

- 구조는 프로젝트 format을 따른다.
- terminology는 일관된다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 smart contract를 legal agreement와 동일시하지 않는다.
- triggering 없는 autonomous execution을 암시하지 않는다.
- contract address를 ordinary wallet로 취급하지 않는다.

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
