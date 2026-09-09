---
tags: [roadmap, stage]
---

# 4단계 · 내 문서에 질문하기 (RAG)

사내 매뉴얼 QA의 축소판을 로컬에서 완전히 오프라인으로 완성하는 단계 (2~3시간). PDF 로드 → 쪼개기 → [[임베딩과 벡터DB|임베딩]] → Chroma 저장 → 검색 → [[RAG]] 프롬프트 구성의 5단계 흐름을 코드로 직접 짭니다.

```bash
pip install langchain langchain-community chromadb pypdf
ollama pull nomic-embed-text
```

## 개념 심화

[[RAG와 리랭킹 MOC]]를 따라 [[Precision과 Recall]] → [[하이브리드 검색]] → [[리랭킹]] → [[RAG 평가]]를 읽습니다. 기본 파이프라인을 만든 뒤 같은 질문과 근거 라벨로 검색·정렬 효과를 비교합니다.

## 이전 / 다음
← [[3단계 모델·양자화 비교]] · → [[5단계 프로덕션 서빙]]

## 관련
- [[RAG]] · [[임베딩과 벡터DB]] — 이 단계에서 구현하는 개념
- [[Midm 2.0]] — 문서 QA에 강점 있는 대안 모델
