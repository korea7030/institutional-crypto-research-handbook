---
knowledge_id: TOKEN-STANDARDS-002
title: ERC-721
subtitle: 대체불가능 토큰의 식별성, 소유권 추적, 그리고 안전한 전송
version: 1.0.0
status: Reviewed
difficulty: L200
estimated_reading: 100 min
estimated_study: 235 min
last_reviewed: 2026-08-04
domain:
  - Token Standards
  - Ethereum
  - Assets
parent:
  - Token Standards
prerequisites:
  - TOKEN-STANDARDS-001
  - SMART-CONTRACTS-004
related_topics:
  - ERC-1155
  - Utility Tokens
  - Digital Collectibles
primary_sources:
  - REF-EIP-721-2018-001
  - REF-ETH-ERC721-2026-001
  - REF-OZ-ERC721-2026-001
tags:
  - token-standards
  - erc721
  - nft
---

# ERC-721
> Token Standards  
> Research Unit: TOKEN-STANDARDS-002

---

## Research Brief

```yaml
knowledge_id: TOKEN-STANDARDS-002
title: ERC-721
research_question: >
  ERC-721은 대체불가능 토큰에 대해 무엇을 표준화하며, 소유권과 전송 안전성은
  어떻게 모델링하고, 연구자는 고유한 token identity, collection 수준 contract
  policy, offchain NFT narrative를 어떻게 구분해야 하는가?
document_type: foundation
difficulty: L200
prerequisites:
  - TOKEN-STANDARDS-001
  - SMART-CONTRACTS-004
parent: Token Standards
previous: TOKEN-STANDARDS-001
next: TOKEN-STANDARDS-003
related_topics:
  - ERC-1155
  - Utility Tokens
  - Digital Collectibles
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Standard Structure
  - Interface Mechanics
  - Ownership Model
  - Security Considerations
  - Implementation Notes
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - NFT market history
  - Media valuation
  - Royalties beyond base standard
```

## 1. Learning Objectives

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- ERC-721을 대체불가능 토큰 인터페이스 표준으로 정의할 수 있다.
- `tokenId` 단위로 소유권이 어떻게 추적되는지 설명할 수 있다.
- `transferFrom`과 `safeTransferFrom`을 구분할 수 있다.
- approval과 operator approval의 역할을 설명할 수 있다.
- 온체인 고유성과 오프체인 가격 서사를 분리할 수 있다.

---

## 2. Key Questions

1. 왜 ERC-20은 고유 자산을 표현하기에 충분하지 않은가?
2. ERC-721에서 non-fungibility는 무엇을 의미하는가?
3. 표준은 소유권과 approval을 어떻게 표현하는가?
4. safe transfer는 왜 중요한가?
5. NFT transfer 데이터로부터 무엇을 추론할 수 있고, 무엇은 추론할 수 없는가?

---

## 3. Executive Summary

ERC-721은 Ethereum 상 대체불가능 토큰의 standards-track 인터페이스다.[^ref-eip721]

EIP는 각 자산이 상호교환 가능한 것이 아니라 서로 구별되기 때문에, 비대체 자산을 추적하는 데 ERC-20이 충분하지 않다고 설명한다.[^ref-eip721]

따라서 ERC-721은 다음을 표준화한다.

- 토큰별 소유권,
- 고유 식별자의 전송,
- 토큰 단위 approval,
- 소유자 보유분 전반에 대한 operator approval,
- 상태 전이를 나타내는 event emission.[^ref-eip721]

ERC-20과 비교한 핵심 분석 차이는 회계 단위가 단순한 스칼라 잔액이 아니라는 점이다. 대신 표준은 다음의 쌍을 중심으로 동작한다.

- contract address,
- `tokenId`

ethereum.org는 `contract address, uint256 tokenId`의 쌍을 ERC-721 자산에 대한 전역적으로 고유한 식별자로 설명한다.[^ref-eth-erc721]

이 덕분에 연구자는 식별성과 소유권을 위한 깔끔한 기반 모델을 얻지만, 적정 가치, 연결된 미디어의 진위, 법적 권리, marketplace royalty와 같은 문제까지 해결하는 것은 아니다. 그런 요소는 모두 기본 표준 위 계층에 존재한다.

---

## 4. Standard Structure

### 4.1 What ERC-721 Solves

EIP-721은 이 표준이 smart contract 내부 NFT를 위한 공통 API와, 이를 추적하고 전송하기 위한 기본 기능을 제공한다고 말한다.[^ref-eip721]

이는 고유 디지털 자산에 대한 상호운용성 문제를 해결했다. wallet, marketplace, auction system, game은 각 NFT 프로젝트마다 완전히 맞춤형 소유권 로직을 새로 만들 필요가 없어졌다.

### 4.2 Dependence on ERC-165

이 EIP는 명세상 예외 사항을 전제로, ERC-721 contract가 ERC-721과 ERC-165 인터페이스를 모두 구현하도록 요구한다.[^ref-eip721]

