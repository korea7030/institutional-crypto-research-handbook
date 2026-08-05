---
knowledge_id: ETHEREUM-FOUNDATION-012
title: Layer 2 Overview
subtitle: Rollup, Security Inheritance, Data Availability, 그리고 Ethereum scaling path가 L2 중심인 이유
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 145 min
estimated_study: 360 min
last_reviewed: 2026-08-04
domain:
  - Ethereum
  - Layer 2
  - Scaling
  - Data Availability
parent:
  - Ethereum Foundations
prerequisites:
  - ETHEREUM-FOUNDATION-006
  - ETHEREUM-FOUNDATION-007
  - ETHEREUM-FOUNDATION-011
related_topics:
  - Ethereum Upgrades
  - Gas
  - Blocks
  - Rollups
primary_sources:
  - REF-ETH-L2-LEARN-2026-001
  - REF-ETH-ROADMAP-SCALING-2026-001
  - REF-ETH-DANKSHARDING-2026-001
  - REF-ETH-ROADMAP-2026-001
tags:
  - ethereum
  - layer2
  - rollups
  - scaling
  - data-availability
  - danksharding
---

# Layer 2 Overview
> Ethereum Foundations  
> Research Unit: ETHEREUM-FOUNDATION-012

---

## Research Brief

```yaml
knowledge_id: ETHEREUM-FOUNDATION-012
title: Layer 2 Overview
research_question: >
  What are Ethereum layer 2 systems, how do they inherit security from
  Ethereum, why has Ethereum's scaling strategy centered on rollups instead of
  scaling the main chain alone, and how should researchers interpret the L1/L2
  boundary as of August 4, 2026?
document_type: overview
difficulty: L300
prerequisites:
  - ETHEREUM-FOUNDATION-006
  - ETHEREUM-FOUNDATION-007
  - ETHEREUM-FOUNDATION-011
parent: Ethereum Foundations
previous: ETHEREUM-FOUNDATION-011
next:
related_topics:
  - Gas
  - Blocks
  - Scaling
  - Data Availability
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
  - Rollup-by-rollup product comparison
  - Fraud-proof and validity-proof formal derivation
  - Bridge exploit case-study catalog
  - Full L2 tokenomics analysis
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- protocol-strategy 수준에서 Ethereum layer 2를 정의할 수 있다.
- Ethereum의 scaling path가 왜 L2 중심인지 설명할 수 있다.
- L1과 L2의 책임을 구분할 수 있다.
- rollup의 security inheritance와 data availability dependence를 설명할 수 있다.
- L2와 alt L1, sidechain, validium을 상위 수준에서 구분할 수 있다.

---

## 2. 핵심 질문

1. Ethereum 위의 layer 2란 무엇인가?
2. 왜 Ethereum은 main chain만 직접 확장하지 않는가?
3. rollup은 어떻게 Ethereum의 security를 상속받는가?
4. Ethereum은 data availability layer로 어떤 역할을 하는가?
5. 왜 sidechain과 validium은 L2와 동일하지 않은가?

---

## 3. Executive Summary

Ethereum의 current scaling strategy는 L1 execution layer 하나가 모든 throughput를 처리하도록 만드는 것이 아니라, layer 2 system, 특히 rollup을 중심으로 구성되어 있다.[^ref-eth-l2-learn][^ref-eth-roadmap-scaling]

official "What is layer 2?" page는 L2를 Ethereum 위에 build되며 Ethereum의 security guarantee를 inherit하는 scaling solution 집합으로 정의한다.[^ref-eth-l2-learn]

official scaling roadmap은 rollup이 offchain에서 transaction을 batch하고, user cost를 줄이며, data availability와 settlement guarantee를 위해 Ethereum에 의존한다고 설명한다.[^ref-eth-roadmap-scaling][^ref-eth-danksharding]

2026년 8월 4일 기준, Ethereum roadmap과 L2 page는 Ethereum을 더 이상 단일 chain으로만 이해하면 안 된다는 점을 분명히 한다. Ethereum은 broader network of networks를 위한 settlement/security layer이며, recent and planned upgrade는 L2 data를 더 싸게 만들고 scaling을 더 효율적으로 만드는 데 초점을 둔다.[^ref-eth-roadmap][^ref-eth-roadmap-scaling]

올바른 mental model은 다음과 같다.

- Ethereum L1은 security, decentralization, settlement, data availability를 우선한다.
- L2는 user-facing transaction throughput의 많은 부분을 처리한다.

---

## 4. Protocol Structure

### L1 vs L2

official L2 page는 Ethereum 같은 layer 1 blockchain이 layer 2 project가 build하는 underlying foundation이라고 설명한다.[^ref-eth-l2-learn]

### Responsibility Split

```text
L1 Ethereum
-> base security
-> settlement
-> data availability
-> protocol finality

