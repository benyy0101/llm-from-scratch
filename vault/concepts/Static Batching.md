---
tags: [concept, serving, batching]
---

# Static Batching

요청 N개를 한 배치로 묶어 처리하는 방식. 배치 안에서 가장 긴 응답이 끝날 때까지 이미 끝난 다른 요청들도 자리를 비우지 못하고 대기해야 해서 GPU가 노는 시간이 생깁니다. [[Continuous Batching]]이 이 비효율을 해결하기 위해 나온 대안입니다.

## 관련
- [[Continuous Batching]] — Static Batching의 비효율을 해결한 방식
- [[처리량과 지연시간]] — 배칭 전략이 조절하는 트레이드오프