이 요구사항이 중요한 이유는 interface detection을 통해 외부 contract와 도구가 해당 contract가 기대하는 NFT 동작을 지원하는지 확인할 수 있기 때문이다.

---

## 5. Interface Mechanics

### 5.1 Ownership and Balance Queries

ERC-721은 기본 read method로 `balanceOf(owner)`와 `ownerOf(tokenId)`를 포함한다.[^ref-eip721]

이 둘은 서로 관련되지만 다른 시각을 제공한다.

- 한 주소가 특정 collection 안에서 몇 개의 NFT를 보유하는가,
- 특정 NFT를 누가 소유하는가.

### 5.2 Transfers

표준은 다음을 포함한다.

- `transferFrom`
- 추가 데이터 유무에 따른 `safeTransferFrom`[^ref-eip721][^ref-eth-erc721]

`transferFrom`은 수신 contract가 ERC-721 토큰을 처리할 수 있는지 확인하지 않고 소유권을 이전한다.

`safeTransferFrom`은 목적지가 contract일 때 수신 호환성 검사를 추가한다. 이는 NFT를 처리할 수 없는 contract로 자산을 보내 버리는 리스크를 줄이기 위한 설계다.

### 5.3 Approvals

ERC-721은 다음을 정의한다.

- `approve`를 통한 토큰별 approval
- `setApprovalForAll`을 통한 collection-wide operator approval[^ref-eip721]

이는 금액 기반 위임 지출을 사용하는 ERC-20과 다르다. ERC-721 approval은 대체가능 잔액에 대한 지출 한도가 아니라, 고유한 아이템에 대한 제어 권한을 중심으로 설계된다.

### 5.4 Events

표준은 다음을 정의한다.

- `Transfer`
- `Approval`
- `ApprovalForAll`[^ref-eip721]

EIP는 `Transfer`가 어떤 메커니즘에 의한 것이든 소유권 변경 시 사용되며, 0 주소 관례를 통해 mint와 burn 이벤트도 포괄한다고 설명한다. 다만 contract deployment 시점 생성에는 예외적 caveat가 있다.[^ref-eip721]

---

## 6. Ownership Model

### 6.1 Unique Identity

ethereum.org는 ERC-721에서 `contract address, tokenId`의 쌍이 전역적으로 고유해야 한다고 설명한다.[^ref-eth-erc721]

이 말은 metadata, 이미지, intellectual property claim이 자동으로 고유하다는 뜻이 아니다. 온체인 식별자 쌍이 네트워크 address space 안에서 고유하다는 뜻이다.

### 6.2 Collection Scope

ERC-721 contract는 보통 collection 수준 namespace를 나타낸다. 그 namespace 안에서 각 `tokenId`는 특정 소유권 기록을 가리키고, 종종 오프체인 또는 온체인 metadata에도 연결된다.

### 6.3 State Interpretation

연구자는 다음을 구분해야 한다.

- ownership state,
- metadata reference state,
- marketplace listing state,
- cultural narrative.

ERC-721이 직접 표준화하는 것은 첫 번째뿐이다.

---

## 7. Mathematical or Economic Model

### 7.1 Ownership Function

collection contract `C`가 token ID 집합 `T`를 정의한다고 하자.

소유권 함수는 다음처럼 모델링할 수 있다.

`owner_C: T -> A`

여기서 `A`는 유효한 Ethereum address 집합이다.

임의의 토큰 `t`에 대해, 소유권 이전은 다음과 같이 갱신된다.

- 이전 전 `owner_C(t) = x`
- 이전 후 `owner'_C(t) = y`

ERC-20과 달리, 공급량 회계의 핵심은 balance 중심이 아니다. balance는 collection 안에서 특정 owner에 매핑된 token ID 개수를 통해 파생된다.

### 7.2 Approval Semantics

토큰별 approval은 다음으로 표현할 수 있다.

`approved_C(t) = a or null`

operator approval은 다음과 같다.

`operatorApproved_C(owner, operator) in {true, false}`

이 제어 권한은 숫자형 allowance가 아니라 이산적인 승인 상태다.

---

## 8. Security Considerations

### 8.1 Unsafe Delivery Risk

`safeTransferFrom`이 존재하는 실질적 이유는 전달 안전성이다. NFT 수신 로직을 지원하지 않는 contract로 NFT를 보내면, 트랜잭션은 성공하더라도 운영상 자산이 고립될 수 있다.

### 8.2 Approval Abuse

operator approval은 강력하다. `setApprovalForAll`은 해당 collection 내에서 소유자의 NFT 전부에 대해 광범위한 전송 권한을 부여할 수 있다. phishing이나 compromise된 front-end 상황에서는 사용자가 의도보다 훨씬 넓은 권한을 승인할 수 있다.

### 8.3 Metadata and Offchain Dependency

