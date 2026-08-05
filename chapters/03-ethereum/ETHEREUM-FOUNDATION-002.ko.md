---
knowledge_id: ETHEREUM-FOUNDATION-002
title: Account Model
subtitle: Externally Owned Account, Contract Account, Nonce, Storage Root, 그리고 현대 Ethereum identity의 경계
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 125 min
estimated_study: 320 min
last_reviewed: 2026-08-04
domain:
  - Ethereum
  - Accounts
  - EVM
  - State
parent:
  - Ethereum Foundations
prerequisites:
  - ETHEREUM-FOUNDATION-001
  - BITCOIN-014
related_topics:
  - World State
  - Transactions
  - Nonce
  - Contract Code
  - Validator Keys
primary_sources:
  - REF-ETH-WP-001
  - REF-ETH-DOC-ACCOUNTS-2026-001
  - REF-ETH-DOC-INTRO-2026-001
  - REF-EIP-7702
  - REF-ETH-EXEC-SPECS-README-2026-001
tags:
  - ethereum
  - accounts
  - eoa
  - contract-account
  - nonce
  - codehash
---

# Account Model
> Ethereum Foundations  
> Research Unit: ETHEREUM-FOUNDATION-002

---

## Research Brief

```yaml
knowledge_id: ETHEREUM-FOUNDATION-002
title: Account Model
research_question: >
  How does Ethereum represent ownership and executable entities through its
  account model, what fields define an account, how do externally owned and
  contract-controlled accounts differ, and which classical account-model
  assumptions now require qualification in August 2026?
document_type: deep-dive
difficulty: L300
prerequisites:
  - ETHEREUM-FOUNDATION-001
  - BITCOIN-014
parent: Ethereum Foundations
previous: ETHEREUM-FOUNDATION-001
next: ETHEREUM-FOUNDATION-003
related_topics:
  - World State
  - Transactions
  - Contract Creation
  - Nonce
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
  - Full wallet UX survey
  - Smart contract language details
  - Address checksum tutorial
  - Exhaustive account-abstraction design space
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Ethereum의 account model을 정의하고 Bitcoin의 UTXO model과 대비할 수 있다.
- Ethereum account를 구성하는 field를 설명할 수 있다.
- externally owned account와 contract account를 구분할 수 있다.
- nonce가 왜 존재하는지, replay protection과 ordering에서 왜 중요한지 설명할 수 있다.
- modern Ethereum에서 전통적인 EOA 대 contract distinction이 왜 qualification을 필요로 하는지 설명할 수 있다.

---

## 2. 핵심 질문

1. Ethereum account란 무엇인가?
2. Ethereum은 왜 UTXO 대신 account model을 선택했는가?
3. account에는 어떤 field가 저장되는가?
4. EOA와 contract account는 어떻게 다른가?
5. nonce는 무엇을 하는가?
6. 왜 전통적인 "EOA에는 code가 없다"는 규칙은 2026년에 더 이상 충분한 완전한 설명이 아닌가?

---

## 3. Executive Summary

Ethereum의 account model은 Ethereum state의 기본 object model이다. original whitepaper는 Ethereum state가 account라 불리는 object들로 이루어지며, 각 account는 20-byte address와 네 개의 field, 즉 nonce, balance, contract code if present, storage를 가진다고 설명한다.[^ref-eth-wp]

현재 Ethereum documentation은 같은 개념을 더 operational한 표현으로 설명한다. Ethereum account는 ETH balance를 가지고 Ethereum에서 message를 보낼 수 있는 entity이며, account는 user-controlled일 수도 있고 smart contract로 deployed될 수도 있다.[^ref-eth-doc-accounts]

고전적 구분은 다음과 같다.

- externally owned account (EOA): private key로 제어됨
- contract account: code로 제어됨[^ref-eth-doc-accounts]

이 구분은 여전히 유용하지만, 2026년에는 더 이상 전체 이야기가 아니다. EIP-7702와 이를 포함한 execution-spec release는 EOA와 code를 둘러싼 오래된 invariant를 수정하므로, 연구자는 과거 mental model을 timeless protocol truth처럼 제시하면 안 된다.[^ref-eip-7702][^ref-eth-exec-specs]

더 깊은 핵심은 Ethereum이 state transition의 직접 주소 지정 대상 객체로 account를 사용한다는 점이다. 이것이 Ethereum이 Bitcoin과 달라지는 foundational한 방식 중 하나다.

---

## 4. Protocol Structure

### Account-Centric Design

Ethereum에서 state는 독립적인 spendable output 집합이 아니라 account를 중심으로 조직된다.

account는 다음을 담을 수 있는 addressable object다.

- ETH balance
- nonce를 통한 replay/order state
- executable code
- persistent storage commitment

### Two Classical Types

현재 Ethereum documentation은 두 가지 account type을 설명한다.[^ref-eth-doc-accounts]

| Type | Controlled By | Can Initiate Transactions? |
|---|---|---|
| Externally owned account | private key | yes |
| Contract account | contract code | 고전적 모델에서는 originator로서 no |

### Why This Matters

즉 Ethereum에서는 identity, value holding, executable logic가 모두 같은 state object family 안에 존재한다.

---

## 5. Historical Context

### Original Whitepaper Framing

Ethereum whitepaper는 account를 base state object로 도입했다. Ethereum이 원한 것은 UTXO-style ledger가 아니라, account 사이의 direct transfer of value and information이었기 때문이다.[^ref-eth-wp]

### Why This Was Attractive

account model은 application-level reasoning의 일부를 단순화한다.

- balance는 account에 존재한다.
- contract는 address에 존재한다.
- 반복 상호작용은 같은 logical object를 재사용한다.
- stateful application은 자신의 address 아래에 data를 지속적으로 저장할 수 있다.

### Trade-Off

이 application addressing의 단순성은, 그 대신 더 복잡한 shared-state machine과 richer execution semantic을 요구한다.

---

## 6. Account Fields

### Four Core Fields

현재 Ethereum accounts documentation과 whitepaper는 모두 네 개의 핵심 field를 설명한다.[^ref-eth-doc-accounts][^ref-eth-wp]

```text
nonce
balance
storageRoot
codeHash
```

### `nonce`

현재 docs는 nonce를, EOA가 보낸 transaction 수 또는 contract account가 생성한 contract 수를 나타내는 counter로 정의한다.[^ref-eth-doc-accounts]

그 기능은 replay를 방지하고, per-account sequencing을 제공하는 것이다.

### `balance`

`balance`는 해당 address가 소유한 wei의 양이다.[^ref-eth-doc-accounts]

### `codeHash`

`codeHash`는 해당 account와 연관된 EVM code를 가리킨다. docs에 따르면 classical model에서 EOA의 경우 이것은 empty string의 hash다.[^ref-eth-doc-accounts]

### `storageRoot`

`storageRoot`는 account의 persistent storage를 encoding하는 Merkle Patricia Trie의 root다.[^ref-eth-doc-accounts]

---

## 7. EOAs and Contract Accounts

### Externally Owned Accounts

EOA는 cryptographic key에 의해 제어된다. 현재 docs는 EOAs가 transaction을 initiate할 수 있고, 네트워크 storage cost 없이 생성된다고 설명한다.[^ref-eth-doc-accounts]

### Contract Accounts

contract account는 deployed code에 의해 제어된다. transaction과 message call에 반응하고, balance를 보유하며, storage를 유지할 수 있다.[^ref-eth-doc-accounts][^ref-eth-doc-intro]

### Contract Accounts Do Not "Sign"

contract account는 일반적인 의미의 private key를 소유하지 않는다. valid transaction flow가 그것을 trigger하면, 네트워크가 그 code를 실행한다.

### Classical Summary

전통적인 concise summary는 다음과 같다.

```text
EOA signs and initiates
contract executes and reacts
```

이것은 여전히 유용한 baseline이지만, 더 이상 현대 행동 전체를 포괄하는 설명은 아니다.

---

## 8. Nonce and Ordering

### Why Nonce Exists

nonce가 없다면 signed transaction은 replay될 수 있다.

### Security Function

현재 docs는 같은 account에 대해 동일한 nonce를 가진 transaction은 하나만 실행될 수 있다고 설명하며, 이를 replay protection과 명시적으로 연결한다.[^ref-eth-doc-accounts]

### Operational Meaning

nonce는 per-account ordering semantic도 만든다. 이것은 chain 전체의 global transaction index가 아니다. account-local하다.

### Contract-Side Meaning

contract account에 대해서는 역사적으로 nonce가 해당 account에서 생성된 contract 수를 센다.[^ref-eth-doc-accounts]

---

## 9. Modern Qualification: EIP-7702

### Why the Old Model Needs Qualification

EIP-7702는 여러 오래된 invariant를 깨뜨린다. 이 EIP는 계정이 delegated된 이후에는 예전 의미에서 그 account가 transaction을 originate하지 않았더라도 account balance가 감소할 수 있고, EOA nonce가 execution이 시작된 후 증가할 수 있으며, `tx.origin == msg.sender`가 더 이상 topmost frame에서만 발생하는 것이 아니라고 명시한다.[^ref-eip-7702]

### Currentness

execution-specs repository는 Prague가 2025년 5월 7일 release되었고, 포함된 EIP 중 하나로 EIP-7702를 나열한다.[^ref-eth-exec-specs]

### Practical Research Consequence

즉 "EOA에는 code가 없으므로 언제나 단 하나의 단순한 방식으로만 동작한다"는 요약은 2026년 8월 4일 기준 더 이상 안전한 universal summary가 아니다.

### What Still Survives

account model은 여전히 key-based authority와 code-based behavior를 구분하지만, 그 distinction을 둘러싼 오래된 invariant 일부는 완화되었다.

---

## 10. Technical Mechanics

### Account Addressing

account는 global state 안의 directly addressed object다. 현재 docs는 account address가 20 byte이며, 보통 `0x` prefix가 붙은 42-character hex string으로 표시된다고 설명한다.[^ref-eth-doc-accounts]

### State Interaction

transaction은 account state를 업데이트한다.

- balance 차감
- nonce 증가
- code 실행
- storage 업데이트
- contract execution path를 통한 log emission

### Persistence

transient execution memory와 달리, account field와 contract storage는 block을 넘어 지속되는 persistent state에 속한다.

---

## 11. Security Assumptions

### Key Security

EOA의 security는 근본적으로 private-key control에 의존한다.

### Code Security

contract account의 security는 code correctness, deployment assumption, message-flow safety에 의존한다.

### Invariant Risk

연구자와 auditor는 종종 "EOA는 X를 할 수 없다" 같은 단순한 가정에 의존한다. 하지만 현대 protocol evolution은 이런 가정을 영구적 진실로 취급하지 말고, current rule과 대조해 점검해야 함을 뜻한다.[^ref-eip-7702]

---

## 12. Mathematical or Economic Model

### Minimal Account State Model

개념적 account object는 다음과 같이 표현할 수 있다.

```text
account = (nonce, balance, storageRoot, codeHash)
```

이것은 여기서 임의로 발명한 implementation detail이 아니라, Ethereum의 primary source에 있는 four-field description을 반영한 structural model이다.[^ref-eth-wp][^ref-eth-doc-accounts]

### Replay Constraint

classical transaction model에서 주어진 origin account에 대해:

```text
valid next nonce = current nonce
```

성공적으로 처리된 뒤에는:

```text
new nonce = old nonce + 1
```

이 단순화된 표현은 ordering intuition을 포착한다. 다만 현대 delegated-code behavior는 nonce 변화가 언제 어떻게 발생하는지에 대한 과거 가정을 더 복잡하게 만들 수 있다.[^ref-eip-7702]

---

## 13. Protocol Implementation

### Current Documentation

account field와 type에 대한 가장 명확한 current operational description은 official Ethereum accounts documentation에 있다.[^ref-eth-doc-accounts]

### Whitepaper Role

whitepaper는 account model이 우연한 implementation detail이 아니라, Ethereum original design의 foundation이라는 점을 보여준다는 의미에서 여전히 유용하다.[^ref-eth-wp]

### Spec Evolution

protocol rule은 진화하므로, current implementation/specification state는 historical summary에만 의존하지 말고 modern execution specs와 adopted EIP를 통해 확인해야 한다.[^ref-eip-7702][^ref-eth-exec-specs]

---

## 14. On-Chain Implications

### What Analysts Observe

on-chain analyst는 다음을 관찰할 수 있다.

- account balance
- nonce progression
- contract deployment
- code-bearing address
- state query와 trace를 통한 간접적인 storage effect

### What Requires Care

단순한 "EOA vs contract" heuristic은 protocol behavior가 진화할수록 신뢰도가 떨어질 수 있다.

### Comparison with Bitcoin

Ethereum address는 Bitcoin UTXO와 유사한 것이 아니다. persistent state object에 더 가깝다.

---

## 15. Institutional Thinking

institution은 Ethereum account를 단순한 balance container가 아니라 stateful control object로 다뤄야 한다.

### Practical Implications

- key-management policy는 필요하지만 충분조건은 아니다.
- contract-risk analysis는 곧 account-risk analysis다.
- address labeling은 user-controlled account와 code-bearing 또는 delegated-behavior account를 신중하게 구분해야 한다.
- EOA에 대한 historical assumption은 version-aware해야 한다.

### Better Research Posture

account behavior를 분류할 때 institution은 다음을 물어야 한다.

- 이 분석은 어떤 protocol era를 가정하는가?
- 이 진술은 classical EOA, contract account, modern delegated code 중 무엇에 대한 것인가?
- 이 주장은 protocol-level인가, 아니면 단지 legacy heuristic인가?

---

## 16. Common Misinterpretations

### "Ethereum accounts are just addresses with balances"

틀렸다. account는 ordering state도 encode하며, storage와 code commitment를 포함할 수도 있다.

### "Contract accounts are just wallets with scripts"

너무 약한 설명이다. contract account는 persistent programmable state object다.

### "EOAs never have code-related behavior concerns"

EIP-7702 이후로는 이것을 current universal statement로 취급하는 것이 더 이상 안전하지 않다.[^ref-eip-7702]

### "Nonce is a chain-wide sequence number"

틀렸다. nonce는 per account다.

---

## 17. Research Questions

1. modern analytics pipeline은 EIP-7702 이후 classical EOA heuristic을 어떻게 업데이트해야 하는가?
2. account classification에서 key risk와 code risk를 가장 잘 분리하는 institutional control은 무엇인가?
3. 연구자는 historical Ethereum account-model explanation을 current behavior에 대해 오해를 주지 않으면서 어떻게 제시해야 하는가?

---

## 18. Practical Exercises

### Exercise 1

`balance`와 `storageRoot`의 차이를 설명하라.

### Exercise 2

Bitcoin UTXO와 Ethereum account를 한 단락으로 비교하라.

### Exercise 3

nonce가 존재하는 이유 세 가지를 나열하라.

### Exercise 4

왜 EIP-7702가 오래된 EOA 요약을 사용할 때 주의를 요구하는지 설명하라.

---

## 19. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Four-field account structure | Directly specified | Whitepaper and official accounts docs |
| Classical EOA / contract distinction | Directly specified | Official accounts docs |
| Modern qualification of old account invariants | Directly specified | EIP-7702 and execution-spec release context |
| Institutional caution about heuristics | Inference from sources | Derived from current protocol evolution |

---

## 20. Knowledge Graph

```text
Account Model
├─ Account Fields
│  ├─ nonce
│  ├─ balance
│  ├─ storageRoot
│  └─ codeHash
├─ Classical Types
│  ├─ externally owned account
│  └─ contract account
├─ Security Surfaces
│  ├─ private keys
│  ├─ contract code
│  └─ replay protection
├─ Modern Qualification
│  ├─ EIP-7702
│  └─ invariant softening
└─ Related Concepts
   ├─ world state
   ├─ transactions
   └─ state transition
