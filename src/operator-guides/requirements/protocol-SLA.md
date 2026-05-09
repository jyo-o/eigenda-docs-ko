# 프로토콜 SLA (Protocol SLA)

# Operator Protocol SLA

operator가 EigenDA에 opt-in 하면, 프로토콜이 부과하는 책임을 떠안게 된다. 즉 EigenDA 노드 서비스를 정직하게, 그리고 일정 수준의 가용성과 성능을 갖추고 제공해야 한다. operator는 이 책임에 대해 네트워크에 의해 책임을 추궁받으며, 가용성 결함에 대해서는 ejection 같은 페널티를 받을 수 있다.

## 책임 (Responsibilities)

operator의 책임은 자신이 등록된 모든 quorum에 대한 책임의 합이다. 각 quorum은 attestation에 대해 서로 분리·독립적이므로, 정확한 회계 단위는 각 &lt;operator, quorum> 쌍이다.

다음은 &lt;operator, quorum>의 라이프사이클이며, operator가 quorum에 opt-in 한 시점부터 opt-out 이후까지 각 단계에서의 책임이다.

<img src="/img/eigenda/eigenda-sla-diagram.png" alt="EigenDA SLA Responsibilities" 
  width="90%">
  </img>


### Operator의 책임 (Operator Responsibilities)
operator는 세 가지 기본 책임을 가진다.

1. **자신에게 dispersal된 blob을 검증·저장·attest**
   1. &lt;operator, quorum> 쌍은, 해당 quorum이 dispersal request에서 그 blob에 대해 요청되면 그 blob에 대한 책임을 진다.
   2. &lt;operator, quorum>는 자신이 책임지는 blob 중 적어도 하나가 batch에 포함되면 그 batch에 대한 책임을 진다. &lt;operator, quorum>가 batch에 책임을 질 때 다음을 수행해야 한다.
      1. batch header, 모든 blob의 header, 그리고 자신이 책임지는 batch 안의 blob들을 수신.
      2. 받은 batch와 blob 검증.
      3. 유효하면 데이터 저장.
      4. batch에 서명: 서명은 operator가 attestation(데이터 검증과 저장)을 수행했다는 약속이며, 이후 데이터를 제공할 책임을 진다는 의미를 가진다.
2. **attest한 blob을 (blob의 end of life까지) 저장**
    1. blob은 onchain confirmation 후 `100,800 blocks` (대략 `14일`)에 end of life에 도달한다.
3. **저장한 blob을 제공**
   1. 참고: 엄밀히 말하면 attest(dispersal)와 serve(retrieval)뿐이다. 데이터를 저장하는 것은 serve에 함축되어 있기 때문이다(serve는 저장된 데이터로 뒷받침되어야 한다). storing을 분리한 것은 더 명확히 보이기 위함이다.

참고: operator가 여러 quorum에 opt-in한 경우, 위 내용은 각 quorum에 적용된다.

### 책임 라이프사이클 (Responsibility Lifecycle)

위 책임은 &lt;operator, quorum>의 다음 라이프사이클 단계에 매핑된다.

- **Live:** &lt;operator, quorum>의 등록부터 등록 해제까지 (블록 `A`부터 `B-1`까지)
  - 참고: &lt;operator, quorum>는 블록 `B`를 reference block으로 하는 dispersal에서는 요청되지 않는다. 그 블록이 만들어내는 상태에 &lt;operator, quorum>가 없기 때문이다.
- **전체 책임 (Full responsibility):** `attest+store+serve` (블록 C까지)
  - 참고: &lt;operator, quorum>가 opt out한 후에도 `BLOCK_STALE_MEASURE` 블록 동안 dispersal에 대한 책임이 남는다. dispersal이 과거의 reference block(단, `BLOCK_STALE_MEASURE` 윈도우 내)을 사용할 수 있기 때문이다.
- **부분 책임 (Partial responsibility, lame duck period):** `store+serve` (블록 D까지)
  - operator는 자신이 서명한 데이터가 모두 만료될 때까지 저장 및 제공 책임을 계속 진다.
- **자유 (Free):** operator는 블록 `D+1`부터 책임에서 자유로워진다.

참고: operator가 `B`에서 `D` 사이 어느 시점이라도 quorum에 다시 opt in 하면 위 라이프사이클은 재시작된다.

## 책임 측정·정책·조치 (Accountability Measurements, Policies, and Actions)

**책임 (Responsibilities)**

operator는 EigenDA 프로토콜에서의 역할의 일환으로 attestation과 serving(retrieval) 기능을 모두 수행해야 한다. 이 영역에서의 성과 평가는 여기서 명시된 service level indicator (SLI)로 수행된다.

| 책임 | Rolling Daily SLI (측정) |
| --- | --- |
| Attesting | Signing rate: num-batches-signed / num-batches-responsible-to-sign |
| Serving | Serving 가용성: num-requests-success / num-total-requests |

SLI는 rolling 24시간 구간에 걸쳐 평가된다.

**SLA**

operator는 자신에게 위임된 stake 양에 따라 attesting과 serving(retrieval)에서 높은 가용성을 유지해야 한다. 이는 아래 service level agreement (SLA) 표에 표시되어 있다. operator의 책임 미수행이 미치는 영향은 위임된 stake 양에 비례하므로, 더 큰 비율의 위임 stake를 보유한 operator는 더 높은 가용성 기준을 충족해야 한다.

| Quorum Stake 비율 | Rolling Daily SLA (정책) | Nominal Maximum Daily Downtime |
| --- | --- | --- |
| Baseline | 90 % | 2.4시간 |
| > 5%  | 95% | 1.2시간 | 
| > 10% | 98% | 29분 |
| > 15% | 99.5% | 7분 |

여러 quorum에 위임 stake를 보유한 operator는 자신이 등록된 각 quorum에 연관된 SLA를 충족해야 한다. 예를 들어 'quorum 0'에서 stake 1%, 'quorum 1'에서 stake 7%를 보유한 operator는 'quorum 0'에 대해 signing rate와 serving 가용성을 90% 이상, 'quorum 1'에 대해 95% 이상으로 유지해야 한다.



**집행 조치 (Enforcement Actions)**

operator는 Rolling Daily SLA를 충족하지 못하면 프로토콜에서 강제 ejection 대상이 될 수 있다. 이 조치는 사전 통지 없이도 일어날 수 있으며, operator의 SLI와 전체 ranking 공개 등 초기의 soft enforcement 단계 이후에 따라올 수 있다. ejection은 quorum 단위로 수행된다. 'quorum 0'에서 stake 10%를 보유한 operator가 45분 동안 blob에 attest 하지 않으면, 특히 그 성능이 네트워크 liveness를 침해하는 경우 그 quorum에서 즉시 ejection 될 수 있다. quorum에서 제거되는 것에 더해, ejection 이후 operator는 3일의 cooldown 기간 동안 어떤 quorum에도 합류할 수 없다.
