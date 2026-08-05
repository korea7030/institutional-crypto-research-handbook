---
knowledge_id: BITCOIN-019
title: Wallets and Key Management
subtitle: HD Wallet, 시드, 디스크립터, PSBT, 서명, watch-only 운영, multisig, custody 통제, 그리고 복구 리스크
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 115 min
estimated_study: 340 min
last_reviewed: 2026-08-04
domain:
  - Bitcoin
  - Wallets
  - Key Management
  - Custody
parent:
  - Bitcoin Internals
prerequisites:
  - BITCOIN-014
  - BITCOIN-015
  - BITCOIN-016
  - BITCOIN-018
related_topics:
  - HD Wallets
  - Seed Backup
  - Output Descriptors
  - PSBT
  - Watch-Only Wallets
  - Multisig
  - Taproot
  - Custody Operations
primary_sources:
  - REF-BIP-0032
  - REF-BIP-0039
  - REF-BIP-0086
  - REF-BIP-0174
  - REF-BIP-0380
  - REF-BTC-DEV-WALLETS-001
  - REF-BTC-CORE-DESCRIPTORS-001
  - REF-BTC-CORE-23-RELEASE-001
  - REF-BTC-CORE-30-RELEASE-001
  - REF-BTC-CORE-31-RELEASE-001
  - REF-BTC-CORE-WALLET-001
  - REF-BTC-CORE-SPKM-001
  - REF-BTC-CORE-WALLET-RPC-001
tags:
  - bitcoin
  - internals
  - wallets
  - key-management
  - hd-wallet
  - bip32
  - bip39
  - descriptors
  - psbt
  - custody
---

# Wallets and Key Management
> Bitcoin Internals  
> Research Unit: BITCOIN-019

---

## Research Brief

```yaml
knowledge_id: BITCOIN-019
title: Wallets and Key Management
research_question: >
  How do Bitcoin wallets derive keys, identify controlled UTXOs, construct
  and sign transactions, support watch-only and multisig workflows, and what
  operational controls are required for institutional key management without
  confusing wallet behavior with consensus rules?
document_type: deep-dive
difficulty: L300
prerequisites:
  - BITCOIN-014
  - BITCOIN-015
  - BITCOIN-016
  - BITCOIN-018
parent: Bitcoin Internals
previous: BITCOIN-018
next: BITCOIN-020
related_topics:
  - UTXO Model
  - Transaction Construction
  - ScriptPubKey
  - Transaction Fees
  - Descriptors
  - PSBT
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
  - Vendor-specific hardware wallet setup instructions
  - Legal custody requirements
  - Enterprise governance policy templates
  - Full Miniscript policy language
  - Lightning channel key management
```

## 1. 학습 목표

이 Research Unit을 마치면 독자는 다음을 수행할 수 있어야 한다.

- Bitcoin wallet이 무엇을 하고 무엇을 하지 않는지 설명할 수 있다.
- private key, public key, 주소, script, descriptor, UTXO를 구분할 수 있다.
- BIP32 기반 hierarchical deterministic wallet derivation을 설명할 수 있다.
- BIP39 mnemonic backup의 역할과 한계를 설명할 수 있다.
- key-only backup보다 descriptor가 복구를 개선하는 이유를 설명할 수 있다.
- PSBT가 transaction creation, update, signing, combining, finalization, broadcast를 어떻게 분리하는지 설명할 수 있다.
- watch-only wallet 동작을 설명할 수 있다.
- multisig가 일부 위험을 줄이는 동시에 coordination과 backup 리스크를 추가하는 이유를 설명할 수 있다.
- `CWallet`, `ScriptPubKeyMan`, descriptor wallet 같은 Bitcoin Core wallet 구현 개념을 식별할 수 있다.
- 서명, backup, recovery, monitoring을 둘러싼 기관용 통제를 설계할 수 있다.

---

## 2. 핵심 질문

1. Bitcoin에서 wallet이란 무엇인가?
2. wallet은 합의 규칙의 일부인가?
3. private key란 무엇인가?
4. HD wallet이란 무엇인가?
5. seed backup은 무엇을 복구하는가?
6. derivation path와 script type이 왜 복구에 필요한가?
7. output descriptor란 무엇인가?
8. watch-only wallet이란 무엇인가?
9. PSBT란 무엇인가?
10. multisig는 custody risk를 어떻게 바꾸는가?
11. 현재 Bitcoin Core에서 descriptor wallet은 무엇을 의미하는가?
12. wallet 관련 사실 중 온체인에서 보이는 것은 무엇이고, 오프체인에 남는 것은 무엇인가?

---

## 3. Executive Summary

Bitcoin wallet은 코인을 담는 저장 상자가 아니다. wallet은 키를 관리하고, script와 주소를 파생하고, 관련 UTXO를 추적하고, transaction을 만들고, 지출에 서명하고, confirmation 상태를 모니터링하는 소프트웨어와 운영 상태의 집합이다. 코인은 active chain의 UTXO로 존재하며, wallet balance는 wallet이 식별하고 지출할 수 있는 UTXO에서 파생된다.

