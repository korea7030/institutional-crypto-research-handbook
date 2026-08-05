---
knowledge_id: ETHEREUM-FOUNDATION-003
title: World State
subtitle: Global State, State Root, Merkle Patricia Trie, Account Storage, 그리고 Ethereum이 state라고 실제로 의미하는 것
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 130 min
estimated_study: 340 min
last_reviewed: 2026-08-04
domain:
  - Ethereum
  - World State
  - Data Structures
  - Execution
parent:
  - Ethereum Foundations
prerequisites:
  - ETHEREUM-FOUNDATION-001
  - ETHEREUM-FOUNDATION-002
related_topics:
  - Account Model
  - State Transition
  - Merkle Patricia Trie
  - Storage Trie
  - State Root
primary_sources:
  - REF-ETH-DOC-INTRO-2026-001
  - REF-ETH-DOC-EVM-2026-001
  - REF-ETH-DOC-MPT-2026-001
  - REF-ETH-WP-001
  - REF-ETH-YP-README-001
tags:
  - ethereum
  - world-state
  - state-root
  - trie
  - storage
  - evm
---

# World State
> Ethereum Foundations  
> Research Unit: ETHEREUM-FOUNDATION-003

---

## Research Brief

```yaml
knowledge_id: ETHEREUM-FOUNDATION-003
title: World State
research_question: >
  What is Ethereum's world state, how is it represented and committed through
  state roots and Merkle Patricia tries, and how should researchers reason
  about global state, account storage, and evolving state data structures as of
  August 4, 2026?
document_type: deep-dive
difficulty: L300
prerequisites:
  - ETHEREUM-FOUNDATION-001
  - ETHEREUM-FOUNDATION-002
parent: Ethereum Foundations
previous: ETHEREUM-FOUNDATION-002
next: ETHEREUM-FOUNDATION-004
related_topics:
  - Account Model
  - State Transition
  - Patricia Merkle Trie
  - Storage Trie
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
  - Full trie algorithm derivation
  - Verkle tree deep dive
  - Stateless client research survey
  - JSON-RPC tutorial
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- protocol analysis에 충분할 정도로 정확하게 Ethereum world state를 정의할 수 있다.
- account, storage, state root가 어떻게 맞물리는지 설명할 수 있다.
- state trie, storage trie, transactions trie, receipts trie를 구분할 수 있다.
- 왜 Ethereum analysis가 UTXO-centric가 아니라 state-centric인지 설명할 수 있다.
- 현재 state data structure 설명이 왜 freshness check를 필요로 하는지 설명할 수 있다.

---

## 2. 핵심 질문

1. Ethereum의 world state란 무엇인가?
2. Ethereum은 block 안에서 global state에 어떻게 commit하는가?
3. `stateRoot`는 무엇을 나타내는가?
4. account storage와 global state는 어떻게 연결되는가?
5. 왜 Merkle Patricia Trie가 Ethereum 설계의 중심인가?
6. state story 중 어떤 부분은 stable하고, 어떤 부분은 앞으로도 진화할 것으로 예상되는가?

---

## 3. Executive Summary

Ethereum의 world state는 특정한 chain history 시점에서의 전체 account 집합과, 각 account에 연결된 balance, nonce, code commitment, storage commitment의 총합이다. 현대 Ethereum documentation은 이를, 모든 node가 합의하는 global virtual computer의 state로 설명한다.[^ref-eth-doc-intro][^ref-eth-doc-evm]

EVM documentation은 Ethereum을 state transition system `Y(S, T) = S'`로 제시한다. 여기서 valid old state `S`와 valid transaction 집합 `T`가 새로운 valid state `S'`를 만든다.[^ref-eth-doc-evm] 따라서 world state는 단순한 "chain 위의 data"가 아니다. 이전의 모든 valid transition이 만든, 현재 consensus-accepted result다.

Ethereum은 이 state에 modified Merkle Patricia Trie를 사용해 commit한다. official documentation에 따르면 Ethereum state는 이 구조로 encode되며, single root hash로 축약되어 blockchain에 저장된다. block header에는 `transactionsRoot`, `receiptsRoot`와 함께 `stateRoot`가 들어간다.[^ref-eth-doc-mpt]

이 architecture는 Ethereum을 Bitcoin과 근본적으로 다르게 만든다. Bitcoin의 active state는 UTXO set으로 모델링하는 것이 가장 적절하다. Ethereum의 active state는 account와 storage trie를 통해 인덱싱되는 global state object graph다.

---

## 4. Protocol Structure

### State-Centric Architecture

Ethereum의 core data model은 다음과 같다.

```text
global world state
-> accounts
-> per-account storage
-> code commitments
-> block-level commitments to state changes
```

### Three Root Commitments in the Block Header

official trie documentation은 execution layer block header에 세 개의 trie root가 포함된다고 설명한다.[^ref-eth-doc-mpt]

- `stateRoot`
- `transactionsRoot`
- `receiptsRoot`

### Why `stateRoot` Matters Most Here

`stateRoot`는 현재 world state를 요약하는 핵심 commitment다. 이는 모든 prior accepted transition 이후의 account/state database view를 대표한다.

---

## 5. Historical Context

### Whitepaper Intent

Ethereum whitepaper는 UTXO-style ledger 대신, account와 account 사이의 direct state transition을 도입했다.[^ref-eth-wp]

### Evolution of Formalization

시간이 지나며 Ethereum protocol description은 Yellow Paper, client implementation, EIP, 그리고 보다 최근에는 actively maintained execution spec를 통해 표현되었다. 그러나 Yellow Paper repository는 이제 later upgrade에 비해 outdated되었다고 스스로 경고한다.[^ref-eth-yp-readme]

### Why This Matters

world-state concept 자체는 stable하지만, exact formalization과 data-structure evolution은 여전히 protocol의 living part다.

---

## 6. State Trie and Storage Trie

### Global State Trie

official Merkle Patricia Trie documentation에 따르면, client가 block을 처리할 때마다 업데이트되는 하나의 global state trie가 존재한다.[^ref-eth-doc-mpt]

이 trie에서:

- path는 `keccak256(ethereumAddress)`
- value는 `rlp(ethereumAccount)`다.[^ref-eth-doc-mpt]

### Account Encoding

같은 documentation은 Ethereum account를 다음의 4-item array로 설명한다.

```text
[nonce, balance, storageRoot, codeHash]
```

[^ref-eth-doc-mpt]

### Per-Account Storage Trie

각 account는 자신의 storage trie를 가질 수 있다. official docs는 storage trie가 contract data가 사는 곳이며, account마다 별도의 storage trie가 있다고 설명한다.[^ref-eth-doc-mpt]

### Consequence

즉 world state는 계층적이다.

```text
state trie
-> account object
-> storageRoot
-> account-specific storage trie
```

---

## 7. What `stateRoot` Represents

### Consensus Commitment

`stateRoot`는 임의의 checksum이 아니다. current trie rule 아래의 full current world state에 대한 cryptographic commitment다.

### Operational Meaning

두 node가 같은 protocol rule 아래서 같은 valid state에 합의한다면, 같은 `stateRoot`로 수렴해야 한다.

### Block-to-Block Meaning

block이 처리되면 그 block의 transaction이 state를 갱신하고, resulting new world state가 새로운 `stateRoot`를 만든다.

---

## 8. Technical Mechanics

### EVM State Transition Function

EVM documentation은 다음과 같은 formal intuition을 제공한다.[^ref-eth-doc-evm]

```text
Y(S, T) = S'
```

여기서:

- `S`는 이전의 valid state
- `T`는 valid transaction 집합
- `S'`는 새로운 valid state다.

