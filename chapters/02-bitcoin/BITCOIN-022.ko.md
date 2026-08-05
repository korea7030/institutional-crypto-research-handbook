---
knowledge_id: BITCOIN-022
title: Nodes and Network Propagation
subtitle: 피어 발견, 핸드셰이크, inventory relay, headers-first 동기화, transaction relay, 그리고 전파 보안
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 110 min
estimated_study: 320 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Networking
  - Nodes
  - Propagation
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-017
  - BITCOIN-021
  - POW-011
  - POW-013
related_topics:
  - P2P Network
  - Headers-First Sync
  - Transaction Relay
  - Compact Blocks
  - SPV
  - AddrMan
  - Eclipse Attacks
  - Mempool Policy
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-P2P-001
  - REF-BIP-0037
  - REF-BIP-0130
  - REF-BIP-0133
  - REF-BIP-0152
  - REF-BIP-0155
  - REF-BIP-0159
  - REF-BIP-0324
  - REF-BIP-0339
  - REF-BTC-CORE-NET-001
  - REF-BTC-CORE-PROTOCOL-001
  - REF-BTC-CORE-NET-PROCESSING-001
  - REF-BTC-CORE-TXREQUEST-001
  - REF-BTC-CORE-TXORPHANAGE-001
  - REF-BTC-CORE-VALIDATIONINTERFACE-001
tags:
  - bitcoin
  - internals
  - networking
  - nodes
  - p2p
  - propagation
  - relay
  - headers-first-sync
---

# Nodes and Network Propagation
> Bitcoin Internals  
> Research Unit: BITCOIN-022

---

## Research Brief