현대 Bitcoin wallet은 보통 hierarchical deterministic key derivation을 사용한다. BIP32는 하나의 seed에서 keypair tree를 파생하는 HD wallet을 정의하며, private key를 노출하지 않고도 public derivation 정보를 선택적으로 공유할 수 있게 한다.[^ref-bip-0032] BIP39는 mnemonic words를 통해 entropy를 인코딩하고 binary seed를 파생하는 방식을 정의하지만, 이는 seed transport format이지 모든 wallet script policy를 완전하게 설명하는 형식은 아니다.[^ref-bip-0039]

descriptor는 복구 문제의 핵심 약점을 보완한다. BIP380은 output script descriptor를 output script 집합을 설명하는 언어로 정의하며, script type, key expression, derivation path, checksum을 포함할 수 있다.[^ref-bip-0380] Bitcoin Core의 descriptor 문서는 descriptor가 wallet software에 어떤 script와 주소를 생성해야 하는지 알려 주며, hardware signer workflow에 필요한 key origin data도 포함할 수 있다고 설명한다.[^ref-btc-core-descriptors]

BIP174가 정의한 PSBT는 unsigned transaction 또는 partially signed transaction을 wallet component와 signer 사이에서 주고받기 위한 표준 format이다. 여기에는 Creator, Updater, Signer, Combiner, Finalizer, Extractor 역할이 포함된다.[^ref-bip-0174]

기관 관점에서 key management는 가장 큰 손실 구간이다. 작업증명(Proof of Work, PoW)은 도난당한 private key를 보호해 주지 않는다. transaction validation은 내부 승인 절차가 실제로 준수되었는지 알 수 없다. 좋은 custody 체계에는 script 설계, descriptor backup, signer isolation, recovery testing, change control, transaction review, policy enforcement, monitoring이 함께 필요하다.

---

## 4. 프로토콜 구조

### Wallet은 합의 객체가 아니다

Bitcoin 합의 규칙은 transaction과 block을 검증한다. 합의는 다음을 알지 못한다.

- wallet 이름
- account balance
- approval policy
- seed phrase
- derivation path
- hardware signer 위치
- 기관 내 역할
- address label

합의가 보는 것은 script, signature, transaction field, UTXO, block이다.

### Wallet의 책임

wallet은 보통 다음 기능을 수행한다.

```text
key generation or import
script/address derivation
UTXO discovery
balance calculation
coin selection
fee estimation
transaction construction
transaction signing
broadcast or PSBT export
confirmation tracking
backup and recovery support
```

모든 wallet이 모든 기능을 수행하는 것은 아니다. 기관 환경에서는 이러한 기능이 watch-only system, offline signer, policy engine, broadcast node로 분리되는 경우가 많다.

### Key와 Script 객체

| Object | Meaning | Consensus field? |
|---|---|---:|
| Private key | 비밀 서명 재료 | No |
| Public key | 공개 검증 재료 | script 또는 witness에 들어갈 수 있음 |
| Address | 사용자 대상 목적지 인코딩 | No |
| `scriptPubKey` | 출력 잠금 script | Yes |
| UTXO | active chain의 spendable output | Yes |
| Descriptor | script와 key를 설명하는 wallet 표현식 | No |
| PSBT | unsigned/partially signed transaction을 위한 coordination format | No |
| Signature | scriptSig 또는 witness의 승인 데이터 | transaction에 포함되면 Yes |

### Wallet 상태 경계

wallet 상태에는 secret과 비밀이 아닌 메타데이터가 모두 포함된다.

```text
secret:
    private keys
    seed material
    passphrases

non-secret but critical:
    descriptors
    derivation paths
    key origins
    labels
    gap limits
    creation times
    policy documents
```

비밀이 아닌 메타데이터도 운영상 치명적일 수 있다. descriptor나 derivation path를 잃으면 seed words가 남아 있어도 복구가 어려워질 수 있다.

---

## 5. HD Wallet과 Seed

### BIP32

BIP32는 hierarchical deterministic wallet을 정의한다. 하나의 seed에서 extended private key와 extended public key tree를 파생할 수 있다. 이를 통해 여러 체인, 선택적 public-key 공유, 결정적 backup 동작을 지원한다.[^ref-bip-0032]

기본 구조는 다음과 같다.

```text
seed
  -> master extended private key
  -> child extended private/public keys
  -> addresses and scripts
```

extended key는 key material과 chain code를 포함한다. extended public key는 private key 없이도 non-hardened child public key를 파생할 수 있다.

### Hardened vs Unhardened Derivation

BIP32는 hardened derivation과 unhardened derivation을 구분한다. hardened derivation은 부모 private key material을 필요로 한다. unhardened public derivation은 extended public key만으로 가능하다.

운영상 의미는 다음과 같다.

| Derivation type | Public derivation possible from xpub? | Use case |
|---|---:|---|
| Hardened | No | derivation level 사이의 보안 경계 |
| Unhardened | Yes | watch-only 주소 생성 |

Bitcoin Developer 문서는 hardened derivation을 firewall로 설명하고, HD wallet의 internal/external chain 개념도 설명한다.[^ref-btc-dev-wallets]

### BIP39

BIP39는 결정적 wallet seed 생성을 위한 mnemonic words를 정의한다. entropy와 checksum을 word index로 인코딩하고, salt로 `"mnemonic" + passphrase`를 사용한 PBKDF2-HMAC-SHA512로 512-bit seed를 파생한다.[^ref-bip-0039]

