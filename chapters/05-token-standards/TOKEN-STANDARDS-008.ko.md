---
knowledge_id: TOKEN-STANDARDS-008
title: Tokenomics
subtitle: 공급 설계, incentive architecture, 그리고 token function과 가격 서사의 차이
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 115 min
estimated_study: 270 min
last_reviewed: 2026-08-04
domain:
  - Token Standards
  - Ethereum
  - Economics
parent:
  - Token Standards
prerequisites:
  - TOKEN-STANDARDS-004
  - TOKEN-STANDARDS-006
  - TOKEN-STANDARDS-007
related_topics:
  - Governance Tokens
  - Utility Tokens
  - Security Budget
primary_sources:
  - REF-CHAINLINK-ECON-2026-001
  - REF-MAKER-RATES-2026-001
  - REF-MAKER-STABILIZER-2026-001
  - REF-MAKER-MKR-2026-001
tags:
  - tokenomics
  - incentives
  - supply
---

# Tokenomics
> Token Standards  
> Research Unit: TOKEN-STANDARDS-008

---

## Research Brief

```yaml
knowledge_id: TOKEN-STANDARDS-008
title: Tokenomics
research_question: >
  Tokenomics는 공급, 수요, incentive, 손실 배분 시스템으로서 어떻게
  연구되어야 하며, 연구자는 실제 protocol mechanics와 가치 포획에 대한
  마케팅 서사를 어떻게 구분해야 하는가?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-004
  - TOKEN-STANDARDS-006
  - TOKEN-STANDARDS-007
parent: Token Standards
previous: TOKEN-STANDARDS-007
next:
related_topics:
  - Governance Tokens
  - Utility Tokens
  - Security Budget
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - System Components
  - Incentive Mechanics
  - Economic Model
  - Security Considerations
  - Implementation Notes
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Token price prediction
  - Jurisdictional tax treatment
  - Macro valuation frameworks
```

## 1. Learning Objectives

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- tokenomics를 프로토콜의 incentive 및 공급 시스템으로 정의할 수 있다.
- issuance, burn, staking, governance, loss-allocation 메커니즘을 분리할 수 있다.
- 토큰이 utility를 포획하는지, risk를 포획하는지, 혹은 둘 다 포획하는지 평가할 수 있다.
- 가격 서사와 실제 메커니즘을 구분할 수 있다.
- token structure에 대한 due diligence checklist를 구성할 수 있다.

---

## 2. Key Questions

1. tokenomics 분석에는 어떤 구성 요소가 포함되어야 하는가?
2. issuance, burn, staking, governance는 어떻게 상호작용하는가?
3. 어떤 token design이 다른 설계보다 더 reflexive해지는 이유는 무엇인가?
4. 손실은 어떻게 사회화되거나 흡수되는가?
5. 분석가는 value-capture claim을 어떻게 평가해야 하는가?

---

## 3. Executive Summary

tokenomics는 market sentiment의 동의어가 아니라, tokenized system의 경제적 아키텍처로 연구되어야 한다.

Chainlink의 economics 문서는 reserve accumulation, staking, reward, build incentive, scale subsidy, payment abstraction을 중심으로 한 명시적 경제 설계를 제시한다.[^ref-chainlink-econ]

Maker 문서는 rates, system stabilization, MKR mint/burn 경로가 token mechanics를 protocol stability와 recapitalization에 어떻게 연결하는지 보여준다.[^ref-maker-rates][^ref-maker-stabilizer][^ref-maker-mkr]

이 사례들은 tokenomics 분석이 최소한 다음을 다뤄야 함을 보여준다.

- issuance,
- supply sink 및 burn,
- utility demand,
- governance right,
- security incentive,
- loss-absorption path.

핵심 규율은 프로토콜의 token이 다음 중 무엇을 포획하는지 검증하는 데 있다.

- usage,
- security,
- control,
- downside.

이 연결이 어느 것도 강하지 않다면, tokenomics는 대부분 narrative에 불과할 수 있다.

---

## 4. System Components

### 4.1 Supply Side

tokenomics는 토큰이 어떻게 circulation에 들어오고 빠져나가는지에서 출발한다.

- issuance,
- inflation,
- mint authority,
- vesting,
- burn,
- lockup.

