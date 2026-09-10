---
tags: [concept, llm-basics, serving]
aliases: ["PTQ", "QAT", "Post-Training Quantization", "Quantization-Aware Training", "양자화 후 학습", "양자화 인지 학습"]
---

# PTQ와 QAT

[[양자화]]는 "언제 개입하느냐"에 따라 두 갈래로 나뉩니다 — 학습이 다 끝난 뒤에 누르는 **PTQ**와, 학습 도중에 미리 눌러보면서 학습시키는 **QAT**입니다.

## PTQ (Post-Training Quantization) — 학습 후 양자화

1. FP16/BF16으로 모델을 다 학습시킵니다 (양자화는 전혀 고려하지 않음).
2. 학습이 끝나 이미 고정된 [[가중치]]를, [[스케일과 제로포인트]] 공식으로 4비트·8비트 정수로 눌러 담습니다.

지금까지 다룬 [[GGUF]]·[[AWQ]]·[[GPTQ]]는 전부 PTQ입니다. 장점은 빠르고 쌉니다 — 추가 학습 없이 기존 체크포인트에 변환 작업만 몇 시간~하루 돌리면 끝입니다. 단점은 "이미 정해놓은 가중치를 나중에 억지로 뭉개는 것"이라, 특히 4비트 이하로 내려가면 특정 레이어에서 오차가 누적돼 정확도가 눈에 띄게 떨어질 수 있습니다.

## QAT (Quantization-Aware Training) — 양자화를 고려한 학습

학습(또는 파인튜닝) 도중에 "이 가중치는 나중에 4비트로 뭉개질 것"임을 미리 시뮬레이션하면서 학습시킵니다.

- forward pass에서 가중치를 일부러 양자화했다가 복원하는 연산(fake quantization)을 끼워 넣어, 모델이 "양자화 오차가 낀 상태"로 예측하고 손실을 계산하게 만듭니다.
- 이 오차가 반영된 손실을 역전파로 되돌려, 가중치 자체가 "나중에 뭉개져도 성능이 덜 깨지는 값"으로 조정되도록 학습합니다. 반올림 연산은 미분이 안 되므로 straight-through estimator라는 트릭으로 역전파를 우회시킵니다.

같은 4비트라도 PTQ보다 정확도 손실이 작지만, 학습 파이프라인·데이터·GPU 시간이 다시 필요해 비용이 훨씬 큽니다.

## 실제 공개 모델들의 지원 현황 (2026년 기준)

정리하면, 지금 공개된 모델 중 "우리가 QAT를 직접 돌린다"는 경우는 사실상 없고, 대부분 "모델 제작사가 QAT까지 미리 해서 공개한 체크포인트를 그냥 받아쓰거나", 그마저도 없이 PTQ만 쓰는 쪽입니다.

| 모델 | 공식 QAT 체크포인트 | 공식 PTQ 지원 | 비고 |
|---|---|---|---|
| **Gemma 3 (Google)** | ✅ 있음 — int4·Q4_0 QAT 체크포인트를 구글이 직접 배포 | ✅ 커뮤니티 GGUF/AWQ 다수 | BF16과 거의 동급 품질 유지하며 용량 최대 4배 축소 |
| **Llama 3.2 1B/3B (Meta)** | ✅ 있음 — QAT+LoRA 방식으로 메타가 직접 배포 | ✅ SpinQuant(PTQ)도 메타가 별도 공식 배포 | 온디바이스(모바일)용, 두 방식을 나란히 공개해 비교 가능 |
| **Llama 3.1/3.3 70B 이상, Llama 4** | ❌ 없음 | ✅ 커뮤니티 GGUF/AWQ/GPTQ | 대형 모델은 QAT 없이 PTQ만 |
| **Qwen3** | ❌ 없음 | ✅ 공식 GGUF·AWQ·FP8 (Alibaba 직접 배포), GPTQ는 공식 미지원 이슈 있음 | 회사가 직접 PTQ 변환본을 내놓는 드문 사례 |
| **Mistral** | ❌ 없음 | ⚠️ 공식은 거의 없고 대부분 커뮤니티(TheBloke 등) GGUF/AWQ/GPTQ | 회사 자체 PTQ 배포는 드묾 |
| **DeepSeek-V3 / R1** | △ 애매함 — QAT라기보다 "처음부터 FP8로 학습"(네이티브 저정밀 학습) | ✅ 커뮤니티 GGUF/AWQ 다수 | 학습 자체가 FP8이라 이 노트의 PTQ/QAT 이분법 밖에 있는 제3의 방식 |
| **Kimi K2 Thinking (Moonshot AI)** | ✅ 있음 — MoE 가중치에 INT4 QAT를 직접 적용해 공식 배포 | ✅ 커뮤니티 GGUF(Unsloth·BatiAI 등) 다수, 공식 GGUF는 아님 | 네이티브 INT4 추론으로 FP16 대비 속도 2배, 메모리 절반 — Gemma·Llama와 함께 회사가 QAT를 직접 돌린 몇 안 되는 사례 |

실무적으로 기억할 포인트는 두 가지입니다.