중요한 한계는 다음과 같다.

```text
BIP39 mnemonic + passphrase = seed
seed alone may not specify script type, derivation path, or wallet policy
```

seed words만으로는 recovery metadata까지 보존되지 않기 때문에 기관용 complete backup이 되지 않는다.

### Passphrase

BIP39 passphrase는 서로 다른 유효한 seed를 만든다. 이는 plausible deniability를 제공할 수 있지만, 동시에 심각한 운영 리스크를 만든다. passphrase를 잃어버리거나 잘못 입력하면 의도한 wallet이 복구 불가능해질 수 있다.[^ref-bip-0039]

기관은 passphrase를 backup, access control, recovery testing이 필요한 cryptographic secret으로 취급해야 한다.

### Taproot Derivation

BIP86은 single-key P2TR output을 위한 derivation scheme을 정의하며, purpose `86'`와 internal key에서 Taproot output key를 파생하는 방식을 설명한다.[^ref-bip-0086]

이는 application-level convention이다. descriptor 기반 backup은 seed-only path assumption보다 script type과 derivation을 더 명시적으로 기록할 수 있다.

---

## 6. Descriptor

### Descriptor가 필요한 이유

BIP380은 wallet이 더 많은 script type과 derivation path를 지원하게 되면서 key-only backup이 충분하지 않게 되었다고 설명한다. private key만으로는 복원된 wallet이 어떤 output script와 주소를 파생해야 하는지 알지 못할 수 있다.[^ref-bip-0380]

descriptor는 script 구성을 명시적으로 만든다.

```text
wpkh([fingerprint/path]xpub.../0/*)
tr([fingerprint/path]xpub.../0/*)
wsh(sortedmulti(2,xpub1/*,xpub2/*,xpub3/*))
```

위 예시는 설명용이다. 실제 운영 환경의 descriptor는 정확한 key origin, range, checksum, backup record를 포함해야 한다.

### Descriptor 내용

BIP380 descriptor는 다음을 포함할 수 있다.

- script expression
- key expression
- extended public/private key
- key origin fingerprint
- derivation path
- wildcard
- checksum[^ref-bip-0380]

Bitcoin Core의 descriptor 문서는 descriptor가 P2PK, P2PKH, P2WPKH, P2SH, P2WSH, P2TR, multisig, sorted multisig, raw script, address, Miniscript expression, MuSig2 관련 expression을 설명할 수 있다고 말한다.[^ref-btc-core-descriptors]

### Descriptor Checksum

descriptor checksum은 필사 오류와 복사 오류를 탐지하는 데 도움을 준다. BIP380은 checksum 동작을 정의하며, Bitcoin Core RPC도 descriptor 출력에 checksum을 포함한다.[^ref-bip-0380] [^ref-btc-core-descriptors]

기관 backup 규칙은 다음과 같이 정리할 수 있다.

```text
backup descriptor with checksum
backup seed or signer material separately
test recovery from both
```

### Watch-Only Descriptor

watch-only wallet은 private key 없이 script와 UTXO를 추적할 수 있다. 이는 다음에 유용하다.

- online monitoring
- invoice generation
- audit reporting
- cold-storage separation
- signing 전 policy review

watch-only wallet은 signing material을 import하거나 외부 signer가 PSBT에 서명하지 않는 한 자금을 지출할 수 없어야 한다.

---

## 7. PSBT와 서명 워크플로

### PSBT 목적

BIP174는 signer가 signature를 생성하는 데 필요한 정보를 담고, 입력이 아직 완전하지 않을 때도 signature를 보관할 수 있는 format으로 PSBT를 정의한다. 이는 offline signer, hardware wallet, multisig, wallet interoperability를 지원하기 위한 것이다.[^ref-bip-0174]

### PSBT 역할

| Role | Function |
|---|---|
| Creator | unsigned transaction과 빈 PSBT map 생성 |
| Updater | UTXO, script, derivation path 등 알려진 데이터 추가 |
| Signer | 자신이 승인할 수 있는 입력에 signature 추가 |
| Combiner | 서로 다른 데이터/서명을 가진 여러 PSBT 결합 |
| Finalizer | partial data를 최종 scriptSig/witness data로 변환 |
| Extractor | fully signed network transaction 추출 |

여러 역할을 하나의 application이 수행할 수도 있지만, 기관 아키텍처에서는 이를 분리하는 것이 유용하다.

### Offline Signing

일반적인 기관 워크플로는 다음과 같다.

```text
online watch-only wallet:
    constructs PSBT
    verifies UTXOs, outputs, change, fee

offline signer:
    verifies transaction details
    signs approved inputs

online broadcaster:
    finalizes if complete
    broadcasts transaction
    monitors confirmation
```

signer는 자신이 무엇에 서명하는지 검증해야 한다. PSBT는 UTXO와 derivation 정보를 담을 수 있지만, coordinator가 손상되었다면 review control이 약한 경우 악의적 결제를 제안할 수 있다.

### Hardware Signer

hardware signer는 internet-connected system에 private key가 노출되는 위험을 줄여 준다. 그러나 다음 위험을 제거하지는 못한다.

