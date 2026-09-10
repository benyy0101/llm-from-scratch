# EP05. Hermes Agent 반입 패키지 제작

> 시리즈: [폐쇄망 멀티에이전트 구축기](README.md) · 이전: [EP04. LiteLLM Router 다중 노드 라우팅 설계](EP04-LiteLLM-Router-다중노드-라우팅-설계.md) · 다음: [EP06. 4노드 vLLM 배포 (역할별 모델 분리)](EP06-4노드-vLLM-배포-역할별-모델-분리.md)

Phase 2(인터넷망 준비) 마지막 편이에요. [1편(구축기 v1.0) EP06](../구축기-v1.0/EP06-반입-패키지-제작.md)에서 만든 `manifest.json` 방식은 그대로 재활용했는데, 이번엔 두 가지가 늘었어요 — **노드가 4개**라 반입물이 노드별로 흩어지고, **Hermes Agent라는 완전히 새로운 컴포넌트**가 추가됐다는 점입니다.

## 🗂️ manifest.json에 `target_node` 필드를 추가했어요

1편 매니페스트는 "이 파일이 뭔지"만 적으면 됐는데, 이번엔 "이 파일이 어느 노드로 가는지"까지 적어야 반입 후 배치 실수를 막을 수 있더라구요.

```json
{
  "package_id": "airgap-package-hermes-2026Q3",
  "requested_by": "AI혁신센터 (담당자명)",
  "purpose": "멀티에이전트 오케스트레이션 플랫폼 구축 (Hermes Agent + 4노드 vLLM)",
  "items": [
    {
      "path": "images/vllm-h100.tar",
      "type": "container_image",
      "target_node": ["node1", "node2", "node3", "node4"],
      "source": "vllm-project/vllm 소스 빌드 (TORCH_CUDA_ARCH_LIST=9.0)",
      "size_gb": 14.2,
      "contains_pii": false
    },
    {
      "path": "models/qwen2.5-72b-instruct",
      "type": "model_weights",
      "target_node": ["node1", "node4"],
      "source": "Qwen/Qwen2.5-72B-Instruct (원본)",
      "size_gb": 145.0,
      "contains_pii": false
    },
    {
      "path": "models/qwen2.5-coder-32b",
      "type": "model_weights",
      "target_node": ["node2", "node4"],
      "source": "Qwen/Qwen2.5-Coder-32B-Instruct (원본)",
      "size_gb": 65.0,
      "contains_pii": false
    },
    {
      "path": "models/bge-m3",
      "type": "model_weights",
      "target_node": ["node3"],
      "source": "BAAI/bge-m3 (Hugging Face)",
      "size_gb": 2.3,
      "contains_pii": false
    },
    {
      "path": "images/hermes-agent.tar",
      "type": "container_image",
      "target_node": ["gateway-host"],
      "source": "Nous Research Hermes Agent (공식 릴리스, 소스 검증 후 오프라인 빌드)",
      "size_gb": 1.8,
      "contains_pii": false
    },
    {
      "path": "images/litellm.tar",
      "type": "container_image",
      "target_node": ["gateway-host"],
      "source": "berriai/litellm 공식 이미지",
      "size_gb": 0.9,
      "contains_pii": false
    },
    {
      "path": "configs/bots/*.yaml",
      "type": "config",
      "target_node": ["gateway-host"],
      "source": "사내 작성 — 리서처·법무검토·품질분석 봇 프로필(코더는 Hermes 미경유라 제외)",
      "size_gb": 0.01,
      "contains_pii": false
    }
  ],
  "checksum_file": "CHECKSUMS.sha256",
  "approval_status": "pending"
}
```

노드4는 예비 노드라 `qwen2.5-72b-instruct`와 `qwen2.5-coder-32b` 둘 다 미리 반입해뒀어요 — [EP03](EP03-DGX-노드-선정과-독립형-vs-클러스터형-판단.md)에서 정한 대로 장애 시 즉시 둘 중 하나로 기동할 수 있어야 하니까요. 평시엔 컨테이너를 내려둔 상태로만 유지합니다.

