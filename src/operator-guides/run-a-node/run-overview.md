# 노드 운영하기 (Run a Node Overview)

node operator가 되기 위한 [eligibility 요구사항](../requirements/requirements-overview.md)을 모두 충족할 수 있다면, 노드를 설정하고 실행할 준비가 된 것이다.

> ℹ️ **Info**
>
> EigenDA의 operator로 등록하기 전, operator는 먼저 [EigenLayer에 operator로 등록](https://docs.eigencloud.xyz/products/eigenlayer/operators/howto/operator-installation)해야 한다. 이 과정에서 아래 DA 노드 설정 단계에서 필요한 BLS와 ECDSA 키를 안전하게 생성한다.


operator 노드를 운영하는 것은 몇 가지 주요 단계로 구성된다.
1. 시스템 환경 설정과 노드 구성 ([run with docker](run-with-docker.mdx)에서 다룸)
2. 노드 소프트웨어 시작 및 기본 동작 확인 ([run with docker](run-with-docker.mdx)에서 다룸)
3. 하나 이상의 quorum에 [노드 등록](./registration/)

현재 처음 두 단계는 [docker로 노드 실행하기](run-with-docker.mdx) 컨텍스트에서 전체 문서를 제공한다. 다른 setup을 사용하는 operator도 이 지침을 가이드로 활용할 수 있다.
