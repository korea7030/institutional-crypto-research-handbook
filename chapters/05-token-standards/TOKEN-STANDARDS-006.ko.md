---
knowledge_id: TOKEN-STANDARDS-006
title: Governance Tokens
subtitle: 의결권, delegation, timelock, 그리고 ownership과 control의 차이
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 110 min
estimated_study: 260 min
last_reviewed: 2026-08-04
domain:
  - Token Standards
  - Ethereum
  - Governance
parent:
  - Token Standards
prerequisites:
  - TOKEN-STANDARDS-001
  - SMART-CONTRACTS-006
related_topics:
  - Utility Tokens
  - Tokenomics
  - Protocol Governance
primary_sources:
  - REF-COMPOUND-GOV-2026-001
  - REF-OZ-GOVERNANCE-2026-001
  - REF-MAKER-GOV-2026-001
tags:
  - governance-tokens
  - comp
  - mkr
---

# Governance Tokens
> Token Standards  
> Research Unit: TOKEN-STANDARDS-006

---

## Research Brief

```yaml
knowledge_id: TOKEN-STANDARDS-006
title: Governance Tokens
research_question: >
  온체인 protocol에서 governance token은 어떤 기능을 수행하며, voting,
  delegation, execution path는 어떻게 동작하고, 연구자는 token ownership과
  실질적인 protocol control을 어떻게 구분해야 하는가?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-001
  - SMART-CONTRACTS-006
parent: Token Standards
previous: TOKEN-STANDARDS-005
next: TOKEN-STANDARDS-007
related_topics:
  - Utility Tokens
  - Tokenomics
  - Protocol Governance
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Governance Structure
  - Voting Mechanics
  - Economic Model
  - Security Considerations
  - Implementation Notes
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Political philosophy of governance
  - Per-protocol vote history
  - Legal control analysis
```

## 1. Learning Objectives

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- governance token을 프로토콜 변경에 대한 의사결정 권한을 배분하는 토큰으로 정의할 수 있다.
- delegation, quorum, threshold, timelock execution을 설명할 수 있다.
- token balance와 실효 voting power를 구분할 수 있다.
- protocol control이 명목상 token ownership과 어떻게 달라질 수 있는지 식별할 수 있다.
- governance-token concentration과 운영 리스크를 평가할 수 있다.

---

## 2. Key Questions

1. governance token은 실제로 무엇을 govern하는가?
2. voting power는 어떻게 측정되고 위임되는가?
3. proposal threshold, quorum, timelock의 역할은 무엇인가?
4. 왜 token ownership은 operational control과 다를 수 있는가?
5. governance가 token-weighted일 때 어떤 리스크가 발생하는가?

---

## 3. Executive Summary

Compound 문서는 프로토콜이 COMP token holder에 의해 governance 및 upgrade되며, 그 구조는 COMP token, governance module, Timelock의 세 요소로 구성된다고 설명한다.[^ref-compound-gov]

OpenZeppelin의 governance 문서는 onchain governance를 Governor contract, vote source, counting module, quorum rule, timelock execution path를 중심으로 구축되는 시스템으로 설명한다.[^ref-oz-governance]

Maker 문서는 MKR voting, proposal execution, voting security를 수행하는 governance contract를 설명한다.[^ref-maker-governance]

이 자료들을 종합하면 governance token은 단순히 branding이 붙은 ERC-20이 아니다. 그것은 다음을 결정할 수 있는 control system 안에 삽입된 토큰이다.

- parameter change,
- treasury action,
- upgrade path,
- collateral policy,
- emergency response.

핵심 분석 경고는 token balance만으로는 control을 측정할 수 없다는 점이다. 실질 control은 다음에 달려 있다.

- delegation,
- voter participation,
- quorum design,
- proposal threshold,
- timelock이나 admin을 통한 execution routing,
- 대형 holder나 delegate에 대한 집중도.

---

## 4. Governance Structure

### 4.1 Core Components

Compound의 governance 문서는 다음 세 요소를 명시적으로 분리한다.

