---
knowledge_id: DEFI-010
title: Bridges
subtitle: 크로스체인 자산 이동, 메시지 전달, 그리고 상호운용성 뒤의 신뢰 tradeoff
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 105 min
estimated_study: 250 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Interoperability
  - Ethereum
parent:
  - DeFi
prerequisites:
  - TOKEN-STANDARDS-005
  - DEFI-003
related_topics:
  - Wrapped Assets
  - DeFi Risks
  - MEV
primary_sources:
  - REF-ETH-BRIDGES-2026-001
tags:
  - bridges
  - cross-chain
  - interoperability
---

# Bridges
> DeFi  
> Research Unit: DEFI-010

---

## Research Brief

```yaml
knowledge_id: DEFI-010
title: Bridges
research_question: >
  Blockchain bridge는 체인 간에 자산과 메시지를 어떻게 이동시키며, 어떤 trust
  model이 존재하고, 왜 bridge는 DeFi에서 가장 결과가 큰 risk surface 중
  하나인가?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-005
  - DEFI-003
parent: DeFi
previous: DEFI-009
next: DEFI-011
```

## 1. Learning Objectives

- blockchain bridge를 정의할 수 있다.
- lock-and-mint, burn-and-mint, atomic-swap 설계를 구분할 수 있다.
- trusted bridge와 trust-minimized bridge를 구분할 수 있다.
- bridge risk가 왜 시스템적 리스크가 되는지 설명할 수 있다.

## 2. Executive Summary

ethereum.org는 bridge를, 서로 고립된 체인 사이에서 자산, 메시지, 데이터를 이동시킬 수 있도록 연결하는 구조로 설명한다.[^ref-eth-bridges]

같은 문서는 주요 메커니즘으로 다음을 제시한다.

- lock and mint,
- burn and mint,
- atomic swaps.[^ref-eth-bridges]

또한 security, convenience, connectivity, data-passing ability를 bridge의 핵심 tradeoff로 강조한다.[^ref-eth-bridges]

## 3. Bridge Models

### 3.1 Lock and Mint

자산은 source chain에서 lock되고, destination chain에서 representation이 발행된다.

### 3.2 Burn and Mint

representation은 한 체인에서 burn되고 다른 체인에서 mint된다.

### 3.3 Liquidity-Network Transfers

Liquidity network는 canonical custody 경로 대신 counterpart liquidity에 의존한다.

## 4. Trust Model

ethereum.org는 trusted bridge와 trustless bridge를 구분하며, 외부 validator가 추가 신뢰 가정을 도입한다는 점을 강조한다.[^ref-eth-bridges]

## 5. Risks

ethereum.org는 bridge가 가장 큰 DeFi 해킹 사례들 중 일부를 차지했다고 명시하며, 다음 리스크를 지적한다.

- smart-contract risk,
- wrapped asset에서 비롯되는 systemic risk,
- counterparty risk,
- 아직 해결되지 않은 open issue.[^ref-eth-bridges]

## 6. Institutional Thinking

- bridging은 단순한 transport가 아니라 risk transformation이다.
- bridge를 사용하는 순간, 그 bridge의 security model이 해당 bridged asset를 사용하는 downstream protocol 전부로 수입된다.

## 7. References

[^ref-eth-bridges]: ethereum.org, "Bridges," official documentation, page last update 2026-04-03, accessed 2026-08-04, https://ethereum.org/developers/docs/bridges

## 8. Cross References

- Previous: DEFI-009 — Restaking
- Next: DEFI-011 — MEV

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