- supply-chain risk
- malicious firmware risk
- wrong-address display risk
- bad backup risk
- coercion risk
- compromised coordinator risk
- inadequate recovery testing

기관 환경에서 hardware signer 배치는 더 큰 governance system 안의 하나의 통제로 봐야 한다.

---

## 8. Multisig와 Threshold Custody

### Multisig 목적

multisig script는 output을 지출하기 위해 여러 개의 signature를 요구한다. 흔한 policy는 다음과 같다.

```text
2-of-3
3-of-5
4-of-7
```

장점:

- 단일 키만으로는 지출할 수 없다.
- threshold를 만족할 수 있다면 한 개 키를 잃어도 자금이 파괴되지 않을 수 있다.
- 키를 지리적·조직적으로 분산할 수 있다.
- 승인 권한을 역할별로 분배할 수 있다.

리스크:

- descriptor 또는 redeemScript 손실
- quorum 가용성 부족
- signer coordination 실패
- address verification 실패
- spend 시 script-policy privacy leakage
- 운영 과신

### Multisig는 거버넌스 자체가 아니다

온체인 multisig는 signature만 강제한다. 사람의 governance를 알 수는 없다. 다음은 multisig만으로 보장되지 않는다.

- 올바른 이사회가 승인했는가
- 법적 조건이 충족되었는가
- 특정 인물이 강요받지 않았는가
- 내부 티켓이 사기성이 없었는가
- 두 개 키가 같은 운영자에게 통제되지 않는가

기관 통제는 인간 승인 정책을 key custody와 signing workflow에 매핑해야 한다.

### Script 가시성

구성 방식에 따라 드러나는 정보가 다르다.

| Construction | Reveal timing |
|---|---|
| P2SH multisig | spend 전까지 script 숨김 |
| P2WSH multisig | spend 전까지 script 숨김, witness discount 적용 |
| Taproot key-path aggregation | threshold-like coordination이 single-key spend처럼 보일 수 있음 |
| Taproot script path | 사용된 script path와 control block 공개 |

분석가는 key-path Taproot spend만 보고 full custody policy를 추론해서는 안 된다.

---

## 9. Balance, Coin Selection, and Fees

### Balance는 파생값이다

wallet balance는 통제 가능한 UTXO에서 계산된다.

```text
wallet_balance = sum(available UTXOs controlled by wallet scripts)
```

서로 다른 balance view는 다음을 제외할 수 있다.

- 미확정 change
- immature coinbase output
- locked coin
- watch-only coin
- conflicted transaction
- 정책상 활용성이 낮은 output

### Coin Selection

coin selection은 어떤 UTXO를 지출할지 결정한다. 이는 다음에 영향을 준다.

- transaction size
- fee
- change
- privacy
- UTXO fragmentation
- future consolidation need

coin selection은 wallet behavior이지 합의 규칙이 아니다. 같은 결제라도 서로 다른 wallet은 서로 다른 유효 transaction을 만들 수 있다.

### Change 관리

change output은 보통 송신자 wallet이 통제한다. change를 잘못 처리하면 다음 문제가 생길 수 있다.

- accidental fee overpayment
- address reuse
- privacy leakage
- change derivation path가 backup되지 않아 복구 누락

해당되는 경우 descriptor는 receive branch와 change branch를 모두 포괄해야 한다.

### Fee 통제

wallet은 fee estimation과 fee bumping과 상호작용한다.

- target fee rate 추정
- RBF signaling 설정
- CPFP child 구성
- payment batching
- 낮은 수수료 구간에서 UTXO consolidation

Bitcoin Core 31.0은 deprecated static wallet fee setting인 `-paytxfee`와 `settxfee`를 제거하고, fee estimation 또는 wallet RPC의 per-transaction fee-rate argument 사용을 권장한다.[^ref-btc-core-31-release]

---

## 10. Bitcoin Core 구현

### Descriptor Wallet 상태

Bitcoin Core 23.0은 새로 생성되는 wallet의 기본값을 descriptor wallet로 바꾸었고, single-key Taproot receiving address를 위한 자동 생성 `tr()` descriptor도 추가했다.[^ref-btc-core-23-release]

Bitcoin Core 30.0은 BDB legacy wallet의 생성과 로딩을 제거했고, 해당 wallet은 descriptor wallet 형식으로 마이그레이션할 수 있다. 여러 legacy-only RPC도 제거되었다.[^ref-btc-core-30-release]

이 문서는 2026년 8월 4일 기준 Bitcoin Core 31.x 동작을 반영한다.

### `CWallet`

Bitcoin Core의 `src/wallet/wallet.h`는 중심 wallet 구현 클래스인 `CWallet`을 정의하며, wallet transaction, script, policy fee, database, scriptPubKey manager 의존성을 포함한다.[^ref-btc-core-wallet]

상위 수준에서 `CWallet`은 다음을 조정한다.

- wallet state
- wallet 관련 transaction
- balance view
- transaction creation
- signing integration
- database persistence
- chain interface interaction

### `ScriptPubKeyMan`

Bitcoin Core의 `ScriptPubKeyMan`은 wallet의 scriptPubKey와 연관된 script 및 key를 관리한다. 문서에 따르면 scriptPubKey를 제공하고, scriptPubKey가 사용되었는지 표시하며, scriptPubKey와 관련 script/key의 저장과 encryption을 처리할 수 있다.[^ref-btc-core-spkm]