- token,
- governor,
- timelock.[^ref-compound-gov]

이 3분할 모델은 많은 protocol에 적용 가능한 기본 프레임워크다.

### 4.2 Voting as a Separate Accounting Layer

Compound는 COMP holder가 자신 또는 다른 address에 voting rights를 delegate할 수 있고, delegated token balance에 따라 vote balance가 변한다고 설명한다.[^ref-compound-gov]

즉 governance power는 특정 시점의 단순 wallet balance와 다를 수 있다.

### 4.3 Governance as an Execution Pipeline

OpenZeppelin은 proposal creation, voting delay, voting period, quorum logic, 선택적 timelock queue/execution stage로 이루어진 governance pipeline을 문서화한다.[^ref-oz-governance]

따라서 governance는 한 번의 투표가 아니라 상태 기계(state machine)로 모델링하는 편이 정확하다.

---

## 5. Voting Mechanics

### 5.1 Delegation

Compound는 direct delegation과 signature 기반 delegation을 모두 문서화하며, governance system이 경제적 ownership과 실제 투표 참여를 분리할 수 있음을 보여준다.[^ref-compound-gov]

### 5.2 Snapshot Logic

OpenZeppelin은 vote power가 현재 balance가 아니라 historical snapshot에서 읽히는 경우가 많다고 설명하며, 이는 일부 이중 계산과 timing abuse를 막아 준다.[^ref-oz-governance]

### 5.3 Proposal Threshold and Quorum

Compound는 proposal threshold와 quorum vote를 governance의 명시적 parameter로 문서화한다.[^ref-compound-gov]

이 메커니즘은 극소 holder의 proposal spam을 막고, 정당성을 위해 최소 참여를 요구한다.

### 5.4 Timelock

Compound는 통과한 proposal이 execution 전에 Timelock에 queue된다고 설명한다.[^ref-compound-gov]

timelock은 exit window를 만들고 즉각적인 적대적 변경의 리스크를 줄이지만, governance capture 리스크 자체를 제거하지는 않는다.

---

## 6. Mathematical or Economic Model

### 6.1 Voting Power Function

다음을 두자.

- `b(a)` = 계정 `a`의 token balance
- `d(a)` = 계정 `a`가 선택한 delegate

그러면 delegate `x`의 실효 voting power는 다음처럼 모델링할 수 있다.

`v(x) = sum(b(a)) for all a such that d(a) = x`

즉 control concentration은 balance layer뿐 아니라 delegate layer에서도 측정해야 한다.

### 6.2 Proposal Success Condition

proposal `p`에 대해 `for`, `against`, `abstain`, quorum `Q`, success rule `S`가 주어지면, 통과에는 일반적으로 다음이 필요하다.

- quorum 충족,
- vote-success criterion 충족,
- execution path 완료.

토큰을 보유하고 있다는 사실만으로 policy가 실행되지는 않는다. 반드시 그 파이프라인을 통과해야 한다.

---

## 7. Security Considerations

### 7.1 Concentration Risk

governance token은 명목상 넓게 분산되어 보여도, 실제 활성 delegate, exchange, foundation, treasury, 초기 내부자에 고도로 집중되어 있을 수 있다.

### 7.2 Low Participation

quorum이 공급량 대비 낮거나 참여율이 지속적으로 약하면, 작은 조직화된 연합이 실질 통제권을 행사할 수 있다.

### 7.3 Governance Capture Through Execution Layer

Maker governance 문서는 governance가 단순 polling이 아니라 proposal execution과 voting security contract까지 포함함을 보여준다.[^ref-maker-governance]

execution path, timelock admin, pause authority가 중앙화되어 있다면 governance token이 control surface의 전부가 아닐 수 있다.

### 7.4 Timing and Tooling Risk

OpenZeppelin은 voting delay, voting period, timepoint-based quorum logic 같은 설정을 강조한다.[^ref-oz-governance]

이 parameter는 공격 surface와 참여자의 대응 가능성에 실질적인 영향을 준다.

---

