# EP04. LiteLLM Router 다중 노드 라우팅 설계

> 시리즈: [폐쇄망 멀티에이전트 구축기](README.md) · 이전: [EP03. DGX 노드 선정과 독립형 vs 클러스터형 판단](EP03-DGX-노드-선정과-독립형-vs-클러스터형-판단.md) · 다음: [EP05. Hermes Agent 반입 패키지 제작](EP05-Hermes-Agent-반입-패키지-제작.md)

EP01에서 못박아뒀던 그 약속 — "Hermes Agent 자체 라우팅(`runtime_provider.py`)과 LiteLLM 게이트웨이가 같이 켜져 있으면 라우팅 판단이 두 군데서 따로 논다"는 문제, 이번 편에서 실제로 풉니다. [1편(구축기 v1.0)](../구축기-v1.0/README.md)의 LiteLLM은 노드가 1대(V100 3장을 한 서버에)였어서 라우팅이랄 게 사실상 없었는데, 이번엔 진짜 "여러 엔드포인트 중 하나를 고르는" 문제가 생겼어요.

## 🗂️ model_list부터 다시 짰어요

[1편 EP05](../구축기-v1.0/EP05-LiteLLM-설정.md)에서는 모델 하나당 엔드포인트 하나였는데, 이번엔 **같은 논리적 모델 이름 뒤에 여러 배포(deployment)를 등록**하는 구조로 바꿨습니다.

```yaml
model_list:
  - model_name: infer-primary        # 봇/사용자가 부르는 이름
    litellm_params:
      model: openai/Qwen2.5-72B-Instruct
      api_base: http://node1-infer:8000/v1
      max_parallel_requests: 24       # EP03에서 잡은 배치 여유 기준
  - model_name: infer-primary          # 같은 이름으로 예비 노드도 등록
    litellm_params:
      model: openai/Qwen2.5-72B-Instruct
      api_base: http://node4-standby:8000/v1
      max_parallel_requests: 24

  - model_name: code-assist
    litellm_params:
      model: openai/Qwen2.5-Coder-32B
      api_base: http://node2-code:8000/v1
      max_parallel_requests: 32

  - model_name: embed-doc
    litellm_params:
      model: openai/bge-m3
      api_base: http://node3-embed:8000/v1
      max_parallel_requests: 40

router_settings:
  routing_strategy: least-busy        # 노드별 진행 중 요청 수 기준 분산
  num_retries: 2
  timeout: 30
  fallbacks: [{"infer-primary": ["infer-primary"]}]   # 같은 이름 안에서 재시도 시 다른 배포로
```

핵심은 `model_name`을 봇·사용자 관점의 "논리적 이름"으로 고정하고, 그 뒤에 실제 노드 엔드포인트(`api_base`)를 여러 개 매달아둔 거예요. 노드4(예비)도 평시엔 `infer-primary` 뒤에 미리 등록해두되 `max_parallel_requests`를 낮게 잡아서, 노드1이 살아있는 한 거의 선택되지 않다가 노드1이 죽으면 자연스럽게 트래픽을 받게 했습니다.

## 🚦 least-busy를 고른 이유

라우팅 전략 후보가 몇 개 더 있었는데(`round-robin`, `latency-based`, `least-busy`), 저희 시나리오엔 `least-busy`가 맞았어요.

| 전략             | 특징                         | 우리 상황에 안 맞는 이유                                                            |     |
| -------------- | -------------------------- | ------------------------------------------------------------------------- | --- |
| round-robin    | 순서대로 균등 분배                 | 노드1(추론)·노드2(코드)는 애초에 역할이 달라 같은 후보군이 아님. 같은 역할 안에서도 노드4(예비)는 평소 거의 안 받아야 함 |     |
| latency-based  | 최근 응답속도가 빠른 곳 우선           | 콜드 스타트 직후 노드가 유리하게 잡히는 등 변동성이 큼                                           |     |
| **least-busy** | **현재 진행 중인 요청 수가 적은 곳 우선** | **노드4를 낮은 상한으로 등록해두면 "바쁘지 않을 때만 자연 유입"이 그대로 구현됨**                         |     |

## 🔒 Hermes Agent 쪽 라우팅은 완전히 죽였어요

`runtime_provider.py`가 여러 프로바이더를 알아서 고르는 기능은 그대로 두면 LiteLLM과 판단이 어긋날 수 있다고 EP01에서 확인했었죠. 그래서 Hermes Agent 설정에서 프로바이더를 딱 하나만 등록했습니다.

```yaml
# hermes agent providers.yaml (발췌)
providers:
  - name: gateway-only
    api_mode: openai_compatible
    base_url: http://litellm-gateway:4000/v1
    api_key: ${LITELLM_MASTER_KEY}
    models: [infer-primary, embed-doc]   # code-assist는 등록하지 않음(아래 참고)
```

이렇게 해두면 Hermes 입장에서는 "프로바이더가 하나뿐"이라 `runtime_provider.py`의 다중 프로바이더 선택 로직이 사실상 작동할 일이 없어요. 봇이 `infer-primary`를 부르면 그 요청은 무조건 LiteLLM 게이트웨이로 가고, "어느 물리 노드가 받을지"는 전적으로 LiteLLM `least-busy` 판단에 맡깁니다.

`code-assist`(코드용 모델)는 Hermes 프로바이더 목록에 아예 안 올렸어요. 개발팀은 Hermes Bot Mode를 거치지 않고 Cline·Continue.dev가 게이트웨이에 직접 붙는 구조라([EP01](EP01-왜-멀티에이전트-오케스트레이션인가.md)), Hermes 쪽이 이 모델을 알 필요가 없었습니다 — 이렇게 두니 Hermes 반입 심사([EP05](EP05-Hermes-Agent-반입-패키지-제작.md)) 범위에도 코드용 모델 관련 항목이 하나 줄었어요.

## 📋 이번 편에서 확정한 것

| 항목 | 결정 |
|---|---|
| model_list 구조 | 논리적 `model_name` 하나에 여러 노드 배포 등록 (노드4는 낮은 상한으로 예비 등록) |
| 라우팅 전략 | `least-busy` |
| 노드별 동시성 상한 | 노드1: 24, 노드2: 32, 노드3: 40 (EP09에서 실측 후 재조정 예정) |
| Hermes 프로바이더 설정 | LiteLLM 게이트웨이 하나만 등록, 자체 라우팅 판단 사실상 비활성화 |
| 페일오버 방식 | 노드4를 동일 `model_name` 뒤에 낮은 상한으로 상시 등록해두는 방식(별도 스크립트 전환 불필요) |

다음 편에서는 이 4개 노드에 실제로 Hermes Agent와 vLLM을 반입하기 전에, 반입 패키지를 어떻게 준비했는지 다룹니다 — 노드가 1대일 때와 4대일 때 반입 절차가 뭐가 달라지는지가 포인트예요.

---

📌 **EP.04 한 줄 요약**
LiteLLM Router에 논리적 모델 이름 하나당 여러 노드 배포를 `least-busy`로 등록해 노드4를 자연스러운 예비로 두고, Hermes Agent는 프로바이더를 게이트웨이 하나로 고정해 자체 라우팅 판단을 사실상 비활성화했다.

**다음 편**: [EP05. Hermes Agent 반입 패키지 제작](EP05-Hermes-Agent-반입-패키지-제작.md)
