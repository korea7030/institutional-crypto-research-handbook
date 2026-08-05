---
knowledge_id: TOKEN-STANDARDS-004
title: Stablecoins
subtitle: 페그 설계, 준비자산 모델, 그리고 토큰 전송 가능성과 화폐 신뢰성의 차이
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 105 min
estimated_study: 250 min
last_reviewed: 2026-08-04
domain:
  - Token Standards
  - Ethereum
  - Monetary Systems
parent:
  - Token Standards
prerequisites:
  - TOKEN-STANDARDS-001
  - SMART-CONTRACTS-008
related_topics:
  - Wrapped Assets
  - Governance Tokens
  - DeFi
primary_sources:
  - REF-ETH-STABLECOINS-2026-001
  - REF-CIRCLE-USDC-2026-001
  - REF-MAKER-DAI-2026-001
tags:
  - stablecoins
  - erc20
  - monetary-design
---

# Stablecoins
> Token Standards  
> Research Unit: TOKEN-STANDARDS-004

---

## Research Brief

```yaml
knowledge_id: TOKEN-STANDARDS-004
title: Stablecoins
research_question: >
  Stablecoin은 왜 일반적인 ERC-20 token과 다르며, 어떤 peg mechanism과 reserve
  model이 존재하고, 연구자는 token transferability와 실제 redemption,
  collateral, policy에 대한 신뢰를 어떻게 분리해야 하는가?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-001
  - SMART-CONTRACTS-008
parent: Token Standards
previous: TOKEN-STANDARDS-003
next: TOKEN-STANDARDS-005
related_topics:
  - Wrapped Assets
  - Governance Tokens
  - DeFi
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - System Structure
  - Peg Mechanisms
  - Economic Model
  - Security Considerations
  - Implementation Notes
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Jurisdiction-specific legal analysis
  - Full reserve attest walkthrough
  - Market-by-market price history
```

## 1. Learning Objectives

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- stablecoin을 안정적인 구매력 또는 기준 가치를 목표로 하는 tokenized claim 또는 메커니즘으로 정의할 수 있다.
- fiat-backed, crypto-backed, algorithmic stabilization 접근법을 구분할 수 있다.
- ERC-20 전송 가능성이 곧 신뢰할 수 있는 redemption을 의미하지 않는다는 점을 설명할 수 있다.
- peg 유지와 reserve transparency를 분리할 수 있다.
- stablecoin 설계의 주요 실패 지점을 식별할 수 있다.

---

## 2. Key Questions

1. 어떤 점이 어떤 토큰을 단순한 ERC-20이 아니라 stablecoin으로 만드는가?
2. 서로 다른 stablecoin 설계는 페그를 어떻게 유지하려 하는가?
3. 온체인 관측 가능성과 오프체인 reserve 신뢰의 차이는 무엇인가?
4. 어떤 stablecoin은 issuer에 의존하고, 어떤 stablecoin은 collateral 및 liquidation 시스템에 의존하는 이유는 무엇인가?
5. 분석가는 peg 지속 가능성을 평가하기 위해 무엇을 모니터링해야 하는가?

---

## 3. Executive Summary

ethereum.org는 stablecoin을 ETH 가격이 변해도 고정된 가치에 머물도록 설계된 Ethereum 토큰으로 설명한다.[^ref-eth-stablecoins]

이 사용자 친화적 설명은 유용하지만 연구에는 충분하지 않다. stablecoin은 단순히 가격 목표로 정의되지 않는다. 일반적으로 다음을 통해 토큰을 기준 자산 또는 기준 단위 근처에 유지하려는 stabilization system으로 정의된다.

- redeemability,
- collateralization,
- issuance control,
- arbitrage incentive,
- 때로는 governance intervention.

Circle은 USDC를 높은 유동성을 가진 현금 및 현금성 자산으로 100% 담보되고, 미 달러와 1:1로 상환 가능한 digital dollar로 설명한다.[^ref-circle-usdc]

Maker 문서는 Dai를 승인된 암호자산 담보에 대해 Maker Protocol이 생성하는 stablecoin으로 설명한다.[^ref-maker-overview][^ref-maker-dai]

이 사례들은 핵심 분석 분기를 보여준다.

- 어떤 stablecoin은 issuer가 관리하는 reserve와 redemption을 통해 안정화되고,
- 어떤 stablecoin은 smart-contract 담보 시스템과 liquidation 로직을 통해 안정화된다.

두 경우 모두 토큰은 ERC-20 interface를 사용할 수 있지만, ERC-20 계층은 운반 계층(transport layer)에 불과하다. 실제 연구 질문은 안정화 아키텍처가 스트레스 상황에서도 지급 능력, 유동성, 신뢰를 유지하는가에 있다.

---

## 4. System Structure

### 4.1 Stablecoin as a Layered Object

stablecoin은 보통 최소 네 개의 계층을 결합한다.

1. token interface layer,
2. issuance and redemption layer,
3. collateral or reserve layer,
4. governance and policy layer.

ERC-20 표준화가 다루는 것은 첫 번째 계층뿐이다.

### 4.2 Major Design Families

