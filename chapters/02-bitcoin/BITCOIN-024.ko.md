---
knowledge_id: BITCOIN-024
title: Chain Reorganization
subtitle: active-chain 교체, fork point, disconnect-and-connect 메커닉, confirmation reversal, 그리고 분석 리스크
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 115 min
estimated_study: 330 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Consensus
  - Chain Selection
  - Reorganization
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-014
  - BITCOIN-017
  - BITCOIN-021
  - BITCOIN-022
  - BITCOIN-023
  - POW-010
  - POW-011
  - POW-013
related_topics:
  - Fork Point
  - Active Chain
  - Chainwork
  - Stale Blocks
  - Mempool Reinsertion
  - UTXO Rollback
  - Double Spend Risk
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-BLOCKCHAIN-001
  - REF-BTC-DEV-OPERATING-MODES-001
  - REF-BTC-CORE-CHAIN-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-TXMEMPOOL-001
  - REF-BTC-CORE-VALIDATIONINTERFACE-001
  - REF-BTC-CORE-CONSENSUS-VALIDATION-001
tags:
  - bitcoin
  - internals
  - consensus
  - reorg
  - chain-selection
  - chainwork
  - mempool
  - utxo
---

# Chain Reorganization
> Bitcoin Internals  
> Research Unit: BITCOIN-024

---

## Research Brief

