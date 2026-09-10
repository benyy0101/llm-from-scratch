# EP09. Ollama 폐쇄망 설치

> 시리즈: [폐쇄망 LLM 구축기](README.md) · 이전: [EP08. vLLM 배포 (V100 3장)](EP08-vLLM-배포-V100-3장.md) · 다음: [EP10. LiteLLM 게이트웨이 구성](EP10-LiteLLM-게이트웨이-구성.md)

vLLM은 LLM 답변 생성 담당이고, 이번 편 주인공인 [[Ollama]]는 [[임베딩과 벡터DB|임베딩]] 담당이에요. [[모델 서빙과 서비스 배포]]에서 얘기했던 "모델은 하나가 아니라 둘"이 실제로 이렇게 구현됩니다. 임베딩 모델(BGE-M3)은 LLM만큼 무겁지 않아서 굳이 vLLM에 태울 필요 없이 Ollama로 가볍게 띄웠어요.

## 📦 오프라인 바이너리 설치

인터넷이 되는 스테이징에서 Ollama 바이너리를 미리 받아 tar로 반입했어요. 인터넷 되는 서버에서 흔히 쓰는 `curl ... | sh` 설치 스크립트는 그 자체가 인터넷 접속을 전제로 하기 때문에 폐쇄망에선 못 씁니다.

```bash
# (인터넷망 스테이징에서 미리 받아둔 걸 반입)
tar -C /usr -xzf ollama-linux-amd64.tgz
```

systemd 유닛도 EP07에서 배운 Quadlet 대신, Ollama는 컨테이너가 아니라 네이티브 바이너리라 일반 systemd 유닛 파일로 등록했어요.

```ini
# /etc/systemd/system/ollama.service
[Unit]
Description=Ollama embedding server

[Service]
User=llmsvc
ExecStart=/usr/bin/ollama serve
Environment="OLLAMA_HOST=0.0.0.0:11434"
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now ollama
```

## ⚠️ 삽질 3 — Ollama가 혼자 인터넷에 나가려고 해요

vault에도 적어뒀던 그 얘기가 저희한테도 그대로 일어났어요. Ollama가 백그라운드로 최신 버전 체크 요청을 주기적으로 보내려고 하더라구요. 폐쇄망이라 당연히 실패하지만, "실패하니까 상관없다"가 아니라 **애초에 그 시도 자체가 방화벽 로그에 계속 남는 게 문제**였어요. 보안팀 모니터링에서 "이상 아웃바운드 시도"로 계속 걸려서, 결국 서버 방화벽에서 아웃바운드를 전면 차단(`iptables` 기본 정책 DROP)하고 필요한 내부 통신만 화이트리스트로 열어주는 방식으로 정리했습니다.

## 📥 BGE-M3를 Modelfile로 반입

BGE-M3는 Ollama 공식 라이브러리에서 `ollama pull`로 받는 방식이 아니라(그 자체가 인터넷 접속), 인터넷망에서 미리 받은 GGUF 파일을 Modelfile로 직접 등록했어요.

```
# Modelfile
FROM ./bge-m3-q8_0.gguf
```

```bash
ollama create bge-m3 -f Modelfile
ollama list   # bge-m3가 등록됐는지 확인
```

## ✅ 임베딩 API 테스트

```bash
curl http://localhost:11434/api/embeddings \
  -d '{"model": "bge-m3", "prompt": "연차는 입사일 기준으로 산정한다"}'
```

벡터가 정상적으로 나오는 걸 확인하고, EP05에서 만든 [[LiteLLM]] 설정 파일의 `bge-m3-embed` 항목이 이 엔드포인트(`http://ollama-server:11434/v1`)를 정확히 가리키는지도 다시 한번 확인했습니다. 다음 편에서 이 LiteLLM을 실제로 폐쇄망 안에서 띄우고 연결을 확정할게요.

---

📌 **EP.09 한 줄 요약**
Ollama는 바이너리를 직접 반입해 systemd로 등록했고, 백그라운드 버전 체크 아웃바운드는 방화벽에서 전면 차단했다. BGE-M3는 `ollama pull` 대신 Modelfile로 GGUF를 직접 등록해 완전 오프라인으로 서빙했다.

**다음 편**: [EP10. LiteLLM 게이트웨이 구성](EP10-LiteLLM-게이트웨이-구성.md)