```yaml
knowledge_id: BITCOIN-022
title: Nodes and Network Propagation
research_question: >
  How do Bitcoin nodes discover peers, establish sessions, exchange addresses
  and inventories, synchronize headers and blocks, relay transactions, and
  defend propagation quality without confusing local relay policy with global
  consensus?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-017
  - BITCOIN-021
  - POW-011
  - POW-013
parent: Bitcoin Internals
previous: BITCOIN-021
next: BITCOIN-023
related_topics:
  - P2P Protocol
  - Transaction Relay
  - Block Relay
  - Headers-First Synchronization
  - Mempool Policy
  - Eclipse Resistance
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
  - Full transport cryptography implementation details
  - Lightning Network routing
  - Compact block reconstruction internals beyond analyst-relevant behavior
  - Tor operational guidance
  - Non-Bitcoin P2P network comparisons
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- node discovery, connection setup, synchronization, relay를 구분할 수 있다.
- `version`/`verack` handshake와 service bit의 역할을 설명할 수 있다.
- `addr`와 `addrv2`가 peer discovery를 어떻게 지원하는지 설명할 수 있다.
- `inv`, `getdata`, `headers`, `getheaders`, `block`가 어떻게 상호작용하는지 설명할 수 있다.
- block relay와 transaction relay를 구분할 수 있다.
- headers-first synchronization이 대역폭과 신뢰 가정을 어떻게 줄이는지 설명할 수 있다.
- `sendheaders`, compact block, `wtxid` relay가 propagation을 어떻게 개선하는지 설명할 수 있다.
- consensus validity와 local mempool relay policy를 구분할 수 있다.
- propagation topology가 latency, stale risk, privacy에 영향을 주는 이유를 설명할 수 있다.
- peer state, relay, orphan handling을 구현하는 Bitcoin Core module을 식별할 수 있다.

---

## 2. 핵심 질문

1. Bitcoin 네트워크 모델에는 어떤 종류의 node가 있는가?
2. node는 peer를 어떻게 발견하고 연결하는가?
3. handshake 동안 어떤 정보가 교환되는가?
4. 새로운 block은 어떻게 announce되는가?
5. transaction은 어떻게 announce되고 request되는가?
6. `inv` 기반 relay와 `headers` 기반 relay의 차이는 무엇인가?
7. compact block은 어떤 문제를 해결하는가?
8. SegWit 이후 `wtxid` relay가 왜 중요한가?
9. relay policy와 consensus의 차이는 무엇인가?
10. orphan transaction은 propagation 중 어떻게 생기는가?
11. propagation layer의 주요 공격 표면은 무엇인가?
12. 온체인 분석가는 propagation behavior에서 무엇을 추론할 수 있고, 무엇은 추론할 수 없는가?

---

## 3. Executive Summary

Bitcoin은 peer-to-peer node 네트워크다. node는 서로를 발견하고, capability를 협상하고, block header와 block을 동기화하고, transaction을 relay한다. 백서의 네트워크 섹션은 transaction을 모든 node에 broadcast하고, 이를 block에 모으고, 수락된 block을 다시 전파해 다음 block 위에서 작업을 계속하는 기본 흐름을 설명한다.[^ref-btc-wp]

현대 Bitcoin에서 propagation은 백서의 짧은 설명보다 훨씬 구조화되어 있다. node는 `version` message를 교환하고 `verack`로 확인하며, `addr` 또는 `addrv2`로 도달 가능한 peer를 advertise하고, `getheaders`와 `headers`로 chain knowledge를 동기화하고, `inv`나 `headers` 같은 announcement를 받은 뒤 `getdata`로 필요한 object를 요청한다.[^ref-btc-dev-p2p]

block propagation과 transaction propagation은 운영 목표가 다르다. block propagation은 latency-sensitive하다. 전파가 느리면 miner의 stale-block risk가 커지고, 네트워크 전체가 most-work tip으로 수렴하는 속도도 떨어진다. transaction propagation은 policy-sensitive하다. node는 transaction이 consensus-invalid가 아니더라도 local mempool 또는 anti-abuse policy를 통과하지 못하면 relay를 거부하거나 지연하거나 아예 하지 않을 수 있다.[^ref-btc-core-net-processing] [^ref-btc-core-txrequest]

현대 relay는 여러 관심사를 분리한다.

- Discovery: address relay와 address management
- Session setup: handshake, service negotiation, relay preference
- Chain sync: headers-first synchronization과 block download
- Unconfirmed transaction relay: announcement, request, orphan handling, fee filter, relay suppression
- Efficiency and robustness: `sendheaders`, compact block, `wtxid` relay, selective peer behavior[^ref-bip-0130] [^ref-bip-0152] [^ref-bip-0339]

분석가에게 propagation은 중요하다. timing, topology, node policy가 transaction이 언제 보이는지, block이 언제 지배적이 되는지, 특정 vantage point가 실제로 얼마나 많은 mempool data를 관측하는지를 결정하기 때문이다. 하나의 node view는 결코 네트워크 전체 view가 아니다.

---

## 4. 프로토콜 구조

### Node 역할

Bitcoin에는 단일한 authoritative node type이 없다. peer는 보유 데이터, 제공 서비스, 적용 정책에 따라 다르다.

분석적으로 유용한 구분은 다음과 같다.

- Full validating node: 합의 규칙을 검증하고 새 block과 transaction을 검증할 수 있을 만큼의 chainstate를 유지
- Archival full node: validating node이면서 pruning하지 않고 historical block data를 보존
- Pruned node: validating node이지만 active chain 검증에 필요한 최소 데이터만 남기고 오래된 block file은 삭제 가능
- Lightweight client: full chain을 직접 검증하지 않고 header와 proof, filtered data 중심으로 동작
- Mining node / pool infrastructure: block reception과 announcement latency에 특히 민감한 peer

`version` message의 service bit는 full network history 제공, limited recent blocks 제공, 새로운 transport/relay feature 지원 같은 capability를 advertise하는 데 사용된다.[^ref-btc-dev-p2p] [^ref-bip-0159] [^ref-bip-0324]

### Connection Lifecycle

상위 수준에서 peer interaction은 다음 순서를 따른다.

```text
peer discovery
-> outbound connection attempt
-> version exchange
-> verack exchange
-> capability-dependent messages
-> headers / address / inventory / data exchange
-> steady-state relay
```

`version` message는 protocol version, service flag, timestamp, address, nonce, user agent, start height, 그리고 BIP37이 정의한 transaction relay preference flag를 전달한다.[^ref-btc-dev-p2p] [^ref-bip-0037]

### Message Family

분석적으로 Bitcoin P2P message는 네 가지 family로 나눌 수 있다.

| Family | Example Messages | Primary Purpose |
|---|---|---|
| Session control | `version`, `verack`, `ping`, `pong` | 사용 가능한 연결 수립과 liveness 추적 |
| Peer discovery | `addr`, `addrv2`, `getaddr`, `sendaddrv2` | 도달 가능한 peer와 transport format 파악 |
| Chain synchronization | `getheaders`, `headers`, `getblocks`, `block`, `cmpctblock` | chain state 파악과 missing block download |
| Data relay | `inv`, `getdata`, `tx`, `notfound`, `feefilter` | transaction/block announce 및 fetch |

### Inventory-Based Relay

많은 relay 흐름은 inventory 기반이다. peer는 `inv`로 object hash를 announce한다. 받는 쪽은 local state와 비교하고, 모르는 object만 `getdata`로 요청한다. 실제 payload는 `tx`, `block`, `cmpctblock` 같은 message로 도착한다.[^ref-btc-dev-p2p]

이 구조는 모든 peer에게 full object를 무조건 보내지 않으므로 중복 전송을 줄인다.

### Headers-First Synchronization

headers-first synchronization은 "먼저 full block을 많이 다운로드"하는 방식에서 "먼저 header를 다운로드하고 검증한 뒤 유망한 branch의 full block만 가져오는 방식"으로 sync를 바꿨다. `getheaders`는 한 번에 최대 2,000개의 header를 요청하며, 각 `headers` entry는 80-byte block header와 zero transaction count byte를 가진다.[^ref-btc-dev-p2p]

따라서 node는 비싼 full block body를 받기 전에, 적은 대역폭으로 proof of work, linkage, chainwork progression을 평가할 수 있다.

---

## 5. Peer Discovery와 Session Establishment

### Address Relay

inbound connection을 받는 node는 `addr` 또는 최신 format 협상 시 `addrv2`로 peer address를 advertise한다. `getaddr`는 추가 peer address를 요청한다. 이 message들은 consensus가 아니라 bootstrapping과 network graph refresh를 지원한다.[^ref-btc-dev-p2p] [^ref-bip-0155]

address information은 advisory일 뿐 authoritative하지 않다. 인증되지 않으며, stale하거나 불완전하거나 악의적일 수 있다.

### Address Management

Bitcoin Core는 관측한 모든 address를 동일하게 취급하지 않고, 확률적 address manager인 `AddrMan`으로 known peer를 추적하고 점수화한다.[^ref-btc-core-net]

이는 peer selection이 다음에 영향을 주기 때문에 중요하다.

- eclipse risk 노출
- geographic/topological diversity
- propagation latency
- future outbound connection의 신뢰성

### Handshake Semantics

`version`/`verack` 교환은 일반 traffic 처리 전에 session을 수립한다. `version` message는 다음을 signal할 수 있다.

- protocol compatibility
- service capability
- user agent를 통한 peer software identity
- reported best height
- 기본적으로 transaction announcement를 원하는지 여부[^ref-btc-dev-p2p] [^ref-bip-0037]

이 handshake는 capability discovery이지 trust establishment가 아니다. remote peer는 service, height, identity를 거짓으로 말할 수 있다.

### Liveness와 Flow Control

`ping`/`pong`는 connection health와 round-trip time을 측정하는 데 도움을 준다. 이는 합의보다 peer eviction, request timing, relay quality에 더 큰 영향을 준다.[^ref-btc-dev-p2p]

---

## 6. Block과 Header Propagation

### New Block Announcement Path

역사적으로 새로운 block은 자주 `inv`로 announce되었고, 받은 쪽은 `getdata`로 full block을 요청했다. BIP130은 `sendheaders`를 도입해 peer가 새 block에 대해 direct `headers` announcement를 선호할 수 있게 했다.[^ref-bip-0130] [^ref-btc-dev-p2p]

차이는 중요하다.

- `inv`는 hash만 알리고, header를 알기 위해 추가 round trip이 필요하다.
- `headers`는 새 header를 직접 제공하므로 즉시 proof-of-work와 parent-link check가 가능하다.

block race에서는 round trip 하나도 중요하다.

### Headers-First Chain Extension

node가 새 header를 받으면 다음을 할 수 있다.

1. 각 header의 format과 proof of work 검증
2. 알려진 parent에 연결되는지 또는 candidate branch를 연장하는지 검증
3. cumulative chainwork 추정 갱신
4. full block을 다운로드할 가치가 있는 branch인지 결정

그래서 header propagation이 네트워크의 first convergence layer이고, full block download는 second layer다.

### Full Block Fetch

유망한 header가 수용되면 node는 `getdata`로 해당 block body를 요청한다. 이후 full block processing은 transport에서 validation logic으로 넘어간다: syntax check, Merkle check, witness commitment check, transaction check, contextual check.[^ref-btc-core-net-processing] [^ref-btc-core-validationinterface]

### Compact Block

BIP152 compact block relay는 block header, short transaction identifier, 수신 측이 없을 가능성이 있는 transaction만 보내 block 전송 대역폭과 지연을 줄인다.[^ref-bip-0152] [^ref-btc-dev-p2p]

compact block 모델은 수신 node가 이미 mempool에 많은 candidate transaction을 가지고 있다고 가정한다. full block을 다시 통째로 보내는 대신, sender는 local reconstruction에 필요한 최소 정보만 전송하고, reconstruction이 실패할 때만 missing transaction을 요청한다.

이는 block propagation delay를 줄여 stale-block probability를 낮추므로 miner에게 운영상 중요하다.

---

## 7. Transaction Propagation

### Basic Transaction Relay

transaction relay는 보통 direct payload push가 아니라 announcement로 시작한다.

```text
peer A learns tx
-> peer A announces tx hash
-> peer B decides whether to request
-> peer B sends getdata
-> peer A sends tx
-> peer B validates and maybe relays onward
```

이는 네트워크 인지와 대역폭 비용이 큰 실제 전송을 분리한다.

### `txid` vs `wtxid`

SegWit는 legacy `txid`에는 영향을 주지 않지만 `wtxid`에는 영향을 주는 witness data를 도입했다. BIP339는 relay semantics를 갱신해 transaction을 `wtxid` 기준으로 announce하고 request할 수 있게 만들었고, 이를 통해 relay behavior가 SegWit 시대의 transaction identity와 더 잘 맞게 되었다.[^ref-bip-0339]

분석적으로 이는 중요하다. mempool observer가 legacy `txid`만으로 추론하면, 현대 transaction transport에서 중요한 relay-layer distinction을 놓칠 수 있기 때문이다.

### Fee Filter와 Relay Suppression

BIP133은 `feefilter`를 도입해, peer가 상대방에게 특정 feerate threshold 이하 transaction announcement를 보내지 말라고 알릴 수 있게 했다.[^ref-bip-0133] [^ref-btc-dev-p2p]

이는 consensus validity를 바꾸지 않는다. steady-state transaction relay에서 특정 peer가 무엇을 듣기로 선택하는지를 바꿀 뿐이다.

### Orphan Transaction

transaction이 부모보다 먼저 도착할 수 있다. 이 경우 수신 node는 즉시 fully processable하다고 보기보다 orphan candidate로 분류할 수 있다. Bitcoin Core의 orphan handling은 공격자가 dependency-missing transaction을 의도적으로 flood할 수 있기 때문에 memory use와 peer abuse를 제한한다.[^ref-btc-core-txorphanage]

중요한 구분:

- relay context의 orphan: 수신자가 현재 view에서 참조 입력을 아직 갖고 있지 않음
- consensus context의 invalid: 구조적 또는 의미적으로 프로토콜 규칙 위반

둘은 같지 않다.

### Transaction Request Scheduling

node는 announce된 모든 transaction을 모든 peer에게서 요청해서는 안 된다. Bitcoin Core는 transaction request management를 통해 어떤 object를 어떤 peer에게 요청할지 조정하며, 중복 request를 줄이고 delay/flood pattern을 방어한다.[^ref-btc-core-txrequest] [^ref-btc-core-net-processing]

---

## 8. Technical Mechanics

### Message Container

Bitcoin P2P message는 공통 message-header container를 공유한다.

```text
start string
command name
payload size
checksum
payload
```

start string은 network를 식별하고, command는 message type을 식별하며, checksum은 payload에서 파생된다. 이 공통 framing은 control, relay, synchronization message 전체에 적용된다.[^ref-btc-dev-p2p]

### Inventory

inventory entry는 36-byte 구조다.

```text
4 bytes   type identifier
32 bytes  object hash
```

inventory type은 transaction, block, filtered block, compact block, witness variant 같은 object를 구분한다.[^ref-btc-dev-p2p]

### Headers Download Limit

Developer Reference는 두 가지 practical sync asymmetry를 설명한다.

- `getblocks` reply는 최대 500개의 block hash를 담는다.
- `headers` reply는 최대 2,000개의 block header를 담을 수 있다.[^ref-btc-dev-p2p]

이 때문에 best chain을 찾는 효율적인 경로로 headers-first synchronization이 유도된다.

### Service Bit와 Capability Surface

service bit는 단순 metadata가 아니다. peer가 full block, limited-history block, new transport 같은 기능을 지원한다고 주장하는 방식이다. peer는 이 claim을 바탕으로 누구에게 어떤 요청을 보낼지 결정하지만, claim 자체는 실제 행동으로 확인되기 전까지 신뢰할 수 없다.[^ref-btc-dev-p2p] [^ref-bip-0159] [^ref-bip-0324]

### Relay Preference Signaling

relay 관련 message는 universal obligation이 아니라 preference를 표현한다.

- `sendheaders`: 새 block에 대해 header announcement 선호[^ref-bip-0130]
- `feefilter`: 이 peer에는 저수수료 transaction announcement 억제[^ref-bip-0133]
- `sendaddrv2`: 새 address message format 선호[^ref-bip-0155]

이들은 local session control이다. peer마다 다른 behavior를 택할 수 있다.

---

## 9. Validation Boundaries

### Transport, Relay, Consensus는 다른 계층이다

node는 peer로부터 message를 성공적으로 받아도, 그 내용을 mempool이나 chainstate에 수용하지 않을 수 있다. decision tree는 계층적이다.

1. Transport layer: message framing과 parsing이 가능한가?
2. Relay layer: 이 object를 request/store/forward할 가치가 있는가?
3. Validation layer: 이 object는 consensus와 local policy를 만족하는가?
4. Chainstate layer: active chain의 일부가 되는가, 아니면 side branch에만 남는가?

이 계층을 혼동하면 분석이 틀어진다.

### Header Acceptance는 Block Acceptance가 아니다

header가 proof-of-work와 linkage check를 통과해도, 이후 full block은 다음에서 실패할 수 있다.

- invalid transaction
- 잘못된 Merkle root
- 잘못된 witness commitment
- coinbase amount violation
- height-dependent deployment 같은 contextual rule

따라서 "header가 전파되었다"는 사실은 "block이 전역적으로 valid했다"는 뜻이 아니다.

### Mempool Admission은 Consensus가 아니다

transaction은 consensus-valid해도 다음 이유로 많은 mempool에 없을 수 있다.

- minimum relay fee
- fee filter suppression
- package topology
- ancestor/descendant limit
- local anti-abuse policy
- temporary missing parent

반대로 한 mempool에서 보였다는 사실이 그 transaction에 어떤 합의 상태를 부여하지도 않는다.

---

## 10. Security Assumptions and Failure Modes

### Latency의 중요성

block propagation은 같은 tip을 연장하는 경쟁과의 race다. 두 miner가 거의 동시에 유효한 block을 찾으면, 전파가 느릴수록 한 branch가 stale이 될 가능성이 커진다. propagation quality는 miner revenue variance와 network convergence에 직접 연결된다.

### Eclipse와 Topology Manipulation

공격자가 node의 peer set을 지배할 수 있으면, 그 node의 mempool, header, block view가 왜곡된다. 이것이 propagation layer의 eclipse risk다. 작업증명을 깨뜨릴 필요는 없다. 피해자의 정보 환경을 충분한 시간 통제하면 된다.

### Address Pollution

address relay는 인증되지 않으므로, 악성 peer는 저품질 또는 attacker-controlled address를 advertise할 수 있다. address management와 peer diversity는 이러한 집중 리스크를 줄이기 위해 존재한다.

### Flooding과 Resource Exhaustion

propagation code는 다음을 방어해야 한다.

- address spam
- inventory flood
- orphan transaction flood
- redundant data request
- outbound request slot을 낭비하는 slow-drip peer

Bitcoin Core의 request scheduling, orphan limit, peer-state tracking은 이런 이유로 존재한다.[^ref-btc-core-txrequest] [^ref-btc-core-txorphanage] [^ref-btc-core-net-processing]

### Privacy Leakage

first-seen timing, transaction announcement path, bloom-filter usage, peer-specific relay behavior는 wallet ownership이나 network location에 대한 정보를 새어 나가게 할 수 있다. BIP37식 bloom filtering은 실용적인 SPV 메커니즘이었지만, 잘 알려진 privacy limitation이 있다.[^ref-bip-0037]

### Transport Upgrade

BIP324는 version 2 encrypted transport protocol을 정의한다. transport encryption은 passive inspection이나 connection-layer tampering 일부에 대한 저항을 높일 수 있지만, peer announcement를 authoritative하게 만들지는 않으며 topology-based attack surface를 제거하지도 않는다.[^ref-bip-0324]

---

## 11. Mathematical or Economic Model

### Propagation Delay와 Stale Risk

다음을 두자.

- `T_p` = 첫 miner 수신부터 network 전체가 널리 알기까지의 propagation delay
- `lambda` = competing block discovery의 aggregate rate

단순 Poisson approximation에서 propagation time 동안 적어도 하나의 competing block이 발견될 확률은:

```text
P(competing block during propagation) = 1 - e^(-lambda * T_p)
```

이것은 거친 모델이지만 방향성은 정확하다. propagation이 느릴수록 stale-block exposure가 커진다.

### Relay Selectivity

node가 local feerate floor `f_min`을 사용한다면, 그 node가 받는 transaction 집합은:

```text
all consensus-valid unconfirmed transactions
```

이 아니라, 오히려 다음에 가깝다.

```text
transactions peers choose to announce
intersect
transactions meeting local relay preferences
intersect
transactions not lost to timing, topology, or outages
```

그래서 한 node의 mempool 측정치는 canonical global pool이 아니라 편향된 sample이다.

### Headers-First Bandwidth Intuition

header를 먼저 다운로드하는 비용은 full block body에 비해 매우 작다.

```text
1 header = 80 bytes
2,000 headers = 160,000 bytes before framing overhead
```

full block body에 비하면 작기 때문에, headers-first synchronization은 더 큰 대역폭과 validation work를 쓰기 전에 candidate chain extension을 저비용으로 평가하게 해 준다.

---

## 12. Bitcoin Core 구현

### `protocol.h`

`protocol.h`는 message type, inventory semantics, service flag, 관련 protocol constant를 정의한다. 이는 peer-to-peer communication의 schema layer다.[^ref-btc-core-protocol]

### `net.h`와 `net.cpp`

`net.h`와 `net.cpp`는 serialized network message, node object, connection management를 포함한 network transport와 peer connection substrate를 제공한다.[^ref-btc-core-net]

이 layer에서 Core는 주로 다음을 다룬다.

- socket과 session
- message queue
- peer bookkeeping
- address handling
- transport abstraction

### `net_processing`

`net_processing`은 relay와 peer-state의 핵심이다. 여기서 Bitcoin Core는 message handling, block/transaction announcement, request logic, peer misbehavior response, synchronization behavior를 조정한다.[^ref-btc-core-net-processing]

분석적으로는 raw network traffic가 다음과 같은 운영 결정을 거치는 지점이다.

- 특정 object request를 어느 peer에게 맡길지
- 지금 announce할지 나중에 announce할지
- 특정 peer가 stalling 중인지 flooding 중인지
- header, compact block, 다른 relay path 중 무엇을 선호할지

### `txrequest`

`txrequest`는 transaction request scheduling을 관리해, Core가 단순 중복 request를 피하고 peer별 announcement/request state를 추적하게 한다.[^ref-btc-core-txrequest]

### `txorphanage`

`txorphanage`는 local view에서 입력이 부족해 실패한 transaction을 관리한다. 이는 orphan handling이 DoS-sensitive network edge라는 사실을 반영해 announcement count, memory usage, dependency-related processing cost를 명시적으로 제한한다.[^ref-btc-core-txorphanage]

### `validationinterface`

`CValidationInterface`는 network layer 자체는 아니지만 중요하다. validation과 mempool event를 subscriber에게 노출하기 때문이다. network에서 받은 데이터가 validation outcome이나 chainstate update로 넘어가는 경계를 표시한다.[^ref-btc-core-validationinterface]

---

## 13. Consensus, Policy, and Presentation

### Consensus Layer

consensus는 block 또는 transaction이 Bitcoin system에 대해 유효한지를 결정한다.

예:

- block proof of work
- consensus rule 기준 transaction syntax
- script validity
- subsidy limit
- witness commitment correctness

### Policy Layer

policy는 특정 node 관점에서 otherwise acceptable object를 relay/store/mine할 가치가 있는지를 결정한다.

예:

- fee floor
- mempool package limit
- orphan retention limit
- relay suppression
- peer-specific announcement choice

### Presentation Layer

presentation은 observer나 API가 무엇을 보는지의 문제다. wallet, explorer, analyst node의 view는 다음에 의존한다.

- 어떤 peer에 연결되었는가
- 언제 연결되었는가
- 어떤 filter나 preference가 활성화되었는가
- pruning node였는가
- initial block download 상태였는가

따라서 observed network state는 항상 vantage-point dependent하다.

---

## 14. 온체인 함의

propagation은 대부분 오프체인 behavior지만, 간접적인 분석 흔적을 남긴다.

### Mempool Visibility Bias

transaction의 특정 dataset에서의 first appearance time은 다음을 반영할 수 있다.

- observer topology
- observer fee filter
- peer-specific delay
- orphan dependency timing
- BIP339 또는 compact-relay 시대 behavior

이는 그 transaction이 "Bitcoin network에 처음 들어온 시간"과 반드시 같지 않다.

### Block Arrival Bias

특정 node에서 관측한 block arrival time은 local receipt time이지 objective creation time이 아니다. miner speed나 geographic advantage를 비교하려면 여러 measurement point가 필요하다.

### Fork와 Reorg 민감도

짧게 지속된 fork는 chain-history fact가 되기 전까지는 propagation event다. 느리거나 비대칭적인 propagation은 결국 most-work branch로 깨끗하게 수렴하더라도, 일시적으로 tip에 대한 불일치를 증가시킬 수 있다.

---

## 15. Institutional Thinking

기관이 mempool 또는 propagation data에 의존한다면, 네트워크를 부분 관측 시스템으로 다뤄야 한다.

### Practical Implications

- one-node mempool feed만으로는 강한 market/compliance claim을 하기 어렵다.
- propagation latency는 miner monitoring, exchange deposit risk, 고빈도 fee estimation에 중요하다.
- network vantage quality는 사소한 인프라 세부가 아니라 핵심 데이터 품질 문제다.
- Bitcoin Core의 relay-policy change는 Bitcoin consensus를 바꾸지 않아도 관측되는 transaction population을 크게 바꿀 수 있다.

### Better Research Posture

진지한 propagation 분석에는 보통 다음이 필요하다.

- geographically distributed multiple node
- peer composition과 software version의 명확한 logging
- local receipt time과 estimated origin time의 분리
- pruning, blocksonly, fee threshold 같은 policy toggle에 대한 인식

---

## 16. Common Misinterpretations

### "내 node가 못 봤으면 네트워크도 못 본 것이다"

틀렸다. 당신의 node는 부분적이고 경로 의존적인 view만 가진다.

### "relay되었으면 valid하다"

틀렸다. relay된 object도 나중에 policy, full validation, contextual check에서 실패할 수 있다.

### "mempool에 없으면 invalid하다"

틀렸다. 수수료가 낮거나, 부모가 없거나, filtering되었거나, 지연되었거나, local policy에서 제외되었을 수 있다.

### "peer가 말한 height나 service는 믿을 수 있다"

틀렸다. 이 field들은 인증되지 않은 claim이다.

### "transport encryption이 propagation attack을 해결한다"

틀렸다. wire level의 confidentiality와 robustness는 높일 수 있어도, peer selection과 topology attack은 여전히 중요하다.

---

## 17. Research Questions

1. geographically distributed first-seen transaction timestamp 사이의 분산은 얼마나 큰가?
2. BIP339는 SegWit-heavy transaction flow의 empirical relay convergence를 얼마나 바꿨는가?
3. fee spike나 mempool fragmentation 상황에서 compact block reconstruction failure는 얼마나 자주 발생하는가?
4. short-lived fork incidence 중 얼마나 많은 부분이 hashpower distribution보다 propagation asymmetry로 설명되는가?
5. 실제로 eclipse exposure를 유의미하게 줄이려면 어느 정도의 address-manager diversity가 필요한가?

---

## 18. Practical Exercises

### Exercise 1

제어된 테스트 환경에서 `version`, `verack`, `inv`, `getdata`, `headers`, `block` message를 캡처하고, block sync와 transaction relay 각각의 sequence를 매핑하라.

### Exercise 2

서로 다른 fee filter를 가진 여러 node를 실행하고, 같은 시간 창에서 first-seen transaction의 집합과 timing을 비교하라.

### Exercise 3

서로 다른 지역의 node들에서 first-seen block arrival time을 측정하고 propagation delay 분포를 추정하라.

### Exercise 4

block-only node profile 활성화 전후의 transaction visibility를 비교해 chain sync와 mempool relay의 차이를 이해하라.

---

## 19. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| `version`/`verack`, `addr`, `inv`, `getdata`, `headers`, and `block` message roles | Directly specified | Bitcoin Developer Reference and Bitcoin Core protocol sources |
| `sendheaders`, `feefilter`, compact blocks, `addrv2`, limited-history serving, and v2 transport roles | Directly specified | Relevant BIPs |
| Request scheduling, orphan limits, and peer-state handling in Core | Directly specified | Bitcoin Core Doxygen sources |
| Relay-policy and vantage-point implications for analysts | Inference from sources | Derived from protocol behavior and node-local policy design |
| Latency and stale-risk relationship | Analytical model | Directionally correct simplification, not a full empirical model |

---

## 20. Knowledge Graph

```text
Nodes and Network Propagation
├─ Peer Discovery
│  ├─ addr
│  ├─ addrv2
│  ├─ getaddr
│  └─ AddrMan
├─ Session Establishment
│  ├─ version
│  ├─ verack
│  ├─ service bits
│  └─ ping/pong
├─ Block Propagation
│  ├─ inv
│  ├─ headers
│  ├─ getheaders
│  ├─ block
│  ├─ cmpctblock
│  └─ chainwork comparison
├─ Transaction Propagation
│  ├─ inv
│  ├─ getdata
│  ├─ tx
│  ├─ wtxid relay
│  ├─ feefilter
│  └─ orphan handling
├─ Implementation
│  ├─ protocol.h
│  ├─ net.h / net.cpp
│  ├─ net_processing
│  ├─ txrequest
│  ├─ txorphanage
│  └─ validationinterface
└─ Risks
   ├─ eclipse
   ├─ flooding
   ├─ stale risk
   └─ privacy leakage
