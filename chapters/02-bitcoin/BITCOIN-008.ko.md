---
knowledge_id: BITCOIN-008
title: 백서 섹션 7 — 디스크 공간 회수(Reclaiming Disk Space)
subtitle: 머클 루트, 블록 헤더, 트랜잭션 프루닝, UTXO 상태, 그리고 Bitcoin Core 프루닝 노드
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 80 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Data Structures
  - Storage
  - Validation
parent:
  - Bitcoin Whitepaper
prerequisites:
  - BITCOIN-004
  - BITCOIN-005
  - BITCOIN-006
  - BITCOIN-007
related_topics:
  - Merkle Tree
  - Block Header
  - UTXO Set
  - Pruned Node
  - Archival Node
  - Block Storage
  - SPV
primary_sources:
  - REF-BTC-WP-001
  - REF-BTC-DEV-BLOCKCHAIN-REF-001
  - REF-BTC-CORE-MERKLE-001
  - REF-BTC-CORE-VALIDATION-001
  - REF-BTC-CORE-VALIDATION-H-001
  - REF-BTC-CORE-BLOCKSTORAGE-001
  - REF-BTC-CORE-0110-PRUNING-001
  - REF-BTC-CORE-RPC-BLOCKCHAIN-001
tags:
  - bitcoin
  - whitepaper
  - reclaiming-disk-space
  - merkle-tree
  - block-header
  - pruning
  - utxo
---

# 백서 섹션 7 — 디스크 공간 회수(Reclaiming Disk Space)
> Deep Dive Series  
> Research Unit: BITCOIN-008

---

## Research Brief

