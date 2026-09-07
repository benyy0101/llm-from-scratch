---
tags: [concept, serving, parallelism]
---

# Pipeline Parallelism

모델이 GPU 하나에 안 들어갈 때 쓰는 병렬화 전략 중 하나. 레이어 묶음을 GPU별로 나눠 순차적으로 통과시킵니다. [[Tensor Parallelism]]보다 GPU 간 통신은 적지만, GPU들이 순서대로 기다리는 "파이프라인 버블"이 생기는 단점이 있습니다.

## 관련
- [[Tensor Parallelism]] — 함께 쓰이는 다른 병렬화 전략
