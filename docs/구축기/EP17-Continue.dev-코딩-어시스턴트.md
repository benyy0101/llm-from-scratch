# EP17. Continue.dev 코딩 어시스턴트

> 시리즈: [폐쇄망 LLM 구축기](README.md) · 이전: [EP16. 폐쇄망 RAG 구축 ②](EP16-폐쇄망-RAG-구축-2.md) · 다음: EP18. 시리즈 총정리 & 운영 회고 (예정)

EP01부터 계속 "현업팀 QA봇 + 개발팀 코딩 어시스턴트" 두 갈래라고 말씀드렸었는데, 드디어 개발팀 쪽 얘기예요. [[Continue.dev]]로 VS Code에 "사내판 Copilot"을 붙이는 편입니다.

## 🎯 자동완성용 모델을 어디에 태우느냐가 문제였어요

Continue.dev는 보통 자동완성(빠르고 작은 모델)이랑 채팅/편집(크고 똑똑한 모델)을 따로 씁니다. 채팅은 이미 떠 있는 Qwen2.5-32B로 되는데, 자동완성용으로 코드 특화 소형 모델(Qwen2.5-Coder-1.5B)을 하나 더 올려야 했어요. 문제는 [[VRAM]]이었죠 — EP08에서 이미 `--gpu-memory-utilization 0.90`으로 V100 3장을 32B 모델에 거의 다 내줬거든요.

이걸 해결한 방법은 두 가지를 같이 썼어요.

1. **32B 모델 쪽 여유를 살짝 양보**: `--gpu-memory-utilization`을 0.90에서 0.75로 낮췄어요. 동시 접속 15명 기준 벤치마크(EP08)를 다시 돌려봤는데, 이 정도 여유를 줄여도 목표(첫 토큰 3초)는 여전히 만족했습니다.
2. **자동완성 모델은 아주 작게**: Qwen2.5-Coder-1.5B를 GPTQ Int4로 양자화하니 2GB 안쪽으로 들어가서, 남는 자투리 VRAM에 세 번째 vLLM 인스턴스로 얹었어요.

```bash
podman run -d --name vllm-coder \
  --device nvidia.com/gpu=all \
  --network airgap-net \
  -v /opt/models/qwen2.5-coder-1.5b-gptq:/models/coder:Z \
  -p 8002:8000 \
  localhost:5000/vllm-v100:2026Q3 \
  vllm serve /models/coder --quantization gptq --gpu-memory-utilization 0.10
```

## 🚪 LiteLLM에 자동완성 모델 등록

```yaml
model_list:
  - model_name: qwen2.5-32b        # 채팅/편집용
    litellm_params:
      model: openai/qwen2.5-32b
      api_base: http://vllm-server:8000/v1
  - model_name: qwen2.5-coder-1.5b  # 자동완성용
    litellm_params:
      model: openai/qwen2.5-coder-1.5b
      api_base: http://vllm-coder:8000/v1
```

## 🖥️ Continue.dev 설정

개발자 PC의 VS Code 확장 설정 파일(`config.yaml`)이에요. 개발팀 전용 LiteLLM 가상 키(`dev-team`, EP10에서 발급)를 여기 씁니다.

```yaml
name: A저축은행 사내 Continue 설정
models:
  - name: 채팅/편집
    provider: openai
    model: qwen2.5-32b
    apiBase: http://litellm-gateway.internal:4000/v1
    apiKey: ${{ secrets.LITELLM_DEV_KEY }}
    roles: [chat, edit]

  - name: 자동완성
    provider: openai
    model: qwen2.5-coder-1.5b
    apiBase: http://litellm-gateway.internal:4000/v1
    apiKey: ${{ secrets.LITELLM_DEV_KEY }}
    roles: [autocomplete]
```

`apiBase`가 vLLM이 아니라 LiteLLM 게이트웨이를 가리키는 건 Open WebUI(EP11) 때랑 똑같은 이유예요 — 개발팀 사용량도 팀별 예산·감사 로그 안에서 관리돼야 하니까요.

## ✅ 실제 써보니

자동완성은 체감상 클라우드 Copilot보다 살짝 느리지만(로컬 1.5B급이니 당연하죠), 사내 코드베이스 특유의 네이밍 컨벤션이나 사내 라이브러리 함수를 더 잘 알아본다는 개발자 피드백이 있었어요. 채팅/편집(32B)은 일반적인 리팩터링·리뷰 요청에는 충분히 쓸 만하다는 평가였고, 아주 복잡한 멀티파일 리팩터링은 아직 사람이 더 잘한다는 게 공통된 의견이었습니다.

---

📌 **EP.17 한 줄 요약**
자동완성용 소형 코드 모델(1.5B)을 위해 32B 모델의 VRAM 점유율을 살짝 낮추고 남는 공간에 세 번째 vLLM 인스턴스를 얹었다. Continue.dev도 Open WebUI와 마찬가지로 LiteLLM 게이트웨이를 거치게 해 팀별 예산 관리 안에 뒀다.

**다음 편**: EP18. 시리즈 총정리 & 운영 회고 (예정)
