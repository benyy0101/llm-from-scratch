# EP06. 4노드 vLLM 배포 (역할별 모델 분리)

> 시리즈: [폐쇄망 멀티에이전트 구축기](README.md) · 이전: [EP05. Hermes Agent 반입 패키지 제작](EP05-Hermes-Agent-반입-패키지-제작.md) · 다음: [EP07. LiteLLM Router least-busy 라우팅 구성](EP07-LiteLLM-Router-least-busy-라우팅-구성.md)

Phase 3(폐쇄망 설치) 첫 편이에요. 반입까지 끝났으니 이제 [EP03](EP03-DGX-노드-선정과-독립형-vs-클러스터형-판단.md)에서 정한 대로 노드 4대에 각자 역할을 심는 차례입니다. [1편(구축기 v1.0)](../구축기-v1.0/README.md)에서 V100 3장 묶느라 rootless GPU 문제로 하루 반나절을 날렸던 기억이 있어서, 이번엔 그 경험부터 재활용했어요.

## 🔁 rootful 예외 정책, H100에서도 그대로 가져왔어요

[1편 EP08](../구축기-v1.0/EP08-vLLM-배포-V100-3장.md)에서 "vLLM 컨테이너만 rootful로 예외 운영"하기로 정했던 결정, H100 + 최신 CDI 조합이면 좀 나아졌을까 싶어서 노드1에서 rootless로 먼저 테스트해봤어요. 결과는 여전히 간헐적 실패 — CDI 자체는 성숙해졌지만, GPU 4장을 텐서 병렬로 동시에 물리는 워크로드에서 rootless 소켓 권한 문제가 똑같이 재현됐습니다. 그래서 **1편과 동일한 정책(vLLM만 rootful, 나머지 rootless + 보완 통제 문서화)을 4노드 전체에 그대로 적용**하기로 했어요 — 세대가 바뀌어도 이 판단 기준 자체는 안 바뀐다는 걸 확인한 셈입니다.

## 🖥️ 노드별 vLLM 실행 커맨드

[EP03](EP03-DGX-노드-선정과-독립형-vs-클러스터형-판단.md)에서 잡은 GPU 배분을 그대로 `--tensor-parallel-size`에 반영했어요. GPU가 H100 80GB로 넉넉해서, 1편처럼 GPTQ Int4까지 갈 필요 없이 **BF16 그대로 서빙**했습니다.

```bash
# 노드1 — 추론용 (Qwen2.5-72B-Instruct, GPU 4장)
podman run -d --name vllm-infer \
  --device nvidia.com/gpu=0,1,2,3 \
  --network airgap-net \
  -v /shared/models/qwen2.5-72b-instruct:/models/infer:Z \
  -p 8000:8000 \
  localhost/vllm-h100:latest \
  vllm serve /models/infer \
    --tensor-parallel-size 4 \
    --max-model-len 16384 \
    --gpu-memory-utilization 0.85 \
    --max-num-seqs 24

# 노드2 — 코드용 (Qwen2.5-Coder-32B, GPU 2장)
podman run -d --name vllm-code \
  --device nvidia.com/gpu=0,1 \
  --network airgap-net \
  -v /shared/models/qwen2.5-coder-32b:/models/code:Z \
  -p 8000:8000 \
  localhost/vllm-h100:latest \
  vllm serve /models/code \
    --tensor-parallel-size 2 \
    --max-model-len 32768 \
    --gpu-memory-utilization 0.85 \
    --max-num-seqs 32

# 노드3 — 임베딩용 (BGE-M3, GPU 1장)
podman run -d --name vllm-embed \
  --device nvidia.com/gpu=0 \
  --network airgap-net \
  -v /shared/models/bge-m3:/models/embed:Z \
  -p 8000:8000 \
  localhost/vllm-h100:latest \
  vllm serve /models/embed \
    --task embed \
    --max-num-seqs 40
```

노드2(코드용)만 `--max-model-len`을 32768로 크게 잡았는데, 코딩 어시스턴트가 파일 여러 개를 컨텍스트에 물고 오는 경우가 많아서 [1편 EP17](../구축기-v1.0/EP17-Continue.dev-코딩-어시스턴트.md)에서 겪었던 "컨텍스트 부족" 문제를 미리 피하려는 목적이었어요.

