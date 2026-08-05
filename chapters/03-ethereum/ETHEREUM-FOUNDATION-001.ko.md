---
knowledge_id: ETHEREUM-FOUNDATION-001
title: Ethereum Vision
subtitle: Ethereum이 왜 blockchain model을 디지털 머니를 넘어 general-purpose state machine으로 확장하려 했는가
version: 1.0.0
status: Reviewed
difficulty: L200
estimated_reading: 120 min
estimated_study: 300 min
last_reviewed: 2026-08-04
domain:
  - Ethereum
  - Smart Contracts
  - Protocol Design
  - Layer 1
parent:
  - Ethereum Foundations
prerequisites:
  - BLOCKCHAIN-FOUNDATION-008
  - BITCOIN-014
  - BITCOIN-033
related_topics:
  - Account Model
  - World State
  - EVM
  - Gas
  - Proof of Stake
  - Protocol Governance
primary_sources:
  - REF-ETH-WP-001
  - REF-ETH-DOC-INTRO-2026-001
  - REF-ETH-DOC-POS-2026-001
  - REF-ETH-YP-README-001
  - REF-EIPS-REPO-001
tags:
  - ethereum
  - vision
  - smart-contracts
  - evm
  - state-machine
  - proof-of-stake
---

# Ethereum Vision
> Ethereum Foundations  
> Research Unit: ETHEREUM-FOUNDATION-001

---

## Research Brief

