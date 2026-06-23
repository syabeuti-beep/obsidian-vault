---
type: learning-note
topic: hybrid-search
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Hybrid Search

## 한 줄 요약

[[Hybrid Search]]는 keyword search와 semantic search를 결합하는 검색 방식이다.

---

## 핵심 개념

- keyword search는 정확한 용어, 약어, 식별자에 강하다.
- semantic search는 의미적 유사성에 강하다.
- RAG에서는 두 방법을 결합하면 검색 recall과 precision을 모두 개선할 수 있다.

---

## 수학적 형태

간단한 결합 점수는 다음처럼 쓸 수 있다.

$$
score=\alpha\,score_{semantic}+(1-\alpha)score_{keyword}
$$

---

## 컴퓨팅 관점의 입력과 출력

입력은 query이고 출력은 keyword score와 vector score를 결합한 ranking입니다. 구현에서는 BM25 결과와 vector top-k 결과를 merge합니다.

---

## 간단한 toy 예시

`FNO Darcy flow`처럼 약어와 전문용어가 섞인 query에서는 keyword가 FNO를 잡고 semantic search가 Darcy flow 관련 문맥을 잡습니다.

---

## 관련 노트

- [[Semantic Search]]
- [[Vector Database]]
- [[Information Retrieval]]
- [[Retrieval-Augmented Generation (RAG)]]

---

## 내가 기억해야 할 핵심

- Hybrid Search는 Hybrid Search는 keyword search와 semantic search를 결합하는 검색 방식이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
