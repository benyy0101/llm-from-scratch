---
tags: [concept, infra]
aliases: ["Red Hat Enterprise Linux", "RHEL"]
---

# RHEL 9

Red Hat Enterprise Linux — Red Hat(현재 IBM 소속)이 만드는 기업용 유료 리눅스 배포판입니다. 리눅스는 오픈소스라 우분투·데비안·페도라·RHEL처럼 여러 "배포판"이 있는데, RHEL은 그중 돈을 내고 공식 기술 지원·긴 보안 패치 기간(한 버전당 약 10년)·인증된 안정성을 받는 기업용 버전입니다. "9"는 버전 번호(2022년 출시)입니다.

## 왜 금융·공공기관이 RHEL을 쓰나

규제 산업은 "문제가 생겼을 때 책임질 곳이 있는 OS"를 요구합니다. 무료 커뮤니티 리눅스는 지원 계약이 없어 보안 심사·감사를 통과하기 어렵고, RHEL은 공식 지원 계약이 있는 회사 제품이라 이 요건을 만족시킵니다.

[[Podman]]이 RHEL 9부터 기본 컨테이너 도구로 채택된 것도 우연이 아닙니다 — Podman 자체를 Red Hat이 직접 만들었기 때문입니다.

## 관련
- [[Podman]] — RHEL 9의 기본 컨테이너 도구, Red Hat이 만든 제품
- [[폐쇄망]] — RHEL 같은 지원 계약이 있는 OS를 요구하는 배경
- [[데몬]] — RHEL의 systemd가 관리하는 백그라운드 프로세스들
- [[WSL2]] — 물리 RHEL 9 서버가 없을 때, 같은 계열(AlmaLinux 9)로 실습을 재현하는 방법

## 실전 사례
- [EP07. RHEL 9에서 Podman 컨테이너 운영](../../docs/구축기-v1.0/EP07-RHEL9-Podman-컨테이너-운영.md) — 실제로 RHEL 9 서버 위에서 스택을 구성한 과정
- [05. GPU 없이 CPU로 구축기 스택 실습하기](../../docs/05-CPU-실습-환경-구축-트러블슈팅.md) — WSL2 위에 AlmaLinux 9(RHEL 9 계열)를 올려 EP07 절차를 그대로 재현한 실습 기록
