
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Docker로 노드 실행하기 (Run with Docker)

Operator Setup Repository에는 docker와 docker compose로 EigenDA node를 손쉽게 운영할 수 있는 다양한 템플릿이 포함되어 있다. 다른 환경을 원하는 operator는 이 docker-compose 기반 setup을 본인 커스텀 setup의 템플릿으로 활용할 수 있다.

다음 절차를 진행하기 전에 시스템에 docker가 설치되어 있는지 반드시 확인한다.
- [Docker Engine on Linux](https://docs.docker.com/engine/install/ubuntu/).


## EigenDA Node 설정

#### Operator Setup Repo 복제 및 환경 변수 설정

다음 명령으로 Operator Setup Repo를 clone하고, 제공된 템플릿에서 새 환경 파일을 생성한다.
`srs_setup.sh` 스크립트는 EigenDA node가 KZG 검증에 사용하는 structured reference string (SRS, 약 8GB)을 `eigenda-operator-setup/resources` 디렉토리로 다운로드한다.


<Tabs groupId="network">
  <TabItem value="mainnet" label="Mainnet">
    ```
    git clone https://github.com/Layr-Labs/eigenda-operator-setup.git
    cd eigenda-operator-setup && ./srs_setup.sh
    cd mainnet && cp .env.example .env
    ```
  </TabItem>
  <TabItem value="hoodi" label="Hoodi">
    ```
    git clone https://github.com/Layr-Labs/eigenda-operator-setup.git
    cd eigenda-operator-setup && ./srs_setup.sh
    cd hoodi && cp .env.example .env
    ```
  </TabItem>
</Tabs>

제공된 `.env` 에는 node에 대한 기본 설정 값이 다수 들어 있다. `TODO`로 표시된 모든 영역은 사용자 환경에 맞춰 갱신해야 한다. 다음 절의 절차를 따라 ECDSA private key 없이 운영되도록 node를 구성하는 것을 권장한다.

> ℹ️ **Info**
>
> [여기](./registration.mdx)에 설명된 대로, `.env` 작성에 필요한 ECDSA 및 BLS key는 EigenLayer operator 등록 과정에서 얻을 수 있다.


> ⚠️ **Warning**
>
> docker-compose.yml 파일은 수정하지 말 것. 이 파일을 수정하면 예기치 않은 동작이 발생할 수 있다.


#### 로컬 스토리지 위치 설정

`$USER_HOME`, `$EIGENLAYER_HOME`, `$EIGENDA_HOME` 가 환경 파일에 올바르게 설정되어 있는지, 모든 폴더가 예상대로 존재하는지 확인한다.
```
source .env
ls $USER_HOME $EIGENLAYER_HOME $EIGENDA_HOME
```

기본적으로 EigenDA node는 다음 위치를 각각 로그 저장과 blob shard 저장에 사용한다.

```
NODE_LOG_PATH_HOST=${EIGENDA_HOME}/logs
NODE_DB_PATH_HOST=${EIGENDA_HOME}/db
```

이 위치는 [System Requirements](../requirements/system-requirements.md#node-storage-requirements)에 명시된 대로 충분한 용량의 고성능 SSD 스토리지여야 한다. 또한 docker user에게 올바른 쓰기 권한이 있는 폴더가 실제로 존재하는지 확인한다:

```
mkdir -p ${NODE_LOG_PATH_HOST}
mkdir -p ${NODE_DB_PATH_HOST}
```

참고: 기본 환경 setup은 `eigend-operator-setup` repo가 `$USER_HOME` 디렉토리에 clone되어 있다고 가정하며, node는 운영에 필요한 여러 파일을 이 위치에서 찾는다:

```
NODE_G1_PATH_HOST=${USER_HOME}/eigenda-operator-setup/resources/g1.point
NODE_G2_PATH_HOST=${USER_HOME}/eigenda-operator-setup/resources/g2.point.powerOf2
NODE_CACHE_PATH_HOST=${USER_HOME}/eigenda-operator-setup/resources/cache
```

#### (권장) ECDSA key 없이 노드를 실행하도록 설정

[EigenDA v0.6.1](https://github.com/Layr-Labs/eigenda-operator-setup/releases/tag/v0.6.1)에서, operator의 ECDSA key 없이 node를 실행할 수 있도록 구성하는 기능이 추가되었다.
node는 attestation 목적으로 BLS key에는 여전히 접근할 수 있어야 한다.
>**_NOTE:_** EigenDA에 opt-in 하려면 ECDSA와 BLS key가 모두 필요하다.

setup을 사용해 이 기능을 활성화하려면 다음 명령을 따른다:
* `docker-compose.yml` 파일에서 `"${NODE_ECDSA_KEY_FILE_HOST}:/app/operator_keys/ecdsa_key.json:readonly"` 마운트를 제거한다.
* `.env` 파일의 `NODE_ECDSA_KEY_FILE`를 비워둔다.
* `.env` 파일의 `NODE_ECDSA_KEY_PASSWORD`를 비워둔다.
* `.env` 파일의 `NODE_PUBLIC_IP_CHECK_INTERVAL`를 `0` 으로 설정한다 (이 플래그는 IP가 바뀌면 onchain에 갱신하기 위한 것이므로, IP가 바뀐다면 직접 갱신해야 한다).

## 네트워크 설정

EigenDA node는 데이터 저장 및 제공 책임을 수행하기 위해 다양한 주체로부터 적절히 도달 가능해야 한다.

### Retrieval 설정

사용자가 node로부터 데이터를 retrieve할 수 있도록, retrieval port에 대한 접근을 열어야 한다.

`.env` 의 `NODE_RETRIEVAL_PORT` 로 지정된 port가 공용 인터넷에서 접근 가능하도록 설정한다.

기본 setup에서는 이 port가 NGINX reverse proxy로 서빙되며, DoS 공격에 대한 기본적인 보호 차원에서 rate limiting이 적용된다. 커스텀 setup으로 운영하기로 결정한다면, 동일한 보호 장치를 본인의 인프라로 구현해야 한다.

### Dispersal 설정

> ⚠️ **Warning**
>
> 이 절의 지침을 따라 node가 DOS 공격에 노출되지 않도록 하는 것이 중요하다.


`.env` 의 `NODE_DISPERSAL_PORT` 로 지정된 port는 EigenLabs가 호스팅하는 disperser에서만 도달 가능해야 한다.

방화벽, security group, 기타 네트워크 설정을 통해 이 port가 다음 IP 주소에서만 접근 가능하도록 구성한다:


<Tabs groupId="network">
  <TabItem value="mainnet" label="Mainnet">
  - `3.216.127.6/32`
  - `3.225.189.232/32`
  - `52.202.222.39/32`
  </TabItem>
  <TabItem value="hoodi" label="Hoodi">
  - `18.209.198.153/32`
  </TabItem>
</Tabs>

<!-- ### Node API Port Setup:

In order to consolidate operator metrics to measure the health of the network, please also open NODE_API_PORT in .env to the internet if possible. Please see Node API Spec for more detail on the data made available via this port. -->


## 노드 실행

### Docker Compose로 EigenDA 시작 및 정지

다음 명령으로 docker container를 시작한다:

```
docker compose up -d
```

이 명령은 node와 nginx container를 시작한다. `docker ps` 를 실행하면 모든 container의 상태가 “Up” 으로 표시되고 port가 할당되어 있는 것이 보여야 한다.

node를 정지하려면 다음 명령을 실행한다.

```
docker compose down
```

> ⚠️ **Warning**
>
> [quorum에 등록](./registration/)한 후에는, deregister하고 [protocol SLA](../requirements/protocol-SLA/)의 모든 요구사항을 충족할 때까지 node를 계속 실행해야 한다.


### EigenDA 로그 보기

다음 명령들 중 하나로 container 로그를 볼 수 있다.

```
docker compose logs -f
docker compose logs -f <container_name>
docker logs -f <container_id>
```

성공적으로 시작되면, DA node는 다음과 유사한 로그를 출력한다:

```
2024/03/22 19:33:28 maxprocs: Leaving GOMAXPROCS=16: CPU quota undefined
2024/03/22 19:33:30 Initializing Node
time=2024-03-22T19:33:34.503Z level=DEBUG source=/app/core/eth/tx.go:791 msg=Addresses blsOperatorStateRetrieverAddr=0xB4baAfee917fb4449f5ec64804217bccE9f46C67 eigenDAServiceManagerAddr=0xD4A7E1Bd8015057293f0D0A557088c286942e84b registryCoordinatorAddr=0x53012C69A189cfA2D9d29eb6F19B32e0A2EA3490 blsPubkeyRegistryAddr=0x066cF95c1bf0927124DFB8B02B401bc23A79730D
2024/03/22 19:33:34     Reading G1 points (4194304 bytes) takes 5.981866ms
2024/03/22 19:33:37     Parsing takes 3.144064399s
numthread 8
time=2024-03-22T19:33:38.141Z level=INFO source=/go/pkg/mod/github.com/!layr-!labs/eigensdk-go@v0.1.3-0.20240318050546-8d038f135826/metrics/eigenmetrics.go:81 msg="Starting metrics server at port :9092"
time=2024-03-22T19:33:38.141Z level=INFO source=/app/node/node.go:174 msg="Enabled metrics" socket=:9092
time=2024-03-22T19:33:38.141Z level=INFO source=/go/pkg/mod/github.com/!layr-!labs/eigensdk-go@v0.1.3-0.20240318050546-8d038f135826/nodeapi/nodeapi.go:104 msg="Starting node api server at address :9091"
time=2024-03-22T19:33:38.141Z level=INFO source=/app/node/node.go:178 msg="Enabled node api" port=9091
time=2024-03-22T19:33:38.141Z level=INFO source=/app/node/node.go:211 msg="The node has successfully started. Note: if it's not opted in on https://app.eigenlayer.xyz/avs/eigenda, then please follow the EigenDA operator guide section in docs.eigenlayer.xyz to register"
time=2024-03-22T19:33:38.141Z level=INFO source=/go/pkg/mod/github.com/!layr-!labs/eigensdk-go@v0.1.3-0.20240318050546-8d038f135826/nodeapi/nodeapi.go:238 msg="node api server running" addr=:9091
time=2024-03-22T19:33:38.141Z level=INFO source=/app/node/node.go:385 msg="Start checkRegisteredNodeIpOnChain goroutine in background to subscribe the operator socket change events onchain"
time=2024-03-22T19:33:38.142Z level=INFO source=/app/node/node.go:231 msg="Start expireLoop goroutine in background to periodically remove expired batches on the node"
time=2024-03-22T19:33:38.142Z level=INFO source=/app/node/node.go:408 msg="Start checkCurrentNodeIp goroutine in background to detect the current public IP of the operator node"
time=2024-03-22T19:33:38.142Z level=INFO source=/app/node/grpc/server.go:123 msg=port 32004=address [::]:32004="GRPC Listening"
time=2024-03-22T19:33:38.142Z level=INFO source=/app/node/grpc/server.go:99 msg=port 32005=address [::]:32005="GRPC Listening"
```
