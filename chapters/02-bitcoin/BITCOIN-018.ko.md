---
knowledge_id: BITCOIN-018
title: Transaction Fees
subtitle: 수수료 계산, fee rate, weight, virtual size, relay policy, fee estimation, RBF, CPFP, 그리고 채굴자 인센티브
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 105 min
estimated_study: 300 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Transaction Fees
  - Fee Market
  - Mempool Policy
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-014
  - BITCOIN-015
  - BITCOIN-017
related_topics:
  - UTXO Model
  - Transaction Validation
  - Mempool
  - Block Space
  - RBF
  - CPFP
  - Fee Estimation
  - Miner Revenue
primary_sources:
  - REF-BTC-WP-001
  - REF-BIP-0125
  - REF-BIP-0141
  - REF-BTC-CORE-29-RELEASE-001
  - REF-BTC-CORE-31-RELEASE-001
  - REF-BTC-CORE-TX-VERIFY-001
  - REF-BTC-CORE-CONSENSUS-VALIDATION-001
  - REF-BTC-CORE-FEERATE-001
  - REF-BTC-CORE-FEE-ESTIMATOR-001
  - REF-BTC-CORE-POLICY-001
  - REF-BTC-CORE-RBF-001
  - REF-BTC-CORE-RPC-FEE-001
tags:
  - bitcoin
  - internals
  - transaction-fees
  - feerate
  - weight
  - vsize
  - mempool
  - rbf
  - cpfp
  - fee-estimation
---

# Transaction Fees
> Bitcoin Internals  
> Research Unit: BITCOIN-018

---

## Research Brief

