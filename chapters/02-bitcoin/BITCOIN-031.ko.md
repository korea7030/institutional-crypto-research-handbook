---
knowledge_id: BITCOIN-031
title: Taproot
subtitle: Schnorr signature, SegWit v1 output, key-path와 script-path spending, Taptree commitment, 그리고 Tapscript 검증
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 140 min
estimated_study: 400 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Consensus
  - Transactions
  - Taproot
parent:
  - Modern Bitcoin
prerequisites:
  - BITCOIN-016
  - BITCOIN-019
  - BITCOIN-030
related_topics:
  - Schnorr Signatures
  - SegWit v1
  - Tapscript
  - Key Path
  - Script Path
  - Merkle Commitments
  - Bech32m
primary_sources:
  - REF-BIP-0340
  - REF-BIP-0341
  - REF-BIP-0342
  - REF-BTC-CORE-BIPS-001
  - REF-BTC-CORE-INTERPRETER-001
  - REF-BTC-CORE-VALIDATION-001
tags:
  - bitcoin
  - taproot
  - schnorr
  - tapscript
  - consensus
  - transactions
  - segwit-v1
  - privacy
---

# Taproot
> Modern Bitcoin  
> Research Unit: BITCOIN-031

---

## Research Brief

```yaml
knowledge_id: BITCOIN-031
title: Taproot
research_question: >
  What did Taproot change in Bitcoin's spending model, how do Schnorr
  signatures, key-path spending, script-path spending, and Taptree commitments
  interact, what does Tapscript add at validation time, and how should analysts
  separate the privacy and efficiency gains from the limits of what remains
  observable on-chain?
document_type: deep-dive
difficulty: L400
prerequisites:
  - BITCOIN-016
  - BITCOIN-019
  - BITCOIN-030
parent: Modern Bitcoin
previous: BITCOIN-030
next: BITCOIN-032
related_topics:
  - SegWit
  - Schnorr
  - Tapscript
  - Script Privacy
  - Descriptor Wallets
required_sections:
  - Learning Objectives
  - Key Questions
  - Executive Summary
  - Protocol Structure
  - Technical Mechanics
  - Mathematical or Economic Model
  - Security Assumptions
  - Bitcoin Core Implementation
  - On-Chain Implications
  - Institutional Thinking
  - Evidence Classification
out_of_scope:
  - Full MuSig2 protocol design
  - Descriptor wallet operational tutorials
  - Full covenant research landscape
  - Historical activation politics in detail
  - Non-Bitcoin Schnorr systems
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Taproot의 상위 수준 목표를 설명할 수 있다.
- BIP340, BIP341, BIP342의 역할을 구분할 수 있다.
- Taproot가 왜 SegWit version 1 output type인지 설명할 수 있다.
- key-path spending과 script-path spending을 구분할 수 있다.
- Taptree가 여러 script branch를 commit하면서도 사용된 branch만 공개하는 방식을 설명할 수 있다.
- Schnorr signature가 Taproot에 왜 중요한지 설명할 수 있다.
- Tapscript가 이전 witness system 대비 script validation을 어떻게 바꾸는지 설명할 수 있다.

---

## 2. 핵심 질문

1. Taproot란 무엇인가?
2. Schnorr signature는 Bitcoin에 무엇을 추가했는가?
3. Taproot output key란 무엇인가?
4. key-path spend와 script-path spend의 차이는 무엇인가?
5. script-path spend에서 control block은 무엇을 증명하는가?
6. Taproot는 privacy와 efficiency를 어떻게 개선하는가?
7. Taproot가 숨기지 못하는 것은 무엇인가?
8. BIP341과 BIP342는 책임을 어떻게 나누는가?

---

## 3. Executive Summary

Taproot는 서로 긴밀히 연결된 세 개의 BIP로 구성된 Bitcoin soft-fork 업그레이드다.

- BIP340: secp256k1용 Schnorr signature
- BIP341: SegWit version 1 output용 Taproot spending rule
- BIP342: Taproot script-path spend에 대한 Tapscript validation semantic[^ref-bip-0340][^ref-bip-0341][^ref-bip-0342]

Taproot는 하나의 coin을 개념적으로 두 가지 방식으로 spend할 수 있게 한다.

- key path: Taproot output key에 대한 Schnorr signature를 제공하는 방식
- script path: 미리 commit된 script leaf 하나와 그것이 commit된 tree 일부였음을 증명하는 control data를 공개하는 방식[^ref-bip-0341]

이 구조는 Bitcoin에 강력한 tradeoff를 제공한다. 복잡한 spending condition을 output 생성 시점에 commit해 두되, 모든 조건을 on-chain에 공개할 필요는 없다. 이후 spending이 key path로 협조적으로 이뤄지면 대체 script는 숨겨진 채 남는다. script path를 사용하더라도 전체 policy tree가 아니라 실제로 실행된 branch만 공개된다.[^ref-bip-0341]

따라서 분석 관점에서 Taproot는 privacy, efficiency, script commitment를 개선하는 업그레이드이지, 마법 같은 invisibility layer가 아니다. key-path spend는 많은 policy를 동일한 on-chain 외형으로 압축하지만, script-path spend는 여전히 사용된 leaf와 control block을 드러낸다. Taproot는 일부 distinguishability를 줄여주지만, observability 자체를 제거하지는 않는다.

---

## 4. Protocol Structure

### Three-BIP Structure

Taproot는 BIP별 책임을 분리해서 보면 이해가 쉬워진다.

| BIP | Role |
|---|---|
| BIP340 | secp256k1 위의 Schnorr signature scheme |
| BIP341 | SegWit v1용 Taproot output/spend 구조 |
| BIP342 | Tapscript leaf의 validation rule |

### SegWit Version 1 Output

BIP341은 Taproot를 SegWit version 1 output type으로 정의한다. 따라서 Taproot는 SegWit 바깥의 독립적인 transaction family가 아니라, SegWit framework 안에 추가된 새로운 witness-program version이다.[^ref-bip-0341]

### High-Level Form

Taproot output은 다음 요소에 commit한다.

- internal key
- 선택적인 script branch의 Merkle root
- 그리고 locking script에 사용되는 파생 output key[^ref-bip-0341]

이것이 privacy와 flexibility 개선의 핵심 구조다.

---

## 5. Schnorr Signatures

### Why BIP340 Matters

BIP340은 secp256k1용 Schnorr signature를 규정한다.[^ref-bip-0340]

Bitcoin의 기존 ECDSA usage와 비교할 때, Schnorr signature가 Taproot에서 중요한 이유는 다음과 같다.

- key-path spending을 위한 깔끔한 기반을 제공한다.
- key aggregation construction을 더 자연스럽게 만든다.
- Taproot spend를 위한 signature handling을 표준화한다.

### Analytical Boundary

Taproot가 도입되었다고 해서 모든 spend가 aggregated multisig라는 뜻은 아니다. 의미는 더 제한적이다. key-path spending을 사용할 경우, 시스템이 복잡한 policy를 하나의 output key 뒤에 표현할 수 있다는 뜻이다.

따라서 다음은 성립하지 않는다.

- single-signature처럼 보인다고 단순 custody라고 단정할 수는 없다.
- key-path usage라고 해서 fallback script branch가 없었다고 단정할 수는 없다.
- script-path usage는 실제로 실행된 branch만 드러낼 뿐이다.

---

## 6. Key-Path and Script-Path Spending

### Key Path

key-path spend에서는 spender가 Taproot output key에 대한 Schnorr signature를 제공한다. script leaf는 공개되지 않는다.[^ref-bip-0341]

참여자들이 key-path condition에 협조할 수 있다면, 이것이 가장 compact하고 privacy-preserving한 성공적 spend 형태다.

### Script Path

script-path spend에서는 spender가 다음을 공개한다.

- 실행된 script leaf
- 그 script를 만족시키는 witness data
- 그리고 공개된 leaf가 Taproot 구조 안에 commit돼 있었음을 증명하는 control block[^ref-bip-0341]

### Control Block Meaning

control block은 숨겨진 전체 policy 자체가 아니다. 공개된 script leaf를 Taproot output commitment에 다시 연결하는 proof material이다. 분석가는 이를 전체 대안 branch의 disclosure가 아니라, Merkle inclusion proof와 key-related data의 결합으로 이해해야 한다.

---

## 7. Taptree Commitments

### Merkleized Script Disclosure

Taproot는 여러 대체 spending script를 Merkle tree, 흔히 Taptree라고 부르는 구조에 배치할 수 있게 한다. script path로 spend할 때는 사용된 leaf만 공개하면 된다.[^ref-bip-0341]

### Privacy and Efficiency

이전 script-hash design에서는 특정 path가 사용되면 전체 redeem script 또는 witness script가 공개되는 경우가 많았다. Taproot에서는 사용되지 않은 대안을 모두 드러내는 대신, 실제 사용된 branch 하나만 공개할 수 있다.

### Important Limit

이것은 selective disclosure이지 total non-disclosure가 아니다.

- 사용되지 않은 branch는 숨겨진다.
- 사용된 branch는 보인다.
- Taproot output type의 존재 자체는 보인다.
- spending 시점에는 key-path spend와 script-path spend도 여전히 구분된다.

---

## 8. Tapscript

### BIP342 Role

BIP342는 BIP341 아래에서 동작하는 초기 script system, 즉 Tapscript의 validation semantic을 정의한다.[^ref-bip-0342]

### Why Separate It?

Taproot output structure와 Taproot script execution은 밀접하지만 동일한 문제는 아니다.

- BIP341은 무엇이 commit되고 spend path가 어떻게 형성되는지를 정의한다.
- BIP342는 공개된 script-path leaf를 실제로 어떻게 평가하는지를 정의한다.

### Key Validation Themes

BIP342는 다음을 포함한다.

- Taproot script execution rule
- signature opcode behavior
- common signature-message extension
- signature validation rule
- Tapscript용 resource-limit semantic[^ref-bip-0342]

Bitcoin Core의 script interpreter는 이 구분을 다음 surface로 드러낸다.

- `SigVersion::TAPROOT`
- `SigVersion::TAPSCRIPT`
- `SCRIPT_VERIFY_TAPROOT`
- upgradeable Taproot version과 `OP_SUCCESS` policy handling을 위한 discouragement flag[^ref-btc-core-interpreter]

---

## 9. Technical Mechanics

### Taproot Output Model

상위 수준에서 보면 구조는 다음과 같다.

```text
internal key
+ optional script Merkle root
-> tweaked output key
-> witness v1 output
```

세부 수식은 BIP341에 정의되어 있지만, 분석 관점의 핵심은 하나의 on-chain output key가 단순한 cooperative key-path spend 또는 숨겨진 script 대안을 가진 key를 동시에 표현할 수 있다는 점이다.[^ref-bip-0341]

### Key-Path Spend Shape

```text
witness = Schnorr signature
```

### Script-Path Spend Shape

```text
witness = stack items + script leaf + control block
```

이 operational distinction이 chain analysis에서 드러난다.

### Upgrade Surface

Taproot는 witness version 1을 사용하고 향후 확장 surface도 남겨두기 때문에, 분석가는 witness-v1 관련 behavior가 BIP341/342만으로 영구히 소진되었다고 가정하면 안 된다.

---

## 10. Validation Boundaries

### Consensus vs Policy

Taproot는 새로운 consensus-valid spend path와 script semantic을 정의한다. 그 위에 relay나 wallet level의 추가 제약이 얹힐 수는 있지만, 그것은 base consensus 의미와 분리해서 봐야 한다.

### Privacy Claims

Taproot는 특정한 의미에서 privacy를 개선한다.

- key-path spend는 많은 cooperative policy를 비슷한 외형으로 만든다.
- script-path spend는 사용된 branch만 공개한다.

하지만 다음을 보장하지는 않는다.

- identity privacy
- amount privacy
- graph privacy
- clustering 불가능성

### Key Path Is Not Proof of Simplicity

가장 흔한 분석 오류 중 하나는 key-path P2TR spend를 보고 "이건 single-sig였다"고 결론내리는 것이다. Taproot는 복잡한 cooperative policy도 하나의 output key 표현으로 spend할 수 있게 한다.

---

## 11. Security Assumptions and Failure Modes

### Cooperative Assumption for Key Path

가장 compact한 Taproot spend는 보통 key path를 만족시킬 수 있는 참여자들의 협조를 전제한다. 협조가 실패하면, 미리 commit된 script branch가 있다면 script path가 fallback behavior를 제공할 수 있다.

### Script Visibility Tradeoff

Taproot는 key path가 사용될 때 일상적인 disclosure를 줄여주지만, fallback condition이 실행되면 script path는 여전히 데이터를 공개한다. 따라서 절대적 opacity가 아니라 conditional privacy로 이해해야 한다.

### Future Extensibility

Taproot와 Tapscript는 향후 확장 surface를 포함한다. upgradeable Taproot version을 discourage하는 policy flag가 존재한다는 사실은, 미래의 모든 script behavior가 현재 표준이거나 현재 완전히 알려져 있다고 보면 안 된다는 뜻이다.[^ref-btc-core-interpreter]

---

## 12. Mathematical or Economic Model

### Information-Revelation Model

Taproot 이전의 script-hash 구조는 실행 시 전체 script를 드러내는 경우가 많았다. Taproot는 이를 branch-revelation model로 바꾼다.

```text
revealed_information
= used_leaf
+ control proof
```

과거 구조의 많은 경우에는 다음과 같았다.

```text
revealed_information
= entire policy script structure
```

### Cost and Size Intuition

일반적인 경우가 key path를 사용할 수 있다면, 항상 큰 script를 노출하는 방식보다 routine spending이 더 compact해질 수 있다. 이는 witness size와 virtual size가 여전히 fee market에 직접 영향을 미치기 때문에, 곧바로 fee economics와 연결된다.

### Analytical Consequence

Taproot는 다음 사이의 균형을 바꾼다.

- expressiveness
- routine disclosure
- spend size
- analyst observability

---

## 13. Bitcoin Core Implementation

### `doc/bips.md`

Bitcoin Core의 implemented-BIP index는 BIP340, BIP341, BIP342 validation rule이 v0.21.0 기준 구현되었고, mainnet activation은 v0.21.1, v24.0부터는 항상 활성화 상태임을 기록한다.[^ref-btc-core-bips]

### `script/interpreter.h`

interpreter surface에는 다음이 포함된다.

- `SigVersion::TAPROOT`
- `SigVersion::TAPSCRIPT`
- `SCRIPT_VERIFY_TAPROOT`
- Taproot 관련 policy flag[^ref-btc-core-interpreter]

이것은 Taproot가 단순한 address convention이 아니라, 새로운 execution context로 script-validation engine 안에 통합되었음을 보여준다.

### Validation Surface

Taproot spend는 궁극적으로 Bitcoin Core의 validation과 script verification path를 통과한다. 다만 구조적으로 중요한 점은 Taproot가 기존 ECDSA/P2WSH behavior를 다시 포장한 것이 아니라, 새로운 valid witness-program semantic과 새로운 script-execution rule을 추가했다는 데 있다.[^ref-btc-core-validation]

---

## 14. On-Chain Implications

### What Analysts Can See

chain data에서 분석가는 보통 다음을 식별할 수 있다.

- SegWit v1 output으로서의 Taproot output
- spend가 key path를 썼는지 script path를 썼는지
- script path 사용 시 공개된 script leaf와 control block

### What Analysts Cannot Infer Reliably

분석가는 일반적으로 다음을 신뢰성 있게 추론할 수 없다.

- key-path spend 뒤에 숨겨진 전체 policy
- script-path spend에서 사용되지 않은 모든 branch
- key-path spend가 한 명의 signer를 의미하는지, aggregation된 cooperation을 의미하는지
- 전체 off-chain coordination model

### Why Taproot Matters for Attribution

Taproot는 visible script complexity에 기대던 기존 heuristic 일부를 약화시킨다. 기관 분석 관점에서는 많은 P2TR observation에 대해 custody-structure inference의 confidence score를 낮춰야 한다는 뜻이다.

---

## 15. Institutional Thinking

Taproot는 기술적 업그레이드이자 분석적 업그레이드로 다뤄야 한다.

### Practical Implications

- surveillance 및 analytics system은 P2TR output을 분류하고 key-path spend와 script-path spend를 구분해야 한다.
- fee 및 size model은 Taproot spend-form 차이를 반영해야 한다.
- 관측된 spend data만으로 custody를 추정할 때는 확신을 낮추고 확률적으로 해석해야 한다.
- audit 및 recovery 관점에서는 숨겨진 script branch가 실제 사용되기 전까지 on-chain에 보이지 않으므로, 내부 문서화가 중요하다.

---

## 16. Common Misinterpretations

### "Taproot means multisig is invisible"

과장된 표현이다. Taproot는 cooperative policy를 더 단순한 on-chain 외형으로 보이게 만들 수 있지만, 모든 multisig나 complex policy가 모든 상황에서 감지 불가능해지는 것은 아니다.

### "A Taproot key-path spend proves single-party control"

틀렸다. output key 뒤에 숨겨진 aggregated 또는 coordinated control을 반영할 수 있다.

### "Script-path Taproot reveals the whole policy"

틀렸다. 사용된 leaf와 control proof를 공개할 뿐, 사용되지 않은 모든 대안을 공개하지는 않는다.

### "Taproot is just Schnorr"

틀렸다. Schnorr는 한 구성요소일 뿐이다. Taproot에는 SegWit v1 output semantic과 Tapscript validation 변화도 포함된다.

### "Taproot removed all script visibility"

틀렸다. 조건부로 disclosure를 줄였을 뿐, script behavior를 완전히 opaque하게 만들지는 않았다.

---

## 17. Research Questions

1. Taproot는 기존 script-based custody heuristic의 신뢰도를 얼마나 낮췄는가?
2. 현재 chain data에서 P2TR activity 중 key path와 script path의 비중은 어떠한가?
3. 기관은 cooperative key-path spend pattern을 통해 실제로 어느 정도의 fee 절감을 얻고 있는가?
4. chain-analysis system은 Taproot 기반 attribution의 불확실성을 어떻게 점수화해야 하는가?

---

## 18. Practical Exercises

### Exercise 1

BIP340, BIP341, BIP342의 차이를 각각 한 문장으로 설명하라.

### Exercise 2

Taproot script-path spend에서, 어떤 witness element가 실행을 증명하고 어떤 element가 committed tree 안의 inclusion을 증명하는지 나열하라.

### Exercise 3

동일한 P2TR key-path spend에 대해 두 가지 analyst interpretation을 작성하라.

- 순진하고 과도하게 확신하는 해석
- 신중하고 evidence-aware한 해석

### Exercise 4

P2WSH의 script reveal과 Taproot script-path reveal을 on-chain에 공개되는 정보 측면에서 비교하라.

---

## 19. Evidence Classification

| Claim Type | Classification | Notes |
|---|---|---|
| Schnorr, Taproot output, and Tapscript rules | Directly specified | BIP340/341/342 |
| Validation-engine integration | Directly specified | Bitcoin Core `doc/bips.md` and interpreter references |
| Privacy and disclosure consequences | Directly specified plus inference | Selective disclosure is specified; analytical implications are inferred |
| Custody-inference limitations | Inference from sources | Based on what Taproot hides or reveals |

---

## 20. Knowledge Graph

```text
Taproot
├─ Cryptography
│  ├─ Schnorr signatures
│  └─ key tweaking
├─ Output Semantics
│  ├─ SegWit v1
│  ├─ internal key
│  ├─ output key
│  └─ Taptree root
├─ Spend Paths
│  ├─ key path
│  ├─ script path
│  ├─ script leaf
│  └─ control block
├─ Validation
│  ├─ Tapscript
│  ├─ SigVersion::TAPROOT
│  ├─ SigVersion::TAPSCRIPT
│  └─ SCRIPT_VERIFY_TAPROOT
└─ Implications
   ├─ selective disclosure
   ├─ fee efficiency
   ├─ weaker visible-script heuristics
   └─ hidden-policy uncertainty
