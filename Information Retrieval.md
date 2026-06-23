---
type: learning-note
topic: information-retrieval
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Information Retrieval

## 한 줄 요약

[[Information Retrieval]]은 대량의 문서 중 사용자의 정보 요구와 관련 있는 문서를 찾아 순위를 매기는 분야이다.

---

## 핵심 개념

- 전통적 IR은 keyword matching, TF-IDF, BM25 같은 방법을 사용했다.
- 현대 RAG에서는 dense embedding 기반 semantic search도 중요하다.
- 검색 품질은 downstream LLM 답변 품질의 상한을 결정한다.

---

## 수학적 형태

검색 시스템은 query $q$와 문서 $d$ 사이의 관련도 점수 $s(q,d)$를 계산해 정렬한다.

$$
d_1,d_2,\cdots=rank_d\ s(q,d)
$$

---

## 컴퓨팅 관점의 입력과 출력

입력은 query와 document corpus이고 출력은 관련도 순위가 매겨진 document list입니다.

---

## 간단한 toy 예시

논문 5000편 중 `subcooled boiling CHF ML feature`와 관련 있는 초록 20개를 찾아오는 것이 IR 문제입니다.

---

## 관련 노트

- [[Semantic Search]]
- [[Hybrid Search]]
- [[Re-ranking]]
- [[Retrieval-Augmented Generation (RAG)]]

---

## 내가 기억해야 할 핵심

- Information Retrieval는 Information Retrieval은 대량의 문서 중 사용자의 정보 요구와 관련 있는 문서를 찾아 순위를 매기는 분야이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