```yaml
knowledge_id: BITCOIN-018
title: Transaction Fees
research_question: >
  How are Bitcoin transaction fees computed, how do weight and virtual size
  convert fees into fee rates, how do mempool and mining policies use fee
  rates, and how should institutions reason about fee estimation, RBF, CPFP,
  and miner incentives without confusing policy with consensus?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-014
  - BITCOIN-015
  - BITCOIN-017
parent: Bitcoin Internals
previous: BITCOIN-017
next: BITCOIN-019
related_topics:
  - UTXO Model
  - Transactions
  - Mempool
  - Block Template Construction
  - Mining Incentives
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
  - Full wallet coin-selection algorithm design
  - Mining pool private fee-market agreements
  - Detailed Lightning fee policy
  - Non-Bitcoin fee markets
  - Real-time fee recommendation for a live payment
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Bitcoin transaction fee가 직렬화된 명시 필드가 아니라는 점을 설명할 수 있다.
- 입력 값과 출력 값으로부터 수수료를 계산할 수 있다.
- sat/vB와 BTC/kvB 문맥에서 fee rate를 정의할 수 있다.
- SegWit의 weight와 virtual size 개념을 설명할 수 있다.
- consensus validity와 fee 기반 relay/mining policy를 구분할 수 있다.
- RBF와 CPFP가 미확정 transaction의 유인을 어떻게 바꾸는지 설명할 수 있다.
- Bitcoin Core fee estimation이 관측된 확인 데이터를 어떻게 활용하는지 설명할 수 있다.
- fee 관련 Bitcoin Core 소스 영역을 식별할 수 있다.
- fee estimation이 확률적이고 로컬하다는 점을 설명할 수 있다.

---

## 2. 핵심 질문

1. Bitcoin 수수료는 transaction 어디에 기록되는가?
2. fee는 입력/출력 값으로 어떻게 계산되는가?
3. fee rate는 왜 절대 수수료보다 중요한가?
4. SegWit의 weight와 vsize는 block-space 비용을 어떻게 바꾸는가?
5. minimum relay fee는 consensus rule인가 policy인가?
6. RBF와 CPFP는 fee management를 어떻게 가능하게 하는가?
7. fee estimate는 왜 보장값이 아니라 확률적 신호인가?
8. 기관은 fee를 비용, 위험, 운영 통제 관점에서 어떻게 다뤄야 하는가?

---

## 3. Executive Summary

Bitcoin transaction fee는 별도의 직렬화 필드가 아니다. 코인베이스가 아닌 transaction에 대해 fee는 입력으로 소비된 UTXO 값의 합에서 출력 값의 합을 뺀 값으로 계산된다.[^ref-btc-core-tx-verify]

```text
fee = sum(inputs) - sum(outputs)
```

블록 공간은 scarce resource이므로 채굴자와 노드는 절대 수수료보다 대개 fee rate를 더 중시한다. SegWit 이후에는 byte 대신 weight와 virtual size 기준이 중요해졌다.[^ref-bip-0141]

```text
weight = base_size * 3 + total_size
vsize = ceil(weight / 4)
fee_rate = fee / vsize
```

수수료 관련 규칙은 세 계층으로 나눠 봐야 한다.

- consensus: value creation 방지, coinbase가 subsidy + collected fees를 초과할 수 없음
- policy: minimum relay fee, dust, RBF replacement 조건, package acceptance 기준
- wallet behavior: fee estimation, coin selection, batching, consolidation, RBF/CPFP 운용

Bitcoin Core 29.1은 기본 `-minrelaytxfee`와 `-incrementalrelayfee`를 100 sat/kvB로 낮췄고, 31.0은 fee estimator minimum bucket을 0.1 sat/vB로 조정했다.[^ref-btc-core-29-release] [^ref-btc-core-31-release]

분석 관점에서 fee는 block-space 수요와 transaction urgency의 단서를 제공하지만, intent를 직접 증명하지는 못한다. 높은 수수료는 혼잡, 잘못된 추정, 많은 입력 수, RBF, CPFP, batching, consolidation 등 여러 원인으로 나타날 수 있다.

---

## 4. 프로토콜 구조

### 수수료의 위치

Bitcoin transaction 구조에는 "fee"라는 직렬화 필드가 없다. 이는 이전 출력 값이 참조 시점에 외부 UTXO 집합에서 찾아져야만 계산되는 파생 값이다.

### 채굴자 인센티브

채굴자는 코인베이스 transaction에서 블록 보조금과 블록에 포함된 non-coinbase transaction의 총 수수료를 가져간다.[^ref-btc-wp]

```text
block_fees = sum(non_coinbase_tx_fees)
miner_revenue = subsidy + block_fees
```

### fee rate 중심 경쟁

블록 공간이 제한되어 있으므로, 어떤 transaction이 candidate block에 포함될 가능성은 대개 절대 fee보다 fee rate와 package context에 더 크게 좌우된다.

---

## 5. Technical Mechanics

### 입력과 출력 값

fee를 계산하려면 각 입력이 참조하는 이전 output의 value를 알아야 한다. raw transaction만 보고는 정확한 fee를 확정할 수 없다. 이전 UTXO 조회가 필요하다.

### `CheckTxInputs`

Bitcoin Core의 `Consensus::CheckTxInputs`는 입력 값 합과 출력 값 합을 바탕으로 수수료를 계산하고, unauthorized value creation이 없는지 검증한다.[^ref-btc-core-tx-verify]

### weight와 vsize

BIP141는 witness discount를 반영한 weight 규칙을 정의하고, Bitcoin Core는 `GetTransactionWeight`로 이를 구현한다.[^ref-bip-0141] [^ref-btc-core-consensus-validation]

```text
weight = base_size * 3 + total_size
vsize = ceil(weight / 4)
```

여기서:

- `base_size`: witness를 제외한 serialization 크기
- `total_size`: witness를 포함한 전체 크기

### fee rate 단위

실무에서는 sat/vB가 가장 널리 쓰인다. 다만 Bitcoin Core 문서와 RPC 일부는 BTC/kvB 또는 sat/kvB 문맥도 사용한다. 단위 전환을 정확히 이해해야 한다.

```text
1 sat/vB = 1,000 sat/kvB
100 sat/kvB = 0.1 sat/vB
```

### RBF

RBF는 더 높은 fee로 기존 미확정 transaction을 교체하는 메커니즘이다. 이는 unconfirmed state에서 fee management를 가능하게 한다.[^ref-bip-0125] [^ref-btc-core-rbf]

### CPFP

CPFP는 낮은 fee의 parent를 높은 fee의 child가 보완하는 구조다. 이 경우 개별 transaction보다는 combined package 경제성이 중요해진다.

### dust와 minimum relay

dust threshold와 minimum relay fee는 policy다. 이는 기본 노드의 relay/acceptance 행동을 바꾸지만 consensus-validity를 직접 결정하지 않는다.[^ref-btc-core-policy]

---

## 6. Mathematical or Economic Model

### 기본 공식

```text
fee = sum(inputs) - sum(outputs)
fee_rate = fee / vsize
```

### package 공식

```text
package_fee = sum(transaction_fees)
package_vsize = sum(transaction_vsizes)
package_feerate = package_fee / package_vsize
```

### block-level 경제성

```text
block_fees = sum(fees of non-coinbase transactions)
miner_revenue = block_subsidy + block_fees
fee_share = block_fees / miner_revenue
```

이 지표는 특정 시점의 fee pressure와 security budget 구성을 분석하는 데 유용하다. 다만 단일 블록에서 장기 경제학을 과도하게 추론하면 안 된다.

---

## 7. Security Assumptions

### policy와 consensus의 분리

수수료와 관련한 많은 규칙은 policy다. 분석가가 minimum relay fee를 consensus minimum fee로 오인하면 잘못된 결론을 내리게 된다.

### estimator 불확실성

fee estimate는 관측된 과거 확인 데이터를 기반으로 한 추정이다. 갑작스러운 혼잡, miner policy 변화, package behavior로 인해 쉽게 빗나갈 수 있다.[^ref-btc-core-fee-estimator] [^ref-btc-core-rpc-fee]

### intent over-inference

높은 fee가 곧 긴급 자금 이동을 뜻하는 것은 아니다. 낮은 fee가 곧 중요하지 않은 거래를 뜻하는 것도 아니다.

---

## 8. Bitcoin Core 구현

### 핵심 소스 영역

| Area | Role |
|---|---|
| `consensus/tx_verify.*` | 입력/출력 값과 fee 계산 |
| `consensus/validation.h` | `GetTransactionWeight`, `GetBlockWeight` |
| `policy/feerate.*` | `CFeeRate` 표현 |
| `policy/fees/block_policy_estimator.*` | fee estimator |
| `policy/policy.*` | relay/mining fee defaults, dust |
| `policy/rbf.cpp` | replacement fee checks |

### `CFeeRate`

`CFeeRate`는 fee-per-kvB 계열 표현을 담당한다. 실무적 해석에서는 sat/vB로 자주 환산해 사용한다.[^ref-btc-core-feerate]

### fee estimator

`CBlockPolicyEstimator`는 confirmation target별로 관측 결과를 bucketized 하여 확률적 추정을 만든다. 이는 deterministic guarantee가 아니다.[^ref-btc-core-fee-estimator]

### RPC

`estimatesmartfee`는 target 기반 추정값을 제공한다. 출력 단위와 반환 불확실성을 문서와 함께 해석해야 한다.[^ref-btc-core-rpc-fee]

---

## 9. Consensus, Policy, and Wallet Behavior

### Consensus

consensus 차원의 fee 관련 규칙은 대체로 다음과 같다.

- 일반 transaction은 가치를 생성할 수 없다.
- coinbase output은 subsidy + included fees를 초과할 수 없다.
- block weight 한도가 전체 포함 데이터를 제한한다.

모든 경우에 positive fee가 필수인 것은 아니다. zero-fee 또는 very-low-fee transaction도 다른 규칙을 만족하면 consensus-valid일 수 있다.

### Policy

policy 차원의 fee 관련 규칙은 다음을 포함한다.

- minimum relay fee
- incremental relay fee
- mempool minimum fee
- dust threshold
- RBF replacement fee requirements
- package acceptance fee requirements
- mining template minimum fee

### Wallet Behavior

wallet behavior는 다음을 포함한다.

- fee estimation
- target confirmation selection
- coin selection
- change output 관리
- RBF opt-in 여부
- CPFP 구성
- batching / consolidation timing

wallet 실수나 운영자 설정은 transaction이 consensus-valid임에도 과도한 수수료 또는 부족한 수수료를 초래할 수 있다.

---

## 10. On-Chain Implications

### Strong Evidence

확정된 transaction 데이터는 다음에 강한 근거를 제공한다.

- 이전 출력 값이 알려질 때 절대 fee
- vsize가 계산될 때 fee rate
- SegWit witness discount가 size accounting에 영향을 주는지
- 입력 수가 많은 transaction인지
- 높은 fee 시기에 확인되었는지
- 블록 단위 miner fee revenue

### Weak Evidence

fee 데이터는 다음 주장에는 약한 근거만 제공한다.

- payer urgency
- wallet sophistication
- exchange withdrawal policy
- emergency movement
- liquidation behavior
- deliberate overpayment
- miner preference beyond observed inclusion

### 다양한 원인

높은 fee는 혼잡, 잘못된 estimation, 많은 입력 수, RBF replacement, CPFP rescue, policy constraints로 나타날 수 있다.

---

## 11. Institutional Thinking

### Treasury Operations

기관은 다음 방식으로 수수료를 관리할 수 있다.

- 가능할 때 withdrawal batching
- 낮은 fee 시기에 UTXO consolidation
- 통제된 fee bumping을 위한 RBF 활용
- stuck dependency 구제를 위한 CPFP 활용
- isolated tx fee rate만이 아니라 package context 모니터링
- urgent settlement와 normal settlement 기준 분리

### Custody and Signing Controls

서명 시스템은 최소한 다음을 명확히 표시해야 한다.

- absolute fee
- fee rate
- input count와 total input value
- output amounts
- change output
- relevant package context
- RBF signaling status

### Accounting

회계 시스템은 다음을 구분해야 한다.

- 수취인에게 보낸 금액
- 기관으로 돌아오는 change
- network fee
- 블록에서 인식된 miner revenue
- pending fee estimate와 confirmed paid fee

### Risk

낮은 fee의 미확정 입금은 지연되거나 교체될 수 있다. 높은 fee의 출금도 갑작스러운 혼잡이나 낮은 fee의 미확정 부모에 의존하면 빨리 확인되지 않을 수 있다.

---

## 12. Common Misinterpretations

### "수수료는 transaction 필드에 적혀 있다"

아니다. fee는 입력 값과 출력 값 차이로 계산된다.

### "절대 수수료가 높으면 항상 더 빨리 확인된다"

아니다. fee rate와 package context가 더 중요하다.

### "consensus는 minimum fee를 요구한다"

아니다. minimum fee는 대체로 relay/mining policy다.

### "fee estimate는 보장이다"

아니다. 확률적 추정일 뿐이다.

### "SegWit는 witness를 공짜로 만든다"

아니다. witness data는 할인될 뿐 free가 아니다.

### "높은 fee는 곧 긴급 송금이다"

반드시 그렇지 않다.

---

## 13. Research Questions

1. 지리적으로 분산된 public node들 사이에서 fee estimate 차이는 얼마나 큰가?
2. wallet fee estimate는 next-block clearing rate 대비 얼마나 자주 과지불하는가?
3. cluster mempool은 CPFP와 RBF 분석을 어떻게 바꾸는가?
4. 거래소 출금 중 RBF signaling을 사용하는 비중은 얼마인가?
5. 서로 다른 시장 국면에서 miner revenue 중 fee 비중은 어떻게 달라지는가?
6. batching과 consolidation은 장기 기관 비용을 얼마나 낮추는가?
7. urgent movement와 large-input-count fee inflation을 구분하는 증거는 무엇인가?

---

## 14. Practical Exercises

### Exercise 1: transaction fee 계산

확정된 non-coinbase transaction 하나를 선택하라.

1. 모든 input outpoint를 나열한다.
2. 각 previous output value를 조회한다.
3. 입력 값 합을 구한다.
4. 출력 값 합을 구한다.
5. fee를 계산한다.
6. virtual size로 fee rate를 계산한다.

### Exercise 2: absolute fee와 fee rate 비교

두 transaction을 찾는다.

- absolute fee는 높지만 vsize도 큰 거래
- absolute fee는 더 낮지만 sat/vB는 더 높은 거래

어느 쪽이 block-space 단위 기준으로 더 매력적인지 설명하라.

### Exercise 3: RBF fee bump

미확정 replacement pair 또는 과거 사례를 하나 찾는다. 다음을 기록하라.

- original fee
- original vsize
- replacement fee
- replacement vsize
- absolute fee increase
- fee-rate increase
- replacement policy relevance

### Exercise 4: CPFP package

미확정 부모 출력을 소비하는 parent-child pair를 찾아 다음을 계산하라.

```text
parent_feerate
child_feerate
combined_package_feerate
```

왜 child가 parent의 confirmation incentive를 바꿀 수 있는지 설명하라.

---

## 15. Evidence Classification

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Fees as miner incentive after issuance declines | A |
| REF-BIP-0125 | Policy BIP | Opt-in RBF signaling and replacement fee rules | A |
| REF-BIP-0141 | Consensus BIP | Transaction weight and virtual size definitions | A |
| REF-BTC-CORE-29-RELEASE-001 | Release documentation | Default relay fee and incremental relay fee change to 100 sat/kvB | A |
| REF-BTC-CORE-31-RELEASE-001 | Release documentation | Fee estimator bucket update, package relay note, static wallet fee removal | A |
| REF-BTC-CORE-TX-VERIFY-001 | Primary implementation source | `CheckTxInputs` fee calculation | A |
| REF-BTC-CORE-CONSENSUS-VALIDATION-001 | Primary implementation source | `GetTransactionWeight` and weight formula | A |
| REF-BTC-CORE-FEERATE-001 | Primary implementation source | `CFeeRate` fee-rate representation | A |
| REF-BTC-CORE-FEE-ESTIMATOR-001 | Primary implementation source | `CBlockPolicyEstimator` algorithm | A |
| REF-BTC-CORE-POLICY-001 | Primary implementation source | Relay and mining policy fee defaults | A |
| REF-BTC-CORE-RBF-001 | Primary implementation source | RBF replacement policy checks | A |
| REF-BTC-CORE-RPC-FEE-001 | RPC documentation | `estimatesmartfee` behavior and units | B |

### Claim Ledger

| Claim | Classification | Evidence |
|---|---|---|
| fee는 input value minus output value로 계산된다 | FACT | Bitcoin Core `CheckTxInputs` |
| fee는 직렬화된 transaction field가 아니다 | FACT | Transaction structure plus `CheckTxInputs` fee computation |
| BIP141는 transaction weight와 virtual size를 정의한다 | FACT | BIP141 |
| Bitcoin Core는 witness/non-witness serialization을 바탕으로 transaction weight를 계산한다 | FACT | Bitcoin Core `consensus/validation.h` |
| minimum relay fee는 policy이지 consensus가 아니다 | FACT | Bitcoin Core release and policy docs |
| Bitcoin Core 29.1은 기본 relay/incremental relay feerate를 100 sat/kvB로 낮췄다 | FACT | Bitcoin Core 29.1 release notes |
| Bitcoin Core 31.0은 fee estimator minimum bucket을 0.1 sat/vB로 조정했다 | FACT | Bitcoin Core 31.0 release notes |
| `estimatesmartfee`는 확률적이며 관측 데이터 기반이다 | FACT | RPC docs and estimator source |
| CPFP는 combined package feerate를 통해 유인을 바꾼다 | INTERPRETATION | Fee-rate model and package relay docs |
| 높은 fee는 긴급 의도를 증명한다 | COUNTERCLAIM | Rejected; multiple causes can produce high fees |
| fee estimate는 confirmation을 보장한다 | COUNTERCLAIM | Rejected; estimates are probabilistic |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | 직접적인 1차 자료로 뒷받침됨 |
| INTERPRETATION | 사실을 기반으로 한 분석적 종합 |
| POLICY | relay, mempool, wallet, mining 관행 |
| HEURISTIC | 실무적 규칙이지만 반례가 있음 |
| UNKNOWN | 근거가 부족함 |

---

## 16. Knowledge Graph

```text
BITCOIN-018 Transaction Fees
|
+-- builds_on: BITCOIN-014 UTXO Model
+-- builds_on: BITCOIN-015 Transactions in Depth
+-- builds_on: BITCOIN-017 Mempool
|
+-- fee
|   +-- computed_as: inputs - outputs
|   +-- collected_by: coinbase
|
+-- size accounting
|   +-- weight: base_size * 3 + total_size
|   +-- vsize: ceil(weight / 4)
|
+-- fee rate
|   +-- unit: sat/vB
|   +-- used_by: relay policy, mining policy, fee estimation
|
+-- fee bumping
|   +-- RBF: replacement transaction
|   +-- CPFP: high-fee child package
|
+-- Bitcoin Core
|   +-- CheckTxInputs
|   +-- GetTransactionWeight
|   +-- CFeeRate
|   +-- CBlockPolicyEstimator
|   +-- estimatesmartfee
|
+-- analysis
    +-- facts: paid fee, vsize, fee rate
    +-- caveats: intent, urgency, miner preference