Maker의 MKR 문서는 authorized minting과 burning 기능을 명시적으로 포함한다.[^ref-maker-mkr]

### 4.2 Demand Side

수요는 다음에서 발생할 수 있다.

- 서비스 결제,
- staking,
- governance 참여,
- collateral 활용,
- treasury 기대,
- speculation.

Chainlink economics는 LINK 관련 사용과 축적에 연결된 여러 demand pathway를 문서화한다.[^ref-chainlink-econ]

### 4.3 Stability and Backstop Side

Maker의 system stabilizer 문서는 debt auction과 surplus auction을 설명하며, 시스템 surplus 또는 deficit 상태에 따라 MKR의 역할이 달라질 수 있음을 보여준다.[^ref-maker-stabilizer]

즉 tokenomics는 단순한 upside capture만이 아니라, 누가 스트레스를 흡수하는지도 정의한다.

---

## 5. Incentive Mechanics

### 5.1 Issuance and Dilution

새 토큰이 발행된다면 연구 질문은 단순히 얼마나 많이 발행되는가가 아니다. 더 중요한 것은 다음이다.

- 누구에게 배분되는가,
- 어떤 조건에서 발행되는가,
- 어떤 행동을 유도하는가,
- lock 또는 exit profile은 무엇인가.

### 5.2 Burn and Buyback Narratives

burn 메커니즘은 공급량을 줄일 수 있지만, 실제 질문은 무엇이 그 burn의 재원을 제공하며 그 재원이 지속 가능한가다.

Maker의 surplus-auction process는 MKR burn을 단순 서사가 아니라 protocol surplus dynamics에 연결한다.[^ref-maker-stabilizer]

### 5.3 Security Incentives

Chainlink staking은 cryptoeconomic security layer로 문서화되어 있다.[^ref-chainlink-econ]

이는 단순 reward 기능이 아니라 security 기능을 가진 tokenomics다.

### 5.4 Rate and Monetary Policy

Maker의 rates module은 stability fee와 DSR accumulation이 cumulative-rate 메커니즘을 통해 어떻게 처리되는지 설명한다.[^ref-maker-rates]

이 점은 tokenomics가 종종 token contract logic을 넘어 프로토콜 전체의 monetary rule과 accounting rule까지 확장된다는 사실을 보여준다.

---

## 6. Mathematical or Economic Model

### 6.1 Supply Decomposition

다음을 두자.

- `S_total` = 총 공급량
- `S_circ` = 유통 공급량
- `S_locked` = lock되었거나 staking된 공급량
- `S_treasury` = treasury가 통제하는 공급량

그러면:

`S_total = S_circ + S_locked + S_treasury + other restricted buckets`

가 된다.

headline supply만으로는 충분하지 않다. bucket 분석이 필요하다.

### 6.2 Net Value-Capture Sketch

프로토콜 연계 수요 동인을 다음처럼 두자.

- `U` = 서비스 utility 수요
- `G` = governance 수요
- `K` = security 또는 staking 수요
- `L` = loss-absorption expectation

그러면 token relevance는 다음처럼 개략화할 수 있다.

`R_token = f(U, G, K, L)`

각 항은 약할 수도, 강할 수도, 대부분 narrative에 불과할 수도 있다.

### 6.3 Reflexivity

token의 보안 역할, collateral quality, governance power, treasury health가 모두 token price에 실질적으로 의존한다면 reflexivity는 높다.

높은 reflexivity는 상승 구간의 convexity를 키우는 동시에 하락 시 취약성도 키운다.

---

## 7. Security Considerations

### 7.1 Narrative Overreach

프로젝트는 protocol usage를 token demand, burn, right에 연결하는 명확한 메커니즘 없이도 value capture를 주장할 수 있다.

### 7.2 Treasury and Governance Concentration

treasury와 voting power가 집중되어 있다면, tokenomics는 소수 행위자에 의해 incentive가 지배되는 시스템을 설명하는 것에 그칠 수 있다.

### 7.3 Hidden Downside

Maker의 stabilizer와 MKR module은 스트레스 상황에서 token holder가 recapitalization path에 노출될 수 있음을 명시적으로 드러낸다.[^ref-maker-stabilizer][^ref-maker-mkr]

