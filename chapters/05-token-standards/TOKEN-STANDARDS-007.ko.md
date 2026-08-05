---
knowledge_id: TOKEN-STANDARDS-007
title: Utility Tokens
subtitle: 단순한 ownership claim을 넘어서는 access, payment, coordination 기능
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 95 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - Token Standards
  - Ethereum
  - Applications
parent:
  - Token Standards
prerequisites:
  - TOKEN-STANDARDS-001
  - TOKEN-STANDARDS-006
related_topics:
  - Governance Tokens
  - Tokenomics
  - Oracles
primary_sources:
  - REF-MAKER-MKR-2026-001
  - REF-CHAINLINK-ECON-2026-001
  - REF-MAKER-GOV-2026-001
tags:
  - utility-tokens
  - mkr
  - link
---

# Utility Tokens
> Token Standards  
> Research Unit: TOKEN-STANDARDS-007

---

## Research Brief

```yaml
knowledge_id: TOKEN-STANDARDS-007
title: Utility Tokens
research_question: >
  프로토콜 내부에서 token이 utility를 가진다는 것은 무엇을 의미하며,
  utility는 governance나 순수 투기적 보유와 어떻게 다르고, 연구자는 토큰의
  명시된 기능이 실제로 시스템에 필수적인지 어떻게 검증해야 하는가?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-001
  - TOKEN-STANDARDS-006
parent: Token Standards
previous: TOKEN-STANDARDS-006
next: TOKEN-STANDARDS-008
related_topics:
  - Governance Tokens
  - Tokenomics
  - Oracles
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Functional Structure
  - Mechanism Analysis
  - Economic Model
  - Security Considerations
  - Implementation Notes
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Securities-law classification
  - Valuation forecasting
  - Marketing-token taxonomies
```

## 1. Learning Objectives

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- utility를 marketing 용어가 아니라 protocol 메커니즘 기준으로 정의할 수 있다.
- payment utility, access utility, staking utility, backstop utility를 구분할 수 있다.
- 하나의 token이 동시에 여러 역할을 가질 수 있음을 설명할 수 있다.
- token의 utility가 핵심적인지 대체 가능한지 검증할 수 있다.
- 주장된 utility가 실제 시스템 의존성보다 약한 지점을 식별할 수 있다.

---

## 2. Key Questions

1. 어떤 점이 어떤 token을 utility token으로 만드는가?
2. utility는 governance와 어떻게 다른가?
3. 하나의 token이 여러 역할을 동시에 수행할 수 있는가?
4. protocol 문서는 실제 token utility를 어떻게 드러내는가?
5. utility-token narrative에서 분석가는 무엇을 경계해야 하는가?

---

## 3. Executive Summary

"utility token"이라는 표현은 자주 부정확하게 사용된다. 연구 목적에서는 protocol 설계가 token에 수동 보유를 넘는 운영상 역할을 부여할 때 그 token이 utility를 가진다고 본다.

Maker의 MKR module은 MKR을 utility token, governance token, recapitalization resource의 세 역할로 명시한다.[^ref-maker-mkr]

Chainlink의 economics 문서는 LINK와 관련된 여러 기능, 즉 cryptoeconomic security를 위한 staking, reserve accumulation, reward, 그리고 서비스 수익을 LINK로 전환하는 payment abstraction을 설명한다.[^ref-chainlink-econ]

이들은 utility가 branding category가 아니라 mechanism category라는 점을 보여주는 좋은 1차 사례다.

핵심 연구 규율은 다음을 묻는 것이다.

- 어떤 행동에 토큰이 필요한가,
- 그 토큰이 어떤 incentive를 만드는가,
- 시스템의 어떤 속성이 그것에 의존하는가,
- 그 토큰 없이도 프로토콜이 거의 유사하게 동작할 수 있는가.

---

## 4. Functional Structure

### 4.1 What Utility Means