```

---

## 17. 참고문헌

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Section 6, incentive and transaction fee discussion, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.
[^ref-bip-0125]: David A. Harding and Peter Todd, "BIP 125: Opt-in Full Replace-by-Fee Signaling," 2015-12-04, https://bips.dev/125/, accessed 2026-08-04.
[^ref-bip-0141]: Eric Lombrozo, Johnson Lau, and Pieter Wuille, "BIP 141: Segregated Witness (Consensus layer)," transaction size calculations, 2015-12-21, https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki, accessed 2026-08-04.
[^ref-btc-core-29-release]: Bitcoin Core Contributors, "Bitcoin Core 29.1 Release Notes," mempool policy changes for default `-minrelaytxfee` and `-incrementalrelayfee`, https://bitcoincore.org/en/releases/29.1/, accessed 2026-08-04.
[^ref-btc-core-31-release]: Bitcoin Core Contributors, "Bitcoin Core 31.0 Release Notes," package relay, fee estimator, wallet fee-setting, and cluster mempool notes, https://bitcoin.org/en/releases/31.0/, accessed 2026-08-04.
[^ref-btc-core-tx-verify]: Bitcoin Core Contributors, `src/consensus/tx_verify.h` and `src/consensus/tx_verify.cpp`, `Consensus::CheckTxInputs`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/tx__verify_8h_source.html, accessed 2026-08-04.
[^ref-btc-core-consensus-validation]: Bitcoin Core Contributors, `src/consensus/validation.h`, `GetTransactionWeight`, `GetBlockWeight`, and transaction input weight calculation, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/consensus_2validation_8h_source.html, accessed 2026-08-04.
[^ref-btc-core-feerate]: Bitcoin Core Contributors, `src/policy/feerate.h` and `src/policy/feerate.cpp`, `CFeeRate`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/feerate_8h_source.html, accessed 2026-08-04.
[^ref-btc-core-fee-estimator]: Bitcoin Core Contributors, `src/policy/fees/block_policy_estimator.h` and `src/policy/fees/block_policy_estimator.cpp`, `CBlockPolicyEstimator`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/class_c_block_policy_estimator.html and https://doxygen.bitcoincore.org/block__policy__estimator_8h_source.html, accessed 2026-08-04.
[^ref-btc-core-policy]: Bitcoin Core Contributors, `src/policy/policy.h` and `src/policy/policy.cpp`, relay fee, dust, standardness, and mining policy defaults, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/policy_8h.html, accessed 2026-08-04.
[^ref-btc-core-rbf]: Bitcoin Core Contributors, `src/policy/rbf.cpp`, `IsRBFOptIn` and replacement fee checks, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/policy_2rbf_8cpp_source.html, accessed 2026-08-04.
[^ref-btc-core-rpc-fee]: Bitcoin Core RPC documentation, "`estimatesmartfee` RPC," fee estimate behavior, confirmation target, virtual size note, and BTC/kvB output, https://bitcoincore.org/en/doc/26.0.0/rpc/util/estimatesmartfee/, accessed 2026-08-04.

---

## 18. 교차 참조

### Parent

- Bitcoin Internals

### Previous

- BITCOIN-017 — Mempool

### Next

- BITCOIN-019 — Wallets and Key Management

### Related

- BITCOIN-007 — Whitepaper Section 6 — Incentive
- BITCOIN-014 — UTXO Model
- BITCOIN-015 — Transactions in Depth
- BITCOIN-017 — Mempool
- BITCOIN-019 — Wallets and Key Management
- POW-009 — Coinbase Transaction and Mining Commitments
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- fee 계산을 UTXO value와 `CheckTxInputs`에 연결했다.
- weight와 vsize 공식을 BIP141와 Core `GetTransactionWeight` 기준으로 정리했다.
- consensus, relay policy, mining policy, wallet behavior를 분리했다.

### Evidence Review

Passed.

- fee formula는 Core validation source에 연결했다.
- weight/vsize는 BIP141와 Core consensus validation source에 연결했다.
- relay policy와 estimator 관련 현재 동작은 29.1/31.0 release notes에 연결했다.

### Editorial Review

Passed.

- deep-dive 구조를 유지했다.
- 용어는 fee, fee rate, weight, vsize, relay policy, mining policy, RBF, CPFP로 일관화했다.

### Adversarial Review

Passed.

- fee가 직렬화 필드라고 주장하지 않았다.
- fee estimate를 보장값으로 취급하지 않았다.
- minimum relay fee와 consensus validity를 혼동하지 않았다.
- 높은 fee만으로 사용자 의도를 추정하지 않았다.

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
