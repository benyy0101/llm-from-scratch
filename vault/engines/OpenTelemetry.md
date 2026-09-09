---
tags: [tool, observability, standard]
aliases: ["OTel"]
---

# OpenTelemetry

메트릭·로그·트레이스 계측(instrumentation)을 벤더 중립적으로 통일하기 위한 CNCF 오픈소스 표준입니다. 이게 없던 시절엔 "메트릭은 A사 SDK, 트레이스는 B사 SDK"처럼 각 신호마다 서로 다른 계측 코드를 심어야 했는데, OpenTelemetry는 계측 코드를 한 번만 심어두면 나중에 백엔드(어디로 데이터를 보낼지)만 바꿀 수 있게 해줍니다.

핵심 컴포넌트는 **OpenTelemetry Collector**입니다. 각 서비스는 이 Collector로만 계측 데이터를 보내고, Collector가 이를 받아 [[Prometheus]]·[[Loki]]·Jaeger·Grafana Tempo 등 원하는 백엔드로 라우팅합니다. 서비스 코드 입장에서는 "OTel Collector 주소 하나"만 알면 되고, 백엔드 구성이 바뀌어도 서비스를 다시 배포할 필요가 없습니다.

## 저희 스택에서의 위치

[[LiteLLM]]은 OTel 계측을 기본 지원하고, [[vLLM]]도 최근 버전에서 OTel 트레이싱을 켤 수 있는 옵션을 제공합니다. 이 둘을 계측해두면 [[분산 트레이싱]]에서 다룬 "게이트웨이에서 서빙까지 요청이 어디서 시간을 먹었는지"를 자동으로 볼 수 있게 됩니다 — 지금 EP13 수준의 수작업 로그 뒤지기를 대체하는 다음 단계입니다.

## 폐쇄망에서의 반입 포인트

Collector 자체는 순수 Go 바이너리라 반입이 단순하지만, 각 언어(Python 등)의 **자동계측(auto-instrumentation) 라이브러리**는 여러 개의 pip 패키지가 딸려오는 경우가 많아 이 묶음 전체를 [[표준 아키텍처|반입 파이프라인]]에 태워야 합니다. LiteLLM처럼 이미 OTel을 내장 지원하는 도구는 이 부담이 적지만, vLLM처럼 옵션으로 켜는 경우엔 관련 계측 패키지가 추가로 필요한지 확인이 필요합니다.

## 관련
- [[분산 트레이싱]] — OpenTelemetry가 표준화하는 대표적인 활용처
- [[관측성]] — 세 기둥 계측을 한 표준으로 통일하는 프레임워크
- [[Prometheus]] · [[Loki]] — OTel Collector가 실제로 데이터를 넘기는 백엔드
- [[LiteLLM]] · [[vLLM]] — OTel 계측을 심어야 할 실제 계층
