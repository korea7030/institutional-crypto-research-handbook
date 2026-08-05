---
knowledge_id: BITCOIN-028
title: Security Budget
subtitle: 채굴자 보상, subsidy+fee 수익, attack-cost proxy, confirmation security, 그리고 단순 수익 지표의 한계
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 125 min
estimated_study: 360 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Economics
  - Security
  - Mining
parent:
  - Bitcoin Economics
prerequisites:
  - BITCOIN-007
  - BITCOIN-020
  - BITCOIN-025
  - BITCOIN-026
  - BITCOIN-027
  - POW-011
  - POW-014
related_topics:
  - Block Subsidy
  - Fee Market
  - Hashrate
  - Attack Cost
  - Confirmation Security
  - Chain Reorganization
  - Mining Economics
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-BLOCKCHAIN-001
  - REF-BTC-DEV-OPERATING-MODES-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-BLOCKASSEMBLER-001
  - REF-BTC-CORE-CHAINPARAMS-001
  - REF-BTC-CORE-AMOUNT-001
tags:
  - bitcoin
  - economics
  - security-budget
  - mining
  - subsidy
  - fees
  - hashrate
  - confirmations
---

# Security Budget
> Bitcoin Economics  
> Research Unit: BITCOIN-028

---

## Research Brief

