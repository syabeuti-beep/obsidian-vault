---
type: learning-note
topic: re-ranking
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Re-ranking

## 한 줄 요약

[[Re-ranking]]은 1차 검색으로 얻은 후보 문서를 더 정밀한 모델로 다시 정렬하는 단계이다.

---

## 핵심 개념

- 빠른 vector search는 recall을 확보하고, re-ranker는 상위 후보의 precision을 높인다.
- cross-encoder, LLM judge, domain-specific scoring model을 사용할 수 있다.
- RAG에서 잘못된 context가 prompt에 들어가는 것을 줄여준다.

---

## 수학적 형태

1차 검색에서 top-50을 얻고 re-ranker가 점수를 다시 계산해 top-5를 고르는 식으로 사용한다.

$$
TopK_{final}=rerank(TopN_{retrieved}, q)
$$

---

## 컴퓨팅 관점의 입력과 출력

입력은 query와 1차 검색 후보 list이고 출력은 재정렬된 후보 list입니다. 예: top-50 → top-5.

---

## 간단한 toy 예시

vector search가 비슷한 문서 50개를 가져오면 cross-encoder가 질문과 각 chunk를 함께 읽고 실제 관련도 순서로 다시 정렬합니다.

---

## 관련 노트

- [[Retrieval-Augmented Generation (RAG)]]
- [[Semantic Search]]
- [[Hybrid Search]]
- [[Information Retrieval]]

---

## 내가 기억해야 할 핵심

- Re-ranking는 Re-ranking은 1차 검색으로 얻은 후보 문서를 더 정밀한 모델로 다시 정렬하는 단계이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
