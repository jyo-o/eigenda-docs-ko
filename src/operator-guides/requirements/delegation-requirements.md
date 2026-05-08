
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Delegation 요구사항

EigenDA operator가 되려면 먼저 EigenLayer operator로 등록한 뒤 stake 요구사항을 충족해야 한다. 이 요구사항은 quorum 단위로 평가되며, 각 quorum의 가중치 기준으로 산정된다:

<Tabs groupId="network">
  <TabItem value="mainnet" label="Mainnet">

    - 최소 stake 하한: 각 operator는 다음 중 하나를 충족해야 한다.
      - ETH quorum에 합류하려면 최소 32 ETH
      - EIGEN quorum에 합류하려면 최소 1 EIGEN
    - 혼잡한 operator set의 stake 하한: 해당 quorum이 글로벌 operator 한도(200)에 도달한 경우, 합류하려는 operator는 그 quorum에서 가장 낮은 가중치를 가진 기존 operator의 가중치보다 1.1배 이상을 가져야 그 operator를 대체할 수 있다.

  </TabItem>
  <TabItem value="hoodi" label="Hoodi">

    - 최소 stake 하한: 각 operator는 다음 중 하나를 충족해야 한다.
      - ETH quorum에 합류하려면 최소 32 ETH
      - EIGEN quorum에 합류하려면 최소 1 EIGEN/bEIGEN
    - 혼잡한 operator set의 stake 하한: 해당 quorum이 글로벌 operator 한도(200)에 도달한 경우, 합류하려는 operator는 그 quorum에서 가장 낮은 가중치를 가진 기존 operator의 가중치보다 1.1배 이상을 가져야 그 operator를 대체할 수 있다.

  </TabItem>
</Tabs>

이 요구사항이 어떻게 강제되며 DA node가 자격이 있는 quorum에 합류하는 절차에 대한 자세한 내용은 [Registration Protocol Overview](../registration-protocol.md)에서 확인할 수 있다.


## 자격 확인 (Checking eligibility)

각 quorum 상위 200개 operator의 현재 TVL을 확인하려면, AVS 페이지([Mainnet](https://app.eigenlayer.xyz/avs/eigenda), [Hoodi](https://hoodi.eigenlayer.xyz/avs/eigenda))를 방문해 `TVL Descending` 으로 정렬한다. 해당 quorum에 나열된 상위 200 operator와 거기에 위임된 ETH TVL 양을 살펴본다. AVS 페이지는 EigenLayer 상의 operator stake를 반영하며, 이 stake는 매주(수요일 17:00 UTC) EigenDA operator set 가중치 갱신에 사용된다. 따라서 EigenDA stake는 실시간 EigenLayer stake보다 최대 7일 정도 지연될 수 있다.
