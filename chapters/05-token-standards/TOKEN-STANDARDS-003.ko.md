---
knowledge_id: TOKEN-STANDARDS-003
title: ERC-1155
subtitle: 멀티 토큰 contract, batch transfer, 그리고 fungibility-agnostic 자산 모델
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 105 min
estimated_study: 250 min
last_reviewed: 2026-08-04
domain:
  - Token Standards
  - Ethereum
  - Assets
parent:
  - Token Standards
prerequisites:
  - TOKEN-STANDARDS-001
  - TOKEN-STANDARDS-002
related_topics:
  - Stablecoins
  - Wrapped Assets
  - Utility Tokens
primary_sources:
  - REF-EIP-1155-2019-001
  - REF-ETH-ERC1155-2026-001
  - REF-OZ-ERC1155-2026-001
tags:
  - token-standards
  - erc1155
  - multi-token
---

# ERC-1155
> Token Standards  
> Research Unit: TOKEN-STANDARDS-003

---

## Research Brief

```yaml
knowledge_id: TOKEN-STANDARDS-003
title: ERC-1155
research_question: >
  ERC-1155는 ERC-20과 ERC-721을 넘어 자산 표현을 어떻게 일반화하며,
  multi-token 및 batch semantics가 어떤 이점을 제공하고, 대체가능 자산과
  대체불가능 자산이 하나의 contract 안에 공존할 때 어떤 분석상 주의점이
  생기는가?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-001
  - TOKEN-STANDARDS-002
parent: Token Standards
previous: TOKEN-STANDARDS-002
next: TOKEN-STANDARDS-004
related_topics:
  - Stablecoins
  - Wrapped Assets
  - Utility Tokens
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Standard Structure
  - Interface Mechanics
  - Asset Model
  - Security Considerations
  - Implementation Notes
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Full game-economy design
  - Royalty extensions
  - Crosschain asset mirrors
```

## 1. Learning Objectives

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- ERC-1155를 multi-token 표준으로 정의할 수 있다.
- 하나의 contract가 여러 token type을 어떻게 관리하는지 설명할 수 있다.
- batch transfer와 single-asset transfer를 구분할 수 있다.
- approval semantics가 ERC-20 allowance와 어떻게 다른지 설명할 수 있다.
- fungibility-agnostic contract 모델의 분석상 tradeoff를 식별할 수 있다.

---

## 2. Key Questions

1. ERC-20과 ERC-721 이후에 왜 ERC-1155가 도입되었는가?
2. 하나의 token ID가 독자적인 asset type을 나타낸다는 것은 무엇을 의미하는가?
3. batch operation은 왜 운영상 중요한가?
4. approval과 safe transfer는 어떻게 동작하는가?
5. 어떤 새로운 indexing 및 해석상의 문제가 나타나는가?

---

## 3. Executive Summary

ERC-1155는 하나의 배포된 contract 안에서 대체가능 자산, 대체불가능 자산, semi-fungible 자산을 함께 담을 수 있도록 여러 token type을 관리하는 standards-track 인터페이스다.[^ref-eip1155][^ref-eth-erc1155]

EIP는 이전 표준들이 token type이나 collection마다 별도 contract를 요구하는 경우가 많았고, 그 결과 bytecode 중복과 기능 제약이 발생했다고 설명한다.[^ref-eip1155]

ERC-1155는 자산 모델을 바꾼다. 하나의 대체가능 토큰마다 contract 하나, 하나의 NFT collection마다 contract 하나를 두는 대신, 하나의 contract가 여러 token ID를 정의할 수 있고, 각 ID는 자체 공급량과 metadata 특성을 가진 독립적인 asset type이 될 수 있다.[^ref-eip1155]

또한 이 표준은 batch operation을 도입해 여러 자산을 하나의 호출에서 전송하거나 조회할 수 있게 한다.[^ref-eth-erc1155]

그 결과 ERC-1155는 게임 inventory, 복합 collectible, 대체가능 포지션과 대체불가능 포지션이 섞인 시스템처럼 다양한 자산 클래스가 공존하는 환경에서 특히 유용한 보다 일반적인 asset container가 된다.

---

## 4. Standard Structure

### 4.1 What ERC-1155 Is

EIP-1155는 여러 token type을 관리하는 contract를 위한 표준 인터페이스를 정의하며, 하나의 contract에 fungible token, non-fungible token, semi-fungible token 같은 다양한 구성이 포함될 수 있다고 명시한다.[^ref-eip1155]

이것이 이전 표준과의 핵심적인 개념적 단절이다. ERC-1155는 단순히 "또 다른 토큰 종류"가 아니다. 하나의 contract address 아래 여러 자산 유형을 담는 더 일반적인 container model이다.

### 4.2 Why It Was Needed

EIP는 ERC-20과 ERC-721이 token type이나 collection마다 별도 contract 배포를 요구하는 경우가 많아 중복 bytecode와 기능상의 한계를 초래했다고 지적한다.[^ref-eip1155]

