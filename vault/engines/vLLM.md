---
tags: [tool, engine]
---

# vLLM

다중 사용자, 높은 동시 처리량이 필요한 프로덕션 서빙의 사실상 표준. [[PagedAttention]]과 [[Continuous Batching]]이 핵심 최적화입니다. `HF_HUB_OFFLINE=1` 등 오프라인 환경변수를 지원해 [[폐쇄망]] 배포에도 적합합니다.

## 관련
- [[PagedAttention]] — vLLM의 핵심 메모리 관리 기법
- [[Continuous Batching]] — vLLM의 핵심 스케줄링 기법
- [[Ollama]] — 학습 단계에서 쓰다가 넘어오는 이전 도구
- [[5단계 프로덕션 서빙]] — vLLM을 실제로 붙이는 로드맵 단계
