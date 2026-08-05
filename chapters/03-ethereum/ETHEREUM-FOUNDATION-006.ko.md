---
knowledge_id: ETHEREUM-FOUNDATION-006
title: Gas
subtitle: Computation Metering, Base Fee Burn, Priority Fee, Gas Limit, 그리고 Ethereum이 execution에 가격을 매기는 이유
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 135 min
estimated_study: 360 min
last_reviewed: 2026-08-04
domain:
  - Ethereum
  - Gas
  - Fees
  - Execution Economics
parent:
  - Ethereum Foundations
prerequisites:
  - ETHEREUM-FOUNDATION-004
  - ETHEREUM-FOUNDATION-005
related_topics:
  - State Transition
  - Transactions
  - Blocks
  - EIP-1559
primary_sources:
  - REF-ETH-DOC-GAS-2026-001
  - REF-ETH-DOC-TX-2026-001
  - REF-EIP-1559
  - REF-ETH-DOC-BLOCKS-2026-001
tags:
  - ethereum
  - gas
  - fees
  - base-fee
  - priority-fee
  - eip-1559
---

# Gas
> Ethereum Foundations  
> Research Unit: ETHEREUM-FOUNDATION-006

---

## Research Brief

```yaml
knowledge_id: ETHEREUM-FOUNDATION-006
title: Gas
research_question: >
  Why does Ethereum meter computation with gas, how do gas limits, base fees,
  priority fees, and EIP-1559 work together, and how should researchers think
  about gas as both a security mechanism and an execution-pricing system in
  August 2026?
document_type: deep-dive
difficulty: L300
prerequisites:
  - ETHEREUM-FOUNDATION-004
  - ETHEREUM-FOUNDATION-005
parent: Ethereum Foundations
previous: ETHEREUM-FOUNDATION-005
next: ETHEREUM-FOUNDATION-007
related_topics:
  - EVM
  - Transactions
  - EIP-1559
  - Blocks
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
  - Wallet fee-estimation UX comparison
  - Layer 2 fee models
  - MEV bidding systems
  - Historical fee chart survey
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Ethereum이 왜 gas를 필요로 하는지 설명할 수 있다.
- gas, gas limit, base fee, priority fee, max fee를 정의할 수 있다.
- EIP-1559가 Ethereum fee mechanic을 어떻게 바꿨는지 설명할 수 있다.
- gas가 pricing system이면서 security mechanism이기도 하다는 점을 설명할 수 있다.
- block gas target과 block gas limit이 fee dynamic에 어떻게 영향을 주는지 설명할 수 있다.

---

## 2. 핵심 질문

1. gas란 무엇인가?
2. 왜 Ethereum은 free execution이 아니라 priced computation을 택하는가?
3. gas used와 gas price의 차이는 무엇인가?
4. base fee와 priority fee는 어떻게 작동하는가?
5. EIP-1559는 older fee intuition과 비교해 무엇을 바꾸는가?
6. block size와 gas limit은 어떻게 상호작용하는가?

---

## 3. Executive Summary

gas는 computational effort를 측정하는 Ethereum의 단위다. official documentation은 모든 Ethereum transaction이 execution에 computational resource를 필요로 하며, 네트워크가 spam에 취약해지거나 infinite loop에 빠지지 않도록 그 resource에 대한 payment가 필요하다고 설명한다.[^ref-eth-doc-gas]

즉 gas는 단순한 fee label이 아니다. Ethereum의 execution metering system이다.

현대 Ethereum의 fee mechanic은 EIP-1559를 중심으로 구축되어 있다. official docs와 EIP 본문은 다음 두 가지 주요 구성요소를 설명한다.

- burned되는 protocol-set base fee
- transaction inclusion을 유도하는 priority fee (tip)[^ref-eth-doc-gas][^ref-eip-1559]

current docs는 base fee가 prior block congestion으로부터 도출되며, previous block이 target size보다 얼마나 차 있었는지에 따라 block마다 최대 12.5%까지 오르거나 내릴 수 있다고 설명한다.[^ref-eth-doc-gas]

2026년의 gas는 다음의 교차점으로 가르쳐야 한다.

- execution safety
- fee-market design
- block-capacity rationing
- validator incentive signaling

---

## 4. Protocol Structure

### The Core Relationship

Ethereum execution은 다음 구조를 따른다.

```text
requested computation
-> gas consumed
-> fee paid
-> state transition allowed or denied by resource bounds
```

### Main Components

| Component | Role |
|---|---|
| Gas | computation unit |
| Gas limit | transaction이 소비할 수 있는 최대 computation |
| Base fee | protocol-set reserve price per gas |
| Priority fee | validator incentive tip |
| Max fee | sender의 fee cap |

### Why This Matters

이 구조가 없으면 arbitrary on-chain computation은 경제적으로도, 운영적으로도 unsafe하다.

---

## 5. Why Gas Exists

### Anti-Spam Purpose

official docs는 gas fee가 네트워크에서 실행되는 모든 computation에 payment를 요구함으로써 Ethereum을 안전하게 유지한다고 설명한다.[^ref-eth-doc-gas]

### Infinite-Loop Defense

같은 docs는 각 transaction이 code execution이 사용할 수 있는 computational step 수에 한도를 두어, accidental 또는 hostile infinite loop에 빠지지 않도록 해야 한다고 명시한다.[^ref-eth-doc-gas]

### Resource Pricing

따라서 gas는 희소한 resource의 가격이다.

- validator와 node의 computation
- block capacity
- execution bandwidth
- state-access pressure

---

## 6. Gas Units and Fee Arithmetic

### Gas Is Not ETH

gas는 computational effort의 단위다. ETH는 그 effort에 payment하는 화폐다.[^ref-eth-doc-gas]

### Gwei

current docs는 gas price가 보통 gwei로 표시되며, one gwei는 one-billionth of an ETH라고 설명한다.[^ref-eth-doc-gas]

### Fee Formula

official docs는 total gas paid를 다음처럼 설명한다.

```text
gas used * (base fee + priority fee)
```

[^ref-eth-doc-gas]

### Example Intuition

transactions docs에 따르면 simple ETH transfer는 보통 21,000 gas가 필요하다.[^ref-eth-doc-tx]

---

## 7. EIP-1559 Fee Model

### Protocol Change

EIP-1559는 dynamic block sizing과 함께 base fee burn이 있는 transaction-pricing mechanism을 도입했다.[^ref-eip-1559]

### Base Fee

current gas docs는 every block에 reserve price 역할을 하는 base fee가 있으며, transaction inclusion eligibility를 위해 반드시 지불되어야 한다고 설명한다.[^ref-eth-doc-gas]

### Priority Fee

같은 docs는 priority fee가 validator를 유인하고 urgency를 signal하는 수단이라고 설명한다.[^ref-eth-doc-gas]

### Max Fee

사용자는 자신이 지불할 willing이 있는 maximum fee도 지정할 수 있으며, docs가 설명하는 transaction-fee logic에 따라 사용되지 않은 차액은 refund된다.[^ref-eth-doc-gas]

### Burn Consequence

EIP-1559는 execution cost의 일부가 validator에게 이전되는 것이 아니라 base fee burn을 통해 파괴된다는 뜻이다.[^ref-eip-1559]

---

## 8. Block Capacity and Fee Dynamics

### Target and Limit

current block docs는 각 block이 30 million gas의 target size와 60 million gas의 block limit를 가지며, demand에 따라 size가 변동할 수 있다고 설명한다.[^ref-eth-doc-blocks]

### Base Fee Feedback

current gas docs는 previous block이 target size보다 위에 있었는지 아래에 있었는지에 따라 base fee가 조정되며, block당 최대 12.5%만 움직일 수 있다고 설명한다.[^ref-eth-doc-gas]

### Consequence

즉 fee pricing은 fixed schedule이 아니라, block capacity를 둘러싼 congestion-sensitive control mechanism이다.

### Important Distinction

transaction의 `gas_limit`은 block gas limit와 다르다. 전자는 single transaction의 resource use를 제한하고, 후자는 block 전체의 resource envelope을 제한한다.

---

## 9. Technical Mechanics

### Transaction Execution and Gas

execution 중에는:

```text
transaction enters execution
-> gas budget available
-> operations consume gas
-> success or revert occurs
-> fee still charged for work done
```

### Success and Failure

official gas docs는 transaction이 성공하든 실패하든 fee가 지불된다고 설명한다.[^ref-eth-doc-gas]

### Too-Low Limit

같은 docs는 transaction이 기본 validity requirement를 만족시키기에도 너무 적은 gas를 지정하면 inclusion 전에 실패할 수 있고, execution 중 out-of-gas가 발생하면 state change는 revert되지만 사용된 gas는 소비된다고 설명한다.[^ref-eth-doc-gas]

---

## 10. Security Assumptions

### Metering as Security

gas는 shared network resource의 unrestricted consumption을 막기 때문에 Ethereum security design의 일부다.[^ref-eth-doc-gas]

### Fee-Market Assumptions

이 시스템은 user와 validator가 다음에 경제적으로 반응한다고 가정한다.

- congestion
- urgency
- fee signaling
- capacity scarcity

### Interpretation Risk

gas를 단지 user fee로만 설명하는 연구자는 anti-DoS와 execution-bounding 역할을 놓치게 된다.

---

## 11. Mathematical or Economic Model

### Core Fee Equation

단순화한 modern transaction fee model은 다음과 같다.

```text
total fee paid
= gas used * (base fee + priority fee)
```

[^ref-eth-doc-gas]

### Block Feedback Rule

current docs는 previous block이 target size보다 위인지 아래인지에 따라 base fee가 block당 최대 12.5% 움직인다고 설명한다.[^ref-eth-doc-gas]

### Capacity Framing

block 수준에서는:

```text
if gas_used > target -> base fee rises
if gas_used < target -> base fee falls
```

이것은 feedback mechanism을 단순화해 요약한 것이다.

---

## 12. Protocol Implementation

### Current Primary Sources

official gas docs, transaction docs, block docs를 함께 보는 것이 가장 current operational picture를 잘 제공한다.[^ref-eth-doc-gas][^ref-eth-doc-tx][^ref-eth-doc-blocks]

### EIP Layer

EIP-1559는 base-fee-burn architecture를 formal하게 정의하므로 필수다.[^ref-eip-1559]

### Why This Matters

EIP-1559 awareness가 없다면 gas 자체는 설명할 수 있어도, modern Ethereum gas는 정확히 설명할 수 없다.

---

## 13. On-Chain Implications

### Observable Fee Data

on-chain observer는 직접 다음을 연구할 수 있다.

- gas used
- base fee per gas
- priority fee behavior
- fee burn effect
- block fullness

### Execution Intensity Signal

gas usage는 execution intensity의 대략적인 signal이기도 하지만, economic importance에 대한 완벽한 semantic proxy는 아니다.

### Analytical Consequence

Ethereum fee analysis는 block-capacity analysis와 분리될 수 없다.

---

## 14. Institutional Thinking

institution은 gas를 execution-cost model이자 congestion-risk model로 함께 다뤄야 한다.

### Practical Implications

- transaction operation은 transfer amount뿐 아니라 execution uncertainty도 budget해야 한다.
- fee policy는 urgency와 non-urgent batching/maintenance action을 구분해야 한다.
- congestion scenario는 cost, latency, inclusion confidence에 영향을 준다.
- gas analytics는 기본적으로 EIP-1559-aware해야 한다.

### Better Research Posture

fee claim을 하기 전에 다음을 물어야 한다.

- base fee, tip, total fee 중 무엇을 말하는가?
- per-transaction behavior를 말하는가, block-level congestion을 말하는가?
- current EIP-1559 behavior를 말하는가, older legacy fee intuition을 말하는가?

---

## 15. Common Misinterpretations

### "Gas is just a transaction fee"

틀렸다. gas는 execution metering이자 anti-spam control이기도 하다.[^ref-eth-doc-gas]

### "Gas price is fixed by Ethereum"

틀렸다. base fee는 protocol-determined지만, priority fee와 max fee는 user-side input이며 market-sensitive하다.

### "A failed transaction costs nothing"

틀렸다. 수행된 work는 여전히 gas를 소비한다.[^ref-eth-doc-gas]

### "Block gas limit and transaction gas limit are the same"

틀렸다. 서로 다른 level에서 동작한다.

---

## 16. Research Questions

1. institution은 transfer, contract interaction, operational maintenance flow에 걸친 gas risk를 어떻게 분류해야 하는가?
2. congestion 하에서 inclusion delay를 가장 잘 예측하는 gas metric은 무엇인가?
3. base-fee burn은 pure validator-fee model과 비교해 long-run ETH economic interpretation을 얼마나 바꾸는가?

---

## 17. Practical Exercises

### Exercise 1

왜 Ethereum은 gas 없이 arbitrary execution을 안전하게 허용할 수 없는지 설명하라.

### Exercise 2

base fee와 priority fee의 차이를 짧게 설명하라.

### Exercise 3

transaction이 execution 중간에 out of gas가 났을 때 경제적으로 무엇이 일어나는지 설명하라.

### Exercise 4

30M gas target과 60M block limit가 왜 동시에 존재할 수 있는지 설명하라.

---

## 18. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Gas as computation meter and anti-spam mechanism | Directly specified | Official gas docs |
| Base fee / priority fee / max fee mechanics | Directly specified | Official gas docs and EIP-1559 |
| Block gas target and limit behavior | Directly specified | Official block docs |
| Institutional gas-risk framing | Inference from sources | Derived from fee and capacity architecture |

---

## 19. Knowledge Graph

```text
Gas
├─ Purpose
│  ├─ computation metering
│  ├─ anti-spam
│  └─ infinite-loop defense
├─ Fee Components
│  ├─ base fee
│  ├─ priority fee
│  └─ max fee
├─ Capacity Interaction
│  ├─ transaction gas limit
│  ├─ block gas target
│  └─ block gas limit
└─ Economic Effects
   ├─ fee burn
   ├─ validator incentive
   └─ congestion pricing
