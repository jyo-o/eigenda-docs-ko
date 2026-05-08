
# 보안 FAQ (Security FAQs)

이 문서의 목적은 Ethereum L2로서 DA solution을 사용할 때 자주 묻는 질문을 정리하고 보안 trade-off를 더 잘 이해하도록 돕는 것이다. 본 문서는 DA에 대한 종합적 보안 평가를 다루지는 않는다.

## alt-DA solution이 Ethereum L2에 어떤 종류의 보안을 제공할 수 있는가?

alt-DA에서 가능한 보안은 BFT security와 cryptoeconomic security뿐이다. EigenDA가 이 두 가지 형태의 보안을 어떻게 달성하는지에 대한 자세한 논의는 [Security Model](security-model.md)에서 확인할 수 있다.

Data Availability Sampling (DAS) 프로토콜의 light client operator에게 단독 검증 가능성(unilateral verifiability)을 제공한다고 주장하는 DA 프로토콜이라 하더라도, L1 smart contract는 이 기능을 평가할 수 없다. smart contract 컨텍스트에서는 networking 능력이 사용 가능하지 않기 때문이다. 따라서 rollup bridge는 DA attestation을 받아들이기 전에 데이터가 DA provider로부터 사용 가능한지 검증할 수 없다. 즉, light node의 관찰을 layer-1으로 증명 가능하게 bridge할 수 없다.

이런 의미에서, BFT 및 cryptoeconomic security를 제공하는 모든 DA 솔루션은 Ethereum rollup의 관점에서 질적으로 동등한 security guarantee를 제공한다.

## DA 프로토콜에서 slashing은 어떻게 동작하는가?

data availability는 객관적으로 증명하거나 반증할 수 없다 — 오직 주관적으로만 관찰될 수 있다. 사실상 모든 일반적인 networking primitive는 데이터를 한쪽 당사자에게는 선택적으로 제공하고 다른 쪽에게는 제공하지 않는 것이 가능하기 때문이다.

이 때문에 dishonest majority 하에서 DA 프로토콜의 cryptoeconomic security를 달성하기 위해 알려진 유일한 방법은, safety 실패에 대응해 chain 또는 staking asset을 fork하는 능력이다. 아이디어는 단순하다 — 다수의 노드가 결탁해 safety를 위반하면(예: 사용 가능하다고 주장한 데이터를 withhold하는 경우), 정직한 소수가 chain이나 staking 토큰을 fork할 수 있다. 그러면 더 넓은 커뮤니티가 두 fork를 평가해 어느 쪽을 지원할지 결정하며, 시장이 dishonest majority의 fork를 가치 절하하는 식으로 경제적 처벌을 가할 수 있다.

data availability는 Ethereum이나 Solana 같은 블록체인의 암묵적 기능이다. 이러한 블록체인에 대해 가정하는 것은, data availability withholding attack이 발생하면 커뮤니티 구성원의 검사로 withholding이 드러나면서 chain이 fork되고 위반 validator의 stake가 slash된다는 것이다. EigenDA처럼 토큰 forking 메커니즘을 slashing에 사용하는 DA 프로토콜은 이러한 cryptoeconomic security의 기본 자세를 그대로 물려받는다.

## EigenDA에 slashing이 있는가? slashing은 어떻게 동작하는가?

slashing은 앞 질문에서 설명한 cryptoeconomic security 모델의 주요 도구다.

