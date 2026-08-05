---
knowledge_id: BITCOIN-032
title: Lightning Network
subtitle: Off-Chain Payment Channel, Commitment Transaction, HTLC Routing, Revocation, 그리고 Base Layer Enforcement와 Layer-2 State의 경계
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 145 min
estimated_study: 420 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Layer 2
  - Lightning
  - Payments
parent:
  - Modern Bitcoin
prerequisites:
  - BITCOIN-009
  - BITCOIN-016
  - BITCOIN-022
  - BITCOIN-030
  - BITCOIN-031
related_topics:
  - Payment Channels
  - HTLC
  - Commitment Transaction
  - Routing
  - CSV
  - CLTV
  - Watchtowers
primary_sources:
  - REF-LN-PAPER-001
  - REF-BOLT-000-INTRO-001
  - REF-BOLT-002-PEER-001
  - REF-BOLT-003-TX-001
  - REF-BOLT-004-ROUTING-001
  - REF-BOLT-007-GOSSIP-001
  - REF-BIP-0112
  - REF-BTC-CORE-BIPS-001
tags:
  - bitcoin
  - lightning
  - layer2
  - payment-channels
  - htlc
  - routing
  - commitment-transactions
  - revocation
---

# Lightning Network
> Modern Bitcoin  
> Research Unit: BITCOIN-032

---

## Research Brief

