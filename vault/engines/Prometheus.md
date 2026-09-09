---
tags: [tool, observability]
---

# Prometheus

메트릭을 수집·저장하는 오픈소스 시계열 데이터베이스입니다. 각 서비스가 노출하는 `/metrics` HTTP 엔드포인트를 주기적으로 직접 방문(pull)해서 값을 긁어가는 방식이라, 서비스 쪽은 "요청이 오면 지금 값을 뱉는" 수동적인 역할만 하면 됩니다. [[vLLM]]의 `vllm:gpu_cache_usage_perc`·`vllm:time_to_first_token_seconds`, [[LiteLLM]]의 팀별 요청 수·실패율이 전부 이 방식으로 노출되고, EP13의 `prometheus.yml`처럼 어느 주소를 긁을지만 설정하면 됩니다.

쿼리 언어(PromQL)로 저장된 시계열을 조합·집계할 수 있고, 이 결과를 [[Grafana]]가 시각화하거나 [[Alertmanager]]가 알람 조건으로 평가합니다.

## 폐쇄망에서의 특징

단일 Go 바이너리라 외부 의존성 없이 셀프호스팅이 쉬워, [[표준 아키텍처]]의 모니터링 계층으로 가장 먼저 채택되는 도구입니다. 다만 기본 구성은 단일 노드 로컬 디스크 저장이고 보존 기간도 기본 15일 수준이라, 이건 [[감사 로그와 WORM|감사 요구]]가 아니라 어디까지나 최근 운영 상태를 보는 용도입니다 — 장기 보존이 필요하면 Thanos·Mimir 같은 추가 레이어가 있지만, GPU 서버 몇 대 규모의 단일 사이트 폐쇄망이라면 대부분 불필요한 오버엔지니어링입니다.

## 관련
- [[관측성]] — Prometheus가 채우는 메트릭 축
- [[Grafana]] — Prometheus가 모은 데이터를 시각화하는 짝
- [[Alertmanager]] — Prometheus 룰이 평가한 알람 조건을 실제로 발송하는 짝
- [[DCGM Exporter]] — vLLM 자체 메트릭이 못 보는 GPU 하드웨어 신호를 Prometheus 포맷으로 추가 노출
- [[vLLM]] · [[LiteLLM]] — `/metrics` 엔드포인트를 노출하는 실제 대상

## 실전 사례
- [EP13. 모니터링 & 트러블슈팅](../../docs/구축기/EP13-모니터링-트러블슈팅.md) — vLLM·LiteLLM을 스크레이프해 계층별 대시보드를 구성하고, KV 캐시 점유율 이상으로 실제 장애를 감지한 사례
