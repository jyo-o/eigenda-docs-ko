# EigenDA 용어집 (Glossary)

EigenDA 프로토콜의 핵심 컴포넌트와 관련 용어 정의를 제공한다.

## 아키텍처 (Architecture)

### Validator Nodes (밸리데이터 노드)
Validator node는 blob의 availability에 attest하고, retrieval node(궁극적으로는 light node)에게 해당 blob을 제공할 책임이 있다. Validator node는 EigenLayer에 등록되어 stake되며, 자신이 위임받은 stake asset에 대응하는 EigenDA operator set에 등록된다. 각 validator node는 프로토콜이 처리하는 각 blob의 일부분만 검증·저장·제공한다.

### Dispersers (디스퍼서)
Disperser는 데이터를 인코딩해 validator node에게 전달한다. Disperser는 데이터 인코딩의 정확성을 증명하는 proof를 생성해 함께 전달해야 한다. 또한 disperser는 validator node로부터 availability attestation을 집계하며, 이는 rollup 같은 use-case를 지원하기 위해 on-chain으로 bridge될 수 있다.

### Retrieval Nodes (조회 노드)
Retrieval node는 validator node로부터 데이터 shard를 모아 원본 데이터 컨텐츠로 디코딩한다.

### Light Nodes (라이트 노드, Planned)
Light node는 observability를 제공해, validator node가 retrieval node에게 데이터를 withhold하더라도 그 withholding이 광범위하게 관찰 가능하도록 한다.

### Cert Verifier (인증서 검증자)
Ethereum 위의 smart contract로, security threshold와 required quorum을 사용해 blob cert를 검증하는 `verifyDACertV2()` 함수를 노출한다.

### EigenDA Network (EigenDA 네트워크)

Validator Node, Disperser, Retrieval Node, Relay, contract를 포함한 EigenDA 네트워크의 모든 행위자(actor) 전체.

## 암호 (Cryptography)

### KZG Polynomial Commitments (KZG 다항식 커미트먼트)
하나의 polynomial에 대해 commit한 뒤, 특정 지점에서의 evaluation을 작고 일정한 크기의 proof로 나중에 증명할 수 있게 해주는 암호 프로토콜이다. EigenDA에서 KZG commitment는 validator가 자신에게 할당된 데이터 chunk가 원래의 blob에 속한다는 점을, 전체 데이터를 다운로드하지 않고도 검증할 수 있게 해 준다 — disperser 동작에 대한 trustless verification을 가능하게 한다.

### Multi-reveal Proof (다중 공개 증명)
서로 다른 지점에서의 여러 KZG polynomial evaluation을 단 하나의 succinct proof로 검증할 수 있게 하는 암호 메커니즘이다. EigenDA에서 validator node는 각 evaluation 지점마다 별도 proof를 요구하지 않고도, 자신에게 할당된 chunk가 원본 blob의 유효한 부분임을 효율적으로 검증하기 위해 이를 사용한다.

### Reed-Solomon Erasure Encoding (소거 부호화)
blob 데이터를 validator node 전반에 분산되는 redundant chunk로 변환하기 위해 사용되며, 일부 노드가 실패하거나 악의적으로 행동해도 원본 데이터가 복원될 수 있도록 보장한다. 충분한 수의 정직한 노드가 접근 가능한 한, EigenDA가 data availability를 유지할 수 있게 한다.

## 일반 개념 (General Concepts)

### Horizontal Scaling (수평 확장)
기존 머신을 업그레이드하는 대신 머신을 더 추가해 시스템 용량을 늘리는 방식이다. EigenDA에서는 validator node를 더 추가해 네트워크 throughput을 증가시키는 것을 의미하며, 각 노드가 인코딩된 데이터의 일부를 처리하므로 네트워크가 확장될수록 더 큰 데이터 볼륨을 처리할 수 있다.

### DA Certificate (DA 인증서)
특정 데이터가 EigenDA에서 올바르게 인코딩·분산되어 사용 가능함을 증명하는 cryptographic proof. validator node의 서명과 기타 메타데이터를 담고 있어, rollup, AVS, application 등 EigenDA 사용자가 검증하는 데 쓴다.

### Payload (페이로드)
사용자가 EigenDA에 제출하는 데이터.

### Blob (블롭)
사용자가 제출한 데이터(payload)를 BN254 prime field 상에서 Reed–Solomon erasure encoding한 중간 표현 형태다(chunk로 쪼개져 field element로 매핑됨). 이 element들은 polynomial coefficient 역할을 하며, validator로 분배되기 위해 KZG-commit된다.

### Blob Key (블롭 키)
`blob_header_hash` 또는 `blobHeaderHash`라고도 부른다. ABI 인코딩된 BlobHeader의 keccak256 해시로 계산되는 32바이트 식별자다. EigenDA 전반에서 dispersal status 조회, validator/relay에서의 blob 가져오기, blob과 그 인증서 연결에 쓰는 메인 lookup key다. Disperser가 이 blob key를 계산해 클라이언트에게 반환하고, 클라이언트는 자신이 보낸 BlobHeader로부터 해시를 다시 계산해 검증할 수 있다.

### Chunk (청크)
erasure-code된 blob의 shard로, validator의 stake weight에 따라 각 validator에게 할당되어 저장된다. 각 validator는 전체 blob이 아니라 자신에게 할당된 특정 chunk만 저장한다.

### Batch (배치)
효율을 위해 함께 처리되는 여러 blob의 모음. 한 번에 많은 blob에 대한 attestation을 validator가 생성할 수 있게 한다.

### ETH / EIGEN / Custom Quorum (쿼럼)
EigenDA에 등록된 validator node 집합으로, DA 작업이 이 노드들에 전송되며 각 노드는 quorum 내 상대적 stake weight에 따라 독립적으로 가중된다. delegation requirement에 명시된 asset으로 구분된다.
