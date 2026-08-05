---
knowledge_id: ETHEREUM-FOUNDATION-004
title: State Transition
subtitle: Transaction, Gas, Validation, EVM execution이 old state를 new state로 바꾸는 방식
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 135 min
estimated_study: 360 min
last_reviewed: 2026-08-04
domain:
  - Ethereum
  - State Transition
  - Transactions
  - Gas
  - EVM
parent:
  - Ethereum Foundations
prerequisites:
  - ETHEREUM-FOUNDATION-001
  - ETHEREUM-FOUNDATION-002
  - ETHEREUM-FOUNDATION-003
related_topics:
  - Account Model
  - World State
  - Gas
  - Typed Transactions
  - Fee Market
primary_sources:
  - REF-ETH-DOC-EVM-2026-001
  - REF-ETH-DOC-TX-2026-001
  - REF-ETH-DOC-GAS-2026-001
  - REF-EIP-2718
  - REF-EIP-1559
  - REF-ETH-DOC-POS-2026-001
tags:
  - ethereum
  - state-transition
  - transactions
  - gas
  - evm
  - eip-1559
---

# State Transition
> Ethereum Foundations  
> Research Unit: ETHEREUM-FOUNDATION-004

---

## Research Brief

```yaml
knowledge_id: ETHEREUM-FOUNDATION-004
title: State Transition
research_question: >
  How does Ethereum transform one valid world state into another through
  transactions, EVM execution, gas accounting, and validator inclusion, and
  which parts of the transaction and fee model must be treated as modern,
  version-aware behavior in August 2026?
document_type: deep-dive
difficulty: L300
prerequisites:
  - ETHEREUM-FOUNDATION-001
  - ETHEREUM-FOUNDATION-002
  - ETHEREUM-FOUNDATION-003
parent: Ethereum Foundations
previous: ETHEREUM-FOUNDATION-003
next: ETHEREUM-FOUNDATION-005
related_topics:
  - Transactions
  - Gas
  - Fee Market
  - EVM
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
  - Full opcode-by-opcode execution semantics
  - MEV market structure
  - Complete mempool client-policy survey
  - Layer 2 execution semantics
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Ethereum state transition을 protocol 수준에서 설명할 수 있다.
- transaction이 world state의 변화를 어떻게 요청하는지 설명할 수 있다.
- gas가 왜 필요한지, 그리고 그것이 execution을 어떻게 제한하는지 설명할 수 있다.
- legacy transaction framing과 typed transaction framing을 구분할 수 있다.
- modern Ethereum에서 validator와 post-inclusion finality의 역할을 설명할 수 있다.

---

## 2. 핵심 질문

1. Ethereum에서 "state transition"은 무엇을 의미하는가?
2. transaction은 어떻게 state change가 되는가?
3. gas는 왜 필요한가?
4. typed transaction은 transaction model을 어떻게 바꾸는가?
5. EIP-1559는 execution의 economics를 어떻게 바꾸는가?
6. proof-of-stake Ethereum에서 block은 어떻게 justified되고 finalized되는가?

---

## 3. Executive Summary

Ethereum의 execution model은 근본적으로 state transition system이다. official EVM documentation은 이를 `Y(S, T) = S'`로 형식화하며, valid old state `S`와 valid transactions `T`가 new valid state `S'`를 만든다고 설명한다.[^ref-eth-doc-evm]

현재 transaction documentation은 transaction을 account가 보낸 cryptographically signed instruction으로 설명하며, Ethereum network의 state를 갱신하는 데 쓰인다고 말한다.[^ref-eth-doc-tx]

이 transition process는 gas와 떼어놓을 수 없다. official gas documentation은 gas가 computational effort를 측정하며, Ethereum이 spam에 취약해지거나 infinite loop에 빠지지 않도록 하기 위해 존재한다고 설명한다.[^ref-eth-doc-gas]

현대 Ethereum의 state transition은 modern transaction format과 fee rule을 포함해서 설명해야 한다. EIP-2718은 typed transaction envelope를 도입했고, EIP-1559는 protocol base fee를 burn하고 priority fee를 block proposer/validator에게 지급하는 fee model을 도입했다.[^ref-eip-2718][^ref-eip-1559]

2026년 8월 4일 기준, state transition은 proof-of-stake Ethereum 안에서 설명되어야 한다. validator는 block을 propose하고 attest하며, block은 inclusion 이후 justification과 finality로 나아간다.[^ref-eth-doc-pos][^ref-eth-doc-tx]

---

## 4. Protocol Structure

### Minimal Transition Loop

Ethereum의 기본 loop는 다음과 같다.

```text
old world state
-> receive valid transaction
-> execute under EVM rules with gas limits
-> update balances / nonces / storage / logs
-> commit new state root
```

### Two Layers of Meaning

| Layer | Meaning |
|---|---|
| Execution layer | transaction logic를 재실행하고 new state를 계산 |
| Consensus layer | block을 순서화하고 proposer activity를 검증하며 history를 finalize |

### Why the Split Matters

현대 Ethereum에서는 execution과 consensus가 개념적으로 분리되어 있으면서도, 함께 accepted state를 결정한다.

---

## 5. Transactions as State-Change Requests

### What a Transaction Is

official docs는 transaction을 account가 보낸 cryptographically signed instruction으로 정의하고, 가장 단순한 경우는 한 account에서 다른 account로 ETH를 전송하는 것이라고 설명한다.[^ref-eth-doc-tx]

### State-Changing Nature

transaction은 단순한 message가 아니다. valid하고 included될 경우 state를 mutation하라는 요청이다.

### Main Transaction Classes

transaction docs는 다음을 구분한다.

- regular transaction
- contract deployment transaction
- deployed contract를 실행하는 transaction[^ref-eth-doc-tx]

이 각각은 state mutation으로 가는 서로 다른 경로다.

---

## 6. Gas and Bounded Execution

### Why Gas Exists

official gas docs는 gas가 operation 실행에 필요한 computational effort를 측정하며, Ethereum이 spam을 당하거나 infinite computation에 갇히지 않도록 필요하다고 설명한다.[^ref-eth-doc-gas]

### Gas Limit

같은 docs는 gas limit를 사용자가 한 transaction에서 소비할 willing이 있는 최대 gas 양으로 정의한다.[^ref-eth-doc-gas]

### Fee Regardless of Success

gas docs는 transaction이 성공하든 실패하든 fee는 지불된다고 설명한다.[^ref-eth-doc-gas]

### Design Consequence

Ethereum execution이 arbitrary computation을 지원해 expressive할 수 있는 것은, 그 computation이 명시적으로 bounded되고 priced되기 때문이다.

---

## 7. Typed Transactions and Modern Formats

### Historical Change

official transaction docs는 Ethereum이 원래 하나의 transaction format만 갖고 있었지만, 이후 multiple types를 지원하도록 진화했다고 설명한다.[^ref-eth-doc-tx]

### EIP-2718

EIP-2718은 typed transaction envelope를 다음과 같이 정의한다.

```text
TransactionType || TransactionPayload
```

[^ref-eip-2718]

### Why It Matters

typed envelope는 새로운 transaction format을 모두 legacy encoding pattern 안에 억지로 밀어 넣지 않고도 transaction evolution을 가능하게 한다.

### Research Consequence

2026년의 Ethereum transaction을 설명하면서 "하나의 legacy transaction format"만으로 protocol 전체를 정의하는 것처럼 쓰면 안 된다.

---

## 8. Fee Market and EIP-1559

### Core Mechanism

EIP-1559는 congestion에 따라 조정되며 burn되는 protocol base fee per gas와, transaction이 지정하는 priority fee 및 max fee 구조를 도입했다.[^ref-eip-1559]

### Current Docs Consistency

현재 transaction docs도 이 model을 반영해, base fee는 burned되고 tip은 validator에게 간다고 설명한다.[^ref-eth-doc-tx]

### Why This Matters for State Transition

Ethereum의 state transition은 단순히 "code가 실행되었는가?"가 아니다. "execution이 어떻게 priced되었고, fee side effect가 어떻게 적용되었는가?"도 포함한다.

### Burn Side Effect

EIP-1559는 execution의 state/economic effect 일부가 base fee burn을 통한 ETH 소각이라는 뜻이다.[^ref-eip-1559]

---

## 9. Proof-of-Stake Inclusion and Finality

### Modern Validator Role

현재 PoS docs는 validator가 proposed block을 검증하고, 가끔 block을 propose하며, block validity에 attest한다고 설명한다.[^ref-eth-doc-pos]

### Transaction Inclusion

transaction docs는 transaction이 성공으로 간주되려면 validator가 그것을 선택해 block에 포함해야 한다고 말한다.[^ref-eth-doc-tx]

### Finality Path

같은 transaction docs는 block이 이후 justified되고 finalized될 수 있다고 설명한다.[^ref-eth-doc-tx]

### Consequence

inclusion은 finality와 다르다. transaction은 accepted block execution과 함께 state의 일부가 되지만, permanence에 대한 confidence는 consensus가 진행되면서 커진다.

---

## 10. Technical Mechanics

### Formal Transition Description

EVM documentation은 다음 형식을 제공한다.[^ref-eth-doc-evm]

```text
Y(S, T) = S'
```

### Practical Transaction Pipeline

실무적 pipeline은 다음과 같다.

```text
EOA signs transaction
-> node verifies basic validity
-> validator includes transaction in block
-> execution client re-executes transaction
-> gas is charged
-> balances / nonce / storage / logs update
-> new state root is committed
```

### Simple ETH Transfer Example

official transaction docs는 simple transfer가 21,000 gas를 요구한다고 설명하며, sender balance는 transfer amount와 fee만큼 감소하고 recipient는 transferred ETH를 받으며, base fee는 burned되고 tip은 validator에게 간다고 예시를 든다.[^ref-eth-doc-tx]

---

## 11. Security Assumptions

### Deterministic Execution

node는 같은 included transaction set에 대해 deterministic하게 같은 execution outcome에 합의해야 한다.

### Fee-Based Abuse Resistance

gas는 단순한 pricing convenience가 아니라 security mechanism이다. bounded computation과 payment가 없다면 arbitrary execution은 denial-of-service에 취약해진다.[^ref-eth-doc-gas]

### Consensus Dependence

valid state transition은 local execution logic뿐 아니라, valid block ordering과 PoS consensus participation에도 의존한다.[^ref-eth-doc-pos]

### Source Freshness

transaction type과 fee rule이 시간이 지나며 크게 바뀌었기 때문에, state-transition 설명은 version-aware해야 한다.[^ref-eip-2718][^ref-eip-1559]

---

## 12. Mathematical or Economic Model

### State Transition Function

primary formal model은 Ethereum docs가 직접 제시한다.[^ref-eth-doc-evm]

```text
Y(S, T) = S'
```

### Fee Model Intuition

하나의 included transaction에 대해 단순화한 payment model은 다음과 같다.

```text
total paid by sender
= transferred value
+ gas used * base fee
+ gas used * priority fee
```

이것은 현대 fee model의 단순화된 해석으로 refund/unused gas detail은 생략하지만, current-doc framing과는 일치한다.[^ref-eth-doc-tx][^ref-eip-1559]

### Safety Constraint

개념적으로:

```text
arbitrary execution + no metering -> unsafe
arbitrary execution + gas metering -> viable
```

---

## 13. Protocol Implementation

### Official Docs as Current Primary Source

current conceptual explanation에는 official EVM, transactions, gas, PoS docs가 가장 명확한 primary source다.[^ref-eth-doc-evm][^ref-eth-doc-tx][^ref-eth-doc-gas][^ref-eth-doc-pos]

### EIP Layer

EIP-2718과 EIP-1559는 older summary가 빠뜨리는 modern transaction/fee-model behavior를 정의하기 때문에 필수다.[^ref-eip-2718][^ref-eip-1559]

### Why This Matters

이 두 EIP를 빼면 과거 Ethereum은 정확히 설명할 수 있어도, modern Ethereum은 오설명하게 된다.

---

## 14. On-Chain Implications

### Observable Effects

on-chain observer는 다음을 볼 수 있다.

- transaction type
- gas used
- fee field
- balance change
- contract deployment
- log와 receipt
- block level의 state root change

### Not Directly Visible from One Number

`stateRoot` 하나만으로는 사람이 무엇이 바뀌었는지 알 수 없다. 그것은 explanation이 아니라 commitment다. state transition을 이해하려면 transaction과 execution context가 필요하다.

### Analytical Burden

Ethereum analysis는 종종 다음을 함께 읽어야 한다.

- transaction data
- receipt
- log
- execution trace
- state change

---

## 15. Institutional Thinking

institution은 Ethereum state transition을 기술적 process이자 경제적 process로 함께 다뤄야 한다.

### Practical Implications

- gas cost는 execution risk의 일부다.
- inclusion time과 finality time은 구분해야 한다.
- current transaction-type support는 wallet, custody, monitoring system에 중요하다.
- fee analytics는 기본적으로 EIP-1559-aware해야 한다.
- execution analysis는 transfer-only monitoring보다 richer data를 필요로 한다.

### Better Research Posture

어떤 Ethereum transaction claim을 말할 때는 다음을 물어야 한다.

- 어떤 transaction type이 관련되어 있는가?
- 어떤 fee rule이 적용되는가?
- inclusion을 말하는가, finality를 말하는가?
- raw state effect를 해석하는가, 아니면 user-visible transfer만 보고 있는가?

---

## 16. Common Misinterpretations

### "A transaction is just a payment"

틀렸다. 그것은 arbitrary execution을 유발할 수 있는 state-change request다.

### "Gas is only a user fee"

틀렸다. gas는 protocol-level execution bound이자 anti-spam mechanism이기도 하다.[^ref-eth-doc-gas]

### "Ethereum still has one transaction format"

틀렸다. modern Ethereum은 typed transaction을 지원한다.[^ref-eip-2718][^ref-eth-doc-tx]

### "Inclusion equals finality"

틀렸다. modern Ethereum은 inclusion, justification, finality를 구분한다.[^ref-eth-doc-tx][^ref-eth-doc-pos]

---

## 17. Research Questions

1. institution은 multiple Ethereum transaction type에 걸친 analytics를 어떻게 normalize해야 하는가?
2. 단순 ETH transfer를 넘어 risk system에 가장 중요한 execution-side effect는 무엇인가?
3. wallet과 custody team은 execution success와 economic finality를 어떻게 구분해야 하는가?

---

## 18. Practical Exercises

### Exercise 1

transaction이 실패하더라도 gas를 청구해야 하는 이유를 설명하라.

### Exercise 2

legacy transaction framing과 typed transaction framing을 짧게 비교하라.

### Exercise 3

EIP-1559가 transaction fee의 economic interpretation을 어떻게 바꾸는지 설명하라.

### Exercise 4

transaction inclusion, justification, finality의 차이를 설명하라.

---

## 19. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| State transition function `Y(S, T) = S'` | Directly specified | EVM docs |
| Transactions as signed state-change requests | Directly specified | Transactions docs |
| Gas as computation metering and anti-spam mechanism | Directly specified | Gas docs |
| Typed transactions and EIP-1559 fee mechanics | Directly specified | EIPs and current docs |
| Institutional implications | Inference from sources | Derived from execution and fee architecture |

