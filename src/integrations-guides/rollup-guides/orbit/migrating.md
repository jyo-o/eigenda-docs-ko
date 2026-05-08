
# Orbit Chain을 EigenDA로 마이그레이션 (Migrating)

다음은 native Arbitrum DA(즉 Ethereum calldata, 4844 blob, anytrust)를 사용하는 vanilla Arbitrum sequencer를, EigenDA를 통해 high throughput과 저비용을 달성하는 sequencer로 마이그레이션하는 절차다. parent chain 컨텍스트(예: Ethereum, Arbitrum One)와 무관하게 절차는 동일하지만, 배포 깊이에 따른 보안 [영향](overview.md#eth-l2-vs-l3-deployments)은 달라진다.

# 절차 (Procedure)

1. node software가 최신 vanilla Arbitrum Nitro 버전인지 확인한다. 일반적으로 Offchain Labs nitro github [releases](https://github.com/OffchainLabs/nitro/releases) 또는 Arbitrum 개발자 [문서](https://docs.arbitrum.io/run-arbitrum-node/arbos-releases/overview)에서 확인할 수 있다.

2. 최신 EigenDA x Nitro [버전](https://github.com/Layr-Labs/nitro/releases)을 사용하도록 node 업그레이드를 수행한다. fork 버전이 nitro reference와 동일한지 확인한다. EigenDA x Nitro fork는 최신 Arbitrum release와 backward compatible하게 설계되어 있어, liveness 손실 없이 native Arbitrum DA로도 동작해야 한다.

3. eigenda v2.1.3 migration action을 호출해 parent chain contract를 EigenDA용으로 업그레이드한다. 방법은 [여기](https://github.com/Layr-Labs/orbit-actions/tree/63ba07bbaa849117d2074ccd3c90c2628c58b36d/scripts/foundry/contract-upgrades/eigenda-v2.1.3#readme)에서 확인한다. 이는 필요한 eigenda contract 업그레이드를 적용하고, EigenDA batch destination 유형으로 검증을 수행하는 새 replay script에 필요한 `wasmModuleRoot` 로 갱신한다. 이 action은 node 백엔드 설정에서 EigenDA 기능 플래그를 활성화하기 **전에** 반드시 실행해야 한다.

4. EigenDA를 활성화하기 위해 Arbitrum node config를 갱신한다. 여기에는 batch poster, validator, sequencer node config 변경이 포함된다. 즉:
- **node** json config가 eigenda-proxy 설정을 사용하도록 갱신한다. 즉:

        `"eigen-da": {"enable": true,"rpc": "http://eigenda_proxy:3100"}`

5. 배포를 검증한다. 방법은 우리의 개발자 [runbook](https://eigen-labs.notion.site/Developer-Runbook-12466062c1a7495ebc1d803169c37644?pvs=4)에서 확인할 수 있다.
