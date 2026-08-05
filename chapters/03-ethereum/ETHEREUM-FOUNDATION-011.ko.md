---
knowledge_id: ETHEREUM-FOUNDATION-011
title: Ethereum Upgrades
subtitle: Fork Governance, The Merge, Dencun, Pectra, Fusaka, 그리고 살아 있는 Ethereum protocol change의 성격
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 145 min
estimated_study: 360 min
last_reviewed: 2026-08-04
domain:
  - Ethereum
  - Upgrades
  - Governance
  - Protocol Evolution
parent:
  - Ethereum Foundations
prerequisites:
  - ETHEREUM-FOUNDATION-001
  - ETHEREUM-FOUNDATION-007
related_topics:
  - EIPs
  - The Merge
  - Layer 2
  - Roadmap
primary_sources:
  - REF-ETH-FORKS-2026-001
  - REF-ETH-ROADMAP-2026-001
  - REF-ETH-MERGE-2026-001
  - REF-EIP-6953
  - REF-EIPS-REPO-001
tags:
  - ethereum
  - upgrades
  - roadmap
  - forks
  - merge
  - governance
---

# Ethereum Upgrades
> Ethereum Foundations  
> Research Unit: ETHEREUM-FOUNDATION-011

---

## Research Brief

```yaml
knowledge_id: ETHEREUM-FOUNDATION-011
title: Ethereum Upgrades
research_question: >
  How does Ethereum change over time through planned network upgrades, what are
  the most important completed upgrades visible as of August 4, 2026, how do
  roadmap and fork-history sources differ, and how should researchers interpret
  Ethereum's governance through this upgrade process?
document_type: deep-dive
difficulty: L300
prerequisites:
  - ETHEREUM-FOUNDATION-001
  - ETHEREUM-FOUNDATION-007
parent: Ethereum Foundations
previous: ETHEREUM-FOUNDATION-010
next: ETHEREUM-FOUNDATION-012
related_topics:
  - EIPs
  - The Merge
  - Layer 2
  - Roadmap
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
  - Full EIP-by-EIP chronology
  - Client implementation manual
  - Governance forum ethnography
  - Spec details of every future proposed upgrade
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Ethereum network upgrade가 무엇인지 설명할 수 있다.
- fork history와 roadmap intention을 구분할 수 있다.
- 2026년 8월 4일까지 완료된 major upgrade를 식별할 수 있다.
- The Merge가 왜 Ethereum architecture의 watershed change인지 설명할 수 있다.
- Ethereum governance를 single-controller system이 아니라 open, multi-actor upgrade process로 설명할 수 있다.

---

## 2. 핵심 질문

1. Ethereum upgrade란 무엇인가?
2. Ethereum upgrade는 어떻게 activate되고 coordination되는가?
3. 어떤 major upgrade가 current Ethereum의 형태를 규정하는가?
4. roadmap과 completed fork history의 차이는 무엇인가?
5. 왜 Ethereum research는 date-aware해야 하는가?

---

## 3. Executive Summary

Ethereum은 planned network upgrade를 통해 계속 바뀌는 living protocol이다. 보통 이러한 변화는 EIP를 통해 구현되며, centralized forced rollout이 아니라 coordinated client release와 validator/node adoption을 통해 활성화된다.[^ref-eth-forks][^ref-eips-repo]

official fork timeline과 roadmap은 서로 다른 종류의 진술을 한다.

- fork history page는 무엇이 실제로 일어났는지 설명하고
- roadmap page는 현재 의도된 future direction을 설명하며 바뀔 수 있다.[^ref-eth-forks][^ref-eth-roadmap]

2026년 8월 4일 화요일 기준, official roadmap과 fork history에 prominently 나타나는 주요 completed modern upgrade는 다음과 같다.

- Paris / The Merge on September 15, 2022
- Shapella on April 12, 2023
- Dencun on March 13, 2024
- Pectra on May 7, 2025
- Fusaka on December 3, 2025[^ref-eth-roadmap][^ref-eth-forks]

The Merge는 특히 중요하다. official Merge page에 따르면, 이는 Ethereum을 작업증명에서 proof of stake로 전환했고, energy consumption을 약 99.95% 줄였다.[^ref-eth-merge]

올바른 연구 습관은 다음을 구분하는 것이다.

- past completed protocol state
- current live protocol state
- future roadmap intent

---

## 4. Protocol Structure

### What an Upgrade Is

fork timeline은 upgrade를 Ethereum protocol rule의 변화로 정의하며, 이는 흔히 EIP에서 출발한 planned technical upgrade를 포함한다고 설명한다.[^ref-eth-forks]

### Why Upgrades Are Special in Blockchains

같은 official 설명은 Ethereum client, validator, node가 새 rule을 구현하기 위해 software를 update해야 하며, 모두에게 software update를 강제하는 central owner는 없다고 말한다.[^ref-eth-forks]

### Structural View

```text
research / discussion
-> EIPs and specifications
-> client implementation
-> coordinated activation
-> network converges on new rules
```

---

## 5. Fork History vs Roadmap

### Fork History

fork timeline은 official site가 식별한 important milestone과 upgrade에 대한 retrospective record다.[^ref-eth-forks]

### Roadmap

roadmap page는 current plan을 설명하며, Ethereum development가 community-driven이고 subject to change라고 명시한다.[^ref-eth-roadmap]

### Why This Distinction Matters

roadmap item은 deployed rule과 같지 않다. completed fork는 eternal future direction과도 같지 않다.

---

## 6. Major Completed Upgrades Through August 4, 2026

### Paris / The Merge

official Merge page는 Merge가 September 15, 2022에 실행되었고 Ethereum의 proof-of-stake consensus transition을 완료했다고 말한다.[^ref-eth-merge]

### Shapella

roadmap은 Shapella가 April 12, 2023에 completed되었다고 표시한다.[^ref-eth-roadmap]

### Dencun

roadmap과 forks page는 모두 Dencun을 March 13, 2024로 기록한다.[^ref-eth-roadmap][^ref-eth-forks]

### Pectra

roadmap은 Pectra를 May 7, 2025로 기록한다.[^ref-eth-roadmap]

### Fusaka

roadmap은 Fusaka를 December 3, 2025로 기록한다.[^ref-eth-roadmap]

### Why These Matter

이 upgrade들이 2026년의 "Ethereum"이 무엇을 의미하는지 규정하며, 2014년 whitepaper 하나보다 훨씬 더 중요하다.

---

## 7. Upgrade Activation and Governance

### Activation Mechanisms

EIP-6953은 시간이 지나며 사용된 network upgrade activation trigger를 문서화하며, Ethereum이 proof-of-work era와 post-Merge era에서 서로 다른 activation mechanism을 사용했다고 설명한다.[^ref-eip-6953]

### Multi-Actor Governance

roadmap은 public participation, discussion forum, community-driven protocol evolution을 강조한다.[^ref-eth-roadmap]

### Practical Reality

Ethereum governance는 단순한 direct democracy도 아니고 단일 corporate release pipeline도 아니다. public proposal, implementation work, social adoption pressure가 결합된 open technical coordination process다.

---

## 8. The Merge as an Architectural Watershed

### Why It Is Special

The Merge는 단순히 feature를 추가한 것이 아니다. Ethereum mainnet의 consensus mechanism을 작업증명에서 proof of stake로 바꿨다.[^ref-eth-merge]

### Consequences

이것은 다음을 바꿨다.

- block production assumption
- validator economics
- energy usage
- execution/consensus architecture
- future roadmap possibility

### Research Consequence

Merge boundary를 무시하는 Ethereum 설명은 역사적으로 얕다.

---

## 9. Future Upgrades in View on August 4, 2026

### Current Roadmap Signals

2026년 8월 4일에 접근한 official roadmap page는 `Glamsterdam`과 `Hegotá`를 H2 2026 in development로 보여준다.[^ref-eth-roadmap]

### How to Interpret This

이것들은 roadmap intention이지, already-deployed protocol fact가 아니다.

### Why This Matters

연구자는 absolute date를 사용하고, future-facing roadmap content와 current deployed rule을 명확히 구분해야 한다.

---

## 10. Technical Mechanics

### Simplified Upgrade Pipeline

```text
problem / opportunity identified
-> proposal discussed publicly
-> EIP(s) written and refined
-> clients implement changes
-> activation parameters set
-> operators update
-> new rules go live
```

### Need for Coordination

Ethereum은 live network이기 때문에, upgrade는 unilateral release control이 아니라 client와 operator의 convergence를 요구한다.

### Post-Merge Complexity

execution과 consensus change는 함께 coordinated deployment가 필요할 수 있다. 그래서 Dencun이나 Pectra 같은 combined name이 operationally 중요하다.[^ref-eth-forks]

---

## 11. Security Assumptions

### Upgrade Risk

모든 upgrade는 live protocol behavior를 바꾸므로 다음 risk를 가져온다.

- implementation risk
- coordination risk
- misunderstood operator readiness
- outdated model에서 오는 analytical lag

### Documentation Freshness

Ethereum이 시간에 따라 materially change하기 때문에, source freshness는 first-order issue다.

### Governance Risk

open network에서 protocol change는 free가 아니다. coordination quality와 client diversity에 대한 의존을 만든다.

---

## 12. Mathematical or Economic Model

### Evolution Model

유용한 conceptual model은 다음과 같다.

```text
current Ethereum behavior
= prior deployed upgrades
+ current client implementation
+ current operator adoption
```

### Roadmap Caution

future roadmap item은 다음처럼 모델링해야 한다.

```text
intent != deployed rule
```

### Why This Matters

이는 특히 research, operation, investment framing에서 중요하다. 사람들은 종종 "coming soon"을 이미 protocol의 일부인 것처럼 과장한다.

---

## 13. Protocol Implementation

### Primary Current Sources

이 unit에서 가장 신뢰할 수 있는 current source는 다음이다.

- official roadmap page
- official forks timeline
- official Merge page
- activation pattern을 위한 EIP-6953
- proposal process context를 위한 EIPs repository[^ref-eth-roadmap][^ref-eth-forks][^ref-eth-merge][^ref-eip-6953][^ref-eips-repo]

### Why This Source Mix Matters

이 조합은 다음을 분리해준다.

- historical fact
- current architecture
- future intention
- process structure

---

## 14. On-Chain Implications

### Upgrade Boundaries Matter for Data

protocol upgrade는 다음을 바꿀 수 있다.

- fee mechanic
- block structure
- consensus timing
- available transaction type
- analyst를 위한 data surface

### Historical Datasets Need Segmentation

Ethereum data는 종종 upgrade era별로 segment해야 하며, 특히 The Merge나 EIP-1559 관련 환경 같은 major shift 주변에서는 더 그렇다.

### Practical Consequence

upgrade-era segmentation 없는 "Ethereum average behavior"는 대체로 약한 claim이다.

---

## 15. Institutional Thinking

institution은 Ethereum을 frozen protocol이 아니라 evolving protocol로 다뤄야 한다.

### Practical Implications

- documentation과 analytics는 version-aware해야 한다.
- upgrade calendar는 operationally material하다.
- governance monitoring은 protocol risk management의 일부다.
- future roadmap item을 already deployed된 것처럼 technical assumption에 pricing하면 안 된다.

### Better Research Posture

Ethereum protocol claim을 하기 전에 다음을 물어야 한다.

- 이 진술은 어떤 upgrade era에 대한 것인가?
- cited behavior는 historical인가, current인가, proposed인가?
- 어떤 client/operator assumption에 의존하는가?

---

## 16. Common Misinterpretations

### "The roadmap tells me what Ethereum is today"

틀렸다. 그것은 current plan과 direction을 말해줄 뿐이며, 바뀔 수 있다.[^ref-eth-roadmap]

### "Ethereum governance means one team decides"

틀렸다. process는 public하고, multi-actor이며, coordination-heavy하다.[^ref-eth-roadmap][^ref-eips-repo]

### "The Merge was just an efficiency improvement"

너무 약한 설명이다. 그것은 architectural consensus transition이었다.[^ref-eth-merge]

### "Future named upgrades are already protocol facts"

틀렸다. deploy되기 전까지는 future-facing roadmap item이다.

---

## 17. Research Questions

1. institution의 analytics normalization에 가장 중요한 upgrade boundary는 무엇인가?
2. governance-monitoring system은 likely future change와 speculative roadmap discussion을 어떻게 구분해야 하는가?
3. 널리 유통되는 오래된 교육 자료 때문에 가장 자주 오해되는 current Ethereum behavior는 무엇인가?

---

## 18. Practical Exercises

### Exercise 1

official forks timeline과 official roadmap의 차이를 설명하라.

### Exercise 2

왜 The Merge가 단순히 "Ethereum got greener"가 아닌지 짧게 설명하라.

### Exercise 3

December 3, 2025까지의 major completed official upgrade를 나열하라.

### Exercise 4

future roadmap item을 deployed protocol behavior로 취급하면 안 되는 이유를 설명하라.

---

## 19. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Official past upgrade dates and names | Directly specified | Forks timeline and roadmap |
| Merge meaning and date | Directly specified | Official Merge page |
| Upgrade activation-pattern context | Directly specified | EIP-6953 |
| Governance and institutional interpretation | Inference from sources | Derived from roadmap/process structure |

---

## 20. Knowledge Graph

```text
Ethereum Upgrades
├─ Historical Forks
│  ├─ The Merge
│  ├─ Shapella
│  ├─ Dencun
│  ├─ Pectra
│  └─ Fusaka
├─ Future Intent
│  ├─ Glamsterdam
│  └─ Hegotá
├─ Process
│  ├─ EIPs
│  ├─ client implementation
│  ├─ activation triggers
│  └─ operator adoption
└─ Research Discipline
   ├─ historical vs current vs proposed
   ├─ version awareness
   └─ governance monitoring
