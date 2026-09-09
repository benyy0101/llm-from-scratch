---
tags: [tool, observability]
---

# Alertmanager

[[Prometheus]] 생태계에서 알람의 "라우팅"을 전담하는 컴포넌트입니다. Prometheus 자체는 룰(예: `gpu_cache_usage_perc > 90%`가 5분 이상 지속)이 참인지 평가만 하고, 그 알람을 **누구에게 어떤 채널로** 보낼지, 같은 알람이 반복 발생할 때 중복을 어떻게 묶을지(grouping), 점검 중에는 어떻게 잠시 꺼둘지(silence)는 Alertmanager가 맡습니다. EP13에서 계층별로 다른 사람에게 알람이 가게 만든 게("서빙 계층은 인프라팀, 게이트웨이는 담당팀, 접근 계층 이상접근 시도는 보안팀도") 실제로는 이 라우팅 트리 설정입니다.

## 폐쇄망에서 리시버가 걸리는 지점

Alertmanager가 표준으로 지원하는 리시버(알람을 실제로 보내는 대상)에는 Slack Webhook, PagerDuty, Opsgenie 같은 외부 SaaS가 많은데, 완전 [[폐쇄망]]에서는 이런 외부 엔드포인트로 나가는 트래픽 자체가 원천 차단됩니다. 대신 쓸 수 있는 건 내부망 SMTP 서버로 보내는 이메일 리시버이거나, 사내에 자체 호스팅된 메신저(Mattermost 등)가 Slack 호환 Webhook API를 제공한다면 그 내부 엔드포인트로 연결하는 방식입니다. EP13의 "인프라팀 슬랙 알람"이라는 표현이 실제로 외부 Slack인지, 사내 메신저의 Slack 호환 API인지에 따라 이게 완전 에어갭인지 제한적 아웃바운드가 허용된 사내망인지가 갈립니다.

## 관련
- [[Prometheus]] — 알람 조건을 평가해 Alertmanager로 넘기는 짝
- [[Grafana]] — 규모가 작으면 Alertmanager 없이 Grafana Alerting 자체 기능으로 대체 가능
- [[관측성]] — 이상 신호를 사람에게 실제로 전달하는 마지막 단계
- [[폐쇄망]] — 리시버 선택지를 제한하는 근본 조건
