# EP08. Hermes Agent 설치 & Bot Mode 구성

> 시리즈: [폐쇄망 멀티에이전트 구축기](README.md) · 이전: [EP07. LiteLLM Router least-busy 라우팅 구성](EP07-LiteLLM-Router-least-busy-라우팅-구성.md) · 다음: [EP09. 서브에이전트 동시성 튜닝 & GPU 부하 분산 검증](EP09-서브에이전트-동시성-튜닝-GPU-부하-분산-검증.md)

Phase 3(폐쇄망 설치) 마지막 편이에요. 게이트웨이까지 살아있으니, 이제 오케스트레이션 계층의 실체 — Hermes Agent를 실제로 설치하고 [EP01](EP01-왜-멀티에이전트-오케스트레이션인가.md)에서 정했던 비즈니스 봇 3개를 Bot Mode로 구성합니다. 개발팀은 이번 편에서 아예 등장하지 않는데, 그 이유부터 짚고 넘어갈게요.

## 🤷 왜 코더는 Hermes 봇으로 안 만들었나

처음 설계할 땐 리서처·법무검토·품질분석·코더, 이렇게 4개 봇을 전부 Hermes Bot Mode로 만들 생각이었어요. 그런데 [EP05](EP05-Hermes-Agent-반입-패키지-제작.md)에서 Hermes Agent 하나 반입하는 데도 공식 릴리스 태그 확인, 정적 분석 스캔, `hermes peer` 비활성화 빌드 요청까지 검토 항목이 꽤 붙는 걸 겪고 나니 생각이 바뀌었어요 — **코드 작업은 이미 Cline·Continue.dev([1편 EP17](../구축기-v1.0/EP17-Continue.dev-코딩-어시스턴트.md), [EP20](../구축기-v1.0/EP20-에이전트-심화-Cline-BFCL.md))로 검증된 방식이 있는데, 굳이 Hermes 봇으로 한 번 더 감쌀 이유가 있나** 싶었습니다.

코드 작업은 애초에 연구소·법무·품질 세 부서와 패턴이 달라요 — 서브에이전트로 쪼갤 필요도, 다른 봇과 그룹 채팅으로 협업할 필요도 없는 "질문 하나에 답변 하나"(또는 파일 편집 하나) 구조거든요. 그래서 개발팀은 **Hermes를 거치지 않고 Cline·Continue.dev가 LiteLLM 게이트웨이(`code-assist`)에 직접 붙는** 기존 1편 방식을 그대로 유지하기로 했습니다. 이렇게 두니 좋은 점이 두 가지였어요.

1. Hermes 반입 심사·정적 분석 대상에 `code-assist` 관련 설정이 아예 안 들어가서 검토 범위가 줄어듦
2. Hermes Agent 자체에 장애가 나도(예: 그룹 채팅 버그, Bot Mode 재기동) 개발팀 코딩 작업은 전혀 영향받지 않음 — 오케스트레이션 계층과 완전히 분리된 장애 도메인

```ini
# ~/.config/containers/systemd/cline-gateway.env (Cline VS Code 확장 설정)
OPENAI_API_BASE_URL=http://litellm-gateway:4000/v1
OPENAI_API_KEY=%t/vk-coder-key
OPENAI_MODEL=code-assist
```

Cline·Continue.dev 둘 다 [1편](../구축기-v1.0/README.md)과 완전히 같은 방식으로 게이트웨이에 붙어서, 이번 편에서 새로 설정할 게 딱히 없었어요 — 그냥 "Hermes를 우회하기로 결정했다"는 판단 자체가 이번 편의 핵심입니다.

## 📦 Hermes Agent 컨테이너 기동

```bash
podman run -d --name hermes-agent \
  --network airgap-net \
  -v /opt/hermes/providers.yaml:/app/config/providers.yaml:Z \
  -v /opt/hermes/bots:/app/config/bots:Z \
  -p 7000:7000 \
  localhost/hermes-agent:latest
```

`providers.yaml`은 [EP05](EP05-Hermes-Agent-반입-패키지-제작.md)에서 미리 만들어둔 그대로 — LiteLLM 게이트웨이 하나만 프로바이더로 등록돼 있는 파일을 그대로 마운트했습니다. 실행 후 로그에서 프로바이더 로딩을 확인했어요.

```
[providers] loaded 1 provider: gateway-only (openai_compatible, http://litellm-gateway:4000/v1)
[providers] runtime_provider resolver: single-provider mode (multi-provider selection disabled)
```

`single-provider mode`라는 로그 문구를 보고 나서야 EP01·EP04에서 설계했던 "라우팅 단일화"가 설정 파일 수준이 아니라 **Hermes 내부 동작 수준에서 실제로 비활성화된다**는 걸 확인할 수 있었어요.

## 🤖 봇 3개, Bot Mode로 프로필 등록

