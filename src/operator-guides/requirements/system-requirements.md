# 시스템 요구사항 (System Requirements)

다음 시스템 요구사항은 최적의 노드 성능과 프로토콜 준수를 유지하는 데 필수다.

## 일반 시스템 요구사항 (General System Requirements)

EigenDA 네트워크 설계상, stake가 더 큰 operator는 더 많은 blob chunk/shard를 저장하도록 요청받는다. 그 결과, operator의 노드 요구사항은 자신이 참여하는 quorum 전체의 stake 양에 비례하며, 이를 'Total Work Share' (TWS)라고 한다.

### TWS 동작 방식 (How TWS Works)

operator의 **TWS**는 다음과 같이 계산된다.

- ETH와 EIGEN quorum의 경우, TWS는 두 stake weight 중 **최댓값**이다.
- 추가 quorum이 있다면, 해당 quorum의 stake가 base TWS에 **더해진다**.

**예시**:
- ETH 5% stake + EIGEN 10% → TWS = 10%
- 세 번째 quorum에서 5%를 추가 → TWS = 15%

### 하드웨어 권장 사양 (Hardware Recommendations)

본인의 TWS에 따라 권장 하드웨어를 결정하려면 아래 표를 참고한다.

| 등급 (Class) | Total Work Share (TWS)      | vCPU (10세대 이상) | 메모리 | 디스크 IOPS | 네트워킹 용량 |
| ----- | --------------------------- | ----------------- | ------ | --------- | ------------------- |
| Small | 최대 2%                    | 4                 | 16 GB  | 3,000     | 1 Gbps              |
| Large | 2% 초과                    | 16                | 64 GB  | 12,000    | 10 Gbps             |

---

## 노드 스토리지 요구사항 (Node Storage Requirements)

EigenDA 노드는 네트워크 저장 및 retrieval 작업을 따라잡기 위해 **반드시** 고성능 SSD 스토리지를 프로비저닝해야 한다. `PCIe 4.0 x4 M.2` 또는 `U.2 NVMe` 같은 enterprise grade SSD가 권장된다.

> ⚠️ **Warning**
>
> 적절한 성능을 유지하지 못하면 허용 불가능한 검증 latency가 발생하고 [자동 ejection](protocol-SLA/)이 일어난다.


---

### 처리량과 스토리지 스케일링 (Throughput and Storage Scaling)

EigenDA operator 노드는 throughput 100 MB/s까지 확장되도록 설계되어 있다.

throughput 증가에 따라 **확장이 필요한 자원은 storage뿐**이다. 그 외 시스템은 일반 요구사항에 따라 고정 상태로 유지될 수 있다.

TWS 5%로 최대 용량(100 MB/s)으로 운영하려면 노드가 약 50 TB의 스토리지를 요구하게 된다. 그러나 최대 용량을 미리 프로비저닝하는 것은 일반적으로 비용 부담이 크고 자원 활용이 비효율적이다.

---

### 권장(탄력적) 프로비저닝 전략 (Recommended Elastic Provisioning Strategy)

**선호되는 접근 방식**은 수요에 따라 확장되도록 storage를 탄력적으로 프로비저닝하는 것이다. 이 모델 아래에서는:
- enterprise-grade SSD 스토리지 **8 TB**로 시작한다.
- **연속 14일 기간** 동안 사용률을 50% 이하로 유지한다.

---

### 탄력적 프로비저닝이 어려운 경우 (When Elastic Provisioning Is Not Feasible)

탄력적 프로비저닝이 불가능한 경우, 다음 공식에 따라 최대 용량으로 storage를 프로비저닝해야 한다.
```
Required Storage (TB) = TWS (%) * 1000
```
예시: TWS 5%인 경우, 최대 throughput 용량을 지원하기 위해 50 TB를 프로비저닝한다.


> ℹ️ **Info**
>
> 위 공식은 다음 식에서 도출·간소화된 것이다.
>
> ```
> <Gross System Throughput(MB/s)> * <14 days in seconds> * <% stake>
> ```


## 시스템 업그레이드 (System Upgrades)

operator에게 위임된 stake 양에 따라 시스템 요구사항이 동적으로 확장되므로, 노드 operator는 [Protocol SLA](protocol-SLA/)를 계속 충족하기 위해 때때로 시스템 구성을 업그레이드해야 할 수 있다. 이러한 업그레이드 수행 가이드는 [System Upgrades](../upgrades/system-upgrades/)에서 다룬다.

## IP 안정성 요구사항 (IP Stability Requirements)

현재 EigenDA 프로토콜은 DA 노드가 자신의 IP 주소를 Ethereum L1에 게시하도록 요구하며, 데이터 제공자와 소비자가 이 주소로 노드에 도달할 수 있어야 한다. 따라서 노드 operator는 다음 표에 정리된 IP 주소 안정성 및 도달 가능성 요구사항을 충족할 수 있어야 한다.

|                        | 공유 IP (Shared IP)                                                                                                                  | 전용 IP (Dedicated IP)                                                                                                                                            |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 안정적 IP (Stable IP)   | ❌ 참고: operator가 직접 IP:Port 도달 가능성을 처리한다면 (예: port forwarding 구성) 동작은 한다. | ✅ EigenDA operator에게 가장 이상적인 경우.                                                                                                                |
| 불안정(변경) IP (Unstable IP) | ❌ 참고: operator가 직접 IP:Port 도달 가능성을 처리한다면 (예: port forwarding 구성) 동작은 한다. | ✅ 동작은 하지만, IP가 변경되면 on-chain IP 업데이트를 위한 Ethereum 트랜잭션이 발생해 gas 비용이 들기 때문에 안정적 IP를 가지길 권장한다.|
