---
tags: [moc]
---

# 폐쇄망 LLM 구축 MOC

이 vault는 [[../docs/01-roadmap.md|기존 로드맵 문서]]와 [[../docs/02-serving-theory.md|서빙 이론 노트]]를 옵시디언 방식으로 원자화한 것입니다. 위에서 아래로 읽을 필요는 없습니다 — 궁금한 개념에서 시작해 링크를 따라가면 됩니다.

"기초 개념"~"서빙 엔진/도구" 아래는 원래 "이미 학습된 모델을 어떻게 폐쇄망에서 서빙하는가" 실무 위주였고, "수학 기초"~"LLM 엔지니어링·에이전트"는 [aiengineeringfromscratch.com](https://aiengineeringfromscratch.com/)(MIT 라이선스, rohitg00/ai-engineering-from-scratch)의 관련 레슨을 참고해 그 실무 노트들이 왜 그렇게 동작하는지 받쳐주는 이론 레이어로 새로 추가한 것입니다 — 원문을 옮긴 게 아니라 개념을 소화해서 이 vault 문체로 다시 쓴 원자 노트입니다.

## 기초 개념
- [[LLM]] · [[모델과 엔진]]
- [[토큰]] → [[토큰 임베딩 (입력 벡터화)]]
- [[컨텍스트 윈도우]]
- [[파라미터 수]] → [[가중치]] → [[행렬곱과 가중치]] · [[FP16과 BF16]]
- [[양자화]] → [[PTQ와 QAT]] · [[스케일과 제로포인트]] → [[GGUF]] · [[AWQ]] · [[GPTQ]] · [[FP8]]
- [[RAG]] → [[청킹]] · [[임베딩과 벡터DB]] · [[하이브리드 검색]] · [[리랭커]] · [[환각]]
- [[폐쇄망]] → [[표준 아키텍처]] · [[체크섬]]
- [[GPU]] → [[VRAM]] → [[GPU 선택]] → [[DGX 멀티노드 아키텍처]]
- [[데몬]]

## 수학 기초
- [[벡터와 행렬]] → [[행렬 변환 (선형 변환)]] · [[텐서 연산]]
- [[미분과 그래디언트]] → [[연쇄법칙과 자동미분]] → [[경사하강법]]
- [[확률분포]] → [[베이즈 정리]] · [[통계 기초]] · [[샘플링 방법]]
- [[정보이론]] — 엔트로피·교차엔트로피·KL divergence, [[양자화]]의 정보손실 이론
- [[노름과 거리]] · [[차원 축소와 SVD]]
- [[수치 안정성]] · [[선형계]] · [[볼록 최적화]]
- [[복소수와 푸리에 변환]] · [[그래프 이론과 확률 과정]] (얕게)

## 고전 ML
- [[머신러닝이란]] → [[선형 회귀]] · [[로지스틱 회귀]]
- [[모델 평가]] · [[편향-분산 트레이드오프]] — LLM 평가·RAG 검색 품질과 직결되는 핵심 노트
- [[결정 트리]] · [[SVM]] · [[KNN]] · [[나이브 베이즈]] · [[비지도 학습]]
- [[앙상블]] · [[하이퍼파라미터 튜닝]] · [[ML 파이프라인]]
- [[피처 엔지니어링]] · [[피처 선택]] · [[시계열]] · [[이상 탐지]] · [[불균형 데이터]]

## 딥러닝 핵심
- [[퍼셉트론]] → [[다층 신경망과 순전파]] → [[역전파]]
- [[활성화 함수]] · [[손실 함수]]
- [[옵티마이저 (SGD·Adam)]] → [[학습률 스케줄]]
- [[정규화 (드롭아웃·가중치 감쇠)]] · [[가중치 초기화]]
- [[신경망 디버깅]]

## 트랜스포머 아키텍처
- [[Transformer]] → [[Query·Key·Value]] → [[Multi-Head Attention]] · [[Causal Masking]]
- [[위치 인코딩]]
- [[BERT와 MLM]] / [[GPT와 인과적 언어모델링]] / [[인코더-디코더 모델]] — 인코더·디코더·인코더-디코더 세 갈래
- [[MoE (Mixture of Experts)]]
- [[어텐션 변형 (GQA·MQA)]]
- [[스케일링 법칙]]

## LLM 학습·정렬
- [[사전학습 데이터 파이프라인]] → [[사전학습(Pretraining)]] → [[SFT (Instruction Tuning)]]
- [[RLHF]] / [[DPO]] · [[Constitutional AI]]
- [[LLM 평가]]
- [[Gradient Checkpointing]] · [[Multi-Token Prediction]] · [[DualPipe 병렬화]]
- [[SSM과 하이브리드 아키텍처 (Jamba)]]
- [[DeepSeek-V3 아키텍처]] · [[오픈 모델 아키텍처 비교]] — 위 이론들이 실제 모델에서 맞물리는 사례

## LLM 엔지니어링·에이전트
- [[프롬프트 엔지니어링]] · [[구조화된 출력]]
- [[LoRA (파인튜닝)]] · [[함수 호출 (Tool Use)]]
- [[가드레일]] · [[프롬프트 캐싱과 비용]]
- [[MCP (Model Context Protocol)]] · [[에이전트 상태 관리 (LangGraph)]] (둘 다 얕게 — 별도 전문 커리큘럼)
- [[Tool-Use 벤치마크 (BFCL)]] — 함수 호출 능력을 정량 비교하는 벤치마크
- [[싱글턴과 멀티턴 (Tool Use)]] — BFCL이 나누는 두 난이도 축, 코딩 에이전트는 사실상 항상 멀티턴
- [[멀티턴과 멀티에이전트 (구분)]] — "몇 번 대화하나"와 "몇 명이 일하나"는 다른 축이라는 구분
- [[에이전트 프레임워크 선택 기준]] — 헤르메스(완성품+가드레일) vs LangGraph(직접 구축) 중 무엇을 고를지의 종합 판단
- [[Cline]] · [[Continue.dev]] — 이 루프를 실제 코딩 작업에 적용한 오픈소스 클라이언트 두 갈래
- [[ACP (Agent Client Protocol)]] — 에이전트를 여러 에디터에 동시 통합하는 표준 프로토콜(MCP의 자매 개념)
- [[Hermes Agent]] — 코딩 에이전트를 포함해 멀티에이전트·영속 메모리·음성까지 아우르는 개인 AI 운영체제형 플랫폼

## RAG와 리랭킹 딥다이브

[[RAG와 리랭킹 MOC]]에서 시작합니다.

- [[Precision과 Recall]] → [[하이브리드 검색]] → [[RRF]] → [[리랭킹]]
- [[청킹과 문서 구조]] · [[쿼리 재작성과 확장]] · [[컨텍스트 구성]]
- [[Bi-Encoder와 Cross-Encoder]] · [[Late Interaction]] · [[ANN 검색과 검색 재현율]]
- [[RAG 평가]] · [[BGE Reranker]]

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
- [[모델 서빙과 서비스 배포]] → [[멀티모델 서빙]]

## 관측성·운영·보안 게이트웨이
[[모델 서빙과 서비스 배포]] 이후 실제 운영에서 붙는 계층입니다. [EP19](../docs/구축기-v1.1-전사확대/EP19-컴플라이언스-프레임워크-매핑.md)·[EP20](../docs/구축기-v1.1-전사확대/EP20-AI-게이트웨이-보안-강화.md)에서 다룬 전사 확대 단계의 컴플라이언스·게이트웨이 강화가 이 섹션의 배경입니다.
- [[관측성]] → [[상관관계 ID]] → [[분산 트레이싱]] · [[로그 집계]]
- [[Prometheus]] → [[Alertmanager]] · [[DCGM Exporter]]
- [[Grafana]] · [[Loki]] · [[OpenTelemetry]] — 시각화·로그·계측 표준
- [[Goodput]] — SLO를 만족한 요청만 세는 처리량 지표
- [[카오스 엔지니어링]] — 장애를 주입해 복원력을 사전 검증
- [[PII 마스킹]] → [[Presidio]] · [[프롬프트 인젝션]] — 게이트웨이 계층 보안
- [[모델 라우팅]] · [[시맨틱 캐싱]] — 게이트웨이 계층 비용·성능 최적화
- [[고영향 AI와 인간개입]] · [[감사 로그와 WORM]] — 금융·공공 폐쇄망 컴플라이언스 요구

## 서빙 엔진 / 도구
- [[Ollama]] · [[vLLM]] · [[LM Studio]] · [[GPT4All]] · [[Open WebUI]] · [[llama.cpp]] · [[SGLang]] · [[TensorRT-LLM]] · [[Hugging Face TGI]] · [[LocalAI]] · [[NVIDIA NIM]]
- [[LiteLLM]] (API 게이트웨이) · [[nginx]] (리버스 프록시) · [[Podman]] (컨테이너 런타임) · [[Continue.dev]] · [[Cline]] · [[Hermes Agent]] (코딩 어시스턴트 활용 사례) · [[faster-whisper]] (Hermes의 로컬 STT) · [[Qdrant]] (벡터DB) · [[RHEL 9]] (운영체제)

## 모델
- [[Qwen]] · [[Llama (모델)]] · [[Gemma]] · [[Mistral]] · [[EXAONE]] · [[HyperCLOVA X]] · [[SOLAR]] · [[Midm 2.0]]
- [[BGE-M3]] (임베딩 전용 모델)

## Windows/WSL2 실습 인프라
GPU 없이 이 스택을 실습하려고 Windows + WSL2로 [[RHEL 9]] 계열([[Podman]])을 재현하는 과정에서 겪은 가상화·네트워킹 개념들입니다.
- [[Hyper-V]] → [[가상머신]] → [[가상 스위치 포트]]
- [[WSL2]] → [[바인드 주소]] · [[Hyper-V 방화벽]]
- [[Hyper-V 방화벽]] → [[New-NetFirewallHyperVRule]] → [[VMCreatorId]] → [[GUID]]

## 실제 운영 사례
- [[한국은행 사례]] · [[미래에셋 사례]] · [[삼성SDS FabriX]] · [[LG CNS AgenticWorks]] · [[SK C&C]] · [[KT Managed AI GPU]] · [[금융권 AI 플랫폼 정책]] · [[IBM watsonx.ai]] · [[AWS Outposts]]

## 학습 로드맵 (0~7단계)
[[0단계 개념 다지기]] → [[1단계 첫 로컬 실행]] → [[2단계 채팅 UI]] → [[3단계 모델·양자화 비교]] → [[4단계 RAG 실습]] → [[5단계 프로덕션 서빙]] → [[6단계 폐쇄망 이관]] → [[7단계 운영·보안]]

[hoft.tistory.com 시리즈](https://hoft.tistory.com/entry/airgap-llm-survival-ep01-why-local-llm)의 목차를 참고해 직접 이어 쓴 [우리 구축기(EP02~18, docs/구축기-v1.0/)](../docs/구축기-v1.0/README.md)의 5단계 분류로 보면 대략 이렇게 겹칩니다 — 기초([[0단계 개념 다지기]] · [[GPU 선택]]) → 인터넷망 준비([[1단계 첫 로컬 실행]]~[[3단계 모델·양자화 비교]]) → 폐쇄망 설치([[5단계 프로덕션 서빙]] · [[6단계 폐쇄망 이관]], 도구는 [[Podman]] · [[LiteLLM]]) → 실전 운영([[7단계 운영·보안]]) → 고급 활용([[4단계 RAG 실습]] · [[Continue.dev]]). 순서는 우리 로드맵대로 두고, 이건 "실제 운영에서는 이렇게도 묶인다"는 참고용 시각입니다.