```yaml
knowledge_id: ETHEREUM-FOUNDATION-001
title: Ethereum Vision
research_question: >
  What problem was Ethereum originally trying to solve, how did that vision
  differ from Bitcoin's narrower design, and how should researchers separate
  Ethereum's 2014 founding vision from the protocol and operational reality of
  Ethereum as of August 4, 2026?
document_type: foundation
difficulty: L200
prerequisites:
  - BLOCKCHAIN-FOUNDATION-008
  - BITCOIN-014
  - BITCOIN-033
parent: Ethereum Foundations
previous:
next: ETHEREUM-FOUNDATION-002
related_topics:
  - Smart Contracts
  - World State
  - Account Model
  - EVM
  - Gas
  - Proof of Stake
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
  - Full EVM opcode tutorial
  - Comprehensive Ethereum upgrade chronology
  - Layer 2 design survey
  - Smart contract language comparison
  - Token-standard deep dive
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Ethereum이 Bitcoin식 디지털 현금을 넘어 무엇을 달성하려 했는지 설명할 수 있다.
- Ethereum의 original whitepaper vision과 현대 Ethereum의 현실을 구분할 수 있다.
- Ethereum을 단순한 payment network가 아니라 general-purpose state machine으로 정의할 수 있다.
- smart contract, account, gas가 왜 Ethereum 설계의 중심인지 설명할 수 있다.
- 오늘날 Ethereum은 작업증명(Proof of Work, PoW)이 아니라 proof of stake로 설명해야 한다는 점을 설명할 수 있다.

---

## 2. 핵심 질문

1. 애초에 Ethereum은 왜 존재했는가?
2. Ethereum은 초기 blockchain의 어떤 한계를 해결하려 했는가?
3. Ethereum을 general-purpose blockchain이라고 부른다는 것은 무엇을 의미하는가?
4. 왜 smart contract와 world state가 Ethereum에서는 Bitcoin보다 더 중심적인가?
5. 연구자는 2014년의 vision과 2026년의 Ethereum을 어떻게 분리해서 봐야 하는가?

---

## 3. Executive Summary

Ethereum의 founding vision은 blockchain이라는 아이디어를, 좁게 규정된 asset-transfer system에서 벗어나 arbitrary application logic을 공유 네트워크 state machine 위에 직접 encode할 수 있는 programmable platform으로 일반화하는 것이었다.[^ref-eth-wp]

2014년 Ethereum whitepaper는 이것을 purpose-specific blockchain의 한계에 대한 해답으로 제시한다. use case마다 새로운 chain을 만드는 대신, Ethereum은 built-in programming language를 가진 blockchain 하나로 여러 application을 공통 base layer 위에서 지원하자고 제안했다.[^ref-eth-wp]

이 비전은 2026년에도 방향성 자체는 유효하지만, 그대로 인용해서는 안 된다. original whitepaper는 launch 이전에 작성되었으며, official Ethereum site도 10년이 넘는 개발과 major upgrade 이후에는 이 문서가 현재 Ethereum을 더 이상 온전히 반영하지 않는다고 명시한다.[^ref-eth-wp]

2026년 8월 4일 화요일 기준, 현대 Ethereum은 다음과 같이 설명하는 것이 가장 정확하다.

- shared world state를 가진 general-purpose blockchain
- EVM을 중심으로 한 execution environment
- transaction이 arbitrary computation을 유발할 수 있는 account-based system
- 작업증명 miner가 아니라 validator가 보안을 담당하는 proof-of-stake network[^ref-eth-doc-intro][^ref-eth-doc-pos]

따라서 올바른 연구 습관은 Ethereum의 vision을 역사적으로 읽고, 그것을 current protocol reality로 번역하는 것이다. 2014년 이후 아무것도 바뀌지 않은 것처럼 whitepaper를 그대로 인용해서는 안 된다.

---

## 4. Protocol Structure

### The Design Shift

Bitcoin은 money에서 출발해 ownership transfer를 위한 transaction-validation system을 구축한다.

Ethereum은 programmable state에서 출발해 shared computation을 위한 blockchain을 구축한다.

이것이 핵심적인 개념 전환이다.

### Functional Contrast

| System | Primary Native Question |
|---|---|
| Bitcoin | 어떤 coin을 누가 어떤 조건에서 spend할 수 있는가? |
| Ethereum | 현재 global state는 무엇이며, 어떤 computation이 그것을 갱신해야 하는가? |

### One Platform, Many Applications

Ethereum whitepaper는 custom asset, domain system, smart property model 같은 기능마다 separate blockchain을 만드는 대신, decentralized application을 위한 더 일반적인 protocol이 필요하다고 명시적으로 주장한다.[^ref-eth-wp]

---

## 5. Historical Context

### Before Ethereum

초기 blockchain 사고는 대체로 두 방향으로 나뉘었다.

- digital money와 asset transfer
- broader programmable logic를 위해 blockchain infrastructure를 활용하려는 시도

Ethereum whitepaper는 colored coin, smart property, Namecoin-style registry, 더 복잡한 application logic 같은 이전 아이디어를 명시적으로 언급하며, 이들이 universal application base로 쓰기에는 너무 제한적이거나 파편화되어 있다고 주장한다.[^ref-eth-wp]

### The Founding Claim

whitepaper의 핵심 주장은 built-in Turing-complete programming language를 가진 blockchain 하나가, use case마다 custom consensus infrastructure를 따로 만들지 않고도 많은 application을 수용할 수 있다는 것이다.[^ref-eth-wp]

### Why That Was Ambitious

이것은 "중개자 없이 value를 전송한다"보다 훨씬 더 넓은 야심이었다. 이는 다음을 전제했다.

- persistent shared state
- arbitrary execution
- application composability
- protocol-level programmability

이러한 야심은 새로운 capability를 만들었지만, 동시에 훨씬 더 큰 complexity와 attack surface도 만들었다.

---

## 6. Definitions

### Ethereum

현대 Ethereum documentation은 Ethereum을, blockchain 안에 computer가 내장된 시스템으로 설명하며, Ethereum Virtual Machine은 네트워크의 모든 참여자가 그 state에 합의하는 global virtual computer라고 설명한다.[^ref-eth-doc-intro]

### EVM

Ethereum Virtual Machine은 transaction-triggered computation을 적용하고, 합의된 네트워크 state를 갱신하는 shared execution environment다.[^ref-eth-doc-intro]

### Smart Contract

smart contract는 Ethereum state에 게시된 code이며, 이후 deterministic rule에 따라 transaction request에 의해 실행될 수 있다.[^ref-eth-doc-intro]

### World State

Ethereum은 UTXO set 중심으로 조직된 시스템이 아니다. transaction이 실행됨에 따라 내용이 바뀌는 account-based state model 중심의 시스템이다.

### Gas

gas는 computation과 resource use를 위한 Ethereum의 accounting mechanism이다. arbitrary computation은 metering이 없으면 unbounded execution과 resource exhaustion을 막을 수 없기 때문에 gas가 필요하다.

---

## 7. Technical Foundations

### Ethereum as a State Machine

Ethereum을 이해하는 가장 유용한 방식은 replicated state machine으로 보는 것이다.

transaction은 그 state를 전이시키라는 요청이다.

node는 요청된 computation을 실행하고, 그것이 valid하면 resulting state transition에 수렴한다.

### Accounts Instead of UTXOs

Ethereum documentation은 Ethereum을 discrete spendable output이 아니라 state, transaction, EVM execution 중심으로 설명한다.[^ref-eth-doc-intro]

즉 Ethereum 연구는 빠르게 state-centric해진다.

- account balance
- contract storage
- nonce
- code
- log
- receipt

### Arbitrary Computation

Ethereum whitepaper는 built-in programming language를 통해 arbitrary state transition function을 지원하는 플랫폼으로 자신을 규정했다.[^ref-eth-wp]

이것은 flexibility를 제공하지만, 동시에 다음도 도입한다.

- execution complexity
- contract bug
- fee-market dependence
- 더 넓은 implementation surface

---

## 8. Why Gas Exists

### The Resource Problem

사용자가 arbitrary computation을 요청할 수 있다면, protocol은 infinite loop와 무의미한 resource abuse를 막아야 한다.

### Economic Metering

현대 Ethereum documentation은 ETH가 computation을 위한 market를 제공하는 역할도 하며, 사용자는 네트워크에 computation을 요청하기 위해 ETH를 지불한다고 설명한다.[^ref-eth-doc-intro]

즉 Ethereum의 fee mechanism은 inclusion 대가만이 아니라 execution metering이기도 하다.

### Analytical Consequence

gas는 cosmetic한 요소가 아니다. metering 없는 programmability는 unsafe하기 때문에, gas는 Ethereum architecture의 중심이다.

---

## 9. Vision vs 2026 Reality

### The Whitepaper Is Foundational but Not Current

official Ethereum whitepaper page는 original whitepaper가 2014년에, Ethereum launch 이전에 작성되었고, major upgrade와 ecosystem growth 이후의 현재 Ethereum을 더 이상 반영하지 않는다고 명시한다.[^ref-eth-wp]

이 점이 중요하다. 연구자는 2026년 Ethereum을 whitepaper만으로 설명해서는 안 된다.

### Consensus Changed

현대 Ethereum documentation은 Ethereum이 proof-of-stake-based consensus mechanism을 사용하며, 2022년에 proof of stake로 전환했다고 설명한다.[^ref-eth-doc-intro][^ref-eth-doc-pos]

즉 Ethereum을 작업증명 network로 설명하는 것은 역사적으로는 중요하지만 현재로서는 outdated된 설명이다.

### Formal Specification Caveat

Yellow Paper repository는 Yellow Paper가 현재 outdated이며, 2023년 4월 Shanghai upgrade까지만 반영하고 Cancun 이후 변화는 반영하지 않는다고 밝힌다.[^ref-eth-yp-readme]

따라서 Ethereum의 유명한 formal specification조차도 2026년에는 신중하게 사용해야 한다.

---

## 10. Security Assumptions

### Shared-State Security Is Harder Than Asset Transfer Alone

Ethereum의 더 넓은 programmability는 잘못될 수 있는 범위를 크게 넓힌다.

security는 다음뿐 아니라:

- consensus integrity
- network participation
- client correctness

다음에도 의존한다.

- smart contract correctness
- gas economics
- client interoperability
- validator behavior

### Proof-of-Stake Security Model

현대 Ethereum의 proof of stake는 validator가 ETH를 stake하고, execution client와 consensus client, validator software를 실행하며, dishonest behavior에 대해 penalty 또는 slashing을 감수하는 구조다.[^ref-eth-doc-pos]

이것은 작업증명 mining security와 근본적으로 다르다.

### Research Caution

"Ethereum security"라는 말이 가리키는 대상은 여러 층위일 수 있다.

- protocol consensus security
- smart contract application security
- validator incentive
- client diversity
- governance response capacity

이 층위들은 서로 다르다.

---

## 11. Mathematical or Economic Model

### Execution-Cost Model

간단한 architectural model은 다음과 같다.

```text
Ethereum request
= state transition request
+ computation request
+ fee payment
```

모든 valid transaction이 execution semantic을 담을 수 있기 때문에, 이것은 순수 value-transfer model보다 더 표현력이 크다.

### Security Funding Intuition

현대 Ethereum documentation은 ETH가 validator 보상, slashable collateral, consensus process에서의 vote weight를 통해 네트워크 security를 돕는다고 설명한다.[^ref-eth-doc-intro]

### Resource Constraint

요청된 computation을 `C`, 실행을 위해 제시된 fee를 `F`라고 하면, Ethereum architecture는 computation이 bounded되고 priced되어야 함을 요구한다.

```text
unbounded C without pricing -> unsafe network
bounded C with metering -> viable shared execution
```

이것은 conceptual model이지 consensus formula가 아니다.

---

## 12. Protocol Implementation

### Modern Client Reality

현대 Ethereum node는 더 이상 하나의 monolithic client로 적절히 설명되지 않는다. proof of stake 체제에서 Ethereum 운영에는 execution client와 consensus client가 필요하고, validator는 validator software도 추가로 실행한다.[^ref-eth-doc-pos]

### Why This Matters

이 client split은 post-Merge architecture를 반영하며, current operational reality의 핵심이다.

### Formal Specification Limits

Yellow Paper는 여전히 역사적으로 중요하지만, repository 자체가 later upgrade에 비해 outdated되었다고 경고한다.[^ref-eth-yp-readme]

### Process Layer

EIPs repository는 Ethereum Improvement Proposal의 공식 publication medium이다. Bitcoin의 BIP repository와 마찬가지로, 이것은 process/specification surface이지 모든 proposal이 adopted되거나 activated되었다는 증거는 아니다.[^ref-eips-repo]

---

## 13. On-Chain Implications

### Richer Surface Area

Bitcoin-style transaction analysis와 비교하면, Ethereum on-chain analysis는 빠르게 다음으로 확장된다.

- account balance
- contract call
- storage update
- internal execution trace
- event와 log
- contract 위에 구축된 token standard

### More Expressiveness, More Ambiguity

Ethereum의 generality는 on-chain interpretation을 더 강력하게도 만들지만, 동시에 더 복잡하게도 만든다. 분석가는 종종 단순한 asset-transfer structure가 아니라 contract logic 자체를 이해해야 한다.

### Visibility Does Not Equal Simplicity

더 많은 on-chain data가 있다고 해서 inference가 쉬워지는 것은 아니다. 오히려 더 많은 해석 계층이 생긴다.

---

## 14. Institutional Thinking

institution은 Ethereum의 vision을 하나의 infrastructure thesis로 이해해야 한다.

- 하나의 shared programmable settlement layer
- 하나의 common execution environment
- blockspace와 execution을 두고 경쟁하는 많은 application

### Practical Implications

- Ethereum은 payment-centric mental model만으로 평가할 수 없다.
- smart contract risk는 first-order risk다.
- gas와 execution economics는 부차적 요소가 아니라 중심 요소다.
- Ethereum은 2014년 이후 materially changed했기 때문에, current protocol description에는 날짜 감각이 필요하다.
- serious research를 위해서는 client, validator, upgrade process에 대한 이해가 필요하다.

### Better Research Posture

올바른 institutional posture는 다음과 같다.

- whitepaper를 founding intent로 읽는다.
- current docs를 current operation의 기준으로 삼는다.
- EIP process를 change surface로 추적한다.
- application-layer risk와 protocol-layer risk를 분리한다.

---

## 15. Common Misinterpretations

### "Ethereum is just Bitcoin plus smart contracts"

틀렸다. shared world state, account model, arbitrary execution, gas accounting, proof-of-stake operation까지 포함하면 구조가 훨씬 더 다르다.

### "The whitepaper is the current specification"

틀렸다. Yellow Paper repository는 later upgrade에 비해 outdated되었다고 밝힌다.[^ref-eth-yp-readme]

### "More programmability automatically means better design"

틀렸다. 더 큰 표현력은 더 큰 complexity, risk, operational burden도 뜻한다.

---

## 16. Research Questions

1. Ethereum의 original vision 중 어떤 부분이 가장 성공적으로 실현되었는가?
2. Ethereum의 complexity 중 어떤 부분이 general-purpose programmability의 불가피한 결과인가?
3. 분석가는 Ethereum을 설명할 때 whitepaper, current docs, evolving EIP를 어떻게 균형 있게 사용할 것인가?
4. Ethereum의 어떤 risk는 protocol-level이고, 어떤 risk는 주로 application-level인가?
5. institution은 Bitcoin의 narrower design과 Ethereum의 broader design을, 한쪽 model을 다른 쪽에 억지로 덮어씌우지 않고 어떻게 비교해야 하는가?

---

## 17. Practical Exercises

### Exercise 1

Ethereum이 arbitrary computation을 원했다면 왜 gas가 필요했는지 자신의 말로 설명하라.

### Exercise 2

UTXO 기반 ledger와 shared world-state machine의 차이를 짧게 정리하라.

### Exercise 3

Ethereum whitepaper에서 2026년에는 반드시 historical qualification이 필요한 진술 세 가지를 나열하라.

### Exercise 4

왜 2026년의 Ethereum은 execution client, consensus client, validator를 사용해 설명해야 하는지 설명하라.

---

## 18. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Ethereum's original design intent | Directly specified | 2014 whitepaper |
| Modern Ethereum as a shared computer / EVM state system | Directly specified | Current official docs |
| Proof-of-stake architecture and validator requirements | Directly specified | Current official docs |
| Yellow Paper currency and freshness limitations | Directly specified | Yellow Paper repository README |
| Institutional framing and historical translation | Inference from sources | Derived from combining original and current sources |

---

## 19. Knowledge Graph

```text
Ethereum Vision
├─ Original Intent
│  ├─ generalized blockchain
│  ├─ smart contracts
│  ├─ decentralized applications
│  └─ common execution layer
├─ Core Concepts
│  ├─ EVM
│  ├─ world state
│  ├─ accounts
│  └─ gas
├─ Historical Shift
│  ├─ 2014 whitepaper
│  ├─ launch-era Ethereum
│  └─ proof-of-stake Ethereum
├─ Modern Reality
│  ├─ execution clients
│  ├─ consensus clients
│  ├─ validators
│  └─ evolving EIPs
└─ Research Discipline
   ├─ vision vs implementation
   ├─ protocol vs application risk
   ├─ current vs historical sources
   └─ institutional interpretation