```yaml
knowledge_id: BITCOIN-032
title: Lightning Network
research_question: >
  How does the Lightning Network move payments off-chain through bidirectional
  payment channels, what roles do commitment transactions, revocation,
  HTLCs, routing, and gossip play, and how should analysts separate Lightning's
  off-chain state from the on-chain enforcement and observability limits of the
  Bitcoin base layer?
document_type: deep-dive
difficulty: L400
prerequisites:
  - BITCOIN-009
  - BITCOIN-016
  - BITCOIN-022
  - BITCOIN-030
  - BITCOIN-031
parent: Modern Bitcoin
previous: BITCOIN-031
next: BITCOIN-033
related_topics:
  - Payment Channels
  - HTLCs
  - Routing Gossip
  - On-Chain Enforcement
  - Watchtowers
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
  - Full BOLT-by-BOLT implementation tutorial
  - Detailed Lightning liquidity management strategies
  - Full watchtower protocol landscape
  - AMP / trampoline / blinded-path deep dive
  - Alt-L2 comparisons
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Lightning를 off-chain Bitcoin payment channel network로 설명할 수 있다.
- on-chain channel funding/settlement와 off-chain balance update를 구분할 수 있다.
- commitment transaction의 역할을 설명할 수 있다.
- revocation과 delay mechanism이 channel security에 왜 중요한지 설명할 수 있다.
- HTLC가 여러 channel에 걸친 routed payment를 trustless하게 만드는 메커니즘임을 설명할 수 있다.
- private channel state와 public network-gossip information을 구분할 수 있다.
- base-layer chain data에서 무엇이 보이고 무엇이 보이지 않는지 설명할 수 있다.

---

## 2. 핵심 질문

1. Lightning Network란 무엇인가?
2. Lightning는 왜 여전히 Bitcoin base layer가 필요한가?
3. payment channel이란 무엇인가?
4. commitment transaction이란 무엇인가?
5. revoked commitment transaction이란 무엇이고, 왜 그것을 broadcast하는 것이 위험한가?
6. routed payment에서 HTLC는 어떤 역할을 하는가?
7. channel gossip과 `short_channel_id`는 무엇에 쓰이는가?
8. on-chain analyst는 Lightning에서 무엇을 관찰할 수 있고, 무엇은 off-chain에 남는가?

---

## 3. Executive Summary

Lightning Network는 다수의 Bitcoin payment를 payment channel network를 통해 off-chain으로 이동시키는 layer-2 protocol이다. Bitcoin blockchain은 channel funding, dispute enforcement, 최종 settlement가 필요할 때만 사용한다.[^ref-ln-paper][^ref-bolt-000]

두 참여자는 jointly controlled on-chain output에 bitcoin을 잠가 channel을 연다. 이후 blockchain에 매 update를 broadcast하지 않고도, channel balance를 재배분하는 updated off-chain commitment state를 서로 교환한다. 협력이 유지되면 많은 payment가 극히 드문 on-chain settlement만으로 처리될 수 있다.[^ref-ln-paper][^ref-bolt-000][^ref-bolt-003]

Lightning의 핵심 security trick은 모든 참여자가 항상 on-chain에서 강제 집행 가능한 exit path를 보유한다는 점이다. commitment transaction, revocation logic, `to_self_delay`, HTLC script는 오래된 state를 publish해 속이려는 시도를 비용 높고 위험하게 만든다. BIP112의 `CHECKSEQUENCEVERIFY`는 이런 channel design을 가능하게 하는 base-layer primitive로 명시적으로 인용된다.[^ref-bip-0112][^ref-bolt-003]

routed payment에는 HTLC가 사용된다. intermediary는 outgoing hop에서 안전하게 reclaim할 수 있을 때만 value를 forwarding한다. 네트워크 계층은 peer messaging, onion routing, gossip을 사용해 channel을 발견하고 multi-hop payment를 blockchain에 직접 기록하지 않고도 전달한다.[^ref-bolt-002][^ref-bolt-004][^ref-bolt-007]

분석가 관점에서 Lightning는 observability의 날카로운 단절점을 만든다. blockchain은 channel open, close, 일부 enforcement event를 보여줄 수 있다. 하지만 모든 off-chain rebalance, route attempt, invoice, failure, intermediate channel state를 보여주지는 않는다.

---

## 4. Protocol Structure

### Layer Split

Lightning는 두 개의 결합된 layer로 보는 것이 가장 정확하다.

| Layer | What Happens There |
|---|---|
| Bitcoin base layer | funding output, unilateral close, mutual close, HTLC enforcement, penalty, 최종 settlement |
| Lightning layer | channel state update, HTLC forwarding, routing, gossip, invoice, peer messaging |

### Channel Lifecycle

상위 수준 lifecycle은 다음과 같다.

```text
open channel on-chain
-> exchange off-chain states
-> forward and settle off-chain payments
-> close cooperatively or force-close on-chain
```

### Why Base Layer Still Matters

Lightning는 Bitcoin finality를 대체하지 않는다. 빈번한 state change를 off-chain으로 외주화할 뿐이며, 당사자가 충돌하거나 settlement를 on-chain으로 강제해야 할 때는 다시 Bitcoin consensus에 의존한다.[^ref-bolt-000]

---

## 5. Payment Channels

### Funding Output

Lightning channel은 jointly controlled output에 bitcoin을 잠그는 funding transaction으로 시작한다. BOLT #3는 funding transaction output과, 채널 참여자가 enforceable state에 합의하기 위해 필요한 on-chain transaction/script format을 정의한다.[^ref-bolt-003]

### Commitment Transactions

각 측은 최신 enforceable channel fund split을 나타내는 commitment transaction을 보유한다. 이 transaction들이 모두 broadcast되는 것은 아니다. 협력이 깨질 경우 사용할 수 있는 pre-signed exit state다.[^ref-bolt-000][^ref-bolt-003]

### State Updates

새 payment가 발생할 때마다 peer는 updated commitment state를 협상하고 이전 state를 revoke한다. BOLT #2는 `commitment_signed`, `revoke_and_ack`를 포함해 update가 peer 사이에서 되돌릴 수 없게 확정되는 message flow를 설명한다.[^ref-bolt-002]

### Revocation

오래된 commitment transaction은 나중에 그것을 broadcast하려는 party에게 위험한 것이 된다. BOLT #0과 BOLT #3는 revoked commitment transaction과 penalty spending path를 설명하며, 이는 상대방이 cheating을 처벌할 수 있게 한다.[^ref-bolt-000][^ref-bolt-003]

---

## 6. HTLCs and Routed Payments

### HTLC Purpose

Hash Time-Locked Contract는 조건부 payment를 가능하게 한다. receiver 또는 downstream hop은 timeout 전에 preimage를 공개해야 value를 claim할 수 있고, 그렇지 않으면 sender가 fund를 회수할 수 있다. BIP112는 `CHECKSEQUENCEVERIFY`의 동기와 적용 맥락으로 HTLC와 bidirectional payment channel을 명시적으로 제시한다.[^ref-bip-0112]

### Routed Safety Condition

BOLT #2는 HTLC를 forwarding하는 node가, outgoing payment는 claim될 수 있지만 대응되는 incoming payment는 reclaim할 수 없는 position을 만들면 안 된다고 요구한다. 그래서 CLTV delta, payment hash, commitment synchronization이 중요하다.[^ref-bolt-002]

### Onion Routing

BOLT #4는 hop별 forwarding instruction을 실어 나르는 onion-routing packet format을 정의한다. routing packet은 associated data로 `payment_hash`에 commit해, 같은 onion이 다른 payment hash에 재사용되는 것을 방지하는 데 도움을 준다.[^ref-bolt-004][^ref-bolt-002]

### End Result

HTLC는 pairwise channel을 payment network로 바꾼다.

```text
A pays B conditionally
B pays C conditionally
C reveals preimage
-> C gets paid
-> B safely claims upstream
-> A's payment completes
```

이 과정에서 B는 C를 신탁할 필요가 없고, A도 route 전체 payment를 B에게 보관시키는 방식으로 신뢰할 필요가 없다.

---

## 7. Commitment and Penalty Mechanics

### `to_self_delay`

BOLT #3는 commitment transaction의 owner에게 fund를 돌려주는 output에 `to_self_delay` block 지연을 두도록 규정한다. 이는 revoked commitment transaction이 publish됐을 때 counterparty가 대응할 시간을 주기 위함이다.[^ref-bolt-003]

### Penalty Path

revoked commitment transaction이 broadcast되면 counterparty는 revocation-related key를 사용해 output을 즉시 spend할 수 있다. 이것이 오래된 state publication을 비싸고 일반적으로 비합리적으로 만드는 실질적 enforcement engine이다.[^ref-bolt-000][^ref-bolt-003]

### Why This Is Layer-2 Security, Not Base-Layer Magic

Bitcoin은 Lightning balance를 "이해"하지 않는다. Bitcoin이 검증하는 것은 on-chain에 broadcast된 transaction과 script뿐이다. Lightning security는 이러한 transaction/script를 미리 어떻게 구성해두는지, 그리고 당사자가 chain을 어떻게 모니터링하는지에서 나온다.

---

## 8. Gossip, Routing, and Public Topology

### Public Channel Discovery

BOLT #7은 node와 channel discovery를 정의하며 다음을 포함한다.

- `channel_announcement`
- `channel_update`
- `node_announcement`
- funding transaction location에 기반한 `short_channel_id`[^ref-bolt-007]

### `short_channel_id`

public channel에 대해 `short_channel_id`는 다음을 encode한다.

- block height
- block 내 transaction index
- transaction 내 output index[^ref-bolt-007]

이것은 Lightning topology와 base chain의 confirmed funding outpoint 사이에 public bridge를 만든다.

### Limits of Public Topology

gossip는 public channel과 public routing information을 보여준다. 하지만 다음은 드러내지 않는다.

- private channel state
- 모든 routed payment
- 네트워크에 announce되지 않은 private channel
- channel의 전체 real-time liquidity

---

## 9. Technical Mechanics

### Channel Management Messages

BOLT #2는 peer channel management를 establishment, normal operation, closing 같은 phase로 구조화하고, HTLC update, commitment, revocation, fee update를 위한 message family를 정의한다.[^ref-bolt-002]

### On-Chain Transaction Format

BOLT #3는 다음을 규정한다.

- funding output
- commitment transaction
- HTLC-timeout 및 HTLC-success transaction
- closing transaction variant
- fee calculation 및 payment rule[^ref-bolt-003]

### SegWit Dependence

Lightning paper는 malleability를 해결하는 새로운 sighash behavior가 trustless chained off-chain update에 필요하다고 명시한다. 이것이 SegWit가 deployable Lightning design의 기반이 되는 이유다.[^ref-ln-paper]

### Outsourced Monitoring

BOLT #3는 revoked transaction에 대한 outsourced watching을 가능하게 하기 위한 맥락에서도 key-derivation과 revocation structure를 설명한다. watchtower protocol의 세부 내용은 여기서 사용한 기본 BOLT 범위를 벗어나지만, 개념적 기반은 여기서 나온다.[^ref-bolt-003]

---

## 10. Validation Boundaries

### What Bitcoin Validates

Bitcoin이 검증하는 것은 다음이다.

- funding transaction
- closing transaction
- HTLC-enforcement transaction
- 실제 on-chain에 broadcast된 timelock과 script condition

### What Lightning Maintains Off-Chain

Lightning가 off-chain으로 유지하는 것은 다음이다.

- 최신 channel balance
- pending HTLC set
- routing attempt
- invoice state
- peer negotiation state

### Consequence

on-chain observer가 보는 것은 경계 이벤트뿐이다.

- open
- close
- force-close
- penalty 또는 timeout style enforcement
- 경우에 따라 public channel linkage through gossip-derived identifier

그 외 대부분은 off-chain state다.

---

## 11. Security Assumptions and Failure Modes

### Honest Monitoring Requirement

Lightning는 channel participant 또는 delegated watcher가 delay window가 만료되기 전에 대응할 수 있을 만큼 자주 chain을 모니터링한다고 가정한다. revoked-state publication을 놓치고 punishment window가 닫히면 fund를 잃을 수 있다.

### Liquidity Constraint

Lightning는 단순히 "무료 즉시 Bitcoin"이 아니다. payment는 channel balance, routing availability, fee policy, CLTV window, topology knowledge의 제약을 받는다.

### Routing Failure

유효한 invoice나 path discovery가 delivery를 보장하지는 않는다. HTLC는 insufficient liquidity, fee change, policy difference, timeout, peer unavailability 때문에 실패할 수 있다.

### Public vs Private Channels

public gossip는 route discovery를 개선하지만, channel은 private 또는 partially private하게 유지될 수 있다. 이는 일부 privacy를 개선하는 동시에 observability와 route knowledge를 줄인다.

---

## 12. Mathematical or Economic Model

### Channel State Model

총 용량이 `C`인 단순한 two-party channel에서는:

```text
balance_A + balance_B = C
```

새로운 funding/splice event가 일어나기 전까지 off-chain update는 총 용량을 유지한 채 split만 재배분한다.

### Forwarding Fee Intuition

Lightning forwarding은 기본적으로 무료가 아니다. BOLT #7은 다음과 같은 channel fee parameter를 설명한다.

```text
fee_base_msat
fee_proportional_millionths
```

따라서 forwarded amount와 필요한 fee는 channel policy에 따라 달라진다.[^ref-bolt-007]

### Settlement-Efficiency Intuition

channel 내부에서 `N`번의 transfer가 발생하고 open/close만 blockchain을 친다면, on-chain footprint는 payment count보다 훨씬 느리게 증가한다. 이것이 Lightning scaling의 핵심 직관이다. 다만 실제 환경에서는 liquidity, routing, rebalancing cost가 이 이상화된 그림을 제한한다.

---

## 13. Bitcoin Core Implementation

### Bitcoin Core Boundary

Bitcoin Core는 Lightning protocol 자체를 구현하지는 않지만, Lightning가 의존하는 많은 base-layer capability를 구현한다. implemented-BIP index에는 다음과 같은 관련 primitive가 기록돼 있다.

- BIP65 `CHECKLOCKTIMEVERIFY`
- BIP68 relative lock-time
- BIP112 `CHECKSEQUENCEVERIFY`
- deployable Lightning-era transaction safety에 필요한 SegWit support[^ref-btc-core-bips]

### Why This Still Matters

Bitcoin Core에 Lightning node logic이 내장되어 있지 않더라도, Lightning channel은 결국 Bitcoin transaction, script, timelock을 통해 settlement되고 집행되므로 base client는 Lightning foundation의 일부다.

---

## 14. On-Chain Implications

### What Can Be Observed

base-layer analyst는 보통 다음을 관찰할 수 있다.

- funding output
- channel open과 close
- 일부 public-channel identifier
- force-close behavior
- timeout 또는 penalty와 유사한 enforcement pattern

### What Cannot Be Reliably Observed

base-layer analyst는 보통 다음을 관찰할 수 없다.

- 모든 off-chain payment
- intermediate channel balance
- private forwarding history
- failed route attempt
- invoice semantic
- real-time usable channel liquidity

### Caution on Attribution

public funding outpoint가 있다고 해서 stable channel usefulness, current liquidity, successful routing performance를 의미하는 것은 아니다. public topology는 Lightning의 실제 payment graph를 부분적으로만 보여준다.

---

## 15. Institutional Thinking

기관은 Lightning를 Bitcoin consensus의 대체물이 아니라 settlement-thinning network로 분석해야 한다.

### Practical Implications

- channel exposure는 operational exposure이기도 하므로 monitoring, liquidity, key-management discipline이 중요하다.
- Lightning에 대한 on-chain analytics는 대부분의 state가 off-chain에 있으므로 명시적인 uncertainty label을 포함해야 한다.
- treasury와 payment team은 channel capacity와 즉시 사용 가능한 outbound/inbound liquidity를 구분해야 한다.
- incident playbook은 force-close, stale-state risk, watchtower dependence를 다뤄야 한다.

---

## 16. Common Misinterpretations

### "Lightning transactions are invisible Bitcoin transactions"

과장된 표현이다. 많은 off-chain state change는 on-chain에 보이지 않지만, channel open, close, enforcement는 여전히 base-layer trace를 남긴다.

### "A channel's capacity equals currently spendable liquidity in either direction"

틀렸다. capacity는 총 잠금 가치이지, 한쪽에서 현재 outbound 또는 inbound로 쓸 수 있는 유동성과 동일하지 않다.

### "Lightning removes trust entirely"

틀렸다. custody 의존성을 줄이고 enforceable contract를 사용하지만, 참여자는 여전히 monitoring, implementation correctness, routing assumption, time-sensitive response window에 의존한다.

### "Bitcoin Core includes Lightning"

틀렸다. Bitcoin Core는 관련 base-layer primitive를 제공할 뿐이고, Lightning protocol logic은 별도 구현체에 존재한다.

### "All Lightning channels are publicly visible"

틀렸다. public gossip는 announced channel만 포함하며, private channel과 대부분의 channel-state evolution은 그 바깥에 있다.

---

## 17. Research Questions

1. 경제적으로 의미 있는 Lightning activity 중 얼마나 많은 부분이 chain-only analysis에서 보이지 않는가?
2. channel liquidity가 대부분 숨겨져 있을 때 network usability를 가장 잘 proxy하는 public metric은 무엇인가?
3. 기관은 stale-state monitoring risk와 watchtower dependence를 어떻게 정량화해야 하는가?
4. congestion regime과 fee spike에 따라 force-close pattern은 어떻게 달라지는가?

---

## 18. Practical Exercises

### Exercise 1

funding transaction, commitment transaction, closing transaction의 차이를 설명하라.

### Exercise 2

`to_self_delay`가 존재하는 이유와 그것이 완화하는 risk를 설명하라.

### Exercise 3

public `short_channel_id`가 주어졌을 때, 그것이 어떤 funding outpoint를 가리키는지 식별하라.

### Exercise 4

on-chain data만으로는 복구할 수 없는 중요한 Lightning 사실 다섯 가지를 나열하라.

---

## 19. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Channel, commitment, HTLC, gossip, and public discovery mechanics | Directly specified | Lightning paper and BOLT specs |
| CSV role in Lightning-style contracts | Directly specified | BIP112 |
| SegWit as deployability foundation | Directly specified plus inference | Lightning paper motivation and Bitcoin base-layer context |
| Off-chain observability limits | Inference from sources | Derived from what BOLTs place off-chain vs on-chain |

---

## 20. Knowledge Graph

```text
Lightning Network
├─ Base Layer Anchors
│  ├─ funding transaction
│  ├─ timelocks
│  ├─ CSV / CLTV
│  └─ final settlement
├─ Channel State
│  ├─ commitment transactions
│  ├─ revocation
│  ├─ to_self_delay
│  └─ force close
├─ Routed Payments
│  ├─ HTLCs
│  ├─ payment hash / preimage
│  ├─ onion routing
│  └─ forwarding fees
├─ Network Topology
│  ├─ channel_announcement
│  ├─ channel_update
│  ├─ node_announcement
│  └─ short_channel_id
└─ Limits
   ├─ off-chain opacity
   ├─ liquidity constraints
   ├─ monitoring requirement
   └─ route failure risk
