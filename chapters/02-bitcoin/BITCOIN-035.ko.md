---
knowledge_id: BITCOIN-035
title: Bitcoin in 2026
subtitle: 2026년 8월 4일 기준 현재 protocol posture, operational reality, policy evolution, 그리고 institutional interpretation
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 145 min
estimated_study: 420 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Current State
  - Bitcoin Core
  - Institutional Research
parent:
  - Modern Bitcoin
prerequisites:
  - BITCOIN-027
  - BITCOIN-030
  - BITCOIN-031
  - BITCOIN-032
  - BITCOIN-033
  - BITCOIN-034
related_topics:
  - SegWit
  - Taproot
  - Lightning
  - Bitcoin Core
  - Fee Market
  - Policy Evolution
primary_sources:
  - REF-BTC-CORE-HOMEPAGE-2026-001
  - REF-BTC-CORE-31-1-RELEASE-001
  - REF-BTC-CORE-31-0-RELEASE-001
  - REF-BTC-CORE-FILES-001
  - REF-BIP-0141
  - REF-BIP-0341
  - REF-BOLT-000-INTRO-001
  - REF-BIPS-REPO-001
tags:
  - bitcoin
  - 2026
  - bitcoin-core
  - current-state
  - taproot
  - segwit
  - lightning
  - institutional
---

# Bitcoin in 2026
> Modern Bitcoin  
> Research Unit: BITCOIN-035

---

## Research Brief

```yaml
knowledge_id: BITCOIN-035
title: Bitcoin in 2026
research_question: >
  What does Bitcoin look like on August 4, 2026 from a protocol,
  implementation, operations, and institutional-research perspective, and how
  should analysts separate stable consensus properties from current software,
  policy, and ecosystem conditions that can continue to evolve?
document_type: current-state synthesis
difficulty: L400
prerequisites:
  - BITCOIN-027
  - BITCOIN-030
  - BITCOIN-031
  - BITCOIN-032
  - BITCOIN-033
  - BITCOIN-034
parent: Modern Bitcoin
previous: BITCOIN-034
next: BITCOIN-036
related_topics:
  - SegWit
  - Taproot
  - Lightning
  - Mempool Policy
  - Bitcoin Core Releases
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
  - Price prediction
  - Jurisdiction-specific regulation analysis
  - Exchange market-share ranking
  - Mining-company equity analysis
  - Comprehensive survey of every wallet or Lightning implementation
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- 2026년 8월 4일 기준 Bitcoin의 현재 technical posture를 설명할 수 있다.
- stable base-layer property와 빠르게 변하는 implementation/policy detail을 구분할 수 있다.
- 최근 Bitcoin Core change 중 operationally 중요한 부분을 식별할 수 있다.
- SegWit, Taproot, Lightning가 2026년 Bitcoin stack에서 어떤 자리를 차지하는지 설명할 수 있다.
- current-state analysis에서 node-local policy와 software version이 왜 여전히 중요한지 설명할 수 있다.

---

## 2. 핵심 질문

1. 2026년 Bitcoin에서 무엇이 여전히 stable한가?
2. Bitcoin의 어떤 부분이 여전히 actively evolving한가?
3. 2026년 8월 4일 기준 현재 Bitcoin Core release state는 무엇인가?
4. SegWit, Taproot, Lightning는 2026년 Bitcoin 그림에서 어디에 위치하는가?
5. 현재 어떤 policy 또는 implementation change가 operator와 analyst에 materially 영향을 주는가?
6. "오늘의 Bitcoin"을 설명할 때 institution은 무엇을 과장하면 안 되는가?

---

## 3. Executive Summary

2026년 8월 4일 화요일 기준, Bitcoin은 여전히 작업증명(Proof of Work, PoW), UTXO model, 누적 작업량(Chainwork) 기반 chain selection, local node validation을 중심으로 하는 보수적인 base-layer monetary and settlement protocol이다. 이 foundation은 바뀌지 않았다.[^ref-btc-core-files][^ref-bips-repo]

변한 것은 base layer 주변의 operational surface다. SegWit와 Taproot는 이제 현대 Bitcoin transaction capability의 확립된 일부이며, Lightning는 Bitcoin Core에 내장된 기능이 아니라 별도의 off-chain payment-layer ecosystem으로 남아 있다. 현재 Bitcoin Core release는 mempool policy, privacy-related broadcast behavior, chainstate handling, operator interface를 계속 다듬고 있다.[^ref-bip-0141][^ref-bip-0341][^ref-bolt-000][^ref-btc-core-31-0][^ref-btc-core-31-1]

official project site에서 2026년 8월 4일에 확인되는 최신 stable Bitcoin Core release는 2026년 7월 8일 공개된 Bitcoin Core 31.1이다.[^ref-btc-core-homepage][^ref-btc-core-31-1] 2026년 4월 20일 공개된 Bitcoin Core 31.0은 cluster mempool, 새로운 `-privatebroadcast` behavior, fee-estimation behavior 변화, 새 index와 RPC, default `-dbcache` 변경을 도입했다.[^ref-btc-core-31-0] 이후 2026년 6월 6일, project는 `-privatebroadcast` 기능이 특정 조건에서 originator의 IP address를 노출할 수 있다고 공지했고, 31.1에서 이 문제가 수정됐다.[^ref-btc-core-homepage][^ref-btc-core-31-1]

분석가와 institution에게 핵심 교훈은 "2026년의 Bitcoin"이 단지 base protocol만을 뜻하지 않는다는 점이다. 그것은 다음의 결합이다.

- stable consensus rule
- current client release behavior
- node-local policy
- layered payment system
- custody, fee, observability를 둘러싼 operational discipline

---

## 4. Protocol Structure

### Stable Core vs Moving Edge

2026년의 Bitcoin은 두 개의 동심원으로 이해하는 것이 가장 쉽다.

```text
stable consensus core
-> proof of work
-> UTXO validation
-> script and witness rules
-> chain selection by work

