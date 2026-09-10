# EP10. LiteLLM 게이트웨이 구성

> 시리즈: [폐쇄망 LLM 구축기](README.md) · 이전: [EP09. Ollama 폐쇄망 설치](EP09-Ollama-폐쇄망-설치.md) · 다음: [EP11. OpenWebUI 연동 & 사용자 추적](EP11-OpenWebUI-연동-사용자-추적.md)

EP05에서 설정 파일까지 다 만들어놨으니, 이번 편은 그걸 실제로 폐쇄망 안에서 띄우는 얘기예요. vLLM([[Tensor Parallelism|EP08]])이랑 Ollama([[임베딩과 벡터DB|EP09]])가 이제 둘 다 살아있으니, 이 둘을 하나로 묶는 게이트웨이만 세우면 서빙 계층이 완성돼요.

## 🐘 먼저 Postgres부터 — 가상 키를 어딘가엔 저장해야 하니까

LiteLLM이 팀별 가상 키·예산 정보를 기억하려면 DB가 필요해요. 재부팅할 때마다 키가 초기화되면 안 되니까요. 이미지 자체는 EP04에서 미리 반입해둔 `postgres:16`을 그대로 로드해서 씁니다 — 이 편에서 갑자기 새로 반입 신청을 넣지 않아도 되도록요.

```ini
# ~/.config/containers/systemd/litellm-db.container
[Unit]
Description=LiteLLM postgres

[Container]
Image=localhost/postgres:16
Network=airgap-net
Environment=POSTGRES_PASSWORD=%t/litellm-db-password
Volume=/opt/airgap-data/litellm-db:/var/lib/postgresql/data:Z

[Service]
Restart=always

[Install]
WantedBy=multi-user.target
```

## 🚪 LiteLLM 컨테이너 기동

```ini
# ~/.config/containers/systemd/litellm.container
[Unit]
Description=LiteLLM gateway
After=litellm-db.service

[Container]
Image=localhost/litellm:main-stable
Network=airgap-net
Volume=/opt/configs/litellm-config.yaml:/app/config.yaml:Z
Environment=LITELLM_MASTER_KEY=%t/litellm-master-key
Environment=LITELLM_DB_URL=postgresql://litellm:%t/litellm-db-password@litellm-db:5432/litellm
PublishPort=4000:4000
Exec=--config /app/config.yaml

[Service]
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
systemctl --user daemon-reload
systemctl --user enable --now litellm-db.service litellm.service
```

EP07에서 정한 대로 이 둘은 rootless로 문제없이 떴어요. GPU가 필요 없는 서비스는 확실히 속 편합니다.

## ⚠️ 삽질 4 — 게이트웨이를 거치니 스트리밍이 뚝뚝 끊겨요

vLLM에 직접 요청할 때는 토큰이 매끄럽게 스트리밍됐는데, LiteLLM을 거치니까 답변이 뭉텅이로 몇 초씩 끊겨서 나오더라구요. vault에 미리 적어뒀던 [[스트리밍 응답]] 문제가 딱 이거였어요 — 프록시 계층의 기본 버퍼링 설정이 스트리밍 응답을 그대로 흘려보내지 않고 일정량 모았다가 보내는 방식이었던 거죠. LiteLLM 앞단에 nginx 리버스 프록시를 하나 더 세워서 버퍼링을 꺼주니 해결됐어요 — 이 nginx 이미지도 EP04에서 미리 받아둔 걸 로드해서 씁니다.

```nginx
location /litellm/ {
    proxy_pass http://litellm-gateway:4000/;
    proxy_buffering off;
    proxy_read_timeout 300s;
}
```

`proxy_buffering off` 한 줄이 이 삽질의 답이었어요. 첫 토큰 지연이 EP08 벤치마크(1.8초)로 나왔던 게, 게이트웨이를 거치고 나니 체감상 훨씬 늦게 느껴졌던 이유가 바로 이거였습니다.

## ✅ 팀별 가상 키 실제 발급

```bash
curl -X POST http://localhost:4000/key/generate \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -d '{
    "models": ["qwen2.5-32b", "bge-m3-embed"],
    "team_id": "product-planning",
    "max_budget": 50,
    "budget_duration": "30d"
  }'

curl -X POST http://localhost:4000/key/generate \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -d '{
    "models": ["qwen2.5-32b"],
    "team_id": "dev-team",
    "max_budget": 100,
    "budget_duration": "30d"
  }'
```

발급받은 키로 LLM·임베딩 둘 다 게이트웨이를 통해 정상적으로 응답하는지 확인하고 마무리했습니다.

---

📌 **EP.10 한 줄 요약**
Postgres로 가상 키를 영속화한 LiteLLM을 rootless로 띄웠고, 게이트웨이 앞단 nginx의 버퍼링 설정을 꺼서 스트리밍 끊김 문제를 해결한 뒤 팀별 가상 키를 실제로 발급했다.

**다음 편**: [EP11. OpenWebUI 연동 & 사용자 추적](EP11-OpenWebUI-연동-사용자-추적.md)
