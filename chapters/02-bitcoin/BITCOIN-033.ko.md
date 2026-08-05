---
knowledge_id: BITCOIN-033
title: Bitcoin Core
subtitle: Reference Implementation, Validation Engine, Policy Node, Wallet Stack, 그리고 Consensus와 Local Operation의 경계
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 155 min
estimated_study: 450 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Bitcoin Core
  - Node Software
  - Validation
parent:
  - Modern Bitcoin
prerequisites:
  - BITCOIN-014
  - BITCOIN-017
  - BITCOIN-021
  - BITCOIN-022
  - POW-013
related_topics:
  - Validation
  - Chainstate
  - Mempool
  - RPC
  - Wallet
  - AssumeUTXO
primary_sources:
  - REF-BTC-CORE-FILES-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-VALIDATIONINTERFACE-001
  - REF-BTC-CORE-NET-PROCESSING-001
  - REF-BTC-CORE-TXMEMPOOL-001
  - REF-BTC-CORE-ASSUMEUTXO-001
  - REF-BTC-CORE-31-RELEASE-001
tags:
  - bitcoin
  - bitcoin-core
  - node
  - validation
  - chainstate
  - mempool
  - rpc
  - wallet
---

# Bitcoin Core
> Modern Bitcoin  
> Research Unit: BITCOIN-033

---

## Research Brief

