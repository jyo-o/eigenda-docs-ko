
# Mainnet (메인넷)

## Quick Links (바로가기)

* [AVS Page][2]
* [Blob Explorer][1]

## Network Specs (네트워크 사양)

| 속성 | 값 |
| --- | --- |
| Disperser Address | `disperser.eigenda.xyz:443` |
| DataAPI Address | `dataapi.eigenda.xyz` |
| Churner Address | `churner.eigenda.xyz:443` |
| Batch Dispersal Interval | 1초마다 (네트워크 상태에 따라 달라질 수 있음) |
| Min Blob Size | 128 KiB |
| Max Blob Size | 16 MiB |
| Stake Sync (AVS-Sync) Interval | 6일마다 |
| Ejection Cooldown Period | 3일 |

## Contract 주소

| Contract | Address |
| --- | --- |
| EigenDADirectory | [0x64AB2e9A86FA2E183CB6f01B2D4050c1c2dFAad4](https://etherscan.io/address/0x64AB2e9A86FA2E183CB6f01B2D4050c1c2dFAad4) |

다른 모든 contract는 이제 EigenDADirectory contract 안에서 추적된다:
1. 위 etherscan 링크를 클릭한다.
2. "Contract" 버튼을 클릭한다.
3. "Read as Proxy" 버튼을 클릭한다.
4. "getAllNames()" 함수를 클릭하면 등록된 모든 contract의 이름을 볼 수 있다.
5. 이름을 사용해 "getAddress()" 함수로 특정 contract 주소를 얻는다.

![](../assets/eigenda/eigenda-directory-etherscan.png)

## Quorum

| Quorum Number | Token |
| --- | --- |
| 0 | ETH, LSTs |
| 1 | [EIGEN](https://etherscan.io/address/0xec53bF9167f50cDEB3Ae105f56099aaaB9061F83) |
| 2 | [reALT](https://etherscan.io/address/0xF96798F49936EfB1a56F99Ceae924b6B8359afFb) |

[1]: https://blobs.eigenda.xyz/
[2]: https://app.eigenlayer.xyz/avs/0x870679e138bcdf293b7ff14dd44b70fc97e12fc0
