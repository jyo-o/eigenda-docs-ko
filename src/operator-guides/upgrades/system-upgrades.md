# 시스템 업그레이드 (System Upgrades)

operator에게 위임된 stake 양에 따라 시스템 요구사항이 동적으로 확장되므로, 노드 operator는 [Protocol SLA](../requirements/protocol-SLA/)를 계속 충족하기 위해 때때로 시스템 setup을 업그레이드해야 할 수 있다.

시스템 업그레이드를 수행할 때 operator는 다음 고려사항을 유의해야 한다.
- BLS signing key의 custody 유지.
- 업그레이드된 노드가 이전에 등록된 주소에서 계속 도달 가능하도록 보장.
- 노드가 저장한 모든 blob 데이터의 무결성 유지.

## 노드 마이그레이션 (Node migration)

[running with docker](../run-a-node/run-with-docker/) 가이드의 setup 단계를 따랐다면, 노드는 `.env` 파일의 `NODE_DB_PATH_HOST`로 지정된 위치에 데이터를 저장한다. 이 경로는 EigenDA docker container에 bind-mount된다.

일반적으로, 노드를 새 머신으로 마이그레이션하려는 경우 데이터 무결성을 유지하고 마이그레이션 중 및 이후에도 [Protocol SLA](../requirements/protocol-SLA/)를 계속 충족하기 위해 다음 순서를 따라야 한다.

**기존 머신**:
1. 머신에 저장된 키와 마이그레이션하려는 다른 config들을 백업한다.
2. 모든 quorum에서 opt-out 한다.
3. 1시간 이상 노드를 계속 실행 상태로 둔다 (이 시점 이후 노드는 dispersal request를 받지 않는다).
4. retrieval 트래픽을 위해 새 머신을 준비하는 동안 계속 노드를 실행한다.

**새 머신**:
1. 기존 머신의 `NODE_DB_PATH_HOST` 위치에 있는 파일들을 복사한다.
2. 기존 머신에서 복사한 파일을 사용해 EigenDA Node를 시작한다(예: `docker-compose up -d`). 파일은 `NODE_DB_PATH_HOST` 경로 아래에 있어야 하며 노드가 도달 가능한지 확인한다.
3. 새 IP 주소로 quorum에 opt-in 한다 (그래야 setup 중에도 기존 머신이 원래 IP로 계속 도달 가능하다). DNS로 등록하는 경우, DNS를 새 IP로 다시 가리킨 뒤 quorum에 opt-in 한다.

마지막으로, 새 노드가 retrieval과 dispersal 모두에서 동작하면 기존 머신의 노드를 종료할 수 있다(예: `docker-compose down`).
