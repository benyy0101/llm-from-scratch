---
tags: [concept, rag, retrieval]
aliases: ["ANN", "Approximate Nearest Neighbor", "ANN Recall"]
---

# ANN 검색과 검색 재현율

ANN(근사 최근접 이웃 검색)은 모든 벡터를 비교하는 대신 인덱스로 탐색량을 줄여 가까운 벡터를 찾는 방법입니다. 속도를 얻는 대신 정확 검색의 상위 결과 일부를 놓칠 수 있습니다.

## ANN Recall

정확 검색 상위 k개를 Eₖ, 근사 검색 상위 k개를 Aₖ라고 두면:

$$ANN\ Recall@k=\frac{|E_k\cap A_k|}{k}$$

정확 검색 상위 10개 중 9개를 찾으면 90%입니다. 기준은 **사람이 정한 정답이 아니라 정확한 벡터 검색 결과**입니다.

## 관련 근거 Recall과의 차이

ANN recall이 100%여도 정답 조항이 벡터 공간에서 멀리 있으면 찾지 못합니다. 검색 알고리즘은 가까운 벡터를 잘 찾았지만, 임베딩이 질문과 정답의 관계를 잘 표현하지 못한 경우입니다.

- 정확 검색에는 정답이 있고 ANN에는 없다 → 탐색 설정 확인
- 둘 다 정답이 없다 → 임베딩·청킹·질의·필터 확인

비교할 때는 문서·임베딩·거리 함수·필터를 같게 둡니다. 탐색량을 바꾸면 지연도 함께 측정합니다.

## 관련

- [[임베딩과 벡터DB]] — 검색 대상 표현과 저장소
- [[Precision과 Recall]] — 사람의 관련성 판단을 기준으로 하는 지표
- [[RAG 평가]] — 실패 원인을 좁혀가는 절차

## 출처

[Qdrant — Measuring ANN Recall](https://qdrant.tech/documentation/tutorials-search-engineering/ann-recall/)
