# 요구사항 (Requirements Overview)

EigenDA 노드 운영을 결정하기 전에, node operation eligibility의 다음 측면을 충분히 이해하라.

- [Delegation 요구사항](delegation-requirements.mdx): EigenDA는 현재 한정된 수의 operator만 프로토콜에 합류시킨다. 즉, 노드를 운영하려면 최소 stake 요구사항을 충족해야 하며, 이 요구사항은 새로운 operator와 stake가 프로토콜에 합류함에 따라 시간에 따라 조정된다.
- [System 요구사항](system-requirements.md): EigenDA는 수평 확장(horizontally scaling) 아키텍처이므로, operator 노드의 시스템 요구사항은 operator에게 위임된 stake 양에 따라 확장된다. 노드 operator는 자신의 위임 stake 양에 기반한 요구사항을 이해해야 하며, 변화하는 stake 분포에 대응해 [setup을 업그레이드](../upgrades/system-upgrades/)할 준비가 되어 있어야 한다.
- [Protocol SLA](protocol-SLA.md): 모든 operator는 service level agreement를 충족해야 하며, 위반 시 특정 프로토콜 수준의 결과를 받는다.
