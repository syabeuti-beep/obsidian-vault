---
type: learning-note
topic: query-key-value
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Query-Key-Value

## 한 줄 요약

[[Query-Key-Value]]는 attention을 계산하기 위해 token 표현을 세 역할의 벡터로 나누는 구조이다.

---

## 핵심 개념

- Query: 내가 찾는 정보.
- Key: 내가 가진 정보의 색인.
- Value: 실제로 전달할 정보.
- 이 구조 덕분에 attention은 content-based retrieval처럼 동작한다.

---

## 수학적 형태

$$
Q=XW_Q,\quad K=XW_K,\quad V=XW_V
$$

$$
Attention(Q,K,V)=softmax\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

---

## 컴퓨팅 관점의 입력과 출력

입력 hidden state `X.shape=[batch, seq_len, d_model]`에서 세 linear projection으로 Q,K,V를 만듭니다. 출력은 attention weight와 V의 곱으로 다시 sequence tensor가 됩니다.

---

## 간단한 toy 예시

도서관 비유로 보면 Query는 질문, Key는 책의 색인, Value는 책의 실제 내용입니다.

---

## 관련 노트

- [[Query]]
- [[Key]]
- [[Value]]
- [[Attention Mechanism]]
- [[Self-Attention]]

---

## 내가 기억해야 할 핵심

- Query-Key-Value는 Query-Key-Value는 attention을 계산하기 위해 token 표현을 세 역할의 벡터로 나누는 구조이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
