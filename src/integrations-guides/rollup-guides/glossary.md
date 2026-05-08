
# Rollup 용어집 (Glossary)

이 용어집은 rollup 통합과 EigenDA 관련 용어를 담는다. stack에 종속되지 않는 용어를 사용하려고 하며, 다양한 rollup stack에서의 동의어도 함께 정리한다.

## Cert Punctuality Window (인증서 적시성 윈도우)

[batcher](#rollup-batcher)가 batch를 생성한 뒤 [rollup inbox](#rollup-inbox)에 제출해야 하는 시간 window (L1 block 수 단위).

cert는 onchain에 cert의 [ReferenceBlockNumber][spec-rbn] (RBN) + cert의 CPW (Cert punctuality window) 이전에 포함되었을 때 유효한 것으로 간주된다.
```
RBN < cert.L1InclusionBlock < RBN+CPW 
```

기본 CPW로 12시간(ethereum mainnet 기준 3600 block)이 권장된다. OP의 경우, 이 값은 [sequencerWindowSize](https://docs.optimism.io/operators/chain-operators/configuration/rollup#sequencerwindowsize) 이상이어야 한다.

## Rollup Batcher (롤업 배처)

transaction(또는 state diff)을 batching해 [rollup inbox](#rollup-inbox)로 보내는 책임을 지는 sequencer 서비스 (별도 binary일 수도 있고, sequencer 내부 thread일 수도 있다).

## Rollup Inbox (롤업 인박스)

[rollup batcher](#rollup-batcher)가 transaction batch (또는 state diff)를 보내는 Ethereum 주소. EOA(op-stack)일 수도 있고 contract(nitro, zksync)일 수도 있다.

- op stack: batcher inbox (EOA)
- nitro stack: sequencer inbox (contract)
- zk stack: [ExecutorFacet](https://docs.zksync.io/zksync-protocol/contracts/l1-contracts#executorfacet) (간단히 [diamond proxy](https://docs.zksync.io/zksync-protocol/contracts/l1-contracts#diamond-also-mentioned-as-state-transition-contract)라고도 불림)




<!-- References -->
[spec-rbn]: https://layr-labs.github.io/eigenda/protobufs/generated/common_v2.html#batchheader
