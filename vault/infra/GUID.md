---
tags: [concept, infra]
aliases: ["Globally Unique Identifier", "UUID"]
---

# GUID

Globally Unique Identifier — 128비트짜리 숫자를 `40E0AC32-46A5-438A-A0B2-2B479E8F2E90`처럼 하이픈으로 묶은 16진수 문자열로 표현한 식별자입니다. 128비트라는 공간이 워낙 넓어서, 서로 전혀 통신하지 않는 여러 컴퓨터가 각자 알고리즘으로 GUID를 만들어도 실질적으로 충돌하지 않습니다 — 중앙 서버에 "이 번호 이미 쓰는 사람 있어요?"라고 물어볼 필요가 없다는 뜻입니다.

Windows API 곳곳에서 "이건 어떤 종류의 객체인지"를 구분하는 용도로 GUID를 씁니다. 대부분은 프로그램이 실행 중에 무작위로 생성하지만, 일부는 **마이크로소프트가 미리 정해서 고정해둔 값**입니다. 예를 들어 [[VMCreatorId]]에서 [[WSL2]]를 가리키는 GUID(`{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}`)는 누가 생성한 게 아니라, "WSL이 만드는 VM은 전부 이 번호로 식별한다"고 마이크로소프트 문서에 박아둔 상수입니다. 벤더가 미리 등록해둔 MAC 주소 대역과 비슷한 성격입니다.

## 관련
- [[VMCreatorId]] — WSL을 가리키는 고정 GUID를 실제로 사용하는 곳
- [[New-NetFirewallHyperVRule]] — 이 GUID를 파라미터 값으로 받는 cmdlet

## 실전 사례
- [05. GPU 없이 CPU로 구축기 스택 실습하기](../../docs/05-CPU-실습-환경-구축-트러블슈팅.md) — WSL을 가리키는 고정 GUID를 방화벽 규칙에 실제로 사용한 사례
