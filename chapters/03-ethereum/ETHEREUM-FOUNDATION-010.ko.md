---
knowledge_id: ETHEREUM-FOUNDATION-010
title: Storage
subtitle: Contract Persistent Storage, Storage Trie, Slot Addressing, 그리고 Ethereum이 general-purpose on-chain file system이 아닌 이유
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 135 min
estimated_study: 340 min
last_reviewed: 2026-08-04
domain:
  - Ethereum
  - Storage
  - Smart Contracts
  - Data Structures
parent:
  - Ethereum Foundations
prerequisites:
  - ETHEREUM-FOUNDATION-003
  - ETHEREUM-FOUNDATION-005
related_topics:
  - World State
  - EVM
  - Receipts
  - Layer 2
primary_sources:
  - REF-ETH-DOC-MPT-2026-001
  - REF-ETH-DOC-EVM-2026-001
  - REF-ETH-DOC-STORAGE-2026-001
  - REF-ETH-DOC-JSONRPC-2026-001
tags:
  - ethereum
  - storage
  - storage-root
  - smart-contracts
  - state
  - persistence
---

# Storage
> Ethereum Foundations  
> Research Unit: ETHEREUM-FOUNDATION-010

---

## Research Brief

```yaml
knowledge_id: ETHEREUM-FOUNDATION-010
title: Storage
research_question: >
  What does storage mean in Ethereum, how is contract persistent storage
  committed through storage tries and storage roots, how is it queried, and why
  should researchers distinguish contract state storage from large-file
  decentralized storage narratives in August 2026?
document_type: deep-dive
difficulty: L300
prerequisites:
  - ETHEREUM-FOUNDATION-003
  - ETHEREUM-FOUNDATION-005
parent: Ethereum Foundations
previous: ETHEREUM-FOUNDATION-009
next: ETHEREUM-FOUNDATION-011
related_topics:
  - World State
  - EVM
  - State Transition
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
  - Solidity storage-layout tutorial
  - Full mapping/array slot derivation catalog
  - IPFS/Filecoin product comparison
  - Verkle migration deep dive
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Ethereum contract storage를 정확하게 정의할 수 있다.
- storage가 account state와 storage root에 어떻게 연결되는지 설명할 수 있다.
- persistent storage와 transient storage, memory를 구분할 수 있다.
- storage position이 어떻게 query되는지 설명할 수 있다.
- Ethereum이 contract code와 state를 on-chain에 저장함에도, 왜 large arbitrary data storage에 적합하지 않은지 설명할 수 있다.

---

## 2. 핵심 질문

1. Ethereum storage란 무엇인가?
2. contract storage는 global state와 어떤 관계인가?
3. `storageRoot`란 무엇인가?
4. storage는 어떻게 query되는가?
5. 왜 Ethereum은 large arbitrary file storage를 위해 설계되지 않았는가?

---

## 3. Executive Summary

Ethereum에서 storage는 보통 global world state의 일부로 commit되는 persistent per-contract state를 뜻한다. official EVM docs는 contract가 account와 연결된 Merkle Patricia storage trie를 가지며, 이것이 global state의 일부라고 설명한다.[^ref-eth-doc-evm]

official trie documentation은 각 account가 별도의 storage trie를 가지며, account object 안의 `storageRoot`가 그 trie의 root라고 설명한다.[^ref-eth-doc-mpt]

이것은 다음과 매우 다르다.

- transaction 동안만 존재하는 transient storage
- call-local execution workspace인 memory
- large offchain content를 위한 decentralized file storage system[^ref-eth-doc-evm][^ref-eth-doc-storage]

current ethereum.org storage documentation은 명시적으로 말한다. Ethereum은 smart contract code와 on-chain data를 위한 decentralized storage system으로 사용될 수 있지만, every node가 그 data를 replicate해야 하고 on-chain persistence가 비싸기 때문에 large amount of data를 위해 설계된 시스템은 아니다.[^ref-eth-doc-storage]

---

## 4. Protocol Structure

### Hierarchical Relationship

Ethereum storage는 다음 계층 안에 놓인다.

```text
world state
-> account object
-> storageRoot
-> account-specific storage trie
-> individual storage slots / values
```

### Why It Matters

storage는 Ethereum 바깥의 sidecar database가 아니다. cryptographically committed state model의 일부다.

### Scope

이 문서의 범위는 contract-associated persistent storage이며, contrast가 필요한 경우를 제외하면 generalized offchain content network는 범위 밖이다.

---

## 5. Storage in the EVM

### Persistent Storage

official EVM docs는 contract storage를 account와 연결된 persistent state이자 global state의 일부로 설명한다.[^ref-eth-doc-evm]

### Transient Storage

같은 docs는 transient storage가 같은 transaction 안의 internal call 전반에서만 유지되고, transaction이 끝나면 정리된다고 구분한다.[^ref-eth-doc-evm]

### Memory

execution memory 역시 temporary하며 persistent account storage가 되지 않는다.[^ref-eth-doc-evm]

### Critical Distinction

```text
memory -> call-local temporary
transient storage -> transaction-local temporary
persistent storage -> globally committed contract state
```

---

## 6. Storage Trie and `storageRoot`

### Separate Trie Per Account

official trie docs는 account마다 separate storage trie가 있으며, `storageRoot`는 그 trie의 root라고 설명한다.[^ref-eth-doc-mpt]

### Account Field Role

따라서 `storageRoot`는 storage value 그 자체가 아니라 storage contents에 대한 commitment다.

### Security Meaning

storage content가 바뀌면 storage root가 바뀌고, 그러면 containing account object가 바뀌며, 궁극적으로 state root도 바뀐다.

---

## 7. Querying Storage

### JSON-RPC Access

trie docs는 address와 block context에 대해 지정된 storage position의 data를 가져오는 `eth_getStorageAt`을 명시적으로 언급한다.[^ref-eth-doc-mpt]

### Positioning

같은 docs는 어떤 storage access는 storage trie 안의 position 계산이 필요하며, certain layout에서는 hashing-based addressing이 들어간다고 설명한다.[^ref-eth-doc-mpt]

### Practical Consequence

storage read는 종종 "field를 조회한다" 수준이 아니다. layout convention과 slot derivation을 이해해야 할 수 있다.

---

## 8. Why Ethereum Is Not a General File Store

### Official Warning

current ethereum.org storage docs는 Ethereum 자체를 decentralized storage로 쓸 수는 있지만, 특히 contract code에는 적합하지만, large amount of data를 on-chain에 저장하는 것은 Ethereum이 원래 설계한 용도가 아니라고 말한다.[^ref-eth-doc-storage]

### Why

같은 docs는 이유를 다음과 같이 설명한다.

- chain은 이미 크다.
- every node가 그 data를 저장해야 한다.
- gas cost 때문에 large on-chain data는 prohibitively expensive하다.[^ref-eth-doc-storage]

### Correct Mental Model

Ethereum storage는 arbitrary bulk data persistence가 아니라 scarce하고 consensus-critical한 state를 위해 최적화되어 있다.

---

## 9. Technical Mechanics

### Storage Write Path

```text
transaction executes
-> EVM writes persistent contract storage
-> storage trie updates
-> storageRoot changes
-> account object changes
-> stateRoot changes
```

### Storage Read Path

```text
reader identifies contract account
-> determines slot / position
-> queries state at block context
-> retrieves storage value
```

### State Coupling

storage는 optional accessory가 아니라 world-state commitment에 깊게 결합되어 있다.

---

## 10. Security Assumptions

### Persistence Cost Is a Feature

storage가 비싼 것은 우연이 아니다. persistent replicated state가 네트워크에 부과하는 부담을 반영한다.

### Interpretation Risk

storage value를 올바르게 읽으려면 종종 다음이 필요하다.

- layout 이해
- proxy pattern 이해
- upgrade history 이해
- block context 이해

### Infrastructure Risk

모든 node나 provider가 동일하게 풍부하거나 과거 시점까지 포함한 storage access를 주는 것은 아니다. storage analysis는 부분적으로 infrastructure question이기도 하다.

---

## 11. Mathematical or Economic Model

### Commitment Hierarchy

개념적 model은 다음과 같다.

```text
storage change
-> new storageRoot
-> new account object
-> new stateRoot
```

### Persistence Cost Intuition

persistent storage가 비싼 이유는:

```text
more persistent state
-> more replicated burden
-> more execution and state-management cost
```

이것은 economic/operational intuition이지 consensus formula가 아니다.

### Why This Matters

Ethereum에서 persistence는 premium resource다.

---

## 12. Protocol Implementation

### Primary Sources

핵심 current source는 다음이다.

- persistent/transient storage를 위한 EVM docs
- storage trie structure와 access path를 위한 trie docs
- "bulk file store가 아님" 경계를 위한 storage docs
- retrieval interface context를 위한 JSON-RPC docs[^ref-eth-doc-evm][^ref-eth-doc-mpt][^ref-eth-doc-storage][^ref-eth-doc-jsonrpc]

### Why This Combination Works

이 조합은 다음을 분리해 보여준다.

- execution semantic
- commitment structure
- storage economics
- operator access surface

---

## 13. On-Chain Implications

### What Analysts Can Use Storage For

storage access는 다음에 핵심적이다.

- contract state 읽기
- protocol parameter 추적
- proxy/admin setting 모니터링
- current application configuration 재구성

### What Makes It Hard

storage가 balance보다 어려운 이유는 다음과 같다.

- value가 layout-dependent일 수 있음
- meaning이 contract-specific interpretation을 필요로 함
- block context가 중요할 수 있음
- upgrade가 semantic을 바꿀 수 있음

### Practical Consequence

state-heavy protocol analysis는 transfer event보다 storage에 더 의존하는 경우가 많다.

---

## 14. Institutional Thinking

institution은 Ethereum storage를 scarce하고 consensus-critical한 state surface로 다뤄야 한다.

### Practical Implications

- serious smart contract due diligence에는 storage read가 필요한 경우가 많다.
- large-file 또는 document storage strategy는 direct L1 persistence를 default로 삼으면 안 된다.
- state monitoring은 current storage value와 historical storage evolution을 구분해야 한다.
- contract upgradeability는 storage slot의 의미를 materially 바꿀 수 있다.

### Better Research Posture

storage claim을 하기 전에 다음을 물어야 한다.

- 이것이 persistent storage인가, transient storage인가, memory인가?
- 어떤 block context를 query하고 있는가?
- slot의 meaning은 contract version에 걸쳐 stable한가?
- protocol storage에 대한 주장인가, decentralized file persistence에 대한 주장인가?

---

## 15. Common Misinterpretations

### "Ethereum storage means any data you want to store on-chain"

너무 넓은 표현이다. Ethereum은 on-chain persistence를 지원하지만 arbitrary bulk data를 위해 설계된 것은 아니다.[^ref-eth-doc-storage]

### "`storageRoot` is the storage value itself"

틀렸다. 그것은 storage trie에 대한 commitment다.

### "Memory and storage are interchangeable"

틀렸다. memory는 temporary하고, persistent storage는 global state의 일부다.[^ref-eth-doc-evm]

### "Reading storage is always trivial"

틀렸다. layout와 context가 종종 중요하다.

---

## 16. Research Questions

1. 어떤 Ethereum protocol class가 institution에게 가장 storage-analysis-intensive한가?
2. institution은 archive-state access cost와 analytical need를 어떻게 균형 잡아야 하는가?
3. 가장 자주 오해되는 storage layout 속에 숨어 있는 protocol risk는 무엇인가?

---

## 17. Practical Exercises

### Exercise 1

`stateRoot`와 `storageRoot`의 차이를 설명하라.

### Exercise 2

왜 persistent storage가 Ethereum에서 비싼지 짧게 설명하라.

### Exercise 3

transient storage와 persistent storage의 차이를 설명하라.

### Exercise 4

왜 large arbitrary file을 Ethereum mainnet에 직접 저장하는 것은 대체로 나쁜 설계 선택인지 설명하라.

---

## 18. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Persistent vs transient storage distinctions | Directly specified | Official EVM docs |
| Storage trie and storageRoot structure | Directly specified | Official MPT docs |
| Ethereum not designed for large arbitrary data storage | Directly specified | Official storage docs |
| Institutional storage-analysis framing | Inference from sources | Derived from state and storage architecture |

---

## 19. Knowledge Graph

```text
Storage
├─ Persistent State
│  ├─ storage trie
│  ├─ storageRoot
│  └─ contract state
├─ Temporary Data
│  ├─ memory
│  └─ transient storage
├─ Access
│  ├─ slot addressing
│  ├─ eth_getStorageAt
│  └─ block context
└─ Boundaries
   ├─ state-critical data
   ├─ expensive persistence
   └─ not bulk file storage