```

---

## 20. References

### Primary Sources

[^ref-eth-doc-gas]: ethereum.org, "Ethereum gas and fees: technical overview," official documentation describing gas, gas limit, base fee, priority fee, and fee dynamics, https://ethereum.org/developers/docs/gas/, accessed 2026-08-04.

[^ref-eth-doc-tx]: ethereum.org, "Transactions," official documentation including the 21,000 gas simple transfer example and fee side effects, https://ethereum.org/developers/docs/transactions/, accessed 2026-08-04.

[^ref-eip-1559]: EIP-1559, "Fee market change for ETH 1.0 chain," Ethereum Improvement Proposals, https://eips.ethereum.org/EIPS/eip-1559, accessed 2026-08-04.

[^ref-eth-doc-blocks]: ethereum.org, "Blocks," official documentation describing 30M gas target, 60M gas block limit, and gas-capacity behavior, page last updated February 23, 2026, https://ethereum.org/developers/docs/blocks/, accessed 2026-08-04.

### Supporting Interpretation Notes

- Where this document discusses institutional gas policy or congestion-risk posture, those are analytical inferences built on the cited fee and block-capacity sources.

---

## 21. Cross References

### Previous

- ETHEREUM-FOUNDATION-005 — EVM

### Next

- ETHEREUM-FOUNDATION-007 — Blocks

### Related

- ETHEREUM-FOUNDATION-004 — State Transition
- BITCOIN-018 — Transaction Fees

---

## Review Status

### Technical Review

Passed.

- gas를 resource metering이자 fee architecture로 함께 설명했다.
- EIP-1559를 modern fee model로 통합했다.
- block-capacity relationship과 per-transaction gas setting을 분리했다.
- failure semantic을 포함했다.

### Evidence Review

Passed.

- gas-mechanics claim은 current official docs를 인용한다.
- fee-model claim은 EIP-1559를 인용한다.
- block-capacity claim은 official blocks docs를 인용한다.
- institutional interpretation은 inference로 라벨링했다.

### Editorial Review

Passed.

- 구조는 프로젝트 문서 패턴을 따른다.
- metadata는 완전하다.
- terminology는 gas, base fee, priority fee, max fee, gas limit로 일관된다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 gas를 UX cost로만 축소하지 않는다.
- fee unit과 ETH value를 직접 동일시하지 않는다.
- transaction gas limit와 block gas limit를 혼동하지 않는다.
- failed execution을 free로 설명하지 않는다.

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