## 🗃️ Quadlet으로 4노드에 동일한 방식 반복

[1편 EP07](../구축기-v1.0/EP07-RHEL9-Podman-컨테이너-운영.md)에서 정리한 Quadlet(systemd 등록) 방식을 그대로 4노드에 반복 적용했습니다. 노드마다 모델·GPU 인덱스만 다르고 유닛 구조는 동일해서, 템플릿 하나를 두고 노드별 값만 바꿔 넣는 방식으로 관리했어요.

```ini
# /etc/containers/systemd/vllm-role.container (노드별로 값만 치환해 배포)
[Container]
Image=localhost/vllm-h100:latest
ContainerName=vllm-%i
Exec=vllm serve /models/%i --tensor-parallel-size ${TP_SIZE} --max-num-seqs ${MAX_SEQS}
Volume=/shared/models/%i:/models/%i:Z
PublishPort=8000:8000
Network=airgap-net

[Service]
Restart=always

[Install]
WantedBy=multi-user.target
```

노드가 1대일 때는 그냥 커맨드 하나 외우면 됐는데, 4대가 되니까 "노드마다 설정이 미묘하게 다른데 어떤 게 어떤 노드 건지" 헷갈리는 문제가 실제로 생기더라구요. 유닛 이름에 역할(`vllm-infer`, `vllm-code`, `vllm-embed`)을 그대로 박아둔 게 나중에 EP11(모니터링 & 페일오버)에서 알람 라우팅할 때도 그대로 도움이 됐습니다.

## ✅ 노드별로 따로 벤치마크했어요

[1편 EP08](../구축기-v1.0/EP08-vLLM-배포-V100-3장.md)처럼 각 노드에 동시 요청을 던져 [EP02](EP02-전체-아키텍처-설계.md) 목표(부서별 피크 동시 접속)를 만족하는지 확인했어요.

| 노드 | 목표 동시 요청 | 실측 첫 토큰 | 실측 전체 응답 |
|---|---|---|---|
| 노드1(추론) | 24 | 평균 1.4초 | 평균 9초 |
| 노드2(코드) | 32 | 평균 0.9초 | 평균 6초 |
| 노드3(임베딩) | 40 | 평균 0.3초 | 평균 0.6초 |

노드4(예비)는 평시에 컨테이너를 내려둔 상태라 이번 벤치마크에선 제외했고, 실제 기동 테스트는 EP11에서 페일오버 훈련으로 따로 다룹니다. 다만 각 노드가 개별로는 목표를 여유 있게 만족했다는 건 확인했어요 — **여러 노드가 동시에 바쁠 때 LiteLLM Router가 실제로 부하를 잘 나누는지**는 다음 편에서 검증합니다.

## 📋 이번 편에서 확정한 것

| 항목 | 결정 |
|---|---|
| GPU 운영 정책 | 1편과 동일하게 vLLM만 rootful, 나머지 rootless(H100에서도 재확인 후 유지) |
| 정밀도 | BF16 그대로 서빙(GPTQ 재양자화 불필요 — H100 VRAM 여유) |
| 노드별 max-num-seqs | 노드1: 24, 노드2: 32, 노드3: 40 |
| 배포 방식 | Quadlet 템플릿 하나를 노드별 값만 치환해 반복 적용 |
| 개별 노드 벤치마크 | 3개 노드 모두 EP02 목표 여유 있게 충족 |

다음 편에서는 이 4노드를 LiteLLM Router가 실제로 `least-busy`로 얼마나 잘 분산하는지, [EP04](EP04-LiteLLM-Router-다중노드-라우팅-설계.md)에서 설계만 했던 걸 실제로 켜고 검증합니다.

---

📌 **EP.06 한 줄 요약**
1편의 rootful 예외 정책과 Quadlet 배포 방식을 H100 4노드에 그대로 재활용해 역할별(추론·코드·임베딩)로 vLLM을 배포했고, 개별 노드는 모두 EP02 목표를 여유 있게 만족했다.

**다음 편**: [EP07. LiteLLM Router least-busy 라우팅 구성](EP07-LiteLLM-Router-least-busy-라우팅-구성.md)
