---
knowledge_id: BITCOIN-029
title: Bitcoin Game Theory
subtitle: incentive compatibility, honest mining, strategic deviation, pool formation, fee sniping, 그리고 단순한 equilibrium claim의 한계
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 130 min
estimated_study: 380 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Economics
  - Game Theory
  - Mining
parent:
  - Bitcoin Economics
prerequisites:
  - BITCOIN-007
  - BITCOIN-020
  - BITCOIN-027
  - BITCOIN-028
  - POW-014
related_topics:
  - Honest Mining
  - Selfish Mining
  - Fee Sniping
  - Mining Pools
  - Security Budget
  - Reorg Incentives
  - Confirmation Policy
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-OPERATING-MODES-001
  - REF-BTC-CORE-BLOCKASSEMBLER-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-EYAL-SIRER-SELFISH-001
  - REF-NIST-SELFISH-001
tags:
  - bitcoin
  - economics
  - game-theory
  - mining
  - incentives
  - selfish-mining
  - fee-sniping
  - pools
---

# Bitcoin Game Theory
> Bitcoin Economics  
> Research Unit: BITCOIN-029

---

## Research Brief

```yaml
knowledge_id: BITCOIN-029
title: Bitcoin Game Theory
research_question: >
  How should Bitcoin be analyzed as a game among miners, users, pools, and
  validators; when is honest behavior incentive-compatible; what strategic
  deviations such as selfish mining or fee sniping challenge simple honest
  equilibrium stories; and how should analysts separate protocol rules from
  payoff assumptions?
document_type: deep-dive
difficulty: L400
prerequisites:
  - BITCOIN-007
  - BITCOIN-020
  - BITCOIN-027
  - BITCOIN-028
  - POW-014
parent: Bitcoin Economics
previous: BITCOIN-028
next: BITCOIN-030
related_topics:
  - Mining Pools
  - Fee Market
  - Security Budget
  - Attack Models
  - Confirmation Security
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
  - Formal theorem proofs
  - Full survey of all blockchain game-theory literature
  - Altchain strategic-comparison analysis
  - National-jurisdiction regulation games
  - High-frequency trading microstructure on exchanges
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Bitcoin security가 암호학과 validation뿐 아니라 incentive에도 의존하는 이유를 설명할 수 있다.
- protocol rule과 participant behavior에 대한 equilibrium claim을 구분할 수 있다.
- 백서의 honest-mining argument와 그 가정을 설명할 수 있다.
- selfish mining이 direct consensus break가 아니라 incentive-compatibility challenge인 이유를 식별할 수 있다.
- fee sniping을 unusually valuable recent block과 연결된 reorg incentive로 설명할 수 있다.
- pool formation이 개별적으로는 합리적이지만 시스템 전체적으로는 위험할 수 있는 이유를 설명할 수 있다.
- miner, user, validator의 incentive를 하나의 actor model로 뭉개지 않고 분리할 수 있다.

---

## 2. 핵심 질문

1. "Bitcoin game theory"는 실제로 무엇을 의미하는가?
2. 왜 protocol validity만으로 시스템 behavior를 설명할 수 없는가?
3. honest mining에 대한 백서의 incentive claim은 무엇인가?
4. selfish mining이란 무엇이며, 왜 game-theoretic한가?
5. fee sniping이란 무엇이며, 언제 매력적일 수 있는가?
6. decentralization이 사회적으로 선호되더라도 왜 mining pool은 커질 수 있는가?
7. transaction value와 threat model에 따라 confirmation policy가 왜 달라지는가?
8. incentive에 대해 온체인 데이터가 보여 줄 수 있는 것은 무엇이고, 여전히 추론에 남는 것은 무엇인가?

---

## 3. Executive Summary

Bitcoin은 valid/invalid message만 있는 프로토콜이 아니다. miner, pool, user, wallet, merchant, node operator가 reward, cost, delay, risk에 반응하는 전략적 환경이기도 하다. "Bitcoin game theory"는 이러한 incentive와 strategic response를 연구하는 것이다.

백서는 핵심 incentive argument를 포함한다. 충분한 hash power를 가진 공격자라면, 사용자를 속이고 시스템을 훼손하는 것보다 규칙을 따르며 새로운 coin을 받는 편이 더 이익이어야 한다는 주장이다.[^ref-btc-wp] 이는 강력한 설계 직관이지만, 모든 네트워크·시장·조직 조건에서 honest behavior가 항상 dominant strategy라는 완전한 증명은 아니다.

이후 연구, 특히 Eyal과 Sirer의 selfish-mining 연구는 Bitcoin mining이 모든 가정에서 fully incentive-compatible하지는 않다고 주장한다. 그들의 모델에서는 miner나 pool이 total hash power의 과반이 아니어도 block을 전략적으로 withheld했다가 나중에 공개함으로써 fair share보다 높은 수익을 얻을 수 있다.[^ref-eyal-sirer-selfish] 이후 NIST의 survey 성격 작업도 selfish-mining profitability가 difficulty-adjustment algorithm과 모델 가정에 민감함을 보여 주며, 이런 문제는 직관만으로 해결되지 않는다고 강조한다.[^ref-nist-selfish]

분석가에게 실무적 결론은 다음과 같다.

- Bitcoin rule은 무엇이 valid한지 정의한다.
- incentive는 actor가 무엇을 선택하는지에 영향을 준다.
- 관측된 행동은 둘 모두에 의존한다.
- "honest majority"는 "universal incentive compatibility"와 같은 말이 아니다.

---

## 4. 프로토콜 구조

### Strategic Actor

Bitcoin의 주요 strategic actor는 다음을 포함한다.

- hash power를 어떻게 배분할지 결정하는 miner와 pool
- fee와 confirmation threshold를 정하는 user와 wallet
- policy setting과 validation behavior를 정하는 full node
- settlement acceptance rule을 정하는 business

각 집단은 서로 다른 objective function을 가진다.

### Incentive Layer

분석적으로 Bitcoin의 게임은 다음 계층으로 나눌 수 있다.

| Layer | Core Question |
|---|---|
| Consensus | 어떤 block과 transaction이 valid한가? |
| Mining | 어떤 valid block 위에서 채굴하고, 발견한 block을 언제 공개할 것인가? |
| Relay / fee market | 어떤 transaction을 relay, replace, mine할 것인가? |
| Settlement | 이 counterparty와 threat model에서 confirmation 몇 개가 충분한가? |

### Protocol vs Equilibrium

프로토콜은 다음을 규정할 수 있다.

- valid subsidy
- valid fee
- cumulative work 기준 valid chain selection
- valid transaction과 block structure

프로토콜이 직접 규정하지 못하는 것:

- miner가 block을 즉시 공개하는지 여부
- miner가 pool에 참여하는지 여부
- user가 fee를 과지급/과소지급하는지 여부
- merchant가 zero-conf를 수락하는지, 많은 confirmation을 기다리는지 여부

이들은 strategic choice다.

---

## 5. Honest Mining Incentive

### Whitepaper Argument

백서의 incentive section은 substantial CPU power를 가진 miner에게 두 가지 선택이 있다고 말한다.

- 시스템을 공격하고 사용자를 속일 것인가
- 아니면 정직하게 행동하며 새 coin과 fee를 받을 것인가[^ref-btc-wp]

핵심 직관은, 공격자가 시스템의 장기 가치도 함께 생각한다면 honest play가 cheating보다 우세해야 한다는 것이다.

### 이 주장이 의존하는 것

이 incentive claim은 암묵적으로 다음에 의존한다.

- attacker의 discount rate
- 표적 transaction의 가치
- attacker의 BTC exposure
- reputational 또는 market damage 가능성
- attacker의 private loss 감내 능력
- network topology와 propagation environment

즉, 백서는 strategic design argument를 제시한 것이지 universal equilibrium theorem을 증명한 것은 아니다.

### Honest Mining as Baseline

실무적으로 "honest mining"은 보통 다음을 의미한다.

- best known valid tip 위에서 채굴
- 발견한 block을 즉시 공개
- branch asymmetry를 의도적으로 만들지 않고 subsidy와 fee를 수취

이 baseline은 network convergence에 효율적이며, 대부분의 운영 가정에서 normal case로 놓인다.

---

## 6. Strategic Deviation

### Selfish Mining

selfish mining은 miner나 pool이 발견한 block을 withheld했다가 전략적으로 공개해, honest miner의 work를 낭비시키고 자신의 revenue share를 높이려는 deviation이다. 여기서 핵심은 direct consensus invalidity가 아니라, 어떤 parameter에서는 이런 deviation이 honest mining보다 더 수익성이 있을 수 있는가 하는 점이다.[^ref-eyal-sirer-selfish]

### Fee Sniping

fee sniping은 unusually large fee를 담은 recent block을 짧은 reorg로 대체하려는 incentive다. 그 block을 교체해 얻는 기대 이익이 비용과 리스크보다 크다면, miner는 현재 tip을 연장하는 대신 이전 block 위에서 채굴하려 할 수 있다.

이는 selfish mining과 같지 않다.

- selfish mining은 publication timing과 private lead를 이용한다.
- fee sniping은 unusually large recent reward opportunity를 이용한다.

### Pool Formation

mining pool은 개별 miner의 variance를 줄여 준다. pool 참여는 개별적으로는 합리적이지만, 더 큰 pool은 concentration과 systemic risk를 늘릴 수 있다. 이는 전형적인 local-vs-global incentive tension이다.

### Confirmation Acceptance

user와 business도 전략 게임을 한다. 적은 confirmation을 받아들이면 UX와 working-capital turnover는 좋아지지만 reversal risk가 늘어난다. 더 오래 기다리면 risk는 줄지만 latency와 cost가 커진다.

---

## 7. Technical Mechanics

### Block Reward와 Strategic Payoff

Bitcoin Core validation과 block assembly는 miner가 최적화하는 raw component를 노출한다.

- subsidy를 위한 `GetBlockSubsidy`
- block assembly에서의 accumulated block fee
- valid cumulative work 기반 chain selection[^ref-btc-core-validation] [^ref-btc-core-blockassembler]

이 자체가 game theory는 아니지만 payoff surface를 정의한다.

### Confirmation Depth as Strategic Input

operating-modes 문서는 block 위에 더 많은 cumulative difficulty가 쌓일수록 뒤집는 비용이 커진다고 설명한다. 그래서 receiver에게 confirmation은 전략적 의미를 가진다.[^ref-btc-dev-operating-modes]

### Publication Timing Matters

consensus는 새로 찾은 valid block을 즉시 공개하도록 강제하지 않는다. 보통은 prompt publication이 표준 전략이지만, block-withholding model은 timing 자체가 strategic variable임을 보여 준다. 이 지점에서 network propagation과 game theory가 만난다.

---

## 8. Validation Boundaries

### Valid가 Incentive-Compatible을 뜻하지는 않는다

프로토콜은 block을 올바르게 validate할 수 있으면서도 incentive problem을 허용할 수 있다. selfish mining이 바로 이 구분을 보여 준다. 공격은 valid protocol behavior 안에서 이루어지지만, strategic timing으로 revenue를 높이려 한다.

### Incentive-Compatible이 Attack-Proof를 뜻하지는 않는다

common assumption 아래 honest mining이 locally rational하더라도, adversary는 다음과 같은 이유로 다르게 행동할 수 있다.

- disruption 자체를 가치 있게 볼 수 있다.
- 외부에서 보조금을 받을 수 있다.
- 비경제적 동기를 가질 수 있다.
- temporary fee spike나 coordination failure를 악용할 수 있다.

### Model Sensitivity

threshold와 profitability에 대한 game-theoretic claim은 다음 가정에 민감하다.

- propagation advantage
- tie-breaking behavior
- difficulty adjustment
- pool coordination
- fee composition
- market response

따라서 문헌의 특정 threshold 하나를 universal truth처럼 취급해서는 안 된다.

---

## 9. Security Assumptions and Failure Modes

### Selfish Mining과 Incentive Compatibility

Eyal과 Sirer의 결과가 중요한 이유는 minority collusion이 언제나 경제적으로 자기파괴적이라는 편안한 heuristic에 도전하기 때문이다. 그들의 주장은 Bitcoin이 즉시 실패한다는 것이 아니라, 일부 가정 아래에서는 majority share보다 낮아도 profitable deviation이 존재할 수 있다는 점이다.[^ref-eyal-sirer-selfish]

### Concentration Feedback Loop

전략적 deviation이 excess profit을 주면, 더 많은 miner가 그 deviating pool에 합류하는 것이 합리적일 수 있다. 이는 정적 attack description이 아니라 game-theoretic feedback problem이다.

### Fee-Driven Reorg Incentive

큰 fee spike는 최근 history를 재구성하려는 short-horizon incentive를 만들 수 있다. 특히 reorg target이 shallow하고 fee windfall이 exceptional할 때 그렇다.

### User-Level Strategic Error

high-value flow에서 low-confirmation settlement를 final로 취급하는 business는, 프로토콜이 의도대로 동작하더라도 게임 안에서 경제적으로 약한 counterparty가 될 수 있다.

---

## 10. Mathematical or Economic Model

### Honest Revenue Benchmark

miner의 hash share가 `alpha`라면, naive honest-mining revenue expectation은 대략:

```text
expected_revenue_share ≈ alpha
```

이것이 proportional baseline이다.

### Strategic Deviation Condition

game-theoretic deviation이 흥미로워지는 조건은:

```text
expected_revenue_share_under_strategy > alpha
```

또는 더 일반적으로:

```text
expected_utility_under_strategy > expected_utility_under_honest_mining
```

utility는 다음을 포함할 수 있다.

- immediate BTC reward
- variance reduction
- capital constraint
- reorg opportunity
- censorship/disruption에서 오는 external gain

### Receiver Strategy

merchant나 exchange에게는:

```text
expected_loss_from_early_acceptance
vs
expected_cost_of_waiting_for_more_confirmations
```

이 역시 game-theoretic tradeoff다. 그들이 채굴을 하지 않더라도 마찬가지다.

---

## 11. Bitcoin Core 구현

### Core는 Rule을 정의할 뿐 Equilibrium을 증명하지 않는다

Bitcoin Core는 miner와 user가 반응하는 rule system과 block-construction logic을 제공한다. 하지만 honest publication이 언제나 optimal이라는 theorem을 코드로 담고 있지는 않다.

### `validation`

`validation.h`는 subsidy logic과 valid chain progression rule surface를 정의한다.[^ref-btc-core-validation]

### `BlockAssembler`

`BlockAssembler`는 fee가 candidate block에서 miner revenue가 되는 과정을 보여 주며, fee competition과 fee sniping incentive를 형성하는 표면을 제공한다.[^ref-btc-core-blockassembler]

### No "Honesty Switch"

Core에는 game-theoretic honesty를 보장하는 설정이 없다. strategic behavior는 외부 participant가 프로토콜을 어떻게 사용할지 결정하면서 나타난다.

---

## 12. 온체인 함의

### 관측 가능한 것

온체인과 public-network data는 때때로 다음을 보여 줄 수 있다.

- reorg depth
- stale-block frequency
- unusual fee concentration
- pool concentration trend
- attack/disruption 이후 confirmation outcome

### 보통은 추론에 남는 것

온체인 데이터만으로는 보통 다음을 증명할 수 없다.

- miner intent
- public하게 드러나지 않은 private withholding
- pool이 비교한 exact payoff
- miner 간 off-chain side agreement
- strategic 행동인지 단순 propagation variance인지 여부

### 왜 중요한가

game-theoretic claim은 evidentiary strength에 따라 등급을 매겨야 한다. branch replacement는 reorg의 strong evidence지만, selfish intent의 weak evidence일 수 있다.

---

## 13. Institutional Thinking

기관은 Bitcoin을 정적인 기계가 아니라 여러 feedback loop를 가진 strategic system으로 보아야 한다.

### Practical Implications

- confirmation policy는 value at risk와 threat model에 따라 달라져야 한다.
- miner concentration monitoring은 technical research report가 아니라 risk dashboard에도 들어가야 한다.
- security-budget metric은 fee spike, stale rate, pool share concentration 같은 incentive-risk indicator와 함께 봐야 한다.
- large-value settlement system은 unusual fee event 주변의 strategic reorg risk를 명시적으로 모델링해야 한다.

### Better Analytical Habit

incentive claim을 할 때는 다음을 명시해야 한다.

- 어떤 actor인지
- 어떤 objective인지
- 어떤 time horizon인지
- 어떤 information set인지
- 어떤 outside option이 있는지

이 정보가 없으면 "rational miner" 같은 표현은 너무 막연해서 신뢰하기 어렵다.

---

## 14. Common Misinterpretations

### "Bitcoin은 honest behavior가 수학적으로 증명되었다"

틀렸다. 강한 incentive design은 있지만, 실제 behavior는 가정과 market condition에 의존한다.

### "Selfish Mining은 Consensus Rule을 깨뜨린다"

틀렸다. 이는 otherwise valid protocol behavior 안에서의 strategic deviation이다.

### "과반 해시레이트만이 유일한 임계값이다"

틀렸다. direct rewrite capability에서는 과반이 중요하지만, 일부 strategic deviation은 특정 가정 아래 50% 미만에서도 매력적일 수 있다.[^ref-eyal-sirer-selfish]

### "Fee Spike는 User에게만 영향을 준다"

틀렸다. 큰 fee spike는 reorg와 block-selection incentive도 바꾼다.

### "Game Theory는 Miner에게만 중요하다"

틀렸다. user, wallet, exchange, merchant도 fee와 confirmation을 둘러싼 strategic tradeoff를 한다.

---

## 15. Research Questions

1. 현재 network/pool condition에서 selfish-mining incentive는 실무적으로 얼마나 강한가?
2. accidental stale event와 strategic withholding을 가장 잘 구분하는 empirical indicator는 무엇인가?
3. 기관은 extreme fee spike 중 fee-sniping risk를 어떻게 모델링해야 하는가?
4. 어떤 수준의 miner concentration이 game-theoretic assumption을 실질적으로 약화시키는가?
5. 어떤 counterparty class가 differentiated confirmation threshold를 써야 하는가?

---

## 16. Practical Exercises

### Exercise 1

백서의 honest-mining incentive claim을 자신의 말로 다시 쓰고, 숨겨진 가정을 목록화하라.

### Exercise 2

honest mining과 selfish mining을 high-level로 비교하라. 새 block을 찾았을 때 각 전략이 무엇을 하는지 적어 보라.

### Exercise 3

low-, medium-, high-value payment에 대한 confirmation policy table을 설계하고, 각 threshold 뒤의 strategic reasoning을 설명하라.

### Exercise 4

의심되는 fee-sniping 또는 strategic-withholding event에 대한 incident template을 작성하되, observed fact와 inferred motive를 분리하라.

---

## 17. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Whitepaper incentive argument | Directly specified | Original design paper |
| Confirmation-security intuition | Directly specified | Whitepaper and operating-modes guide |
| Selfish-mining incentive challenge | Directly specified | Eyal and Sirer; NIST survey of profitability context |
| Pool-concentration and fee-sniping interpretation | Inference from sources | Derived from revenue and reorg incentive structure |
| Universal equilibrium claims | Not supported | Must remain caveated and model-specific |

---

## 18. Knowledge Graph

```text
Bitcoin Game Theory
├─ Baseline Incentives
│  ├─ honest mining
│  ├─ subsidy
│  ├─ fees
│  └─ confirmation security
├─ Strategic Deviations
│  ├─ selfish mining
│  ├─ fee sniping
│  ├─ withholding
│  └─ opportunistic reorgs
├─ Coordination
│  ├─ mining pools
│  ├─ concentration
│  ├─ variance reduction
│  └─ local vs global incentives
├─ User Strategy
│  ├─ confirmation thresholds
│  ├─ fee bidding
│  └─ settlement policy
└─ Limits
   ├─ model sensitivity
   ├─ hidden assumptions
   ├─ off-chain payoffs
   └─ evidence limits
