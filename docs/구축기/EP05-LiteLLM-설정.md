# EP05. LiteLLM 설정

> 시리즈: [폐쇄망 LLM 구축기](README.md) · 이전: [EP04. 컨테이너 이미지 & 모델 다운로드](EP04-컨테이너-이미지-모델-다운로드.md) · 다음: [EP06. 반입 패키지 제작](EP06-반입-패키지-제작.md)

EP02에서 4계층 아키텍처 얘기할 때 "접근 계층 앞에 게이트웨이를 하나 세운다"고 했었죠. 이번 편은 그 게이트웨이, [[LiteLLM]] 설정 얘기예요. 아직 폐쇄망에 반입도 안 한 시점인데 설정부터 하냐고 물으실 수 있는데, 설정 파일 자체를 반입 대상에 포함시켜야 해서 인터넷망에서 미리 만들어두는 거예요.

## 🎯 왜 vLLM을 바로 안 열어주고 게이트웨이를 두냐면

처음엔 "그냥 Open WebUI가 vLLM을 바로 보게 하면 되지 않나" 싶었는데, 팀 두 개(현업 40명, 개발팀 25명)가 같이 쓰는 순간부터 얘기가 달라져요.

- 현업팀이 던지는 요청 때문에 개발팀 코드 자동완성이 밀리면 안 돼요 — 팀별 요청량 제한이 필요해요.
- 누가 언제 뭘 물어봤는지 감사 로그가 있어야 해요 (EP11에서 더 다룰 거예요).
- 나중에 모델을 Qwen에서 다른 모델로 바꿀 때, 클라이언트(Open WebUI, Continue.dev) 설정은 안 건드리고 게이트웨이 쪽만 바꾸고 싶었어요.

이 세 가지가 전부 "vLLM 앞단에 뭔가 하나 더 있어야 한다"는 결론으로 이어졌고, [[표준 아키텍처]]에서 얘기했던 API 게이트웨이 자리를 LiteLLM으로 채웠습니다.

## 📋 설정 파일은 이렇게 짰어요

```yaml
# litellm-config.yaml
model_list:
  - model_name: qwen2.5-32b
    litellm_params:
      model: openai/qwen2.5-32b-gptq-int4
      api_base: http://vllm-server:8000/v1
      api_key: "dummy-key"

  - model_name: bge-m3-embed
    litellm_params:
      model: openai/bge-m3
      api_base: http://ollama-server:11434/v1
      api_key: "dummy-key"

general_settings:
  master_key: os.environ/LITELLM_MASTER_KEY
  database_url: os.environ/LITELLM_DB_URL

router_settings:
  routing_strategy: least-busy
  num_retries: 2
  timeout: 30
```

`api_base`가 전부 폐쇄망 내부 호스트명(`vllm-server`, `ollama-server`)을 가리키고 있고, 외부 URL은 하나도 없어요. LiteLLM 자체도 기본적으로 사용량 리포팅 같은 걸 어딘가로 보내려는 옵션이 있는 서비스라, 이런 자체 텔레메트리 옵션도 다 꺼서 설정에 명시했습니다. 폐쇄망 안의 어떤 구성요소도 스스로 인터넷에 나가면 안 된다는 원칙, EP02에서부터 계속 반복되는 얘기예요.

`master_key`나 `database_url`처럼 민감한 값은 설정 파일에 그대로 박지 않고 환경변수(`os.environ/...`)로 빼뒀어요. 이 파일 자체가 반입 심사를 거치는데, 심사하는 분들이 파일을 열어봤을 때 키 값이 그대로 노출돼 있으면 곤란하니까요.

## 팀별 가상 키(virtual key) 발급

LiteLLM이 뜬 다음에 실제로 발급하는 건 폐쇄망 안에서 하는 작업이지만, 어떤 값으로 발급할지는 미리 정해뒀어요.

```bash
curl -X POST http://litellm-gateway:4000/key/generate \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -d '{
    "models": ["qwen2.5-32b", "bge-m3-embed"],
    "team_id": "product-planning",
    "max_budget": 50,
    "budget_duration": "30d"
  }'
```

현업팀(`product-planning`)이랑 개발팀(`dev-team`)을 서로 다른 `team_id`로 나누고, 각자 예산(요청량 상한)을 따로 걸었어요. 이렇게 해두면 한 팀이 갑자기 요청을 몰아 보내도 다른 팀 서비스 품질에는 영향이 안 가요 — EP02에서 "챗봇이 느려요"라는 민원의 원인을 계층별로 가려낸다고 했었는데, 이 팀별 예산 분리도 같은 맥락이에요.

## 다음 편으로 넘어가기 전에

이 설정 파일, `docker-compose.yml`, 그리고 EP04에서 만든 이미지·모델 파일까지 전부 모아서 다음 편에서 실제 "반입 패키지"로 묶을 거예요.

---

📌 **EP.05 한 줄 요약**
vLLM 앞에 LiteLLM을 세워 팀별 요청량·예산을 나누고, 설정 파일에 텔레메트리를 끄고 민감값은 환경변수로 분리해 반입 심사를 통과할 수 있게 준비했다.

**다음 편**: [EP06. 반입 패키지 제작](EP06-반입-패키지-제작.md)
