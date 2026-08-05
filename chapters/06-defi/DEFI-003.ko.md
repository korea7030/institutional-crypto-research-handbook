---
knowledge_id: DEFI-003
title: DEX
subtitle: 탈중앙화 거래소 아키텍처, self-custody execution, 그리고 온체인 시장 접근
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 95 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Trading
  - Ethereum
parent:
  - DeFi
prerequisites:
  - DEFI-001
  - DEFI-002
related_topics:
  - MEV
  - Bridges
  - Token Standards
primary_sources:
  - REF-UNI-HOW-2026-001
  - REF-UNI-PROTOCOLS-2026-001
  - REF-UNI-SWAP-2026-001
tags:
  - dex
  - trading
  - uniswap
---

# DEX
> DeFi  
> Research Unit: DEFI-003

---

## Research Brief

```yaml
knowledge_id: DEFI-003
title: DEX
research_question: >
  프로토콜 계층에서 decentralized exchange를 정의하는 요소는 무엇이며,
  self-custodial execution은 centralized exchange workflow와 어떻게 다르고,
  custody를 제거한 뒤에도 어떤 리스크가 남는가?
document_type: foundation
difficulty: L300
prerequisites:
  - DEFI-001
  - DEFI-002
parent: DeFi
previous: DEFI-002
next: DEFI-004
related_topics:
  - MEV
  - Bridges
```

## 1. Learning Objectives

- DEX를 프로토콜 관점에서 정의할 수 있다.
- custody, execution, governance layer를 구분할 수 있다.
- routing과 swap safety parameter를 설명할 수 있다.
- DEX 특유의 execution risk를 식별할 수 있다.

## 2. Executive Summary

Uniswap은 자신을 decentralization, censorship resistance, self-custody를 위해 설계된 Ethereum 기반 decentralized exchange protocol로 문서화한다.[^ref-uni-how]

DEX에서는 일반적으로 사용자가 smart contract와 상호작용하기 전까지 자산 custody를 유지한다. execution은 중앙 운영자의 internal ledger 유지에 의존하지 않고, 규칙 기반이며 공개적으로 이루어진다.

이는 centralized exchange의 일부 리스크를 제거하지만, 다음 리스크까지 없애지는 못한다.

- smart-contract risk,
- oracle 또는 pricing mistake,
- frontrunning,
- adverse routing,
- bridge 및 wrapped-asset dependency.

## 3. Architecture

### 3.1 Core Exchange Logic

Uniswap의 protocol overview는 유동성을 보유하고 swap을 실행하는 AMM contract를 중심으로 한 온체인 protocol 집합을 설명한다.[^ref-uni-protocols]

### 3.2 Self-Custody

DEX execution은 CEX처럼 중앙 운영자의 대차대조표 안으로 자산을 미리 deposit하도록 요구하지 않는다.

### 3.3 Routing and Safety

Uniswap의 swap 가이드는 smart-contract trading에서 external price source와 minimum output 또는 maximum input bound 같은 safety parameter를 사용해 frontrun 손실을 줄여야 한다고 말한다.[^ref-uni-swap]

## 4. Operational Differences vs CEX

- settlement는 온체인에서 발생한다.
- market data와 transaction intent는 포함 전에 공개된다.
- execution은 block inclusion과 ordering에 의존한다.
- 자산은 contract interaction 전까지 사용자 지갑에 남아 있다.

## 5. Economic and Market Implications

DEX는 다른 protocol이 exchange logic을 직접 통합할 수 있기 때문에 접근성과 composability를 넓힌다. 동시에 composability는 DEX를 다음의 핵심 인프라로 만든다.

- arbitrage,
- liquidation,
- collateral management,
- token price formation.

## 6. Risks

- 얇은 유동성에서 발생하는 slippage.
- 공개 mempool로 인한 MEV.
- token approval abuse.
- wrapped 또는 bridged asset 오인식.
- smart-contract 또는 router bug.

## 7. Institutional Thinking

- DEX는 단순한 "venue"가 아니라 programmable market infrastructure다.
- custody를 제거해도 execution risk는 남는다.
- 온체인 투명성은 auditability를 주는 동시에 extractable value 기회도 만든다.

## 8. References

[^ref-uni-how]: Uniswap Developers, "How Uniswap Works," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/get-started/concepts/how-uniswap-works

[^ref-uni-protocols]: Uniswap Developers, "Protocols Overview," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/protocols/overview

[^ref-uni-swap]: Uniswap Developers, "Implement a Swap," official documentation, accessed 2026-08-04, https://developers.uniswap.org/docs/protocols/v2/guides/swapping

## 9. Cross References

- Previous: DEFI-002 — Liquidity Pools
- Next: DEFI-004 — Lending

## Review Status

### Technical Review
Passed.

### Evidence Review
Passed.

### Editorial Review
Passed.

### Adversarial Review
Passed.

### Quality Gate

| Gate | Status |
|---|---|
| Research scope was followed | Pass |
| Required primary sources were reviewed | Pass |
| Material claims are cited | Pass |
| Fact and interpretation are separated | Pass |
| No unresolved critical review issue remains | Pass |