```

---

## 21. 참고문헌

### Primary Sources

[^ref-bip-0340]: BIP340, "Schnorr Signatures for secp256k1." https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki

[^ref-bip-0341]: BIP341, "Taproot: SegWit version 1 spending rules." https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki

[^ref-bip-0342]: BIP342, "Validation of Taproot Scripts." https://github.com/bitcoin/bips/blob/master/bip-0342.mediawiki

[^ref-btc-core-bips]: Bitcoin Core `doc/bips.md`, implemented BIP index including BIP340/341/342 support and activation notes. https://github.com/bitcoin/bitcoin/blob/master/doc/bips.md

[^ref-btc-core-interpreter]: Bitcoin Core Doxygen, `script/interpreter.h`, including `SigVersion::TAPROOT`, `SigVersion::TAPSCRIPT`, and `SCRIPT_VERIFY_TAPROOT`. https://doxygen.bitcoincore.org/interpreter_8h_source.html

[^ref-btc-core-validation]: Bitcoin Core validation and script-verification surfaces, including Taproot-related verification references in Doxygen. https://doxygen.bitcoincore.org/interpreter_8h.html

### Supporting Interpretation Notes

- Where this document discusses attribution uncertainty, custody inference limits, or privacy consequences for analysts, those statements are analytical inferences from what Taproot reveals and hides on-chain rather than explicit BIP commands about surveillance.

---

## 22. 교차 참조

### Previous

- BITCOIN-030 — SegWit

### Next

- BITCOIN-032 — Lightning Network

### Related

- BITCOIN-016 — Script & ScriptPubKey
- BITCOIN-019 — Wallets and Key Management
- BITCOIN-030 — SegWit
- BITCOIN-032 — Lightning Network

---

## Review Status

### Technical Review

Passed.

- BIP340, BIP341, BIP342의 역할을 분리했다.
- key-path spending과 script-path spending을 각각 독립적으로 설명했다.
- Taptree commitment와 control block의 의미를 전체 policy disclosure와 구분했다.
- Taproot를 분리된 시스템이 아니라 SegWit v1으로 일관되게 설명했다.

### Evidence Review

Passed.

- Taproot structure와 spending claim은 BIP341을 인용한다.
- Schnorr 관련 claim은 BIP340을 인용한다.
- Tapscript 관련 claim은 BIP342를 인용한다.
- Core implementation-state claim은 `doc/bips.md`와 interpreter reference를 인용한다.
- analytical privacy consequence는 해당되는 곳에서 inference로 라벨링했다.

### Editorial Review

Passed.

- 구조는 프로젝트 deep-dive format을 따른다.
- metadata는 완전하다.
- terminology는 internal key, output key, key path, script path, script leaf, control block, Tapscript로 일관된다.
- table과 code fence는 모두 닫혀 있다.

### Adversarial Review

Passed.

- 문서는 Taproot가 complex policy를 완전히 invisible하게 만든다고 주장하지 않는다.
- key-path spend에서 single-party control을 추론하지 않는다.
- script-path spend가 전체 hidden tree를 드러낸다고 주장하지 않는다.
- Taproot를 Schnorr 하나로 축소하지 않는다.
- chain data가 hidden policy에 대해 증명할 수 있는 범위를 과장하지 않는다.

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
