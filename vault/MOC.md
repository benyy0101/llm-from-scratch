---
tags: [moc]
---

# 폐쇄망 LLM 구축 MOC

이 vault는 [[../docs/01-roadmap.md|기존 로드맵 문서]]와 [[../docs/02-serving-theory.md|서빙 이론 노트]]를 옵시디언 방식으로 원자화한 것입니다. 위에서 아래로 읽을 필요는 없습니다 — 궁금한 개념에서 시작해 링크를 따라가면 됩니다.

## 기초 개념
- [[LLM]]
- [[토큰]]
- [[컨텍스트 윈도우]]
- [[파라미터 수]] → [[가중치]]
- [[양자화]] → [[스케일과 제로포인트]] → [[GGUF]] · [[AWQ]] · [[GPTQ]]
- [[RAG]] → [[임베딩과 벡터DB]]
- [[폐쇄망]] → [[표준 아키텍처]] · [[체크섬]]
- [[GPU]] → [[VRAM]] → [[GPU 선택]]

## 서빙 이론
- [[Query·Key·Value]] → [[Multi-Head Attention]] · [[Causal Masking]]
- [[Prefill과 Decode]]
- [[KV 캐시]]
- [[배치]] → [[Static Batching]] / [[Continuous Batching]]
- [[PagedAttention]]
- [[처리량과 지연시간]]
- [[Tensor Parallelism]] / [[Pipeline Parallelism]]
- [[Speculative Decoding]]
- [[Stateful 서빙]]
- [[스트리밍 응답]]
- [[전통 서버 서빙과의 차이]]
- [[모델 서빙과 서비스 배포]]

## 서빙 엔진 / 도구
- [[Ollama]] · [[vLLM]] · [[LM Studio]] · [[GPT4All]] · [[Open WebUI]] · [[llama.cpp]] · [[SGLang]] · [[TensorRT-LLM]] · [[Hugging Face TGI]] · [[LocalAI]] · [[NVIDIA NIM]]
- [[LiteLLM]] (API 게이트웨이) · [[Podman]] (컨테이너 런타임) · [[Continue.dev]] (코딩 어시스턴트 활용 사례) · [[Qdrant]] (벡터DB)

## 모델
- [[Qwen]] · [[Llama (모델)]] · [[Gemma]] · [[Mistral]] · [[EXAONE]] · [[HyperCLOVA X]] · [[SOLAR]] · [[Midm 2.0]]
- [[BGE-M3]] (임베딩 전용 모델)

## 실제 운영 사례
- [[한국은행 사례]] · [[미래에셋 사례]] · [[삼성SDS FabriX]] · [[LG CNS AgenticWorks]] · [[SK C&C]] · [[KT Managed AI GPU]] · [[금융권 AI 플랫폼 정책]] · [[IBM watsonx.ai]] · [[AWS Outposts]]

## 학습 로드맵 (0~7단계)
[[0단계 개념 다지기]] → [[1단계 첫 로컬 실행]] → [[2단계 채팅 UI]] → [[3단계 모델·양자화 비교]] → [[4단계 RAG 실습]] → [[5단계 프로덕션 서빙]] → [[6단계 폐쇄망 이관]] → [[7단계 운영·보안]]

[hoft.tistory.com 시리즈](https://hoft.tistory.com/entry/airgap-llm-survival-ep01-why-local-llm)의 목차를 참고해 직접 이어 쓴 [우리 구축기(EP02~18, docs/구축기/)](../docs/구축기/README.md)의 5단계 분류로 보면 대략 이렇게 겹칩니다 — 기초([[0단계 개념 다지기]] · [[GPU 선택]]) → 인터넷망 준비([[1단계 첫 로컬 실행]]~[[3단계 모델·양자화 비교]]) → 폐쇄망 설치([[5단계 프로덕션 서빙]] · [[6단계 폐쇄망 이관]], 도구는 [[Podman]] · [[LiteLLM]]) → 실전 운영([[7단계 운영·보안]]) → 고급 활용([[4단계 RAG 실습]] · [[Continue.dev]]). 순서는 우리 로드맵대로 두고, 이건 "실제 운영에서는 이렇게도 묶인다"는 참고용 시각입니다.