```

---

## 21. 참고문헌

### Primary Sources

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Sections 5 and 8. https://bitcoin.org/bitcoin.pdf
[^ref-btc-dev-p2p]: Bitcoin Developer Reference, "P2P Network." https://developer.bitcoin.org/reference/p2p_networking.html
[^ref-bip-0037]: BIP37, "Connection Bloom filtering." https://github.com/bitcoin/bips/blob/master/bip-0037.mediawiki
[^ref-bip-0130]: BIP130, "sendheaders message." https://github.com/bitcoin/bips/blob/master/bip-0130.mediawiki
[^ref-bip-0133]: BIP133, "feefilter message." https://github.com/bitcoin/bips/blob/master/bip-0133.mediawiki
[^ref-bip-0152]: BIP152, "Compact Block Relay." https://github.com/bitcoin/bips/blob/master/bip-0152.mediawiki
[^ref-bip-0155]: BIP155, "addrv2 message." https://github.com/bitcoin/bips/blob/master/bip-0155.mediawiki
[^ref-bip-0159]: BIP159, "NODE_NETWORK_LIMITED service bit." https://github.com/bitcoin/bips/blob/master/bip-0159.mediawiki
[^ref-bip-0324]: BIP324, "Version 2 P2P Encrypted Transport Protocol." https://github.com/bitcoin/bips/blob/master/bip-0324.mediawiki
[^ref-bip-0339]: BIP339, "WTXID-based transaction relay." https://github.com/bitcoin/bips/blob/master/bip-0339.mediawiki
[^ref-btc-core-net]: Bitcoin Core Doxygen, `net.h` and `net.cpp`. https://doxygen.bitcoincore.org/net_8cpp.html
[^ref-btc-core-protocol]: Bitcoin Core Doxygen, `protocol.h` and `NetMsgType`. https://doxygen.bitcoincore.org/namespace_net_msg_type.html
[^ref-btc-core-net-processing]: Bitcoin Core Doxygen, `net_processing.cpp` and `net_processing.h`, including `PeerManager` and peer message processing references. https://doxygen.bitcoincore.org/
[^ref-btc-core-txrequest]: Bitcoin Core Doxygen, `txrequest.h` and transaction request coordination references. https://doxygen.bitcoincore.org/
[^ref-btc-core-txorphanage]: Bitcoin Core Doxygen, `txorphanage.h` and orphan transaction management references. https://doxygen.bitcoincore.org/txorphanage_8h.html
[^ref-btc-core-validationinterface]: Bitcoin Core Doxygen, `validationinterface.h` and `CValidationInterface`. https://doxygen.bitcoincore.org/class_c_validation_interface.html

### Supporting Interpretation Notes

- Where this document discusses analyst bias, local view asymmetry, or institutional measurement limits, those statements are inferences from the documented message flow, relay preferences, and node-local policy architecture rather than explicit protocol claims.

---

## 22. 교차 참조

### Previous

- BITCOIN-021 — Blocks and Block Headers

### Next

- BITCOIN-023 — Forks

### Related

- BITCOIN-006 — Whitepaper Section 5 — Network
- BITCOIN-009 — Whitepaper Section 8 — Simplified Payment Verification
- BITCOIN-017 — Mempool
- BITCOIN-018 — Transaction Fees
- BITCOIN-021 — Blocks and Block Headers
- POW-011 — Cumulative Chainwork
- POW-013 — Bitcoin Core Proof-of-Work Validation
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- peer discovery, handshake, header sync, block relay, transaction relay를 분리했다.
- relay policy와 consensus validity를 구분했다.
- `sendheaders`, compact block, `addrv2`, `feefilter`, `wtxid` relay를 consensus가 아닌 protocol/peer-service feature로 배치했다.
- Bitcoin Core 구현 참조는 networking과 relay에 직접 관련된 module로 제한했다.

### Evidence Review

Passed.

- 기본 네트워크 workflow는 백서와 Bitcoin Developer Reference에 연결했다.
- message semantics는 Bitcoin Developer Reference에 연결했다.
- relay feature는 각 BIP에 연결했다.
- Core 구현 설명은 `net`, `protocol`, `txorphanage`, `validationinterface` Doxygen reference에 연결했다.
- analyst-facing interpretation은 필요한 부분에서 inference로 표시했다.

### Editorial Review

Passed.

- Markdown structure는 프로젝트 deep-dive template을 따른다.
- metadata가 완전하다.
- node role, relay, synchronization, policy 용어가 일관적이다.
- table과 code fence가 닫혀 있다.

### Adversarial Review

Passed.

- mempool visibility를 global truth로 취급하지 않았다.
- peer claim을 인증된 사실로 취급하지 않았다.
- relay acceptance를 consensus validity와 동일시하지 않았다.
- transport encryption을 topology attack에 대한 완전한 방어로 보지 않았다.
- 백서가 modern Bitcoin relay behavior 전체를 규정한다고 주장하지 않았다.

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