ERC-1155는 하나의 contract 내부 token ID를 서로 다른 asset class로 사용함으로써 이 문제를 해결한다.

---

## 5. Interface Mechanics

### 5.1 Token IDs as Asset Types

EIP-1155는 각 token ID가 자체 metadata, supply, 기타 속성을 가지는 구성 가능한 token type을 나타낼 수 있다고 설명한다.[^ref-eip1155]

즉 `_id` 인자는 단순한 serial number가 아니다. 트랜잭션이 어떤 asset type을 참조하는지 선택하는 selector다.

### 5.2 Batch Operations

ethereum.org는 ERC-1155의 핵심 기능으로 다음을 요약한다.

- batch transfer,
- batch balance query,
- operator approval,
- receive hook,
- NFT support,
- safe transfer rule.[^ref-eth-erc1155]

batch model이 중요한 이유는 서로 연관된 여러 자산 이동을 다수의 트랜잭션으로 쪼개지 않고 한 번의 호출에서 처리할 수 있기 때문이다.

### 5.3 Events

ERC-1155는 다음을 정의한다.

- `TransferSingle`
- `TransferBatch`
- `ApprovalForAll`
- `URI`[^ref-eip1155]

EIP는 mint 시 `_from`에 0 주소를 사용하고, burn 시 `_to`에 0 주소를 사용한다고 명시한다.[^ref-eip1155]

### 5.4 Safe Transfers

EIP는 safe transfer 동작을 요구하며, 수신자가 0 주소이거나, 잔액이 부족하거나, 기타 오류가 있는 경우 전송이 revert되어야 한다고 규정한다.[^ref-eip1155]

이 덕분에 전송 semantics는 일반 ERC-20보다 수신 가능성(delivery awareness)을 더 명시적으로 포함한다.

### 5.5 Approval Model

ERC-1155는 amount-based allowance 대신 operator-style approval을 사용한다. ethereum.org는 특정 수량을 승인하는 대신, 보유자가 `setApprovalForAll`을 통해 operator를 승인 또는 해제한다고 설명한다.[^ref-eth-erc1155]

즉 approval 로직은 구조적으로 ERC-20보다 ERC-721에 더 가깝다.

---

## 6. Asset Model

### 6.1 One Contract, Many Asset Classes

contract `C`가 token ID 집합 `I`를 지원한다고 하자.

각 `i in I`에 대해 다음을 정의한다.

- 공급량 `S_i`
- metadata reference `M_i`
- address `a`에 대한 balance 함수 `b_i(a)`

이때 contract는 하나의 자산이 아니라 다음과 같은 자산 가족이다.

`AssetFamily(C) = {i_1, i_2, ..., i_n}`

이 자산들은 코드를 공유하고, 종종 operator permission도 공유한다.

### 6.2 Fungibility-Agnostic Design

어떤 token ID가 크고 분할 가능한 공급량을 가지면 대체가능 자산처럼 동작할 수 있다.

token ID의 공급량이 1이면, ethereum.org가 설명하듯 NFT처럼 취급할 수 있다.[^ref-eth-erc1155]

어떤 token ID가 처음에는 fungible하게 시작했다가 점차 더 고유한 claim 상태로 좁아진다면, application 관점에서는 semi-fungible 모델처럼 작동할 수 있다.

### 6.3 Batch State Transition

ID 목록 `I = [i1, i2, ..., in]`와 값 목록 `V = [v1, v2, ..., vn]`에 대해 `x`에서 `y`로 batch transfer가 일어난다면, 각 `k`에 대해:

- `b'ik(x) = bik(x) - vk`
- `b'ik(y) = bik(y) + vk`

batch operation은 본질적으로 여러 single-asset 상태 전이를 하나의 호출에서 벡터화해 처리하는 방식이다.

---

## 7. Security Considerations

### 7.1 Broad Operator Power

approval이 대개 amount-specific이 아니라 operator-wide이기 때문에, compromise되었거나 악의적인 operator는 해당 approval 범위 안의 여러 token ID 전체에 영향을 줄 수 있다.

### 7.2 Mixed-Asset Interpretation Risk

fungible 자산과 non-fungible 자산이 하나의 contract 안에 섞여 있으면, 단순한 analytics는 contract-level activity를 잘못 읽을 수 있다. transfer 급증이 실제로는 다음 중 하나일 수 있다.

- 여러 token type이 동시에 이동한 경우,
- 하나의 batch settlement,
- 이질적인 자산이 포함된 하나의 application workflow.

contract-level aggregate만으로는 대개 너무 거칠다.

### 7.3 Metadata Dependency

`URI` 이벤트와 metadata 체계는 discoverability를 돕지만, metadata의 영속성과 무결성은 여전히 application이 참조 자원을 어떻게 관리하는지에 달려 있다.

---

## 8. Implementation Notes