### Deterministic Shared State

node는 각자 임의의 state result를 만들어내지 않는다. 같은 valid transaction을 같은 rule 아래서 재실행하고, 같은 state commitment로 수렴한다.

### Storage vs Execution Memory

persistent account storage는 committed state의 일부다. temporary execution memory는 같은 것이 아니며, execution이 persistent value를 write하지 않는 한 account storage가 되지 않는다.

---

## 9. Why Ethereum State Is Hard

### Richer Than Balances

Ethereum state에는 balance보다 훨씬 많은 것이 포함된다.

- account nonce
- code commitment
- contract storage
- prior execution의 effect

### Shared Mutable Surface

많은 application이 하나의 global state machine을 공유하기 때문에, state growth, storage layout, proof structure는 모두 핵심 protocol concern이다.

### Future Evolution Signal

official trie documentation은 Ethereum이 가까운 미래에 Verkle Tree structure로 migrate할 계획이 있다고 설명한다.[^ref-eth-doc-mpt]

이것은 2026년 8월 4일 기준 current consensus state는 아니지만, state representation이 여전히 active protocol frontier라는 강한 신호다.

---

## 10. Security Assumptions

### State Integrity

Ethereum security는 각 state transition이 valid하고, resulting commitment가 protocol rule과 일치함을 node가 독립적으로 검증하는 데 의존한다.

