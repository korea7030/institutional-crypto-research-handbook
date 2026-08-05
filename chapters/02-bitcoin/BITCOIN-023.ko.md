---
knowledge_id: BITCOIN-023
title: Forks
subtitle: 일시적 체인 분기, 합의 규칙 변경, soft fork, hard fork, activation, 그리고 분석 경계
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 110 min
estimated_study: 320 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Consensus
  - Forks
  - Governance
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-021
  - BITCOIN-022
  - POW-010
  - POW-011
  - POW-012
  - POW-013
related_topics:
  - Soft Fork
  - Hard Fork
  - Chain Split
  - Reorganization
  - Version Bits
  - Activation
  - Chainwork
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-BLOCKCHAIN-001
  - REF-BTC-DEV-OPERATING-MODES-001
  - REF-BIP-0016
  - REF-BIP-0034
  - REF-BIP-0009
  - REF-BIP-0341
  - REF-BTC-CORE-CHAIN-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-BIPS-001
tags:
  - bitcoin
  - internals
  - consensus
  - forks
  - soft-forks
  - hard-forks
  - activation
  - reorg
---

# Forks
> Bitcoin Internals  
> Research Unit: BITCOIN-023

---

## Research Brief

```yaml
knowledge_id: BITCOIN-023
title: Forks
research_question: >
  What kinds of forks occur in Bitcoin, how do temporary chain splits differ
  from consensus-rule forks, how do soft forks and hard forks change validity
  sets, how does activation interact with miner and node behavior, and how does
  Bitcoin Core converge on one active chain when valid branches compete?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-021
  - BITCOIN-022
  - POW-010
  - POW-011
  - POW-012
  - POW-013
parent: Bitcoin Internals
previous: BITCOIN-022
next: BITCOIN-024
related_topics:
  - Chain Selection
  - Reorganizations
  - Activation Mechanisms
  - Miner Signaling
  - SPV Security
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
  - Altcoin governance comparisons
  - Social history of every Bitcoin upgrade
  - Detailed politics of specific activation disputes
  - Full chain reorganization implementation walkthrough
  - Non-Bitcoin scripting roadmap debates
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- 일시적 blockchain fork와 consensus-rule fork를 구분할 수 있다.
- simultaneous block discovery가 왜 short-lived competing tip을 만드는지 설명할 수 있다.
- Bitcoin이 cumulative work를 사용해 valid competing branch를 어떻게 해소하는지 설명할 수 있다.
- soft fork를 validity-set restriction으로, hard fork를 validity-set expansion으로 정의할 수 있다.
- soft fork는 non-upgraded validator에 대한 호환성을 유지할 수 있지만 hard fork는 그렇지 못한 이유를 설명할 수 있다.
- flag day와 version-bits signaling 같은 activation mechanism의 역할을 설명할 수 있다.
- fork와 reorganization을 구분할 수 있다.
- on-chain data만으로 unreleased private fork를 드러낼 수 없는 이유를 설명할 수 있다.
- Bitcoin Core의 chain-selection과 activation 관련 code surface를 식별할 수 있다.

---

## 2. 핵심 질문

1. Bitcoin에서 fork란 무엇인가?
2. rule change 없이도 temporary chain split은 어떻게 생기는가?
3. branch, fork point, reorganization의 차이는 무엇인가?
4. soft fork란 무엇인가?
5. hard fork란 무엇인가?
6. soft fork는 왜 backward compatible할 수 있지만 hard fork는 그렇지 않은가?
7. activation mechanism은 enforcement timing을 어떻게 바꾸는가?
8. miner signaling이란 무엇이며, 무엇을 증명하지 못하는가?
9. 왜 fork는 local, transient, permanent일 수 있는가?
10. fork event를 해석할 때 발생하는 주요 분석 오류는 무엇인가?

---

## 3. Executive Summary

Bitcoin에서 "fork"라는 단어는 최소 두 가지 다른 현상을 가리킨다. 첫째, 하나의 공통 부모 이후 여러 valid block이 경쟁하면서 blockchain history가 실제로 갈라지는 현상이다. 둘째, consensus rule change로 인해 서로 다른 software population 사이에서 validity judgment가 갈라질 수 있는 상황이다.[^ref-btc-wp] [^ref-btc-dev-blockchain]

백서는 기본적인 temporary-fork case를 설명한다. 두 node가 동시에 서로 다른 block을 찾으면, 어떤 node는 하나를 먼저 받고 다른 node는 다른 것을 먼저 받는다. miner는 일시적으로 서로 다른 branch 위에서 작업하고, 그중 한 branch가 다음 block을 얻으면 node는 더 길거나 더 어려운 valid chain으로 수렴한다.[^ref-btc-wp]

현대 문서는 valid-block set이 어떻게 바뀌는지에 따라 hard fork와 soft fork를 구분한다. Bitcoin Developer 문서는 hard fork가 upgraded node에게는 valid하지만 non-upgraded node에게는 invalid인 block을 만들 수 있어 영구적 chain divergence를 초래할 수 있다고 설명한다. soft fork는 upgraded node에게 valid-block set을 더 좁게 만든다. upgraded miner가 충분한 hash rate를 통제하면 non-upgraded node도 upgraded branch를 best valid chain으로 따라갈 수 있다.[^ref-btc-dev-blockchain]

따라서 fork는 다음으로 나눠 봐야 한다.

- Temporary valid branch competition
- Incompatible consensus rule로 인한 permanent divergence
- software population이 future block을 다르게 해석할 수 있는 activation window

분석가에게 이 구분은 중요하다. 모든 visible split이 governance event는 아니고, 모든 governance event가 즉시 public chain split으로 나타나는 것도 아니기 때문이다.

---

## 4. 프로토콜 구조

### Branch Competition으로서의 Fork

가장 단순한 fork는 공통 조상 이후 branch가 나뉘는 것이다.

```text
common ancestor
├─ branch A tip
└─ branch B tip
```

이는 모든 참여자가 동일한 합의 규칙을 실행하더라도 일어날 수 있다. 원인은 propagation delay와 거의 동시에 발생한 block discovery일 수 있다.

### Rule Change로서의 Fork

같은 단어가 validity rule 자체의 변경을 의미하기도 한다. 이 경우 질문은 "어느 valid branch가 먼저 도착했는가?"가 아니라 "어떤 block을 어떤 software population이 valid로 보느냐?"다.

### 세 가지 개념의 구분

다음 용어를 혼동하면 안 된다.

| Concept | Core Meaning |
|---|---|
| Fork | branch divergence 또는 rule-set divergence |
| Fork point | competing branch의 마지막 공통 조상 |
| Reorganization | fork point 이후 active chain을 다른 branch로 전환하는 것 |

즉, fork는 reorg 없이 존재할 수 있고, reorg는 이미 fork point가 존재함을 전제한다.

---

## 5. Rule Change 없는 Temporary Fork

### Simultaneous Block Discovery

두 miner가 같은 parent 위에서 거의 동시에 valid block을 찾았다고 하자. network propagation은 즉시 일어나지 않으므로, 어떤 node는 block `A`를 먼저 보고 다른 node는 block `B`를 먼저 본다.

```text
H
├─ A
└─ B
```

`A`와 `B`는 모두 동일한 합의 규칙 아래에서 valid할 수 있다. 이 불일치는 temporary하고 topological할 뿐, normative disagreement는 아니다.

### More Work로 해소

다음 valid block이 `A`를 연장하면, `A+1` branch는 `B` branch보다 더 많은 cumulative work를 얻게 된다. 이를 학습한 node는 `A` branch를 best valid chain candidate로 보고 `B`를 active tip에서 버린다.[^ref-btc-wp] [^ref-btc-dev-blockchain]

그래서 temporary fork는 consensus failure의 증거가 아니라 distributed proof-of-work consensus의 자연스러운 부산물이다.

### Public Branch와 Private Branch

모든 fork가 존재하는 순간 public하게 보이는 것은 아니다.

- Public temporary fork: 두 tip이 모두 broadcast되어 관측 가능
- Private fork: 한 branch가 withheld되었다가 나중에 release되거나 끝내 공개되지 않음

온체인 분석은 public record만 볼 수 있을 뿐, 존재했을 수도 있는 모든 private branch를 볼 수는 없다.

---

## 6. Soft Fork와 Hard Fork

### Set-Theoretic Framing

다음을 두자.

- `V_old` = old rule에서 valid한 block 집합
- `V_new` = new rule에서 valid한 block 집합

그러면:

- Soft fork: `V_new`는 `V_old`의 부분집합
- Hard fork: `V_old`는 `V_new`의 부분집합

이 framing이 가장 깔끔하다.

### Soft Fork

soft fork는 consensus를 더 엄격하게 만든다. upgraded node는 old node가 허용했을 block 일부를 거부한다. 하지만 upgraded node가 허용한 모든 block은 old rule도 만족하므로, old node는 stronger branch가 유지되는 한 upgraded chain을 계속 따라갈 수 있다.[^ref-btc-dev-blockchain]

현재 Bitcoin documentation에 반영된 deployed soft-fork mechanism 또는 rule change의 예:

- BIP16 pay-to-script-hash enforcement[^ref-bip-0016] [^ref-btc-core-bips]
- BIP34 coinbase height rule[^ref-bip-0034] [^ref-btc-core-bips]
- 병렬 soft-fork deployment를 위한 BIP9 version-bits activation framework[^ref-bip-0009] [^ref-btc-core-bips]
- BIP341의 Taproot consensus rule[^ref-bip-0341]

### Hard Fork

hard fork는 consensus를 완화하거나 바꾸어, upgraded node에게 valid한 block 일부가 non-upgraded node에게는 invalid가 되게 만든다. 네트워크가 실제로 그런 block을 사용하면 non-upgraded node는 그 chain을 valid로 따라갈 수 없으므로 permanent divergence가 가능해진다.[^ref-btc-dev-blockchain]

이것이 backward compatibility가 핵심 운영 차이인 이유다.

### 중요한 정리

"이 proposal은 hard fork가 필요하다"는 말은 chain split이 이미 일어났다는 뜻이 아니다. 해당 rule change가 universal upgrade 없이 사용되면 incompatible validity judgment가 생긴다는 뜻이다.

---

## 7. Activation과 Enforcement

### Activation은 Timing 문제다

consensus rule change는 rule definition만으로 끝나지 않는다. 언제부터 node가 새 rule을 어기는 block을 거부할 것인지라는 enforcement schedule이 필요하다.

### Flag-Day Activation

일부 historical soft fork는 특정 time이나 height에서 활성화되었다. 이 모델에서는 activation point가 오면, miner가 널리 readiness를 signal했는지와 무관하게 upgraded node가 새 rule을 집행하기 시작한다.[^ref-btc-dev-blockchain]

### Miner-Signaled Activation

BIP9는 여러 soft-fork deployment가 block-version bit로 readiness를 signal하고 threshold 기반 activation window를 쓸 수 있게 했다.[^ref-bip-0009] [^ref-btc-core-bips]

중요한 분석 한계:

- signaling은 rule 자체가 아니다.
- signaling은 의도의 보장이 아니다.
- signaling은 모든 economic actor가 change를 지지한다는 증거가 아니다.

이는 특정 mechanism 아래의 activation input일 뿐이다.

### Post-Activation Reality

일단 active가 되면 rule은 validating node가 집행한다. active soft-fork rule을 어기는 miner는 upgraded validator가 거부하는 block을 만들 위험을 진다.

즉, activation은 governance 언어가 끝나고 consensus enforcement가 시작되는 지점이다.

---

## 8. Technical Mechanics

### Last Common Ancestor

Bitcoin Core는 parent link를 가진 block index entry를 기준으로 fork 구조를 모델링한다. 핵심 구조 질문은 "두 branch가 어디서 갈라지는가?"이며, `LastCommonAncestor`는 두 tip 사이의 fork point를 찾아 준다.[^ref-btc-core-chain]

### Best-Chain Activation

더 나은 valid candidate branch가 알려지면, Core의 best-chain logic은 fork point를 찾고, 더 이상 active branch가 아닌 block을 disconnect하고, 더 나은 branch의 block을 connect한다.[^ref-btc-core-validation]

추상적으로는 다음과 같다.

```text
find fork point
disconnect old active branch after fork point
connect better valid branch after fork point
set new tip
```

이 절차가 fork와 reorg가 관련은 있지만 동일하지 않은 이유다.

### Validity Level

Bitcoin Core는 block-index entry가 점차 더 깊은 validity check를 만족했는지 추적한다. competing branch는 script level까지 fully validated되기 전에 구조적으로 먼저 알려질 수 있다. 따라서 header가 존재한다고 해서 competing branch가 모두 동등한 것은 아니다.[^ref-btc-core-chain] [^ref-btc-core-validation]

### SPV와 Fork Awareness

operating-mode 문서는 SPV client가 모든 block content를 full validation하는 대신 header와 cumulative difficulty를 추적한다고 설명한다.[^ref-btc-dev-operating-modes]

즉, SPV client는 header level에서 branch competition을 인지할 수는 있지만, 더 강한 security judgment를 위해서는 full node와 accumulated work 가정에 의존한다.

---

## 9. Validation Boundaries

### "Fork Detected"가 "Consensus Failed"를 뜻하지는 않는다

short-lived competing tip은 Nakamoto consensus에서 정상적이다. 다음일 때 더 깊은 운영상 우려가 된다.

- divergence가 지속될 때
- branch depth가 커질 때
- incompatible rule population이 존재할 때
- losing branch에서 finality를 가정한 경우

### "Soft Fork"가 "No Risk"를 뜻하지는 않는다

soft fork는 stronger branch가 여전히 old rule 아래에서도 valid할 때만 non-upgraded node에 호환 경로를 제공한다. activation window 동안에는 miner behavior, node adoption, enforcement timing이 여전히 중요하다.

### "Hard Fork"가 "Immediate Permanent Split"을 뜻하지는 않는다

hard fork는 incompatible validity judgment를 만들지만, lasting public split이 실제로 일어나는지는 adoption, mining, user behavior에 달려 있다.

---

## 10. Security Assumptions and Failure Modes

### Temporary Fork Risk

temporary fork가 중요한 이유:

- stale block을 만든다.
- 아주 최근 confirmation을 되돌릴 수 있다.
- merchant와 service에 short-horizon settlement risk를 노출한다.

### Rule-Change Risk

consensus rule change는 추가 failure mode를 도입한다.

- split signaling과 enforcement expectation
- validator subset에 invalid한 block을 miner가 생산
- non-upgraded infrastructure가 다른 view를 따름
- software population 간 chain fragmentation

### Private-Fork Uncertainty

작업증명은 withheld branch를 불가능하게 만드는 것이 아니라 비싸게 만든다. 분석가는 unreleased private fork를 직접 관찰할 수 없으므로, 리스크 평가에서 다음을 구분해야 한다.

- 이미 네트워크에서 관찰된 public branch competition
- incentive나 attack model로부터 추론된 hypothetical hidden competition

### Measurement Risk

competing block에 대한 local receipt는 topology와 timing에 의존한다. 어떤 observer는 다른 observer가 기록한 short-lived public fork를 아예 보지 못할 수도 있다. 따라서 fork dataset은 vantage dependent하다.

---

## 11. Mathematical or Economic Model

### Fork Probability Intuition

block arrival을 Poisson process로, propagation에 finite time이 든다고 보면, discovery overlap은 가끔 생길 수밖에 없다. propagation window `T_p` 동안 aggregate discovery rate `lambda`에서 적어도 하나의 competing discovery가 발생할 확률은:

```text
P(competing discovery) = 1 - e^(-lambda * T_p)
```

이는 단순화지만, propagation delay가 줄어들수록 temporary-fork frequency가 줄어든다는 점을 설명한다.

### Validity-Set Model

consensus-rule fork는 timing race보다 set relation으로 보는 편이 낫다.

```text
soft fork  => V_new ⊂ V_old
hard fork  => V_old ⊂ V_new
```

activation을 둘러싼 정치 과정과 무관하게 이 framing은 유효하다.

### Economic Implications

fork는 다음을 통해 incentive를 바꾼다.

- stale-block risk
- confirmation reliability
- miner revenue variance
- infrastructure upgrade cost
- wallet/exchange operational complexity

짧게 지속된 fork라도 사용자가 losing branch의 transaction에 따라 행동했다면 경제적 의미가 있다.

---

## 12. Bitcoin Core 구현

### `chain.h`

`chain.h`는 `CBlockIndex`, ancestor traversal, block-proof helper, `LastCommonAncestor`를 정의한다. 이는 fork point와 chain comparison을 다루는 구조적 primitive다.[^ref-btc-core-chain]

### `validation`

Bitcoin Core의 validation layer에는 active chain을 most-work valid branch로 옮기는 best-chain activation logic이 있다. detected fork가 실제 active-chain switch가 되는 구현 surface가 여기다.[^ref-btc-core-validation]

### `doc/bips.md`

Bitcoin Core의 `doc/bips.md`는 BIP9, BIP16, BIP34 같은 deployed soft-fork mechanism을 포함한 implemented BIP index를 제공한다. 이는 abstract fork taxonomy를 현재 Core documentation의 concrete deployed rule change와 연결하는 데 유용하다.[^ref-btc-core-bips]

---

## 13. Consensus, Policy, and Presentation

### Consensus Layer

consensus는 어떤 block이 valid하고, 어떤 valid chain이 더 많은 cumulative work를 갖는지 결정한다.

### Policy Layer

policy는 relay, mempool admission, mining template choice에 영향을 주지만, soft fork나 hard fork 자체를 정의하지는 않는다. standardness policy change를 consensus change와 혼동하는 것은 흔한 분석 오류다.

### Presentation Layer

사용자, explorer, 내부 system이 보는 것은 timing에 따라 달라진다.

- competing branch를 알기 전
- 두 branch가 모두 보이는 동안
- 한 branch가 active가 된 뒤
- 나중의 reorganization으로 visible history가 확정된 뒤

presentation lag 때문에 이미 해소된 temporary fork도 갑작스러운 transaction reversal처럼 보일 수 있다.

---

## 14. 온체인 함의

### Observable Public Fork

두 branch가 모두 public하게 알려지면 분석가는 다음을 식별할 수 있다.

- fork point
- branch length
- block timestamp
- chainwork 차이
- publicly survived branch

### Unobservable Private Competition

한 branch가 release 전까지 withheld되었거나 끝내 공개되지 않았다면, 온체인 history만으로 그 branch의 전체 존재 시간을 알 수 없다.

### Confirmation Semantics

losing branch의 block에 있던 transaction은 한때 confirmed처럼 보이다가 active chain view에서 사라질 수 있다. 이 때문에 fork 분석은 reorg 분석과 운영상 연결되지만 동일하지는 않다.

---

## 15. Institutional Thinking

기관은 세 가지 질문을 분리해야 한다.

1. branch competition이 있었는가?
2. 그것이 ordinary propagation 때문인가, incompatible rule adoption 때문인가?
3. 어떤 business process가 losing branch에서 너무 이르게 finality를 가정했는가?

### Practical Implications

- exchange와 custodian은 매우 최근 confirmation을 fork-sensitive하게 다뤄야 한다.
- monitoring system은 competing-tip event와 rule-change event를 별도로 분류해야 한다.
- version signaling을 네트워크 consensus의 완전한 proxy로 취급하면 안 된다.
- upgrade 이후 risk model은 software release date가 아니라 actual enforcement와 active-chain outcome을 추적해야 한다.

---

## 16. Common Misinterpretations

### "Fork는 거버넌스 분쟁이다"

틀렸다. 많은 fork는 rule disagreement가 없는 ordinary short-lived branch competition이다.

### "Hard Fork와 Chain Split은 같다"

틀렸다. hard fork는 incompatibility class이고, lasting public split은 그 가능한 결과 중 하나다.

### "Soft Fork면 모두 완벽히 동기화된다"

틀렸다. activation window와 enforcement timing은 여전히 temporary divergence와 운영 리스크를 만들 수 있다.

### "Longest Chain Rule은 Block Count가 가장 많은 체인을 뜻한다"

틀렸다. 현대 Bitcoin 분석에서는 raw block count가 아니라 most cumulative work valid chain을 말해야 한다.[^ref-btc-dev-blockchain] [^ref-btc-dev-operating-modes]

### "Miner Signaling이 Social Consensus를 증명한다"

틀렸다. 특정 activation mechanism 아래의 관측 가능한 입력 중 하나일 뿐이다.

---

## 17. Research Questions

1. 현재 propagation condition에서 public temporary fork는 얼마나 자주 발생하는가?
2. 서로 다른 network vantage point에서 수집한 public fork dataset은 얼마나 차이가 나는가?
3. block version bit는 actual enforcement readiness를 얼마나 잘 대변하는가?
4. 기관은 settlement-risk model에서 private-fork uncertainty를 어떻게 정량화해야 하는가?
5. 어떤 historical incident가 propagation race보다 rule-compatibility failure로 더 잘 설명되는가?

---

## 18. Practical Exercises

### Exercise 1

orphaned block이 포함된 chain dataset을 사용해 fork point를 식별하고, 해소 전 branch depth를 측정하라.

### Exercise 2

알려진 soft-fork deployment 하나를 골라 activation mechanism, signaling window, 실제 enforcement 시작 날짜를 비교하라.

### Exercise 3

temporary public fork 하나를 매핑하고 다음을 구분하라.

- fork point
- competing branch
- active-chain switch
- stale branch outcome

### Exercise 4

하나의 soft fork와 하나의 hypothetical hard fork에 대해 old/new validity set 관계를 set diagram으로 그려 보라.

---

## 19. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Simultaneous-block temporary fork model | Directly specified | Whitepaper and Developer Guide |
| Soft fork and hard fork definitions | Directly specified | Developer Guide and BIPs |
| Activation and version-bits framework | Directly specified | BIP9 and Bitcoin Core docs |
| Fork-point and active-chain implementation mechanics | Directly specified | Bitcoin Core `chain` and `validation` references |
| Private-fork observability limits and institutional risk framing | Inference from sources | Derived from consensus and propagation structure |

---

## 20. Knowledge Graph

```text
Forks
├─ Temporary Forks
│  ├─ simultaneous discovery
│  ├─ propagation delay
│  ├─ competing valid branches
│  └─ stale blocks
├─ Consensus Rule Forks
│  ├─ soft forks
│  ├─ hard forks
│  ├─ activation
│  └─ software population divergence
├─ Resolution
│  ├─ cumulative chainwork
│  ├─ last common ancestor
│  ├─ active-chain switch
│  └─ reorganization
├─ Evidence
│  ├─ public branch competition
│  ├─ version signaling
│  └─ header / block observations
└─ Risks
   ├─ settlement reversal
   ├─ stale revenue
   ├─ split validity
   └─ private-fork uncertainty