```yaml
knowledge_id: BITCOIN-028
title: Security Budget
research_question: >
  What does "security budget" mean in Bitcoin, how should analysts relate miner
  compensation from subsidy and fees to confirmation security and attack cost,
  and where are the boundaries between protocol facts, economic proxies, and
  claims that the network is or is not sufficiently secure?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-007
  - BITCOIN-020
  - BITCOIN-025
  - BITCOIN-026
  - BITCOIN-027
  - POW-011
  - POW-014
parent: Bitcoin Economics
previous: BITCOIN-027
next: BITCOIN-029
related_topics:
  - Miner Revenue
  - Reorg Risk
  - Double-Spend Resistance
  - Cumulative Work
  - Fee Share
  - Hashprice
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
  - Precise real-time energy market modeling
  - Full industrial-mining cost accounting
  - Nation-state strategic scenarios
  - Altchain security-budget comparisons
  - Deterministic valuation models
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Bitcoin security budget을 honest proof-of-work 참여를 지탱하는 채굴자 보상으로 정의할 수 있다.
- 명목상 BTC 표시 보상과 실제 경제적 공격 저항성을 구분할 수 있다.
- subsidy + fee가 유용하지만 불완전한 security proxy인 이유를 설명할 수 있다.
- confirmation과 cumulative work가 replacement cost와 어떻게 연결되는지 설명할 수 있다.
- fee share와 total security budget이 서로 다른 metric인 이유를 설명할 수 있다.
- attack cost가 단일 block reward보다 더 많은 변수에 달려 있는 이유를 설명할 수 있다.
- security analysis에서 protocol-determined quantity와 market-determined quantity를 분리할 수 있다.

---

## 2. 핵심 질문

1. Bitcoin security budget이란 무엇인가?
2. 왜 miner compensation이 security의 중심인가?
3. security budget은 subsidy + fee와 같은가?
4. security budget이 높다고 해서 왜 기계적으로 perfect security가 되는 것은 아닌가?
5. 높은 fee share가 왜 adequate security를 자동으로 의미하지 않는가?
6. confirmation은 공격의 경제적 비용과 어떻게 연결되는가?
7. consensus가 보장할 수 있는 것은 무엇이고, 시장이 공급해야 하는 것은 무엇인가?
8. 기관은 Bitcoin security condition을 평가할 때 무엇을 측정해야 하는가?

---

## 3. Executive Summary

Bitcoin 분석에서 "security budget"은 보통 active chain의 valid block에서 miner가 얻는 보상, 즉 block subsidy와 transaction fee를 뜻한다. 이는 유용한 출발점이다. honest proof-of-work security는 지속적인 miner participation에 의존하고, miner participation은 보상이 비용 대비 충분한가에 달려 있기 때문이다.[^ref-btc-wp] [^ref-btc-core-validation] [^ref-btc-core-blockassembler]

하지만 security budget이 security와 같은 것은 아니다. 프로토콜은 nominal block reward quantity를 결정할 수 있지만, 다음은 결정할 수 없다.

- miner cost structure
- hardware availability
- electricity price
- hash-rate centralization
- BTC market value
- attacker motivation
- 특정 시점의 compensation sufficiency

따라서 올바른 framing은 다음과 같다.

- subsidy와 fee는 protocol-visible revenue component다.
- cumulative work와 confirmation은 security evidence다.
- attack cost는 economic inference다.
- security sufficiency는 consensus field가 아니라 judgment다.

subsidy가 era를 거치며 감소할수록 fee는 security budget에서 더 중요해진다. 하지만 rising fee share만으로 네트워크가 잘 자금을 공급받고 있다는 뜻은 아니다. 분석가는 relative measure와 absolute measure를 모두 봐야 한다.

---

## 4. 프로토콜 구조

### Miner Compensation으로서의 Security Budget

가장 단순하게는:

```text
security_budget_per_block = subsidy + transaction_fees
```

다만 코드에 "security budget"이라는 이름의 field가 있는 것은 아니다. 이는 프로토콜에서 보이는 miner revenue component를 바탕으로 만든 analytical identity다.[^ref-btc-core-validation] [^ref-btc-core-blockassembler]

### 왜 중요한가

Bitcoin security는 honest miner가 proof of work로 valid chain을 연장하는 데서 나온다. honest mining이 경제적으로 충분히 보상된다면, recent history를 대체하는 비용은 기대값 기준 더 커진다. compensation이 cost에 비해 줄어들면 네트워크는 여전히 동작할 수 있지만 security margin은 좁아질 수 있다.

### Active-Chain Qualification

active chain의 reward만이 realized miner compensation으로 계산된다. stale block이 valid했더라도 fork race에서 지면 그 subsidy와 fee는 main-chain revenue가 되지 않는다.

---

## 5. Revenue Component

### Subsidy

subsidy는 consensus가 강제하는 height-based issuance schedule에 따라 새로 만들어지는 BTC다.[^ref-btc-core-validation] [^ref-btc-core-chainparams]

### Fee

fee는 coinbase claim ceiling을 통해 transaction spender가 miner에게 넘기는 기존 BTC다. 새로운 공급을 만들지는 않지만 miner compensation은 늘린다.[^ref-btc-dev-blockchain] [^ref-btc-core-validation]

### Block Reward

valid block의 coinbase claim ceiling은:

```text
max_coinbase_value = subsidy + included_fees
```

즉, block reward는 채굴된 block이 제공하는 protocol revenue opportunity이고, security budget은 그 revenue opportunity를 시간과 active chain 전체에서 해석하는 더 넓은 분석 개념이다.

---

## 6. Technical Mechanics

### Coinbase Reward Enforcement

Bitcoin Core는 subsidy component를 위해 `GetBlockSubsidy`를 노출하고, block validation은 coinbase value가 subsidy + fee를 초과하지 않도록 강제한다.[^ref-btc-core-validation]

### Block Assembly에서 Fee 실현

Bitcoin Core의 block assembly는 candidate block을 구성하면서 누적 fee를 추적한다. `BlockAssembler`는 선택된 transaction을 template에 추가할 때 `nFees`를 유지하며, 이는 fee revenue가 miner economics에 어떻게 들어가는지 운영적으로 보여 준다.[^ref-btc-core-blockassembler]

### Amount Bound

`CAmount`, `COIN`, `MoneyRange`는 validation logic 전반에서 사용되는 monetary range와 integer accounting primitive를 정의한다. 이 amount primitive는 올바른 reward accounting에는 중요하지만, 그 자체로 security adequacy를 결정하지는 않는다.[^ref-btc-core-amount]

### Confirmation Security Link

백서와 operating-modes 문서는 모두 transaction이 담긴 block 위에 더 많은 work가 쌓일수록 그 transaction을 뒤집는 비용이 커진다고 설명한다. 그래서 receiver에게는 confirmation 수가 전략적으로 중요하다.[^ref-btc-wp] [^ref-btc-dev-operating-modes]

---

## 7. Security Assumptions and Failure Modes

### Revenue는 Proxy이지 Guarantee가 아니다

높은 nominal security budget은 honest mining incentive를 강화하지만, 실제 security는 다음에도 의존한다.

- total 및 distributed hash rate
- miner concentration
- attacker의 유동성과 자금 조달
- short-term opportunity cost
- direct/indirect censorship incentive
- 공격 대상 transaction의 depth

### Subsidy Decline

halving으로 subsidy가 감소하면 시스템은 더 많이 fee compensation에 기대게 된다. 이는 구조적 전환이지, 자동적인 weakness의 증거가 아니다. 핵심은 total compensation이 attack opportunity에 비해 충분히 큰가이다.

### Centralization Risk

aggregate miner revenue가 커 보여도 hash power가 과도하게 집중되거나 전략적으로 정렬되면 security는 약해질 수 있다. revenue quantity와 revenue distribution은 서로 다른 risk dimension이다.

### Fee Volatility

fee-dominated security budget은 subsidy-dominated security budget보다 더 변동적일 수 있다. 장기 평균이 괜찮아 보여도, 보상이 크게 흔들리면 effective security가 강한 기간과 약한 기간이 생길 수 있다.

---

## 8. Mathematical or Economic Model

### Per-Block Budget

다음을 두자.

- `S` = subsidy
- `F` = included transaction fee

그러면:

```text
B_block = S + F
```

여기서 `B_block`은 nominal per-block security-budget proxy다.

### Interval Budget

여러 accepted block 구간에 대해:

```text
B_interval = sum(S_i + F_i)
```

이는 일별, 주별, era별 비교에 유용하다.

### Fee Share

```text
fee_share = F / (S + F)
```

fee share는 composition을 나타낼 뿐 adequacy를 말하지 않는다. total reward가 작은데 fee share만 60%여도 total compensation이 약할 수 있다.

### Confirmation과 Replacement Cost

target transaction이 inclusion block 이후 cumulative public work `W_pub` 아래에 있다면, 성공적인 replacement attack은 일반적으로 relevant fork point에서 public branch를 overtaking하는 valid alternative branch를 요구한다.

```text
W_alt > W_pub
```

그래서 confirmation은 rising replacement cost의 evidence다. 하지만 work difference 역시 full attack-cost model은 아니다. 비용과 기회는 외부 경제 변수이기 때문이다.

---

## 9. Validation Boundaries

### Consensus가 아는 것

consensus는 다음을 결정할 수 있다.

- 특정 height의 valid subsidy
- confirmed block의 fee
- coinbase overclaim 여부
- 알려진 branch의 cumulative work
- active chain의 confirmation depth

### Consensus가 모르는 것

consensus는 다음을 모른다.

- electricity price
- ASIC depreciation
- miner debt burden
- attacker financing
- regulatory coercion
- 특정 compensation level이 "충분한지" 여부

### 함의

security-budget analysis는 필연적으로 protocol fact와 economic interpretation의 혼합이다.

---

## 10. Bitcoin Core 구현

### `validation`

`validation.h`는 consensus rule 아래 miner compensation의 subsidy component를 정의하는 `GetBlockSubsidy`를 노출한다.[^ref-btc-core-validation]

### `BlockAssembler`

`node::BlockAssembler`는 candidate block을 구성하면서 `nFees`를 유지한다. 이는 block production logic에서 fee component가 miner revenue로 반영되는 방식이다.[^ref-btc-core-blockassembler]

### `kernel/chainparams`

chain parameter는 subsidy-halving interval을 결정하며, subsidy-heavy regime에서 fee-heavier regime로의 장기 security-budget transition shape을 규정한다.[^ref-btc-core-chainparams]

### `consensus/amount.h`

`CAmount`, `COIN`, range checking은 valid reward와 amount를 계산할 때 쓰는 monetary accounting substrate다.[^ref-btc-core-amount]

---

## 11. 온체인 함의

### 직접 측정 가능한 것

active-chain data에서 분석가는 다음을 직접 계산할 수 있다.

- block당 subsidy
- block당 fee
- total block reward
- fee share
- 일별 또는 epoch별 aggregate miner revenue in BTC
- confirmation depth와 chainwork progress

### 온체인에서 직접 측정할 수 없는 것

온체인 데이터만으로는 보통 다음을 직접 측정할 수 없다.

- miner operating cost
- hedging strategy
- effective profit margin
- attacker capital access
- off-chain side payment
- beneficial ownership 기준 miner concentration

### Reporting Standard

serious security-budget report는 다음을 구분해야 한다.

- BTC-denominated reward
- fiat-denominated reward
- fee share
- realized vs projected budget
- chainwork 또는 confirmation evidence
- attack-cost interpretation confidence

---

## 12. Institutional Thinking

기관은 "security budget"을 마법 같은 단일 scalar로 다루면 안 된다.

### Practical Implications

- subsidy, fee, fee share를 함께 보되 서로 대체어처럼 쓰지 말아야 한다.
- miner compensation의 composition과 absolute magnitude를 모두 보고해야 한다.
- confirmation policy는 folklore가 아니라 work와 risk tolerance에 연결해야 한다.
- revenue metric과 함께 reorg, stale rate, miner concentration도 모니터링해야 한다.
- 결론이 off-chain cost assumption에 의존할 때는 이를 명확히 밝혀야 한다.

### Better Security Framing

엄격한 내부 모델은 보통 최소 세 계층이 필요하다.

- protocol layer: subsidy, fee, confirmation, chainwork
- market layer: BTC price, fee demand, hash-rate response
- adversarial layer: concentration, attack incentive, replacement opportunity

---

## 13. Common Misinterpretations

### "Security Budget = Security"

틀렸다. 이는 miner compensation의 proxy이지 complete security proof가 아니다.

### "High Fee Share면 Subsidy 없이도 안전하다"

틀렸다. fee share는 relative composition이지 total economic sufficiency가 아니다.

### "Low Subsidy면 즉시 불안전하다"

틀렸다. security는 subsidy 자체가 아니라 total compensation과 attack cost에 달려 있다.

### "Six Confirmations은 프로토콜 보장이다"

틀렸다. 이는 cost intuition에 기반한 실무적 관행이지 consensus constant가 아니다.[^ref-btc-dev-operating-modes]

### "온체인 수익 지표만으로 attack cost를 전부 알 수 있다"

틀렸다. attack cost는 많은 off-chain variable에 달려 있다.

---

## 14. Research Questions

1. BTC-denominated reward, fiat reward, chainwork metric 중 어떤 조합이 관측된 reorg resistance를 가장 잘 설명하는가?
2. fee volatility는 short-horizon security-budget uncertainty를 얼마나 키우는가?
3. 기관은 security-budget dashboard에 miner concentration을 어떻게 반영해야 하는가?
4. fee-dominated regime와 subsidy-dominated regime에서 confirmation policy는 어떻게 달라져야 하는가?
5. nominal miner compensation이 effective attack cost와 decoupling되는 신호로 어떤 leading indicator가 유용한가?

---

## 15. Practical Exercises

### Exercise 1

최근 block sample에서 일별 miner compensation을 계산하고 subsidy와 fee로 분리하라.

### Exercise 2

fee share는 비슷하지만 total BTC-denominated reward는 다른 두 기간을 비교하고, 왜 security implication이 다를 수 있는지 설명하라.

### Exercise 3

특정 transaction depth에 대해 cumulative work와 confirmation count가 replacement-cost reasoning에 어떻게 기여하는지 설명하되, exact attack probability를 주장하지 말라.

### Exercise 4

protocol fact와 inferred economic variable을 분리하는 security-budget dashboard schema를 초안하라.

---

## 16. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Subsidy and fee reward components | Directly specified | Whitepaper, dev docs, and Core validation / block assembly |
| Confirmation and cumulative-work security logic | Directly specified | Whitepaper and operating-modes guide |
| Security-budget-as-proxy framing | Inference from sources | Standard analytical interpretation of miner compensation |
| Sufficiency and attack-cost claims | Economic inference | Require off-chain assumptions beyond consensus |

---

## 17. Knowledge Graph

```text
Security Budget
├─ Revenue Components
│  ├─ subsidy
│  ├─ fees
│  └─ block reward
├─ Security Evidence
│  ├─ confirmations
│  ├─ cumulative work
│  └─ active-chain persistence
├─ Economic Interpretation
│  ├─ fiat value
│  ├─ miner costs
│  ├─ fee share
│  └─ attack cost
├─ Risks
│  ├─ subsidy decline
│  ├─ fee volatility
│  ├─ concentration
│  └─ reorg exposure
└─ Implementation
   ├─ GetBlockSubsidy
   ├─ BlockAssembler.nFees
   ├─ chainparams
   └─ amount accounting