## 🤖 Hermes Agent는 소스 검증 절차가 하나 더 붙었어요

1편의 vLLM·LiteLLM은 이미 사내에서 여러 번 반입해본 이미지라 검증 절차가 익숙했는데, Hermes Agent는 이번이 처음이라 보안팀이 추가 조건을 걸었습니다.

- [ ] 공식 GitHub 릴리스 태그 기준으로만 받고, `main` 브랜치 최신 커밋은 반입 금지
- [ ] Python 의존성(`requirements.txt`) 전체를 오프라인 pip 미러로 사전 빌드해 `.whl` 목록을 매니페스트에 첨부
- [ ] `runtime_provider.py`를 포함한 소스 전체를 정적 분석 도구로 1차 스캔 (외부 네트워크 호출 지점 확인)
- [ ] Bot Mode의 `hermes peer`(머신 간 통신 기능)는 이번 반입에서 **비활성화 빌드**로 요청 — 4노드가 이미 물리적으로 분리돼 있어 이 기능 자체가 불필요하고, 괜히 열어두면 보안 검토 항목만 늘어남

마지막 항목이 좀 의외였는데, 저희도 처음엔 "나중에 쓸 수도 있는데 왜 빼냐"고 반문했어요. 근데 [EP01](EP01-왜-멀티에이전트-오케스트레이션인가.md)에서 이미 "라우팅은 LiteLLM 하나로 단일화"하기로 정했으니, 노드 간 직접 통신 기능(`hermes peer`)은 애초에 쓸 일이 없는 게 맞았습니다. 안 쓰는 기능을 열어두는 것 자체가 반입 심사 항목만 늘리는 거더라구요 — 이번 반입 준비하면서 배운 것 중 하나예요.

## ✅ 반입 전 체크리스트 (1편 대비 추가분만)

1편 체크리스트([EP06](../구축기-v1.0/EP06-반입-패키지-제작.md))는 그대로 따르고, 이번엔 이 항목들을 더했어요.

- [ ] `manifest.json`의 모든 `target_node`가 [EP03](EP03-DGX-노드-선정과-독립형-vs-클러스터형-판단.md)에서 정한 배치와 정확히 일치하는지 재확인
- [ ] Hermes Agent 릴리스 태그·커밋 해시를 `manifest.json`에 별도 명시
- [ ] `hermes peer` 비활성화 빌드 여부를 반입물 자체에서 확인(설정이 아니라 빌드 단계에서 빠졌는지)
- [ ] 봇 프로필 YAML(`configs/bots/*.yaml`)에 하드코딩된 API 키·비밀값이 없는지 확인

## 📋 이번 편에서 확정한 것

| 항목 | 결정 |
|---|---|
| 매니페스트 확장 | 1편 구조에 `target_node` 필드만 추가, 나머지는 그대로 재사용 |
| 노드4(예비) 반입물 | 추론용·코드용 모델 둘 다 사전 반입, 평시엔 미기동 |
| Hermes Agent 반입 조건 | 공식 릴리스 태그만, `hermes peer` 비활성화 빌드로 요청 |
| 검증 절차 추가분 | 정적 분석 스캔, 오프라인 pip 미러 `.whl` 목록 첨부 |

Phase 2는 여기서 끝나고, 다음 편부터 Phase 3(폐쇄망 설치) — 이 패키지를 실제로 4개 노드에 풀고 vLLM을 역할별로 배포합니다.

---

📌 **EP.05 한 줄 요약**
1편의 manifest.json 방식에 `target_node` 필드만 추가해 4노드 반입물을 관리하고, Hermes Agent는 공식 릴리스만 반입하되 불필요한 노드 간 통신 기능(`hermes peer`)은 비활성화 빌드로 요청해 보안 검토 항목을 줄였다.

**다음 편**: [EP06. 4노드 vLLM 배포 (역할별 모델 분리)](EP06-4노드-vLLM-배포-역할별-모델-분리.md)
