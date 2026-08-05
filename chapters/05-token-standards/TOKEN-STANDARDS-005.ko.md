---
knowledge_id: TOKEN-STANDARDS-005
title: Wrapped Assets
subtitle: 네이티브 자산의 토큰화, 브리지 표현, 그리고 composable interface 뒤에 숨은 custody 가정
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 95 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - Token Standards
  - Ethereum
  - Interoperability
parent:
  - Token Standards
prerequisites:
  - TOKEN-STANDARDS-001
  - TOKEN-STANDARDS-004
related_topics:
  - Stablecoins
  - Bridges
  - DeFi
primary_sources:
  - REF-ETH-WETH-2026-001
  - REF-CIRCLE-XRESERVE-2026-001
  - REF-CIRCLE-USDC-2026-001
tags:
  - wrapped-assets
  - weth
  - bridges
---

# Wrapped Assets
> Token Standards  
> Research Unit: TOKEN-STANDARDS-005

---

## Research Brief

```yaml
knowledge_id: TOKEN-STANDARDS-005
title: Wrapped Assets
research_question: >
  Wrapped asset란 무엇이며 왜 이런 표현 방식이 필요한가? 그리고 연구자는
  wrapped token의 기술적 fungibility와 그 뒤에 있는 custody, bridge,
  redemption 가정을 어떻게 분리해서 봐야 하는가?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-001
  - TOKEN-STANDARDS-004
parent: Token Standards
previous: TOKEN-STANDARDS-004
next: TOKEN-STANDARDS-006
related_topics:
  - Stablecoins
  - Bridges
  - DeFi
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - System Structure
  - Conversion Mechanics
  - Economic Model
  - Security Considerations
  - Implementation Notes
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Full bridge taxonomy
  - Per-chain wrapped-token registry
  - Legal custody analysis
```

## 1. Learning Objectives

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- wrapped asset를 다른 자산의 tokenized representation으로 정의할 수 있다.
- native-asset wrapper와 bridge-backed representation을 구분할 수 있다.
- deposit 시 mint, redeem 시 burn되는 흐름을 설명할 수 있다.
- wrapped token 뒤에 숨어 있는 신뢰 surface를 식별할 수 있다.
- composability의 이점과 custody·bridge 리스크를 분리할 수 있다.

---

## 2. Key Questions

1. wrapped asset는 왜 존재하는가?
2. wrapping은 일반적인 token issuance와 무엇이 다른가?
3. WETH와 bridged asset representation의 차이는 무엇인가?
4. 어떤 가정이 wrapped token을 신뢰 가능하게 만드는가?
5. wrapped asset를 평가할 때 분석가는 무엇을 추적해야 하는가?

---

## 3. Executive Summary

ethereum.org는 native ETH가 ERC-20보다 먼저 존재했고 ERC-20 인터페이스를 따르지 않기 때문에, 많은 application이 기대하는 ERC-20 semantics를 제공하기 위해 wrapped ETH(WETH)가 존재한다고 설명한다.[^ref-eth-weth]

WETH 모델에서 사용자는 smart contract에 ETH를 deposit하고, 같은 양의 WETH를 mint받는다. 이후 WETH를 다시 ETH로 redeem할 수 있으며, 이때 상환된 WETH는 burn된다.[^ref-eth-weth]

이것이 wrapped asset의 가장 단순한 형태다. 다른 곳에 보관되거나 lock된 자산을 토큰 형태로 표현하는 방식이다.

더 넓은 범주에는 다음이 포함된다.

- WETH 같은 native-asset wrapper,
- custodial token wrapper,
- 체인 간 bridge-backed representation.

Circle의 xReserve 문서는 source-chain smart contract에 보관된 USDC reserve가 attestation을 통해 remote chain상의 stablecoin representation을 뒷받침하는 구조를 설명한다.[^ref-circle-xreserve]

이 사례는 왜 wrapped asset 분석이 토큰 인터페이스 검토에서 멈출 수 없는지 보여준다. 핵심 질문은 다음과 같다.

- 기초 자산이 실제로 어디에 있는가,
- mint와 burn을 누가 통제하는가,
- redemption은 어떻게 이루어지는가,
- custodian, bridge, attestation system이 실패하면 어떤 일이 벌어지는가.

---

## 4. System Structure