```

---

## 21. 참고문헌

### Primary Sources

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Sections 5 and 11. https://bitcoin.org/bitcoin.pdf
[^ref-btc-dev-blockchain]: Bitcoin Developer Guide, "Block Chain," including fork, soft-fork, hard-fork, and chain-selection discussion. https://developer.bitcoin.org/devguide/block_chain.html
[^ref-btc-dev-operating-modes]: Bitcoin Developer Guide, "Operating Modes," including full-node and SPV chain-validation discussion. https://developer.bitcoin.org/devguide/operating_modes.html
[^ref-bip-0016]: BIP16, "Pay to Script Hash." https://github.com/bitcoin/bips/blob/master/bip-0016.mediawiki
[^ref-bip-0034]: BIP34, "Block v2, Height in Coinbase." https://github.com/bitcoin/bips/blob/master/bip-0034.mediawiki
[^ref-bip-0009]: BIP9, "Version bits with timeout and delay." https://github.com/bitcoin/bips/blob/master/bip-0009.mediawiki
[^ref-bip-0341]: BIP341, "Taproot: SegWit version 1 spending rules." https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki
[^ref-btc-core-chain]: Bitcoin Core Doxygen, `chain.h`, including `CBlockIndex`, `GetAncestor`, block-proof helpers, and `LastCommonAncestor`. https://doxygen.bitcoincore.org/chain_8h_source.html
[^ref-btc-core-validation]: Bitcoin Core Doxygen, `validation.h`, including `ActivateBestChain` and `ActivateBestChainStep`. https://doxygen.bitcoincore.org/validation_8h_source.html
[^ref-btc-core-bips]: Bitcoin Core `doc/bips.md`, implemented BIP index. https://github.com/bitcoin/bitcoin/blob/master/doc/bips.md

### Supporting Interpretation Notes

- Where this document discusses institutional measurement limits, private-fork uncertainty, or signaling interpretation, those statements are inferences from the documented consensus, chain-selection, and activation architecture rather than explicit protocol claims.

---

## 22. 교차 참조

### Previous

- BITCOIN-022 — Nodes and Network Propagation

### Next

- BITCOIN-024 — Chain Reorganization

### Related

- BITCOIN-006 — Whitepaper Section 5 — Network
- BITCOIN-012 — Whitepaper Section 11 — Calculations
- BITCOIN-013 — Whitepaper Section 12 — Conclusion
- BITCOIN-021 — Blocks and Block Headers
- BITCOIN-022 — Nodes and Network Propagation
- POW-010 — Longest Chain Rule and Fork Resolution
- POW-011 — Cumulative Chainwork
- POW-012 — Difficulty Adjustment
- POW-013 — Bitcoin Core Proof-of-Work Validation
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- temporary chain fork, consensus-rule fork, reorganization을 분리했다.
- soft fork와 hard fork 정의를 validity-set 관점으로 표현했다.
- activation timing과 version-bits signaling을 rule content 자체와 구분했다.
- Bitcoin Core fork-point/best-chain reference는 직접 관련된 chain/validation source로 제한했다.

### Evidence Review

Passed.

- temporary-fork behavior는 백서와 Developer Guide에 연결했다.
- soft-fork와 hard-fork 정의는 Developer Guide와 deployed BIP에 연결했다.
- activation discussion은 BIP9와 Bitcoin Core implemented-BIP documentation에 연결했다.
- best-chain implementation은 `chain.h`, `validation.h`에 연결했다.
- analytical interpretation은 필요한 부분에서 inference로 표시했다.

### Editorial Review

Passed.

- structure는 프로젝트 deep-dive format을 따른다.
- metadata가 완전하다.
- fork, branch, fork point, reorganization 용어가 일관된다.
- table과 code fence가 닫혀 있다.

### Adversarial Review

Passed.

- 모든 fork를 social dispute와 동일시하지 않았다.
- miner signaling만으로 consensus가 증명된다고 하지 않았다.
- "longest chain"을 block count만으로 축소하지 않았다.
- 모든 private fork가 public하게 관찰된다고 주장하지 않았다.
- soft-fork compatibility를 zero operational risk로 보지 않았다.

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