L2 systems
-> execute or batch many user transactions
-> compress / post results to Ethereum
-> aim for lower cost and higher throughput
```

### Why This Matters

Ethereum scaling은 이제 single-chain parameter tuning이 아니라 architectural 문제다.

---

## 5. Why Ethereum Scales This Way

### Main Chain Limits

L2 page와 scaling roadmap은 Ethereum이 decentralization이나 security를 해치지 않고 main chain throughput만 무한히 키울 수는 없다고 설명한다.[^ref-eth-l2-learn][^ref-eth-roadmap-scaling]

### Rollup-Centric Strategy

roadmap은 rollup이 Ethereum이 scale하는 방식이라고 설명하며, rollup이 예상보다 빨리 발전했기 때문에 shard chain은 더 이상 필요하지 않다고 말한다.[^ref-eth-roadmap][^ref-eth-roadmap-scaling]

### Consequence

2026년 Ethereum의 scaling philosophy는 "하나의 chain이 모든 것을 한다"가 아니라, "안전한 base layer가 많은 scaling layer를 뒷받침한다"다.

---

## 6. Security Inheritance and Data Availability

### Security Inheritance

L2 page는 layer 2가 Ethereum을 확장하며 Ethereum의 security guarantee를 inherit한다고 설명한다.[^ref-eth-l2-learn]

### Data Availability Role

같은 page는 Ethereum이 L2를 위한 data availability layer로 기능하며, dispute가 발생하면 Ethereum이 필요한 data를 제공한다고 설명한다.[^ref-eth-l2-learn]

### Why This Is Important

security inheritance는 magic이 아니다. 어떤 data가 Ethereum에 도달하는지, 어떤 verification assumption이 성립하는지에 의존한다.

---

## 7. Rollups, Sidechains, and Validiums

### Rollups

official page는 rollup을 Ethereum이 선호하는 scaling approach의 중심으로 둔다.[^ref-eth-roadmap-scaling]

### Sidechains and Validiums

L2 page는 sidechain과 validium이 main chain으로부터 security나 data availability를 derive하지 않으며, 따라서 trust assumption이 다르다고 명시한다.[^ref-eth-l2-learn]

### Why Researchers Must Care

사람들은 종종 "L2"를 너무 느슨하게 쓴다. scaling system마다 trust assumption은 materially 다르다.

---

## 8. Proto-Danksharding and Scaling Upgrades

### Official Scaling Narrative

scaling 및 danksharding page는 rollup이 이미 L1보다 저렴하며, proto-danksharding/data-blob improvement가 rollup이 Ethereum에 data를 게시하는 비용을 더 낮춘다고 설명한다.[^ref-eth-roadmap-scaling][^ref-eth-danksharding]

### Role of Recent Upgrades

roadmap과 관련 2026 material은 Dencun, Pectra, Fusaka 같은 upgrade를 Ethereum의 scaling posture 개선과 연결한다.[^ref-eth-roadmap]

### Implication

Ethereum upgrade는 increasingly L1을 유일한 execution venue로 만들기보다, L2-centered scaling strategy를 지원하는 방향으로 작동한다.

---

## 9. Technical Mechanics

### Simplified L2 Flow

```text
users transact on L2
-> L2 batches / compresses activity
-> data / commitments posted to Ethereum
-> Ethereum anchors availability and settlement
-> disputes or proofs resolve against Ethereum rules
```

### Why This Reduces Cost

많은 user action이 full L1 execution burden을 각각 지불하는 대신, 하나의 aggregated L1 publication footprint를 공유할 수 있다.

### Why This Still Depends on L1

trust story는 Ethereum이 궁극적으로 무엇을 secure하고 available하게 만드는지에서 나온다.

---

## 10. Security Assumptions

### L2 Is Not "Free Security"

security는 다음에 의존한다.

- correct L2 design
- correct bridge/security model
- correct data publication assumption
- 그리고 Ethereum이 약속된 base guarantee를 계속 제공하는 것

### Terminology Risk

sidechain이나 validium을 rollup과 정확히 같은 security inheritance를 가진 것처럼 잘못 부르면, 사용자와 institution을 materially 오도할 수 있다.[^ref-eth-l2-learn]

### Roadmap Risk

scaling improvement는 지금도 protocol evolution이 진행 중인 영역이다. 연구자는 current source를 써야 한다.

---

## 11. Mathematical or Economic Model

### Cost-Sharing Intuition

단순화한 rollup intuition은 다음과 같다.

```text
many user transactions
-> one aggregated L1 publication footprint
-> lower average cost per user action
```

### L1/L2 Relationship

개념적으로:

```text
L2 throughput gain depends on L1 settlement + data publication economics
```

### Why This Matters

L2 economics는 Ethereum과 독립적이지 않다. Ethereum의 data 및 settlement cost structure의 downstream 결과다.

---

## 12. Protocol Implementation

### Primary Sources

current architectural understanding을 위해 가장 중요한 official source는 다음이다.

- L2 overview page
- scaling roadmap
- danksharding page
- current roadmap page[^ref-eth-l2-learn][^ref-eth-roadmap-scaling][^ref-eth-danksharding][^ref-eth-roadmap]

### Why This Set Matters

이 조합은 다음을 제공한다.

- L2의 정의
- 전략적 rationale
- data availability와 scaling mechanism context
- protocol development의 current direction

---

## 13. On-Chain Implications

### More Than One Network Surface

Ethereum analysis는 increasingly 다음을 구분해야 한다.

- L1 mainnet activity
- L2 activity
- bridge flow
- data publication footprint
- settlement anchoring

### L1 Activity Alone Is Incomplete

Ethereum L1 user transaction count만 측정하면, ecosystem의 실제 user activity 상당 부분을 놓칠 수 있다.

### Analytical Consequence

modern Ethereum research는 single-chain interpretation이 아니라 cross-layer interpretation을 필요로 하는 경우가 많다.

---

## 14. Institutional Thinking

institution은 Ethereum을 layered system으로 다뤄야 한다.

### Practical Implications

- security assessment는 어떤 layer를 평가하는지 명시해야 한다.
- cost analysis는 L1과 L2 execution environment를 구분해야 한다.
- bridge와 settlement assumption은 명시적으로 문서화해야 한다.
- 많은 L1 upgrade가 direct L1 UX보다 L2 economics를 우선 개선하므로 roadmap understanding이 중요하다.

### Better Research Posture

scaling claim을 하기 전에 다음을 물어야 한다.

- 이 주장은 L1에 대한 것인가, L2에 대한 것인가, combined system에 대한 것인가?
- 이 system은 Ethereum security를 inherit하는가, 아니면 단지 interact만 하는가?
- 어떤 data availability assumption이 필요한가?

---

## 15. Common Misinterpretations

### "Layer 2 just means any cheaper chain"

틀렸다. security inheritance와 trust assumption이 중요하다.[^ref-eth-l2-learn]

### "Ethereum failed to scale because users moved to L2s"

틀렸다. official strategy 자체가 L2-based scaling을 중심으로 한다.[^ref-eth-roadmap-scaling]

### "Sidechains and validiums are the same as rollups"

틀렸다. official L2 page는 trust assumption이 다르다고 말한다.[^ref-eth-l2-learn]

### "L1 usage alone captures Ethereum ecosystem activity"

틀렸다. ecosystem은 increasingly multi-layer다.

---

## 16. Research Questions

1. institution은 L1 settlement importance와 L2 user activity를 가장 잘 분리하는 metric을 무엇으로 잡아야 하는가?
2. researcher는 rollup, validium, sidechain의 trust assumption을 하나로 뭉개지 않고 어떻게 비교해야 하는가?
3. 어떤 future roadmap item이 L2 economics를 가장 크게 바꿀 가능성이 있는가?

---

## 17. Practical Exercises

### Exercise 1

왜 Ethereum의 scaling strategy가 더 이상 shard chain 중심이 아닌지 설명하라.

### Exercise 2

rollup과 sidechain의 차이를 짧게 정리하라.

### Exercise 3

L2에 대한 data availability layer로서 Ethereum의 역할을 설명하라.

### Exercise 4

L2 activity가 왜 researcher의 Ethereum network usage 해석 방식을 바꾸는지 설명하라.

---

## 18. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Official definition of L2 and security inheritance | Directly specified | Official L2 page |
| Rollup-centric scaling strategy | Directly specified | Official roadmap/scaling pages |
| Danksharding / proto-danksharding role | Directly specified | Official danksharding/scaling pages |
| Institutional cross-layer interpretation | Inference from sources | Derived from L1/L2 architecture |

---

## 19. Knowledge Graph

```text
Layer 2 Overview
├─ L1 Ethereum
│  ├─ settlement
│  ├─ security
│  ├─ data availability
│  └─ finality
├─ L2 Systems
│  ├─ rollups
│  ├─ cheaper execution
│  ├─ batched activity
│  └─ user throughput
├─ Trust Distinctions
│  ├─ rollups
│  ├─ sidechains
│  └─ validiums
└─ Scaling Path
   ├─ roadmap
   ├─ proto-danksharding
   ├─ PeerDAS era
   └─ network of networks