moving operational edge
-> mempool policy
-> relay behavior
-> RPC surfaces
-> indexes
-> wallet features
-> layered systems such as Lightning
```

### Why This Distinction Matters

관측자는 종종 "Bitcoin changed"라고 말하지만, 실제로는 client policy나 operator interface만 바뀐 경우가 많다. 많은 경우 consensus core는 전혀 바뀌지 않았다.

### 2026 Layer Inventory

| Layer | 2026 Characterization |
|---|---|
| Consensus base layer | 보수적이며 continuity-focused |
| Current reference implementation | 2026년 8월 4일 기준 official site에 올라온 Bitcoin Core 31.x series |
| Modern transaction capability | SegWit와 Taproot 사용 가능 |
| Off-chain scaling layer | Lightning는 별도 protocol family로 존재 |
| Operator policy surface | 특히 mempool과 relay 주변에서 여전히 진화 중 |

---

## 5. Stable Foundations in 2026

### Consensus Still Anchors Everything

Bitcoin Core의 source-tree documentation은 여전히 Bitcoin Core를, P2P network에 연결해 block과 transaction을 다운로드하고 fully validate하는 software로 설명한다.[^ref-btc-core-files] 이 표현은 오늘도 핵심 모델을 정확히 포착한다. 각 node는 centralized server에 위임하지 않고 locally validate한다.

### Base-Layer Conservatism

BIPs repository는 여전히 proposal의 게시와 보관 수단으로 기능하며, repository에 게시되었다고 해서 community consensus나 임박한 adoption을 의미하지 않는다고 명시한다.[^ref-bips-repo] 이는 2026년에도 Bitcoin이 protocol change에 신중하다는 중요한 governance signal이다.

### What Has Not Changed

다음은 개념적으로 안정적으로 유지되고 있다.

- 작업증명 기반 ordering
- UTXO spending model
- 누적 작업량에 따른 block-by-block settlement finality
- local validation의 필요성
- consensus와 policy의 차이

---

## 6. Modern Bitcoin Transaction Stack

### SegWit Is Part of Normal Bitcoin

BIP141은 여전히 Segregated Witness의 1차 자료이며, transaction serialization과 block-resource accounting을 weight를 통해 재구성한 consensus-layer extension으로 남아 있다.[^ref-bip-0141] 2026년 시점에서 SegWit는 더 이상 "새 기능"이 아니다. 현대 Bitcoin transaction environment의 일반적인 일부다.

### Taproot Is Also Part of the Modern Baseline

BIP341은 Schnorr 기반 output과 script path를 위한 Taproot spending rule과 signature-validation context를 정의한다.[^ref-bip-0341] 2026년에 Taproot를 무시하는 분석은 현대 Bitcoin을 제대로 설명하지 못한다.

### Lightning Remains a Separate Layer

Lightning BOLTs introduction은 오늘도 Lightning를 Bitcoin 위에 올라가는 별도의 protocol suite로 정의한다. base node implementation의 built-in feature가 아니다.[^ref-bolt-000] 따라서 Lightning adoption은 base-layer consensus change를 의미하지 않는다.

---

## 7. Bitcoin Core Release State on August 4, 2026

### Current Official Release Status

official Bitcoin Core site는 다음을 보여준다.

- Bitcoin Core 31.1 released on July 8, 2026
- Bitcoin Core 30.3 released on July 8, 2026
- Bitcoin Core 29.4 released on July 13, 2026[^ref-btc-core-homepage]

즉, 2026년 8월 4일 기준 newest visible major-line stable release는 31.1이다.

### Why 31.1 Matters

31.1 release note는 normal operation 동안 repeated large chainstate rewrite가 excessive disk read/write를 일으키던 문제를 수정했고, `-privatebroadcast` feature의 IP address leak도 수정했다고 밝힌다.[^ref-btc-core-31-1]

### Why 31.0 Still Matters

31.0은 연구 관점에서 더 구조적으로 중요하다. 다음과 같은 operator-visible shift를 도입했기 때문이다.

- cluster mempool
- private broadcast control
- embedded `asmap` data
- fee estimation 변화
- 새로운 RPC 및 REST surface
- 새 `-txospenderindex`
- 더 높은 기본 `-dbcache`
- 일부 deprecated setting 제거[^ref-btc-core-31-0]

---

## 8. Current Policy Evolution

### Cluster Mempool

Bitcoin Core 31.0은 mempool이 새로운 "cluster mempool" design으로 재구현되었다고 설명한다. 목적은 block template construction, eviction, relay, RBF validation 개선이다.[^ref-btc-core-31-0] 이것은 policy/implementation change이지 consensus change가 아니다.

### Replacement and Package Reasoning

31.0 note는 replacement validation이 resulting mempool의 feerate diagram이 이전보다 엄격히 좋아질 것을 요구하며, transaction ordering은 함께 채굴될 것으로 예상되는 chunk의 feerate에 따라 결정된다고 설명한다.[^ref-btc-core-31-0]

### Fee Estimator Update

31.0은 fee estimator의 minimum tracked bucket도 1 sat/vB에서 0.1 sat/vB로 낮췄다. 이는 node의 기본 `minrelaytxfee`와 맞춘 것이다.[^ref-btc-core-31-0] institution에게 중요하다는 것은, 오래된 가정 위에 세운 fee data model이 금방 낡을 수 있다는 뜻이다.

### The Analytical Point

operator나 researcher가 2026년 mempool behavior를 과거 mental model과 비교할 때는, 실제로 서로 다른 policy engine을 비교하고 있는 것은 아닌지 먼저 확인해야 한다.

---

## 9. Privacy and Relay in 2026

### `-privatebroadcast`

Bitcoin Core 31.0은 `sendrawtransaction`을 Tor 또는 I2P privacy network를 통해서만 broadcast하도록 하는 option을 도입했다.[^ref-btc-core-31-0]

### Security Disclosure and Fix

2026년 6월 6일 official site는 이 새 feature가 특정 상황에서 sender의 IP address를 드러낼 수 있다고 공지했다.[^ref-btc-core-homepage] 이후 31.1 release note는 해당 IP leak가 수정되었다고 밝힌다.[^ref-btc-core-31-1]

### Why This Matters

이 사례는 current-state discipline의 좋은 예다.

- protocol은 바뀌지 않았다.
- implementation surface는 바뀌었다.
- privacy expectation은 수정돼야 했다.
- release-version awareness가 operationally 중요해졌다.

---

## 10. Wallet, RPC, and Operator Surface

### Core Still Ships a Wallet and Interfaces

Bitcoin Core의 source-tree overview는 여전히 `bitcoind`, `bitcoin-qt`, `bitcoin-cli`, `bitcoin-wallet`을 주요 binary로 나열한다.[^ref-btc-core-files]

### Operator-Facing Changes in 31.0

31.0은 다음과 같은 operator-visible surface를 추가하거나 변경했다.

- `getmempoolcluster`
- `getmempoolfeeratediagram`
- `gettxspendingprevout` 추가 기능
- `getblock`의 coinbase transaction object 지원
- 새로운 block-part REST endpoint
- `-txospenderindex` 지원[^ref-btc-core-31-0]

### Institutional Consequence

2026년의 현대 Bitcoin analysis는 increasingly 다음에 의존한다.

- software-version awareness
- index configuration awareness
- node-local RPC output의 신중한 해석

---

## 11. Security Assumptions and Failure Modes

### Stable Assumption Set

2026년의 Bitcoin도 여전히 다음을 가정한다.

- node의 honest validation
- accepted rule 뒤에 충분한 economic weight
- 충분한 작업증명 security
- 안전한 custody practice
- functioning P2P propagation

### Contemporary Failure Surfaces

그러나 현재 운영은 여전히 다음으로 실패할 수 있다.

- release-specific bug
- node misconfiguration
- wallet exposure
- stale fee assumption
- topology bias
- local mempool view에 대한 과신

### Version-Specific Reality

2026년의 `-privatebroadcast` 이슈는 현대 Bitcoin risk가 consensus break에만 있는 것이 아님을 보여준다. current release의 implementation detail도 risk surface다.[^ref-btc-core-homepage][^ref-btc-core-31-1]

---

## 12. Mathematical or Economic Model

### Stability vs Change Decomposition

2026년의 Bitcoin을 단순화하면 다음처럼 생각할 수 있다.

```text
observed Bitcoin behavior
= consensus rules
+ implementation behavior
+ local policy
+ operator configuration
+ layered protocol usage
```

### Analytical Asymmetry

consensus-valid state를 `C`, node `i`의 local operational view를 `N_i`라 하면:

```text
N_i is a projection of C plus local policy state
```

즉, `N_i`는 whole network의 완전한 설명이 아니다.

### Economic Interpretation

현재의 fee, relay, wallet behavior는 asset의 monetary issuance schedule을 바꾸지 않으면서도 operational economics에는 영향을 준다. 그래서 장기 monetary policy는 고정되어 있어도, 단기 user experience는 크게 달라질 수 있다.

---

## 13. Bitcoin Core Implementation

### Reference Implementation Context

Bitcoin Core는 2026년 현재 Bitcoin operation을 이해하기 위한 main implementation anchor로 남아 있다.[^ref-btc-core-files]

### 31.x as the Current Operator Baseline

2026년 8월 4일 기준 official site는 31.1을 current newest stable major release로 보여준다.[^ref-btc-core-homepage][^ref-btc-core-31-1] 따라서 implementation claim을 현재 release note가 문서화한 material policy/operational change를 무시한 채 과거 release assumption에 고정해 두면 안 된다.

### Implementation Areas That Matter Most in 2026

검토한 source를 기준으로, 현재 가장 중요한 implementation area는 다음이다.

- mempool과 relay policy
- privacy-aware broadcast behavior
- chainstate efficiency와 durability
- index와 RPC 확장
- 현대적인 fee-estimation behavior[^ref-btc-core-31-0][^ref-btc-core-31-1]

---

## 14. On-Chain Implications

### What the Chain Still Shows

blockchain은 여전히 다음을 직접 기록한다.

- confirmed transaction history
- confirmed fee payment
- confirmed block
- on-chain Taproot 및 SegWit activity
- Lightning가 base layer에 닿을 때의 channel open과 close

### What It Still Does Not Show

blockchain은 다음을 보여주지 않는다.

- local cluster-mempool state
- private broadcast queue
- wallet configuration
- internal exchange ledgering
- 대부분의 Lightning state transition
- operator-specific peer/policy decision

### 2026 Analytical Warning

누군가 "2026년의 Bitcoin"을 on-chain data만으로 설명한다면, 현재 operational picture의 큰 부분을 빠뜨리고 있는 것이다.

---

## 15. Institutional Thinking

institution은 2026년의 Bitcoin을 성숙했지만 여전히 operationally active한 시스템으로 해석해야 한다.

### Practical Implications

- consensus는 보수적이지만, policy와 tooling은 계속 진화한다.
- Bitcoin Core release note는 optional reading이 아니라 primary operational research material이다.
- fee, relay, mempool analytics는 version-aware하게 해석돼야 한다.
- Lightning는 base-layer settlement constraint가 사라졌다는 증거가 아니라, 별도의 operational layer로 다뤄야 한다.
- privacy 또는 broadcasting claim은 오래된 가정이 아니라 current release 및 security disclosure 기준으로 점검해야 한다.

### Better Institutional Posture

2026년 8월 4일 기준 올바른 posture는 다음과 같다.

- locally validate한다.
- release-specific behavior를 추적한다.
- consensus와 policy를 분리한다.
- local node output을 partial view로 취급한다.
- current implementation behavior를 timeless protocol law로 바꾸어 말하지 않는다.

---

## 16. Common Misinterpretations

### "Bitcoin in 2026 is just the same as Bitcoin in 2021"

틀렸다. consensus core는 연속적이지만, operator surface, mempool logic, relay behavior, transaction tooling은 변했다.

### "Bitcoin Core 31.x changes mean Bitcoin consensus changed"

틀렸다. 여기서 인용한 31.x change는 주로 implementation, policy, interface, operational change다.

### "Lightning is now part of Bitcoin Core"

틀렸다. Lightning는 여전히 Bitcoin 위의 별도 protocol family다.[^ref-bolt-000]

### "Current Bitcoin privacy claims can be repeated without version context"

틀렸다. `-privatebroadcast`를 둘러싼 2026년 6월 6일 공지는 왜 version context가 중요한지를 직접 보여준다.[^ref-btc-core-homepage]

### "One current node tells me what the whole network is doing"

틀렸다. 하나의 node는 software-mediated local view만을 보여준다.

---

## 17. Research Questions

1. cluster mempool은 empirical transaction visibility와 miner-template interpretation을 얼마나 바꾸었는가?
2. current RPC addition 중 어떤 것이 institutional surveillance 및 reconciliation workflow를 가장 크게 개선하는가?
3. modern Core policy가 과거 version과 더 크게 달라진 지금, analyst는 historical mempool comparison을 어떻게 조정해야 하는가?
4. "2026년의 Bitcoin" operational reality 중 얼마나 많은 부분이 chain-only dataset에 여전히 보이지 않는가?
5. 향후 adoption될 수 있는 next protocol 또는 policy proposal 중, 이 2026년 스냅샷을 가장 크게 바꿀 것은 무엇인가?

---

## 18. Practical Exercises

### Exercise 1

Bitcoin consensus rule과 Bitcoin Core 31.0 mempool-policy change를 짧게 비교하라.

### Exercise 2

Bitcoin Core 31.1이 2026년 7월 8일에 release된 것이, consensus rule change가 없더라도 왜 중요한지 설명하라.

### Exercise 3

chain-only data가 아니라 node-level 또는 release-level context가 필요한 observation 다섯 가지를 나열하라.

### Exercise 4

2026년 Bitcoin stack에서 SegWit, Taproot, Lightning의 위치를 하나의 diagram으로 설명하라.

---

## 19. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Current official Bitcoin Core release timing and recent disclosures | Directly specified | Official Bitcoin Core website |
| 31.0 and 31.1 behavioral changes | Directly specified | Official release notes |
| SegWit, Taproot, and Lightning role in the modern stack | Directly specified plus inference | BIPs and BOLT intro, with 2026 synthesis |
| Version-aware institutional implications | Inference from sources | Derived from current implementation and operational changes |

---

## 20. Knowledge Graph

```text
Bitcoin in 2026
├─ Stable Core
│  ├─ proof of work
│  ├─ UTXO model
│  ├─ local validation
│  └─ cumulative-work settlement
├─ Modern Base Layer
│  ├─ SegWit
│  └─ Taproot
├─ Layered Systems
│  └─ Lightning
├─ Current Core Operations
│  ├─ Bitcoin Core 31.1
│  ├─ cluster mempool
│  ├─ private broadcast
│  ├─ fee estimator changes
│  └─ chainstate fixes
└─ Institutional Lens
   ├─ version awareness
   ├─ node-local visibility
   ├─ policy vs consensus
   └─ operational discipline
