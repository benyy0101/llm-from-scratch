# GPU 없이 CPU로 구축기 스택 실습하기 — 진행 기록

> 이 문서는 [구축기](구축기/README.md) 시리즈("A저축은행" 가상 시나리오, V100 3장 기준)와 [03-트러블슈팅-딥리서치](03-트러블슈팅-딥리서치.md)(외부 사례 큐레이션)와 성격이 다릅니다. **실제로 GPU 없는 Windows 11 PC 한 대에서, 이 스택을 직접 손으로 재현하며 겪은 1차 기록**입니다.

## 왜 이렇게 하나

구축기 스택(`Podman · vLLM · Ollama · LiteLLM · Qdrant · Open WebUI · Continue.dev`)에서 GPU가 실제로 필수인 건 vLLM(EP08)뿐입니다. 나머지는 CPU로도 파이프라인 구조와 운영 절차(컨테이너 관리, 게이트웨이 설정, RAG 흐름)를 그대로 익힐 수 있어서, vLLM 자리를 Ollama(소형 양자화 모델)로 치환해 실습 중입니다.

| 구축기 원본 | 이 실습에서의 대체 |
|---|---|
| RHEL 9 서버 | WSL2 위 AlmaLinux 9 (RHEL 9 계열, 무료) |
| vLLM + V100 3장 | Ollama (`qwen2.5:7b`, CPU) — Windows 호스트에서 실행 |
| Ollama (임베딩 전용, EP09) | Ollama (`nomic-embed-text`) — LLM과 같은 프로세스에서 겸용 |
| Podman rootless + Quadlet (EP07) | 동일 — AlmaLinux 9 안에서 진짜로 재현 |
| LiteLLM, Qdrant, Open WebUI | 동일 (컨테이너로 실행, GPU 불필요) |

## 진행 상태

- [x] WSL2에 AlmaLinux 9 설치, systemd PID1 확인
- [x] `dnf install podman` (5.8.2)
- [x] `llmsvc` 서비스 계정 생성 + subuid/subgid 자동 할당 + `loginctl enable-linger`
- [x] Windows Ollama에 `qwen2.5:7b`, `nomic-embed-text` 모델 이미 존재 확인
- [ ] WSL(AlmaLinux 9) → Windows Ollama 네트워크 연결 (Hyper-V 방화벽 규칙 적용 대기 중)
- [ ] `airgap-net` 네트워크 생성, Qdrant/LiteLLM/Open WebUI Quadlet 유닛 작성

---

## 삽질 1 — Ollama가 Windows 호스트에서 `127.0.0.1`에만 바인딩

**문제 상황**: WSL(AlmaLinux 9)에서 `curl http://<host-ip>:11434/api/version`이 전부 실패.

**원인**: `Get-NetTCPConnection -LocalPort 11434`로 확인해보니 Ollama가 `127.0.0.1`에만 리스닝 중이었음. 루프백 주소는 같은 머신 안에서만 접근 가능해서, WSL이라는 별도 네트워크 네임스페이스에서는 애초에 도달 불가능. (구축기 [EP09](구축기/EP09-Ollama-폐쇄망-설치.md)에서 다룬 `OLLAMA_HOST=0.0.0.0:11434` 설정이 정확히 이 문제를 막기 위한 것이었음 — 폐쇄망 서버에서도 다른 컨테이너가 Ollama에 접근하려면 같은 조치가 필요.)

**해결**:
```powershell
[Environment]::SetEnvironmentVariable("OLLAMA_HOST", "0.0.0.0:11434", "User")
# 기존 ollama 프로세스 종료 후 재시작 (환경변수 반영 위해 새 프로세스로)
```
재시작 후 `Get-NetTCPConnection -LocalPort 11434`에서 `::`(모든 인터페이스)로 리스닝 확인.

---

## 삽질 2 — 바인딩을 고쳐도 WSL에서 여전히 연결 실패 (Hyper-V 방화벽)

**문제 상황**: `OLLAMA_HOST=0.0.0.0`으로 바꾼 뒤에도 WSL(AlmaLinux 9)에서 `curl http://192.168.112.1:11434/...`가 계속 실패. 그런데 **Windows 자신**이 같은 주소로 curl하면 정상 응답(`{"version":"0.33.3"}`).

**원인**: WSL의 네트워크 어댑터 이름이 `vEthernet (WSL (Hyper-V firewall))`인 데서 힌트를 얻음. 최근 Windows는 WSL2를 경량 Hyper-V VM으로 취급하면서, 일반 Windows Defender 방화벽(`Get-NetFirewallRule`로 관리, 어댑터/프로필 단위)과는 **완전히 별개인 Hyper-V 방화벽 계층**을 WSL 가상 스위치 포트에 추가로 적용함. `ollama.exe` 인바운드 규칙이 Public 프로필에 Any/Any로 이미 허용돼 있었는데도(일반 방화벽 계층), 이 두 번째 계층이 WSL → 호스트 방향 트래픽을 별도로 걸러서 막고 있었음.

**해결 (관리자 PowerShell에서 실행 필요 — 시스템 보안 설정 변경이라 사용자가 직접 실행)**:
```powershell
New-NetFirewallHyperVRule -Name "Allow-WSL-Ollama" `
  -DisplayName "Allow WSL to reach Ollama (11434)" `
  -Direction Inbound `
  -VMCreatorId "{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}" `
  -Protocol TCP -LocalPorts 11434 -Action Allow
```
`VMCreatorId {40E0AC32-46A5-438A-A0B2-2B479E8F2E90}`는 마이크로소프트가 WSL이 생성한 VM을 식별하는 고정 GUID. 즉 "WSL에서 오는 TCP 11434 트래픽은 허용"이라는 뜻.

**상태**: 명령 실행 및 재검증 대기 중.

---

## 참고용 진단 명령 모음

```bash
# WSL 안에서 Windows 호스트 게이트웨이 IP 확인
ip route show default   # via 뒤 IP가 Windows 호스트

# Windows에서 특정 포트 리스닝 상태 확인
Get-NetTCPConnection -LocalPort 11434
```
