
# Hoodi

EigenDA Hoodi testnet은 operator용 EigenDA testnet이다.

## Quick Links (바로가기)

* [AVS Page][2]

## 사양 (Specs)

| 속성 | 값 |
| --- | --- |
| Disperser Address | `disperser-hoodi.eigenda.xyz:443` |
| Churner Address | `churner-hoodi.eigenda.xyz:443` |
| Batch Dispersal Interval | 3초마다 (네트워크 상태에 따라 달라질 수 있음) |
| Max Blob Size | 16 MiB |
| Default Blob Dispersal Rate limit | 100초당 blob 1개 이하 |
| Default Blob Size Rate Limit | 10분당 1.8 MiB 이하 |
| Stake Sync (AVS-Sync) Interval | 24시간마다 |
| Ejection Cooldown Period | 24시간 |

## Contract 주소

| Contract | Address |
| --- | --- |
| EigenDADirectory | [0x5a44e56e88abcf610c68340c6814ae7f5c4369fd](https://hoodi.etherscan.io/address/0x5a44e56e88abcf610c68340c6814ae7f5c4369fd#readProxyContract) |

다른 모든 contract는 이제 EigenDADirectory contract 안에서 추적된다:
1. 위 etherscan 링크를 클릭한다.
2. "Contract" 버튼을 클릭한다.
3. "Read as Proxy" 버튼을 클릭한다.
4. "getAllNames()" 함수를 클릭하면 등록된 모든 contract의 이름을 볼 수 있다.
5. 이름을 사용해 "getAddress()" 함수로 특정 contract 주소를 얻는다.

![](../assets/eigenda/eigenda-directory-etherscan.png)

## Quorum

| Quorum Number | Stake Minimum | Token |
| --- | --- | --- |
| 0 | 32 | [ETH, LSTs](https://hoodi.eigenlayer.xyz/token) |
| 1 | 1 | [bEIGEN](https://hoodi.eigenlayer.xyz/token/bEIGEN) |

참고: EIGEN을 restaking하면 자동으로 bEIGEN으로 변환된다.

[2]: https://hoodi.eigenlayer.xyz/avs/eigenda