1. QAT 공식 체크포인트를 직접 낸 회사는 지금 **Google(Gemma)**, **Meta(Llama 3.2 소형)**, **Moonshot AI(Kimi K2 Thinking)** 정도뿐입니다 — 나머지는 다 PTQ.
2. DeepSeek는 QAT도 PTQ도 아닌 "애초에 FP8로 학습"이라는 세 번째 길을 갔다는 점이 특이합니다. Kimi(Moonshot)도 계보상 DeepSeek과 가까운 MoE 아키텍처라 네이티브 FP8 학습을 같이 쓰면서, 거기서 한 단계 더 나아가 MoE 부분만 INT4 QAT까지 얹은 조합형입니다.

### 포맷별로 돌아가는 GPU가 다르다

같은 표에 있는 포맷이라도 "어느 GPU에서 실제로 빠르게 돌아가느냐"는 또 다른 문제입니다. [[양자화]] 노트에서 다룬 원칙이 여기서도 그대로 적용됩니다 — 포맷이 요구하는 GPU 세대(compute capability)가 낮을수록 더 많은 GPU에서 돌아갑니다.

| 포맷 | 최소 요구 GPU 세대 | 비고 |
|---|---|---|
| **GGUF** | 없음 (CPU도 가능) | Apple Silicon·CPU까지 포괄, 가장 폭넓게 돌아감 |
| **GPTQ** | 구형 GPU도 지원 | V100(Volta)급도 커버 |
| **AWQ** | Ampere 이상 (compute capability 7.5+) | V100(Volta, 7.0)에서는 아예 안 돌아감 |
| **FP8** | Hopper(H100) 이상 | A100(Ampere)은 전용 회로가 없어 소프트웨어 흉내만 가능, 속도 이득 거의 없음 |
| **INT4 QAT (Kimi K2 Thinking 등 MoE 전용)** | vLLM·SGLang 등 INT4 MoE 커널을 지원하는 최신 서빙 엔진 필요 | 포맷 자체보다 "이 커널을 구현한 서빙 엔진을 쓰느냐"가 관건 |

즉 표 위쪽 모델 중 QAT 체크포인트가 있어도, 그걸 실제로 빠르게 돌리려면 그 포맷을 지원하는 GPU 세대와 서빙 엔진(커널)이 갖춰져 있어야 합니다 — [[GPU 선택]]을 양자화 전략보다 먼저 정해야 하는 이유가 여기서도 똑같이 적용됩니다.

### 출처
- [Gemma 3 QAT Models — Google Developers Blog](https://developers.googleblog.com/en/gemma-3-quantized-aware-trained-state-of-the-art-ai-to-consumer-gpus/)
- [Meta — Introducing quantized Llama models](https://ai.meta.com/blog/meta-llama-quantized-lightweight-models/)
- [Qwen 공식 X 발표 — Qwen3 양자화 모델](https://x.com/Alibaba_Qwen/status/1921907010855125019)
- [Qwen 공식 문서 — GPTQ](https://qwen.readthedocs.io/en/latest/quantization/gptq.html)
- [DeepSeek-V3 README_WEIGHTS.md](https://github.com/deepseek-ai/DeepSeek-V3/blob/main/README_WEIGHTS.md)
- [TheBloke Mistral 양자화 배포 사례](https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.2-GGUF)
- [moonshotai/Kimi-K2-Thinking — Hugging Face](https://huggingface.co/moonshotai/Kimi-K2-Thinking)
- [Kimi K2 Thinking 네이티브 INT4 QAT 설명 (Turing Post)](https://x.com/TheTuringPost/status/1989001234217594944)

## 폐쇄망 실무에서는 왜 PTQ만 다루는가

"이미 나온 오픈 모델을 우리 GPU에 맞게 눌러 쓰는" 시나리오에서는 QAT를 직접 돌리는 경우가 드뭅니다. 우리가 학습 데이터·학습 파이프라인을 갖고 있지 않기 때문입니다. 대부분 모델 제작사가 QAT까지 마친 체크포인트를 미리 공개하거나(일부 4비트 네이티브 모델), 우리는 배포 단계에서 PTQ만 적용하는 쪽입니다. 이 볼트의 [[양자화]]·[[GGUF]]·[[AWQ]]·[[GPTQ]] 노트가 전부 PTQ 관점으로 쓰여 있는 이유이기도 합니다.

## 관련
- [[양자화]] — 이 노트가 전제하는 "정수로 눌러 담는다"는 개념 자체
- [[가중치]] — 양자화의 압축 대상, 정밀도가 필요한 이유
- [[스케일과 제로포인트]] — PTQ가 실제로 쓰는 변환 공식
- [[GGUF]] · [[AWQ]] · [[GPTQ]] — 전부 PTQ 방식의 구체적 포맷
- [[Gemma]] · [[Llama (모델)]] — 공식 QAT 체크포인트를 직접 배포하는 모델 계열 (Kimi K2 Thinking도 여기 속함)
- [[Qwen]] · [[DeepSeek-V3 아키텍처]] — PTQ 위주(또는 네이티브 FP8 학습)인 다른 사례
- [[GPU 선택]] — 포맷별 GPU/커널 제약을 실제로 GPU 구매 결정에 반영하는 다음 단계
