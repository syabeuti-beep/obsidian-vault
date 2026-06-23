---
type: learning-note
topic: semantic-search
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Semantic Search

## 한 줄 요약

[[Semantic Search]]는 단어가 정확히 일치하는지보다 의미적으로 유사한 문서를 찾는 검색 방식이다.

---

## 핵심 개념

- 질문과 문서를 embedding vector로 바꾸고 vector similarity로 검색한다.
- 동의어, paraphrase, 표현 차이를 어느 정도 처리할 수 있다.
- RAG에서 관련 context를 찾는 기본 방법이다.

---

## 수학적 형태

$$
score(q,d)=\cos(E(q),E(d))
$$

점수가 높은 문서 chunk를 top-k로 가져온다.

---

## 컴퓨팅 관점의 입력과 출력

입력은 query string 또는 query embedding이고 출력은 유사도 점수로 정렬된 문서 목록입니다.

---

## 간단한 toy 예시

질문 `boiling heat transfer correlation`을 넣으면 정확히 같은 단어가 없어도 `CHF prediction model` 문서가 검색될 수 있습니다.

---

## 관련 노트

- [[Embedding Model]]
- [[Vector Database]]
- [[Hybrid Search]]
- [[Retrieval-Augmented Generation (RAG)]]

---

## 내가 기억해야 할 핵심

- Semantic Search는 Semantic Search는 단어가 정확히 일치하는지보다 의미적으로 유사한 문서를 찾는 검색 방식이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