`DescriptorScriptPubKeyMan`은 descriptor 기반 wallet을 관리한다. Doxygen에 따르면 scriptPubKey, pubkey, key, encrypted key, descriptor cache, keypool size, wallet descriptor, MuSig2 nonce data용 map을 저장한다.[^ref-btc-core-spkm]

### Wallet Interface

Bitcoin Core wallet interface는 encryption, lock/unlock state, balance, address, transaction, PSBT 관련 wallet operation을 abstract interface를 통해 노출한다.[^ref-btc-core-wallet-rpc]

이 interface boundary가 중요한 이유는 wallet 기능이 Bitcoin Core에서 optional이며, 합의 검증과 분리되어 있기 때문이다.

### Descriptor RPC

Bitcoin Core RPC 문서의 `listdescriptors`는 descriptor-enabled wallet에 import된 descriptor를 나열하며, descriptor string, timestamp, active/internal status, range, next index field를 포함한다고 설명한다.[^ref-btc-core-wallet-rpc]

운영과 감사에서 descriptor 출력은 wallet configuration의 증거가 되지만, private descriptor가 private key를 포함한다면 secret으로 취급해야 한다.

---

## 11. 보안 가정

### 암호가 보호하는 것

private-key cryptography는 다음 조건이 만족될 때 unauthorized spending을 막아 준다.

- private key가 비밀로 유지된다.
- intended transaction에 대해서만 signature가 생성된다.
- 필요한 경우 randomness 또는 nonce handling이 올바르다.
- signing device가 올바른 transaction detail을 표시하고 검증한다.
- recovery material이 노출되거나 파괴되지 않는다.

BIP32 보안은 elliptic curve discrete logarithm hardness를 전제로 하며, derivation에 대한 의도된 성질도 포함한다.[^ref-bip-0032]

### Wallet이 보호해야 하는 것

wallet system은 다음을 보호해야 한다.

- seed entropy
- mnemonic words
- BIP39 passphrase
- hardware signer PIN과 backup
- private descriptor 또는 xprv
- PSBT signing workflow
- change address derivation
- address display integrity
- recovery metadata
- backup과 restoration process

### Failure Mode

| Failure mode | Description | Consequence |
|---|---|---|
| Seed theft | 공격자가 signing seed를 획득 | 자금 도난 가능 |
| Descriptor loss | seed는 있지만 script/path를 모름 | 복구가 불완전할 수 있음 |
| Passphrase loss | 올바른 seed라도 passphrase 없으면 다른 wallet 파생 | 자금 복구 불가 가능 |
| Wrong change path | change가 추적되지 않는 script로 전송 | 겉보기 손실 또는 복구 실패 |
| Address substitution | 악성코드가 목적지를 바꿈 | 되돌릴 수 없는 오송금 |
| Blind signing | signer가 output을 검증할 수 없음 | 도난 또는 운영 오류 |
| Quorum failure | multisig signer를 사용할 수 없음 | 자금이 일시적/영구적으로 묶일 수 있음 |
| Backup correlation | 모든 backup을 한곳에 저장 | 단일 물리적 침해가 통제를 무력화 |

### 위협 모델 경계

Bitcoin 합의 규칙은 invalid transaction을 거부하지만, stolen key로 서명된 valid transaction은 받아들인다. 따라서 custody security는 작업증명 문제가 아니라 운영·암호·키관리 문제다.

---

## 12. 온체인 함의

### 온체인에서 보이는 것

온체인 데이터는 다음을 드러낼 수 있다.

- script type
- address reuse
- input consolidation
- change pattern
- 공개된 multisig script
- spend 시점의 P2WSH witnessScript
- 사용된 경우 Taproot script-path data
- fee와 transaction construction 선택

### 직접 보이지 않는 것

온체인 데이터는 보통 다음을 드러내지 않는다.

- seed phrase
- derivation path
- descriptor backup 상태
- hardware signer vendor
- human approval workflow
- legal owner
- key의 지리적 분산 여부
- multisig 참여자의 독립성

### 분석상 주의

wallet fingerprinting은 유용할 수 있지만 위험하다. script type, input ordering, output ordering, RBF signaling, fee behavior, address reuse가 wallet software나 운영 관행을 시사할 수는 있지만, 보강 증거 없이는 거의 증명으로 볼 수 없다.

---

## 13. Institutional Thinking

### Architecture

견고한 기관 custody architecture는 보통 다음을 분리한다.

- watch-only monitoring
- transaction proposal
- policy approval
- signing device
- backup storage
- broadcast path
- reconciliation and accounting
- incident response

목표는 단일 시스템이나 인물이 unauthorized spend를 생성하고 승인하는 일을 동시에 하지 못하게 하는 것이다.

### Backup 설계

backup은 다음을 포함해야 한다.

- seed 또는 key share
- 사용하는 경우 BIP39 passphrase
- descriptor와 checksum
- xpub와 key origin 정보
- multisig quorum 구조
- signer inventory
- creation date 또는 scan-start 정보
- recovery procedure