```

---

## 19. 참고문헌

### Primary Sources

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," especially Sections 6 and 11. https://bitcoin.org/bitcoin.pdf
[^ref-btc-dev-operating-modes]: Bitcoin Developer Guide, "Operating Modes," including confirmation-security and cumulative-work discussion. https://developer.bitcoin.org/devguide/operating_modes.html
[^ref-btc-core-blockassembler]: Bitcoin Core Doxygen, `node::BlockAssembler`, including fee accumulation and candidate block construction. https://doxygen.bitcoincore.org/classnode_1_1_block_assembler.html
[^ref-btc-core-validation]: Bitcoin Core Doxygen, `validation.h`, including reward and chain-validation surfaces. https://doxygen.bitcoincore.org/validation_8h_source.html
[^ref-eyal-sirer-selfish]: Ittay Eyal and Emin Gun Sirer, "Majority Is Not Enough: Bitcoin Mining Is Vulnerable," arXiv:1311.0243 / FC 2014. arXiv abstract record: https://scixplorer.org/abs/2013arXiv1311.0243E/abstract ; author publication page: https://www.cs.cornell.edu/people/egs/
[^ref-nist-selfish]: Michael Davidson and Tyler Diamond, "On the Profitability of Selfish Mining Against Multiple Difficulty Adjustment Algorithms," NIST / Cryptology ePrint context page. https://www.nist.gov/publications/profitability-selfish-mining-against-multiple-difficulty-adjustment-algorithms

### Supporting Interpretation Notes

- Where this document discusses fee sniping, concentration feedback loops, or institutional confirmation strategy, those claims are analytical interpretations built on incentive structure and attack-model literature rather than explicit protocol commands.

---

## 20. 교차 참조

### Previous

- BITCOIN-028 — Security Budget

### Next

- BITCOIN-030 — SegWit

### Related

- BITCOIN-007 — Whitepaper Section 6 — Incentive
- BITCOIN-018 — Transaction Fees
- BITCOIN-020 — Mining
- BITCOIN-024 — Chain Reorganization
- BITCOIN-027 — Fee Market
- BITCOIN-028 — Security Budget
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- protocol rule와 incentive claim을 분리했다.
- honest mining, selfish mining, fee sniping, pool formation, confirmation strategy를 서로 다른 strategic problem으로 다뤘다.
- game-theoretic statement를 universal law가 아니라 assumption-dependent claim으로 유지했다.
- 구현 참조는 payoff와 직접 연결되는 reward/block-construction surface로 제한했다.

### Evidence Review

Passed.

- 백서와 developer 문서가 baseline incentive와 confirmation-security claim을 뒷받침한다.
- Core reference가 miner가 반응하는 reward-accounting과 fee-selection surface를 뒷받침한다.
- Eyal-Sirer 및 NIST 문맥 자료가 selfish-mining 관련 incentive-compatibility caveat를 뒷받침한다.
- interpretive claim은 필요한 부분에서 inference로 표시했다.

### Editorial Review

Passed.

- 구조는 프로젝트 deep-dive format을 따른다.
- metadata가 완전하다.
- honest mining, selfish mining, fee sniping, pool formation, confirmation strategy, incentive compatibility 용어가 일관된다.
- table과 code fence가 닫혀 있다.

### Adversarial Review

Passed.

- honest behavior가 형식적으로 보장된다고 과장하지 않았다.
- strategic deviation을 consensus invalidity와 혼동하지 않았다.
- 모든 attack threshold를 50%로 환원하지 않았다.
- chain data만으로 strategic intent가 증명된다고 주장하지 않았다.
- 모든 actor가 같은 payoff function을 가진다고 보지 않았다.

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