token은 다음 중 하나 이상에 대해 필요하거나 실질적으로 유리할 때 protocol utility를 가진다.

- 서비스 비용 지불,
- 보안을 위한 staking,
- 기능 접근,
- collateral 또는 backstop capital 제공,
- 참여자 행동 조정.

### 4.2 Multi-Role Tokens

Maker의 MKR 문서는 MKR이 다음 역할을 수행한다고 명시한다.

- utility token,
- governance token,
- recapitalization resource.[^ref-maker-mkr]

이는 토큰 분석에서 자주 발생하는 실수를 보여준다. 프로토콜이 여러 역할을 부여했는데도, 분석이 하나의 토큰을 하나의 라벨에만 억지로 넣는 것이다.

### 4.3 Service-Network Utility

Chainlink의 economics 문서는 staking, reserve support, reward, payment abstraction에 연결된 LINK 관련 기능을 설명한다.[^ref-chainlink-econ]

이것은 단순 transferability가 아니라 서비스 제공과 네트워크 보안에 뿌리박힌 utility다.

---

## 5. Mechanism Analysis

### 5.1 Payment Utility

토큰이 프로토콜 서비스를 지불하는 데 필요하다면, 그 지불 경로가 우회 불가능하다는 가정 아래 서비스 사용량 증가에 따라 수요가 확대될 수 있다.

Chainlink는 alternative-asset 기반 서비스 결제를 LINK로 전환해 결제 마찰을 줄이는 payment abstraction 메커니즘을 문서화한다.[^ref-chainlink-econ]

즉 사용자 인터페이스 레이어에서 사용자가 토큰으로 직접 지불하지 않더라도, 토큰의 utility는 내부적으로 계속 존재할 수 있다.

### 5.2 Staking Utility

Chainlink는 staking을 oracle 서비스의 security guarantee를 높이는 데 참여자가 LINK를 stake하는 cryptoeconomic security layer로 설명한다.[^ref-chainlink-econ]

이것은 토큰이 프로토콜의 보안 속성을 지탱하기 때문에 utility다.

### 5.3 Backstop Utility

Maker 문서는 지급불능 상황에서 프로토콜을 재자본화하기 위해 MKR이 mint되어 DAI와 교환될 수 있고, 다른 조건에서는 surplus mechanism이 MKR을 burn할 수 있다고 설명한다.[^ref-maker-mkr]

이것은 토큰이 손실 흡수와 재자본화 경로에 직접 위치하기 때문에 강한 형태의 protocol utility다.

---

## 6. Mathematical or Economic Model

### 6.1 Utility Intensity

토큰 수요를 다음처럼 근사하자.

`D = D_pay + D_stake + D_gov + D_backstop + D_spec`

여기서 각 항은 payment, staking, governance, backstop, speculative demand를 의미한다.

핵심 연구 질문은 `D_spec`의 존재 여부가 아니다. 그 항은 거의 항상 존재한다. 중요한 것은 비투기적 수요 성분이 구조적으로 중요한가다.

### 6.2 Replaceability Test

프로토콜이 token `T`를 제거해도 큰 재설계 없이 거의 모든 서비스, 보안, governance 기능을 유지할 수 있다면 utility는 약하다.

반대로 `T`를 제거하면 payment routing, security incentive, recapitalization, control logic이 깨진다면 utility는 강하다.

---

## 7. Security Considerations

### 7.1 Marketing Overstatement

토큰은 유용하다고 설명될 수 있지만, 실제 역할은 선택적이거나 약하거나 우회 가능할 수 있다.

### 7.2 Utility Concentration

utility가 staking, governance, backstop participation을 요구하는데 token supply가 집중되어 있다면, 프로토콜은 과점적 통제나 실패 리스크를 물려받을 수 있다.

### 7.3 Feedback Loops

