

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Operator 등록하기 (Registration)

Operator는 EigenDA의 한 개 이상 quorum에 등록되기 전까지는 EigenDA disperser로부터 트래픽을 받지 않는다.
[delegation 요구사항](../requirements/delegation-requirements/)에서 다룬 것처럼, EigenDA quorum에 등록하려면 operator가 [EigenLayer operator로 등록](../../../eigenlayer/operators/howto/registeroperators/operator-installation.md)되어 있어야 하고, 등록하려는 각 quorum에 최소량의 stake가 위임되어 있어야 한다.

> ℹ️ **Info**
>
> quorum에서 ejection된 후에는 mainnet에서 3일, testnet에서 1일의 cooldown이 있다.

## EigenDA Quorum에 Opt-in

한 개 이상의 [quorum](https://docs.eigenlayer.xyz/eigenlayer/operator-guides/operator-introduction#quorums)에 opt-in할 수 있는 delegation 요구사항을 충족한다면, `eigenda-operator-setup` 폴더에서 다음 명령을 실행해 원하는 quorum에 opt-in할 수 있다:


<Tabs groupId="network">
  <TabItem value="mainnet" label="Mainnet">
    ```
    cd mainnet

    ./run.sh opt-in <quorum>

    # for opting in to quorum 0:
    ./run.sh opt-in 0

    # for opting in to quorum 0 and 1:
    ./run.sh opt-in 0,1 
    ```

    참고: EigenDA는 Mainnet에서 두 개의 [quorum](https://docs.eigenlayer.xyz/eigenda/networks/mainnet)을 운영한다: Restaked ETH (Native 및 LST Restaked ETH 포함) 와 EIGEN. EigenDA는 Operator가 단일 quorum 또는 두 quorum 모두(dual-quorum)에 opt-in할 수 있도록 허용한다.
    - ETH (Native & LST) Quorum:  `0`
    - EIGEN Quorum: `1`
    - Dual Quorum: `0,1`

    opt-in하려는 quorum만 지정하면 된다. 예를 들어 이미 quorum `0` 에 등록되어 있고 quorum `1` 에 추가로 opt-in하려면, 다시 opt-in할 때 `<quorum>` 을 `1` 로만 지정하면 된다.

    두 quorum ('`0,1`')에 동시에 opt-in을 시도하면, 두 quorum 모두에서 active Operator set에 진입할 수 있는 충분한 TVL이 있어야 한다. 그렇지 않으면 두 quorum 모두에 대한 opt-in 시도가 실패한다. 두 quorum에 동시에 opt-in하는 시도는 "all or nothing" 방식이다.


  </TabItem>
  <TabItem value="hoodi" label="Hoodi">
    ```
    cd hoodi

    ./run.sh opt-in <quorum>

    # for opting in to quorum 0:
    ./run.sh opt-in 0

    # for opting in to quorum 0 and 1:
    ./run.sh opt-in 0,1
    ```

    참고: EigenDA는 Hoodi에서 두 개의 [quorum](https://docs.eigenlayer.xyz/eigenda/networks/hoodi)을 운영한다: Restaked ETH (Native 및 LST Restaked ETH 포함) 와 Restaked EIGEN/bEIGEN. EigenDA는 Operator가 단일 quorum이나 모든 quorum에 동시에 opt-in할 수 있도록 허용한다.
    - ETH (Native & LST) Quorum:  `0`
    - EIGEN (EIGEN/bEIGEN) Quorum: `1`

    opt-in하려는 quorum만 지정하면 된다. 예를 들어 이미 quorum `0` 에 등록되어 있고 quorum `1` 에 추가로 opt-in하려면, 다시 opt-in할 때 `<quorum>` 을 `1` 로만 지정하면 된다.

    여러 quorum ('`0,1,2`')에 동시에 opt-in을 시도하면, 모든 quorum의 active Operator set에 진입할 수 있는 충분한 TVL이 있어야 한다. 그렇지 않으면 모든 quorum에 대한 opt-in 시도가 실패한다. 여러 quorum에 동시에 opt-in하는 시도는 "all or nothing" 방식이다.

  </TabItem>
</Tabs>


https://docs.eigenda.xyz/networks/
https://docs.eigenlayer.xyz/eigenlayer/operator-guides/operator-introduction#quorums
> ⚠️ **Warning**
>
> EigenDA AVS에 opt-in한 후에 delegation이 발생했다면, operator는 stake가 sync될 때까지 기다려야 한다. EigenLayer의 AVS-Sync 컴포넌트는 일정 주기로 실행되어 각 operator에 대한 onchain delegation 합계를 갱신한다. 충분한 위임 stake가 있는데도 opt-in이 되지 않는다면, 최소 stake sync에 필요한 시간만큼 기다린 후 opt-in을 재시도한다. 이 sync 주기는 네트워크마다 다르며 자세한 내용은 [Mainnet](../../networks/mainnet)과 [Hoodi](../../networks/hoodi)에서 확인할 수 있다.



스크립트는 [.env](https://github.com/Layr-Labs/eigenda-operator-setup/blob/31d99e2aa67962878969b81a15c7e8d13ee69750/mainnet/.env.example#L71)의 `NODE_HOSTNAME` 을 현재 IP로 사용한다.

operator가 EigenDA opt-in에 실패하거나 Churn Approver에 의해 ejection되었다면, rate limiting 임계값이 지난 후 opt-in 명령을 다시 실행할 수 있다. 현재 rate limiting 임계값은 5분이다.

“error: failed to request churn approval .. Rate Limit Exceeded” 오류가 발생하면 임계값이 지난 후 재시도한다. “insufficient funds” 오류가 발생하면, Operator의 위임 TVL을 필요한 최소치로 늘린 뒤 임계값이 지나면 재시도한다.

> ℹ️ **Info**
>
> 위 명령들이 실행하는 등록 절차에 대한 자세한 내용은 [Registration Protocol Overview](../registration-protocol.md)에서 확인할 수 있다.


## 네트워크 트래픽 확인

EigenDA는 reorg 위험을 피하기 위해 현재 chain head보다 75 block (15분) 뒤의 operator state를 사용한다.
quorum에 성공적으로 opt-in한 뒤 약 15분이 지나면, node가 네트워크로부터 batch를 수신, 검증, 저장하고 있음을 보여주는 다음과 같은 로그가 보이기 시작해야 한다:

```
Batch verify 1 frames of 256 symbols out of 1 blobs
time=2024-03-22T19:34:39.858Z level=DEBUG source=/app/node/node.go:330 msg="Validate batch took" duration:=96.155565ms
time=2024-03-22T19:34:39.858Z level=DEBUG source=/app/node/node.go:340 msg="Store batch took" duration:=0s
time=2024-03-22T19:34:39.859Z level=DEBUG source=/app/node/node.go:346 msg="Signed batch header hash" pubkey=0x00cea342f086977a33b3f1bba57d09c6cdf8eaf20b9dec856dc874ab65414b6e2377a91ab3bc2360224f3ba071eb4753da650e957d9c0535b14922609a9ff052150595f3a89c06e87a78d3e3ebad09771f181b632bd971c1d58deb3e1fde9397087c1cc1097c48b1e900d418ef43538a8abdccde72921c3148ae4de5e0f39ef3
time=2024-03-22T19:34:39.859Z level=DEBUG source=/app/node/node.go:349 msg="Sign batch took" duration=1.32679ms
time=2024-03-22T19:34:39.860Z level=INFO source=/app/node/node.go:351 msg="StoreChunks succeeded"
time=2024-03-22T19:34:39.860Z level=DEBUG source=/app/node/node.go:353 msg="Exiting process batch" duration=97.815499ms
```

## Quorum 목록 확인

다음 명령으로 node가 현재 opt-in된 quorum 목록을 확인할 수 있다.

```
./run.sh list-quorums
```

## EigenDA Quorum에서 Opt-Out

> ⚠️ **Warning**
>
> 현재 (또는 의도한) quorum에서만 opt-out하도록 주의한다.


다음 명령으로 EigenDA AVS에서 opt-out할 수 있다:

```
./run.sh opt-out <quorum>

# for opting out to quorum 0:
./run.sh opt-out 0

# for opting out to quorum 0 and 1:
./run.sh opt-out 0,1 
```

## Node Socket 갱신
node 설정 변경에 따라 node Socket을 갱신한다.

예: dispersal 또는 retrieval port가 변경된 경우

> ⚠️ **Warning**
>
> 실행 전에 [.env](https://github.com/Layr-Labs/eigenda-operator-setup/blob/31d99e2aa67962878969b81a15c7e8d13ee69750/mainnet/.env.example) 파일을 반드시 갱신한다.

```
./run.sh update-socket
```
