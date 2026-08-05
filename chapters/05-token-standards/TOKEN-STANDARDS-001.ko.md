---
knowledge_id: TOKEN-STANDARDS-001
title: ERC-20
subtitle: 대체가능 토큰 인터페이스, allowance 의미론, 그리고 온체인 자산 표현의 기본 표준
version: 1.0.0
status: Reviewed
difficulty: L200
estimated_reading: 95 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - Token Standards
  - Ethereum
  - Assets
parent:
  - Token Standards
prerequisites:
  - SMART-CONTRACTS-001
  - SMART-CONTRACTS-003
related_topics:
  - ERC-721
  - ERC-1155
  - Stablecoins
primary_sources:
  - REF-EIP-20-2015-001
  - REF-ETH-ERC20-2026-001
  - REF-OZ-ERC20-2026-001
tags:
  - token-standards
  - erc20
  - fungible-tokens
---

# ERC-20
> Token Standards  
> Research Unit: TOKEN-STANDARDS-001

---

## Research Brief

```yaml
knowledge_id: TOKEN-STANDARDS-001
title: ERC-20
research_question: >
  ERC-20은 어떤 문제를 해결하며, 정확히 어떤 interface와 event model을
  표준화하고, allowance 기반의 대체가능 토큰 모델을 Ethereum application에서
  사용할 때 어떤 운영상·보안상 결과가 발생하는가?
document_type: foundation
difficulty: L200
prerequisites:
  - SMART-CONTRACTS-001
  - SMART-CONTRACTS-003
parent: Token Standards
previous:
next: TOKEN-STANDARDS-002
related_topics:
  - ERC-721
  - ERC-1155
  - Stablecoins
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Standard Structure
  - Interface Mechanics
  - Operational Model
  - Security Considerations
  - Implementation Notes
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Full Solidity implementation tutorial
  - Stablecoin-specific reserve design
  - Token valuation
```

## 1. Learning Objectives

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- ERC-20을 특정 자산이 아니라 인터페이스 표준으로 정의할 수 있다.
- `balanceOf`, `transfer`, `approve`, `allowance`, `transferFrom`의 역할을 설명할 수 있다.
- 직접 토큰 전송과 위임 지출(delegated spending)을 구분할 수 있다.
- ERC-20 상호운용성이 함수뿐 아니라 이벤트에도 의존한다는 점을 설명할 수 있다.
- allowance 처리와 토큰 통합에서 나타나는 주요 리스크를 식별할 수 있다.

---

## 2. Key Questions

1. ERC-20은 정확히 무엇을 표준화하는가?
2. Ethereum에서 왜 대체가능 토큰 인터페이스가 필요했는가?
3. allowance 모델은 어떻게 동작하는가?
4. wallet, exchange, DeFi protocol은 ERC-20 동작에 대해 무엇을 전제하는가?
5. 실제 통합 실패는 주로 어디에서 발생하는가?

---

## 3. Executive Summary

ERC-20은 대체가능 토큰을 위한 공통 API를 정의하는 standards-track Ethereum 토큰 인터페이스다.[^ref-eip20]

이 표준의 목적은 wallet, exchange, smart contract, 기타 application이 서로 다른 토큰 contract와 동일한 기본 함수 및 event surface를 통해 상호작용할 수 있게 만드는 데 있다.[^ref-eip20]

최소 기준으로 ERC-20은 다음을 정의한다.

- 총 공급량 조회,
- 계정별 잔액 조회,
- 직접 전송,
- allowance를 통한 위임 전송,
- 표준화된 `Transfer` 및 `Approval` 이벤트.[^ref-eip20]

경제적 관점에서 핵심은 단순하다. ERC-20은 application별 smart contract 내부의 잔액 테이블을 재사용 가능한 자산 인터페이스로 바꾼다. 이 추상화 덕분에 stablecoin, governance token, utility token, vault share, 다양한 DeFi 포지션 wrapper가 Ethereum 전반에서 조합 가능해졌다.

동시에 운영상 주의점도 명확하다. ERC-20이 표준화하는 것은 인터페이스이지, 보편적인 business logic이 아니다. 전송 제한, fee-on-transfer 메커니즘, rebase 동작, mint 권한, pausable 기능, blacklist 정책은 모두 표준 바깥 상위 계층에 위치한다. 따라서 연구자는 다음을 분리해서 봐야 한다.

- ERC-20의 consensus-neutral 인터페이스 사실,
- 발행 주체의 정책과 tokenomics,
- application의 가정,
- 구현별 동작 차이.

---