```yaml
knowledge_id: BITCOIN-008
title: Whitepaper Section 7 — Reclaiming Disk Space
research_question: >
  How does Bitcoin use Merkle roots and compact block headers to preserve
  proof-of-work commitments while allowing old transaction data to be
  discarded or pruned after validation?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-004
  - BITCOIN-005
  - BITCOIN-006
  - BITCOIN-007
parent: Bitcoin Whitepaper
previous: BITCOIN-007
next: BITCOIN-009
related_topics:
  - Merkle Root
  - Block Header
  - UTXO Set
  - Pruned Node
  - Block File Pruning
  - Archival Data
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Original Design
  - Protocol Structure
  - Technical Mechanics
  - Mathematical or Economic Model
  - Security Assumptions
  - Bitcoin Core Implementation
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Full SPV security model
  - BIP37 bloom-filter privacy
  - AssumeUTXO bootstrapping
  - Compact block relay
  - Database-engine internals
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- 머클 루트가 왜 블록 헤더가 모든 트랜잭션에 커밋할 수 있게 해주는지 설명할 수 있다.
- 오래된 트랜잭션 데이터가 언제, 왜 검증 상태에 더 이상 필요하지 않을 때만 버려질 수 있는지 설명할 수 있다.
- 블록 헤더와 전체 직렬화 블록을 구분할 수 있다.
- 백서의 80바이트 헤더 저장량 추정을 재현할 수 있다.
- 백서의 머클 트리 압축과 현대 Bitcoin Core의 블록 파일 프루닝 차이를 설명할 수 있다.
- 프루닝된 풀노드와 아카이벌 풀노드를 구분할 수 있다.
- 프루닝이 검증 생략을 의미하지 않는 이유를 설명할 수 있다.
- 프루닝 노드가 나중에 어떤 데이터를 제공하지 못할 수 있는지 설명할 수 있다.
- 어떤 저장 관련 주장이 합의 규칙이고 어떤 것이 구현 선택인지 식별할 수 있다.
- 머클 루트를 전체 트랜잭션 검증의 대체재로 오해하지 않을 수 있다.

---

## 2. 핵심 질문

1. 백서 섹션 7에서 정확히 무엇을 회수(reclaim)한다는 것인가?
2. 블록 헤더에 머클 루트만 포함해도 블록 해시가 보존되는 이유는 무엇인가?
3. 오래된 트랜잭션 데이터를 버리면 무엇을 잃게 되는가?
4. 검증 노드가 계속 운영되기 위해 반드시 보존해야 하는 것은 무엇인가?
5. 연간 4.2MB 헤더 저장량 추정은 어떻게 계산되는가?
6. 블록 헤더와 직렬화 블록의 차이는 무엇인가?
7. Bitcoin Core는 트랜잭션 머클 루트를 어떻게 계산하고 검증하는가?
8. Bitcoin Core의 프루닝은 백서의 부분 머클 트리 그림과 어떻게 다른가?
9. 프루닝 노드도 새 블록을 독립적으로 검증할 수 있는가?
10. 프루닝 노드는 모든 과거 블록을 제공할 수 있는가?
11. 왜 미래 검증에는 오래된 사용 완료 트랜잭션보다 UTXO 세트가 더 중요한가?
12. 기관은 프루닝 인프라를 운영하기 전에 무엇을 고려해야 하는가?

---

## 3. Executive Summary

백서 섹션 7은 저장 공간 최적화를 설명한다. 사용된(spent) 트랜잭션이 충분히 깊게 묻히면, 블록 헤더가 머클 루트를 통해 트랜잭션 집합에 커밋하고 있으므로 오래된 트랜잭션 데이터는 디스크 공간을 절약하기 위해 버릴 수 있다.[^ref-btc-wp]

이 설계는 세 가지를 분리한다.

1. **블록 헤더:** 머클 루트를 포함하는 작고 고정 크기의 합의 데이터
2. **트랜잭션 데이터:** 블록 내용을 검증하고 상태를 갱신하는 데 사용되는 큰 가변 크기 데이터
3. **미래 검증에 필요한 상태:** UTXO 세트, 블록 인덱스 메타데이터, reorg 대응을 위한 최근 데이터

백서의 요지는 전체 검증을 생략할 수 있다는 뜻이 아니다. 오래된 사용 완료 트랜잭션 데이터는 검증 목적을 다한 뒤 메모리나 디스크에 영구히 남아 있을 필요가 없다는 것이다. 머클 루트는 오래된 트랜잭션 집합에 대한 압축된 암호학적 커밋먼트를 보존한다.

Bitcoin Developer 문서는 블록 헤더를 80바이트 구조로 설명하고, 머클 루트 필드가 블록의 모든 트랜잭션에서 도출된다고 설명한다.[^ref-btc-dev-blockchain-ref] Bitcoin Core는 `BlockMerkleRoot`를 다시 계산해 헤더의 `hashMerkleRoot`와 다르면 블록을 거부함으로써 이 커밋먼트를 검증한다.[^ref-btc-core-validation]

현대 Bitcoin Core에서 사용자가 체감하는 저장 최적화는 블록 파일 프루닝(block-file pruning)이다. Bitcoin Core 0.11.0은 검증은 모두 수행하되 검증 후 모든 raw block 및 undo 데이터를 디스크에 계속 보관하지 않는 노드를 지원하기 시작했다. 릴리스 노트는 raw block, undo data, block index, UTXO set를 구분하며, pruning은 데이터베이스가 구축된 뒤 raw block과 undo data를 삭제한다고 설명한다.[^ref-btc-core-0110-pruning]

이 구분이 중요하다.

| Concept | Whitepaper Section 7 | Modern Bitcoin Core |
|---|---|---|
| Main idea | 트랜잭션 트리 가지를 잘라 오래된 블록을 압축 | 검증 후 오래된 raw block/undo 파일 삭제 |
| Preserved commitment | 블록 헤더의 머클 루트 | 블록 인덱스/헤더 메타데이터와 검증된 chainstate |
| Needed for future validation | 현재 미사용 상태 | UTXO set와 체인 메타데이터 |
| Tradeoff | 과거 트랜잭션 데이터 축소 | 임의의 오래된 블록 데이터를 로컬에서 제공/재스캔 불가 |

분석가 관점에서 섹션 7은 Bitcoin을 계속 늘어나는 역사 원장으로 보는 시각과, 활성 검증 상태는 전체 과거 트랜잭션 아카이브보다 훨씬 작게 유지될 수 있는 시스템으로 보는 시각을 연결하는 다리다.

---

## 4. 원문 해석

백서 섹션 7은 코인의 최신 트랜잭션이 충분한 수의 블록 아래에 묻히면, 이전에 사용된 트랜잭션은 공간 절약을 위해 버릴 수 있다고 말한다. 이를 블록 해시를 바꾸지 않고 가능하게 하기 위해, 트랜잭션은 머클 트리로 배치되고 루트만 블록 해시에 포함된다고 설명한다.[^ref-btc-wp]

이 섹션은 또한 트리의 가지를 제거해 오래된 블록을 압축할 수 있고, 내부 해시는 저장할 필요가 없다고 말한다. 마지막으로 80바이트 헤더 저장량 추정을 제시한다.

```text
80 bytes * 6 blocks/hour * 24 hours/day * 365 days/year
= 4,204,800 bytes/year
~= 4.2 MB/year
```

백서는 2008년의 가정 아래에서 헤더만 메모리에 유지하는 것은 심각한 저장 부담이 아니라고 주장하기 위해 이 수치를 사용한다.[^ref-btc-wp]

---

## 5. 문자적 해석

### "Once the latest transaction in a coin is buried under enough blocks"

백서는 더 최근의 상태가 확정되면 더 오래된 사용 완료 트랜잭션의 관련성이 줄어든다는 소유권 체인을 가정한다. 현대 Bitcoin 용어로 보면, 핵심 활성 상태는 모든 과거 사용 완료 출력이 아니라 미사용 트랜잭션 출력(UTXO) 집합이다.

그렇다고 오래된 데이터가 무가치하다는 뜻은 아니다. 과거 데이터는 감사, 인덱싱, 재스캔, 피어 서비스, 포렌식, 제네시스부터 UTXO 세트를 독립적으로 재구성하는 데 여전히 중요하다.

### "Spent transactions before it can be discarded"

이는 어떤 노드의 운영 모드에서 더 이상 필요하지 않다면, 오래된 사용 완료 트랜잭션 데이터를 로컬 저장소에서 제거할 수 있다는 의미다. 합의가 역사를 지운다는 뜻은 아니다. 다른 아카이벌 노드는 전체 과거 데이터를 계속 보관할 수 있고, 제네시스부터 모든 상태를 독립적으로 재구성하려는 노드는 과거 블록에 접근할 수 있어야 한다.

### "Without breaking the block's hash"

블록 해시는 블록 헤더에 커밋한다. 헤더에는 모든 트랜잭션 바이트가 아니라 머클 루트가 포함된다. 따라서 헤더에 머클 루트가 고정된 뒤에는 오래된 트랜잭션 데이터를 로컬에서 삭제해도 과거 헤더 해시는 바뀌지 않는다.

### "The interior hashes do not need to be stored"

백서의 이 문장은 커밋이 완료된 뒤의 저장을 말한다. 내부 머클 해시는 전체 트랜잭션 데이터가 있을 때 재계산할 수 있고, 포함 증명을 위해 머클 브랜치 형태로 제공할 수도 있다. 모든 노드가 이를 별도 객체로 영구 저장할 필요는 없다.

---

## 6. 프로토콜 구조

### 블록 헤더 커밋먼트

Bitcoin 블록 헤더는 다음 필드를 포함한다.

| Field | Size | Role |
|---|---:|---|
| Version | 4 bytes | 블록 버전 규칙 신호 |
| Previous block hash | 32 bytes | 부모 헤더와 연결 |
| Merkle root | 32 bytes | 순서 있는 트랜잭션 목록에 커밋 |
| Time | 4 bytes | 타임스탬프 필드 |
| nBits | 4 bytes | 작업증명 목표값 인코딩 |
| Nonce | 4 bytes | 마이닝 탐색 필드 |

총합은 80바이트다. Bitcoin Developer 문서는 직렬화된 블록 헤더 포맷이 80바이트이며, 머클 루트가 블록의 모든 트랜잭션 해시에서 도출된다고 설명한다.[^ref-btc-dev-blockchain-ref]

### 머클 트리의 역할

머클 루트는 블록 헤더가 순서 있는 트랜잭션 목록 전체에 커밋하도록 만든다.

```text
transactions
    -> transaction IDs
    -> Merkle tree leaves
    -> intermediate hashes
    -> Merkle root
    -> block header
    -> block hash / proof of work
