---
tags: [tool, speech, local]
aliases: ["Faster-Whisper"]
---

# faster-whisper

OpenAI Whisper 모델을 CTranslate2 런타임으로 재구현해 같은 정확도에서 훨씬 빠르게(그리고 더 적은 VRAM으로) 돌아가게 만든 음성 인식(STT) 라이브러리입니다. API 키나 외부 호출 없이 로컬에서만 처리되므로 [[폐쇄망]] 환경에 적합합니다.

[[Hermes Agent]]의 Voice Mode가 사용자 음성을 텍스트로 바꾸는 데 이 라이브러리를 씁니다 — 변환된 텍스트는 그 뒤로는 일반 텍스트 에이전트 파이프라인(메모리·도구·스킬)과 동일하게 처리됩니다. [[BGE-M3]] 같은 다른 로컬 모델과 마찬가지로, 모델 가중치 파일을 인터넷망에서 미리 내려받아 사전 반입해야 합니다.

## 관련
- [[Hermes Agent]] — 이 라이브러리를 Voice Mode로 사용하는 에이전트 플랫폼
- [[폐쇄망]] — 로컬 처리가 필요한 이유
