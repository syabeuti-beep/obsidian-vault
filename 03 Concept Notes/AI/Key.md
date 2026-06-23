---
type: learning-note
topic: key
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Key

## 한 줄 요약

[[Key]]는 attention에서 각 위치가 어떤 정보를 가지고 있는지 색인처럼 나타내는 벡터이다.

---

## 핵심 개념

- Query가 찾는 기준이라면 Key는 각 token의 검색 가능한 주소 또는 색인이다.
- Query와 Key가 유사할수록 해당 token의 Value가 더 많이 반영된다.
- 각 token hidden vector에 $W_K$를 곱해 만든다.

---

## 수학적 형태

$$
K=XW_K
$$

$$
Attention\ weight=softmax\left(\frac{QK^T}{\sqrt{d_k}}\right)
$$

---

## 컴퓨팅 관점의 입력과 출력

Key tensor shape은 Query와 비슷하게 `[batch, heads, seq_len, d_head]`입니다. 각 token이 검색될 때 비교되는 색인 벡터입니다.

---

## 간단한 toy 예시

문서 데이터베이스의 index처럼, 각 token의 key는 query가 어느 token을 참고할지 결정하는 기준이 됩니다.

---

## 관련 노트

- [[Query]]
- [[Value]]
- [[Query-Key-Value]]
- [[Attention Mechanism]]

---

## 내가 기억해야 할 핵심

- Key는 Key는 attention에서 각 위치가 어떤 정보를 가지고 있는지 색인처럼 나타내는 벡터이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