```yaml
# /opt/hermes/bots/researcher.yaml
bot:
  name: researcher
  display_name: "리서처 봇"
  pinned_model: infer-primary       # 항상 추론용 모델만 사용
  system_prompt: |
    당신은 연구소 전용 리서처 봇입니다. 특허·논문 조사와 교차검증을 담당합니다.
  delegate:
    enabled: true
    max_concurrent_subagents: 4     # EP02에서 정한 부서당 상한
```

법무검토 봇·품질분석 봇도 같은 형식으로 `pinned_model: infer-primary`, `max_concurrent_subagents: 4`로 등록했습니다. 봇마다 모델을 `pinned_model`로 고정해두니 좋은 점이 하나 있었어요 — 나중에 "품질분석 봇이 이상하게 느리다"는 문의가 왔을 때, 어느 노드(어느 vLLM 인스턴스)를 봐야 하는지 봇 이름만 보고 바로 알 수 있더라구요.

## 🧪 봇 하나씩 연결 테스트

```bash
curl http://localhost:7000/bots/researcher/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "최근 등록된 특허 3건 중 우리 특허와 겹치는 부분이 있는지 확인해줘"}'
```

리서처 봇이 요청을 받아 LiteLLM 게이트웨이(`infer-primary`)로 넘기고, 노드1에서 응답이 오는 것까지 확인했어요. 서브에이전트 위임(`delegate_tool.py`)이 실제로 여러 개를 동시에 띄우는지는 이번 편에서는 연결 확인 정도만 하고, 실제 부하 상황에서의 동시성 튜닝은 다음 편에서 제대로 다룹니다 — 지금은 "봇 하나가 게이트웨이를 거쳐 올바른 모델에 도달하는가"만 확인된 상태예요.

## 🔐 봇별 접근 통제도 같이 걸었어요

봇 3개가 전부 같은 LiteLLM 마스터 키를 쓰면, 나중에 "누가 얼마나 썼는지" 구분이 안 돼요. 그래서 [1편 EP05](../구축기-v1.0/EP05-LiteLLM-설정.md)에서 썼던 가상 키(virtual key) 방식을 봇 단위로 확장했습니다. 개발팀은 Hermes 봇은 아니지만, Cline·Continue.dev가 게이트웨이에 직접 붙는 것도 결국 같은 가상 키 메커니즘이라 표에 같이 넣었습니다.

| 호출 주체 | LiteLLM 가상 키 | 허용 모델 | Hermes 경유 여부 |
|---|---|---|---|
| 리서처 봇 | `vk-researcher-***` | infer-primary, embed-doc | O |
| 법무검토 봇 | `vk-legal-***` | infer-primary | O |
| 품질분석 봇 | `vk-quality-***` | infer-primary | O |
| Cline·Continue.dev(개발팀) | `vk-coder-***` | code-assist | **X — 게이트웨이 직결** |

가상 키를 주체마다 따로 발급해두니, `embed-doc`(임베딩)을 개발팀 쪽이 실수로 호출하는 것도 자연스럽게 막혔어요. 이 가상 키별 사용량은 EP10(봇별 역할 분리 운영)에서 실제 대시보드로 확인합니다.

## 📋 이번 편에서 확정한 것

| 항목 | 결정 |
|---|---|
| Hermes 프로바이더 | 게이트웨이 단일 등록, `single-provider mode` 로그로 실동작 확인 |
| 봇 3개 | 리서처·법무검토·품질분석, 전부 서브에이전트 최대 4개 |
| 개발팀 경로 | Hermes 미경유 — Cline·Continue.dev가 게이트웨이(`code-assist`)에 직접 연결 |
| 모델 고정 | 봇마다 `pinned_model`로 1개 모델만 사용 |
| 접근 통제 | 호출 주체별(봇 3개 + 개발팀) LiteLLM 가상 키 발급, 허용 모델 범위 제한 |

다음 편에서는 서브에이전트를 실제로 동시에 여러 개 띄워보면서, [EP02](EP02-전체-아키텍처-설계.md)에서 추정했던 "GPU 기준 최대 100건 동시 요청"이 진짜 감당 가능한 수준인지 검증합니다.

---

📌 **EP.08 한 줄 요약**
Hermes Agent 프로바이더를 게이트웨이 하나로 고정하고 비즈니스 봇 3개(리서처·법무검토·품질분석)를 Bot Mode로 등록했으며, 개발팀은 반입 심사 부담과 장애 도메인 분리를 위해 Hermes를 거치지 않고 Cline·Continue.dev가 게이트웨이에 직접 연결하도록 했다.

**다음 편**: [EP09. 서브에이전트 동시성 튜닝 & GPU 부하 분산 검증](EP09-서브에이전트-동시성-튜닝-GPU-부하-분산-검증.md)