```

---

## 18. 참고문헌

### Primary Sources

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," especially Sections 6 and 11. https://bitcoin.org/bitcoin.pdf
[^ref-btc-dev-blockchain]: Bitcoin Developer Reference, "Block Chain," including subsidy and fee description. https://developer.bitcoin.org/reference/block_chain.html
[^ref-btc-dev-operating-modes]: Bitcoin Developer Guide, "Operating Modes," including full-node and SPV security discussion. https://developer.bitcoin.org/devguide/operating_modes.html
[^ref-btc-core-validation]: Bitcoin Core Doxygen, `validation.h`, including `GetBlockSubsidy`. https://doxygen.bitcoincore.org/validation_8h_source.html
[^ref-btc-core-blockassembler]: Bitcoin Core Doxygen, `node::BlockAssembler`, including fee accumulation via `nFees`. https://doxygen.bitcoincore.org/classnode_1_1_block_assembler.html
[^ref-btc-core-chainparams]: Bitcoin Core Doxygen, `kernel/chainparams.cpp`, including subsidy-halving parameters. https://doxygen.bitcoincore.org/kernel_2chainparams_8cpp_source.html
[^ref-btc-core-amount]: Bitcoin Core Doxygen, `consensus/amount.h`, including `CAmount`, `COIN`, and `MoneyRange`. https://doxygen.bitcoincore.org/amount_8h_source.html

### Supporting Interpretation Notes

- Where this document discusses sufficiency, attack cost, miner margin, or concentration-adjusted security, those claims are analytical inferences that combine protocol-visible quantities with off-chain economic assumptions.

---

## 19. 교차 참조

### Previous

- BITCOIN-027 — Fee Market

### Next

- BITCOIN-029 — Bitcoin Game Theory

### Related

- BITCOIN-007 — Whitepaper Section 6 — Incentive
- BITCOIN-012 — Whitepaper Section 11 — Calculations
- BITCOIN-020 — Mining
- BITCOIN-024 — Chain Reorganization
- BITCOIN-025 — Bitcoin Monetary Policy
- BITCOIN-026 — Halving
- BITCOIN-027 — Fee Market
- BITCOIN-029 — Bitcoin Game Theory
- POW-011 — Cumulative Chainwork
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- subsidy, fee, block reward, confirmation, attack-cost interpretation을 분리했다.
- security budget을 protocol field가 아니라 analytical proxy로 다뤘다.
- confirmation security를 cumulative work와 연결하되 exact attack probability를 과장하지 않았다.
- 구현 참조는 miner compensation과 직접 관련된 reward-accounting, block-assembly surface로 제한했다.

### Evidence Review

Passed.

- 백서와 developer 문서가 reward와 confirmation-security foundation을 뒷받침한다.
- Core validation과 block-assembly reference가 subsidy와 fee accounting을 뒷받침한다.
- economic sufficiency claim은 해석으로 명시했다.

### Editorial Review

Passed.

- 구조는 프로젝트 deep-dive format을 따른다.
- metadata가 완전하다.
- security budget, subsidy, fee, fee share, confirmation, cumulative work 용어가 일관된다.
- table과 code fence가 닫혀 있다.

### Adversarial Review

Passed.

- security budget을 complete security와 동일시하지 않았다.
- fee share만으로 adequacy가 증명된다고 하지 않았다.
- six confirmations을 consensus constant로 만들지 않았다.
- 온체인 데이터만으로 full attack cost를 알 수 있다고 주장하지 않았다.
- concentration risk를 무시하지 않았다.

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
