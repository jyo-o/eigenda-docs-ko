
# OP Stack 과 EigenDA

[OP Stack](https://github.com/ethereum-optimism/optimism)은 [Optimism](https://l2beat.com/scaling/projects/op-mainnet) rollup을 구동하는 software 컴포넌트 집합으로, 독립적으로 배포해 third-party rollup을 운영할 수도 있다.

기본 동작에서, OP Stack sequencer의 [op-batcher](https://github.com/ethereum-optimism/optimism/tree/develop/op-batcher)는 canonical L2 chain에 포함된 transaction에 대한 commit으로 Ethereum에 calldata 또는 4844 blob 형태의 batch를 기록한다. Alt-DA 모드에서는 op-batcher와 op-node(validator)가 third-party HTTP proxy server와 통신하도록 설정되어 tx batch를 DA에 쓰고(op-batcher) DA에서 읽는다(op-node). 이 시스템들이 어떻게 상호작용하는지에 대한 더 자세한 분석은 Optimism의 Alt-DA [spec](https://specs.optimism.io/experimental/alt-da.html)을 참고한다.

이 server spec을 구현하기 위해, EigenDA는 OP Stack sequencer와 full node 옆에서 dependency로 실행되며 EigenDA disperser와 안전하게 통신하는 [EigenDA Proxy](../../eigenda-proxy/eigenda-proxy.md)를 제공한다.

## OP Fork (포크)

upstream OP Stack에 빠진 [3가지 기능](https://github.com/Layr-Labs/optimism?tab=readme-ov-file#fork-features)을 제공하기 위해 현재 OP Stack의 [fork](https://github.com/Layr-Labs/optimism)를 유지하고 있다:
1. Performance: parallel blob 제출을 통한 high-throughput rollup 지원 ([Release 2](https://github.com/Layr-Labs/optimism/releases/tag/op-node%2Fv1.11.1-eigenda.2) 참고)
2. Liveness: EigenDA가 가용하지 않을 경우 Ethereum calldata로 failover 제공 ([Release 1](https://github.com/Layr-Labs/optimism/releases/tag/op-node%2Fv1.11.1-eigenda.1) 참고)
3. Safety: op의 [rust derivation pipeline](https://github.com/op-rs/kona)에 대한 [hokulea](https://github.com/Layr-Labs/hokulea) 확장을 사용해 완전한 보안 통합을 작업 중

## Kurtosis Devnet (개발 네트워크)

EigenDA 기반 OP rollup을 빠르게 둘러보기 위해 op의 kurtosis-devnet을 [확장](https://github.com/Layr-Labs/optimism/tree/eigenda-develop/kurtosis-devnet)했다. repo를 clone한 뒤 해당 디렉토리로 이동한다:
```bash
git clone git@github.com:Layr-Labs/optimism.git
cd optimism/kurtosis-devnet
```
그 다음 devnet 관련 just 명령들을 살펴본다:
```bash
$ just --list
  [...] # other commands
  [eigenda]
  eigenda-devnet-add-tx-fuzzer ENCLAVE_NAME="eigenda-devnet" *ARGS=""
  eigenda-devnet-clean ENCLAVE_NAME="eigenda-devnet"
  eigenda-devnet-configs ENCLAVE_NAME="eigenda-devnet"
  eigenda-devnet-failback ENCLAVE_NAME="eigenda-devnet"
  eigenda-devnet-failover ENCLAVE_NAME="eigenda-devnet" # to failover to ethDA. Use `eigenda-devnet-failback` to revert.
  eigenda-devnet-grafana ENCLAVE_NAME="eigenda-devnet"
  eigenda-devnet-restart-batcher ENCLAVE_NAME="eigenda-devnet" # Restart batcher with new flags or image.
  eigenda-devnet-start VALUES_FILE="eigenda-template-values/memstore-concurrent-large-blobs.json" ENCLAVE_PREFIX="eigenda" # We also start a tx-fuzzer separately, since the optimism-package doesn't currently have that configurable as part of its package.
  eigenda-devnet-sync-status ENCLAVE_NAME="eigenda-devnet"
  eigenda-devnet-test-sepolia *ARGS=""               # Take a look at how CI does it in .github/workflows/kurtosis-devnet.yml .
  eigenda-devnet-test-memstore *ARGS=""              # meaning with a config file in eigenda-template-values/memstore-* .
```

`just eigenda-devnet-start` 를 실행하면 devnet이 시작되며 memstore 모드의 [eigenda-proxy](../../eigenda-proxy/eigenda-proxy.md)가 EigenDA를 시뮬레이션한다. 실제 EigenDA [Sepolia](../../../networks/sepolia.md) testnet과 상호작용하려면 `just eigenda-devnet-start "eigenda-template-values/sepolia-concurrent-small-blobs.json"` 를 실행한다. 그 [config file](https://github.com/Layr-Labs/optimism/blob/eigenda-develop/kurtosis-devnet/eigenda-template-values/sepolia-v2-concurrent-small-blobs.json)에서 누락된 secret 값들을 채워야 한다: `eigenda-proxy.secrets.eigenda.signer-private-key-hex`, `eigenda-proxy.secrets.eigenda.v2.signer-payment-key-hex`, `eigenda-proxy.secrets.eigenda.eth_rpc`. 다른 값을 수정하거나, 필요하다면 kurtosis eigenda [template file](https://github.com/Layr-Labs/optimism/blob/e1d636081550caacae42d88b79404899f0e45888/kurtosis-devnet/eigenda.yaml)을 직접 수정해도 된다.

## 배포 (Deploying)

공식 OP [deployment docs](https://docs.optimism.io/builders/chain-operators/tutorials/create-l2-rollup)에 따라 OP Stack을 배포한다. 우리 fork는 현재 op-batcher와 op-node만 수정하므로, 아래의 op-batcher와 op-node 배포 지침도 함께 읽는다.

### Rollup 설정 (Rollup Config)

[chain 초기화](https://docs.optimism.io/operators/chain-operators/tools/op-deployer#init-configure-your-chain)에 op-deployer를 사용한다면, intent 파일에서 [DangerousAltDAConfig](https://github.com/ethereum-optimism/optimism/blob/d474182026cb0a56874c1c2658849f7a1951b55d/op-deployer/pkg/deployer/state/chain_intent.go#L69) 필드를 반드시 설정한다 (OP의 FUD에 겁먹지 말 것 — EigenDA rollup은 물지 않는다):

```toml
[[chains]]
  # Your chain's ID, encoded as a 32-byte hex string
  id = "0x00000000000000000000000000000000000000000000000000000a25406f3e60"
  # Only called dangerous because it hasn't been tested by OP Labs
  [chains.dangerousAltDAConfig]
    useAltDA = true
    daCommitmentType = "GenericCommitment" # instead of KeccakCommitment
    daChallengeWindow = 300  # unused random value
    daResolveWindow = 300 # unused random value
```

`GenericCommitment` 를 사용하면 DAChallengeContract 배포는 건너뛰며 (그 이유는 아래 [분석](#da-challenge-contract) 참고), 다음 alt_da 필드를 가진 `rollup.json` 설정 파일이 생성된다:

```json
{
  "alt_da": {
    "da_commitment_type": "GenericCommitment",
    "da_challenge_contract_address": "0x0000000000000000000000000000000000000000",
    "da_challenge_window": 300,
    "da_resolve_window": 300
  }
}
```

op-deployer를 사용하지 않고 이 파일을 직접 생성하는 경우, [keccak commitment](https://specs.optimism.io/experimental/alt-da.html#input-commitment-submission) 대신 generic commitment를 쓰도록 `da_commitment_type` 을 반드시 설정한다. 다른 값들은 의미는 없지만 어떤 값이든 설정되어 있어야 한다.

:::note
batch 파라미터를 설정할 때, encoding 오버헤드와 비용 영향을 이해하기 위해 [batch sizing reference](https://github.com/Layr-Labs/eigenda/blob/master/encoding/utils/codec/README.md)를 참고한다.
:::

### EigenDA Proxy 배포

최신 정보는 [eigenda-proxy](https://github.com/Layr-Labs/eigenda/tree/master/api/proxy#eigenda-proxy-) 사용자 가이드를 참고한다.

proxy가 제공하는 다양한 [feature](https://github.com/Layr-Labs/eigenda/tree/master/api/proxy#features-and-configuration-options-flagsenv-vars)를 읽어 다양한 플래그 옵션을 이해한다. Proxy 설정에 필요한 env var이 담긴 예시 [config](https://github.com/Layr-Labs/eigenda/blob/master/api/proxy/.env.example)을 제공한다.

### OP Node 배포

op-node와 eigenda-proxy 사이의 올바른 통신을 보장하기 위해 다음 env config 값을 설정한다. `{EIGENDA_PROXY_URL}` 은 본인의 EigenDA Proxy server URL로 바꾼다.

- `OP_NODE_ROLLUP_CONFIG={ROLLUP_CONFIG_PATH}`: [위](#rollup-config)에서 언급한 `rollup.json` 파일 경로
- `OP_NODE_ALTDA_ENABLED=true`
- `OP_NODE_ALTDA_DA_SERVICE=true`: 이 이상한 이름은 keccak commitment 대신 generic commitment를 쓰겠다는 뜻이다.
- `OP_NODE_ALTDA_VERIFY_ON_READ=false`: 또 다른 이상한 이름이며 keccak commitment에서만 사용된다.
- `OP_NODE_ALTDA_DA_SERVER={EIGENDA_PROXY_URL}`

### OP Batcher 배포

OP Batcher와 EigenDA Proxy 사이의 올바른 통신을 보장하기 위해 다음 env config 값을 설정한다. `{EIGENDA_PROXY_URL}` 은 본인의 EigenDA Proxy server URL로 바꾼다.

- `OP_BATCHER_ALTDA_ENABLED=true`
- `OP_BATCHER_ALTDA_DA_SERVICE=true`: 이 이상한 이름은 keccak commitment 대신 generic commitment를 쓰겠다는 뜻이다.
- `OP_BATCHER_ALTDA_VERIFY_ON_READ=false`: 또 다른 이상한 이름이며 keccak commitment에서만 사용된다.
- `OP_BATCHER_ALTDA_DA_SERVER={EIGENDA_PROXY_URL}`
- `OP_BATCHER_TARGET_NUM_FRAMES=8`
- `OP_BATCHER_MAX_L1_TX_SIZE_BYTES=120000`: default value
- `OP_BATCHER_ALTDA_MAX_CONCURRENT_DA_REQUESTS=10`

EigenDA에 제출되는 각 blob은 `OP_BATCHER_TARGET_NUM_FRAMES` 개의 frame으로 구성되며, 각 frame의 크기는 `OP_BATCHER_MAX_L1_TX_SIZE_BYTES` 다. 위 값은 약 1MiB의 blob을 제출한다. [failover](#failover)가 필요할 때 frame이 단일 transaction 안에 들어가야 하므로 (최대 128KiB), `OP_BATCHER_MAX_L1_TX_SIZE_BYTES` 를 default보다 크게 설정하지 않는 것을 권장한다.

EigenDA dispersal의 p99 latency는 약 10초이므로, 1MiB/s의 throughput을 달성하려면 그 10초를 채울 수 있도록 10개의 pipelined 요청을 허용하는 `OP_BATCHER_ALTDA_MAX_CONCURRENT_DA_REQUESTS=10` 으로 설정한다.

#### **Failover**

Failover는 이 [PR](https://github.com/Layr-Labs/optimism/pull/34)에서 추가되었으며, batcher가 자동으로 지원한다. 각 channel은 먼저 proxy를 통해 EigenDA에 dispersal을 시도한다. `503` HTTP error를 받으면 그 channel은 failover되어 ethereum에 calldata로 제출된다. proxy가 `503` error를 반환하는 시점을 설정하려면 Proxy README의 [failover signals](https://github.com/Layr-Labs/eigenda/tree/master/api/proxy#failover-signals-) 절을 참고한다.

## 보안 보장

위 setup은 EigenDA disperser에 불필요한 신뢰 가정을 추가하지 않으면서 [trusted integration](../integrations-overview.md#trusted-integration) 수준의 보안 보장을 제공한다.

### DA Challenge Contract

OP의 Alt-DA spec에는 [DA challenge contract](https://specs.optimism.io/experimental/alt-da.html#data-availability-challenge-contract)가 포함되어 있어, L2 자산 보유자가 sequencer 또는 DA 네트워크가 실행하는 data withholding 공격을 막을 수 있도록 한다. EigenDA는 이 challenge contract를 사용하지 않는다. high-throughput 대역폭을 Ethereum에 업로드하는 것은 물리적으로 불가능하며, 저 throughput rollup이라 해도 challenge contract는 경제적으로 타당하지 않기 때문이다. challenge contract의 경제적 분석은 [l2beat의 redstone rollup 분석](https://l2beat.com/scaling/projects/redstone#da-layer-risk-analysis) 또는 donnoh의 [Universal Plasma and DA challenges](https://ethresear.ch/t/universal-plasma-and-da-challenges/18629) 글을 참고한다.

즉, 우리 op stack fork가 (현재는 ethereum calldata로만 failover 가능한데) keccak commitment로의 failover를 구현하더라도 challenge contract 사용이 추가 보안 보장을 제공하지 않으므로, 모든 EigenDA OP rollup이 [rollup.json](#deploying-op-node) config에서 GenericCommitment를 사용할 것을 권장한다.