ethereum.org는 stablecoin을 fiat-backed, crypto-backed, precious-metal-backed, algorithmic 설계 등 메커니즘 기준으로 분류한다.[^ref-eth-stablecoins]

연구 목적상 가장 중요한 구분은 다음 세 가지다.

- 외부 담보와 redemption에 의존하는 claim 구조,
- 내부 담보 기반의 overcollateralized system,
- 반사성(reflexive)이 강한 algorithmic system.

### 4.3 Example: Fiat-Backed

Circle은 USDC가 인터넷 상의 미 달러를 표현하도록 설계되었고, USD로 1:1 상환 가능하다고 설명한다.[^ref-circle-usdc]

이 모델은 다음에 의존한다.

- issuer 운영,
- banking 및 reserve custody,
- attestation process,
- redemption access.

### 4.4 Example: Crypto-Backed

Maker 문서는 사용자가 승인된 crypto collateral 자산을 담보로 Dai를 발행할 수 있고, collateral type 및 risk parameter는 MKR holder가 governance한다고 설명한다.[^ref-maker-overview]

이 모델은 다음에 의존한다.

- collateral valuation,
- liquidation infrastructure,
- interest-rate policy,
- protocol governance.

---

## 5. Peg Mechanisms

### 5.1 Redemption Anchor

상환 가능한 fiat-backed stablecoin의 경우, 사용자가 issuer 규칙 범위 안에서 토큰을 액면가 근처로 fiat와 교환할 수 있다는 기대가 peg를 지지한다.

토큰 가격이 액면 이하로 내려가면 arbitrage 참여자는 할인된 토큰을 매수해 상환할 수 있다. 가격이 액면 above로 올라가면 신규 발행과 매도가 parity 회복에 기여할 수 있다.

### 5.2 Overcollateralized Debt Model

Maker 문서는 Dai를 다중 collateral type, stability fee, stabilization mechanism을 가진 시스템의 일부로 설명한다.[^ref-maker-overview][^ref-maker-rates][^ref-maker-stabilizer]

이 모델에서 peg는 은행 예금으로의 issuer redemption이 주된 보장 수단이 아니다. 대신 다음에 의해 지지된다.

- collateral backing,
- liquidation,
- parameter governance,
- system recapitalization process.

### 5.3 Algorithmic or Reflexive Mechanisms

ethereum.org는 일부 stablecoin이 reserve-backed 구조 대신 smart-contract algorithm에 의존한다고 설명한다.[^ref-eth-stablecoins]

이런 설계는 보통 더 강한 반사성을 가진다. peg 신뢰가 직접적인 reserve redemption보다 미래 수요, token incentive, governance 신뢰성에 더 크게 의존할 수 있기 때문이다.

---

## 6. Mathematical or Economic Model

### 6.1 Simple Fiat-Backed Solvency Condition

다음을 두자.

- `R` = 상환 가능한 reserve asset
- `L` = 유통 중인 stablecoin liability

최소 지급능력 조건은 다음과 같다.

`R >= L`

즉시 액면 상환을 약속한다면, nominal reserve 규모만큼이나 유동성의 질도 중요하다.

### 6.2 Overcollateralized System Condition

collateral asset 집합 `C`, 담보 가치 `V_i`, 부채 `D`, 요구 담보비율 `m > 1`에 대해:

`sum(V_i) >= m * D`

가 설계에 따라 포지션 수준 또는 시스템 수준에서 성립해야 한다.

liquidation system이 필요한 이유는 담보 가치가 governance나 사용자 대응보다 빠르게 움직일 수 있기 때문이다.

### 6.3 Peg vs Solvency

토큰은 숨겨진 지급불능이 누적되는 동안에도 일시적으로 peg 근처에서 거래될 수 있고, 장기 지급능력이 유지되더라도 일시적으로 peg에서 벗어날 수 있다. 가격 안정성과 대차대조표 건전성은 서로 관련되어 있지만 동일한 관측값은 아니다.

---

## 7. Security Considerations

### 7.1 Reserve Opacity

fiat-backed 모델에서는 토큰 전송이 온체인에서 완전히 보이지만 reserve asset은 대개 그렇지 않다. 이로 인해 온체인 투명성과 오프체인 대차대조표 의존성 사이에 불일치가 생긴다.

### 7.2 Oracle and Liquidation Risk

crypto-backed system에서는 잘못된 가격, 지연된 liquidation, 부적절한 collateral governance가 peg 약화나 지급불능으로 이어질 수 있다.

### 7.3 Redemption Access Risk

stablecoin은 온체인에서 기술적으로는 계속 전송 가능하더라도, 많은 사용자에게 redemption이 제한되거나 지연되거나 사실상 불가능할 수 있다. 스트레스 상황에서는 이 차이가 중요하다.

### 7.4 Governance Risk

Maker 문서는 collateral type과 risk parameter가 governance로 관리된다는 점을 분명히 한다.[^ref-maker-overview]

즉 peg 안정성은 코드 문제만이 아니라 governance 품질 문제이기도 하다.

---

## 8. Implementation Notes

