---
tags: [tool, container]
---

# Podman

Docker와 같은 컨테이너를 다루지만, 백그라운드 [[데몬]]이 없고(daemonless) 루트 권한 없이도 동작하는(rootless) 컨테이너 엔진입니다. [[RHEL 9]]부터 기본 컨테이너 런타임으로 채택돼 있어서, 금융·공공기관처럼 규제가 엄격한 [[폐쇄망]] 환경에서 "왜 루트 권한이 필요한 데몬을 돌려야 하냐"는 보안 심사를 피하기 위해 Docker 대신 선택되는 경우가 많습니다. Red Hat이 Podman을 직접 만들었기 때문에 자기 회사 OS(RHEL)의 기본값으로 넣은 것이기도 합니다.

이미지 포맷은 Docker와 같은 OCI 표준을 쓰기 때문에, `docker save`/`docker load`로 익힌 이관 절차([[6단계 폐쇄망 이관]])를 `podman save`/`podman load`로 거의 그대로 옮길 수 있습니다.

rootless가 항상 매끄러운 건 아닙니다 — 특히 GPU를 컨테이너에 물려야 할 때는 CDI(Container Device Interface) 방식이 rootless보다 rootful(root 권한)에서 훨씬 안정적으로 동작하는 경우가 있어, GPU가 필요한 컨테이너만 예외적으로 rootful로 운영하고 나머지는 rootless를 유지하는 절충이 실무에서 흔합니다.

## 최악의 시나리오: Podman 자체를 반입해야 한다면

[[RHEL 9]]가 아닌 배포판(우분투·데비안 등)이라면 Podman이 기본 내장돼 있지 않아, Podman 자체도 [[체크섬|반입 대상]]이 됩니다. 이 경우에도 [[6단계 폐쇄망 이관]]에서 확립한 것과 같은 절차를 그대로 적용하면 됩니다 — 다만 두 가지를 더 신경 써야 합니다.

1. **의존 패키지까지 통째로 챙겨야 합니다.** Podman은 혼자 오지 않습니다 — `conmon`(컨테이너 프로세스 감시), `runc`/`crun`(실제 격리를 실행하는 저수준 실행기), `slirp4netns`·`netavark`·`aardvark-dns`(rootless 네트워킹), `fuse-overlayfs`(rootless 파일시스템)가 딸려옵니다. 파이썬 패키지를 `pip download -r requirements.txt`로 의존성까지 통째로 챙겼던 것과 같은 이유로, Podman 패키지 하나만 반입하면 설치가 안 됩니다.

   ```bash
   dnf download --resolve podman conmon runc slirp4netns fuse-overlayfs netavark aardvark-dns \
     --destdir ./staging/packages/podman
   ```

2. **공급망 검증 범위가 넓어집니다.** RHEL이 기본 내장해줄 때는 "Red Hat이 이미 검증한 것"이라는 암묵적 신뢰가 있지만, 직접 반입하면 "이 패키지가 진짜 공식 배포본이 맞는가"까지 심사팀이 별도로 확인해야 합니다 — 컨테이너 이미지 하나의 출처를 검증하는 걸 넘어 **컨테이너 엔진 자체의 공급망**까지 심사 범위가 넓어지는 셈입니다.

## 관련
- [[6단계 폐쇄망 이관]] — Docker 대신 Podman으로도 그대로 되는 이관 절차
- [[폐쇄망]] — 루트리스 구조가 특히 매력적인 이유
- [[RHEL 9]] — Podman을 기본 도구로 채택한 OS이자, Podman을 만든 회사(Red Hat)의 제품
- [[데몬]] — Podman이 갖지 않는 것, Docker가 가진 것
- [[WSL2]] — GPU 없이 실습할 때 물리 RHEL 9 서버 대신 쓰는 환경

## 실전 사례
- [EP07. RHEL 9에서 Podman 컨테이너 운영 (구축기)](../../docs/구축기/EP07-RHEL9-Podman-컨테이너-운영.md) — rootless + Quadlet으로 서비스를 등록한 과정
- [EP08. vLLM 배포 — V100 3장 (구축기)](../../docs/구축기/EP08-vLLM-배포-V100-3장.md) — rootless GPU 패스스루가 불안정해 vLLM 컨테이너만 rootful로 예외 운영한 실제 판단
- [05. GPU 없이 CPU로 구축기 스택 실습하기](../../docs/05-CPU-실습-환경-구축-트러블슈팅.md) — WSL2 위 AlmaLinux 9에 실제로 Podman을 설치하고 rootless 계정을 구성한 과정
