---
tags: [concept, rag, retrieval]
aliases: ["Bi-Encoder", "Cross-Encoder", "바이인코더", "크로스인코더"]
---

# Bi-Encoder와 Cross-Encoder

질문과 문서를 **따로 인코딩해 비교하느냐**, **함께 입력해 관련성을 계산하느냐**의 차이입니다. 문서 계산을 재사용할 수 있는지가 검색 비용을 가릅니다.

| 구분 | Bi-Encoder | Cross-Encoder |
|---|---|---|
| 입력 처리 | 질문·문서를 각각 인코딩 | 질문·문서를 한 쌍으로 입력 |
| 점수 | 두 표현의 유사도 | 쌍의 관련성 출력 |
| 문서 계산 | 미리 저장·재사용 가능 | 질문마다 다시 계산 |
| 대표 용도 | 대규모 후보 검색 | 후보 리랭킹 |

## 계산 구조

$$s_{bi}(q,d)=sim(E_q(q),E_d(d))$$
$$s_{cross}(q,d)=f(q,d)$$

첫 식의 E_d(d)는 질문이 바뀌어도 재사용합니다. 두 번째 식은 질문과 문서 사이의 토큰 관계를 함께 계산하므로 질문이 바뀌면 다시 평가합니다. 세부 조건을 구분하는 데 유리할 수 있지만 전체 문서에 적용하면 비용이 큽니다.

Bi-Encoder의 질문·문서 인코더는 가중치를 공유할 수도 있습니다. 이름이 반드시 별개 모델 두 개를 뜻하지는 않습니다.

## 관련

- [[임베딩과 벡터DB]] — Bi-Encoder의 문서 표현을 저장·검색
- [[리랭킹]] — 좁힌 후보에 Cross-Encoder를 적용
- [[Late Interaction]] — 문서의 토큰별 표현을 저장해 나중에 비교
- [[RAG 평가]] — 구조가 아닌 실제 질문으로 성능 확인

## 출처

[Sentence Transformers — Cross-Encoder와 Bi-Encoder](https://www.sbert.net/examples/cross_encoder/applications/README.html)
