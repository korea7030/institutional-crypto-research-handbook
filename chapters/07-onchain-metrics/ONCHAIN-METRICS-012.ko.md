---
knowledge_id: ONCHAIN-METRICS-012
title: Stablecoin Supply
subtitle: dry powder 서사, 결제 유동성, 그리고 gross issuance와 deployable demand를 분리해야 하는 이유
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 85 min
estimated_study: 200 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Liquidity
parent:
  - Onchain Metrics
prerequisites:
  - TOKEN-STANDARDS-004
related_topics:
  - Exchange Reserve
  - TVL
primary_sources:
  - REF-GN-SSR-2026-001
  - REF-LLAMA-STABLES-2026-001
tags:
  - stablecoin-supply
  - liquidity
---

# Stablecoin Supply
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-012

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-012
title: Stablecoin Supply
research_question: >
  분석가는 stablecoin supply를 liquidity와 risk-appetite signal로 어떻게
  사용해야 하며, 왜 gross stablecoin balance는 자동으로 crypto asset에 대한
  deployable demand와 동일해지지 않는가?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-004
parent: Onchain Metrics
previous: ONCHAIN-METRICS-011
next: ONCHAIN-METRICS-013
```

## 1. Learning Objectives

- stablecoin-supply metric을 정의할 수 있다.
- dry-powder narrative와 그 한계를 설명할 수 있다.
- issuance와 deployable risk appetite를 구분할 수 있다.

## 2. Executive Summary

Glassnode는 Stablecoin Supply Ratio(SSR)를 Bitcoin market cap과 aggregate stablecoin supply의 관계로 문서화하며, stablecoin issuance를 liquidity proxy로 사용한다.[^ref-gn-ssr]

DefiLlama 역시 체인과 발행자별 stablecoin supply를 추적하는 market-structure dataset을 유지한다.[^ref-llama-stables]

stablecoin supply가 중요한 이유는 다음을 시사할 수 있기 때문이다.

- settlement liquidity,
- fiat onboarding intensity,
- risk appetite,
- collateral availability.

## 3. Limits

- stablecoin이 유휴 상태로 남을 수 있다.
- 공급이 여러 체인에 분산될 수 있다.
- issuer action만으로도 즉각적인 시장 deploy 없이 supply가 변할 수 있다.

## 4. Institutional Thinking

- stablecoin supply는 liquidity backdrop metric이지, 임박한 현물 매수의 직접 증거가 아니다.

## 5. References

[^ref-gn-ssr]: Glassnode Docs, "Stablecoin Supply Ratio (SSR)" metric guide, accessed 2026-08-04, https://docs.glassnode.com/guides-and-tutorials/metric-guides/stablecoin-supply-ratio-ssr

[^ref-llama-stables]: DefiLlama Docs and dashboards, stablecoin methodology and chain/issuer supply tracking, accessed 2026-08-04, https://defillama.com/stablecoins

## 6. Cross References

- Previous: ONCHAIN-METRICS-011 — Realized Cap
- Next: ONCHAIN-METRICS-013 — TVL

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
