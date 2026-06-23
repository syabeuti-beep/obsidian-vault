---
type: learning-note
topic: vector-database
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Vector Database

## 한 줄 요약

[[Vector Database]]는 embedding vector를 저장하고 유사한 벡터를 빠르게 검색하기 위한 데이터베이스이다.

---

## 핵심 개념

- RAG에서 문서 chunk embedding을 저장하는 핵심 구성 요소이다.
- query embedding이 들어오면 가까운 문서 벡터 top-k를 찾는다.
- metadata filtering, approximate nearest neighbor search, hybrid search 기능을 제공할 수 있다.

---

## 수학적 형태

가장 기본적인 검색은 다음 최적화로 볼 수 있다.

$$
\operatorname{argmax}_{d_i}\ sim(E(q),E(d_i))
$$

---

## 컴퓨팅 관점의 입력과 출력

입력은 vector와 metadata입니다. 검색 시 query vector `[embedding_dim]`를 넣으면 top-k document IDs, scores, metadata가 출력됩니다.

---

## 간단한 toy 예시

10000개 논문 chunk embedding을 저장해 두고 질문 embedding 하나를 넣으면 가장 가까운 5개 chunk를 반환합니다.

---

## 관련 노트

- [[Embedding Model]]
- [[Semantic Search]]
- [[Hybrid Search]]
- [[Retrieval-Augmented Generation (RAG)]]

---

## 내가 기억해야 할 핵심

- Vector Database는 Vector Database는 embedding vector를 저장하고 유사한 벡터를 빠르게 검색하기 위한 데이터베이스이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