EIGEN에 대해 우리는 token forking 방식의 slashing을 가지며, 이는 chain forking과 동등하다(자세한 내용은 [EIGEN Token Whitepaper](https://docs.eigenlayer.xyz/assets/files/EIGEN_Token_Whitepaper-0df8e17b7efa052fd2a22e1ade9c6f69.pdf) 참고).

quorum 안의 staked asset 중 임계값 이상을 악의적 stake가 통제하는 safety 실패가 발생하면([Security Model](security-model.md) 참고), 커뮤니티 구성원이 데이터 unavailability에 대한 alarm을 트리거할 수 있다. 충분한 수의 커뮤니티 구성원이 safety가 위반됐다는 데 동의하면, 토큰 forking을 시작해 dishonest majority를 slash할 수 있다.

EIGEN은 marketplace가 slashing을 가할 수 있는 여러 Eigen fork를 지원할 수 있는 반면, Tendermint consensus를 사용하는 솔루션은 정직한 소수가 chain forking으로 dishonest majority를 slash하려 할 때 minority fork에서 진행이 안 되는 문제(non-progress)에 직면할 수 있다.

## DAS의 한계는 무엇인가?

DAS는 일반적으로 DA 프로토콜에 대해 두 가지 목적을 가진다.

1. **observability 향상**: slashing을 지원하기 위해 DA fault에 대한 커뮤니티 observability를 개선.
2. **verifiability**: 최종 사용자가 availability에 대해 판단할 수 있게 하여 rollup이 정상 상태인지 검증하고 서비스를 신뢰하게 함으로써, rollup에서 거래하는 사용자에게 effective finality 시간을 단축.

그러나 현재의 Data Availability Sampling 구현체는 그 가치 제안을 제한하는 심각한 한계를 갖는 경향이 있다.

1. **L2에 대한 효용 제한**: 앞서 언급했듯, smart contract의 본질적 한계 때문에 data availability 검증을 L1 bridge contract 안에서 수행할 수 없다.
2. **불충분한 탐지 속성**: 대부분의 DAS 프로토콜은 실제 시스템에서는 충족되지 않는 중요한 네트워크 수준 가정을 숨긴 채 추상화 수준에서 제시된다. 이로 인해 다수의 light client가 데이터가 가용하다고 잠재적으로 표적된 방식으로 속을 수 있다. 구체적으로, 악의적인 data storage node는 특정 light node에게 chunk를 선택적으로 release해 데이터가 가용하다고 믿게 만들 수 있지만, release된 chunk는 원본 데이터를 복원하기에 충분하지 않다(즉, 데이터가 실제로는 가용하지 않다). 이 주제에 대한 더 많은 논의는 [Joachim Neu의 블로그 글](https://www.paradigm.xyz/2022/08/das)에서 확인할 수 있다.
3. **불완전한 복구 메커니즘**: 많은 DAS 프로토콜은 데이터가 light node 집합에 집합적으로 release되었는지 탐지하는 것을 목표로 하지만(즉, light node가 보유한 chunk로부터 이론적으로 데이터를 재구성할 수 있다는 의미), 다수의 프로토콜이 이 재구성을 완전히 성능 좋게 또는 아예 수행할 수 있는 메커니즘을 제공하지 않는다. 즉, 적이 light node에게 데이터를 release하면서도 실제 데이터 소비자에게는 데이터를 거부할 수 있어, *모든* light node가 데이터 상태에 대해 속을 수 있다.
4. **숨은 신뢰 가정**: 많은 DAS 프로토콜은 프로토콜이 제공하는 전체 속성에 의문을 던지는 신뢰 가정에 의존한다. 예를 들어, Celestia는 인코딩이 잘못된 경우 fraud proof를 받기 위해 각 light node가 정직한 full node에 연결되어 있어야 한다. 네트워크 topology에 따라 이는 제한된 Celestia full node 집합에 대한 다양한 BFT 형태의 신뢰 가정으로 이어질 수 있다.

## EigenDA는 DAS 프로토콜을 만들고 있는가?

그렇다. EigenDA는 앞 절의 한계 다수를 해결하는 확장 가능한 DAS 프로토콜을 적극적으로 개발 중이다. EigenDA DAS의 whitepaper가 곧 발표될 예정이다.

## restaked ETH는 EigenDA에 어떤 보안을 제공하는가?

EigenDA는 \$8.8B를 초과하는 ETH restake quorum에 의해 추가로 validate된다. 즉, 시스템을 공격하려면 colluding operator 집합이 ETH re-staker로부터 \$4.4B*를 초과하는 delegation을 받아야 한다는 의미다.

## EigenDA는 KZG Polynomial Commitment를 무엇에 사용하는가?

EigenDA에서 KZG commitment는 데이터 chunk가 data blob으로부터 올바르게 인코딩되었음을 보장하는 데 사용된다. 이를 통해 validator는 disperser로부터 받은 chunk의 유효성을 효율적으로 검증할 수 있다. 비교하자면, fraud proof는 데이터 chunk의 유효성을 보장하기 위해 더 긴 시간 윈도우와 추가 신뢰 가정을 요구한다.

*이 결론은 confirmation threshold 63%에 기반하며, 이는 safety threshold 50%에 대응한다. 더 자세한 분석은 [Security Model](security-model.md)에서 확인할 수 있다.
