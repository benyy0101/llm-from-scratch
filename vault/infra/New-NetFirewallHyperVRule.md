---
tags: [concept, security]
---

# New-NetFirewallHyperVRule

[[Hyper-V 방화벽]] 계층에 규칙을 추가하는 PowerShell cmdlet입니다. 이름이 비슷한 `New-NetFirewallRule`(일반 Windows 방화벽용)과는 완전히 다른 API로, 물리 어댑터가 아니라 [[가상 스위치 포트]]에 규칙을 건다는 점이 다릅니다.

가장 큰 차이는 대상을 지정하는 방식입니다. 일반 방화벽 규칙은 IP·포트·프로그램 경로로 대상을 지정하지만, 이 cmdlet은 그 위에 **`-VMCreatorId`라는 파라미터로 "어떤 종류의 VM이 만든 트래픽이냐"**를 추가로 지정할 수 있습니다. 이 값은 [[GUID]] 형식이며, [[WSL2]]처럼 특정 컴포넌트가 만드는 VM에는 마이크로소프트가 고정된 값을 미리 배정해뒀습니다 — 자세한 값과 이유는 [[VMCreatorId]] 참고.

실제 사용 예시([[Ollama]]를 Windows에서 [[WSL2]]로 열어주기 위한 규칙):
```powershell
New-NetFirewallHyperVRule -Name "Allow-WSL-Ollama" `
  -DisplayName "Allow WSL to reach Ollama (11434)" `
  -Direction Inbound `
  -VMCreatorId "{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}" `
  -Protocol TCP -LocalPorts 11434 -Action Allow
```
방화벽 규칙 추가는 시스템 보안 설정 변경이라, 반드시 **관리자 권한 PowerShell**에서 사용자가 직접 실행해야 하는 작업입니다.

## 관련
- [[Hyper-V 방화벽]] — 이 cmdlet이 다루는 계층
- [[VMCreatorId]] — `-VMCreatorId` 파라미터에 들어가는 값과 그 의미
- [[GUID]] — VMCreatorId 값의 형식
- [[WSL2]] — 이 cmdlet을 쓰게 되는 가장 흔한 실무 상황

## 실전 사례
- [05. GPU 없이 CPU로 구축기 스택 실습하기](../../docs/05-CPU-실습-환경-구축-트러블슈팅.md) — Ollama를 WSL2에서 접근 가능하게 만들기 위해 실제로 이 규칙을 적용한 과정