```yaml
knowledge_id: BITCOIN-024
title: Chain Reorganization
research_question: >
  What is a Bitcoin chain reorganization, how does a node replace one active
  branch with another valid higher-work branch, how are UTXO and mempool state
  updated during the transition, and what does a reorg prove or fail to prove
  to analysts and institutions?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-014
  - BITCOIN-017
  - BITCOIN-021
  - BITCOIN-022
  - BITCOIN-023
  - POW-010
  - POW-011
  - POW-013
parent: Bitcoin Internals
previous: BITCOIN-023
next: BITCOIN-025
related_topics:
  - Fork Resolution
  - UTXO State
  - Mempool Consistency
  - Confirmation Risk
  - Double-Spend Analysis
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
  - Full attacker game theory
  - Exhaustive review of historical Bitcoin reorg incidents
  - Exchange-specific operational policies
  - Altchain finality models
  - Wallet UI design for reorg handling
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- reorg를 단순한 fork가 아니라 active-chain replacement event로 정의할 수 있다.
- fork creation, fork resolution, reorganization을 구분할 수 있다.
- node가 current tip과 better branch 사이의 fork point를 어떻게 찾는지 설명할 수 있다.
- reorg decision이 단순 header count가 아니라 valid higher cumulative work에 기반하는 이유를 설명할 수 있다.
- disconnected block이 UTXO state에 어떤 영향을 미치는지 설명할 수 있다.
- disconnected transaction이 여전히 valid하고 policy-acceptable하다면 mempool로 돌아갈 수 있음을 설명할 수 있다.
- reorg가 malicious intent를 증명하지 않고도 confirmation을 되돌릴 수 있는 이유를 설명할 수 있다.
- reorg handling에 관여하는 Bitcoin Core의 주요 function을 식별할 수 있다.

---

## 2. 핵심 질문

1. Bitcoin chain reorganization이 정확히 무엇인가?
2. 언제 fork가 reorg가 되는가?
3. old branch에서 무엇이 disconnect되고 new branch에서 무엇이 connect되는가?
4. losing branch에서 생성되거나 소비된 UTXO는 어떻게 되는가?
5. disconnected block의 transaction은 어떻게 되는가?
6. 왜 reorg는 confirmation을 되돌릴 수 있는가?
7. 더 높은 work의 header chain이 있다고 해서 왜 즉시 reorg가 보장되지는 않는가?
8. Bitcoin Core는 reorg 동안 mempool consistency를 어떻게 유지하는가?
9. 온체인 데이터는 reorg에 대해 무엇을 보여 줄 수 있고 무엇을 보여 주지 못하는가?
10. 언제 analyst는 ordinary propagation race가 아니라 adversarial behavior를 의심해야 하는가?

---

## 3. Executive Summary

chain reorganization은 node가 현재 active chain의 일부를 버리고, 그것보다 더 큰 cumulative proof of work를 가진 다른 valid branch로 대체하는 상황을 말한다. 이는 fork 자체와는 다르다. fork는 공통 조상 이후 competing branch가 존재하는 상태이고, reorg는 active branch를 실제로 바꾸는 사건이다.[^ref-btc-wp] [^ref-btc-dev-blockchain]

백서의 "longest chain" 설명은 이미 이 replacement logic을 함의한다. 두 block이 동시에 발견되면 node는 일시적으로 불일치할 수 있지만, 결국 한 branch를 더 연장하고 다른 branch를 버린다. 현대 Bitcoin 용어로는 higher-work valid branch가 선택되는 순간 이것이 explicit active-chain switch가 된다.[^ref-btc-wp]

reorg는 header 이상의 상태 변화를 갖는다. node는 disconnected block의 chainstate 변화를 되돌리고, replacement branch의 chainstate 변화를 새로 적용해야 한다. 이는 다음을 뜻한다.

- losing branch에서만 생성된 output은 active UTXO set에서 사라질 수 있다.
- 이전에 spent되었던 output이 다시 spendable해질 수 있다.
- disconnected block의 transaction은 new active chain 아래에서 여전히 valid하면 unconfirmed 상태로 돌아와 mempool에 재진입할 수 있다.[^ref-btc-core-validation] [^ref-btc-core-txmempool]

분석가에게 핵심은 reorg가 관측 가능한 chain behavior라는 점이지, 자동으로 intent의 증거는 아니라는 점이다. one-block reorg는 propagation race로도 자연스럽게 생길 수 있다. 더 깊거나 전략적으로 보이는 reorg는 stronger adversarial hypothesis를 뒷받침할 수 있지만, 의도를 주장하려면 branch replacement 외의 추가 증거가 필요하다.[^ref-btc-dev-blockchain] [^ref-btc-dev-operating-modes]

---

## 4. 프로토콜 구조

### Fork vs Reorg

공통 조상 `H`에서 시작해 보자.

```text
H
├─ A1 ─ A2
└─ B1 ─ B2 ─ B3
```

node가 현재 branch `A` 위에 있다가 나중에 branch `B`가 best valid higher-work branch라는 사실을 알게 되면, node는 `A2`에서 `B3`로 전환할 수 있다. 이 전환이 reorganization이다.

### 최소 정의

reorg에는 세 가지가 필요하다.

1. old/new branch가 공유하는 fork point
2. fork point 이후 disconnect될 current active branch
3. fork point 이후 connect될 better valid branch

### Reorg Depth

reorg depth는 보통 이전 active branch에서 몇 개 block이 제거되었는지로 표현한다. 하지만 분석가는 work difference도 함께 봐야 한다. difficulty regime이 다르면 같은 depth라도 replacement cost는 같지 않기 때문이다.

---

## 5. High-Level Mechanics

### Fork Point 발견

Bitcoin Core는 `CBlockIndex` parent link를 통해 chain history를 표현한다. branch를 바꾸기 위해 node는 먼저 현재 tip과 better candidate tip의 마지막 공통 조상을 찾는다.[^ref-btc-core-chain]

### Old Branch Disconnect

losing branch에서 fork point 이후의 모든 block은 chainstate에서 disconnect되어야 한다. 이 과정은 그 block들의 active effect를 되돌린다.

- spent output이 복원될 수 있다.
- disconnected block에서만 생성된 output은 제거될 수 있다.
- 해당 block의 transaction에 붙어 있던 confirmation은 active chain view에서 사라진다.

### New Branch Connect

이후 node는 fork point 이후 replacement branch의 block을 validate하고 connect한다. 그 branch의 transaction은 새로운 active history에서 confirmed가 된다.

### 결과

이 시퀀스가 끝나면 node는 다시 하나의 active chain만 갖는다. losing branch는 block index data에는 남아 있을 수 있지만, confirmation과 spendability 판단에 사용되는 active history는 아니다.

---

## 6. UTXO와 Transaction 영향

### UTXO Rollback

UTXO set은 모든 known block이 아니라 active chain만 반영한다. 따라서 reorg는 spendable set을 바꾼다.

disconnected block이:

- output `X`를 생성했다면, `X`는 active UTXO set에서 사라질 수 있다.
- output `Y`를 소비했다면, replacement branch가 같은 output을 소비하지 않는 한 `Y`는 다시 unspent가 될 수 있다.

즉, reorg는 header만이 아니라 chainstate까지 반드시 갱신해야 한다.

### Confirmation Reversal

losing branch에서 confirmed였던 transaction은 winning branch에서도 다시 confirmed되지 않는 한 active-chain 관점에서는 unconfirmed가 된다.

이것이 reorg에서 settlement reversal risk가 생기는 프로토콜적 이유다.

### Conflict와 Double Spend

winning branch가 동일 입력을 다른 방식으로 소비하는 conflicting transaction을 포함한다면, losing-branch transaction은 단순히 "pending again"이 되지 않는다. winning chain에서 입력이 이미 소비되었기 때문에 active history에서 영구적으로 제외될 수 있다.

### Coinbase 특수성

코인베이스 트랜잭션은 reorg에서 특히 민감하다. coinbase가 들어 있던 block이 disconnect되면 그 block reward 전체가 active chain에서 사라지기 때문이다. coinbase maturity는 freshly mined reward를 즉시 spendable하게 만들지 않는 이유 중 하나다.

---

## 7. Mempool 영향

### Disconnected Transaction이 자동으로 사라지는 것은 아니다

disconnected block의 transaction은 new active chain 아래에서 여전히 valid하고 policy-compliant하다면 mempool admission 대상으로 다시 고려될 수 있다. Bitcoin Core는 new block connect 이후 또는 reorg 이후 mempool consistency를 갱신하는 메커니즘을 명시적으로 포함한다.[^ref-btc-core-txmempool]

### Reinsertion 조건

disconnected transaction이 mempool로 돌아가려면 다음이 필요하다.

- new chainstate 아래에서 입력이 available할 것
- winning-branch confirmed transaction과 conflict하지 않을 것
- local mempool policy를 여전히 통과할 것
- dependency 조건이 여전히 충족될 것

### Reorg Consistency

Bitcoin Core는 `BlockConnected`, `BlockDisconnected`, mempool-removal notification 같은 validation/mempool callback을 노출한다. 이는 reorg handling이 단순한 내부 bookkeeping이 아니라 downstream consumer가 알아야 할 state transition임을 보여 준다.[^ref-btc-core-validationinterface]

---

## 8. Technical Mechanics

### Most-Work Valid Chain

node는 alternative header path를 봤다고 해서 곧바로 reorganize하지 않는다. candidate branch는 validation을 통과하고 cumulative work 기준 best valid branch가 되어야 한다. header awareness, block validity, active-chain replacement는 서로 다른 단계다.[^ref-btc-core-validation] [^ref-btc-core-consensus-validation]

### Core Reorg Skeleton

상위 수준에서:

```text
find best valid candidate tip
find last common ancestor with active tip
disconnect active blocks after fork point
connect candidate-branch blocks after fork point
update active tip
repair mempool consistency
emit validation callbacks
```

개념상 단순하지만, chainstate, mempool, wallet logic, observer가 모두 일관된 순서를 필요로 하므로 운영상 민감하다.

### External Interface에서의 Atomicity

Bitcoin Core test는 reorg 처리 중 caller가 임의의 불일치 intermediate mempool state를 보지 않도록 신경 쓴다. "내부 intermediate step이 전혀 없다"가 목적이 아니라, 외부에서 관측되는 state transition이 coherent하도록 만드는 것이 목적이다.[^ref-btc-core-validation] [^ref-btc-core-txmempool]

### Reorg Depth vs Work Depth

depth 2의 reorg 두 개가 항상 비슷한 것은 아니다. difficulty, timestamp spacing, branch work가 다르면 replacement effort와 security implication도 달라진다. depth-only reporting보다 work-aware reporting이 더 견고하다.

---

## 9. Validation Boundaries

### Header Competition이 곧 Reorg는 아니다

node는 competing header chain을 알고 있어도 아직 모든 block을 받지 못했거나, activation에 충분한 validation을 마치지 않았을 수 있다. 따라서:

- competing header: possible candidate
- valid full block: stronger candidate
- active-chain switch: actual reorg

### Reorg가 곧 Attack은 아니다

reorg는 프로토콜 동작이다. 작은 reorg는 ordinary propagation race에서도 발생한다. 모든 reorg를 attack으로 부르면 다음 구분이 무너진다.

- accidental near-simultaneous mining
- natural topology delay
- deliberate withholding
- deliberate double-spend attempt

### Confirmation Count는 Finality가 아니다

confirmation은 replacement cost를 늘릴 뿐 absolute irreversibility를 만들지는 않는다. reorg는 이를 직접 보여 준다. previously confirmed transaction도 higher-work valid branch가 이를 대체하면 active history에서 사라질 수 있다.

---

## 10. Security Assumptions and Failure Modes

### Natural Reorg

one-block 또는 다른 shallow reorg는 adversarial coordination 없이도 일어날 수 있다. block discovery timing이 가깝고 propagation이 완벽하지 않으면 충분하다.

### Adversarial Reorg

더 깊거나 전략적인 timing을 가진 reorg는 다음에 대한 더 강한 우려를 뒷받침할 수 있다.

- double-spend attempt
- selfish-mining style withholding
- majority-hashrate attack
- eclipse-assisted victim-specific misperception

하지만 public reorg만으로 어떤 motive가 적용되었는지를 보통 증명할 수는 없다.

### SPV 노출

operating-mode 문서는 SPV client가 모든 block을 full validation하지 않고 header와 cumulative work에 의존한다고 설명한다.[^ref-btc-dev-operating-modes]

즉, SPV client는 reorg risk를 판단할 때 confirmation depth와 honest majority assumption에 특히 더 의존한다.

### Infrastructure Risk

reorg는 다음을 압박한다.

- exchange crediting logic
- deposit finality assumption
- wallet transaction-state tracking
- monotonic confirmation growth를 가정한 accounting system
- active-chain change를 명시적으로 모델링하지 않는 analytics pipeline

---

## 11. Mathematical or Economic Model

### Replacement-Cost Intuition

transaction이 `k` block 깊이에 있다면, 이를 뒤집으려면 그 transaction block 이전 fork point부터 public branch를 cumulative work로 따라잡고 overtaking하는 competing branch가 필요하다. depth는 이를 거칠게 나타내는 proxy이고, cumulative work는 더 나은 proxy다.

### Simple Confirmation Model

다음을 두자.

- `W_pub` = target block 이후 public branch에 더해진 cumulative work
- `W_alt` = 같은 fork point 이후 competing branch에 더해진 cumulative work

alternative branch로의 reorg는 다음일 때 가능해진다.

```text
W_alt > W_pub
```

단, 두 branch 모두 node의 consensus rule 아래에서 valid해야 한다.

### Economic Meaning

reorg는 경제적으로 중요하다. 왜냐하면:

- settlement assumption을 뒤집을 수 있고
- prior input의 spendability를 복원할 수 있고
- miner에 대한 fee attribution을 바꿀 수 있고
- losing branch의 realized block reward를 무효화할 수 있고
- exchange와 custodian이 crediting을 늦추게 만들 수 있기 때문이다

---

## 12. Bitcoin Core 구현

### `chain.h`

`chain.h`는 block-index structure와 `LastCommonAncestor`를 제공한다. 이는 reorg가 어디서 시작되는지 식별하는 structural primitive다.[^ref-btc-core-chain]

### `validation`

Bitcoin Core의 validation layer에는 `ActivateBestChain`, `ActivateBestChainStep`, `DisconnectBlock`, `ConnectBlock`, `MaybeUpdateMempoolForReorg`가 있다. 이름 그대로 reorg의 핵심 실행 단계를 보여 준다: best valid branch 선택, old state disconnect, new state connect, mempool reconcile.[^ref-btc-core-validation]

### `txmempool`

`CTxMemPool` 문서는 new block connect 이후나 reorg 이후 mempool을 best chain과 일치하도록 갱신하는 기능을 명시하고, chain change 영향을 받은 transaction을 복구하는 데 유용한 구조를 유지한다고 설명한다.[^ref-btc-core-txmempool]

### `validationinterface`

`CValidationInterface`는 `UpdatedBlockTip`, `BlockConnected`, `BlockDisconnected`, mempool-related notification 같은 callback을 노출한다. 이를 통해 reorg state transition이 wallet, index, external integration 같은 의존 subsystem에 전달된다.[^ref-btc-core-validationinterface]

---

## 13. Consensus, Policy, and Presentation

### Consensus Layer

consensus는 replacement branch가 valid한지, 그리고 더 큰 cumulative work를 가지는지를 결정한다.

### Policy Layer

policy는 disconnected transaction이 mempool에 재진입하거나 다시 relay될 수 있는지를 결정한다. consensus-valid transaction도 local mempool policy나 conflict 때문에 재진입하지 못할 수 있다.

### Presentation Layer

presentation은 사용자와 system이 관측하는 상태다.

- transaction이 confirmed였다가 unconfirmed로 바뀔 수 있다.
- deposit이 credit되었다가 reversal될 수 있다.
- block explorer가 branch replacement 뒤에 block을 stale/orphaned로 재분류할 수 있다.

이러한 presentation change는 active-chain switch의 downstream effect다.

---

## 14. 온체인 함의

### Public Data가 보여 줄 수 있는 것

두 branch가 public하게 알려지면 분석가는 다음을 재구성할 수 있다.

- fork point
- losing-branch depth
- winning-branch depth
- 온체인에서 보이는 conflicting transaction
- replaced confirmation

### Public Data가 보통 보여 주지 못하는 것

public chain data만으로는 보통 다음을 증명하지 못한다.

- release 전 private fork의 존재 여부
- 공격자의 intent
- event가 accidental이었는지 여부
- 각 시점에 정확히 어떤 node가 이미 view를 바꿨는지

### Reorg Evidence Requirements

강한 reorg analysis라면 최소한 다음을 보고해야 한다.

- old tip과 new tip
- fork point height
- block 기준 reorg depth
- 가능한 경우 work difference
- conflicting spend의 존재 여부
- event가 처음부터 public했는지, release 후에야 visible해졌는지

---

## 15. Institutional Thinking

기관은 reorg handling을 단순 security headline이 아니라 state-management 문제로 다뤄야 한다.

### Practical Implications

- confirmation threshold는 미신이 아니라 reorg tolerance를 반영해야 한다.
- accounting system은 reversible settlement state를 가져야 한다.
- mempool/chain observer는 first-seen event뿐 아니라 active-chain transition도 기록해야 한다.
- incident response는 "public reorg observed"와 "malicious intent established"를 구분해야 한다.

### Better Reporting

고품질 내부 보고에는 다음 두 가지가 모두 필요하다.

- block 기준 reorg depth
- 재구성 가능할 경우 replacement work differential

depth만으로는 serious risk comparison에 부족하다.

---

## 16. Common Misinterpretations

### "Reorg는 Header만 바꾼다"

틀렸다. active chainstate를 바꾸므로 UTXO state, confirmation, 때로는 mempool content도 바뀐다.

### "한 번 Confirmed된 tx는 Final이다"

틀렸다. confirmation은 replacement cost가 커진다는 증거일 뿐 absolute permanence는 아니다.

### "모든 Reorg는 Attack이다"

틀렸다. 작은 reorg는 propagation race로도 자연스럽게 생긴다.

### "대안 Branch에 Header가 더 많으면 Reorg가 반드시 일어난다"

틀렸다. branch는 valid해야 하고, visible header count가 아니라 cumulative work에서 이겨야 한다.

### "Disconnected Transaction은 항상 Mempool로 돌아간다"

틀렸다. new active chain 아래에서 여전히 valid하고 policy-acceptable할 때만 돌아간다.

---

## 17. Research Questions

1. 현재 propagation condition에서 Bitcoin mainnet의 shallow reorg는 얼마나 자주 발생하는가?
2. reorg risk dashboard에서 raw depth 대비 work differential은 얼마나 설명력을 더해 주는가?
3. public observer는 accidental reorg와 strategic reorg를 얼마나 신뢰성 있게 구분할 수 있는가?
4. 어떤 mempool dataset이 reorg-aware transaction history를 가장 잘 보존하는가?
5. 기관 finality model은 topology-specific observation lag를 어떻게 반영해야 하는가?

---

## 18. Practical Exercises

### Exercise 1

chain data에서 public reorg 하나를 재구성하고 fork point, losing branch, winning branch, depth를 식별하라.

### Exercise 2

reorg로 제거된 transaction에 대해 다음 중 무엇이 일어났는지 분류하라.

- mempool 재진입
- conflicting spend로 대체
- new chain 아래 invalid가 되어 소멸

### Exercise 3

cumulative work data가 있을 경우, 같은 depth지만 replacement-cost profile이 다른 두 reorg event를 비교하라.

### Exercise 4

candidate branch 발견부터 `ActivateBestChain`과 post-reorg mempool reconciliation까지의 high-level Core flow를 추적하라.

---

## 19. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Reorg as branch replacement after fork competition | Directly specified | Whitepaper and Developer Guide |
| Fork-point and best-chain mechanics | Directly specified | Bitcoin Core `chain` and `validation` references |
| Mempool consistency and disconnected-tx handling | Directly specified | `txmempool` and validation references |
| Analyst interpretation of intent | Inference from sources | Derived from observable protocol behavior, not explicit proof |
| Work-aware risk framing | Analytical model | Derived from chainwork and confirmation semantics |

---

## 20. Knowledge Graph

```text
Chain Reorganization
├─ Preconditions
│  ├─ fork point
│  ├─ competing branch
│  ├─ validity
│  └─ greater cumulative work
├─ Mechanics
│  ├─ last common ancestor
│  ├─ disconnect old blocks
│  ├─ connect new blocks
│  ├─ update tip
│  └─ reconcile mempool
├─ State Effects
│  ├─ UTXO rollback
│  ├─ confirmation reversal
│  ├─ stale blocks
│  └─ tx reinsertion or conflict
├─ Observation
│  ├─ public reorg
│  ├─ private-fork uncertainty
│  └─ on-chain evidence limits
└─ Risk
   ├─ settlement reversal
   ├─ double-spend exposure
   ├─ accounting inconsistency
   └─ SPV sensitivity
