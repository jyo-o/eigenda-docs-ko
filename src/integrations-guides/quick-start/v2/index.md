
# EigenDA Payment 및 Data Dispersal 가이드 (Quick Start V2)
이 가이드는 Sepolia에서 EigenDA를 사용해 payment를 설정하고 data를 dispersal하는 절차를 안내한다.

> 💡 **Tip**
>
> 이 가이드는 go client를 사용해 payment를 설정하고 data를 dispersal한다. EigenDA API와 통합하는 다른 방법에 대한 정보는 [Overview](../../overview.md)를 참고한다.


## On-Demand Data Dispersal (온디맨드 데이터 분산 전송)
### Onchain 설정
> ℹ️ **Info** — Pre-Requisites
>
> - Ethereum Sepolia testnet의 ETH
> - [Foundry](https://book.getfoundry.sh/getting-started/installation) 설치
> - Sepolia용 RPC URL
> - transaction용 Private key


네트워크에 dispersal하려면 차감할 잔액이 필요하다. EigenDA의 Payment Module에 대해 더 알고 싶다면 [여기](../../../core-concepts/payments.md) 참고 문서를 확인한다.

먼저 Ethereum Sepolia testnet에 ETH가 있는지 확인하고, Payment Vault에 deposit한다. 이후 EigenDA 요청에 따른 비용은 여기서 차감된다.

먼저 Foundry의 `cast` 를 사용해 payment vault에 deposit한다.
> 📝 **Note** — Installation
>
> Foundry를 설치하지 않았다면 [여기](https://book.getfoundry.sh/getting-started/installation)의 설치 명령을 따른다.


다음 명령은 Sepolia의 Payment Vault에 1 ETH를 deposit한다:
> 📝 **Note** — Deposits
>
> 보낼 데이터의 양을 잘 산정한다. payment vault에 deposit된 자금은 환불되지 않는다.


```bash
cast send --rpc-url <YOUR_RPC_URL> \
 --private-key <YOUR_PRIVATE_KEY> \
 0x2E1BDB221E7D6bD9B7b2365208d41A5FD70b24Ed \
 "depositOnDemand(address)" \
<YOUR_ADDRESS> \
 --value 1ether
```
이제 on-demand payment 계정 설정이 끝났으니, EigenDA로 data를 보내보자.

## Data Dispersal (데이터 분산 전송)
### 설정
data를 dispersal하기 위해, EigenDA disperser와 통신할 `Disperser Client` 부터 설정한다.

1. 프로젝트 디렉토리 생성
```bash
mkdir v2disperse
cd v2disperse
```

2. main 파일 생성:
```bash
go mod init
```
### 구현
#### 1. 의존성 임포트 (Dependency Import)
```Golang
package main

import (
	"context"
	"fmt"
	"time"

    "github.com/joho/godotenv"

    

	"github.com/Layr-Labs/eigenda/api/clients/v2"
	authv2 "github.com/Layr-Labs/eigenda/core/auth/v2"
	corev2 "github.com/Layr-Labs/eigenda/core/v2"
	"github.com/Layr-Labs/eigenda/encoding/utils/codec"
)
``` 

#### 2. Disperser Client 생성
> 📝 **Note**
>
> `signer` 는 deposit한 주소와 동일해야 한다.

```Golang
err := godotenv.Load()
	if err != nil {
		fmt.Println("Error loading .env file")
	}
	privateKey := os.Getenv("EIGENDA_AUTH_PK")

signer, err := authv2.NewLocalBlobRequestSigner(privateKey)
disp, err := clients.NewDisperserClient(&clients.DisperserClientConfig{
	Hostname:          "disperser-testnet-sepolia.eigenda.xyz",
	Port:              "443",
	UseSecureGrpcFlag: true,
}, signer, nil, nil)
if err != nil {
	println("Error creating disperser client")
	panic(err)
}
```


#### 3. Context 및 Data 설정
```Golang
ctx, cancel := context.WithTimeout(context.Background(), time.Second*30)
defer cancel()
```

#### 4. 보낼 Data 준비
```Golang
bytesToSend := []byte("Hello World")
bytesToSend = codec.ConvertByPaddingEmptyByte(bytesToSend)
quorums := []uint8{0, 1}
```
#### 5. Data 전송
EigenDA에 data를 보내려면 `DisperseBlob()` 을 호출한다.
```Golang
status, request_id, err := disp.DisperseBlob(ctx, bytesToSend, 0, quorums, 0)
if err != nil {
	panic(err)
}
```

#### 6. Blob status 확인
data와 상호작용하려면 `GetBlobStatus()` 를 호출한다.
```Golang
blobStatus, err = disp.GetBlobStatus(ctx, request_id)
```

이제 EigenDA로 data를 dispersal할 준비가 끝났다. EigenDA 클라이언트와 상호작용하는 추가 예시는 [여기](https://github.com/Layr-Labs/eigenda/tree/master/api/clients/v2/examples) repo 또는 [여기](../../eigenda-proxy/eigenda-proxy.md) EigenDA Proxy 가이드를 참고한다.