## 8. Implementation Notes

Compound는 delegation이 가능한 ERC-20 voting, proposal threshold, quorum, voting window, timelocked execution으로 구성된 구체적인 governance-token 패턴을 제공한다.[^ref-compound-gov]

Maker는 MKR voting이 protocol business logic과 security process에 영향을 미치는 governance architecture를 제시한다.[^ref-maker-governance]

OpenZeppelin은 이러한 패턴을 재사용 가능한 governance module로 일반화한다.[^ref-oz-governance]

핵심 교훈은 "governance token"이 ERC-20 같은 하나의 단일 표준이 아니라, 별도의 governance architecture 안에서 토큰이 맡는 역할이라는 점이다.

---

## 9. On-Chain Implications

### 9.1 Observable Governance Surface

분석가는 대개 다음을 관측할 수 있다.

- delegation event,
- proposal creation,
- vote casting,
- queueing,
- execution.

### 9.2 Hidden Control Surface

동시에 다음도 점검해야 한다.

- multisig,
- emergency role,
- proxy admin,
- treasury custody,
- offchain coordination.

### 9.3 Balance vs Influence

가장 중요한 측정값은 총 token ownership이 아니라, 실행 가능한 governance outcome에 대한 실효 영향력인 경우가 많다.

---

## 10. Institutional Thinking

- governance token은 단순 거래 자산이 아니라 control instrument로 봐야 한다.
- concentration은 holder layer와 delegate layer 모두에서 측정해야 한다.
- governance가 실제로 어떤 action을 실행할 수 있는지 물어야 한다.
- signaling vote와 실행 가능한 onchain governance를 구분해야 한다.
- control이 분산되어 있다는 결론을 내리기 전에 timelock, guardian, emergency power, admin escape hatch를 검토해야 한다.

---

## 11. Evidence Classification

| Claim | Classification | Basis |
|---|---|---|
| Compound governance is built from token, governance module, and timelock | Directly specified | Compound docs |
| Governance tokens often use delegation and snapshots | Directly specified | Compound and OpenZeppelin docs |
| Timelocks provide delayed execution after successful votes | Directly specified | Compound and OpenZeppelin docs |
| Effective control can diverge from nominal ownership | Analytical inference | Delegation, quorum, and admin-path structure |
| Governance tokens are a role within an architecture, not a single universal token standard | Analytical inference | Comparison across sources |

---

## 12. References

[^ref-compound-gov]: Compound v2 Docs, "Governance," official documentation, accessed 2026-08-04, https://docs.compound.finance/v2/governance/

[^ref-oz-governance]: OpenZeppelin Docs, "How to set up on-chain governance" and governance API documentation, accessed 2026-08-04, https://docs.openzeppelin.com/contracts/5.x/governance and https://docs.openzeppelin.com/contracts/5.x/api/governance

[^ref-maker-governance]: MakerDAO Technical Docs, "Governance Module," official documentation, accessed 2026-08-04, https://docs.makerdao.com/smart-contract-modules/governance-module

---

## 13. Cross References

### Previous

- TOKEN-STANDARDS-005 — Wrapped Assets

### Next

- TOKEN-STANDARDS-007 — Utility Tokens

---

## Review Status

### Technical Review

Passed.

- delegation, quorum, execution을 명확히 분리했다.
- governance token balance와 실질 control을 혼동하지 않았다.

### Evidence Review

Passed.

- 핵심 governance pipeline 주장은 Compound와 OpenZeppelin 문서에 연결했다.
- Maker는 보편적 템플릿이 아니라 구체적 governance-system 사례로 사용했다.

### Editorial Review

Passed.

- 용어는 정밀함을 유지하며 decentralization을 과장하지 않는다.
- 구조는 기존 챕터 형식과 정렬된다.

### Adversarial Review

Passed.

- 이 문서는 token ownership만으로 protocol control이 보장된다고 암시하지 않는다.
- non-token admin surface를 무시하지 않는다.
- governance participation이 균일하게 분산되어 있다고 가정하지 않는다.

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