```yaml
knowledge_id: BITCOIN-033
title: Bitcoin Core
research_question: >
  What is Bitcoin Core as the dominant Bitcoin full-node implementation, how do
  its validation, networking, mempool, wallet, and RPC subsystems interact, and
  why must analysts separate consensus rules from local node policy,
  implementation details, and operational configuration?
document_type: deep-dive
difficulty: L400
prerequisites:
  - BITCOIN-014
  - BITCOIN-017
  - BITCOIN-021
  - BITCOIN-022
  - POW-013
parent: Modern Bitcoin
previous: BITCOIN-032
next: BITCOIN-034
related_topics:
  - Validation
  - Chainstate
  - Mempool
  - Wallet
  - RPC
  - Indexes
  - AssumeUTXO
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Protocol Structure
  - Technical Mechanics
  - Mathematical or Economic Model
  - Security Assumptions
  - Bitcoin Core Implementation
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Full contributor guide for building Bitcoin Core from source
  - Exhaustive RPC catalog
  - Detailed wallet descriptor tutorial
  - Line-by-line source walkthrough of every subsystem
  - Non-Core client comparison survey
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Bitcoin Core가 무엇이고 무엇이 아닌지 설명할 수 있다.
- consensus validation과 local relay/mining policy를 구분할 수 있다.
- Bitcoin Core의 주요 subsystem인 validation, chainstate, mempool, networking, wallet, RPC, index를 식별할 수 있다.
- Bitcoin Core가 "명령으로서의 권위"가 아니라 implementation evidence라는 점을 설명할 수 있다.
- pruning, index, wallet usage 같은 configuration choice가 Bitcoin consensus를 바꾸지 않으면서 node behavior를 어떻게 바꾸는지 설명할 수 있다.
- mempool, wallet, RPC output을 global truth가 아니라 node-local view로 해석할 수 있다.

---

## 2. 핵심 질문

1. Bitcoin Core란 무엇인가?
2. Bitcoin Core가 reference implementation처럼 취급되는 이유는 무엇인가?
3. Bitcoin behavior 중 어느 부분이 consensus rule이고 어느 부분이 local policy인가?
4. validation, chainstate, mempool, networking은 어떻게 상호작용하는가?
5. wallet, RPC, index의 역할은 무엇인가?
6. pruning과 AssumeUTXO는 운영과 분석에 어떤 영향을 주는가?
7. 분석가는 Bitcoin Core node로부터 무엇을 추론할 수 있고, 무엇은 local implementation state에만 남는가?

---

## 3. Executive Summary

Bitcoin Core는 Bitcoin에서 가장 널리 사용되는 full-node implementation이며, block/transaction validation, P2P networking, mempool behavior, wallet management, RPC interface를 위한 지배적인 operational reference 역할을 한다.[^ref-btc-core-files][^ref-btc-core-validation]

그러나 그것이 곧 Bitcoin protocol 자체는 아니다. consensus는 특정 repository에 대한 사회적 복종으로 정의되는 것이 아니라, node가 block과 transaction에 대해 실제로 강제하는 rule로 정의된다. 다만 Bitcoin Core는 많은 operator, exchange, researcher, infrastructure provider가 사용하는 primary implementation이기 때문에, 현대 Bitcoin이 실제로 어떻게 동작하는지 이해하기 위한 핵심 1차 자료다.[^ref-btc-core-validation][^ref-btc-core-net-processing]

Bitcoin Core는 서로 다른 역할을 한 구현체 안에 결합한다.

- full-node validation engine
- P2P relay participant
- mempool 및 mining-policy engine
- wallet 및 key-management software
- RPC 및 data-serving surface
- optional indexing 및 operational tooling layer[^ref-btc-core-files][^ref-btc-core-txmempool]

분석에서 가장 중요한 규율은 분리다. Bitcoin Core node는 software version, configuration, peer set, index, pruning mode, wallet state, policy parameter의 영향을 받는 local viewpoint를 보여준다. 분석가는 이 local viewpoint를 network-wide truth와 혼동해서는 안 된다.[^ref-btc-core-net-processing][^ref-btc-core-31-release]

---

## 4. Protocol Structure

### Bitcoin Core as a Layered System

상위 수준에서 Bitcoin Core는 다음과 같은 stack으로 이해할 수 있다.

```text
user / operator
-> CLI, GUI, RPC, config
-> wallet and indexes
-> mempool and policy
-> validation and chainstate
-> p2p networking
-> disk, database, and data directory
```

### Core Roles

| Role | What Bitcoin Core Does |
|---|---|
| Consensus validation | block, transaction, script, subsidy constraint, context rule를 검증 |
| Chainstate maintenance | active UTXO set과 block index를 갱신/조회 |
| Relay node | block, header, transaction, peer metadata를 교환 |
| Policy node | mempool, standardness, anti-DoS, mining-template rule을 local하게 적용 |
| Wallet stack | key, descriptor, balance, signing을 관리 |
| RPC server | operational/programmatic interface를 노출 |
| Indexing / storage | block file, index, chain metadata, optional derived view를 저장 |

### What Bitcoin Core Is Not

Bitcoin Core는 다음이 아니다.

- central coordinator
- 유일하게 유효한 Bitcoin implementation
- 모든 local policy choice가 universal하다는 보장
- global network state의 완벽한 거울

---

## 5. Major Components

### Binaries and Operator Surface

Bitcoin Core의 source-tree overview는 `bitcoind`, `bitcoin-qt`, `bitcoin-cli`, `bitcoin-wallet` 같은 주요 binary를 문서화한다.[^ref-btc-core-files]

운영 관점에서는 다음과 같이 이해하면 된다.

- `bitcoind`는 대부분의 기관과 infrastructure operator가 실행하는 daemon이다.
- `bitcoin-cli`는 RPC interaction의 일반적인 control surface다.
- `bitcoin-qt`는 node와 desktop GUI를 함께 제공한다.
- `bitcoin-wallet`은 wallet-file management task를 지원한다.[^ref-btc-core-files]

### Validation and Chainstate

`validation.cpp`와 `validation.h`는 block acceptance, active chain maintenance, block connect/disconnect, chainstate transition을 담당하는 핵심 validation path다.[^ref-btc-core-validation][^ref-btc-core-validation-h]

### Networking

`net_processing`은 relay와 peer-state의 핵심이다. message-driven synchronization, object announcement, transaction request, peer-specific logic, 각종 anti-abuse decision이 여기서 처리된다.[^ref-btc-core-net-processing]

### Mempool and Policy

`CTxMemPool`은 node의 local candidate transaction pool과, admission/eviction/package reasoning/block construction에 필요한 metadata를 유지한다.[^ref-btc-core-txmempool]

### Wallet and RPC

Bitcoin Core는 wallet과 RPC surface도 함께 제공한다. 이것들은 Bitcoin consensus를 정의하지는 않지만, operator와 application, institution이 시스템과 상호작용하는 실제 방식에는 큰 영향을 준다.[^ref-btc-core-files]

---

## 6. Why Bitcoin Core Matters

### Dominant Implementation Evidence

Bitcoin Core가 중요한 이유는 생태계의 상당 부분이 실제로 그것을 실행하거나, 그것의 동작을 전제로 삼기 때문이다. 따라서 Core는 다음을 이해하는 primary implementation source가 된다.

- current relay behavior
- validation pathway
- mempool structure
- wallet default
- operational convention

### But Not Protocol Sovereignty

protocol이 유효한 것은 Core가 그렇게 선언해서가 아니다. 오히려 Core가 계속 중요할 수 있는 이유는, 다른 node도 받아들일 consensus-compatible result를 계속 만들어내야 하기 때문이다. 어떤 rule change가 한 implementation에 들어갔다고 해서, 경제적으로 그리고 운영적으로 네트워크가 수렴하지 않으면 그 변화가 곧 Bitcoin이 되는 것은 아니다.

### Practical Consequence

분석가가 "Bitcoin does X"라고 말할 때, 실제 의미는 종종 다음 셋 중 하나다.

1. consensus rule이 X를 요구한다.
2. Bitcoin Core가 현재 X를 구현한다.
3. 많은 operator가 Bitcoin Core를 특정 설정으로 실행해 X가 흔하다.

이 셋은 서로 바꿔 쓸 수 없다.

---

## 7. Consensus vs Policy

### Consensus

consensus는 어떤 block 또는 transaction이 Bitcoin에서 유효한지 여부를 답한다.

예:

- 작업증명(Proof of Work, PoW) validity
- block-weight limit
- script correctness
- subsidy 및 amount-range constraint
- 코인베이스 트랜잭션 maturity
- witness commitment rule[^ref-btc-core-validation]

### Policy

policy는 node가 consensus-valid할 수 있는 object를 relay, store, request, mine할지 여부를 답한다.

예:

- mempool feerate threshold
- standardness filter
- orphan limit
- request scheduling
- eviction behavior
- local block template construction[^ref-btc-core-net-processing][^ref-btc-core-txmempool][^ref-btc-core-31-release]

### Why the Distinction Matters

어떤 transaction은 consensus-valid하면서도 많은 mempool에 없을 수 있다. 반대로 어떤 transaction은 한 mempool에는 있지만 결국 채굴되지 않을 수 있다. policy는 local하고 operational하며, consensus는 global하고 final하다.

---

## 8. Validation Path and Chainstate

### Active-Chain Maintenance

Bitcoin Core는 block index를 추적하면서, 새로운 header와 block이 검증됨에 따라 active-chain state를 갱신한다. `validation.cpp`에는 normal extension과 reorganization 처리 동안 chainstate를 전진 또는 후진시키는 connect/disconnect path가 포함돼 있다.[^ref-btc-core-validation][^ref-btc-core-validation-h]

### UTXO-Set Transition

각 accepted block은 coins view에 state transition을 적용한다. input은 기존 UTXO를 소비하고, output은 새로운 UTXO를 생성하며, resulting state는 이후 transaction과 block validation의 기준이 된다.[^ref-btc-core-validation]

### Multiple Chainstates

Bitcoin Core의 AssumeUTXO design documentation은 현대적인 synchronization에서 multiple chainstate가 사용될 수 있음을 설명한다. snapshot-based fast start를 위한 background-validation path와 fully validated chainstate가 공존할 수 있다.[^ref-btc-core-assumeutxo]

이것은 consensus split이 아니라 implementation architecture 포인트다. node는 여전히 동일한 valid chainstate에 수렴하는 것을 목표로 하며, 바뀌는 것은 synchronization workflow이지 Bitcoin rule 자체가 아니다.

---

## 9. Networking and Relay

### Peer Message Processing

`net_processing`은 node가 incoming/outgoing peer event에 어떻게 반응할지 조정한다. synchronization progress, inventory announcement, transaction/block request, peer-specific state tracking이 여기에 포함된다.[^ref-btc-core-net-processing]

### Object Requests and Anti-Abuse Logic

Bitcoin Core는 모든 announced object를 모든 peer에게서 순진하게 요청하지 않는다. request scheduling, announcement tracking, anti-stall logic이 존재하는 이유는 bandwidth, latency, adversarial behavior가 실제 운영에 중요하기 때문이다.[^ref-btc-core-net-processing]

### Local Network View

node의 peer set은 그 node가 무엇을 언제 보게 되는지 결정한다. 이는 다음에 영향을 준다.

- first-seen transaction timing
- mempool composition
- stale 또는 delayed block receipt
- topology bias
- eclipse resilience

따라서 networking은 단순 transport plumbing이 아니라 data quality의 일부다.

---

## 10. Mempool and Mining-Adjacent Policy

### What the Mempool Is

Bitcoin Core의 mempool은 node가 메모리에 유지하고 relay 또는 block construction 후보로 고려할 의사가 있는 unconfirmed transaction의 local set이다. 이것은 network-wide canonical pool이 아니다.[^ref-btc-core-txmempool]

### Cluster-Aware Modern Behavior

Bitcoin Core 31.0 release note는 cluster mempool redesign을 설명한다. transaction ordering, eviction, replacement reasoning이 chunk와 feerate-diagram 개념을 통해 더 package-aware해졌다.[^ref-btc-core-31-release]

이것이 분석적으로 중요한 이유는 mempool observation이 implementation version의 영향을 받기 때문이다. 두 node가 비슷해 보여도, 하나가 더 새로운 policy engine을 실행하면 결과가 달라질 수 있다.

### Mining Boundary

Bitcoin Core는 block template를 만들고 mining-relevant policy를 적용할 수 있지만, template construction은 consensus와 다르다. miner는 어떤 node가 mempool에 보관하지 않았더라도 valid transaction을 block에 넣을 수 있다.

---

## 11. Wallet, RPC, and Data Interfaces

### Wallet as an Optional Layer

Bitcoin Core의 wallet은 distribution의 일부지만, validating node를 운영하는 데 wallet 사용이 필수는 아니다. validation infrastructure와 custody/signing environment를 분리하려는 기관에게 이 점은 중요하다.[^ref-btc-core-files]

### RPC as the Operational Contract

많은 production system에서 Bitcoin Core와의 실질적 interface는 RPC다. application은 내부 validation code에 직접 링크하지 않고, RPC result, notification, log를 통해 Core를 질의하는 경우가 많다.

### Node-Local Semantics

RPC output은 local node state에 의해 형성된다.

- pruning은 historical block data availability를 줄일 수 있다.
- wallet enablement는 balance/signing function을 바꾼다.
- index는 queryability를 바꾼다.
- peer state는 node가 현재 무엇을 아는지 바꾼다.
- mempool policy는 pending-transaction visibility를 바꾼다.

따라서 RPC answer는 그 시점 그 node의 local answer다.

---

## 12. Pruning, Indexes, and Storage Modes

### Pruning

pruned node는 normal operation에 더 이상 필요하지 않은 old block file을 삭제하면서도 chain을 fully validate할 수 있다. 이는 archival capability를 바꾸는 것이지 consensus validity를 바꾸는 것이 아니다.

### Indexes

index는 추가적인 derived lookup surface를 제공한다. queryability와 analysis ergonomics는 좋아지지만, Bitcoin consensus 참여에 필수는 아니다.

### Data Availability vs Validation Ability

이 구분은 중요하다.

```text
can validate != can serve every historical object quickly
can answer an RPC query != can prove a network-wide fact
```

운영상의 storage choice는 Core node가 analyst와 downstream system에 어떤 data를 반환할 수 있는지 결정한다.

---

## 13. AssumeUTXO and Modern Synchronization

### Goal

AssumeUTXO는 node가 UTXO snapshot에서 시작하면서 historical chain을 background에서 검증할 수 있게 하려는 synchronization architecture다.[^ref-btc-core-assumeutxo]

### Why It Exists

이 설계는 긴 initial synchronization 시간과, eventual full validation goal을 포기하지 않으면서도 새 node를 더 빨리 usable하게 만들고자 하는 실용적 요구에 대응한다.

### Analytical Importance

이 기능은 더 넓은 포인트를 강화한다. operational state와 consensus state는 관련되어 있지만 동일하지 않다. synchronization phase와 configuration에 따라 node는 모든 historical validation step이 끝나기 전에도 operationally useful할 수 있다.[^ref-btc-core-assumeutxo]

기관은 자신이 관찰하는 Core node가 어떤 mode에 있는지 추적해야 한다.

---

## 14. Technical Mechanics

### Simplified Subsystem Flow

```text
peer announces tx or block
-> net_processing evaluates request/relay path
-> object reaches validation path
-> consensus and contextual checks run
-> chainstate or mempool updates
-> validationinterface subscribers notified
-> RPC / wallet / indexes observe resulting state
```

### Validation Notifications

`CValidationInterface`는 validation outcome이 subscriber에게 노출되는 경계를 표시한다. 이는 wallet, index, 기타 component가 validation 자체를 재정의하지 않고도 chain 또는 mempool change에 반응할 수 있게 해준다.[^ref-btc-core-validationinterface]

### Implementation Stability vs Evolution

Core의 내부 decomposition은 release별로 바뀔 수 있지만, 연구에 필요한 핵심 architecture boundary는 충분히 안정적이다.

- networking은 받아서 조정한다.
- validation은 판정한다.
- chainstate는 consensus result를 지속화한다.
- mempool은 local pending state를 저장한다.
- wallet과 RPC는 node-local interface를 제시한다.

---

## 15. Security Assumptions and Failure Modes

### Software Correctness Matters

Bitcoin Core는 security-critical software다. validation bug, wallet bug, persistence bug, relay bug는 네트워크와 operator에 심각한 결과를 줄 수 있다.

### Configuration Risk

운영 misconfiguration은 오해를 부르는 local view 또는 저하된 security를 만들 수 있다.

- 부족한 peer diversity
- 예상치 못한 pruning
- validation host에서의 wallet 노출
- 과도하게 열린 RPC 접근
- index coverage에 대한 잘못된 가정

### Version and Deployment Risk

특히 mempool admission, replacement, relay 같은 policy-heavy 영역에서는 version에 따라 behavior가 달라질 수 있다. 기관은 software version을 pin하고 upgrade assumption을 문서화해야 한다.[^ref-btc-core-31-release]

### Local View Risk

단일 Core node는 다음 상태일 수 있다.

- partially synchronized
- eclipsed 또는 topology-biased
- policy-divergent
- index 없음
- wallet disabled

이 상태들이 Bitcoin consensus를 다시 쓰지는 않지만, operator가 보는 것은 바꾼다.

---

## 16. Mathematical or Economic Model

### Validation Cost Intuition

node 작업량은 다음처럼 분해해 생각할 수 있다.

```text
total operational load
= block validation load
+ transaction relay load
+ mempool maintenance load
+ disk and index maintenance load
+ wallet / RPC query load
```

이것은 protocol formula가 아니라, node resource demand에 대한 engineering decomposition이다.

### Local Mempool Non-Identity

node `i`가 보는 mempool을 `M_i`라고 하면, 실제 운영에서는 다음이 흔하다.

```text
M_a != M_b
```

peer topology, fee filter, orphan state, software version, policy setting이 다르기 때문이다.

### Operational Economics

기관이 Core deployment profile을 선택할 때 암묵적으로 다음을 trade-off한다.

- storage vs queryability
- sync speed vs historical completeness
- wallet convenience vs attack surface
- one-node simplicity vs multi-node observability

이것은 consensus economics가 아니라 operator economics다.

---

## 17. Bitcoin Core Implementation

### Source-Tree Orientation

Bitcoin Core의 `doc/files.md`는 source-tree organization과 주요 user-facing binary를 설명한다. repository를 평평한 코드베이스처럼 오해하지 않고 subsystem 위치를 파악할 수 있게 해주기 때문에, implementation orientation의 출발점으로 적절하다.[^ref-btc-core-files]

### `validation.cpp` and `validation.h`

이 파일들은 main chain-validation 및 active-state coordination surface다. block acceptance, reorg-sensitive state change, chainstate handling, 많은 contextual validation path를 포함한다.[^ref-btc-core-validation][^ref-btc-core-validation-h]

### `net_processing`

이 module은 peer-driven object handling, announcement logic, synchronization behavior, anti-abuse response를 조정한다.[^ref-btc-core-net-processing]

### `txmempool.h`

`CTxMemPool`과 관련 structure는 node-local pending-transaction set과 relay/mining-adjacent policy에 필요한 metadata를 표현한다.[^ref-btc-core-txmempool]

### `validationinterface.h`

`CValidationInterface`는 validation-relevant event가 발생했을 때 subscriber에게 notification을 노출한다. 덕분에 validation engine과 downstream consumer 사이의 분리가 유지된다.[^ref-btc-core-validationinterface]

### `doc/design/assumeutxo.md`

이 design note는 추상적인 미래 개념이 아니라 current Core architecture feature를 문서화한다는 점에서 중요하다. operational sync convenience와 eventual validation completeness를 Core가 어떻게 분리하는지 보여준다.[^ref-btc-core-assumeutxo]

---

## 18. On-Chain Implications

### What On-Chain Data Reflects

blockchain은 accepted consensus outcome을 기록한다.

- confirmed transaction
- confirmed block
- eventually observed reorg history
- accepted chain history가 함의하는 spent/unspent output

### What It Does Not Reflect

blockchain은 대부분의 local Bitcoin Core state를 기록하지 않는다.

- mempool content
- orphan pool content
- peer request choice
- wallet label
- RPC client
- pruning configuration
- background sync phase

### Consequence for Analysts

chain analysis만으로는 Core node의 전체 behavior를 재구성할 수 없다. node behavior의 큰 부분은 off-chain implementation state다.

---

## 19. Institutional Thinking

기관은 Bitcoin Core를 infrastructure software이자 measurement instrument로 동시에 다뤄야 한다.

### Practical Implications

- validation node와 analytics node는 서로 다른 configuration이 필요할 수 있다.
- 강한 mempool/propagation claim을 하려면 하나의 Core node로는 부족하다.
- wallet-enabled Core instance는 특별한 이유가 없으면 넓은 노출면과 분리하는 편이 안전하다.
- version upgrade는 단순 유지보수 이벤트가 아니라 data-model change로도 분석해야 한다.
- RPC consumer는 analytical output과 함께 node version, pruning mode, wallet mode, index coverage를 기록해야 한다.

### Better Research Posture

Bitcoin research를 Core 위에 구축할 때는 다음을 식별해야 한다.

- 어떤 주장이 consensus claim인지
- 어떤 주장이 Core implementation claim인지
- 어떤 주장이 local configuration claim인지
- 어떤 주장이 단지 한 node의 observation window로부터 나온 inference인지

---

## 20. Common Misinterpretations

### "Bitcoin Core is Bitcoin"

너무 강한 표현이다. Bitcoin Core는 지배적 implementation이지, 형이상학적 의미에서 protocol 그 자체는 아니다.

### "If Core relays it, Bitcoin accepts it"

틀렸다. relay policy와 consensus validity는 다르다.

### "If my Core node shows it, the network shows it"

틀렸다. node는 topology-dependent하고 policy-dependent한 local view를 노출한다.

### "Pruned nodes are not real full nodes"

consensus 의미에서는 틀렸다. pruned node도 fully validate할 수 있으며, 단지 모든 historical block data를 archival serving용으로 보관하지 않을 뿐이다.

### "Wallet and node are the same thing"

틀렸다. Bitcoin Core는 기관의 custody/signing system으로 쓰이지 않더라도 validating node로 실행될 수 있다.

### "RPC output is objective ground truth"

틀렸다. 그것은 특정 설정을 가진 한 node가 특정 시점에 직렬화해 보여준 view일 뿐이다.

---

## 21. Research Questions

1. fee spike 구간에서 Bitcoin Core version별 mempool과 relay observation은 얼마나 다르게 나타나는가?
2. 기관은 pruned node와 archival node에서 나온 analytical claim을 어떻게 분류해야 하는가?
3. node-local RPC output에 대한 오해를 가장 줄여주는 operational control은 무엇인가?
4. AssumeUTXO는 시간이 지나면서 기관의 node-deployment strategy를 얼마나 바꾸는가?
5. exported analytic과 함께 항상 캡처해야 하는 Core configuration field는 무엇인가?

---

## 22. Practical Exercises

### Exercise 1

announced transaction이 mempool admission 또는 rejection에 이르기까지의 경로를, network handling, policy check, consensus-relevant validation을 구분해 매핑하라.

### Exercise 2

서로 다른 pruning/index setting을 가진 두 node의 output을 비교하고, 각 node가 어떤 질문에 신뢰성 있게 답할 수 있는지 식별하라.

### Exercise 3

consensus-valid transaction이 왜 어떤 node의 mempool에는 없고 다른 node의 mempool에는 있을 수 있는지 설명하라.

### Exercise 4

`CValidationInterface`가 downstream component를 validator로 만들지 않으면서도 validation event를 관찰하게 해주는 이유를 설명하라.

---

## 23. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Main binaries and source-tree organization | Directly specified | `doc/files.md` |
| Validation, chainstate, and notification boundaries | Directly specified | `validation.cpp`, `validation.h`, `validationinterface.h` |
| Peer message handling and relay coordination | Directly specified | `net_processing.cpp` |
| Mempool local-state behavior | Directly specified | `txmempool.h` and release notes |
| AssumeUTXO architecture | Directly specified | Core design documentation |
| Measurement-bias and local-view implications | Inference from sources | Derived from node-local policy, peer topology, and storage/config modes |

---

## 24. Knowledge Graph

```text
Bitcoin Core
├─ Operator Surface
│  ├─ bitcoind
│  ├─ bitcoin-cli
│  ├─ bitcoin-qt
│  └─ bitcoin-wallet
├─ Consensus Engine
│  ├─ validation.cpp
│  ├─ validation.h
│  ├─ chainstate
│  └─ UTXO updates
├─ Network Layer
│  ├─ net_processing
│  ├─ peer state
│  ├─ requests
│  └─ relay behavior
├─ Local Policy
│  ├─ mempool
│  ├─ standardness
│  ├─ eviction
│  └─ mining templates
├─ Interfaces
│  ├─ RPC
│  ├─ wallet
│  ├─ indexes
│  └─ validationinterface
└─ Operational Modes
   ├─ pruning
   ├─ archival
   ├─ AssumeUTXO
   └─ version-specific behavior