```

---

## 21. 참고문헌

### Primary Sources

[^ref-eth-wp]: Vitalik Buterin, "Ethereum Whitepaper," official ethereum.org whitepaper page, including the account-model description, https://ethereum.org/whitepaper/, accessed 2026-08-04.

[^ref-eth-doc-accounts]: ethereum.org, "Ethereum accounts," official documentation describing account types and fields, published 2026, https://ethereum.org/developers/docs/accounts, accessed 2026-08-04.

[^ref-eth-doc-intro]: ethereum.org, "Technical intro to Ethereum," official documentation describing the EVM and shared state, page last updated April 22, 2026, https://ethereum.org/developers/docs/intro-to-ethereum/, accessed 2026-08-04.

[^ref-eip-7702]: EIP-7702, "Set Code for EOAs," Ethereum Improvement Proposals, including backward-compatibility notes about changed invariants, https://eips.ethereum.org/EIPS/eip-7702, accessed 2026-08-04.

[^ref-eth-exec-specs]: Ethereum execution-specs repository README, protocol releases table showing Prague released on May 7, 2025 and including EIP-7702, https://github.com/ethereum/execution-specs, accessed 2026-08-04.

### Supporting Interpretation Notes

- Where this document discusses heuristic risk or institutional classification risk, those statements are inferences from the cited current protocol sources rather than direct normative protocol text.

---

## 22. 교차 참조

### Previous

- ETHEREUM-FOUNDATION-001 — Ethereum Vision

### Next

- ETHEREUM-FOUNDATION-003 — World State

### Related

- BITCOIN-014 — UTXO Model
- ETHEREUM-FOUNDATION-004 — State Transition

---

## Review Status

### Technical Review

Passed.

- account model을 its four fields와 classical account type을 중심으로 설명했다.
- nonce, storage, code semantic을 분리했다.
- outdated EOA claim을 막기 위해 EIP-7702를 포함했다.
- Bitcoin UTXO comparison은 개념적이고 제한적으로 유지했다.

### Evidence Review

Passed.

- whitepaper와 official accounts docs는 four-field model과 classical type distinction을 뒷받침한다.
- EIP-7702와 execution-specs는 current-state qualification을 뒷받침한다.
- interpretive caution은 inference로 라벨링했다.

### Editorial Review

Passed.

- 구조는 프로젝트 문서 패턴을 따른다.
- metadata는 완전하다.
- terminology는 EOA, contract account, nonce, codeHash, storageRoot로 일관된다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 2015-era EOA rule을 timeless truth로 취급하지 않는다.
- account를 balance로만 환원하지 않는다.
- private-key control과 contract-code control을 혼동하지 않는다.
- 단순한 address heuristic의 완전성을 과장하지 않는다.

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
