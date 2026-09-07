# LLM From Scratch — 폐쇄망 LLM 구축 학습 저장소

LLM 사전지식이 없는 상태에서 시작해, 폐쇄망(air-gapped network)에 LLM을 올리는 것까지 가기 위한 개인 학습 자료 모음입니다. 로드맵/실습 가이드와 이론 노트를 문서로 정리하고, 실습 코드가 생기면 이 저장소에 함께 쌓습니다.

## 문서 구성

| 문서 | 내용 |
|---|---|
| [docs/01-roadmap.md](docs/01-roadmap.md) | 폐쇄망 LLM 구축 가이드 — 기초 개념, 실제 기업/기관 운영 사례, 표준 아키텍처, 0~7단계 학습 로드맵, 서빙 엔진·모델·하드웨어 비교표, 반입 절차 체크리스트 |
| [docs/02-serving-theory.md](docs/02-serving-theory.md) | 서빙 이론 노트 — Prefill/Decode, KV 캐시, 배칭, PagedAttention, 병렬화, 양자화가 서빙 속도에 미치는 영향 |

## 학습 순서

1. `docs/01-roadmap.md`의 **Part 1(기초 개념)** → **Part 2(사례)** → **Part 4(로드맵 0~7단계)** 순으로 실습
2. 5단계(프로덕션 서빙 엔진 전환)를 진행하기 전에 `docs/02-serving-theory.md`로 원리를 먼저 다지기
3. 실습 중 만든 스크립트·Dockerfile·requirements 등은 이 저장소의 하위 폴더(`labs/` 등)에 단계별로 쌓기

## 상태

- [x] 폐쇄망 LLM 로드맵 초안 정리
- [x] 서빙 이론 정리
- [ ] Stage 1~7 실습 코드/스크립트 추가
- [ ] 실제 하드웨어에서 벤치마크한 tok/s 수치로 7장 표 업데이트
