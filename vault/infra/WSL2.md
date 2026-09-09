---
tags: [concept, infra]
aliases: ["Windows Subsystem for Linux 2"]
---

# WSL2

Windows 안에서 리눅스를 실행하는 두 번째 세대 방식입니다. 1세대(WSL1)가 리눅스 시스템 콜을 Windows 커널이 흉내 내는 방식이었다면, WSL2는 실제 리눅스 커널을 [[Hyper-V]] 위의 경량 [[가상머신]]에서 통째로 돌립니다. 그래서 호환성은 훨씬 좋아졌지만, 대신 "같은 컴퓨터인데 사실은 네트워크로 분리된 별도 머신"이라는 성격을 갖게 됐습니다.

## 기본값은 NAT — 호스트에 접근하려면 게이트웨이 IP를 거쳐야 함

WSL2는 기본적으로 자기만의 가상 네트워크 어댑터(`vEthernet (WSL...)`)를 하나 받고, Windows 호스트와는 NAT로 연결됩니다. 즉 WSL 안에서 `localhost`는 WSL 자기 자신만 가리키고, Windows 호스트에 떠 있는 서비스에 접근하려면 **게이트웨이 IP**를 통해야 합니다.

```bash
ip route show default   # via 뒤에 나오는 IP가 Windows 호스트
```

이 구조 때문에 WSL에서 Windows 쪽 서비스에 접근하려면 최소 두 가지가 맞아야 합니다.

1. Windows 쪽 서비스가 `127.0.0.1`이 아니라 `0.0.0.0`([[바인드 주소]] 참고)에 열려 있어야 함
2. [[Hyper-V 방화벽]]이 WSL이 속한 [[가상 스위치 포트]]의 트래픽을 막고 있지 않아야 함 — 막고 있다면 [[New-NetFirewallHyperVRule]]로 예외를 열어야 함

두 조건 중 하나만 맞춰서는 연결이 안 됩니다. 실제로 Ollama의 바인드 주소만 고쳤을 때는 여전히 실패했고, Hyper-V 방화벽 규칙까지 추가해야 했습니다.

## 실습 환경으로서의 WSL2

이 vault의 실습 계획에서는 GPU 없이 [[RHEL 9]] 계열([[Podman]] 실습용 AlmaLinux 9)을 WSL2 위에 올려 씁니다. RHEL 9와 AlmaLinux 9는 systemd를 그대로 쓰기 때문에, EP07에서 다룬 rootless [[Podman]] + Quadlet을 물리 서버 없이도 그대로 재현할 수 있습니다.

## 관련
- [[Hyper-V]] · [[가상머신]] — WSL2가 실제로는 무엇인지에 대한 배경
- [[바인드 주소]] — Windows 쪽 서비스가 WSL에서 보이려면 만족해야 하는 조건 1
- [[Hyper-V 방화벽]] · [[New-NetFirewallHyperVRule]] — 조건 2
- [[RHEL 9]] · [[Podman]] — WSL2 위에서 재현하는 실습 스택
- [[Ollama]] — 이 구조 때문에 실제로 연결 문제를 겪은 대상

## 실전 사례
- [05. GPU 없이 CPU로 구축기 스택 실습하기](../../docs/05-CPU-실습-환경-구축-트러블슈팅.md) — AlmaLinux 9 설치부터 Ollama 연결 트러블슈팅까지 전 과정
