
# Custom Quorum과 Threshold (Custom Security)

EigenDA는 사용자가 보안 보장을 유지하면서 자신만의 data availability solution을 맞춤화할 수 있도록 한다.

이는 사용자가 자신의 [custom quorum](../core-concepts/security/security-model.md#quorums-and-security-models)과 [security threshold](../core-concepts/security/security-model.md#safety-and-liveness-analysis)를 정의할 수 있도록 함으로써 이뤄진다.

이러한 설정을 사용하는 rollup은, custom quorum을 포함해 각 quorum에 대해 자신이 설정한 `thresholds` 를 disperser로부터 받은 DA Certificate가 충족하는지 강제해야 한다.

custom quorum에 dispersal하는 것은 사실상, 그 custom quorum을 정의하는 custom token을 보유한 operator 집합에 데이터를 추가로 복제하는 효과가 있다.

이는 rollup의 token 보유자가 자신의 token을 위임함으로써, 어떤 operator를 신뢰해 rollup의 data availability를 보장하게 할지 결정할 수 있다는 의미다.

## 개요

custom quorum과 threshold는 rollup 및 기타 사용자에게 다음을 가능하게 한다:
- 자신의 token 위임을 통해 데이터 검증을 위한 specific operator set 정의
- 특정 activation block number부터 시작해 custom quorum의 서명 검증을 강제
- data availability confirmation을 위한 custom confirmation threshold 설정
- 보안 요구가 진화함에 따라 이 threshold들을 안전하게 업그레이드

## Native Token에 대한 경제적 효용

custom quorum의 핵심 이점 중 하나는 사용자가 자신의 native ERC20 token에 경제적 효용을 부여할 수 있다는 점이다. rollup은:
- 자신의 native token re-staking을 요구하는 전용 quorum을 만들 수 있다.
- 자신의 token 생태계가 뒷받침하는 economic security를 구축할 수 있다.
- token 보유자가 rollup의 data availability를 보호하는 데 참여하도록 할 수 있다.

이로써 rollup의 성공이 곧 자신의 native token의 효용과 가치를 직접 강화하고, 그 token으로 다시 rollup의 보안을 강화하는 강력한 경제적 flywheel이 생긴다.

## 안전하게 업그레이드 가능한 Cert 검증

custom quorum과 threshold에 대한 backward-compatible secure update는, EigenDA Cert 검증 로직을 매끄럽게(그리고 안전하게) 업데이트하는 데 사용되는 것과 정확히 동일한 메커니즘으로 구현된다.

이 덕분에 이전에 EigenDA certificate를 검증하지 않던 rollup에 cert 검증을 안전하게 추가할 수 있고, 기존 cert 검증을 새 버전으로 업그레이드하거나 추가 custom quorum을 검증하도록 할 수 있다.

## 구현 절차

custom security를 구현하는 절차는 다음 핵심 단계로 이뤄진다:

### 1. Custom EigenDACertVerifierRouter 배포

custom quorum 구성에 대한 certificate 검증을 관리할 EigenDACertVerifierRouter contract의 자체 인스턴스를 [여기](https://github.com/Layr-Labs/eigenda/blob/e586028cf9688935eca5949ba469961c09ddfc4e/contracts/script/deploy/router/README.md)의 절차에 따라 배포한다.

### 2. Proxy 인스턴스 설정

custom router contract를 가리키는 설정으로 EigenDA proxy 인스턴스를 재시작해 custom security 검증을 활성화한다.

### 3. Certificate Verifier Contract 배포

본인의 specific custom quorum과 threshold 요구사항을 구현하는 새 certificate verifier contract를 배포한다.

### 4. Custom Verifier 활성화

특정 block number에서 새 verifier가 활성화되도록 구성하여 매끄러운 전환을 보장하고 업그레이드 절차 전반에 걸쳐 보안 보장을 유지한다.

## 보안 고려사항

custom quorum과 threshold를 구현할 때 다음을 고려한다:

- custom quorum operator가 의미 있는 보안을 제공할 수 있을 만큼 충분한 stake를 유지하도록 한다.
- 보안과 성능 요구사항의 균형을 맞춘 적절한 confirmation threshold를 설정한다.
- 전환 중 보안 공백이 생기지 않도록 activation block number를 신중히 계획한다.
- custom quorum operator에 대한 경제적 인센티브를 고려한다.
- custom quorum의 건강 상태와 operator 참여를 정기적으로 모니터링한다.

## 시작하기

본인 rollup에 custom security 구현을 시작하려면:

1. EigenDA team에 연락해 specific 요구사항을 논의한다.
2. quorum mechanic을 이해하기 위해 security model 문서를 검토한다.
3. custom token delegation 전략을 계획한다.
4. mainnet 배포 전에 testnet에서 구현을 테스트한다.

기술적 구현 세부 사항과 smart contract 인터페이스에 대해서는 [EigenDA integration guides](overview.md)를 참고하고 EigenDA 개발팀과 상의한다.