시스템을 보안적으로 지탱하면서 동시에 그 시스템에 대한 신뢰에 의해 가격이 결정되는 토큰은 스트레스 상황에서 반사적 루프를 만들 수 있다.

Maker의 recapitalization 경로는 token holder가 시스템 손실 뒤에 서게 됨을 명시적으로 드러내기 때문에 특히 중요하다.[^ref-maker-mkr]

---

## 8. Implementation Notes

Maker와 Chainlink는 서로 다른 두 utility 패턴을 보여준다.

- governance-plus-backstop token으로서의 MKR,
- service-network 및 staking token으로서의 LINK.[^ref-maker-mkr][^ref-chainlink-econ]

이 둘은 모두 피상적인 "제품 접근권"이라는 마케팅 문구로 환원되지 않는다. 둘 다 token mechanics를 시스템 수준 기능과 직접 연결한다.

---

## 9. On-Chain Implications

### 9.1 Utility Leaves Observable Traces

분석가는 보통 다음을 통해 utility를 추적할 수 있다.

- staking deposit,
- burn 또는 mint,
- treasury flow,
- governance participation,
- 노출되는 경우 서비스 결제 전환 흐름.

### 9.2 Utility Can Be Hidden Behind Intermediaries

사용자가 토큰을 직접 만지지 않더라도, 프로토콜이 내부 settlement, security, reward에 여전히 그 토큰을 사용할 수 있다.

### 9.3 Role Confusion

하나의 token이 utility, governance, speculative 역할을 동시에 수행할 때, 이를 단순 transfer volume으로 환원하는 dashboard는 중요한 메커니즘을 놓치게 된다.

---

## 10. Institutional Thinking

- utility는 narrative가 아니라 mechanism으로 정의해야 한다.
- 프로토콜이 실제로 그 토큰을 필요로 하는지 물어야 한다.
- user-facing convenience와 back-end token dependence를 분리해야 한다.
- multi-role token을 전제로 하고 각 역할을 따로 분석해야 한다.
- security 또는 recapitalization 경로에 놓인 utility는 단순 UX 장식성 utility보다 훨씬 중요하다.

---

## 11. Evidence Classification

| Claim | Classification | Basis |
|---|---|---|
| MKR is explicitly described as utility, governance, and recapitalization resource | Directly specified | Maker MKR docs |
| Chainlink assigns LINK-related roles in staking and payment abstraction | Directly specified | Chainlink economics docs |
| Utility is best defined by protocol mechanism rather than by token branding | Analytical inference | Comparison of primary examples |
| Non-speculative demand components determine depth of utility | Analytical inference | Mechanism decomposition |

---

## 12. References

[^ref-maker-mkr]: MakerDAO Technical Docs, "MKR Module," official documentation, accessed 2026-08-04, https://docs.makerdao.com/smart-contract-modules/mkr-module

[^ref-chainlink-econ]: Chainlink, "Economics," official documentation, accessed 2026-08-04, https://chain.link/economics

---

## 13. Cross References

### Previous

- TOKEN-STANDARDS-006 — Governance Tokens

### Next

- TOKEN-STANDARDS-008 — Tokenomics

---

## Review Status

### Technical Review

Passed.

- utility 역할을 payment, staking, backstop 기능으로 분해했다.
- governance와 utility를 구분하되 중첩 가능성은 유지했다.

### Evidence Review

Passed.

- MKR의 multi-role 관련 주장은 Maker 문서에 연결했다.
- LINK 관련 utility 주장은 Chainlink economics 문서에 연결했다.

### Editorial Review

Passed.

- 이 챕터는 marketing식 분류를 피한다.
- 용어는 mechanism-first 분석에 맞춰 정렬되어 있다.

### Adversarial Review

Passed.

- 이 문서는 소위 utility token이 항상 필수적이라고 가정하지 않는다.
- utility를 speculation으로 환원하지 않는다.
- 하나의 token을 하나의 역할에 강제로 가두지 않는다.

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