## 4. Standard Structure

### 4.1 What ERC-20 Is

EIP-20은 ERC-20을 smart contract 내부 토큰을 위한 표준 API로 설명한다.[^ref-eip20]

이 표준이 Ethereum 내부에 새로운 네이티브 프로토콜 객체를 만드는 것은 아니다. ERC-20 토큰은 여전히 contract code가 관리하는 일반적인 contract state다. 표준은 단지 외부 주체가 그 state와 어떻게 상호작용해야 하는지를 명시할 뿐이다.

### 4.2 Why It Matters

EIP는 표준 인터페이스가 있으면 Ethereum 상의 토큰을 wallet, decentralized exchange 등 다른 application이 재사용할 수 있다고 말한다.[^ref-eip20]

이 지점이 기관 분석 관점의 핵심 통찰이다. ERC-20은 인프라 제공자에게 자산마다 별도 adapter를 만드는 대신 하나의 정규화된 interaction surface를 제공함으로써 통합 비용을 낮췄다.

### 4.3 Fungibility

ERC-20은 대체가능 토큰(fungible token) 모델이다. 대체가능하다는 것은 프로토콜 인터페이스 수준에서 각 단위가 서로 교환 가능하도록 설계되었다는 뜻이다. Alice가 100 단위를 가지고 Bob도 100 단위를 가진다면, 표준은 전송과 회계 처리에서 각 단위를 동일한 것으로 취급한다.

이는 각 자산 식별자를 개별적으로 추적하는 NFT 표준과 다르다.

---

## 5. Interface Mechanics

### 5.1 Core Read Functions

표준은 총 발행량과 계정별 잔액을 조회하는 view 함수로 `totalSupply()`와 `balanceOf(address)`를 정의한다.[^ref-eip20]

이 함수들은 현재 잔액이 필요한 wallet, explorer, protocol에게 기본 회계 계층을 제공한다.

### 5.2 Direct Transfer

`transfer(address,uint256)`는 호출자에서 수신자에게 토큰을 이동시키며, 반드시 `Transfer` 이벤트를 발생시켜야 한다.[^ref-eip20]

EIP-20은 또한 0값 전송도 일반 전송처럼 처리되어야 하며 이벤트를 발생시켜야 한다고 명시한다.[^ref-eip20]

이 요구사항이 중요한 이유는 많은 오프체인 시스템이 internal storage를 직접 검사하기보다 로그를 통해 토큰 이동을 재구성하기 때문이다.

### 5.3 Delegated Spending

위임 지출 모델은 세 함수로 구성된다.

- `approve(spender, value)`
- `allowance(owner, spender)`
- `transferFrom(from, to, value)`[^ref-eip20]

흐름은 다음과 같다.

1. 토큰 보유자가 spender에게 지출 한도를 부여한다.
2. 이후 spender 또는 spender가 제어하는 contract가 `transferFrom`을 사용한다.
3. 토큰 contract가 지출이 승인되었는지 확인한다.

이 모델은 DEX routing, vault deposit, lending collateral 이동, subscription 유사 토큰 인출 구조의 기반이 되었다.

### 5.4 Events

ERC-20은 두 개의 핵심 이벤트를 정의한다.

- `Transfer`
- `Approval`[^ref-eip20]

EIP는 토큰 생성 시 `from = 0x0`인 `Transfer`를 emit해야 한다고 명시한다.[^ref-eip20]

즉 mint 이벤트도 일반 전송과 동일한 로그 어휘 안에서 표현되므로, indexer 로직이 단순해진다.

---

## 6. Operational Model

### 6.1 State Model

개념적으로 최소 ERC-20 contract는 다음을 추적한다.

- total supply 변수,
- address에서 balance로의 매핑,
- allowance를 위한 중첩 매핑.

표준이 특정 storage layout을 강제하지는 않지만, 거의 모든 구현은 이러한 회계 객체로 환원된다.

### 6.2 Allowance as a Capability

allowance는 위임된 지출 capability이지, 그 자체가 전송은 아니다. allowance를 부여해도 자금은 이동하지 않는다. spender가 그 권한을 실제로 행사할 때만 한도 내에서 미래의 이동이 허용된다.

이 구분은 analytics에서 중요하다. Approval activity와 transfer activity는 서로 다른 이벤트 클래스이며, 서로 다른 사용자 의도를 나타낸다.

### 6.3 Interface vs Asset Behavior

토큰은 ERC-20을 준수하면서도 다음과 같은 추가 business rule을 적용할 수 있다.

