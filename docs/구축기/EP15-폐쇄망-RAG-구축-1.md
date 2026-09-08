# EP15. 폐쇄망 RAG 구축 ①

> 시리즈: [폐쇄망 LLM 구축기](README.md) · 이전: [EP14. 모델 업데이트 전략](EP14-모델-업데이트-전략.md) · 다음: [EP16. 폐쇄망 RAG 구축 ②](EP16-폐쇄망-RAG-구축-2.md)

Phase 5, "고급 활용 편"이에요. 지금까지는 인프라를 세우는 얘기였다면, 이제부터 두 편은 현업팀이 실제로 쓰는 [[RAG]] 파이프라인 자체를 다뤄볼게요. 이번 편은 문서를 어떻게 쪼개고 임베딩해서 [[임베딩과 벡터DB|Qdrant]]에 넣었는지, 다음 편은 검색·답변 생성 쪽입니다.

## 📄 청킹부터 삽질이었어요

처음엔 그냥 "500자마다 자르기"로 단순하게 시작했어요. 근데 검색 결과를 사람이 눈으로 확인해보니 문장 중간에서 뚝 잘려서 맥락이 끊긴 조각이 많았어요. 예를 들어 "연차는 입사일 기준으로 산정하며, 육아휴직 기간은"까지 잘리고 다음 조각이 "산정 대상 기간에서 제외한다"로 시작하는 식이었죠. 이러면 임베딩 자체가 애매한 의미를 담게 되고, 검색 정확도가 떨어져요.

그래서 글자 수 기준이 아니라 **문장 경계를 먼저 잡고, 문장 단위로 묶어서 청크를 만드는 방식**으로 바꿨어요. 한국어 문장 분리는 마침표만으로는 안 되는 경우(약관 문서 특유의 "제1조(목적)" 같은 조항 번호, 소수점 등)가 많아서, 문장 분리 라이브러리로 먼저 문장 단위를 잡고 그 위에서 청크를 조립했습니다.

```python
# chunk_docs.py
from kiwipiepy import Kiwi

kiwi = Kiwi()

def split_sentences(text: str) -> list[str]:
    return [s.text for s in kiwi.split_into_sents(text)]

def make_chunks(sentences: list[str], max_chars: int = 400, overlap_sentences: int = 1) -> list[str]:
    chunks, current = [], []
    current_len = 0
    for sent in sentences:
        if current_len + len(sent) > max_chars and current:
            chunks.append(" ".join(current))
            current = current[-overlap_sentences:]  # 문맥 유지를 위해 마지막 문장 겹치기
            current_len = sum(len(s) for s in current)
        current.append(sent)
        current_len += len(sent)
    if current:
        chunks.append(" ".join(current))
    return chunks
```

`overlap_sentences`로 청크 사이에 문장 하나를 겹치게 한 것도 포인트예요. 완전히 딱 잘라 나누면 경계에 걸친 내용이 양쪽 청크 어디에도 온전히 안 담기는 경우가 있어서, 약간의 중복을 감수하고 겹치게 했습니다.

## 🧮 임베딩은 BGE-M3로, 메타데이터를 같이 태웠어요

```python
# embed_and_upsert.py
import requests
from qdrant_client import QdrantClient
from qdrant_client.models import PointStruct

client = QdrantClient(host="qdrant", port=6333)

def embed(text: str) -> list[float]:
    resp = requests.post(
        "http://ollama-server:11434/api/embeddings",
        json={"model": "bge-m3", "prompt": text},
    )
    return resp.json()["embedding"]

def upsert_chunks(doc_id: str, revision_date: str, chunks: list[str]):
    points = [
        PointStruct(
            id=f"{doc_id}-{i}",
            vector=embed(chunk),
            payload={
                "doc_id": doc_id,
                "revision_date": revision_date,
                "text": chunk,
                "archived": False,
            },
        )
        for i, chunk in enumerate(chunks)
    ]
    client.upsert(collection_name="internal_docs", points=points)
```

`payload`에 `doc_id`·`revision_date`·`archived`를 같이 넣은 게 EP12에서 만든 upsert 파이프라인이랑 맞물리는 부분이에요. 검색할 때 `archived: false`인 것만 걸러서, 항상 최신 개정본만 답변에 쓰이게 했습니다.

## ✅ 컬렉션 생성

```python
from qdrant_client.models import VectorParams, Distance

client.create_collection(
    collection_name="internal_docs",
    vectors_config=VectorParams(size=1024, distance=Distance.COSINE),  # BGE-M3 임베딩 차원
)
```

이렇게 문서 반입 → 문장 분리 → 청킹 → 임베딩 → Qdrant 저장까지 한 파이프라인으로 묶었어요. 다음 편은 이제 사용자가 질문했을 때 이 데이터를 어떻게 검색하고, LLM한테 어떻게 넘겨서 답을 만드는지 다뤄볼게요.

---

📌 **EP.15 한 줄 요약**
글자 수 기준 청킹은 문맥이 끊겨서, 한국어 문장 분리기로 문장 경계를 먼저 잡고 약간의 겹침을 두고 청킹했다. BGE-M3로 임베딩하면서 개정일자·archived 여부를 메타데이터로 같이 저장해 항상 최신 문서만 검색되게 했다.

**다음 편**: [EP16. 폐쇄망 RAG 구축 ②](EP16-폐쇄망-RAG-구축-2.md)
