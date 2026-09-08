# EP08. vLLM 배포 (V100 3장)

> 시리즈: [폐쇄망 LLM 구축기](README.md) · 이전: [EP07. RHEL 9에서 Podman 컨테이너 운영](EP07-RHEL9-Podman-컨테이너-운영.md) · 다음: [EP09. Ollama 폐쇄망 설치](EP09-Ollama-폐쇄망-설치.md)

이번 편이 이 시리즈에서 제일 오래 걸렸어요. EP03에서 예고했던 V100 관련 이슈들이 여기서 한꺼번에 터졌거든요. 결론부터 정리하면 — GPU를 rootless로 물리는 건 결국 포기하고, **vLLM 컨테이너 하나만 rootful(root 권한)로 예외 운영**하기로 했어요. 이 결정까지 어떤 삽질을 거쳤는지 순서대로 적어볼게요.

## ⚠️ 삽질 2 — rootless Podman에서 GPU가 안 잡혀요

EP07에서 만든 rootless 환경 그대로 vLLM 컨테이너를 띄웠더니 GPU를 아예 못 찾더라구요. 알아보니 NVIDIA의 **CDI(Container Device Interface)** 방식으로 GPU를 전달해야 하는데, 이게 privileged(root) 컨테이너에서는 잘 되는데 rootless에서는 아직 버그가 많다고 하더라구요.

일단 정석대로 CDI 스펙부터 만들어봤어요.

```bash
sudo dnf install -y nvidia-container-toolkit
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
nvidia-ctk cdi list   # GPU 3장이 잘 보이는지 확인
```

```bash
podman run --rm -it --device nvidia.com/gpu=all \
  --security-opt=label=disable \
  --group-add keep-groups \
  ubuntu nvidia-smi -L
```

`--group-add keep-groups`까지 넣어서 rootless 사용자 계정(`llmsvc`)의 보조 그룹 권한이 컨테이너 안에서도 유지되게 했는데도, vLLM처럼 GPU 3장을 동시에 물어야 하는 워크로드에서는 간헐적으로 GPU 인식이 실패했어요. 하루 반나절을 여기 쓰고 나서 결론을 내렸습니다 — **당장은 안정성이 더 중요하니, vLLM 컨테이너만 rootful로 돌리자.**

정보보호팀에는 이렇게 설명하고 예외 승인을 받았어요.

- GPU 디바이스 접근이 필요한 이 컨테이너 하나만 root 권한으로 운영한다.
- 나머지 서비스(LiteLLM, Qdrant, Open WebUI, Ollama)는 EP07대로 rootless를 유지한다.
- 이 컨테이너는 외부 네트워크 접근이 전혀 없고(`--network=airgap-net`으로 격리), 이미지 자체도 EP04에서 소스 빌드해 내용을 다 아는 이미지다.
- 대신 이 서버에 대한 SSH 접근 로그와 컨테이너 실행 이력을 감사 로그로 이중으로 남긴다(EP13에서 다룰 예정).

"전부 다 rootless로 완벽하게" 보다 "위험을 인지하고, 그 위험을 보완 통제로 감싸고, 문서로 남긴다"는 쪽으로 방향을 잡았어요 — 보안이라는 게 결국 이런 판단의 연속이더라구요.

## 🖥️ Tensor Parallelism으로 V100 3장 묶기

```bash
podman run -d --name vllm-server \
  --device nvidia.com/gpu=all \
  --network airgap-net \
  -v /opt/models/qwen2.5-32b-gptq-int4:/models/qwen2.5-32b:Z \
  -p 8000:8000 \
  localhost/vllm-v100:latest \
  vllm serve /models/qwen2.5-32b \
    --tensor-parallel-size 3 \
    --quantization gptq \
    --max-model-len 8192 \
    --gpu-memory-utilization 0.90 \
    --max-num-seqs 32
```

- `--tensor-parallel-size 3` — [[Tensor Parallelism]]으로 V100 3장에 가중치를 나눔. EP03에서 계산한 "가중치 64GB를 3장에 분산"이 여기서 실현됩니다.
- `--quantization gptq` — EP04에서 만든 GPTQ Int4 체크포인트를 명시적으로 지정.
- `--gpu-memory-utilization 0.90` — VRAM의 90%까지 [[KV 캐시]]·[[배치]] 용도로 씀. 나머지 10%는 안전 마진.
- `--max-num-seqs 32` — 동시 배치 상한. EP02에서 잡은 "동시 15명" 목표에 여유를 좀 두고 32로 잡았어요.

## ✅ 첫 벤치마크 — EP02 목표를 실제로 만족하는지

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "/models/qwen2.5-32b", "messages": [{"role": "user", "content": "사내 연차 규정 요약해줘"}], "stream": true}'
```

동시 15개 요청을 부하 테스트 스크립트로 던져봤더니 첫 토큰까지 평균 1.8초, 전체 응답 완료까지 평균 12초가 나왔어요. EP02에서 잡은 목표(첫 토큰 3초, 전체 20초)를 여유 있게 만족했습니다. GPTQ Int4로 정확도가 얼마나 깎였는지는 별도로 사내 QA 셋 50문항으로 사람이 직접 채점했는데, AWQ 기준 자료에서 흔히 언급되는 손실 폭과 비슷한 수준이라 실사용엔 문제없다고 판단했어요.

---

📌 **EP.08 한 줄 요약**
rootless Podman에서 GPU가 안정적으로 안 잡혀서, vLLM 컨테이너만 예외적으로 rootful 운영하기로 하고 보완 통제를 문서화했다. Tensor Parallelism으로 V100 3장을 묶어 EP02 목표(첫 토큰 3초, 동시 15명)를 여유 있게 만족시켰다.

**다음 편**: [EP09. Ollama 폐쇄망 설치](EP09-Ollama-폐쇄망-설치.md)

출처: [Podman rootless GPU 이슈 (GitHub containers/podman #17539)](https://github.com/containers/podman/issues/17539) · [NVIDIA CDI 설정 (oneuptime)](https://oneuptime.com/blog/post/2026-03-18-run-nvidia-gpu-containers-podman/view)