### Data-Structure Security

trie structure가 중요한 이유는, 그것이 state data에 대한 cryptographic commitment와 proof path를 제공하기 때문이다. 구조가 ambiguous하거나 implementation마다 일관되지 않다면 consensus가 깨질 수 있다.

### Freshness Risk

연구자는 source freshness에 주의해야 한다. Yellow Paper repository는 post-Shanghai change에 비해 outdated되었다고 명시하므로, currentness가 중요한 state claim은 더 신선한 official source로 확인해야 한다.[^ref-eth-yp-readme]

---

## 11. Mathematical or Economic Model

### Minimal State Transition Abstraction

Ethereum docs는 다음 abstraction을 동기화한다.[^ref-eth-doc-evm]

```text
old state + valid transactions -> new state
```

또는 보다 형식적으로:

```text
Y(S, T) = S'
```

### Commitment Model

개념적으로는:

```text
stateRoot = commitment(world state)
```

이것은 literal protocol equation이 아니라 analytical shorthand지만, root가 왜 중요한지 잘 포착한다.

### Hierarchical State

구조적으로 보면:

```text
world state
= set of accounts
+ per-account storage commitments
```

즉 Ethereum에서는 account-level reasoning과 storage-level reasoning을 깔끔히 분리할 수 없다.

---

## 12. Protocol Implementation

### Official Documentation

official `intro-to-ethereum`, `EVM`, `Merkle Patricia Trie` docs는 current conceptual/structural world-state model을 설명하는 가장 명확한 primary documentation이다.[^ref-eth-doc-intro][^ref-eth-doc-evm][^ref-eth-doc-mpt]

### Yellow Paper Limitation

Yellow Paper는 역사적으로 중요하지만, repository는 그것이 outdated되었고 Shanghai에서 멈추며 later upgrade를 반영하지 않는다고 경고한다.[^ref-eth-yp-readme]

### Why This Is Enough for This Unit

foundational document 수준에서는 다음 stable concept를 잡아두면 충분하다.

- state는 global하게 존재한다.
- account는 state trie 안에 encode된다.
- storage는 storage root를 통해 commit된다.
- block header는 `stateRoot`를 통해 resulting state에 commit한다.

---

## 13. On-Chain Implications

### Rich Query Surface

Ethereum의 world-state model은 단순 transfer history를 넘어서는 query를 가능하게 한다.

- current balance
- current nonce
- contract code 존재 여부
- storage slot value
- archive-capable infrastructure를 통한 과거 block 시점의 historical state

### Archive vs Current View

모든 node가 모든 historical state query에 동일하게 답할 수 있는 것은 아니다. state access는 protocol question인 동시에 infrastructure question이기도 하다.

### Analytical Consequence

Ethereum analyst는 종종 다음 둘 다를 다뤄야 한다.

- event history
- current 또는 historical state snapshot

이 workload는 transaction flow 중심의 chain analysis와 다르다.

---

## 14. Institutional Thinking

institution은 Ethereum에서 transaction traffic이 아니라 state 자체를 primary object of analysis로 다뤄야 한다.

### Practical Implications

- contract risk는 transfer event가 아니라 storage transition에 존재하는 경우가 많다.
- research와 operation에서는 state-query infrastructure quality가 중요하다.
- historical state reconstruction은 operationally 비쌀 수 있다.
- data-structure change와 future state-model evolution은 trivia가 아니라 real protocol risk다.

### Better Research Posture

Ethereum state를 논할 때는 다음을 물어야 한다.

- current state, historical state, state proof 중 무엇을 말하는가?
- conceptual architecture에 대한 주장인가, 특정 data structure version에 대한 주장인가?
- 해당 진술을 뒷받침하는 source는 충분히 current한가?

---

## 15. Common Misinterpretations

### "Ethereum state is just balances"

틀렸다. balance뿐 아니라 nonce, code commitment, storage commitment도 포함한다.

### "The blockchain stores everything in one flat table"

틀렸다. Ethereum은 state, transaction, receipt를 위해 structured trie-based commitment를 사용한다.[^ref-eth-doc-mpt]

### "`stateRoot` is just another hash"

너무 약한 설명이다. 그것은 full current world state에 대한 root commitment다.

### "The Yellow Paper alone is enough for current state architecture"

2026년에는 틀렸다. repository 자체가 outdated되었다고 경고한다.[^ref-eth-yp-readme]

---

## 16. Research Questions

