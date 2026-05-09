# Arbitrum Orbit 개요 (Overview)

다음은 EigenDA를 활용한 Arbitrum에서 fraud proof와 high throughput을 안전하게 가능하게 하기 위해 적용한 핵심 변경사항에 대한 기술적 정리다. 핵심 caveat과 feature도 함께 다룬다.

## Runbook (운영 절차)

Core Arbitrum은 다양한 software repository와 프로그래밍 언어로 매우 복잡하게 구성되어 있다. 핵심 시스템 흐름을 더 잘 풀어내기 위해, 핵심 테스트 및 시스템 절차를 설명하는 운영 개발자용 [runbook](https://eigen-labs.notion.site/Arbitrum-x-EigenDA-Developer-Runbook-12466062c1a7495ebc1d803169c37644?pvs=4)을 만들었다.

## ETH L2와 L3 배포

EigenDA를 사용하는 Arbitrum L2는 M0 통합이고, L3는 M1이다. 즉 현재 EigenDA를 사용하는 L3는 보안과 throughput 모두에서 다소 저하된다. 다양한 EigenDA rollup 단계에 대한 더 종합적인 개요는 우리의 Integrations [Overview](../integrations-overview.md)를 참고한다.

EigenDA bridging은 현재 Ethereum에서만 지원되므로, L2에 settle하는 L3는:
- `Sequencer Inbox` contract 내에서 cert verification에 의존할 수 없다.
- batch 인증을 위해 eigenda proxy를 통한 disperser confirmation을 기다릴 수 없다.

현재 L3 배포의 경우 다음을 권장한다:

- `EIGENDA_PROXY_EIGENDA_CONFIRMATION_DEPTH` 를 ETH finalization에 가까운 값(예: 64 block 또는 두 consensus epoch)으로 설정한다. reorg된 EigenDA bridge confirmation tx를 rollup이 직접 감지할 수 없기 때문이다. 이 위험은 Ethereum에 settle하는 L2에는 존재하지 않는다. inbox의 EigenDA certificate tx는 EigenDA bridge confirmation tx에 의해 설정되는 `EigenDAServiceManger` storage state를 읽기 때문에, EigenDA bridge confirmation tx의 reorg가 inbox의 EigenDA certificate tx의 reorg로 이어진다.

- 위험을 줄이면서 더 높은 throughput의 L3를 지원하려면, EigenDA proxy 인스턴스를 secondary storage fallback으로 설정할 수 있다. 이렇게 하면 blob certificate가 무효화되더라도 데이터가 부분적으로는 가용하게 유지된다. 다만 정직한 verifier node가 confirmed chain head로부터 sync 중에 reorg가 발생하면 sequencer의 secondary store에 접근할 수 없어 정지될 수 있어, rollup의 신뢰 모델은 일부 손상된다.

### EigenDA Proxy (프록시)

[EigenDA Proxy](https://github.com/Layr-Labs/eigenda-proxy)는 rollup과 EigenDA disperser 사이의 안전하고 최적화된 통신에 사용된다. Arbitrum은 client/server 상호작용과 DA certificate 표현에 [*Simple Commitment Mode*](https://github.com/Layr-Labs/eigenda-proxy?tab=readme-ov-file#simple-commitment-mode)를 사용한다. EigenDA Proxy와 그 보안 기능에 대해 더 알고 싶다면 [여기](../../eigenda-proxy/eigenda-proxy.md)를 참고한다.

### EigenDA에 batch 게시

batch poster config 변경은 모든 batch poster 인스턴스에 일괄 적용되도록 한다. 그렇지 않으면 collective processing 로직 차이로 불일치가 발생할 수 있다. Arbitrum batch poster에 대한 자세한 내용은 다음 개요 [spec](https://hackmd.io/@epociask/ByHk6x_TC)을 참고한다.

**최대 batch 크기 조정**

현재 batch poster는 EigenDA에 batch를 dispersal할 때 기본 최대 `16mib` 를 사용한다. 이 값을 더 낮추려면 node config의 batch poster 영역에서 직접 조정할 수 있다:

```json
    "node": {
        ...
        "batch-poster": {
            "enable": true,
            ...
            "max-eigenda-batch-size": 12_000_000, // 12 MB
        }
    }
```
:::note
batch 파라미터 설정 시, encoding 오버헤드와 비용 영향을 이해하기 위해 [batch sizing reference](https://github.com/Layr-Labs/eigenda/blob/master/encoding/utils/codec/README.md)를 참고한다.
:::

**Failover 활성화**

rollup의 liveness가 EigenDA의 liveness에 의존하지 않도록, EigenDA의 서비스 불가 신호가 있을 때 다른 DA destination(예: AnyTrust, EIP-4844, calldata)로의 opt-in failover를 지원하도록 Arbitrum Nitro batch poster의 로직을 확장했다. 이 로직은 기본적으로 비활성화되어 있으며, batch poster config에 다음 필드를 추가해 활성화할 수 있다:
```json
    "node": {
        ...
        "batch-poster": {
            "enable": true,
            ...
            "enable-eigenda-failover": true, 
        }
    }
```

**NOTE:** 4844 failover는 구현되고 audit되었지만, vanilla Arbitrum에 4844 end-to-end 정확성을 자동으로 검증하는 테스트가 없어 E2E system test로 검증되지는 않았다. 사용은 본인 책임 하에 진행한다. 4844 대신 calldata DA를 사용하려면, node config의 `dangerous` sub-config에 다음 필드를 추가한다:
```json
    "dangerous": {
        "disable-blob-reader": true,
    },
```

Arbitrum failover 설계 방법론에 대해서는 다음 [spec](https://hackmd.io/@epociask/SJUyIZlZkx)을 참고한다.

# Diff 개요

EigenDA를 안전하게 가능하게 하기 위해 많은 핵심 Arbitrum repository를 fork했다. 정확한 변경 내역에 대한 더 기술적인 정리는 다음 개요들을 참고한다:

- [nitro](https://layr-labs.github.io/nitro/)
- [nitro-contracts](https://layr-labs.github.io/nitro-contracts/)
- [nitro-testnode](https://layr-labs.github.io/nitro-testnode/)
- [nitro-go-ethereum](https://layr-labs.github.io/nitro-go-ethereum/)