- 특권 역할에 의한 mint,
- pause 상태에서 전송 비활성화,
- blacklist,
- 수수료 차감,
- rebase 공급 로직.

따라서 "ERC-20 token"이라는 표현은 중립적이거나 단순한 화폐 수단을 뜻하지 않는다. 그것이 의미하는 것은 contract가 표준 인터페이스 surface를 노출한다는 사실뿐이다.

---

## 7. Mathematical or Economic Model

### 7.1 Fungible Balance Accounting

계정 집합 `A`, 잔액 `b(a)`, 총 공급량 `S`에 대해:

`S = sum(b(a))`

이는 mint와 burn에 따른 상태 전이를 반영하여 모든 토큰 보유 주소의 잔액 합으로 표현된다.

주소 `x`에서 `y`로 금액 `v`를 직접 전송하면 공급량은 보존된다.

- `b'(x) = b(x) - v`
- `b'(y) = b(y) + v`
- `S' = S`

mint는 공급량을 증가시킨다.

- `b'(y) = b(y) + v`
- `S' = S + v`

burn은 공급량을 감소시킨다.

- `b'(x) = b(x) - v`
- `S' = S - v`

ERC-20 표준 자체는 mint 또는 burn 함수를 정의하지 않지만, 이벤트 가이드는 0 주소에서 오는 `Transfer`를 통해 토큰 생성이 표현된다는 점을 명시적으로 인정한다.[^ref-eip20]

### 7.2 Allowance Constraint

owner `o`, spender `s`, 승인 한도 `L(o,s)`에 대해, 값 `v`의 유효한 위임 전송은 다음을 만족해야 한다.

`v <= L(o,s)`

그리고 owner 잔액도 충분해야 한다.

많은 구현에서는 `transferFrom`이 성공한 뒤:

`L'(o,s) = L(o,s) - v`

가 되며, contract가 의도적으로 무한 승인 semantics를 별도로 구현한 경우는 예외다.

---

## 8. Security Considerations

### 8.1 Allowance Race Condition

EIP-20은 allowance를 0이 아닌 값에서 다른 0이 아닌 값으로 변경할 때 알려진 리스크가 있음을 명시적으로 언급하며, 사용자 인터페이스는 새로운 값을 설정하기 전에 먼저 allowance를 0으로 설정할 것을 권고한다.[^ref-eip20]

이 경고는 인터페이스 수준의 주의사항이지, 필수 contract rule은 아니다. 표준은 하위 호환성을 유지했고, 완화 책임의 상당 부분을 wallet과 application에 남겨두었다.[^ref-eip20]

### 8.2 Token Reception Assumptions

ethereum.org는 잘 알려진 문제 하나를 지적한다. 일반 ERC-20 전송은 수신 contract에 대해 필수적인 notification hook을 제공하지 않는다.[^ref-eth-erc20]

즉, 수신 application이 callback 기반 회계 처리를 기대하더라도 contract는 아무 수신 로직 실행 없이 ERC-20 토큰을 받을 수 있고, 이로 인해 통합 리스크가 생긴다.

### 8.3 Non-Standard Implementations

실제 배포 환경에서는 많은 토큰이 이상화된 가정에서 벗어난다.

- 예상과 달리 `bool`을 반환하지 않는 경우,
- transfer fee를 부과하는 경우,
- 잔액을 rebase하는 경우,
- mint 및 freeze 권한이 중앙화된 경우가 있다.

이러한 동작은 토큰이 대체로 ERC-20 interface를 제공하더라도 application 차원에서 매우 중요할 수 있다. 따라서 분석가는 표준 적합성 계층과 발행자 로직 계층을 함께 봐야 한다.

---

## 9. Implementation Notes

OpenZeppelin은 ERC-20을 대체가능 토큰의 표준 인터페이스로 문서화하고, 널리 사용되는 구현 기준선을 제공한다.[^ref-oz-erc20]

이 점이 중요한 이유는 생태계의 실제 동작이 추상적 EIP 본문보다 reference implementation 주변으로 수렴하는 경우가 많기 때문이다. 실제로 많은 통합 주체는 다음에 대해 OpenZeppelin에 가까운 semantics를 암묵적으로 가정한다.

- revert 동작,
- allowance 업데이트,
- metadata helper,
- mint/burn 확장.

연구자는 이러한 가정을 ERC-20 consensus rule이 아니라 구현 관행으로 취급해야 한다.

---

## 10. On-Chain Implications

### 10.1 Indexing

