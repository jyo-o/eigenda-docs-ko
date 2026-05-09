
# 등록 프로토콜 상세 (Registration Protocol)

이 페이지는 EigenDA operator의 등록 절차에 대한 추가 배경 정보를 담고 있다. 이 절에서 설명하는 단계들은 [registration instructions](./run-a-node/registration/)에서 참조하는 스크립트가 자동으로 수행한다.


## 등록 통제 (Registration Controls)

EigenDA 네트워크는 각 quorum에서 quorum weight 기준 상위 N=200 operator를 포함하도록 설계되어 있다. 이 설계는 securing stake 총량을 최대화하여 네트워크 전반의 성능과 보안을 강화하는 것을 목표로 한다.

quorum weight 기준 최소 operator 정보를 smart contract에 유지하는 것은, onchain에서 sorting 또는 priority queue를 유지하는 데 드는 높은 연산 비용과 복잡도 때문에 현실적이지 않다. 이를 관리하기 위해, 네트워크는 인가된 off-chain churn approver와 일련의 onchain 검증의 조합을 사용한다.

### EigenDA Churn Approver (처닝 승인자)

churn approver는 등록 contract에 quorum weight 기준 가장 작은 operator 정보를 제공하는 신뢰된 서비스를 수행한다.

네트워크가 operator cap에 도달했고 새로운 operator가 합류하려는 경우, 새 operator는 Churn Approver에게 서명을 요청할 수 있다. Churn Approver는 새 operator가 stake 요구사항을 충족하는지 확인하고, 현재 stake가 가장 낮은 operator 제거를 승인하는 서명을 제공한다. 새 operator는 이후 Churn Approver의 서명과 stake가 가장 낮은 기존 operator의 정보를 추가 입력으로 EigenDA의 smart contract에 제공해 EigenDA에 opt-in한다.

### Smart Contract 검증

smart contract는 operator 교체 절차의 무결성을 보장하기 위해 일련의 검증을 수행한다:

1. Churn Approver의 서명을 검증한다.
2. 새로 가입하는 operator와 (ejection 대상인) 현재 stake가 가장 낮은 operator의 stake를 비교 검증한다:
    - 새 operator는 ejection되는 operator stake의 최소 1.1배를 보유해야 한다.
    - ejection되는 operator는 전체 stake의 10.01% 미만이어야 한다.

step 2의 검증 파라미터는 contract governance에서 변경 가능하다.

이 검증 단계가 모두 통과하면, contract는 churner가 식별한 stake 최소 operator를 ejection하고 평소대로 새 operator의 opt-in을 진행한다.


## smart contract 기반 operator 지원

[registration instructions](./run-a-node/registration/)에 제공된 opt-in 스크립트는 EigenDA operator가 transaction 서명을 위한 ECDSA private key를 준비한다고 가정하지만, 원리상 EigenDA operator가 smart contract에서 등록하는 것도 가능하다. 이 통합에 대한 상세한 안내가 필요하다면 문의해 주기 바란다.
