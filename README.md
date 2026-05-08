# EigenDA Core Concepts — 한국어 번역 (Unofficial Korean Translation)

[EigenDA 공식 문서](https://docs.eigencloud.xyz/eigenda/core-concepts/overview)의 **비공식 한국어 번역본**입니다.

번역 정책상 핵심 기술 용어·제품명·고유명사는 영어 그대로 두었으며, 본문 산문은 자연스러운 한국어로 의역했습니다 (예: `KZG polynomial commitment`, `data availability`, `validator`, `quorum` 등은 영문 그대로).

> ⚠️ This is an unofficial Korean translation of the EigenDA documentation. For authoritative content, refer to the original. 번역 오류·해석 차이가 있을 수 있으므로, 의사결정·구현에는 반드시 원문을 확인하십시오.

## 원본 (Upstream)

- 사이트: <https://docs.eigencloud.xyz/eigenda>
- 저장소: <https://github.com/Layr-Labs/eigencloud-docs>
- 경로: `docs/eigenda/`

## 다루는 섹션 (42 페이지)

- **Core Concepts** — Overview, Glossary, Payments, Whitepaper, Security Model, Security FAQs
- **Node Operators** — Overview, Requirements, Run a Node, Registration Protocol, Metrics & Monitoring, Troubleshooting, FAQ, Upgrades
- **Integration Guides** — Overview, Quick Start V2, Custom Security, EigenDA Proxy, Clients, Rollup Guides (OP Stack / Arbitrum Orbit / ZKsync / Secure Trustless Upgrade)
- **Networks** — Mainnet, Sepolia, Hoodi
- **API** — Overview, Data Structures
- **Resources**

## 로컬 빌드

```bash
# 사전 요구사항
brew install rust   # cargo
cargo install mdbook --version 0.4.40
cargo install mdbook-katex --version 0.5.10
cargo install mdbook-mermaid --version 0.14.0

# PATH 설정 (cargo bin이 brew mdbook 0.5.x 보다 앞에 와야 함)
export PATH="$HOME/.cargo/bin:$PATH"

# 빌드
mdbook build

# 로컬 서버
mdbook serve --port 4001
# → http://127.0.0.1:4001/
```

> 호환성: mdbook 0.5.x는 위 preprocessor와 비호환이므로 **0.4.40**을 사용하세요.

## 번역 규칙

- 본문 산문은 한국어 (평어체 `~다`)
- 기술 용어·제품명·약어·코드 식별자는 영어 그대로 (한국어 병기 형식 미사용)
- 코드/명령어/링크/파일 경로/수식은 원문 유지
- 자연스러운 한국어가 도움되는 일반 단어만 한국어 사용

## 라이선스 및 저작권

- 원본 문서의 저작권은 [Eigen Labs / Layr-Labs](https://github.com/Layr-Labs/eigencloud-docs)에 있습니다.
- 본 저장소는 학습·이해 보조 목적의 비공식 한국어 번역물로 fair use 범위 안에서 제공됩니다.
- 원본 저장소가 LICENSE 파일을 포함하지 않으므로, 상업적 활용 전에는 반드시 원저자에게 허가를 받으십시오.

## 자매 프로젝트

- **EigenDA Spec 한국어 번역**: <https://github.com/jyo-o/eigenda-spec-ko> — 더 깊은 protocol/architecture spec

## 기여

오역, 어색한 표현, 누락 등 발견 시 이슈/PR 환영합니다.
