# EP11. OpenWebUI 연동 & 사용자 추적

> 시리즈: [폐쇄망 LLM 구축기](README.md) · 이전: [EP10. LiteLLM 게이트웨이 구성](EP10-LiteLLM-게이트웨이-구성.md) · 다음: EP12. 오프라인 패키지 설치 (예정)

Phase 3 마지막 편이에요. 서빙 계층(vLLM, Ollama, LiteLLM)까지 다 세웠으니, 이제 현업팀이 실제로 마주할 화면인 [[Open WebUI]]를 붙일 차례예요. 개발팀용 [[Continue.dev]]는 훨씬 뒤(EP17, 고급 활용 편)에서 다뤄요 — 이번 편은 현업 챗봇 UI에 집중합니다.

## 🚪 Open WebUI는 vLLM이 아니라 LiteLLM을 봐요

```ini
# ~/.config/containers/systemd/open-webui.container
[Unit]
Description=Open WebUI

[Container]
Image=localhost/open-webui:main
Network=airgap-net
Environment=OPENAI_API_BASE_URL=http://litellm-gateway:4000/v1
Environment=OPENAI_API_KEY=%t/open-webui-litellm-key
Environment=ENABLE_OLLAMA_API=false
Volume=/opt/airgap-data/open-webui:/app/backend/data:Z
PublishPort=3000:8080

[Service]
Restart=always

[Install]
WantedBy=multi-user.target
```

`OPENAI_API_BASE_URL`이 vLLM(`8000`)이 아니라 LiteLLM(`4000`)을 가리키는 게 핵심이에요. [[표준 아키텍처]]에서 채팅 UI가 API 게이트웨이를 거치도록 그렸던 그대로예요 — Open WebUI가 직접 vLLM을 보게 하면 EP05에서 만든 팀별 예산·요청량 제한이 다 무의미해지거든요.

## 🔑 사내 계정 연동에서 한 번 막혔어요

Open WebUI 기본 설정대로면 사용자가 이메일·비밀번호로 직접 계정을 새로 만들어요. 근데 저희는 이미 사내 AD(Active Directory)가 있는데 계정을 또 만들게 하는 건 보안팀이 절대 승인 안 해줄 거였어요. 그래서 사내 LDAP 연동으로 로그인하게 설정했습니다.

```bash
# Open WebUI 환경변수 (LDAP 연동)
ENABLE_LDAP=true
LDAP_SERVER_HOST=ldap.internal.corp
LDAP_SERVER_PORT=636
LDAP_USE_TLS=true
LDAP_SEARCH_BASE=ou=users,dc=corp,dc=internal
LDAP_ATTRIBUTE_FOR_USERNAME=uid
```

여기서 걸린 건 로그인 자체가 아니라 **권한 분리**였어요. 현업팀 계정으로 로그인했는데 개발팀용 모델 옵션까지 다 보이면 안 되잖아요. LDAP 그룹(`cn=product-planning`, `cn=dev-team`)을 Open WebUI의 그룹 기능과 매핑하고, 그 그룹을 다시 EP10에서 발급한 LiteLLM `team_id`(가상 키)와 짝지어서 — 현업팀 로그인 사용자에게는 `qwen2.5-32b`만, 필요하면 개발팀 전용 모델 옵션은 아예 안 보이도록 정리했어요. LDAP 그룹 → WebUI 그룹 → LiteLLM 팀, 이 삼단 매핑을 문서로 표까지 그려서 남겨뒀습니다 (나중에 사람 바뀌면 저 매핑 기억하는 사람이 저밖에 없어질 거라서요).

## 📋 사용자 추적 — 감사 로그는 두 군데서 나와요

금융권이라 "누가 언제 무슨 질문을 했는가"를 남기는 게 옵션이 아니라 필수 요구사항이었어요. 저희는 이걸 한 군데서 다 해결하려 하지 않고 두 로그를 따로 남겨서 대조 가능하게 했습니다.

| 로그 출처 | 남기는 내용 |
|---|---|
| Open WebUI 자체 로그 | 사용자 ID, 대화 시각, 대화 전문(내용까지) |
| LiteLLM 사용량 로그 | 가상 키(팀) 기준 요청 수·토큰 사용량·비용 |

Open WebUI 쪽은 "누가 뭘 물었는지" 원문까지 남고, LiteLLM 쪽은 "어느 팀이 얼마나 썼는지" 집계 위주라 둘의 목적이 달라요. 대화 로그 보관 기간은 개인정보보호팀 협의로 6개월로 정했고, 이 이후엔 자동 삭제되도록 배치를 걸어뒀어요. 로그 자체를 어떻게 안전하게 쌓고 알람까지 거는 얘기는 EP13(모니터링 편)에서 이어집니다.

Phase 3는 여기서 끝이에요. 서빙 계층과 접근 계층까지 전부 붙었고, 다음 Phase 4부터는 이걸 실제로 "운영"하는 이야기 — 오프라인 패키지 설치 마무리, 모니터링, 모델 업데이트 전략을 다룰게요.

---

📌 **EP.11 한 줄 요약**
Open WebUI는 vLLM이 아니라 LiteLLM 게이트웨이를 보게 붙이고, 사내 LDAP 로그인 + 그룹별 모델 접근 제한을 걸었다. 감사 로그는 대화 원문(Open WebUI)과 사용량 집계(LiteLLM)를 따로 남겨 서로 대조 가능하게 했다.

**다음 편**: EP12. 오프라인 패키지 설치 (예정)
