# LLM From Scratch — 폐쇄망 LLM 구축 학습 저장소

LLM 사전지식이 없는 상태에서 시작해, 폐쇄망(air-gapped network)에 LLM을 올리는 것까지 가기 위한 개인 학습 자료 모음입니다. 로드맵/실습 가이드와 이론 노트를 문서로 정리하고, 실습 코드가 생기면 이 저장소에 함께 쌓습니다.

이 저장소는 같은 내용을 **두 가지 형태**로 담고 있습니다. 처음부터 끝까지 순서대로 읽고 싶으면 `docs/`, 개념 하나에서 시작해 링크를 따라 탐색하고 싶으면 `vault/`를 쓰세요.

## docs/ — 선형으로 읽는 문서

| 문서 | 내용 |
|---|---|
| [docs/01-roadmap.md](docs/01-roadmap.md) | 폐쇄망 LLM 구축 가이드 — 기초 개념, 실제 기업/기관 운영 사례, 표준 아키텍처, 0~7단계 학습 로드맵, 서빙 엔진·모델·하드웨어 비교표, 반입 절차 체크리스트 |
| [docs/02-serving-theory.md](docs/02-serving-theory.md) | 서빙 이론 노트 — Prefill/Decode, KV 캐시, 배칭, PagedAttention, 병렬화, 양자화가 서빙 속도에 미치는 영향, 전통 서버 서빙과의 차이 |
| [docs/구축기/](docs/구축기/README.md) | 가상의 금융권 시나리오(A저축은행, V100×3)로 쓰는 실전 구축기 — 개념이 아니라 "그래서 실제로 뭘 설치·설정하는가"에 집중. 회차별 진행 상황은 인덱스 참고 |

## vault/ — 옵시디언 제텔카스텐 vault

위 두 문서를 개념 단위로 쪼개 `[[위키링크]]`로 서로 연결한 노트 모음입니다. [Obsidian](https://obsidian.md)에서 `vault/` 폴더를 그대로 열면 그래프 뷰·백링크가 바로 동작합니다.

| 폴더 | 내용 |
|---|---|
| [vault/MOC.md](vault/MOC.md) | 전체 지도(Map of Content) — 여기서 시작 |
| `vault/concepts/` | 원자 개념 노트 23개 (LLM, 토큰, 양자화, KV 캐시, PagedAttention, 전통 서버 서빙과의 차이 등) |
| `vault/engines/` | 서빙 엔진/도구 노트 11개 (Ollama, vLLM, Open WebUI, NVIDIA NIM 등) |
| `vault/models/` | 모델 노트 8개 (Qwen, EXAONE, HyperCLOVA X, Midm 2.0 등) |
| `vault/cases/` | 실제 운영 사례 노트 9개 (한국은행, 삼성SDS, IBM watsonx.ai 등) |
| `vault/roadmap/` | 0~7단계 로드맵 노트 8개, 각각 이전/다음 단계로 링크 |

각 노트는 짧은 정의 + `## 관련` 섹션으로 구성되어 있고, 다른 노트로의 링크가 곧 "왜 이게 다음으로 알아야 할 개념인가"를 나타냅니다. 원문(마케팅 수준 공개 vs 공식 문서 기반)의 신뢰도 표시(`[확인됨]`/`[참고용]`)는 사례 노트에 그대로 남겨뒀습니다.

## 학습 순서

1. `vault/MOC.md` 또는 `docs/01-roadmap.md` **Part 1(기초 개념)** → **Part 2(사례)** → **Part 4(로드맵 0~7단계)** 순으로 실습
2. 5단계(프로덕션 서빙 엔진 전환)를 진행하기 전에 `docs/02-serving-theory.md` 또는 `vault/concepts/`의 서빙 이론 노트로 원리를 먼저 다지기
3. 실습 중 만든 스크립트·Dockerfile·requirements 등은 이 저장소의 하위 폴더(`labs/` 등)에 단계별로 쌓기
4. vault에 새 개념이 생기면 `vault/concepts/`(또는 알맞은 폴더)에 노트를 추가하고, 관련 있는 기존 노트에서도 역방향 링크를 추가하기

## 상태

- [x] 폐쇄망 LLM 로드맵 초안 정리
- [x] 서빙 이론 정리 (전통 서버 서빙과의 차이 포함)
- [x] vault/ 옵시디언 제텔카스텐 노트 61개로 원자화
- [ ] Stage 1~7 실습 코드/스크립트 추가
- [ ] 실제 하드웨어에서 벤치마크한 tok/s 수치로 7장 표 업데이트