OpenZeppelin은 ERC-1155를 이전 표준의 아이디어를 차용한 fungibility-agnostic, gas-efficient 토큰 contract 모델로 설명한다.[^ref-oz-erc1155]

이 표현은 운영상의 가치 제안을 잘 요약한다.

- 배포 수 감소,
- 더 일반적인 자산 모델링,
- 효율적인 그룹 연산.

하지만 효율성은 실제 application 설계에 달려 있다. contract가 ID별 복잡한 규칙, access control layer, metadata indirection을 많이 도입하면 전체 운영 단순성은 여전히 제한될 수 있다.

---

## 9. On-Chain Implications

### 9.1 Indexing Complexity

ERC-1155 analytics는 최소한 다음의 복합 키를 요구한다.

- contract address,
- token ID,
- holder address

이벤트 소비자는 single transfer log와 batch transfer log를 모두 정확히 파싱해야 한다.

### 9.2 Gas and Workflow Efficiency

batch operation은 특히 inventory 스타일 시스템에서, 관련된 상태 변경을 처리하는 데 필요한 트랜잭션 수와 event stream 수를 줄일 수 있다.

### 9.3 Contract-Centric Risk

많은 asset type이 하나의 contract 안에 들어갈 수 있기 때문에, bug, pause, proxy upgrade, compromise된 admin path 하나가 one-contract-per-asset 구조보다 훨씬 넓은 자산 surface에 영향을 줄 수 있다.

---

## 10. Institutional Thinking

- ERC-1155 contract는 단일 자산이 아니라 multi-asset system으로 다뤄야 한다.
- dashboard와 risk view는 contract level만이 아니라 `contract + tokenId` level에서 구축해야 한다.
- approval surface가 여러 asset class를 포괄할 수 있으므로 operator approval 모니터링이 중요하다.
- shared-code 효율성은 shared failure domain도 키울 수 있다. 배포 수가 적다고 governance 또는 contract risk가 저절로 줄어드는 것은 아니다.
- 프로젝트가 하나의 환경 안에서 fungible, semi-fungible, collectible 자산을 함께 설명할 때, ERC-1155는 그 제품 서사 아래의 기술 표현 계층을 설명해 주는 경우가 많다.

---

## 11. Evidence Classification

| Claim | Classification | Basis |
|---|---|---|
| ERC-1155 manages multiple token types in one contract | Directly specified | EIP-1155 summary and abstract |
| Each token ID can define its own type and attributes | Directly specified | EIP-1155 motivation |
| Batch transfer and batch balance are key features | Secondary explanatory summary | ethereum.org documentation |
| ERC-1155 uses operator approvals rather than ERC-20-style numeric allowances | Directly specified and summarized | EIP-1155 plus ethereum.org |
| ERC-1155 is fungibility-agnostic and gas-efficient in design intent | Implementation summary | OpenZeppelin documentation |
| Contract-level aggregation can mislead analysis | Analytical inference | Multi-asset architecture |

---

## 12. References

[^ref-eip1155]: Ethereum Improvement Proposals, "ERC-1155: Multi Token Standard," created 2019-06-17, accessed 2026-08-04, https://eips.ethereum.org/EIPS/eip-1155

[^ref-eth-erc1155]: ethereum.org, "ERC-1155 Multi-Token Standard," official documentation, last updated 2026-07-30, accessed 2026-08-04, https://ethereum.org/developers/docs/standards/tokens/erc-1155/

[^ref-oz-erc1155]: OpenZeppelin Docs, "ERC-1155," OpenZeppelin Contracts 5.x documentation, accessed 2026-08-04, https://docs.openzeppelin.com/contracts/5.x/erc1155

---

## 13. Cross References

### Previous

- TOKEN-STANDARDS-002 — ERC-721

### Next

- TOKEN-STANDARDS-004 — Stablecoins

---

## Review Status

### Technical Review

Passed.

- multi-token architecture와 token-ID semantics를 명확히 분리했다.
- batch mechanics를 벡터화된 상태 전이로 설명했다.
- approval semantics를 ERC-20 allowance와 구분했다.

### Evidence Review

Passed.

- multi-token 및 per-ID 주장은 EIP-1155에 근거한다.
- batch 기능 요약은 ethereum.org에 연결했다.
- 구현 관련 framing은 OpenZeppelin 문서에 귀속했고 프로토콜 규칙으로 과장하지 않았다.

### Editorial Review

Passed.

- 문서 구조는 기존 모듈과 정렬되어 있다.
- fungible, non-fungible, semi-fungible 용어는 정밀하게 유지했다.
- 수학 표기는 개념 모델 수준에 한정했다.

### Adversarial Review

Passed.

- 이 문서는 모든 ERC-1155 배포가 실무적으로 ERC-20이나 ERC-721보다 단순하다고 암시하지 않는다.
- 모든 token ID를 하나의 자산으로 평탄화하지 않는다.
- URI 지원만으로 metadata permanence를 가정하지 않는다.

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