backup은 주기적으로 테스트해야 한다. 테스트하지 않은 backup은 통제가 아니라 가정이다.

### Transaction Review

서명 전 기관 시스템은 다음을 검증해야 한다.

- recipient script와 amount
- change script와 amount
- total input value
- fee와 fee rate
- RBF signaling
- 필요한 경우 locktime과 sequence
- PSBT input UTXO
- descriptor와 key-origin consistency
- approval ticket 또는 policy reference

### Key Rotation과 Migration

key rotation은 쉽지 않다. Bitcoin에는 교체 가능한 account key가 없기 때문이다. 자금은 old UTXO에서 new script가 통제하는 new output으로 실제 이동되어야 한다.

migration에는 다음이 필요하다.

- fee planning
- address/script verification
- staged signing
- confirmation monitoring
- old receive path 폐기
- historical audit record 보존

---

## 14. Common Misinterpretations

### "Wallet이 Bitcoin을 저장한다"

아니다. Bitcoin은 UTXO로 표현된다. wallet은 관련 UTXO를 식별하고 지출하는 데 필요한 정보를 저장하거나 통제한다.

### "Seed Words는 언제나 완전한 Backup이다"

아니다. seed words는 script type, derivation path, descriptor, multisig policy, passphrase, scan metadata를 인코딩하지 않을 수 있다.

### "Xpub는 무해하다"

아니다. xpub만으로 지출은 못 하지만, 특정 derivation branch의 address history와 future receive address를 노출할 수 있다.

### "Multisig면 자동으로 기관 보안이다"

아니다. multisig는 signature threshold만 강제한다. 실제로 key가 독립적이고, 복구 가능하며, governance에 따라 운영되는지는 운영 설계에 달려 있다.

### "Watch-Only면 저위험이다"

꼭 그렇지 않다. watch-only system은 지출은 못 하지만 transaction intent, address, balance, xpub 기반 activity를 유출할 수 있다.

### "Hardware Wallet이면 안전하다"

아니다. hardware signer는 key exposure를 줄일 뿐, 악성 transaction proposal, 부실 backup, 약한 verification, governance failure를 해결하지는 못한다.

---

## 15. Research Questions

1. 기관용 descriptor multisig wallet을 처음부터 복구하려면 어떤 metadata가 필요한가?
2. 기관은 production key를 노출하지 않고 backup recovery를 어떻게 테스트해야 하는가?
3. wallet software를 fingerprint할 수 있는 온체인 특징은 무엇이며, 그 신뢰도는 어느 정도인가?
4. 내부 데이터 보안 정책에서 xpub access는 어떻게 분류해야 하는가?
5. custody system은 서명 전에 change output을 어떻게 검증해야 하는가?
6. 어떤 signer quorum이 theft resistance와 disaster recovery의 균형을 맞추는가?
7. Taproot와 MuSig2는 관측 가능한 custody pattern을 어떻게 바꾸는가?

---

## 16. Practical Exercises

### Exercise 1: Descriptor Inspection

descriptor-enabled Bitcoin Core wallet에서 다음을 실행하라.

```bash
bitcoin-cli -rpcwallet=<wallet> listdescriptors
```

다음을 기록하라.

- descriptor string
- checksum
- active 여부
- internal 여부
- range
- next index

shared note에는 private descriptor를 노출하지 말아야 한다.

### Exercise 2: Seed vs Descriptor Recovery

다음 wallet에 대한 recovery checklist를 작성하라.

```text
wsh(sortedmulti(2,xpub1/0/*,xpub2/0/*,xpub3/0/*))
```

어떤 데이터가 secret인지, 어떤 데이터가 public이지만 민감한지, 어떤 데이터가 operational metadata인지 구분하라.

### Exercise 3: PSBT Role Mapping

기관 transaction workflow를 PSBT 역할에 매핑하라.

| Step | PSBT role |
|---|---|
| Watch-only system creates unsigned spend | Creator |
| System adds UTXO and derivation data | Updater |
| Hardware devices sign | Signer |
| Coordinator merges signatures | Combiner |
| Coordinator creates final witness data | Finalizer |
| Broadcast node extracts transaction | Extractor |

### Exercise 4: On-Chain vs Off-Chain Claims

각 진술을 분류하라.

| Statement | On-chain fact | Wallet metadata | Operational claim | Heuristic |
|---|---:|---:|---:|---:|
| Output is P2WSH | Yes | No | No | No |
| Wallet uses 2-of-3 multisig | Maybe after spend | Yes | Maybe | Heuristic if unrevealed |
| Institution has three independent signers | No | Maybe | Yes | Not proven on-chain |
| Xpub can derive receive addresses | No | Yes | No | No |
| Hardware signer approved transaction | No | Maybe | Yes | Not proven on-chain |

---

## 17. Evidence Classification

### Source Ledger