### 4.1 What a Wrapped Asset Is

wrapped asset는 특정 전환 규칙 아래 다른 기초 자산을 나타내는 blockchain token이다. 그 기초 자산은 다음 중 하나일 수 있다.

- native asset,
- 다른 체인의 token,
- escrow된 token,
- custodian이 보유한 오프체인 자산.

### 4.2 Why Wrapping Is Needed

ethereum.org는 ETH가 ERC-20과 호환되지 않으며, wrapping을 통해 ERC-20 동작이 필요한 곳에서 ETH를 사용할 수 있게 된다고 설명한다.[^ref-eth-weth]

이 일반 패턴은 다른 경우에도 적용된다. wrapped representation은 원래 자산이 application이 요구하는 인터페이스 또는 네트워크 문맥을 직접 만족시키지 못하기 때문에 등장한다.

### 4.3 Main Design Families

연구자는 다음을 구분해야 한다.

- same-chain functional wrapper,
- cross-chain representation,
- custodian-backed synthetic form.

이들은 wallet 인터페이스에서는 비슷해 보여도 신뢰 구조와 실패 모드는 크게 다르다.

---

## 5. Conversion Mechanics

### 5.1 Canonical WETH Flow

ethereum.org는 기본적인 WETH 흐름을 다음과 같이 설명한다.

1. ETH를 WETH contract에 deposit한다.
2. 동일 수량의 WETH를 mint받는다.
3. 이후 WETH를 ETH로 redeem한다.
4. 상환된 WETH는 공급량에서 burn된다.[^ref-eth-weth]

이는 1:1 same-chain wrapper다.

### 5.2 Bridge-Backed Flow

Circle의 xReserve 가이드는 source chain에 reserve USDC를 보관하고, dual-attestation model을 사용해 remote chain의 USDC-backed token을 지원하는 구조를 설명한다.[^ref-circle-xreserve]

즉 representation layer는 다음에 의존한다.

- lock 또는 reserve accounting,
- attester,
- mint authorization,
- remote-chain token issuance logic.

### 5.3 Wrapped vs Native

ethereum.org는 WETH가 ETH의 ERC-20 representation이며, native ETH만이 gas fee에 사용되는 단위이고 WETH로 gas를 지불하는 것은 네이티브하게 지원되지 않는다고 설명한다.[^ref-eth-weth]

이 점은 중요한 원칙을 보여준다. wrapping은 전송 인터페이스를 맞춰줄 수는 있어도, 프로토콜 특권까지 동일하게 만들지는 않는다.

---

## 6. Mathematical or Economic Model

### 6.1 1:1 Wrapper Invariant

lock된 기초 자산 `U`와 wrapped supply `W`를 가진 단순 wrapper에 대해:

`U = W`

는 각 wrapped unit이 완전하게 담보되고 즉시 상환 가능하다면 성립해야 하는 조건이다.

### 6.2 Bridge Representation Condition

source reserve `R_s`와 remote wrapped supply `W_r`에 대해:

`R_s >= W_r`

는 최소한의 담보 조건이며, 여기에 attestation 정확성과 settlement liveness가 함께 충족되어야 한다.

### 6.3 Liquidity vs Backing

wrapped token은 담보는 유지되더라도 다음 상황에서는 parity에서 이탈해 거래될 수 있다.

- redemption이 지연될 때,
- bridge exit가 혼잡할 때,
- 사용자가 운영자를 신뢰하지 않을 때.

즉 가격 parity는 nominal backing뿐 아니라 운영 신뢰에도 의존한다.

---

## 7. Security Considerations

### 7.1 Smart Contract Risk

ethereum.org는 canonical WETH를 단순하고 충분히 검증된 contract로 설명하며 formal verification도 언급하지만, 동시에 실제 환경에는 다르게 동작할 수 있는 다른 WETH variant도 존재한다고 경고한다.[^ref-eth-weth]

따라서 연구자는 canonical wrapper와 모방 토큰을 구분해야 한다.

### 7.2 Bridge and Attestation Risk

bridge-backed wrapped asset는 same-chain wrapper보다 더 많은 trust surface를 도입한다.

- message validation,
- reserve accounting,
- signer integrity,
- upgrade control.

### 7.3 Redemption Risk

