# EP07. LiteLLM Router least-busy 라우팅 구성

> 시리즈: [폐쇄망 멀티에이전트 구축기](README.md) · 이전: [EP06. 4노드 vLLM 배포 (역할별 모델 분리)](EP06-4노드-vLLM-배포-역할별-모델-분리.md) · 다음: [EP08. Hermes Agent 설치 & Bot Mode 구성](EP08-Hermes-Agent-설치-Bot-Mode-구성.md)

[EP04](EP04-LiteLLM-Router-다중노드-라우팅-설계.md)에서 설계만 해뒀던 게이트웨이, 이번 편에서 실제로 띄우고 검증합니다. 노드마다 vLLM이 살아있는 건 [지난 편](EP06-4노드-vLLM-배포-역할별-모델-분리.md)에서 확인했으니, 이제 그 앞단에 LiteLLM Router를 세우고 "정말 설계대로 동작하는가"를 실측할 차례예요.

## 🚀 게이트웨이 컨테이너 기동

```bash
podman run -d --name litellm-gateway \
  --network airgap-net \
  -v /opt/litellm/config.yaml:/app/config.yaml:Z \
  -e LITELLM_MASTER_KEY=${LITELLM_MASTER_KEY} \
  -e DATABASE_URL=postgresql://litellm:***@postgres:5432/litellm \
  -p 4000:4000 \
  localhost/litellm:latest \
  --config /app/config.yaml --port 4000
```

[1편 EP10](../구축기-v1.0/EP10-LiteLLM-게이트웨이-구성.md)에서 이미 Postgres로 가상 키를 영속화하는 구조를 만들어뒀어서, 이번엔 그 DB를 그대로 재사용하고 `config.yaml`만 [EP04](EP04-LiteLLM-Router-다중노드-라우팅-설계.md)에서 설계한 다중 노드 버전으로 교체했습니다.

## 🩺 헬스체크가 진짜 핵심이었어요

`least-busy` 전략이 제대로 동작하려면 게이트웨이가 "노드4가 지금 죽어있다"는 걸 알아야 해요. 안 그러면 죽은 노드로 요청을 계속 보내다가 타임아웃만 쌓입니다.

```yaml
router_settings:
  routing_strategy: least-busy
  enable_pre_call_checks: true
  background_health_checks: true
  health_check_interval: 30      # 30초마다 각 배포에 ping
```

평시(노드4 컨테이너가 꺼져 있는 상태)에 헬스체크 로그를 확인해봤어요.

```
[health_check] infer-primary/node1-infer: healthy
[health_check] infer-primary/node4-standby: unhealthy (connection refused)
[router] infer-primary 요청 100% → node1-infer로만 라우팅
```

설계대로 노드4가 꺼져 있으면 자동으로 라우팅 후보에서 빠지고, 요청은 전부 노드1로만 갑니다. 여기까지는 예상대로였는데, 진짜 확인하고 싶었던 건 **노드4를 살렸을 때 자연스럽게 트래픽을 나눠 받는지**였어요.

## 🧪 노드4를 임시로 띄워 라우팅 분산을 테스트했어요

```bash
# 노드4에 임시로 추론용 컨테이너 기동
podman start vllm-infer-standby

# 부하 테스트 스크립트로 infer-primary에 40개 동시 요청 발사
./scripts/load-test.sh --model infer-primary --concurrency 40
```

| 시점 | node1-infer 진행 중 요청 | node4-standby 진행 중 요청 |
|---|---|---|
| 요청 시작 직후 | 24(상한 도달) | 0 |
| 3초 후 | 24 | 16 |
| 완료 시점 | - | - |

노드1이 상한(`max_parallel_requests: 24`)에 닿자마자 나머지 요청이 노드4로 자연스럽게 흘러가는 걸 확인했어요. [EP04](EP04-LiteLLM-Router-다중노드-라우팅-설계.md)에서 "노드4를 낮은 상한이 아니라 같은 상한으로 등록해뒀다가, 평시엔 컨테이너 자체를 꺼둬서 헬스체크로 자연스럽게 빠지게 하자"고 방향을 살짝 바꾼 게 이 테스트 이후예요 — 처음엔 상한값 차등으로 우선순위를 주려고 했는데, 컨테이너를 아예 꺼두는 쪽이 "평시엔 예비 자원을 놀리지 않고 완전히 유휴 상태로 둔다"는 목표에 더 맞았습니다.

테스트가 끝난 뒤 노드4는 다시 내렸습니다 — 실제 페일오버 자동화(장애 감지 → 자동 기동)는 아직 수동 개입이 필요한 상태이고, 이 부분은 [EP11](EP11-모니터링-노드-페일오버.md)에서 마무리합니다.

## 📋 이번 편에서 확정한 것

| 항목 | 결정 |
|---|---|
| 헬스체크 | 30초 간격 `background_health_checks`, 죽은 배포는 라우팅 후보에서 자동 제외 |
| 노드4 운영 방식 | 상한 차등이 아니라 평시 컨테이너 정지로 "완전 유휴" 구현 |
| 검증 결과 | 노드1 포화 시 노드4로 자연 분산되는 것을 실측으로 확인 |
| 남은 과제 | 장애 감지 → 노드4 자동 기동까지는 아직 수동, EP11에서 자동화 |

다음 편에서는 이 게이트웨이 위에 실제로 Hermes Agent를 설치하고, [EP01](EP01-왜-멀티에이전트-오케스트레이션인가.md)에서 정했던 비즈니스 봇 3개(리서처·법무검토·품질분석)를 Bot Mode로 실제로 구성합니다. 개발팀(코더 역할)은 이 게이트웨이에 Hermes 없이 직접 붙는데, 그 이유도 다음 편에서 다룹니다.

---

📌 **EP.07 한 줄 요약**
LiteLLM Router에 30초 간격 헬스체크를 걸어 죽은 노드를 자동으로 라우팅 후보에서 빼고, 노드4는 상한 차등 대신 평시 컨테이너 정지로 완전한 유휴 상태를 유지하며, 노드1 포화 시 자연스럽게 트래픽이 넘어가는 것을 실측으로 확인했다.

**다음 편**: [EP08. Hermes Agent 설치 & Bot Mode 구성](EP08-Hermes-Agent-설치-Bot-Mode-구성.md)
