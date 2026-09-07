---
tags: [concept, serving, batching]
aliases: ["In-flight Batching"]
---

# Continuous Batching

[[vLLM]]·[[Hugging Face TGI|TGI]] 계열이 쓰는 배칭 방식. 요청이 끝나는 즉시 배치에서 빼고, 대기 중이던 새 요청을 그 자리에 바로 채워 넣어 GPU 유휴 시간을 최소화합니다. [[Static Batching]]과 달리 긴 요청 하나 때문에 다른 요청들이 기다릴 필요가 없습니다.

## 관련
- [[Static Batching]] — Continuous Batching이 개선한 이전 방식
- [[PagedAttention]] — Continuous Batching과 함께 vLLM 처리량을 만드는 다른 한 축
- [[처리량과 지연시간]] — 배치 크기를 키울수록 처리량은 오르고 지연시간은 늘어나는 트레이드오프
