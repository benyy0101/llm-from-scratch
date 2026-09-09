---
tags: [concept, rag, evaluation]
aliases: ["정밀도와 재현율", "Recall@k", "Precision@k"]
---

# Precision과 Recall

**Precision(정밀도)**은 가져온 결과 중 관련 있는 비율, **Recall(재현율)**은 찾아야 할 관련 자료 중 찾아온 비율입니다. `@k`는 상위 k개를 기준으로 평가한다는 뜻입니다.

## 계산

관련 자료 집합을 G, 상위 k개 결과를 Sₖ라고 하면 다음과 같습니다. 결과가 k개 반환되는 경우입니다.

$$Precision@k=\frac{|S_k\cap G|}{k}, \qquad Recall@k=\frac{|S_k\cap G|}{|G|}$$

## 리랭킹 전후 예시

관련 근거가 A·B·C·D 네 개이고, 검색 후보 20개에는 A·B·C가 포함됐다고 가정합니다.

| 평가 범위 | 포함된 근거 | Precision | Recall |
|---|---|---|---|
| 후보 20개 | A·B·C | 3/20 = 15% | 3/4 = 75% |
| 기존 상위 5개 | A | 1/5 = 20% | 1/4 = 25% |
| 리랭킹 후 상위 5개 | A·B·C | 3/5 = 60% | 3/4 = 75% |

**Recall@5는 올랐지만 Recall@20은 그대로**입니다. 후보에 없는 D는 정렬만으로 찾을 수 없습니다. 후보에서 고르기만 한다면 최종 Recall@k는 후보 Recall@N을 넘지 못합니다.

## 평가 기준을 고정할 것

- **단위** — 문서인지 청크인지 정합니다. 같은 근거를 여러 조각으로 나눴다고 독립적인 정답 여러 개로 세지 않습니다.
- **정답 집합** — 라벨이 불완전하면 recall도 그 라벨 기준의 측정값입니다.
- **답 없는 질문** — 분모가 0이므로 별도로 처리하고 답변 보류를 평가합니다.
- **Hit@k** — 관련 결과가 하나라도 있으면 1입니다. 여러 근거가 필요한 질문의 누락을 드러내기 어렵습니다.

## 관련

- [[리랭킹]] — 최종 k개 안에 정답 근거를 올리는 단계
- [[RAG 평가]] — 순위·답변 품질까지 측정
- [[ANN 검색과 검색 재현율]] — 정답 근거 대신 정확 벡터 검색을 기준으로 하는 지표

## 출처

[Stanford IR 교재 — Precision과 Recall](https://nlp.stanford.edu/IR-book/html/htmledition/evaluation-of-unranked-retrieval-sets-1.html)