```

---

## 20. 참고문헌

### Primary Sources

[^ref-eth-wp]: Vitalik Buterin, "Ethereum Whitepaper," official ethereum.org whitepaper page, including the note that the 2014 whitepaper no longer reflects what Ethereum is today, https://ethereum.org/whitepaper/, accessed 2026-08-04.

[^ref-eth-doc-intro]: ethereum.org, "Technical intro to Ethereum," official documentation describing Ethereum as a blockchain with an embedded computer and EVM-based shared state, page last updated April 22, 2026, https://ethereum.org/developers/docs/intro-to-ethereum/, accessed 2026-08-04.

[^ref-eth-doc-pos]: ethereum.org, "Proof-of-stake (PoS)," official documentation describing Ethereum's proof-of-stake consensus, validator requirements, slots, epochs, and finality, page published 2026, https://ethereum.org/developers/docs/consensus-mechanisms/pos/, accessed 2026-08-04.

[^ref-eth-yp-readme]: Ethereum Yellow Paper repository README, including the statement that the Yellow Paper is currently outdated and reflects Ethereum only up to the Shanghai upgrade, https://github.com/ethereum/yellowpaper, accessed 2026-08-04.

[^ref-eips-repo]: Ethereum Improvement Proposals repository, formal proposal publication repository for EIPs, https://github.com/ethereum/EIPs, accessed 2026-08-04.

### Supporting Interpretation Notes

- Where this document contrasts Bitcoin and Ethereum at a high level, those contrasts are analytical summaries of the cited primary sources rather than literal one-line protocol definitions.

---

## 21. 교차 참조

### Previous

- BLOCKCHAIN-FOUNDATION-008 — Trade-offs of Blockchain
- BITCOIN-033 — Bitcoin Core

### Next

- ETHEREUM-FOUNDATION-002 — Account Model

### Related

- BITCOIN-014 — UTXO Model
- BITCOIN-034 — Institutional Perspective on Bitcoin

---

## Review Status

### Technical Review

Passed.

- 문서는 Ethereum의 founding ambition과 current architecture를 분리한다.
- world-state computation과 UTXO transfer logic를 구분한다.
- current Ethereum consensus를 proof of stake로 정확히 설명한다.
- Yellow Paper를 2026년에도 fully current한 문서처럼 사용하지 않는다.

### Evidence Review

Passed.

- original-vision claim은 official whitepaper page를 인용한다.
- current architecture claim은 current official documentation을 인용한다.
- Yellow Paper freshness limitation은 repository README를 인용한다.
- institutional interpretation은 inference로 라벨링했다.

### Editorial Review

Passed.

- 구조는 프로젝트 deep-dive format을 따른다.
- metadata는 완전하다.
- terminology는 EVM, world state, gas, validator, proof of stake로 일관된다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 2014 whitepaper를 current specification 전체로 제시하지 않는다.
- 2026년의 Ethereum을 작업증명 network로 설명하지 않는다.
- smart-contract risk를 consensus risk로 축소하지 않는다.
- 더 많은 programmability가 더 적은 trade-off를 뜻한다고 가정하지 않는다.
- proposal repository를 automatic adoption으로 혼동하지 않는다.

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