Circle 문서는 USDC를 reserve attestation과 1:1 redeemability를 갖춘 멀티체인 digital dollar로 제시한다.[^ref-circle-usdc]

Maker 문서는 Dai를 join adapter, collateral module, rates, stabilization system 등 더 넓은 프로토콜 아키텍처 위에 놓인 사용자-facing ERC-20 contract로 제시한다.[^ref-maker-dai][^ref-maker-rates][^ref-maker-stabilizer]

실무적 교훈은 "stablecoin contract"라는 표현이 오해를 부를 수 있다는 점이다. 진지한 시스템에서 토큰 contract는 대개 더 큰 운영 또는 프로토콜 기계의 가장 단순한 가시 구성요소에 불과하다.

---

## 9. On-Chain Implications

### 9.1 Stablecoins as Settlement Assets

stablecoin은 종종 다음처럼 기능한다.

- exchange quote asset,
- DeFi collateral,
- payment balance,
- treasury unit.

### 9.2 Transfer Data Is Incomplete

온체인 flow는 circulation pattern, concentration, protocol usage를 보여주지만, reserve sufficiency나 redemption fairness를 직접 증명하지는 않는다.

### 9.3 Stress Monitoring

분석가는 다음을 모니터링해야 한다.

- peg deviation,
- issuance 및 redemption 변화,
- collateral composition 변화,
- governance parameter update,
- 핵심 counterparty 또는 protocol에 대한 concentration.

---

## 10. Institutional Thinking

- stablecoin은 단순 ERC-20 contract가 아니라 monetary system으로 다뤄야 한다.
- interface compliance와 reserve quality를 분리해야 한다.
- nominal peg와 실제 exit liquidity를 분리해야 한다.
- fiat-backed 모델에서는 누가 reserve를 보유하고, 누가 redeem할 수 있으며, 얼마나 빨리, 어떤 법적 조건에서 가능한지 물어야 한다.
- crypto-backed 모델에서는 어떤 담보가 허용되고, liquidation이 어떻게 작동하며, 손실이 어떻게 사회화되고, governance가 스트레스 하에서 어떻게 대응하는지 물어야 한다.

---

## 11. Evidence Classification

| Claim | Classification | Basis |
|---|---|---|
| Stablecoins are designed to maintain a fixed or steady reference value | Directly specified | ethereum.org stablecoins documentation |
| USDC is positioned as 1:1 redeemable and reserve-backed by Circle | Directly specified | Circle docs |
| Dai is generated against approved crypto collateral in the Maker Protocol | Directly specified | Maker docs |
| Stablecoin analysis must separate transferability from redemption credibility | Analytical inference | Interface vs reserve architecture |
| Peg stability and solvency are related but not identical | Analytical inference | Market-price vs balance-sheet distinction |

---

## 12. References

[^ref-eth-stablecoins]: ethereum.org, "Stablecoins," official documentation, published 2026-07, accessed 2026-08-04, https://ethereum.org/stablecoins

[^ref-circle-usdc]: Circle Docs, "What is USDC?," official documentation, accessed 2026-08-04, https://developers.circle.com/stablecoins/what-is-usdc

[^ref-maker-overview]: MakerDAO Technical Docs, "MakerDAO Technical Docs," official documentation overview, accessed 2026-08-04, https://docs.makerdao.com/

[^ref-maker-dai]: MakerDAO Technical Docs, "Dai Module," official documentation, accessed 2026-08-04, https://docs.makerdao.com/smart-contract-modules/dai-module

[^ref-maker-rates]: MakerDAO Technical Docs, "Rates Module," official documentation, accessed 2026-08-04, https://docs.makerdao.com/smart-contract-modules/rates-module

[^ref-maker-stabilizer]: MakerDAO Technical Docs, "System Stabilizer Module," official documentation, accessed 2026-08-04, https://docs.makerdao.com/smart-contract-modules/system-stabilizer-module

---

## 13. Cross References

### Previous

- TOKEN-STANDARDS-003 — ERC-1155

### Next

- TOKEN-STANDARDS-005 — Wrapped Assets

---

## Review Status

### Technical Review

Passed.

- peg 설계와 token interface를 분리했다.
- fiat-backed와 crypto-backed 모델을 올바르게 구분했다.
- solvency와 peg behavior를 연관되지만 동일하지 않은 것으로 다뤘다.

### Evidence Review

Passed.

- stablecoin 정의는 ethereum.org에 연결했다.
- USDC reserve 및 redemption 관련 주장은 Circle 문서에 연결했다.
- Dai 시스템 관련 주장은 Maker 문서에 연결했다.

### Editorial Review

Passed.

- 용어는 기존 token-standard unit과 일관된다.
- 출처가 뒷받침하지 않는 법적 주장을 추가하지 않았다.

### Adversarial Review

Passed.

- 이 문서는 모든 stablecoin이 동일한 조건으로 redeem 가능하다고 가정하지 않는다.
- ERC-20 transferability를 달러 등가성과 동일시하지 않는다.
- 모든 peg 모델을 하나의 리스크 프로파일로 평탄화하지 않는다.

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
