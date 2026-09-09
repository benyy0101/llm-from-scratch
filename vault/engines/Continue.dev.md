---
tags: [tool, application]
aliases: ["Continue"]
---

# Continue.dev

VS Code·JetBrains용 오픈소스 AI 코딩 어시스턴트입니다. GitHub Copilot과 비슷하게 자동완성·채팅·코드 편집을 지원하지만, 클라우드 API 대신 [[Ollama]]나 [[vLLM]] 같은 자체 호스팅 엔드포인트를 그대로 가리킬 수 있습니다 — [[폐쇄망]] 환경에서 사내 개발자에게 "사내판 Copilot"을 제공하는 대표적인 활용 사례입니다.

보통 자동완성용 작은 모델과 채팅/편집용 큰 모델을 따로 붙이는 구성이 흔합니다 — 자동완성엔 빠른 코드 특화 소형 모델, 채팅엔 더 똑똑한 모델을 씁니다.

## 관련
- [[Ollama]] · [[vLLM]] — Continue.dev가 붙는 백엔드
- [[폐쇄망]] — 이 조합이 특히 의미 있는 배포 환경
- [[Cline]] — 자율 에이전트 성격이 더 강한 대안, 승인 흐름이 촘촘함

## 실전 사례
- [EP17. Continue.dev 코딩 어시스턴트 (구축기)](../../docs/구축기/EP17-Continue.dev-코딩-어시스턴트.md) — 이미 VRAM이 꽉 찬 상황에서 자동완성용 소형 모델을 추가로 얹은 실제 절충