```

트랜잭션이 바뀌면 그 트랜잭션 ID도 바뀐다. 그러면 머클 경로와 머클 루트, 블록 헤더, 블록 해시도 함께 바뀐다. 따라서 과거 트랜잭션 데이터는 로컬에서 제거할 수 있어도, 헤더에 이미 박혀 있는 역사적 커밋먼트는 바뀌지 않는다.

### 검증 데이터 vs 아카이브 데이터

| Data | Needed For New-Block Validation? | Needed For Historical Audit? |
|---|---:|---:|
| Best-chain headers | Yes | Yes |
| UTXO set | Yes | Useful but not sufficient alone |
| Recent undo data | Useful for reorg handling | Useful |
| Old raw blocks | No, after validation and state update | Yes |
| Old spent transaction bodies | No, after state update | Yes |
| Block index metadata | Yes | Yes |

UTXO 세트는 활성 검증 상태다. 새 트랜잭션은 현재 미사용 출력만 소비한다. 노드가 이미 올바른 UTXO 세트를 가지고 있다면, 새 블록을 검증하는 데 모든 오래된 사용 완료 트랜잭션 본문이 필요한 것은 아니다.

---

## 7. 기술적 메커니즘

### 머클 루트 구성

Bitcoin의 트랜잭션 머클 트리는 트랜잭션 ID를 리프(leaf)로 사용한다. Bitcoin Developer 문서는 코인베이스 TXID가 첫 번째에 오고, 이후 트랜잭션 순서는 합의 요구에 따르며, 쌍은 이어붙인 후 double-SHA256으로 해시되고, 마지막 TXID가 홀수 개일 경우 복제된다고 설명한다.[^ref-btc-dev-blockchain-ref]

Bitcoin Core는 이를 `ComputeMerkleRoot`와 `BlockMerkleRoot`로 구현한다. `BlockMerkleRoot`는 블록의 트랜잭션 해시를 리프로 모아 `ComputeMerkleRoot`를 호출한다.[^ref-btc-core-merkle]

### 변이된 머클 트리(mutated Merkle trees)

Bitcoin Core의 `merkle.cpp`는 중복 트랜잭션 ID와 복제된 리프와 관련된 역사적인 머클 트리 결함을 경고한다. 구현은 끝부분에서 동일한 해시가 짝지어지는 경우를 감지하고 mutation 플래그를 세운다.[^ref-btc-core-merkle]

검증은 이 신호를 사용한다. `CheckMerkleRoot`는 루트를 다시 계산하고, 헤더의 `hashMerkleRoot`와 다르면 블록을 `bad-txnmrklroot`로 거부한다. mutated 플래그가 세트되면 블록은 `bad-txns-duplicate`로 거부된다.[^ref-btc-core-validation]

이 점은 섹션 7에 중요하다. 머클 루트는 압축된 커밋먼트이지만, Bitcoin의 실제 머클 구성에는 구현이 조심스럽게 다뤄야 하는 엣지 케이스가 존재한다.

### 프루닝 노드 메커니즘

Bitcoin Core의 프루닝은 단순한 메모리 내 가지 제거가 아니라 블록 파일 프루닝이다. Bitcoin Core 0.11.0 릴리스 노트는 노드가 전체 검증을 수행하면서도 모든 raw block과 undo data를 유지하지 않을 수 있다고 설명한다. raw block과 undo data는 검증이 끝나고, block index와 UTXO 데이터베이스를 구축하는 데 사용된 뒤 삭제된다.[^ref-btc-core-0110-pruning]

현재 Bitcoin Core는 pruning 관련 상수와 동작을 노출한다.

- `MIN_BLOCKS_TO_KEEP = 288`은 활성 tip에 가까운 블록을 담은 파일은 프루닝하지 않음을 뜻한다.[^ref-btc-core-validation-h]
- `MIN_DISK_SPACE_FOR_BLOCK_FILES = 550 MiB`는 블록 및 undo 파일에 대해 사용자가 최소한 할당해야 하는 디스크 공간이다.[^ref-btc-core-validation-h]
- `Chainstate::PruneAndFlush`는 필요할 경우 블록 파일을 프루닝하고, 프루닝이 발생했다면 chainstate 변경사항을 flush한다.[^ref-btc-core-validation]
- `BlockManager::PruneOneBlockFile`는 프루닝된 블록 파일에 대해 block index 엔트리에서 `BLOCK_HAVE_DATA`와 `BLOCK_HAVE_UNDO` 상태를 제거한다.[^ref-btc-core-blockstorage]

### 헤더 저장량 계산

백서의 저장량 추정은 다음과 같다.

```text
80 bytes/header
6 headers/hour
24 hours/day
365 days/year
```

```text
80 * 6 * 24 * 365 = 4,204,800 bytes
```

십진법 메가바이트로는:

```text
4,204,800 / 1,000,000 = 4.2048 MB
```

메비바이트로는:

```text
4,204,800 / 1,048,576 ~= 4.01 MiB
```

백서는 이를 대략 연 4.2MB로 반올림했다.[^ref-btc-wp]

---

## 8. 수학적 또는 경제적 모델

### 헤더만의 성장

다음을 정의하자.

```text
H = header size in bytes = 80
B = expected blocks per day = 144
D = days per year = 365
```

그러면:

```text
annual_header_growth = H * B * D
```

```text
annual_header_growth = 80 * 144 * 365 = 4,204,800 bytes
```

이 값이 전체 블록 데이터 성장보다 훨씬 작은 이유는, 전체 직렬화 블록에는 헤더만이 아니라 모든 트랜잭션이 들어가기 때문이다.

### 전체 과거 데이터 성장

전체 블록 기준으로는:

```text
annual_block_data_growth = average_serialized_block_size * blocks_per_year
```

여기서:

```text
blocks_per_year ~= 144 * 365 = 52,560
```

결과는 평균 블록 크기와 위트니스 데이터에 좌우되며, 단순 헤더 수로 결정되지 않는다. 따라서 섹션 7의 헤더 계산을 전체 블록체인 저장량 성장으로 오해하면 안 된다.

### 상태 vs 역사

검증 저장소는 개념적으로 다음과 같이 표현할 수 있다.

```text
node_storage = chain_metadata
             + active UTXO set
             + recent block/undo data
             + optional archival historical blocks
             + optional indexes