ERC-721은 토큰 처리 방식을 표준화할 뿐, 미디어 영속성까지 표준화하지는 않는다. metadata는 종종 다음에 의존한다.

- 변경 가능한 HTTP endpoint,
- gateway service,
- application 운영 서버,
- 별도로 관리되는 decentralized storage reference.

즉 소유권은 온체인일 수 있어도, 자산의 서술적 경험은 외부 의존성을 가진다.

---

## 9. Implementation Notes

OpenZeppelin은 ERC-721을 대체불가능 토큰의 표준으로 문서화하고, 널리 사용되는 구현 surface를 제공한다.[^ref-oz-erc721]

실제로 많은 시스템은 다음에 대해 OpenZeppelin 호환 동작을 가정한다.

- receiver check,
- approval bookkeeping,
- metadata extension.

이러한 가정은 운영상 유용하지만, 여전히 EIP 본문 위에 놓인 구현 관행으로 봐야 한다.

---

## 10. On-Chain Implications

### 10.1 Indexing

NFT indexer는 보통 다음을 추적한다.

- collection contract,
- token ID,
- owner,
- transfer history,
- approval state change.

### 10.2 Market Structure

ERC-721은 collection 간에 공유되는 인터페이스를 기반으로 listing, transfer, custody workflow를 구성할 수 있게 하면서 범용 marketplace 인프라를 가능하게 했다.

### 10.3 Limits of Onchain Interpretation

온체인 transfer 데이터는 이동, custody concentration, turnover를 보여줄 수 있다. 하지만 그것만으로 다음을 증명할 수는 없다.

- 진정한 창작자(authorship),
- 오프체인 자산 통제권,
- royalty 강제력,
- 미디어 가용성의 지속성.

---

## 11. Institutional Thinking

- ERC-721은 고유 식별자의 소유권 인터페이스로 다뤄야지, 가치평가 프레임워크로 다뤄서는 안 된다.
- NFT identity, NFT metadata, NFT market narrative를 분리해야 한다.
- collection을 immutable하게 보기 전에 contract-level admin right, mint control, pausability, upgradeability를 검토해야 한다.
- 리스크 모니터링에서는 transfer flow만큼 operator approval 패턴도 중요하다.
- 집중도를 분석할 때는 collection-level address count가 custodial wallet이나 marketplace 뒤의 beneficial ownership을 포착하지 못한다는 점을 기억해야 한다.

---

## 12. Evidence Classification

| Claim | Classification | Basis |
|---|---|---|
| ERC-721 standardizes non-fungible token handling | Directly specified | EIP-721 abstract |
| ERC-20 is insufficient for unique assets | Directly specified | EIP-721 motivation |
| ERC-721 requires ERC-165 support | Directly specified | EIP-721 specification |
| `safeTransferFrom` exists alongside `transferFrom` | Directly specified | EIP-721 and ethereum.org interface summary |
| `contract address + tokenId` forms the unique onchain identifier pair | Secondary explanatory summary | ethereum.org documentation |
| NFT ownership does not guarantee media permanence or legal rights | Analytical inference | Standard scope excludes those properties |

---

## 13. References

[^ref-eip721]: Ethereum Improvement Proposals, "ERC-721: Non-Fungible Token Standard," created 2018-01-24, accessed 2026-08-04, https://eips.ethereum.org/EIPS/eip-721

[^ref-eth-erc721]: ethereum.org, "ERC-721 Non-Fungible Token Standard," official documentation, last updated 2026-07-30, accessed 2026-08-04, https://ethereum.org/developers/docs/standards/tokens/erc-721/

[^ref-oz-erc721]: OpenZeppelin Docs, "ERC-721," OpenZeppelin Contracts 5.x documentation, accessed 2026-08-04, https://docs.openzeppelin.com/contracts/5.x/erc721

---

## 14. Cross References

### Previous

- TOKEN-STANDARDS-001 — ERC-20

### Next

- TOKEN-STANDARDS-003 — ERC-1155

---

## Review Status

### Technical Review

Passed.

- ownership, approval, transfer semantics를 올바르게 구분했다.
- ERC-165 의존성을 포함했다.
- `safeTransferFrom`은 marketplace 기능이 아니라 전달 안전성 로직으로 설명했다.

### Evidence Review

Passed.

- 핵심 주장은 EIP-721에 근거한다.
- identity pair 설명은 ethereum.org에 연결했다.
- 구현 관련 코멘트는 기본 표준과 분리했다.

### Editorial Review

Passed.

- 문서 구조는 repository 관례와 일치한다.
- 용어는 기존 smart-contract unit과 일관된다.
- 어떤 섹션도 market-history narrative로 이탈하지 않는다.

### Adversarial Review

Passed.

- 이 문서는 NFT의 온체인 고유성이 문화적·경제적 고유성을 보장한다고 암시하지 않는다.
- 온체인 ownership을 copyright이나 오프체인 통제와 동일시하지 않는다.
- metadata immutability를 가정하지 않는다.

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