| Source ID | Type | Description | Evidence Weight |
|---|---|---|---|
| REF-BIP-0032 | Application BIP | HD wallet key tree, extended keys, hardened derivation | A |
| REF-BIP-0039 | Application BIP | Mnemonic generation, checksum, PBKDF2 seed derivation, passphrase behavior | A |
| REF-BIP-0086 | Application BIP | Single-key P2TR derivation convention | A |
| REF-BIP-0174 | Application BIP | PSBT format and roles | A |
| REF-BIP-0380 | Application BIP | Output descriptor syntax, key expressions, checksums | A |
| REF-BTC-DEV-WALLETS-001 | Official developer documentation | HD wallet derivation notation, hardened derivation, internal/external chains | A |
| REF-BTC-CORE-DESCRIPTORS-001 | Bitcoin Core documentation | Descriptor language, script expressions, key origin info, checksums | A |
| REF-BTC-CORE-23-RELEASE-001 | Release documentation | Descriptor wallets default and Taproot descriptor generation | A |
| REF-BTC-CORE-30-RELEASE-001 | Release documentation | BDB legacy wallet removal and migration to descriptor wallets | A |
| REF-BTC-CORE-31-RELEASE-001 | Release documentation | Static wallet fee-setting removal and current wallet notes | A |
| REF-BTC-CORE-WALLET-001 | Primary implementation source | `CWallet` and wallet implementation dependencies | A |
| REF-BTC-CORE-SPKM-001 | Primary implementation source | `ScriptPubKeyMan` and `DescriptorScriptPubKeyMan` | A |
| REF-BTC-CORE-WALLET-RPC-001 | Official interface/RPC documentation | Wallet interface and descriptor listing behavior | A |

### Claim Ledger

| Claim | Classification | Evidence |
|---|---|---|
| wallet balance는 controlled UTXO에서 파생된다 | INTERPRETATION | UTXO model plus wallet responsibilities |
| BIP32는 seed로부터 HD wallet key derivation을 정의한다 | FACT | BIP32 |
| BIP39 mnemonic words는 seed를 파생하지만 wallet script를 완전히 규정하지는 않는다 | FACT | BIP39, BIP380 |
| descriptor는 output script와 key derivation data를 명시적으로 설명한다 | FACT | BIP380, Bitcoin Core descriptor docs |
| PSBT는 partially signed/offline/multi-signer workflow를 지원한다 | FACT | BIP174 |
| Bitcoin Core 23.0은 descriptor wallet을 기본값으로 만들었다 | FACT | Bitcoin Core 23.0 release notes |
| Bitcoin Core 30.0은 BDB legacy wallet 생성/로딩을 제거했다 | FACT | Bitcoin Core 30.0 release notes |
| `ScriptPubKeyMan`은 wallet scriptPubKey와 관련 key/script를 관리한다 | FACT | Bitcoin Core `scriptpubkeyman.h` |
| multisig가 실세계 기관 거버넌스를 증명한다 | COUNTERCLAIM | Rejected; signatures do not encode human approval |
| hardware wallet이 custody risk를 제거한다 | COUNTERCLAIM | Rejected; key exposure만 줄일 뿐 모든 운영 리스크를 제거하지 않음 |
| xpub 공개는 무위험이다 | COUNTERCLAIM | Rejected; xpub는 address graph data를 노출할 수 있음 |

### Evidence Labels

| Label | Meaning |
|---|---|
| FACT | Directly supported by cited source |
| INTERPRETATION | Analytical synthesis based on cited facts |
| POLICY | Wallet, custody, or operational convention rather than consensus |
| HEURISTIC | Practical inference with known counterexamples |
| UNKNOWN | Evidence is insufficient |

---

## 18. Knowledge Graph

```text
BITCOIN-019 Wallets and Key Management
|
+-- builds_on: BITCOIN-014 UTXO Model
+-- builds_on: BITCOIN-015 Transactions in Depth
+-- builds_on: BITCOIN-016 Script & ScriptPubKey
+-- builds_on: BITCOIN-018 Transaction Fees
|
+-- wallet
|   +-- tracks: UTXOs
|   +-- derives: scripts, addresses
|   +-- constructs: transactions
|   +-- signs: spends
|
+-- key material
|   +-- BIP32: HD derivation
|   +-- BIP39: mnemonic to seed
|   +-- BIP86: single-key P2TR derivation
|
+-- metadata
|   +-- descriptors
|   +-- derivation paths
|   +-- key origins
|   +-- labels
|
+-- workflows
|   +-- watch_only
|   +-- PSBT
|   +-- hardware_signing
|   +-- multisig
|
+-- Bitcoin Core
|   +-- CWallet
|   +-- ScriptPubKeyMan
|   +-- DescriptorScriptPubKeyMan
|   +-- listdescriptors
|
+-- institutional risk
    +-- theft: key compromise
    +-- loss: backup failure
    +-- error: wrong transaction signing
    +-- privacy: xpub and address leakage
```

---

## 19. 참고문헌

