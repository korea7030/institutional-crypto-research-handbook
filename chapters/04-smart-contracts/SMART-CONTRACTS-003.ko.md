---
knowledge_id: SMART-CONTRACTS-003
title: ABI
subtitle: Function Selector, Calldata Encoding, Schema-Dependent Decoding, 그리고 인간, 도구, bytecode 사이의 interface contract
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 125 min
estimated_study: 320 min
last_reviewed: 2026-08-04
domain:
  - Smart Contracts
  - ABI
  - Ethereum
parent:
  - Smart Contracts
prerequisites:
  - SMART-CONTRACTS-001
  - ETHEREUM-FOUNDATION-004
related_topics:
  - Events
  - Contract Interaction
primary_sources:
  - REF-SOLIDITY-ABI-001
  - REF-ETH-DOC-SC-INTERACT-2026-001
tags:
  - smart-contracts
  - abi
  - calldata
  - function-selector
---

# ABI
> Smart Contracts  
> Research Unit: SMART-CONTRACTS-003

---

## Research Brief

```yaml
knowledge_id: SMART-CONTRACTS-003
title: ABI
research_question: >
  What is the Ethereum Contract ABI, how do function selectors and type-based
  encoding work, and why must researchers treat ABI-based decoding as
  schema-dependent interpretation rather than self-describing truth?
document_type: deep-dive
difficulty: L300
prerequisites:
  - SMART-CONTRACTS-001
  - ETHEREUM-FOUNDATION-004
parent: Smart Contracts
previous: SMART-CONTRACTS-002
next: SMART-CONTRACTS-004
related_topics:
  - Contract Interaction
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
  - Full ABI coder implementation details
  - Full dynamic type encoding derivation
  - Language-specific SDK wrappers
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Ethereum 맥락에서 ABI를 정의할 수 있다.
- function selector와 calldata encoding을 설명할 수 있다.
- ABI decoding이 known schema를 왜 필요로 하는지 설명할 수 있다.
- ABI가 contract interaction tooling의 기반이라는 점을 설명할 수 있다.
- wrong ABI assumption이 wrong interpretation을 만든다는 점을 설명할 수 있다.

---

## 2. 핵심 질문

1. ABI란 무엇인가?
2. 왜 Ethereum은 ABI를 필요로 하는가?
3. function selector란 무엇인가?
4. 왜 ABI data는 self-describing하지 않은가?
5. ABI가 틀리면 무엇이 잘못되는가?

---

## 3. Executive Summary

Solidity ABI specification은 Ethereum ecosystem에서 contract와 상호작용하는 standard way를 정의한다. 이는 blockchain 외부에서의 interaction뿐 아니라 contract-to-contract interaction도 포함한다.[^ref-solidity-abi]

ABI는 schema-dependent하다. specification은 encoded data가 self-describing하지 않으며, decoding을 위해 schema가 필요하다고 명시한다.[^ref-solidity-abi]

calldata의 첫 4 byte는 function selector이며, 이것은 function signature의 Keccak-256 hash의 첫 4 byte로 정의된다.[^ref-solidity-abi]

따라서 ABI는 Ethereum에서 가장 중요한 interface layer 중 하나다.

- 인간은 source definition을 작성하고
- compiler와 tool은 ABI schema를 만들며
- caller는 calldata를 encode하고
- decoder는 같은 schema assumption을 사용해 return data와 log를 해석한다.

---

## 4. Protocol Structure

### What ABI Connects

```text
source interface
-> ABI schema
-> calldata / returndata encoding
-> contract interaction
```

### Why It Exists

contract는 external caller와 other contract가 argument를 encode하고 result를 decode할 수 있도록 common machine-readable interface convention이 필요하다.

### Scope

ABI는 interaction standard이지 bytecode나 state 자체와 같은 것은 아니다.

---

## 5. Function Selectors

### Four-Byte Dispatch Key

ABI specification은 calldata의 첫 4 byte가 호출할 function을 지정한다고 말한다.[^ref-solidity-abi]

### Construction

이 selector는 function signature의 Keccak-256 hash에서 계산한 뒤, 처음 4 byte를 취해 만든다.[^ref-solidity-abi]

### Why This Matters

onchain에서 calldata는 human-readable name으로 dispatch되지 않는다. encoded selector와 argument로 dispatch된다.

---

## 6. Schema-Dependent Encoding

### Not Self-Describing

ABI spec은 encoded data가 self-describing하지 않다고 명시한다.[^ref-solidity-abi]

### Consequence

correct ABI를 모르면:

- calldata decoding이 틀릴 수 있고
- return-value decoding이 틀릴 수 있으며
- event interpretation도 틀릴 수 있다.

### Research Implication

ABI-based interpretation은 raw byte 그 자체의 direct ground truth가 아니라, declared schema를 사용한 structured inference다.

---

## 7. Contract Interaction

### Reading and Writing Depend on ABI

Ethereum interaction tooling은 call format, response decode, human-usable interface 노출을 위해 ABI에 의존한다.[^ref-eth-sc-interacting]

### External and Internal Use

ABI는 offchain client뿐 아니라 known interface 아래에서 다른 contract를 call하는 contract에도 사용된다.[^ref-solidity-abi]

### Why This Matters

ABI는 Ethereum smart contract가 대규모로 compose되고 integrate될 수 있는 실질적 이유 중 하나다.

---

## 8. Technical Mechanics

### Simplified Call Structure

```text
call data
= 4-byte selector
+ encoded arguments
```

### Decode Structure

```text
raw bytes + correct ABI -> typed interpretation
```

### Failure Mode

```text
raw bytes + wrong ABI -> misleading interpretation
```

---

## 9. Security Assumptions

### ABI Trust Boundary

ABI는 caller나 decoder가 공급한 interface assumption이 정확할 때만 유효하다.

### Wrong Interface Risk

decoding tool은 wrong ABI가 주어졌을 때도 confident하지만 틀린 human-readable interpretation을 생성할 수 있다.

### Why This Matters

많은 analytics, wallet, dashboard가 ABI accuracy에 의존한다.

---

## 10. Mathematical or Economic Model

### Selector Model

```text
selector = first4(keccak256(function_signature))
```

이는 Solidity ABI specification에서 직접 따라온다.[^ref-solidity-abi]

### Interpretation Model

```text
decoded meaning
= raw bytes
+ schema assumption
```

### Consequence

ABI decoding은 schema dependence를 제거하지 못한다. 그것을 formalize할 뿐이다.

---

## 11. Protocol Implementation

### Primary Sources

이 unit의 핵심 source는 다음이다.

- Solidity ABI specification
- Ethereum contract interacting docs[^ref-solidity-abi][^ref-eth-sc-interacting]

### Why These Matter

이 둘은 formal encoding rule과 practical interaction context를 함께 제공한다.

---

## 12. On-Chain Implications

### What Analysts See

analyst는 raw calldata, returndata, log bytes를 볼 수 있다.

### What They Need

이를 human-meaningful form으로 바꾸려면 ABI가 필요하다.

### What They Still Do Not Get

ABI를 안다고 해서 다음을 자동으로 얻는 것은 아니다.

- contract intent
- governance context
- declared interface를 넘어선 semantic truth

---

## 13. Institutional Thinking

institution은 ABI를 analytics와 operation에서 critical interface assumption으로 다뤄야 한다.

### Practical Implications

- decoded interpretation과 함께 raw bytes도 보존해야 한다.
- contract version과 ABI provenance를 추적해야 한다.
- ABI mismatch를 real operational risk로 취급해야 한다.

### Better Research Posture

다음을 물어야 한다.

- 어떤 ABI를 사용했는가?
- contract는 upgradeable했는가?
- decoded label은 verified인가, inferred인가?

---

## 14. Common Misinterpretations

### "Calldata explains itself"

틀렸다. ABI data는 self-describing하지 않다.[^ref-solidity-abi]

### "A decoded function name is always ground truth"

틀렸다. correct ABI를 썼을 때만 성립한다.

### "ABI is only for frontend developers"

틀렸다. contract-to-contract interaction과 tool-to-contract interaction 모두에 fundamental하다.

---

## 15. Research Questions

1. 어떤 institutional dataset이 ABI mismatch risk에 가장 많이 노출되는가?
2. interface confidence가 partial할 때 ABI-decoded field는 어떻게 라벨링해야 하는가?
3. upgradeable contract는 ABI provenance를 어떻게 더 복잡하게 만드는가?

---

## 16. Practical Exercises

### Exercise 1

calldata를 올바르게 decode하려면 왜 ABI를 알아야 하는지 설명하라.

### Exercise 2

function selector의 공식을 쓰라.

### Exercise 3

wrong ABI를 사용해 발생할 수 있는 현실적인 failure mode 하나를 설명하라.

---

## 17. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| ABI as standard interaction schema | Directly specified | Solidity ABI spec |
| Selector derivation | Directly specified | Solidity ABI spec |
| ABI mismatch risk framing | Inference from sources | Derived from non-self-describing encoding |

---

## 18. Knowledge Graph

```text
ABI
├─ Function Selectors
├─ Argument Encoding
├─ Return Decoding
├─ Schema Dependence
└─ Tooling / Interaction Layer
```

---

## 19. References

### Primary Sources

[^ref-solidity-abi]: Solidity documentation, "Contract ABI Specification," accessed 2026-08-04, https://docs.soliditylang.org/en/latest/abi-spec.html

[^ref-eth-sc-interacting]: ethereum.org, "Interacting with smart contracts," accessed 2026-08-04, https://ethereum.org/developers/docs/smart-contracts/interacting/

---

## 20. Cross References

### Previous

- SMART-CONTRACTS-002 — Contract Lifecycle

### Next

- SMART-CONTRACTS-004 — Events

---

## Review Status

### Technical Review

Passed.

- ABI를 schema-based interface convention으로 정의했다.
- selector derivation과 non-self-describing encoding을 포함했다.
- wrong-ABI interpretation risk를 명시했다.

### Evidence Review

Passed.

- core claim은 Solidity ABI specification을 인용한다.
- interaction context는 official Ethereum docs를 인용한다.

### Editorial Review

Passed.

- 구조는 프로젝트 format을 따른다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 decoded data가 self-proving이라고 암시하지 않는다.
- ABI를 bytecode나 state와 혼동하지 않는다.

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