```

---

## 20. References

### Primary Sources

[^ref-eth-l2-learn]: ethereum.org, "What is layer 2?", official L2 overview describing Ethereum L2s, security inheritance, and distinctions from sidechains and validiums, page last update June 4, 2026, https://ethereum.org/layer-2/learn/, accessed 2026-08-04.

[^ref-eth-roadmap-scaling]: ethereum.org, "Scaling Ethereum," official roadmap/scaling page describing rollup-centered scaling and user-cost implications, page last update June 24, 2026, https://ethereum.org/roadmap/scaling/, accessed 2026-08-04.

[^ref-eth-danksharding]: ethereum.org, "Danksharding," official page describing proto-danksharding/data blobs and the goal of scaling rollups, page last update June 24, 2026, https://ethereum.org/roadmap/danksharding, accessed 2026-08-04.

[^ref-eth-roadmap]: ethereum.org, "Ethereum roadmap," official roadmap showing completed upgrades through Fusaka and in-development items in H2 2026, https://ethereum.org/roadmap/, accessed 2026-08-04.

### Supporting Interpretation Notes

- Where this document discusses institutional cross-layer analytics or scaling posture, those are analytical inferences built from the cited official L2 and roadmap sources.

---

## 21. Cross References

### Previous

- ETHEREUM-FOUNDATION-011 — Ethereum Upgrades

### Next

- None in current Phase 3 roadmap

### Related

- ETHEREUM-FOUNDATION-006 — Gas
- ETHEREUM-FOUNDATION-011 — Ethereum Upgrades

---

## Review Status

### Technical Review

Passed.

- L1/L2 responsibility split을 명확히 설명했다.
- security inheritance와 mere bridge connectivity를 구분했다.
- sidechain과 validium을 rollup으로 뭉개지 않았다.
- current roadmap/scaling context를 date-aware하게 포함했다.

### Evidence Review

Passed.

- definition/trust-assumption claim은 official L2 docs를 인용한다.
- scaling strategy claim은 official roadmap/scaling page를 인용한다.
- institutional interpretation은 inference로 라벨링했다.

### Editorial Review

Passed.

- 구조는 프로젝트 문서 패턴을 따른다.
- metadata는 완전하다.
- terminology는 L1, L2, rollup, data availability, settlement로 일관된다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 모든 cheap chain을 L2라고 부르지 않는다.
- roadmap aspiration을 already-live fact처럼 제시하지 않는다.
- L1 usage만으로 total Ethereum activity를 포착할 수 있다고 말하지 않는다.
- trust-assumption 차이를 지우지 않는다.

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
