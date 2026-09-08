---
tags: [concept, serving, parallelism]
---

# Tensor Parallelism

모델이 GPU 하나에 안 들어갈 때 쓰는 병렬화 전략 중 하나. 레이어 하나의 행렬 연산 자체를 여러 GPU에 쪼갭니다. GPU 간 통신이 잦아서 NVLink처럼 빠른 인터커넥트가 필요합니다. [[Pipeline Parallelism]]과 섞어서 70B급 이상 모델을 서빙할 때 흔히 함께 씁니다.

## 관련
- [[Pipeline Parallelism]] — 함께 쓰이는 다른 병렬화 전략

## 실전 사례
- [EP08. vLLM 배포 — V100 3장 (구축기)](../../docs/구축기/EP08-vLLM-배포-V100-3장.md) — `--tensor-parallel-size 3`으로 V100 3장에 32B 모델을 실제로 나눠 올린 사례
