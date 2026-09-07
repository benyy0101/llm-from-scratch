---
tags: [concept, serving, advanced]
---

# Speculative Decoding

작은 draft 모델이 여러 [[토큰]]을 미리 추측하고, 큰 모델이 그 추측을 한 번에 검증하는 방식입니다. [[Prefill과 Decode|Decode]] 단계가 토큰을 하나씩만 순차적으로 생성해야 하는 병목을 일부 우회하는 최신 기법으로, [[5단계 프로덕션 서빙]] 이후의 심화 주제입니다.

## 관련
- [[Prefill과 Decode]] — 이 기법이 우회하려는 병목
