---
type: learning-note
topic: query
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Query

## 한 줄 요약

[[Query]]는 attention에서 현재 위치가 찾고 싶은 정보의 기준 벡터이다.

---

## 핵심 개념

- 검색 시스템의 검색어와 비슷한 역할을 한다.
- 각 token의 hidden vector에 학습 가능한 행렬 $W_Q$를 곱해 만든다.
- Query와 [[Key]]의 유사도가 attention score가 된다.

---

## 수학적 형태

$$
Q=XW_Q
$$

attention score는 보통 다음 내적으로 계산된다.

$$
score=QK^T
$$

---

## 컴퓨팅 관점의 입력과 출력

Query tensor shape은 보통 `[batch, heads, seq_len, d_head]`입니다. 각 위치가 ‘무엇을 찾는지’를 나타내며 Key와 내적됩니다.

---

## 간단한 toy 예시

검색창에 `pump failure cause`를 입력하는 것처럼, attention의 query는 현재 token이 필요한 정보를 찾기 위한 벡터입니다.

---

## 관련 노트

- [[Key]]
- [[Value]]
- [[Query-Key-Value]]
- [[Attention Mechanism]]

---

## 내가 기억해야 할 핵심

- Query는 Query는 attention에서 현재 위치가 찾고 싶은 정보의 기준 벡터이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
