# 볼트 확장 핸드오프 — 기초 이론 노트 추가

이 문서는 [aiengineeringfromscratch.com](https://aiengineeringfromscratch.com/) (GitHub: [rohitg00/ai-engineering-from-scratch](https://github.com/rohitg00/ai-engineering-from-scratch), MIT 라이선스, 523레슨/20phase 커리큘럼)을 참고해서, 지금 `vault/`에 없는 **수학·고전 ML·딥러닝·트랜스포머 이론** 레이어를 추가하기 위한 작업 지시서입니다.

**이 문서는 여러 세션에 나눠 배포됩니다.** 아래 "패키지 0"은 모든 세션이 공통으로 지켜야 하는 규칙이고, "패키지 1~6"은 서로 독립적인 작업 단위입니다. 세션 하나당 패키지 하나만 맡으세요 (동시에 여러 패키지를 하면 안 됩니다 — 이유는 패키지 0의 "충돌 방지" 항목 참고).

---

## 패키지 0 — 공통 지침 (모든 세션 필독)

### 지금 이 저장소의 상태
`vault/`는 이미 "폐쇄망 LLM 서빙" 실무 위주로 구축돼 있습니다. `vault/MOC.md`가 진입점이고, `vault/concepts/`, `vault/engines/`, `vault/models/`, `vault/cases/`, `vault/roadmap/`로 나뉩니다. 지금 추가하려는 건 이 실무 노트들이 "왜 그렇게 동작하는가"를 받쳐주는 **이론 레이어**입니다. 예: `vault/concepts/양자화.md`는 이미 있지만 스케일/제로포인트를 어떻게 계산하는지의 선형대수는 없고, `vault/concepts/Multi-Head Attention.md`는 있지만 역전파가 왜 필요한지, 손실 함수가 뭔지는 없습니다.

### 소스 사용 규칙 (중요)
- 레슨 문서는 `https://github.com/rohitg00/ai-engineering-from-scratch/blob/main/phases/<phase-폴더>/<레슨-폴더>/docs/en.md` 에 있습니다 (raw 텍스트는 `blob/main`을 `raw.githubusercontent.com/.../main`으로 바꾸면 됩니다).
- **레슨 원문을 요약 복사/번역해서 붙여넣지 마세요.** 읽고 개념을 이해한 다음, 이 vault의 문체로 **새로 설명을 쓰세요** (아래 "노트 작성 컨벤션" 참고). MIT 라이선스라 법적으로는 재사용이 자유롭지만, 이 vault의 가치는 "원자화된 우리말 설명"이지 원문 번역이 아닙니다.
- 코드 스니펫이 개념 이해에 꼭 필요하면 1~3줄 정도의 핵심만 인용하고, 레슨의 전체 구현을 그대로 옮기지 마세요.

### 노트 작성 컨벤션 (기존 노트에서 관찰된 규칙)
기존 노트 3개(`vault/concepts/Query·Key·Value.md`, `Multi-Head Attention.md`, `vault/MOC.md`)를 먼저 읽고 감을 잡으세요. 핵심 패턴:

```markdown
---
tags: [concept, ...]
aliases: ["영문 표기", "한글 표기", "약어"]
---

# 노트 제목 (개념 하나, 원자적으로)

개념을 정의하고, 구체적인 숫자나 비유로 감을 잡아준다. "왜 이렇게 설계됐는가"가
비자명하면 별도 문단이나 "## 왜 ~인가" 섹션으로 설명한다. 메커니즘이 그림으로
설명하는 게 나으면 mermaid flowchart를 넣는다 (Query·Key·Value.md 참고).

## 관련
- [[다른 노트]] — 관계를 한 줄로
```

- 한 노트 = 개념 하나. 여러 개념을 욱여넣지 않습니다.
- 제목은 한국 엔지니어들이 실제로 부르는 이름 우선 (영문이 관용어면 영문 그대로: 예 `Multi-Head Attention`, 한글이 자연스러우면 한글: 예 `가중치`, `배치`).
- `aliases`에 영/한/약어를 다 넣어서 어느 쪽으로 검색해도 찾히게 합니다.
- 설명은 "왜"에 방점 — 정의만 나열하지 말고, 이게 없으면 뭐가 안 되는지, 실무 노트(서빙/양자화 등)와 어떻게 연결되는지 짚어줍니다.
- 코드는 지양하고 수식+말로 풀어씁니다 (이 vault는 "이해용" 노트지 구현 튜토리얼이 아닙니다). 수식이 필요하면 인라인으로 짧게.

### 기존 노트와 겹치는 개념은 "새로 만들지 말고 보강"
아래 개념들은 이미 파일이 있습니다. 겹치는 레슨을 다룰 때는 **새 파일을 만들지 말고 기존 파일에 이론 섹션을 추가**하세요 (Read 먼저, Edit로 섹션 추가 — 기존 내용을 지우거나 통째로 다시 쓰지 마세요):
`Query·Key·Value.md` · `Multi-Head Attention.md` · `Causal Masking.md` · `KV 캐시.md` · `양자화.md` · `스케일과 제로포인트.md` · `Speculative Decoding.md` · `RAG.md` · `청킹.md` · `임베딩과 벡터DB.md` · `하이브리드 검색.md` · `환각.md` · `토큰.md` · `PagedAttention.md` · `처리량과 지연시간.md` · `Tensor Parallelism.md` · `Pipeline Parallelism.md`

각 패키지에서 어떤 레슨이 여기에 해당하는지 표시해뒀습니다.

### 충돌 방지 — MOC.md는 아무도 직접 수정하지 마세요
`vault/MOC.md` 한 파일을 6개 세션이 동시에 건드리면 병합 충돌이 납니다. 대신:
1. 새로 만든 노트 목록(위키링크 한 줄씩)을 **자기 패키지 섹션 맨 아래에 있는 "MOC에 추가할 목록" 항목**에 그대로 적어두세요.
2. `vault/MOC.md`는 건드리지 말고 그대로 두세요.
3. 모든 패키지가 끝난 뒤, 사람이 (또는 마지막에 지정된 세션 하나가) 6개 패키지의 목록을 모아 `MOC.md`에 한 번에 반영합니다.

### 완료 체크
각 패키지 끝에 있는 레슨 목록을 다 처리했으면, 세션 마지막에 `git status`로 새로 만든/수정한 파일을 확인하고 무엇을 했는지 요약하세요. **커밋은 하지 마세요** (사용자가 전체를 검토한 뒤 한 번에 커밋합니다).

---

## 패키지 1 — 수학 기초 (Phase 1: Math Foundations)

새 폴더 `vault/theory/math/` 를 만들어서 원자 노트를 씁니다. 목표는 "왜 이 수학이 LLM/서빙 노트에 등장하는가"를 짚어주는 것 — 순수 수학 교과서가 아니라, 예를 들어 `벡터`, `행렬` 노트는 나중에 임베딩·어텐션 노트에서 링크를 받습니다.

레슨 경로: `phases/01-math-foundations/<슬러그>/docs/en.md`

| 슬러그 | 제안 노트 제목 |
|---|---|
| 01-linear-algebra-intuition, 02-vectors-matrices-operations | 벡터와 행렬 |
| 03-matrix-transformations | 행렬 변환 (선형 변환) |
| 04-calculus-for-ml | 미분과 그래디언트 |
| 05-chain-rule-and-autodiff | 연쇄법칙과 자동미분 |
| 06-probability-and-distributions | 확률분포 |
| 07-bayes-theorem | 베이즈 정리 |
| 08-optimization | 경사하강법 |
| 09-information-theory | 정보이론 (엔트로피·교차엔트로피·KL divergence) |
| 10-dimensionality-reduction | 차원 축소 |
| 11-singular-value-decomposition | 특이값분해(SVD) |
| 12-tensor-operations | 텐서 연산 |
| 13-numerical-stability | 수치 안정성 (오버플로/언더플로) |
| 14-norms-and-distances | 노름과 거리 |
| 15-statistics-for-ml | 통계 기초 (평균·분산·정규화) |
| 16-sampling-methods | 샘플링 방법 (temperature·top-k·top-p의 이론적 배경) |
| 17-linear-systems | 선형계 |
| 18-convex-optimization | 볼록 최적화 |
| 19-complex-numbers | 복소수 |
| 20-fourier-transform | 푸리에 변환 |
| 21-graph-theory | 그래프 이론 |
| 22-stochastic-processes | 확률 과정 |

전부 신규 노트입니다 (기존 파일과 겹치지 않음). 개수가 많으니 관련 있는 것끼리 하나로 합쳐도 됩니다 (예: 19/20/21/22는 LLM과 직결도가 낮으니 얕게, 04/05/08/09/16은 딥러닝·서빙과 바로 연결되니 깊게). 09(정보이론)는 나중에 `양자화.md`(스케일/제로포인트가 정보손실과 관련), 16(샘플링)은 나중에 서빙 노트의 `temperature`/`top-p` 관련 설명과 연결해두면 좋습니다.

MOC에 추가할 목록: (완료 후 여기에 실제 만든 노트의 위키링크를 채워넣으세요)

---

## 패키지 2 — 고전 ML (Phase 2: ML Fundamentals)

새 폴더 `vault/theory/ml-classical/`. 이 패키지는 "LLM이 아닌 전통적 ML"이라 우선순위가 가장 낮습니다 — 시간이 부족하면 스킵해도 되는 패키지라고 사용자에게 미리 알려주세요. 다만 `모델 평가`, `과적합/편향-분산`, `앙상블` 정도는 나중에 LLM 평가(Phase 10 evaluation) 노트에서도 재사용되는 개념이라 우선순위를 높게 잡습니다.

레슨 경로: `phases/02-ml-fundamentals/<슬러그>/docs/en.md`

| 슬러그 | 제안 노트 제목 | 우선순위 |
|---|---|---|
| 01-what-is-machine-learning | 머신러닝이란 | 중 |
| 02-linear-regression | 선형 회귀 | 중 |
| 03-logistic-regression | 로지스틱 회귀 | 중 |
| 04-decision-trees | 결정 트리 | 하 |
| 05-support-vector-machines | SVM | 하 |
| 06-knn-and-distances | KNN | 하 |
| 07-unsupervised-learning | 비지도 학습 (클러스터링) | 하 |
| 08-feature-engineering | 피처 엔지니어링 | 하 |
| 09-model-evaluation | 모델 평가 (정밀도·재현율·F1) | **상** |
| 10-bias-variance | 편향-분산 트레이드오프 | **상** |
| 11-ensemble-methods | 앙상블 (배깅·부스팅) | 하 |
| 12-hyperparameter-tuning | 하이퍼파라미터 튜닝 | 하 |
| 13-ml-pipelines | ML 파이프라인 | 하 |
| 14-naive-bayes | 나이브 베이즈 | 하 |
| 15-time-series | 시계열 | 하 |
| 16-anomaly-detection | 이상 탐지 | 하 |
| 17-imbalanced-data | 불균형 데이터 | 하 |
| 18-feature-selection | 피처 선택 | 하 |

전부 신규 노트. "하" 우선순위는 한 문단짜리 짧은 노트로 충분합니다 (다른 개념의 사전지식용).

MOC에 추가할 목록: (완료 후 채워넣기)

---

## 패키지 3 — 딥러닝 핵심 (Phase 3: Deep Learning Core)

새 폴더 `vault/theory/dl-core/`. 이 패키지가 사실상 이 프로젝트에 가장 크게 빠져있던 구멍입니다 — 지금 vault는 "학습된 모델을 어떻게 서빙하는가"만 다루고 "그 모델이 어떻게 학습되는가"가 전혀 없습니다.

레슨 경로: `phases/03-deep-learning-core/<슬러그>/docs/en.md`

| 슬러그 | 제안 노트 제목 |
|---|---|
| 01-the-perceptron | 퍼셉트론 |
| 02-multi-layer-networks | 다층 신경망과 순전파 |
| 03-backpropagation | 역전파 |
| 04-activation-functions | 활성화 함수 |
| 05-loss-functions | 손실 함수 |
| 06-optimizers | 옵티마이저 (SGD·Adam) |
| 07-regularization | 정규화 (드롭아웃·가중치 감쇠) |
| 08-weight-initialization | 가중치 초기화 |
| 09-learning-rate-schedules | 학습률 스케줄 |
| 10-mini-framework | (스킵 가능 — 프레임워크 구현 실습, 이론 노트로 만들 내용 적음) |
| 11-intro-to-pytorch | (스킵 가능 — 도구 소개) |
| 12-intro-to-jax | (스킵 가능 — 도구 소개) |
| 13-debugging-neural-networks | 신경망 디버깅 (loss NaN 등 — `03-트러블슈팅-딥리서치.md`와 연결 고려) |

전부 신규 노트 (10/11/12는 이론이 아니라 도구 사용법이라 제외 권장). `02 다층 신경망과 순전파`는 나중에 `vault/concepts/`의 어텐션 노트들이 어텐션도 결국 순전파의 한 종류라는 걸 짚어줄 때 링크 대상이 됩니다. `03 역전파`는 패키지 1의 `연쇄법칙과 자동미분`을 참조하게 하세요.

MOC에 추가할 목록: (완료 후 채워넣기)

---

## 패키지 4 — 트랜스포머 딥다이브 (Phase 7: Transformers Deep Dive)

여기부터는 `vault/concepts/`(새 폴더가 아니라 기존 폴더)에 씁니다 — 이미 어텐션 계열 노트가 거기 있기 때문입니다. **패키지 0의 "기존 노트와 겹치는 개념" 표를 반드시 확인하세요.**

레슨 경로: `phases/07-transformers-deep-dive/<슬러그>/docs/en.md`

| 슬러그 | 처리 방법 |
|---|---|
| 01-why-transformers | 신규 노트 `Transformer.md` (RNN 대비 왜 병렬화가 되는지 — 개괄) |
| 02-self-attention-from-scratch | **기존 파일 `Query·Key·Value.md`에 보강** — 지금 없는 "학습 과정에서 Wq·Wk·Wv가 어떻게 업데이트되는가"(역전파 관점) 섹션 추가 |
| 03-multi-head-attention | **기존 파일 `Multi-Head Attention.md`에 보강** — 필요시만, 대부분 이미 커버됨 |
| 04-positional-encoding | 신규 노트 `위치 인코딩.md` |
| 05-full-transformer | 신규 노트 `Transformer.md`에 통합 (인코더-디코더 전체 구조) |
| 06-bert-masked-language-modeling | 신규 노트 `BERT와 MLM.md` |
| 07-gpt-causal-language-modeling | 신규 노트 `GPT와 인과적 언어모델링.md` — **기존 `Causal Masking.md`와 상호 링크 필수** |
| 08-t5-bart-encoder-decoder | 신규 노트 `인코더-디코더 모델.md` |
| 09-vision-transformers | 스킵 권장 (LLM 서빙 vault와 관련도 낮음, 필요시 짧게만) |
| 10-audio-transformers-whisper | 스킵 권장 |
| 11-mixture-of-experts | 신규 노트 `MoE (Mixture of Experts).md` — 최신 모델(DeepSeek 등) 이해에 중요 |
| 12-kv-cache-flash-attention | **기존 파일 `KV 캐시.md`에 보강** — FlashAttention 부분만 추가 |
| 13-scaling-laws | 신규 노트 `스케일링 법칙.md` |
| 14-build-a-transformer-capstone | 스킵 (실습 캡스톤, 노트화할 이론 없음) |
| 15-attention-variants | 신규 노트 `어텐션 변형 (GQA·MQA 등).md` |
| 16-speculative-decoding | **기존 파일 `Speculative Decoding.md`에 보강** — 필요시만 |

MOC에 추가할 목록: (완료 후 채워넣기)

---

## 패키지 5 — LLM 자체 만들기 (Phase 10: LLMs from Scratch)

계속 `vault/concepts/`에 씁니다.

레슨 경로: `phases/10-llms-from-scratch/<슬러그>/docs/en.md`

| 슬러그 | 처리 방법 |
|---|---|
| 01-tokenizers, 02-building-a-tokenizer | **기존 파일 `토큰.md`에 보강** — BPE 알고리즘 자체가 지금 없다면 추가 |
| 03-data-pipelines | 신규 노트 `사전학습 데이터 파이프라인.md` (얕게) |
| 04-pre-training-mini-gpt | 신규 노트 `사전학습(Pretraining).md` |
| 05-scaling-distributed | **기존 파일 `Tensor Parallelism.md`/`Pipeline Parallelism.md`에 보강** — 지금은 추론 시점 병렬화만 있다면 "학습 시점" 관점 추가 |
| 06-instruction-tuning-sft | 신규 노트 `SFT (Instruction Tuning).md` |
| 07-rlhf | 신규 노트 `RLHF.md` |
| 08-dpo | 신규 노트 `DPO.md` — RLHF.md와 상호 링크 |
| 09-constitutional-ai-self-improvement | 신규 노트 `Constitutional AI.md` (얕게) |
| 10-evaluation | 신규 노트 `LLM 평가.md` — 패키지 2의 `모델 평가`와 링크 |
| 11-quantization | **기존 파일 `양자화.md`/`스케일과 제로포인트.md`에 보강** — 지금 없는 수학적 유도가 있으면 추가, 대부분 이미 커버됐을 가능성 높음 (먼저 읽고 판단) |
| 12-inference-optimization | **기존 파일 `처리량과 지연시간.md`에 보강** |
| 13-building-complete-llm-pipeline | 스킵 (실습 캡스톤) |
| 14-open-models-architecture-walkthroughs | 신규 노트 `오픈 모델 아키텍처 비교.md` — `vault/models/`의 기존 모델 노트들과 링크 |
| 15-speculative-decoding-eagle3, 25-speculative-decoding | **기존 파일 `Speculative Decoding.md`에 보강** |
| 16-differential-attention-v2 | 신규 노트 `Differential Attention.md` (최신 기법, 얕게) |
| 17-native-sparse-attention | 신규 노트 `Sparse Attention.md` |
| 18-multi-token-prediction | 신규 노트 `Multi-Token Prediction.md` |
| 19-dualpipe-parallelism | 신규 노트, `Pipeline Parallelism.md`와 링크 |
| 20-deepseek-v3-walkthrough | 신규 노트 `DeepSeek-V3 아키텍처.md` — MoE·MLA 등 실제 사례 |
| 21-jamba-hybrid-ssm-transformer | 신규 노트 `SSM과 하이브리드 아키텍처 (Jamba).md` |
| 22-async-hogwild-inference | 스킵 권장 (연구성 최신 기법, 우선순위 낮음) |
| 34-gradient-checkpointing | 신규 노트 `Gradient Checkpointing.md` |

이 패키지가 기존 vault와 겹치는 부분이 가장 많습니다 — **먼저 겹친다고 표시된 기존 파일들을 전부 Read해서, 이미 다뤄진 내용이면 손대지 말고 스킵**하세요. 빈 구멍만 채우는 게 목표입니다.

MOC에 추가할 목록: (완료 후 채워넣기)

---

## 패키지 6 — LLM 엔지니어링 (Phase 11: LLM Engineering)

계속 `vault/concepts/`에 씁니다. 이 패키지는 "이론"보다 "응용 개념"에 가까워서, 다른 패키지보다 실무 vault와 자연스럽게 잘 붙습니다.

레슨 경로: `phases/11-llm-engineering/<슬러그>/docs/en.md`

| 슬러그 | 처리 방법 |
|---|---|
| 01-prompt-engineering, 02-few-shot-cot | 신규 노트 `프롬프트 엔지니어링.md` (Few-shot·CoT 포함) |
| 03-structured-outputs | 신규 노트 `구조화된 출력.md` |
| 04-embeddings | **기존 파일 `임베딩과 벡터DB.md`에 보강** — 임베딩이 학습되는 원리(대조학습 등)가 없다면 추가 |
| 05-context-engineering | **기존 파일 `컨텍스트 윈도우.md`에 보강** |
| 06-rag, 07-advanced-rag | **기존 파일 `RAG.md`/`하이브리드 검색.md`에 보강** |
| 08-fine-tuning-lora | 신규 노트 `LoRA (파인튜닝).md` |
| 09-function-calling | 신규 노트 `함수 호출 (Tool Use).md` |
| 10-evaluation | 패키지 5의 `LLM 평가.md`와 중복 — 그쪽에 병합, 새로 만들지 말 것 |
| 11-caching-cost | **기존 파일 `KV 캐시.md`에 보강**, 또는 신규 `프롬프트 캐싱과 비용.md` |
| 12-guardrails | 신규 노트 `가드레일.md` |
| 13-production-app | 스킵 (실습 캡스톤) |
| 14-model-context-protocol | 신규 노트 `MCP (Model Context Protocol).md` (얕게 — 별도 전문 커리큘럼이라 여기선 개괄만) |
| 15-prompt-caching | 11번과 병합 |
| 16-langgraph-state-machines | 신규 노트 `에이전트 상태 관리 (LangGraph).md` (얕게) |
| 17-agent-framework-tradeoffs | 스킵 권장 (도구 비교, 이론 아님) |

MOC에 추가할 목록: (완료 후 채워넣기)

---

## 마무리 (모든 패키지 완료 후, 사람이 진행)

1. 6개 패키지의 "MOC에 추가할 목록"을 모아서 `vault/MOC.md`에 새 섹션(예: `## 이론 기초`)으로 추가하고, 기존 `## 기초 개념`/`## 서빙 이론` 섹션에도 보강된 노트가 있으면 화살표(`→`)로 자연스럽게 끼워넣습니다.
2. 겹치는 개념을 여러 패키지가 동시에 "신규 노트"로 판단해서 중복 파일이 생겼는지 확인합니다 (예: 패키지 5와 6이 둘 다 "LLM 평가" 노트를 만들었을 가능성).
3. `git status`로 전체 diff를 검토하고 커밋합니다.
