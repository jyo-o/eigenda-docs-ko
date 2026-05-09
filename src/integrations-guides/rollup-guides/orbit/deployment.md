
# Arbitrum Orbit 배포 (Deployment)

[Arbitrum Orbit](https://docs.arbitrum.io/launch-orbit-chain/orbit-gentle-introduction)은 [Offchain Labs](https://www.offchainlabs.com/)가 개발한 Rollup Development Kit (RDK)로, *Arbitrum One* 과 *Arbitrum Nova* 를 구동하는 동일한 software를 사용해 rollup 개발자가 자신만의 rollup을 만들 수 있도록 한다.

## EigenDA Proxy (프록시)

Arbitrum node는 안전한 통신과 낮은 코드 오버헤드를 위해 proxy를 통해 EigenDA와 통신한다. 자세한 내용은 [여기](../../eigenda-proxy/eigenda-proxy.md)에서 확인할 수 있다. 이 통합을 안전하게 사용하려면 proxy 인스턴스를 **반드시** 띄워야 한다. node config에서는 다음과 같이 표시된다:
```
"eigen-da": {"enable": true,"rpc": "http://eigenda_proxy:3100"}
```

Nitro node에서 EigenDA를 활성화하는 CLI 플래그도 제공된다:
```
--node.eigen-da.enable=true
--node.eigen-da.rpc=http://eigenda_proxy:3100
```

## EigenDA가 통합된 Rollup Creator 배포

1. yarn과 hardhat이 설치되어 있다고 가정한다.

2. 최신 stable [release](https://github.com/Layr-Labs/nitro-contracts/releases)를 사용해 EigenDA Nitro contracts fork에서 nitro contracts source [code](https://github.com/Layr-Labs/nitro-contracts)를 다운로드한다.

3. 최상위 디렉토리에서 기존 템플릿으로부터 새 deployment config를 만든다:
```
cp scripts/config.ts.example scripts/config.ts
```

parent chain 레벨(즉 L1 vs L2)에 따라 `maxDataSize` 필드를 적절히 갱신한다. 일반적으로:
- Ethereum에 settle하는 L2의 경우 `117964`
- L3의 경우 `104857`

**이 값은 네트워크별 파라미터(예: tx calldata 한도)에 따라 설정되며, 새로운 settlement 도메인에 배포할 때는 변경이 필요할 수 있음에 유의한다**

4. 다음 명령으로 배포를 시작한다:
```bash
yarn deploy-factory --network ${NETWORK_ID} 
```

어떤 env var을 제공해야 하는지에 대한 자세한 내용은 [*hardhat.config.ts*](https://github.com/Layr-Labs/nitro-contracts/blob/278fdbc39089fa86330f0c23f0a05aee61972c84/hardhat.config.ts)를 참고한다.

스크립트 실행에는 몇 분이 걸리며, 진행하면서 배포된 contract 주소들을 출력한다. 완료되면 새 chain 배포에 사용할 수 있는 rollup creator factory가 준비된다.

**NOTE: 이 스크립트는 hardhat이므로, 실행 중간에 치명적 실패가 발생해도 state checkpoint가 없다. 안정적인 RPC provider에 연결되어 있고 충분한 자금이 있는지 확인한 뒤 본인 책임으로 사용한다.**

### 호스팅된 Rollup Creator로 배포
Orbit [문서](https://docs.arbitrum.io/launch-orbit-chain/how-tos/orbit-sdk-deploying-rollup-chain)는 이미 배포된 rollup creator를 사용해 새 chain 배포를 트리거하는 방법에 대한 종합적인 안내를 제공한다. orbit-sdk를 사용하려면 우리의 fork [여기](https://github.com/Layr-Labs/eigenda-orbit-sdk)를 사용한다.

또한 다음 Rollup Creator factory를 유지한다:

| Contracts Version | Network | Rollup Creator Address | EigenDAV1 CertVerifier Address |
|---------|---------|---------|-----------|
| [v2.1.3](https://github.com/Layr-Labs/nitro-contracts/releases/tag/v2.1.3)  | Ethereum Mainnet | [0xdD6258539c41687B9afd38983c0456493423C73d](https://etherscan.io/address/0xdD6258539c41687B9afd38983c0456493423C73d#code) | [0x787c88E70900f6AE10E7B9D18024482895EBD1eb](https://etherscan.io/address/0x787c88E70900f6AE10E7B9D18024482895EBD1eb#code) |
| [v2.1.3](https://github.com/Layr-Labs/nitro-contracts/releases/tag/v2.1.3)  | Ethereum Sepolia | [0x5af6fe79EB79A8177268ab143f31f7e0A9b7Fd53](https://sepolia.etherscan.io/address/0x5af6fe79EB79A8177268ab143f31f7e0A9b7Fd53#code) | [0xb1ffa45789f1e3ea513d58202389c8eea1e6de4e](https://sepolia.etherscan.io/address/0xb1ffa45789f1e3ea513d58202389c8eea1e6de4e#code) |
| [v2.1.3](https://github.com/Layr-Labs/nitro-contracts/releases/tag/v2.1.3)  | Arbitrum Mainnet | [0x4231Dd9e6717aB9a9ABC5618d8a4Fcf1a432F698](https://arbiscan.io/address/0x4231Dd9e6717aB9a9ABC5618d8a4Fcf1a432F698#code) | **NA** |
| [v2.1.3](https://github.com/Layr-Labs/nitro-contracts/releases/tag/v2.1.3)  | Arbitrum Sepolia | [0x0F7f71c48c6278422736a4a9441cd1d59ba0C2dB](https://sepolia.arbiscan.io/address/0x0F7f71c48c6278422736a4a9441cd1d59ba0C2dB#code) | **NA** |
| [v2.1.3](https://github.com/Layr-Labs/nitro-contracts/releases/tag/v2.1.3)  | Base Mainnet     | [0xcC272c9249d1638B7985eFb84c0E9Cdc001b73F7](https://basescan.org/address/0xcC272c9249d1638B7985eFb84c0E9Cdc001b73F7#code) | **NA** |
| [v2.1.3](https://github.com/Layr-Labs/nitro-contracts/releases/tag/v2.1.3)  | Base Sepolia     | [0xfc2a0CD44A6CB0b72d5a7F8Db2C044F62db50781](https://sepolia.basescan.org/address/0xfc2a0CD44A6CB0b72d5a7F8Db2C044F62db50781) | **NA**


**`SequencerInbox` 내에서 V1 EigenDA blob을 검증해 sequencer에 대한 신뢰 가정을 제거하려면 cert verifier 주소가 필요하다. 이 주소는 orbit sdk의 `params` 영역에 설정할 수 있다.**

### 호스팅된 `NitroContractsEigenDA2Point1Point3UpgradeAction` 으로 마이그레이션 또는 업그레이드
직접 실행 또는 배포 방법은 [여기](https://github.com/Layr-Labs/orbit-actions/tree/main/scripts/foundry/contract-upgrades/eigenda-v2.1.3)에서 확인한다. 아래 모든 contract는 consensus-eigenda-v32.3 WASM [artifact](https://github.com/Layr-Labs/nitro/releases/tag/consensus-eigenda-v32.3)로의 업그레이드가 활성화되어 있다.

| Network          | Address                                      | Cert Verification Enabled | Explorer Link                                                                                    | MaxDataSize |
| ---------------- | -------------------------------------------- | --------------------- | ------------------------------------------------------------------------------------------------ | ----------- |
| **Eth Mainnet**  | `0x128f64272804f17502A189A862449F2C8d6B5448` | true                 | [Etherscan](https://etherscan.io/address/0x128f64272804f17502A189A862449F2C8d6B5448)     | 117964      |
| **Eth Sepolia**  | `0x8b4b9BA6715aB493073d9e8426f3E9eb8404f12a` | true                 | [Etherscan](https://sepolia.etherscan.io/address/0x8b4b9BA6715aB493073d9e8426f3E9eb8404f12a)     | 117964      |
| **Base Sepolia** | `0x28303a297e31ac5376047b128867e9D339B58Bf0` | false                 | [BaseScan](https://sepolia.basescan.org/address/0x28303a297e31ac5376047b128867e9D339B58Bf0#code) | 104857      |
| **Arbitrum One** | `0xf099152D84dd3473442Ee659276b6d374c008c5a` | false                  | [Arbiscan](https://sepolia.arbiscan.io/address/0xf099152D84dd3473442Ee659276b6d374c008c5a)       | 104857      |


## UI로 Testnet에 Rollup 배포

배포된 Rollup creator와 직접 상호작용할 수도 있지만, 더 친절한 devx와 사용하기 쉬운 config를 위해 우리의 [orbit chain deployment portal](https://orbit.eigenda.xyz/)을 사용해 rollup을 배포할 것을 권장한다. 현재 UI는 다음 testnet만 지원한다:
- Ethereum Sepolia
- Arbitrum Sepolia
- Base Sepolia


### 문제 해결
nitro setup script node가 `error getting latest batch count: no contract code at given address` 경고를 만난다면, 먼저 다음을 확인한다:
- `/config/orbitSetupScriptConfig` 의 `SequencerInbox` 항목이 성공적으로 배포된 contract를 가리키는지
- RPC provider가 충분히 안정적인지. 무료 및 공용 RPC provider 사용 시 일시적 오류는 흔하다.

## Token Bridge (토큰 브릿지)

Arbitrum token bridge는 ERC-20 자산의 L1 ↔ L2 bridging을 지원하도록 활성화할 수 있다. token bridge는 기존 L1 ↔ L2 native bridge 위에 얹은 wrapper이므로 활성화에 별다른 변경이 필요 없다. 또한 Offchain labs가 유지하는 [기존](https://docs.arbitrum.io/build-decentralized-apps/reference/contract-addresses#token-bridge-smart-contracts) token bridge creator를 활용해 EigenDA 통합 inbox 위에 token bridge를 배포할 수 있다.