분석가는 upside가 어디에 약속되는지뿐 아니라 downside가 어디에 배정되는지도 항상 물어야 한다.

---

## 8. Implementation Notes

Chainlink와 Maker는 tokenomics를 marketing slide가 아니라 operational system으로 드러낸다는 점에서 좋은 1차 사례다.

- Chainlink: 서비스, staking, reserve, payment routing.[^ref-chainlink-econ]
- Maker: fee, surplus, deficit, recapitalization, governance-linked control.[^ref-maker-rates][^ref-maker-stabilizer][^ref-maker-mkr]

따라서 실무적인 tokenomics review는 token contract뿐 아니라 가치 흐름을 라우팅하고, rate를 강제하며, 실패를 흡수하는 주변 contract도 함께 살펴야 한다.

---

## 9. On-Chain Implications

### 9.1 Observable Variables

분석가는 대개 다음을 관찰할 수 있다.

- mint 및 burn event,
- staking flow,
- delegation 또는 governance participation,
- treasury transfer,
- lockup 변화.

### 9.2 Partially Observable Variables

어떤 중요한 변수는 여전히 부분적으로 또는 전부 오프체인에 남아 있다.

- insider coordination,
- market-making arrangement,
- treasury deployment plan,
- user expectation formation.

### 9.3 Dashboard Discipline

tokenomics dashboard는 다음을 분리해야 한다.

- supply fact,
- governance fact,
- security fact,
- inference 기반 valuation narrative.

---

## 10. Institutional Thinking

- tokenomics는 brand analysis가 아니라 mechanism analysis로 다뤄야 한다.
- 수요가 어디서 오고 손실이 어디로 가는지 물어야 한다.
- token utility와 token price를 분리해야 한다.
- lockup, treasury control, burn 재원, dilution path를 함께 검토해야 한다.
- 실질 downside absorption을 가진 토큰은, 더 시끄러운 upside marketing만 가진 토큰보다 경제적으로 더 중요할 수 있다.

---

## 11. Evidence Classification

| Claim | Classification | Basis |
|---|---|---|
| Chainlink documents reserve, staking, rewards, and payment-abstraction mechanisms | Directly specified | Chainlink economics docs |
| Maker documents rate accumulation, debt and surplus auctions, and MKR mint/burn pathways | Directly specified | Maker docs |
| Tokenomics must include downside allocation as well as upside capture | Analytical inference | Maker stabilization design |
| Price narrative should be separated from protocol mechanism | Analytical inference | Comparative mechanism review |

---

## 12. References

[^ref-chainlink-econ]: Chainlink, "Economics," official documentation, accessed 2026-08-04, https://chain.link/economics

[^ref-maker-rates]: MakerDAO Technical Docs, "Rates Module," official documentation, accessed 2026-08-04, https://docs.makerdao.com/smart-contract-modules/rates-module

[^ref-maker-stabilizer]: MakerDAO Technical Docs, "System Stabilizer Module," official documentation, accessed 2026-08-04, https://docs.makerdao.com/smart-contract-modules/system-stabilizer-module

[^ref-maker-mkr]: MakerDAO Technical Docs, "MKR Module," official documentation, accessed 2026-08-04, https://docs.makerdao.com/smart-contract-modules/mkr-module

---

## 13. Cross References

### Previous

- TOKEN-STANDARDS-007 — Utility Tokens

### Next

- Phase 5 complete

---

## Review Status

### Technical Review

Passed.

- supply, demand, security, downside allocation을 분리해 정리했다.
- tokenomics를 valuation rhetoric이 아니라 protocol mechanism으로 프레이밍했다.

### Evidence Review

Passed.

- Chainlink와 Maker의 mechanism 관련 주장을 공식 문서에 연결했다.
- 분석적 결론은 inference로 명시했다.

### Editorial Review

Passed.

- 구조는 repository 관례를 따른다.
- 용어는 Phase 5 나머지 문서와 일관된다.

### Adversarial Review

Passed.

- 이 문서는 tokenomics만으로 가격을 예측할 수 있다고 가장하지 않는다.
- burn narrative를 지속 가능한 value capture와 동일시하지 않는다.
- 숨겨진 downside 배정을 무시하지 않는다.

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
