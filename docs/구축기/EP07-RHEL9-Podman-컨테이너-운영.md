# EP07. RHEL 9에서 Podman 컨테이너 운영

> 시리즈: [폐쇄망 LLM 구축기](README.md) · 이전: [EP06. 반입 패키지 제작](EP06-반입-패키지-제작.md) · 다음: [EP08. vLLM 배포 (V100 3장)](EP08-vLLM-배포-V100-3장.md)

Phase 3, "폐쇄망 설치 편" 시작이에요. EP06에서 승인받은 패키지가 드디어 폐쇄망 안으로 넘어왔어요. 이번 편에서는 그걸 실제로 서버에 풀고, [[Podman]]으로 컨테이너를 띄우는 기본기를 다져볼게요.

## 🎯 왜 Docker가 아니라 Podman이냐면

저희 RHEL 9 서버엔 애초에 Docker가 안 깔려 있고, Podman이 기본 컨테이너 런타임으로 들어가 있었어요. 근데 그거랑 별개로, 이번 프로젝트는 정보보호팀 심의를 거쳐야 하는데 "루트 권한으로 상시 떠 있는 데몬이 있다"는 것 자체가 심의에서 걸고넘어지기 좋은 포인트더라구요. Podman은 데몬이 없고 rootless로 동작해서, 이 질문 자체를 피해갈 수 있었어요. 이미지 포맷은 Docker랑 똑같은 OCI라서 EP04에서 `docker build`로 만든 이미지도 그대로 씁니다.

## 📦 반입된 이미지부터 로드했어요

```bash
cd /opt/airgap-package-2026Q3
for img in images/*.tar; do
  podman load -i "$img"
done

podman images
```

## 🔒 rootless로 기본 골격 잡기

일반 사용자 계정(`llmsvc`)으로 컨테이너를 돌리게 했어요. 이 계정한테 subuid/subgid 범위를 할당해야 rootless 네임스페이스가 제대로 동작합니다.

```bash
sudo useradd -m llmsvc
sudo usermod --add-subuids 100000-165535 --add-subgids 100000-165535 llmsvc
loginctl enable-linger llmsvc   # 로그아웃 후에도 컨테이너가 계속 떠있게
```

네트워크는 서비스 간 통신용으로 하나 따로 만들었어요.

```bash
podman network create airgap-net
```

## ⚙️ systemd로 관리하기 — Quadlet

컨테이너를 그냥 `podman run`으로 띄워두면 서버 재부팅했을 때 자동으로 안 올라와요. Podman 4.x부터 들어온 **Quadlet**을 써서 systemd가 직접 컨테이너 생명주기를 관리하게 했습니다. `.container` 파일 하나가 systemd 유닛 하나가 되는 방식이에요.

```ini
# ~/.config/containers/systemd/qdrant.container
[Unit]
Description=Qdrant vector DB

[Container]
Image=localhost/qdrant:latest
Network=airgap-net
Volume=/opt/airgap-data/qdrant:/qdrant/storage:Z
PublishPort=6333:6333

[Service]
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
systemctl --user daemon-reload
systemctl --user enable --now qdrant.service
```

Qdrant, LiteLLM, Open WebUI처럼 GPU가 필요 없는 서비스는 전부 이 방식으로 rootless + Quadlet 조합으로 등록했어요. **GPU가 필요한 vLLM 컨테이너만큼은 얘기가 좀 달랐는데**, 그건 다음 편에서 자세히 풀게요 — 스포일러하자면 rootless로 GPU를 물리려다가 진짜 며칠을 썼습니다.

---

📌 **EP.07 한 줄 요약**
GPU가 필요 없는 서비스(Qdrant, LiteLLM, Open WebUI)는 rootless Podman + Quadlet으로 systemd에 등록해 안정적으로 관리했다. GPU가 필요한 vLLM은 별도로 다룬다.

**다음 편**: [EP08. vLLM 배포 (V100 3장)](EP08-vLLM-배포-V100-3장.md)

출처: [Podman GPU Passthrough 가이드 (oneuptime)](https://oneuptime.com/blog/post/2026-03-18-use-gpu-passthrough-podman/view) · [RHEL Podman AI 워크로드 (lucaberton.com)](https://lucaberton.com/blog/containerized-ai-workloads-with-podman-on-rhel/)