```

---

## 21. 참고문헌

### Primary Sources

[^ref-btc-core-homepage]: Bitcoin Core official website, recent posts including Bitcoin Core 31.1 released on July 8, 2026, Bitcoin Core 30.3 released on July 8, 2026, Bitcoin Core 29.4 released on July 13, 2026, and the June 6, 2026 `-privatebroadcast` disclosure, https://bitcoincore.org/, accessed 2026-08-04.

[^ref-btc-core-31-1]: Bitcoin Core Contributors, "Bitcoin Core 31.1" release notes, published July 8, 2026, including chainstate rewrite fix and `-privatebroadcast` IP leak fix, https://bitcoincore.org/en/releases/31.1/, accessed 2026-08-04.

[^ref-btc-core-31-0]: Bitcoin Core Contributors, "Bitcoin Core 31.0" release notes, published April 20, 2026, including cluster mempool, `-privatebroadcast`, `-txospenderindex`, fee-estimation changes, and operator-surface updates, https://bitcoincore.org/en/releases/31.0/, accessed 2026-08-04.

[^ref-btc-core-files]: Bitcoin Core Contributors, `doc/files.md`, source-tree and binary overview, Bitcoin Core master branch, https://github.com/bitcoin/bitcoin/blob/master/doc/files.md, accessed 2026-08-04.

[^ref-bip-0141]: BIP141, "Segregated Witness (Consensus layer)," Bitcoin BIPs repository, https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki, accessed 2026-08-04.

[^ref-bip-0341]: BIP341, "Taproot: SegWit version 1 spending rules," Bitcoin BIPs repository, https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki, accessed 2026-08-04.

[^ref-bolt-000]: Lightning BOLTs, BOLT #0 Introduction and Index, Lightning specification repository, https://github.com/lightning/bolts/blob/master/00-introduction.md, accessed 2026-08-04.

[^ref-bips-repo]: Bitcoin BIPs repository, BIP process and repository description, https://github.com/bitcoin/bips, accessed 2026-08-04.

### Supporting Interpretation Notes

- Statements about "Bitcoin in 2026" as a synthesis are partly interpretive because they combine stable protocol documents with current release-state evidence from August 4, 2026.

---

## 22. 교차 참조

### Previous

- BITCOIN-034 — Institutional Perspective on Bitcoin

### Next

- BITCOIN-036

### Related

- BITCOIN-027 — Fee Market
- BITCOIN-030 — SegWit
- BITCOIN-031 — Taproot
- BITCOIN-032 — Lightning Network
- BITCOIN-033 — Bitcoin Core
- BITCOIN-034 — Institutional Perspective on Bitcoin

---

## Review Status

### Technical Review

Passed.

- stable consensus property와 current implementation/policy behavior를 분리했다.
- 31.0과 31.1 change를 operator surface 및 implementation update로 한정했다.
- SegWit, Taproot, Lightning를 modern Bitcoin stack 안에서 올바르게 배치했다.
- current-state claim이 중요한 부분에는 절대 날짜를 사용했다.

### Evidence Review

Passed.

- current release timing과 disclosure claim은 official Bitcoin Core site에 근거한다.
- 31.0과 31.1의 feature/fix 설명은 official release note에 근거한다.
- SegWit, Taproot, Lightning 설명은 BIP와 BOLT source에 근거한다.
- institutional interpretation은 필요한 곳에서 inference로 라벨링했다.

### Editorial Review

Passed.

- 구조는 프로젝트 deep-dive format을 따른다.
- metadata는 완전하다.
- terminology는 consensus, policy, cluster mempool, private broadcast, version-aware analysis로 일관된다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 current Bitcoin Core behavior를 timeless consensus rule과 혼동하지 않는다.
- Lightning를 Bitcoin Core의 일부로 취급하지 않는다.
- chain-only observability를 과장하지 않는다.
- 2026년 7월 8일 release date와 2026년 8월 4일 document date의 차이를 숨기지 않는다.
- current implementation detail을 영구적인 protocol destiny로 제시하지 않는다.

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
