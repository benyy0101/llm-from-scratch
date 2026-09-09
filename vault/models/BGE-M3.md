---
tags: [model, embedding, rag]
---

# BGE-M3

BAAI가 공개한 다국어 검색 표현 모델입니다. [[RAG]]에서 문서·질문을 검색용 표현으로 바꾸며, 답변 문장을 생성하는 모델은 아닙니다.

## M3의 의미

- **Multi-Functionality** — dense·sparse·multi-vector 검색 지원
- **Multi-Linguality** — 다국어 지원
- **Multi-Granularity** — 다양한 길이의 입력 처리

Dense는 밀집 벡터, sparse는 학습된 어휘별 가중치, multi-vector는 토큰별 표현을 사용합니다. **Sparse 출력은 BM25와 동일한 계산이 아닙니다.**

## 모델 기능과 API 출력 구분

모델이 지원해도 서빙 API가 모든 표현을 반환하는 것은 아닙니다. 구축기의 Ollama 임베딩 호출은 dense 벡터 예시이며, 그것만으로 sparse·multi-vector가 색인된 것은 아닙니다.

[[BGE Reranker|bge-reranker-v2-m3]]는 질문·문서 쌍에 점수를 주는 별도 모델입니다. BGE-M3를 설치한 것으로 리랭커 준비까지 끝나지는 않습니다.

## 관련

- [[임베딩과 벡터DB]] — dense 표현의 저장·검색
- [[하이브리드 검색]] — 여러 검색 표현 결합
- [[Late Interaction]] — 토큰별 표현 비교
- [[BGE Reranker]] — 후보 관련성 평가용 모델
- [[RAG 평가]] — 한국어 사내 문서 성능은 별도 검증 필요

## 실전 사례

[EP15](../../docs/구축기/EP15-폐쇄망-RAG-구축-1.md) — 임베딩을 Qdrant에 저장하는 가상 구축 예시

## 출처

[BGE-M3 공식 모델 카드](https://huggingface.co/BAAI/bge-m3)