[^ref-bip-0032]: Pieter Wuille, "BIP 32: Hierarchical Deterministic Wallets," 2012-02-11, https://bips.dev/32/, accessed 2026-08-04.
[^ref-bip-0039]: Marek Palatinus, Pavol Rusnak, Aaron Voisine, and Sean Bowe, "BIP 39: Mnemonic code for generating deterministic keys," 2013-09-10, https://bips.dev/39/, accessed 2026-08-04.
[^ref-bip-0086]: Ava Chow, "BIP 86: Key Derivation for Single Key P2TR Outputs," 2021-06-22, https://bips.dev/86/, accessed 2026-08-04.
[^ref-bip-0174]: Ava Chow, "BIP 174: Partially Signed Bitcoin Transaction Format," 2017-07-12, https://bips.dev/174/, accessed 2026-08-04.
[^ref-bip-0380]: Pieter Wuille and Ava Chow, "BIP 380: Output Script Descriptors General Operation," 2021-06-27, https://bips.dev/380/, accessed 2026-08-04.
[^ref-btc-dev-wallets]: Bitcoin Developer Documentation, "Wallets," HD wallet derivation notation, hardened derivation, and internal/external chains, https://developer.bitcoin.org/devguide/wallets.html, accessed 2026-08-04.
[^ref-btc-core-descriptors]: Bitcoin Core Contributors, "Support for Output Descriptors in Bitcoin Core," descriptor language, key expressions, script expressions, key origin information, and checksums, https://github.com/bitcoin/bitcoin/blob/master/doc/descriptors.md, accessed 2026-08-04.
[^ref-btc-core-23-release]: Bitcoin Core Contributors, "Bitcoin Core 23.0 Release Notes," descriptor wallets default and Taproot descriptor wallet notes, https://bitcoincore.org/en/releases/23.0/, accessed 2026-08-04.
[^ref-btc-core-30-release]: Bitcoin Core Contributors, "Bitcoin Core 30.0 Release Notes," BDB legacy wallet removal and descriptor wallet migration, https://bitcoin.org/en/releases/30.0/, accessed 2026-08-04.
[^ref-btc-core-31-release]: Bitcoin Core Contributors, "Bitcoin Core 31.0 Release Notes," wallet setting changes and current release context, https://bitcoin.org/en/releases/31.0/, accessed 2026-08-04.
[^ref-btc-core-wallet]: Bitcoin Core Contributors, `src/wallet/wallet.h`, `CWallet` and wallet implementation structures, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/wallet_2wallet_8h_source.html, accessed 2026-08-04.
[^ref-btc-core-spkm]: Bitcoin Core Contributors, `src/wallet/scriptpubkeyman.h` and `DescriptorScriptPubKeyMan`, Bitcoin Core Doxygen 31.99.0 documentation, https://doxygen.bitcoincore.org/scriptpubkeyman_8h_source.html and https://doxygen.bitcoincore.org/classwallet_1_1_descriptor_script_pub_key_man.html, accessed 2026-08-04.
[^ref-btc-core-wallet-rpc]: Bitcoin Core Contributors, `src/interfaces/wallet.h` wallet interface and `listdescriptors` RPC documentation, Bitcoin Core Doxygen 31.99.0 documentation and RPC docs, https://doxygen.bitcoincore.org/interfaces_2wallet_8h_source.html and https://bitcoincore.org/en/doc/26.0.0/rpc/wallet/listdescriptors/, accessed 2026-08-04.

---

## 20. 교차 참조

### Parent

- Bitcoin Internals

### Previous

- BITCOIN-018 — Transaction Fees

### Next

- BITCOIN-020 — Mining

### Related

- BITCOIN-011 — Whitepaper Section 10 — Privacy
- BITCOIN-014 — UTXO Model
- BITCOIN-015 — Transactions in Depth
- BITCOIN-016 — Script & ScriptPubKey
- BITCOIN-017 — Mempool
- BITCOIN-018 — Transaction Fees
- POW-014 — Proof-of-Work Attack Models

---

## Review Status

### Technical Review

Passed.

- wallet 기능과 consensus state를 분리했다.
- BIP32, BIP39, BIP86, BIP174, BIP380을 올바른 application layer에 배치했다.
- descriptor backup과 seed-only backup의 차이를 분명히 했다.
- Bitcoin Core descriptor wallet 변화 이력을 날짜와 함께 구분했다: 23.0 기본 descriptor wallet, 30.0 BDB legacy wallet 생성/로딩 제거, 31.x 현재 맥락.
- `CWallet`, `ScriptPubKeyMan`, `DescriptorScriptPubKeyMan` 참조를 현재 Doxygen 기준으로 맞췄다.

### Evidence Review

Passed.

- HD wallet 관련 설명은 BIP32와 Bitcoin Developer 문서에 연결했다.
- mnemonic seed 관련 설명은 BIP39에 연결했다.
- descriptor 관련 설명은 BIP380과 Bitcoin Core descriptor 문서에 연결했다.
- PSBT workflow는 BIP174에 연결했다.
- Bitcoin Core wallet 동작은 release notes와 Doxygen source reference에 연결했다.
- custody control 관련 설명은 consensus fact가 아니라 operational interpretation으로 표시했다.

### Editorial Review

Passed.

- Markdown heading은 프로젝트 deep-dive 구조를 따른다.
- metadata가 완전하다.
- table과 code fence가 닫혀 있다.
- 용어는 wallet, seed, descriptor, xpub, xprv, PSBT, watch-only, multisig, custody로 일관된다.

### Adversarial Review

Passed.

- wallet이 코인을 저장한다고 주장하지 않았다.
- seed word를 언제나 완전한 backup으로 보지 않았다.
- xpub를 무해하다고 보지 않았다.
- multisig를 실세계 거버넌스의 증거로 보지 않았다.
- hardware wallet을 완전한 custody security로 보지 않았다.

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