```

---

## 25. 참고문헌

### Primary Sources

[^ref-btc-core-files]: Bitcoin Core Contributors, `doc/files.md`, source-tree and binary overview, Bitcoin Core master branch, https://github.com/bitcoin/bitcoin/blob/master/doc/files.md, accessed 2026-08-04.

[^ref-btc-core-validation]: Bitcoin Core Contributors, `src/validation.cpp`, block validation, chainstate coordination, and active-chain processing, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/validation_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-validation-h]: Bitcoin Core Contributors, `src/validation.h`, validation interfaces, chainstate-related declarations, and block connection/disconnection surfaces, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/validation_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-validationinterface]: Bitcoin Core Contributors, `src/validationinterface.h`, `CValidationInterface` notification interface, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/validationinterface_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-net-processing]: Bitcoin Core Contributors, `src/net_processing.cpp`, peer message processing and synchronization logic, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/net__processing_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-txmempool]: Bitcoin Core Contributors, `src/txmempool.h`, `CTxMemPool` structures and node-local transaction pool metadata, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/txmempool_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-assumeutxo]: Bitcoin Core Contributors, `doc/design/assumeutxo.md`, snapshot-based chainstate synchronization design, Bitcoin Core master branch, https://github.com/bitcoin/bitcoin/blob/master/doc/design/assumeutxo.md, accessed 2026-08-04.

[^ref-btc-core-31-release]: Bitcoin Core Contributors, Bitcoin Core 31.0 release notes, mempool and policy-related implementation changes, https://bitcoincore.org/en/releases/31.0/, accessed 2026-08-04.

### Supporting Interpretation Notes

- Where this document discusses measurement bias, node-local truth, or institutional observability limits, those claims are analytical inferences from Bitcoin Core's documented architecture and configuration-dependent behavior rather than explicit consensus claims.

---

## 26. 교차 참조

### Previous

- BITCOIN-032 — Lightning Network

### Next

- BITCOIN-034

### Related

- BITCOIN-014 — UTXO Model
- BITCOIN-017 — Mempool
- BITCOIN-021 — Blocks and Block Headers
- BITCOIN-022 — Nodes and Network Propagation
- POW-013 — Bitcoin Core Proof-of-Work Validation

---

## Review Status

### Technical Review

Passed.

- Bitcoin Core의 역할을 validation, relay, mempool policy, wallet, RPC, storage/indexing으로 분리했다.
- consensus rule과 local policy/operational configuration을 명시적으로 구분했다.
- AssumeUTXO를 consensus change가 아니라 synchronization architecture feature로 설명했다.
- mempool과 RPC output을 protocol truth가 아닌 node-local state로 다뤘다.

### Evidence Review

Passed.

- source-tree, binary, subsystem orientation은 `doc/files.md`에 근거한다.
- validation, networking, mempool, notification claim은 Bitcoin Core source documentation에 근거한다.
- version-sensitive mempool claim은 31.0 release note와 연결했다.
- local-view bias에 대한 analytical claim은 inference로 라벨링했다.

### Editorial Review

Passed.

- 구조는 프로젝트 deep-dive format을 따른다.
- metadata는 완전하다.
- terminology는 consensus, policy, chainstate, mempool, pruning, RPC, wallet, AssumeUTXO로 일관된다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 Bitcoin Core를 Bitcoin 그 자체와 동일시하지 않는다.
- local relay behavior가 consensus를 정의한다고 암시하지 않는다.
- 한 node의 mempool이나 RPC output을 global truth로 취급하지 않는다.
- pruning을 validation capability 상실로 설명하지 않는다.
- AssumeUTXO를 eventual validation 우회로 설명하지 않는다.

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