wrapped asset의 강도는 wrapped representation에서 underlying으로 되돌아가는 경로의 강도와 같다. redemption이 멈추면 wrapper는 계속 거래되더라도 더 이상 신뢰할 수 있는 대체재로 기능하지 않을 수 있다.

---

## 8. Implementation Notes

Circle의 USDC 문서는 native token 자체를 설명하고, xReserve 문서는 reserve-held USDC가 remote chain representation을 뒷받침하는 구조를 설명한다.[^ref-circle-usdc][^ref-circle-xreserve]

이 구분은 매우 유용하다.

- native issuance,
- versus wrapped 또는 backed remote representation.

동일한 ticker나 동일한 경제적 서사 아래에서도 체인별 settlement architecture는 전혀 다를 수 있다.

---

## 9. On-Chain Implications

### 9.1 Composability

wrapped asset는 원래 호환되지 않던 자산을 ERC-20 기반 protocol, DEX pool, vault, lending system에서 사용할 수 있게 만든다.

### 9.2 Identity Ambiguity

사용자는 종종 하나의 symbol을 보고 하나의 자산이라고 생각한다. 분석가는 다음을 반드시 확인해야 한다.

- contract address,
- chain,
- issuer 또는 wrapper,
- redemption route.

### 9.3 Monitoring

주요 관측 변수는 다음과 같다.

- 공급량 증가,
- 보이는 범위의 reserve 변화,
- mint/burn pattern,
- bridge pause 또는 attestation failure,
- 기초 자산 대비 discount 또는 premium.

---

## 10. Institutional Thinking

- wrapped asset는 단순 토큰이 아니라 claim으로 다뤄야 한다.
- wrapper 아래 무엇이 놓여 있는지, exit를 누가 통제하는지 항상 물어야 한다.
- WETH 같은 same-chain wrapper와 bridge IOU, custodian claim을 구분해야 한다.
- 동일한 ERC-20 인터페이스는 전혀 다른 운영 리스크를 숨길 수 있다.
- treasury 또는 settlement 용도로 사용할 때는 유동성 깊이보다 redemption reliability를 먼저 평가해야 한다.

---

## 11. Evidence Classification

| Claim | Classification | Basis |
|---|---|---|
| WETH exists because ETH is not ERC-20 compatible | Directly specified | ethereum.org WETH documentation |
| WETH is minted on deposit and burned on redemption | Directly specified | ethereum.org WETH documentation |
| Wrapped-asset security varies by wrapper implementation | Directly specified and inferred | ethereum.org warning about variants |
| Reserve-backed remote representations depend on attestation and reserve flow | Directly specified | Circle xReserve docs |
| Interface parity does not imply equal protocol privilege or risk | Analytical inference | Native ETH vs WETH and bridge architecture |

---

## 12. References

[^ref-eth-weth]: ethereum.org, "Wrapped ether (WETH)," official documentation, last updated 2026-07-23, accessed 2026-08-04, https://ethereum.org/wrapped-eth/

[^ref-circle-usdc]: Circle Docs, "What is USDC?," official documentation, accessed 2026-08-04, https://developers.circle.com/stablecoins/what-is-usdc

[^ref-circle-xreserve]: Circle Docs, "xReserve Technical Guide," official documentation, accessed 2026-08-04, https://developers.circle.com/xreserve/concepts/technical-guide

---

## 13. Cross References

### Previous

- TOKEN-STANDARDS-004 — Stablecoins

### Next

- TOKEN-STANDARDS-006 — Governance Tokens

---

## Review Status

### Technical Review

Passed.

- WETH 메커니즘과 bridge-backed representation을 분리했다.
- native asset의 특권과 ERC-20 호환성을 혼동하지 않았다.

### Evidence Review

Passed.

- WETH 메커니즘은 ethereum.org에 연결했다.
- remote reserve-backed representation 관련 주장은 Circle 문서에 연결했다.

### Editorial Review

Passed.

- 구조는 repository 관례와 일치한다.
- 리스크 용어는 bridge 및 token 모듈과 일관된다.

### Adversarial Review

Passed.

- 이 문서는 모든 wrapped asset가 동일하게 신뢰 가능하다고 암시하지 않는다.
- ticker가 같다고 redemption equivalence가 성립한다고 보지 않는다.
- 모든 WETH 유사 contract가 canonical하다고 가정하지 않는다.

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
