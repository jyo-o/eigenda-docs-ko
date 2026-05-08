
# 안전한 무신뢰 업그레이드 (Secure Trustless Upgrade)

이 문서는 rollup 통합 업그레이드 절차를 정리한다. rollup derivation pipeline을 안전하게 업그레이드하는 방법에 대한 완전한 이해는 [spec](https://layr-labs.github.io/eigenda/integration/spec/7-secure-upgrade.html)을 참고한다.

EigenDA를 v2 또는 v3 cert로 사용 중이고 v4 또는 최신 통합으로 업그레이드하고자 한다면, 이 가이드는 핵심 개념(CertVerifier, CertVerifierRouter, activation block number)을 다루고, 구체적인 업그레이드 시나리오, 절차, 제약을 단계별로 설명한다.

## CertVerifier / CertVerifier Router / Activation Block Number 개요

`CertVerifier` 는 DA cert가 EigenDA 네트워크에 의해 충분히 저장되고 attestation되었는지 판단하는 contract다. DA cert는 검증에 필요한 모든 정보를 담은 versioned data structure다. 자세한 내용은 EigenDA [spec](https://layr-labs.github.io/eigenda/integration/spec/4-contracts.html#eigendacertverifier)을 참고한다.

`CertVerifierRouter` 는 block number에서 배포된 `CertVerifier` contract 주소로의 key-value map이다. key는 activation block number (ABN)라고 불리며, 각 `CertVerifier` 가 활성화되는 시점을 결정한다. 자세한 내용은 EigenDA [spec](https://layr-labs.github.io/eigenda/integration/spec/4-contracts.html#eigendacertverifierrouter)을 참고한다.

> rollup이 V2 또는 V3 DA cert를 사용한다면, EigenLabs가 배포한 router를 사용하거나 `CertVerifier` 를 직접 사용 중일 가능성이 높다. 자체 router를 배포할 것을 강력히 권장한다.

### CertVerifier Router 배포

`CertVerifierRouter` 를 배포하려면 먼저 `CertVerifier` 가 배포되어 있어야 한다. 그렇지 않다면 [EigenDA V2 Cert Verifier Deployer](https://github.com/Layr-Labs/eigenda/blob/26709ca468f176eb23c09f52a3122e5e18681c7d/contracts/script/deploy/certverifier/README.md#eigenda-v2-cert-verfier-deployer) 가이드를 참고한다. 최신 [release](https://github.com/Layr-Labs/eigenda/releases)를 사용한다.

> EigenDA V2는 V2 및 그 이후의 cert version (V3, V4)을 지원하는 업그레이드된 네트워크다. 이 secure upgrade는 deprecated된 EigenDA V1 네트워크를 지원하지 않는다.

router를 배포하고 default certVerifier를 설정하려면 [GitHub의 가이드](https://github.com/Layr-Labs/eigenda/blob/26709ca468f176eb23c09f52a3122e5e18681c7d/contracts/script/deploy/router/README.md)를 따른다.

DA cert를 처리할 때 router는 자동으로 reference block number를 추출하고 key-value map에서 적절한 `CertVerifier` 구현을 선택한다.

rollup은 자체 router를 배포할 것을 **강력히 권장**한다. EigenLabs가 배포한 router를 사용하면 EigenLabs의 업그레이드 일정을 따라야 한다. 예를 들어 EigenLabs가 1월 1일에 router를 업그레이드했지만 본인의 rollup은 3월에 업그레이드해야 한다면, 1월 1일 이전에 업그레이드하지 않은 L2 consensus node는 **halt** 된다. batcher와 L2 consensus node 모두 이전 버전을 실행해도, contract는 업그레이드된 후에는 이전 버전 cert를 거부할 수 있다. 이는 악의적 batcher가 버그가 있을 수 있는 이전 cert를 제출하는 것을 막기 위해 의도된 동작이다.

자체 CertVerifier를 배포할 수도 있고, 이미 배포된 immutable CertVerifier를 사용할 수도 있다.

### 현재 V3 CertVerifier 주소

현재 V3 cert를 사용 중이라면 [EigenDA directory](https://docs.eigencloud.xyz/eigenda/networks/mainnet#contract-addresses) 또는 아래에서 배포된 `CertVerifier` 주소를 확인할 수 있다:

| Network    | V3 CertVerifier |
| -------- | ------- |
| Mainnet  | [0x61692e93b6B045c444e942A91EcD1527F23A3FB7](https://etherscan.io/address/0x64AB2e9A86FA2E183CB6f01B2D4050c1c2dFAad4#readProxyContract)    |
| Sepolia | [0x19a469Ddb7199c7EB9E40455978b39894BB90974](https://sepolia.etherscan.io/address/0x9620dC4B3564198554e4D2b06dEFB7A369D90257#readProxyContract)     |

## 업그레이드 절차

이 절은 V3에서 V4 cert로의 업그레이드, V2에서 V4 cert로의 업그레이드를 설명한다.

batcher가 proxy release v2.x.x를 사용 중이라면 V3 cert를 사용 중이다. proxy release v1.8.x를 사용 중이라면 V2 cert를 사용 중이다. cert 버전은 L1 inbox의 calldata를 살펴 확인할 수도 있다:

1. Etherscan에서 본인의 batcher inbox 주소로 이동한다.
2. calldata를 복사한 뒤 integration_utils 도구의 [parse-altdacommitment](https://github.com/Layr-Labs/eigenda/tree/master/tools/integration_utils#parse-altdacommitment) subcommand를 사용해 Certificate Version을 출력한다.

### 시나리오 1 - V3에서 V4 Cert로 업그레이드

**컨텍스트:** batcher가 L1 inbox에 V3 cert를 게시 중이고, L2 consensus node의 EigenDA proxy가 L1 inbox의 V3 cert를 처리 중이다. router는 이미 배포되었고, 현재 L1 block number는 24136054 (2026년 1월 1일), 업그레이드는 L1 block number 24560854 (대략 3월 1일)에 예정되어 있다고 가정한다.

#### 절차
1. 업그레이드할 EigenDA release를 찾는다.
2. release에서 새 `CertVerifier` 구현을 배포한다.
3. `addCertVerifier(uint32 abn, address certVerifier)` 를 사용해 `CertVerifier` 를 activation block number (ABN)와 함께 등록한다. `abn` 을 `24560854` 로, `certVerifier` 를 배포된 CertVerifier 주소로 설정한다.
4. `24560854` 에 업그레이드를 공지하고, L2 consensus node가 `24560854` 이전에 proxy release로 업그레이드하도록 권장한다.
5. `24560854` 이전의 임의 시점에 batcher를 업그레이드한다.

proxy 업그레이드 후에도 batcher의 proxy는 계속 V3 cert를 생성하다가, blob의 Reference Block Number (RBN)가 `24560854` 이상이 되면 자동으로 V4 cert로 전환한다.

`24560856` (활성화 후 두 L1 block 시점)에는 dispersal된 blob의 RBN이 여전히 활성화 시점보다 이전일 수 있다. RBN은 EigenDA disperser가 현재 L1 block number보다 75 block 아래에서 선택하므로, L1 block number가 ABN을 지난 뒤에도 batcher가 V3 cert를 dispersal할 수 있다.

ABN 이후에 V3 cert 제출을 완전히 피하려면 시나리오 2의 수동 방식을 사용한다.

### 시나리오 2 - V2에서 V4 Cert로 업그레이드

**컨텍스트:** 시나리오 1과 동일하나, batcher가 L1 inbox에 V2 cert를 게시 중이다.

현재 EigenDA proxy는 V2 cert 제출을 지원하지 않는다. 가능한 업그레이드 방법은 두 가지다:
- (i) V2 cert를 구성하는 기능을 proxy에 추가
- (ii) ABN 이후 batcher를 수동으로 업그레이드

여기서는 두 번째 방법의 절차를 설명한다. 옵션 (i)의 코드가 구현된다면 절차는 시나리오 1과 정확히 동일하다.

1. 업그레이드할 EigenDA release를 찾는다.
2. release에서 새 `CertVerifier` 구현을 배포한다.
3. `addCertVerifier(uint32 abn, address certVerifier)` 를 사용해 `CertVerifier` 를 activation block number (ABN)와 함께 등록한다. `abn` 을 `24560854` 로, `certVerifier` 를 배포된 `CertVerifier` 주소로 설정한다.
4. `24560854` 에 업그레이드를 공지하고, L2 consensus node가 `24560854` 이전에 proxy release로 업그레이드하도록 권장한다.
5. `24560929` 이상의 시점에 batcher의 proxy를 정지한다.
6. proxy를 업그레이드한다.

`24560854` 대신 `24560929` 를 선택하는 이유는, disperser가 현재 L1 block number에서 [75](https://github.com/Layr-Labs/eigenda/blob/72f377a19a301f30eecad1b856532b4cc4fc4ffc/disperser/controller/controller_config.go#L185)를 빼서 RBN을 선택하기 때문이다.

batcher가 `24560940` 에 정지했고, 그 결과 ABN 이후인 `24560860` 에 V2 cert가 제출된 경우를 생각해 본다. 업그레이드된 모든 L2 consensus node는 그 V2 cert를 무시함으로써 거부한다.

batcher가 `24560929` 보다 이른 시점에 재시작하면, batcher software는 crash하거나 V3 cert를 생성하게 되며, 그 V3 cert는 최신 release로 업그레이드한 L2 consensus node만 처리할 수 있다.