대부분의 토큰 analytics 파이프라인은 `Transfer`와 `Approval` 로그를 1차 관측 surface로 사용한다. 이는 효율적이지만, 토큰이 특이한 로직이나 proxy upgrade, 비표준 부작용을 가진 경우에는 불완전하다.

### 10.2 Composability

ERC-20은 토큰 잔액을 application 간에 이식 가능하게 만들었다. DEX router, lending market, bridge, vault는 어느 정도 표준을 따르는 토큰이라면 훨씬 낮은 한계 엔지니어링 비용으로 통합할 수 있다.

### 10.3 Policy Layer Above the Standard

연구자는 다음을 분리해야 한다.

- 표준 인터페이스 준수 여부,
- contract ownership 및 privileged role,
- 토큰 발행 정책,
- 외부의 법적 주장이나 reserve 주장.

예를 들어 stablecoin은 ERC-20을 사용하면서도 오프체인 reserve attestation과 issuer 재량에 의존할 수 있다. ERC-20 계층만으로는 그런 외부 주장을 검증할 수 없다.

---

## 11. Institutional Thinking

- ERC-20은 자산 품질 인증서가 아니라 상호운용성 문법으로 다뤄야 한다.
- 전송 가능성과 trustlessness를 구분해야 한다. 많은 ERC-20 자산은 여전히 중앙 관리된다.
- 토큰 잔액과 법적 청구권을 구분해야 한다. contract는 단위를 표현할 수는 있어도, 오프체인 권리의 집행 가능성을 자동으로 보장하지는 않는다.
- 실사에서는 admin power, mint 권한, pause control, blacklist 로직, upgradeability, 외부 시스템 의존성을 점검해야 한다.
- analytics에서는 approval과 실제 flow를 구분해야 한다. approval 급증은 완료된 자산 이동이 아니라 향후 routing이나 protocol migration을 의미할 수 있다.

---

## 12. Evidence Classification

| Claim | Classification | Basis |
|---|---|---|
| ERC-20 defines a standard API for tokens in smart contracts | Directly specified | EIP-20 abstract |
| ERC-20 improves wallet and exchange interoperability | Directly specified | EIP-20 motivation |
| Zero-value transfers must emit `Transfer` | Directly specified | EIP-20 specification |
| Allowance changes from nonzero to nonzero create known UX/security risk | Directly specified | EIP-20 note |
| OpenZeppelin semantics influence ecosystem expectations | Inference from implementation practice | Widely used reference implementation |
| ERC-20 interface compliance does not guarantee neutral issuer policy | Analytical inference | Standard scope is interface-only |

---

## 13. References

[^ref-eip20]: Ethereum Improvement Proposals, "ERC-20: Token Standard," created 2015-11-19, accessed 2026-08-04, https://eips.ethereum.org/EIPS/eip-20

[^ref-eth-erc20]: ethereum.org, "ERC-20 Token Standard," official documentation, last updated 2026-07-30, accessed 2026-08-04, https://ethereum.org/developers/docs/standards/tokens/erc-20/

[^ref-oz-erc20]: OpenZeppelin Docs, "ERC-20," OpenZeppelin Contracts 5.x documentation, accessed 2026-08-04, https://docs.openzeppelin.com/contracts/5.x/erc20

---

## 14. Cross References

### Previous

- SMART-CONTRACTS-008 — Oracle Fundamentals

### Next

- TOKEN-STANDARDS-002 — ERC-721

---

## Review Status

### Technical Review

Passed.

- ERC-20의 핵심 함수, 이벤트, allowance 흐름을 명확히 분리했다.
- 인터페이스 표준과 구현 관행을 혼동하지 않았다.
- mint와 burn은 필수 기본 함수가 아니라 일반적 패턴으로 다뤘다.

### Evidence Review

Passed.

- 핵심 인터페이스 주장은 EIP-20에 연결했다.
- allowance 관련 경고는 EIP note에 연결했다.
- 구현 관련 관찰은 프로토콜 규칙이 아니라 OpenZeppelin 관행으로 귀속했다.

### Editorial Review

Passed.

- 섹션 순서는 프로젝트 형식을 따른다.
- 용어는 기존 Ethereum 및 Smart Contract 모듈과 일관된다.
- 표와 수식 표기는 과도하지 않고 읽기 가능하게 유지했다.

### Adversarial Review

Passed.

- 이 문서는 모든 ERC-20 토큰을 trust-minimized 자산으로 다루지 않는다.
- 인터페이스 준수가 건전한 tokenomics를 보장한다는 식으로 서술하지 않는다.
- approval activity를 완료된 transfer로 환원하지 않는다.

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