```

---

## 21. 참고문헌

### Primary Sources

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Sections 5 and 11. https://bitcoin.org/bitcoin.pdf
[^ref-btc-dev-blockchain]: Bitcoin Developer Guide, "Block Chain," including most-difficult-chain and fork behavior. https://developer.bitcoin.org/devguide/block_chain.html
[^ref-btc-dev-operating-modes]: Bitcoin Developer Guide, "Operating Modes," including full-node and SPV chain-validation discussion. https://developer.bitcoin.org/devguide/operating_modes.html
[^ref-btc-core-chain]: Bitcoin Core Doxygen, `chain.h`, including `CBlockIndex`, block proof helpers, and `LastCommonAncestor`. https://doxygen.bitcoincore.org/chain_8h_source.html
[^ref-btc-core-validation]: Bitcoin Core Doxygen, `validation.h`, including `ActivateBestChain`, `ActivateBestChainStep`, `DisconnectBlock`, `ConnectBlock`, and `MaybeUpdateMempoolForReorg`. https://doxygen.bitcoincore.org/validation_8h_source.html
[^ref-btc-core-txmempool]: Bitcoin Core Doxygen, `CTxMemPool` / `txmempool.h`, including mempool consistency after a new block or reorg. https://doxygen.bitcoincore.org/class_c_tx_mem_pool.html
[^ref-btc-core-validationinterface]: Bitcoin Core Doxygen, `validationinterface.h` and `CValidationInterface`, including `UpdatedBlockTip`, `BlockConnected`, and `BlockDisconnected`. https://doxygen.bitcoincore.org/validationinterface_8h_source.html
[^ref-btc-core-consensus-validation]: Bitcoin Core Doxygen, `consensus/validation.h`, including validation-state distinctions for transactions and blocks. https://doxygen.bitcoincore.org/consensus_2validation_8h.html

### Supporting Interpretation Notes

- Where this document discusses intent, institutional reporting standards, or work-aware risk framing, those statements are inferences from Bitcoin's chain-selection and validation architecture rather than explicit protocol claims.

---

## 22. 교차 참조

### Previous

- BITCOIN-023 — Forks

### Next

- BITCOIN-025 — Bitcoin Monetary Policy

### Related

- BITCOIN-007 — Whitepaper Section 6 — Incentive
- BITCOIN-014 — UTXO Model
- BITCOIN-017 — Mempool
- BITCOIN-021 — Blocks and Block Headers
- BITCOIN-022 — Nodes and Network Propagation
- BITCOIN-023 — Forks
- POW-010 — Longest Chain Rule and Fork Resolution
- POW-011 — Cumulative Chainwork
- POW-013 — Bitcoin Core Proof-of-Work Validation
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- fork, branch replacement, stale block, confirmation reversal을 분리했다.
- UTXO rollback, disconnected transaction handling, mempool reconciliation을 포함했다.
- active-chain replacement를 raw block count가 아니라 valid-higher-work 기준으로 설명했다.
- Core reference는 reorg에 직접 관련된 chain, validation, mempool, validation-interface surface로 제한했다.

### Evidence Review

Passed.

- 백서와 Developer Guide가 branch-replacement model을 뒷받침한다.
- Core 구현 설명은 `chain.h`, `validation.h`, `txmempool.h`, `validationinterface.h`에 연결했다.
- mempool과 chainstate side effect는 명시적 Core 문서에 연결했다.
- attacker intent 관련 해석은 inference로 표시했다.

### Editorial Review

Passed.

- 문서 구조는 프로젝트 deep-dive format을 따른다.
- metadata가 완전하다.
- fork point, active chain, reorg depth, cumulative work, disconnected block, replacement branch 용어가 일관된다.
- table과 code fence가 닫혀 있다.

### Adversarial Review

Passed.

- 모든 reorg를 malicious하다고 주장하지 않았다.
- reorg decision을 header count만으로 환원하지 않았다.
- disconnected transaction이 항상 mempool로 돌아간다고 하지 않았다.
- headers-only awareness가 branch activation을 보장한다고 하지 않았다.
- public observability를 private-fork absence proof로 혼동하지 않았다.

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