```

---

## 21. 참고문헌

### Primary Sources

[^ref-ln-paper]: Joseph Poon and Thaddeus Dryja, "The Bitcoin Lightning Network: Scalable Off-Chain Instant Payments," January 14, 2016. https://nakamotoinstitute.org/library/lightning-network/

[^ref-bolt-000]: Lightning BOLTs, BOLT #0 Introduction and Index. https://github.com/lightning/bolts/blob/master/00-introduction.md

[^ref-bolt-002]: Lightning BOLTs, BOLT #2 Peer Protocol for Channel Management. https://github.com/lightning/bolts/blob/master/02-peer-protocol.md

[^ref-bolt-003]: Lightning BOLTs, BOLT #3 Bitcoin Transaction and Script Formats. https://github.com/lightning/bolts/blob/master/03-transactions.md

[^ref-bolt-004]: Lightning BOLTs, BOLT #4 Onion Routing Protocol. https://github.com/lightning/bolts/blob/master/04-onion-routing.md

[^ref-bolt-007]: Lightning BOLTs, BOLT #7 P2P Node and Channel Discovery. https://github.com/lightning/bolts/blob/master/07-routing-gossip.md

[^ref-bip-0112]: BIP112, "CHECKSEQUENCEVERIFY," including Lightning-related motivation and HTLC / channel examples. https://github.com/bitcoin/bips/blob/master/bip-0112.mediawiki

[^ref-btc-core-bips]: Bitcoin Core `doc/bips.md`, implemented BIP index including BIP65, BIP68, BIP112, SegWit, and related primitives. https://github.com/bitcoin/bitcoin/blob/master/doc/bips.md

### Supporting Interpretation Notes

- Where this document discusses observability limits, institutional liquidity uncertainty, or payment-graph incompleteness, those statements are inferences from Lightning's off-chain design rather than explicit BOLT claims about analysts.

---

## 22. 교차 참조

### Previous

- BITCOIN-031 — Taproot

### Next

- BITCOIN-033 — Bitcoin Core

### Related

- BITCOIN-009 — Whitepaper Section 8 — Simplified Payment Verification
- BITCOIN-016 — Script & ScriptPubKey
- BITCOIN-022 — Nodes and Network Propagation
- BITCOIN-030 — SegWit
- BITCOIN-031 — Taproot

---

## Review Status

### Technical Review

Passed.

- channel funding, commitment state, HTLC forwarding, revocation, gossip를 분리해 설명했다.
- base-layer enforcement와 off-chain state를 명확히 구분했다.
- Lightning의 Bitcoin primitive 의존성을 설명하되, Lightning가 Bitcoin Core의 일부라는 식으로 오해되지 않게 서술했다.
- chain-only overinterpretation을 막기 위해 observability limit를 포함했다.

### Evidence Review

Passed.

- Lightning paper와 BOLT spec은 channel, HTLC, routing claim을 뒷받침한다.
- BIP112는 CSV 및 Lightning-style contract의 동기를 뒷받침한다.
- Bitcoin Core `doc/bips.md`는 base-layer implementation context를 뒷받침한다.
- hidden state와 liquidity limit에 대한 analytical statement는 inference로 라벨링했다.

### Editorial Review

Passed.

- 구조는 프로젝트 deep-dive format을 따른다.
- metadata는 완전하다.
- terminology는 funding transaction, commitment transaction, revoked commitment, HTLC, `to_self_delay`, `short_channel_id`로 일관된다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 Lightning state가 on-chain에 완전히 드러난다고 주장하지 않는다.
- total capacity를 directional liquidity와 혼동하지 않는다.
- Lightning가 monitoring이나 trust assumption을 제거한다고 주장하지 않는다.
- Bitcoin Core가 Lightning를 native하게 구현한다고 암시하지 않는다.
- public gossip가 실제 payment success를 완전히 증명한다고 과장하지 않는다.

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