1. eventual Verkle migration 같은 future state-commitment change에 institution은 어떻게 대비해야 하는가?
2. 어떤 research task가 event/log indexing만으로는 부족하고, truly archive-state access를 요구하는가?
3. source freshness가 protocol era에 따라 다를 때, historical state claim은 어떻게 라벨링해야 하는가?

---

## 17. Practical Exercises

### Exercise 1

`stateRoot`와 `storageRoot`의 차이를 설명하라.

### Exercise 2

하나의 account storage가 global state 안에 어떻게 들어가는지 짧게 설명하라.

### Exercise 3

Ethereum block-header documentation에 나오는 세 trie root를 나열하고, 각각이 무엇에 commit하는지 설명하라.

### Exercise 4

왜 Ethereum on-chain analysis가 종종 state-centric인지 설명하라.

---

## 18. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Ethereum as a state transition system | Directly specified | EVM docs |
| Global state trie and storage trie structure | Directly specified | Merkle Patricia Trie docs |
| Whitepaper account/state motivation | Directly specified | Whitepaper |
| Yellow Paper freshness limitation | Directly specified | Yellow Paper README |
| Institutional implications for state-heavy analysis | Inference from sources | Derived from state architecture |

---

## 19. Knowledge Graph

```text
World State
├─ Global Commitment
│  └─ stateRoot
├─ Global State Trie
│  ├─ account path = keccak256(address)
│  └─ account value = rlp(account)
├─ Account Object
│  ├─ nonce
│  ├─ balance
│  ├─ storageRoot
│  └─ codeHash
├─ Per-Account Storage
│  └─ storage trie
└─ Related Commitments
   ├─ transactionsRoot
   └─ receiptsRoot
```

---

## 20. 참고문헌

### Primary Sources

[^ref-eth-doc-intro]: ethereum.org, "Technical intro to Ethereum," official documentation describing Ethereum as a shared state machine, page last updated April 22, 2026, https://ethereum.org/developers/docs/intro-to-ethereum/, accessed 2026-08-04.

[^ref-eth-doc-evm]: ethereum.org, "Ethereum Virtual Machine (EVM)," official documentation describing the state transition function `Y(S, T) = S'`, published 2026, https://ethereum.org/developers/docs/evm/, accessed 2026-08-04.

[^ref-eth-doc-mpt]: ethereum.org, "Merkle Patricia Trie," official documentation describing the state trie, storage trie, and block-header roots, published 2026, https://ethereum.org/developers/docs/data-structures-and-encoding/patricia-merkle-trie/, accessed 2026-08-04.

[^ref-eth-wp]: Vitalik Buterin, "Ethereum Whitepaper," official ethereum.org whitepaper page, including the account/state framing, https://ethereum.org/whitepaper/, accessed 2026-08-04.

[^ref-eth-yp-readme]: Ethereum Yellow Paper repository README, including the statement that the Yellow Paper is currently outdated and stops at Shanghai rather than later upgrades, https://github.com/ethereum/yellowpaper, accessed 2026-08-04.

### Supporting Interpretation Notes

- Where this document discusses institutional infrastructure implications or archive-query burden, those are analytical inferences from the official state model rather than direct normative protocol statements.

---

## 21. 교차 참조

### Previous

- ETHEREUM-FOUNDATION-002 — Account Model

### Next

- ETHEREUM-FOUNDATION-004 — State Transition

### Related

- ETHEREUM-FOUNDATION-001 — Ethereum Vision
- BITCOIN-014 — UTXO Model

---

## Review Status

### Technical Review

Passed.

- global state, state trie, storage trie를 명확히 분리했다.
- `stateRoot`를 generic hash가 아니라 commitment로 정의했다.
- current specification과 historical specification의 경계를 구분했다.
- future Verkle reference는 current fact가 아니라 future-looking signal로 라벨링했다.

### Evidence Review

Passed.

- state transition과 world-state claim은 current official docs를 인용한다.
- trie-root와 storage-structure claim은 official MPT docs를 인용한다.
- Yellow Paper freshness caveat는 repository README를 인용한다.
- institutional implication은 inference로 라벨링했다.

### Editorial Review

Passed.

- 구조는 프로젝트 문서 패턴을 따른다.
- metadata는 완전하다.
- terminology는 world state, stateRoot, storageRoot, trie, EVM으로 일관된다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 Ethereum state를 balance로만 평탄화하지 않는다.
- Yellow Paper를 fully current한 문서처럼 사용하지 않는다.
- future Verkle plan을 current state commitment와 혼동하지 않는다.
- 모든 node가 모든 historical state를 동등하게 노출한다고 암시하지 않는다.

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