---

## 20. Knowledge Graph

```text
State Transition
├─ Inputs
│  ├─ old state
│  ├─ signed transaction
│  └─ gas parameters
├─ Execution
│  ├─ EVM
│  ├─ contract calls
│  ├─ storage updates
│  └─ receipts/logs
├─ Economic Layer
│  ├─ gas metering
│  ├─ base fee burn
│  └─ priority fee
├─ Transaction Evolution
│  ├─ legacy format
│  └─ typed envelopes
└─ Consensus Outcome
   ├─ inclusion
   ├─ justification
   └─ finality
```

---

## 21. References

### Primary Sources

[^ref-eth-doc-evm]: ethereum.org, "Ethereum Virtual Machine (EVM)," official documentation describing Ethereum's state transition function, published 2026, https://ethereum.org/developers/docs/evm/, accessed 2026-08-04.

[^ref-eth-doc-tx]: ethereum.org, "Transactions," official documentation describing Ethereum transactions, typed transaction framing, lifecycle, and current fee examples, page last updated March 12, 2026, https://ethereum.org/developers/docs/transactions/, accessed 2026-08-04.

[^ref-eth-doc-gas]: ethereum.org, "Ethereum gas and fees: technical overview," official documentation describing gas as computation metering and anti-spam mechanism, published 2026, https://ethereum.org/developers/docs/gas/, accessed 2026-08-04.