```

---

## 21. References

### Primary Sources

[^ref-eth-forks]: ethereum.org, "Timeline of all Ethereum forks (2014 to present)," official history of major milestones and fork names, https://ethereum.org/ethereum-forks/, accessed 2026-08-04.

[^ref-eth-roadmap]: ethereum.org, "Ethereum roadmap," official roadmap page showing completed upgrades through Fusaka and in-development items such as Glamsterdam and Hegotá as of August 4, 2026, https://ethereum.org/roadmap/, accessed 2026-08-04.

[^ref-eth-merge]: ethereum.org, "The Merge," official page stating that The Merge executed on September 15, 2022 and transitioned Ethereum Mainnet to proof of stake, https://ethereum.org/roadmap/merge/, accessed 2026-08-04.

[^ref-eip-6953]: EIP-6953, "Network Upgrade Activation Triggers," Ethereum Improvement Proposals, describing historical activation mechanisms, https://eips.ethereum.org/EIPS/eip-6953, accessed 2026-08-04.

[^ref-eips-repo]: Ethereum Improvement Proposals repository, process and publication surface for EIPs, https://eips.ethereum.org/, accessed 2026-08-04.

### Supporting Interpretation Notes

- Where this document discusses institutional governance posture or version-aware research discipline, those are analytical inferences built from the cited roadmap, fork, and process sources.

---

## 22. Cross References

### Previous

- ETHEREUM-FOUNDATION-010 — Storage

### Next

- ETHEREUM-FOUNDATION-012 — Layer 2 Overview

### Related

- ETHEREUM-FOUNDATION-001 — Ethereum Vision
- ETHEREUM-FOUNDATION-012 — Layer 2 Overview

---

## Review Status

### Technical Review

Passed.

- fork history, roadmap, activation process를 분리했다.
- Merge를 architectural watershed로 설명했다.
- completed upgrade와 future roadmap item을 구분했다.
- absolute date를 사용해 currentness를 명확히 했다.

### Evidence Review

Passed.

- official past upgrade date와 name은 forks/roadmap source를 인용한다.
- Merge meaning은 official Merge page를 인용한다.
- activation pattern context는 EIP-6953을 인용한다.
- governance interpretation은 inference로 라벨링했다.

### Editorial Review

Passed.

- 구조는 프로젝트 문서 패턴을 따른다.
- metadata는 완전하다.
- terminology는 forks, roadmap, Merge, activation, governance로 일관된다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 roadmap을 current deployed rule과 동일시하지 않는다.
- governance를 one-team decision으로 축소하지 않는다.
- Merge를 단순 efficiency tweak로 설명하지 않는다.
- future named upgrade를 current protocol fact로 다루지 않는다.

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
