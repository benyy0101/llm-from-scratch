---
tags: [tool, container]
---

# Podman

Docker와 같은 컨테이너를 다루지만, 백그라운드 데몬이 없고(daemonless) 루트 권한 없이도 동작하는(rootless) 컨테이너 엔진입니다. RHEL 9부터 기본 컨테이너 런타임으로 채택돼 있어서, 금융·공공기관처럼 규제가 엄격한 [[폐쇄망]] 환경에서 "왜 루트 권한이 필요한 데몬을 돌려야 하냐"는 보안 심사를 피하기 위해 Docker 대신 선택되는 경우가 많습니다.

이미지 포맷은 Docker와 같은 OCI 표준을 쓰기 때문에, `docker save`/`docker load`로 익힌 이관 절차([[6단계 폐쇄망 이관]])를 `podman save`/`podman load`로 거의 그대로 옮길 수 있습니다.

rootless가 항상 매끄러운 건 아닙니다 — 특히 GPU를 컨테이너에 물려야 할 때는 CDI(Container Device Interface) 방식이 rootless보다 rootful(root 권한)에서 훨씬 안정적으로 동작하는 경우가 있어, GPU가 필요한 컨테이너만 예외적으로 rootful로 운영하고 나머지는 rootless를 유지하는 절충이 실무에서 흔합니다.

## 관련
- [[6단계 폐쇄망 이관]] — Docker 대신 Podman으로도 그대로 되는 이관 절차
- [[폐쇄망]] — 루트리스 구조가 특히 매력적인 이유

## 실전 사례
- [EP07. RHEL 9에서 Podman 컨테이너 운영 (구축기)](../../docs/구축기/EP07-RHEL9-Podman-컨테이너-운영.md) — rootless + Quadlet으로 서비스를 등록한 과정
- [EP08. vLLM 배포 — V100 3장 (구축기)](../../docs/구축기/EP08-vLLM-배포-V100-3장.md) — rootless GPU 패스스루가 불안정해 vLLM 컨테이너만 rootful로 예외 운영한 실제 판단