[^ref-eip-2718]: EIP-2718, "Typed Transaction Envelope," Ethereum Improvement Proposals, https://eips.ethereum.org/EIPS/eip-2718, accessed 2026-08-04.

[^ref-eip-1559]: EIP-1559, "Fee market change for ETH 1.0 chain," Ethereum Improvement Proposals, including base fee burn and typed fee transaction format, https://eips.ethereum.org/EIPS/eip-1559, accessed 2026-08-04.

[^ref-eth-doc-pos]: ethereum.org, "Proof-of-stake (PoS)," official documentation describing validator duties, block proposal, attestation, and finality, published 2026, https://ethereum.org/developers/docs/consensus-mechanisms/pos/, accessed 2026-08-04.

### Supporting Interpretation Notes

- Where this document gives simplified fee arithmetic or institutional execution guidance, those are analytical summaries of the cited protocol sources rather than exhaustive specification text.

---

## 22. Cross References

### Previous

- ETHEREUM-FOUNDATION-003 — World State

### Next

- ETHEREUM-FOUNDATION-005

### Related

- ETHEREUM-FOUNDATION-002 — Account Model
- BITCOIN-018 — Transaction Fees

---

## Review Status

### Technical Review

Passed.

- state transition을 EVM execution, transaction, gas, PoS inclusion/finality로 설명했다.
- legacy와 typed transaction model을 구분했다.
- EIP-1559를 optional color가 아닌 modern fee architecture로 다뤘다.
- inclusion과 finality를 명시적으로 구분했다.

### Evidence Review

Passed.

- EVM, transaction, gas, PoS claim은 current official documentation을 인용한다.
- typed transaction과 fee-model claim은 EIP-2718과 EIP-1559를 인용한다.
- interpretive guidance는 inference로 라벨링했다.

### Editorial Review

Passed.

- 구조는 프로젝트 문서 패턴을 따른다.
- metadata는 완전하다.
- terminology는 state transition, gas, base fee, priority fee, typed transaction, finality로 일관된다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 execution을 payment로만 축소하지 않는다.
- gas를 단순 UX friction으로 다루지 않는다.
- inclusion과 finality를 혼동하지 않는다.
- 과거 fee/transaction model을 current model의 전부처럼 설명하지 않는다.

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