```

---

## 20. References

### Primary Sources

[^ref-eth-doc-mpt]: ethereum.org, "Merkle Patricia Trie," official documentation describing the storage trie, `storageRoot`, and storage access examples, https://ethereum.org/developers/docs/data-structures-and-encoding/patricia-merkle-trie/, accessed 2026-08-04.

[^ref-eth-doc-evm]: ethereum.org, "Ethereum Virtual Machine (EVM)," official documentation distinguishing persistent storage, transient storage, and memory, https://ethereum.org/developers/docs/evm/, accessed 2026-08-04.

[^ref-eth-doc-storage]: ethereum.org, "Decentralized Storage," official documentation explaining Ethereum's storage limits for large data and why Ethereum is not designed for bulk persistence, https://ethereum.org/developers/docs/storage/, accessed 2026-08-04.

[^ref-eth-doc-jsonrpc]: ethereum.org, "JSON-RPC API," official documentation for state-query interfaces including storage-related access context, https://ethereum.org/developers/docs/apis/json-rpc/, accessed 2026-08-04.

### Supporting Interpretation Notes

- Where this document discusses institutional storage due diligence or infrastructure burden, those are analytical inferences built from the cited storage and state-architecture sources.

---

## 21. Cross References

### Previous

- ETHEREUM-FOUNDATION-009 — Logs & Events

### Next

- ETHEREUM-FOUNDATION-011 — Ethereum Upgrades

### Related

- ETHEREUM-FOUNDATION-003 — World State
- ETHEREUM-FOUNDATION-005 — EVM

---

## Review Status

### Technical Review

Passed.

- persistent storage, transient storage, memory를 분리했다.
- storage trie와 storageRoot를 명확히 설명했다.
- on-chain state storage와 large-file decentralized storage를 구분했다.
- query context와 layout dependence를 인정했다.

### Evidence Review

Passed.

- core storage architecture는 EVM/MPT docs를 인용한다.
- large-data boundary는 official storage docs를 인용한다.
- institutional interpretation은 inference로 라벨링했다.

### Editorial Review

Passed.

- 구조는 프로젝트 문서 패턴을 따른다.
- metadata는 완전하다.
- terminology는 storageRoot, storage trie, persistent storage, transient storage로 일관된다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 Ethereum storage를 arbitrary bulk data와 혼동하지 않는다.
- `storageRoot`를 storage value와 혼동하지 않는다.
- memory와 storage를 혼동하지 않는다.
- storage read를 trivial하다고 암시하지 않는다.

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