```

프루닝이 주로 줄이는 것은 선택적인 과거 블록/undo 보관 영역이다. chain metadata나 UTXO 세트를 없애는 것은 아니다.

---

## 9. 보안 가정

### 머클 루트가 제공하는 것

머클 루트는 압축된 무결성 커밋먼트를 제공한다. 노드가 트랜잭션과 필요한 브랜치 해시를 가지고 있다면, 그 트랜잭션이 특정 블록 헤더에 커밋되었는지 검증할 수 있다.

하지만 머클 루트는 다음을 증명하지 않는다.

- 트랜잭션이 유효한지
- 입력이 실제로 미사용 상태였는지
- 스크립트가 통과했는지
- 블록이 best chain 위에 있는지
- 블록이 충분한 confirmation을 가졌는지
- 노드가 보고 있는 피어 시야가 정직한지

이러한 한계 때문에 섹션 7 다음에 섹션 8의 SPV가 이어진다.

### 프루닝이 가정하는 것

프루닝된 풀노드는 다음을 가정한다.

1. 원시 데이터를 삭제하기 전에 과거 블록을 모두 검증했다.
2. 자신의 UTXO 세트와 블록 인덱스가 올바르다.
3. 예상 가능한 reorg 처리를 위해 충분한 최근 데이터를 유지한다.
4. 필요하면 피어로부터 오래된 블록을 다시 내려받을 수 있다. 다만 이는 네트워크 가용성에 의존한다.
5. 임의의 과거 블록을 제공하거나 오래된 지갑 이력을 로컬에서 재스캔할 필요가 없다.

프루닝은 디스크를 절약한다. 새 블록에 적용되는 검증 규칙을 약화시키는 것은 아니다.

### 아카이브 가용성

네트워크는 여전히 아카이벌 노드의 존재로부터 이익을 얻는다. 모든 노드가 강하게 프루닝한다면, 과거 블록 가용성, 재스캔, 독립 부트스트래핑, 포렌식 조사, 데이터 인덱싱이 더 어려워질 것이다. 이는 네트워크 수준의 가용성 고려사항이지 블록별 합의 규칙은 아니다.

---

## 10. Bitcoin Core 구현

### 머클 검증

Bitcoin Core는 `validation.cpp`의 `CheckMerkleRoot`를 통해 블록 머클 루트를 검증한다. 이 함수는:

1. `BlockMerkleRoot(block, &mutated)`를 호출한다.
2. 결과를 `block.hashMerkleRoot`와 비교한다.
3. 다르면 `bad-txnmrklroot`로 거부한다.
4. mutation이 감지되면 `bad-txns-duplicate`로 거부한다.
5. 머클 루트가 검사되었음을 캐시한다.[^ref-btc-core-validation]

### 머클 구성

`consensus/merkle.cpp`는 다음을 정의한다.

- `ComputeMerkleRoot`
- `BlockMerkleRoot`
- `BlockWitnessMerkleRoot`
- `TransactionMerklePath`

`ComputeMerkleRoot`는 어떤 레벨에 해시 수가 홀수이면 마지막 해시를 복제하고, `SHA256D64`로 각 쌍을 해시해 다음 레벨을 계산한다.[^ref-btc-core-merkle]

### 프루닝 상수

`validation.h`는 다음을 정의한다.

```text
MIN_BLOCKS_TO_KEEP = 288
MIN_DISK_SPACE_FOR_BLOCK_FILES = 550 MiB
```

주석은 최소 디스크 할당량이 블록/undo 파일을 위한 것이며, 활성 tip 근처의 최근 블록은 프루닝되지 않는다고 설명한다.[^ref-btc-core-validation-h]

### 프루닝 실행

`Chainstate::PruneAndFlush`는 pruning 플래그를 세우고 `FlushStateToDisk`를 호출한다. Doxygen 설명은 필요할 경우 블록 파일을 디스크에서 프루닝하고, 프루닝이 발생했다면 chainstate 변경을 flush한다고 말한다.[^ref-btc-core-validation]

`BlockManager::PruneOneBlockFile`는 프루닝된 파일에 대해 `BLOCK_HAVE_DATA`와 `BLOCK_HAVE_UNDO`를 지우고 파일 위치를 초기화함으로써 block index 엔트리를 갱신한다.[^ref-btc-core-blockstorage]

### RPC 및 운영 동작

Bitcoin Core의 RPC 코드도 pruning 제약을 반영한다. 예를 들어 `getblockfrompeer`에는 pruning을 방해하는 방식의 블록 fetch를 피하는 로직이 있으며, 블록을 받은 뒤 다시 프루닝될 수 있다는 점도 언급한다.[^ref-btc-core-rpc-blockchain]

이것은 운영상 차이를 다시 확인해 준다. 프루닝 노드는 검증은 할 수 있지만, 만능 과거 데이터 아카이브는 아니다.

---

## 11. 온체인 함의

### 분석가가 관측할 수 있는 것

온체인 데이터에서 분석가는 다음을 볼 수 있다.

- 블록 헤더
- 머클 루트
- 전체 블록이 उपलब्ध한 경우 트랜잭션 목록
- 전체 블록 또는 증명이 있으면 특정 트랜잭션의 포함 여부
- 어떤 과거 트랜잭션이 활성 체인에 남는지에 영향을 주는 reorg
- 전체 블록이 있을 경우, 블록의 트랜잭션 목록이 머클 루트와 일치하는지 여부

### 헤더만으로는 알 수 없는 것

헤더만으로는 다음을 알 수 없다.

- 블록의 모든 트랜잭션
- 거래 수수료
- UTXO 변화
- 스크립트 유효성
- 코인베이스 출력 값
- 위트니스 데이터
- 자금 흐름의 정확한 역사

헤더는 트랜잭션 데이터에 커밋할 뿐, 트랜잭션 데이터 자체는 아니다.

### 프루닝 인프라의 한계

프루닝 노드를 쓰는 분석가는 다음을 이해해야 한다.

| Task | Pruned Node Suitability |
|---|---|
| 동기화 후 새 블록 검증 | Strong |
| 현재 UTXO 상태 추적 | Strong |
| 모든 과거 블록 제공 | Weak |
| 처음부터 과거 주소 인덱스 구축 | 외부 데이터 없이는 Weak |
| 오래된 트랜잭션 조사 | 외부 아카이브 소스에 좌우 |
| 신규 입금 모니터링 | 독립 동기화 기준 Strong |
| 오래된 지갑 이력 재스캔 | Limited |

기관 리서치에서는 프루닝 노드가 독립 검증에는 유용하지만, 과거 분석이 필요하다면 아카이벌 인프라나 신뢰할 수 있는 과거 데이터셋과 함께 운용해야 한다.

---

## 12. 기관 관점에서의 해석

### 노드 전략

기관은 노드 역할을 분리해야 한다.

| Role | Recommended Storage Model |
|---|---|
| Settlement validation | Full node, pruned may be acceptable |
| Historical analytics | Archival node or indexed archival dataset |
| Chain surveillance | Archival plus real-time node feeds |
| Wallet recovery/rescan | Archival or wallet-specific indexed data |
| Public block serving | Archival node |
| Low-cost independent verification | Pruned full node |

### 리스크 통제

프루닝 노드를 사용할 때는:

- 과거 조사를 위한 최소 하나의 아카이브 소스를 유지하고,
- pruning 설정과 디스크 사용량을 모니터링하며,
- 어떤 워크플로가 오래된 블록 데이터를 필요로 하는지 문서화하고,
- 프루닝 노드가 임의의 과거 질의를 처리할 수 있다고 가정하지 말고,
- 입금 검증이 검증되지 않은 제3자 API에 의존하지 않도록 하며,
- 운영 정책에 따라 chainstate와 지갑 데이터를 백업하고,
- 실제 의존 전에 복구 및 reindex 절차를 테스트해야 한다.

### 리서치 해석

섹션 7은 종종 "Bitcoin은 저장 문제를 영구히 해결했다"는 식으로 오해된다. 더 정확한 해석은 다음과 같다.

> Bitcoin의 블록 헤더와 머클 루트 설계는 검증 커밋먼트를 작게 유지하면서, 전체 과거 트랜잭션 데이터 저장을 활성 검증 상태와 분리할 수 있게 만든다.

이것은 वास्तविक한 확장성 특성이지만, 모든 분석용 역사 데이터를 무한 압축할 수 있다는 뜻은 아니다.

---

## 13. 흔한 오해

### Misinterpretation 1: Pruned nodes are not full nodes.

틀렸다. 프루닝된 Bitcoin Core 노드도 블록을 완전히 검증하고 현재 UTXO 세트를 유지할 수 있다. 아카이벌 노드와의 차이는 검증 후 모든 오래된 raw block과 undo data를 보관하지 않는다는 점이다.[^ref-btc-core-0110-pruning]

### Misinterpretation 2: Merkle roots prove transaction validity.

틀렸다. 머클 루트는 순서 있는 트랜잭션 목록에 대한 커밋먼트를 증명한다. 유효성에는 트랜잭션, 스크립트, UTXO, 블록 구조, 컨텍스트 검증이 필요하다.

### Misinterpretation 3: Headers contain all information needed for chain analytics.

틀렸다. 헤더에는 커밋먼트와 작업증명 필드가 들어 있다. 과거 체인 분석에는 보통 전체 트랜잭션 데이터가 필요하다.

### Misinterpretation 4: The whitepaper's 4.2 MB per year is total blockchain growth.

틀렸다. 이는 10분 블록 간격 가정 아래 헤더 성장량일 뿐이다. 전체 블록 데이터는 트랜잭션과 위트니스 데이터에 따라 증가한다.

### Misinterpretation 5: Pruning deletes consensus history.

틀렸다. 프루닝은 로컬 raw block 및 undo 파일을 삭제한다. 합의 역사는 유효한 블록과 헤더에 의해 계속 정의되며, 아카이벌 노드는 전체 데이터를 보관할 수 있다.

### Misinterpretation 6: Interior Merkle hashes are consensus state.

과장이다. 블록 헤더 안의 루트는 합의적으로 중요하다. 내부 해시는 구성 및 증명을 위해 파생되는 데이터다.

---

## 14. 연구 질문

1. 현대 Bitcoin 노드에서 가장 큰 저장 구성 요소는 무엇인가: 블록, undo data, UTXO set, index, logs?
2. 프루닝은 오래된 주소에 대한 지갑 재스캔에 어떤 영향을 주는가?
3. 견고한 과거 데이터 가용성을 위해서는 몇 개의 아카이벌 노드가 필요한가?
4. 기관은 프루닝 검증 노드와 아카이벌 분석 인프라를 어떻게 조합해야 하는가?
5. 실제 블록 전파 과정에서 머클 루트 검증 실패는 얼마나 자주 발생하는가?
6. 제3자 과거 블록 API에 의존하는 운영상 리스크는 무엇인가?
7. UTXO 세트 성장은 전체 블록 데이터 성장과 비교해 어떤가?
8. reorg 가정은 최근 블록 최소 보존 정책에 어떤 영향을 주는가?
9. 사고 이후 포렌식 재구성을 위해 어떤 데이터를 보존해야 하는가?
10. 프루닝 노드의 한계는 리서치 방법론에 어떻게 공시해야 하는가?

---

## 15. Practical Exercises

1. 백서의 연간 헤더 저장량 추정을 다시 계산하라.
2. 트랜잭션 데이터를 삭제해도 오래된 블록 헤더 해시가 바뀌지 않는 이유를 설명하라.
3. 블록의 트랜잭션 목록이 주어졌을 때 머클 루트를 재계산하는 절차를 설명하라.
4. Bitcoin의 머클 구성에서 TXID 수가 홀수일 때 마지막 해시를 복제해야 하는 이유를 설명하라.
5. 5년 전 트랜잭션 하나에 대해 아카이벌 노드와 프루닝 노드가 각각 무엇에 답할 수 있는지 비교하라.
6. 현재 UTXO 세트는 왜 새 지출 검증에는 충분하지만 완전한 과거 분석에는 충분하지 않은지 설명하라.

---

## 16. 증거 분류

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BTC-WP-001 | Primary design paper | Whitepaper Section 7 storage-reclamation design | A |
| REF-BTC-DEV-BLOCKCHAIN-REF-001 | Official developer documentation | Block header and Merkle tree construction | A |
| REF-BTC-CORE-MERKLE-001 | Primary implementation source | `ComputeMerkleRoot`, `BlockMerkleRoot`, mutation handling | A |
| REF-BTC-CORE-VALIDATION-001 | Primary implementation source | `CheckMerkleRoot`, `PruneAndFlush`, block validation | A |
| REF-BTC-CORE-VALIDATION-H-001 | Primary implementation source | Pruning constants in `validation.h` | A |
| REF-BTC-CORE-BLOCKSTORAGE-001 | Primary implementation source | Block-file pruning and block-index status updates | A |
| REF-BTC-CORE-0110-PRUNING-001 | Release documentation | Bitcoin Core 0.11.0 block-file pruning behavior | A |
| REF-BTC-CORE-RPC-BLOCKCHAIN-001 | Primary implementation source | RPC behavior around pruned block fetching | B |

### Claim Ledger

| Claim ID | Claim | Classification | Evidence | Source |
|---|---|---|---|---|
| C001 | Whitepaper Section 7 says spent transactions can be discarded after later transactions are buried enough. | FACT | A | REF-BTC-WP-001 |
| C002 | The block header includes only the Merkle root, not all transaction data. | FACT | A | REF-BTC-WP-001; REF-BTC-DEV-BLOCKCHAIN-REF-001 |
| C003 | Bitcoin block headers are 80 bytes. | FACT | A | REF-BTC-WP-001; REF-BTC-DEV-BLOCKCHAIN-REF-001 |
| C004 | The whitepaper's header-growth estimate is about 4.2 MB per year. | FACT | A | REF-BTC-WP-001 |
| C005 | Bitcoin Core computes block Merkle roots from transaction hashes using `BlockMerkleRoot`. | FACT | A | REF-BTC-CORE-MERKLE-001 |
| C006 | Bitcoin Core rejects Merkle-root mismatch as `bad-txnmrklroot`. | FACT | A | REF-BTC-CORE-VALIDATION-001 |
| C007 | Bitcoin Core detects certain duplicate-transaction Merkle mutation cases. | FACT | A | REF-BTC-CORE-MERKLE-001; REF-BTC-CORE-VALIDATION-001 |
| C008 | Bitcoin Core pruning deletes raw block and undo data after validation while preserving databases needed for operation. | FACT | A | REF-BTC-CORE-0110-PRUNING-001 |
| C009 | A pruned node can validate new blocks but cannot serve arbitrary old block data locally. | FACT | A | REF-BTC-CORE-0110-PRUNING-001; REF-BTC-CORE-RPC-BLOCKCHAIN-001 |
| C010 | Pruned validation infrastructure should be paired with archival sources for historical analytics. | INTERPRETATION | B | REF-BTC-CORE-0110-PRUNING-001 |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| HEURISTIC | Practical operating rule requiring context |
| OPEN | Unresolved or environment-dependent question |

---

## 17. 지식 그래프

```text
BITCOIN-008 Reclaiming Disk Space
|
+-- interprets: Whitepaper Section 7
|
+-- uses: Merkle Tree
|   +-- leaves: transaction IDs
|   +-- root: committed in block header
|   +-- enables: compact inclusion commitments
|
+-- block_header
|   +-- size: 80 bytes
|   +-- contains: previous hash, Merkle root, time, nBits, nonce, version
|   +-- supports: proof-of-work chain tracking
|
+-- validation_state
|   +-- requires: UTXO set
|   +-- requires: block index
|   +-- may not require: old spent transaction bodies
|
+-- Bitcoin Core pruning
|   +-- deletes: old raw block files
|   +-- deletes: old undo files
|   +-- preserves: validation databases
|   +-- limits: historical serving/rescans
|
+-- leads_to: BITCOIN-009 Simplified Payment Verification
```

---

## 18. 참고문헌

[^ref-btc-wp]: Satoshi Nakamoto, "Bitcoin: A Peer-to-Peer Electronic Cash System," Section 7, 2008, https://bitcoin.org/bitcoin.pdf, accessed 2026-08-04.

[^ref-btc-dev-blockchain-ref]: Bitcoin Developer Documentation, "Block Chain — Block Headers and Merkle Trees," https://developer.bitcoin.org/reference/block_chain.html, accessed 2026-08-04.

[^ref-btc-core-merkle]: Bitcoin Core Contributors, `src/consensus/merkle.cpp`, functions `ComputeMerkleRoot`, `BlockMerkleRoot`, `BlockWitnessMerkleRoot`, and `TransactionMerklePath`, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/consensus_2merkle_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-validation]: Bitcoin Core Contributors, `src/validation.cpp`, functions `CheckMerkleRoot`, `IsBlockMutated`, and `Chainstate::PruneAndFlush`, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/validation_8cpp_source.html, accessed 2026-08-04.

[^ref-btc-core-validation-h]: Bitcoin Core Contributors, `src/validation.h`, constants `MIN_BLOCKS_TO_KEEP` and `MIN_DISK_SPACE_FOR_BLOCK_FILES`, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/validation_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-blockstorage]: Bitcoin Core Contributors, `src/node/blockstorage.cpp` and `src/node/blockstorage.h`, `BlockManager` pruning and block-file status handling, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/blockstorage_8cpp_source.html and https://doxygen.bitcoincore.org/blockstorage_8h_source.html, accessed 2026-08-04.

[^ref-btc-core-0110-pruning]: Bitcoin Core Contributors, "Bitcoin Core 0.11.0 Release Notes — Block file pruning," https://bitcoincore.org/en/releases/0.11.0/, accessed 2026-08-04.

[^ref-btc-core-rpc-blockchain]: Bitcoin Core Contributors, `src/rpc/blockchain.cpp`, `getblockfrompeer` and pruning-related RPC behavior, Bitcoin Core Doxygen 31.99.0 / master documentation, https://doxygen.bitcoincore.org/blockchain_8cpp_source.html, accessed 2026-08-04.

---

## 19. 교차 참조

### Parent

- Bitcoin Whitepaper

### Previous

- BITCOIN-007 — Whitepaper Section 6 — Incentive

### Next

- BITCOIN-009 — Whitepaper Section 8 — Simplified Payment Verification

### Related

- BITCOIN-004 — Whitepaper Section 3 — Timestamp Server
- BITCOIN-005 — Whitepaper Section 4 — Proof of Work
- BITCOIN-018 — Blocks & Block Headers
- BITCOIN-023 — Chain Reorganization
- POW-004 — SHA-256, Double SHA-256, and Merkle Roots
- POW-009 — Coinbase Transaction and Mining Commitments

---

## Review Status

### Technical Review

Passed.

- Whitepaper Section 7과 현대 Bitcoin Core pruning을 분리했다.
- 머클 루트 커밋먼트, 헤더 저장, UTXO 상태, raw block 보관 데이터를 분리했다.
- `ComputeMerkleRoot`, `BlockMerkleRoot`, `CheckMerkleRoot`, `PruneAndFlush`, block-storage pruning 동작을 Bitcoin Core 소스와 대조했다.
- 머클 루트가 트랜잭션 유효성을 증명한다는 식으로 서술하지 않았다.

### Evidence Review

Passed.

- 백서 관련 주장은 섹션 7을 직접 인용한다.
- 블록 헤더와 머클 트리 포맷 주장은 공식 Bitcoin Developer 문서를 인용한다.
- Bitcoin Core 구현 주장은 Doxygen 소스 페이지를 인용한다.
- 프루닝 노드 동작은 Bitcoin Core 릴리스 문서와 소스 참조를 인용한다.

### Editorial Review

Passed.

- Markdown 헤딩은 프로젝트 딥다이브 구조를 따른다.
- 메타데이터는 완전하다.
- 표와 코드 펜스는 닫혀 있다.
- Merkle root, header, raw block data, undo data, UTXO set, pruned node, archival node 용어를 일관되게 사용했다.

### Adversarial Review

Passed.

- 프루닝을 불완전한 검증과 동일시하지 않았다.
- 모든 과거 분석이 헤더만으로 가능하다고 암시하지 않았다.
- 로컬 데이터 삭제와 합의 역사 자체를 구분했다.
- 아카이벌 노드 가용성의 트레이드오프를 명시했다.

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
