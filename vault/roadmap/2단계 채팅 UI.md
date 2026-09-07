---
tags: [roadmap, stage]
---

# 2단계 · ChatGPT처럼 생긴 웹 UI 붙이기

[[Open WebUI]]를 [[Ollama]] 위에 Docker로 띄워, 브라우저에서 채팅하고 대화 기록·모델 전환이 되는 "사내용 ChatGPT" 최소 형태를 만듭니다 (30분).

```bash
docker run -d -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  ghcr.io/open-webui/open-webui:main
```

## 이전 / 다음
← [[1단계 첫 로컬 실행]] · → [[3단계 모델·양자화 비교]]

## 관련
- [[Open WebUI]] · [[Ollama]] — 이 단계에서 쓰는 도구
