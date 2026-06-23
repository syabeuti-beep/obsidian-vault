---
type: learning-note
topic: self-attention
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Self-Attention

## 한 줄 요약

[[Self-Attention]]은 같은 sequence 안의 token들이 서로를 참고하도록 만드는 attention 연산이다.

---

## 핵심 개념

- 입력 sequence의 각 token이 자기 자신을 포함한 모든 token을 바라본다.
- 대명사 참조, 장거리 의존성, 문법적 관계를 학습하는 데 유리하다.
- [[Transformer Model]]의 encoder와 decoder block에서 핵심 연산으로 사용된다.

---

## 수학적 형태

입력 행렬 $X$에서 query, key, value를 만든다.

$$
Q=XW_Q,\quad K=XW_K,\quad V=XW_V
$$

그 다음 attention을 계산한다.

$$
Y=softmax\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

---

## 컴퓨팅 관점의 입력과 출력

입력은 하나의 sequence hidden tensor `X.shape=[batch, seq_len, d_model]`입니다. Q,K,V가 모두 같은 X에서 만들어지고 출력도 `[batch, seq_len, d_model]`입니다.

---

## 간단한 toy 예시

`it overheated`에서 `it` token이 앞의 `pump` token을 크게 참고하도록 attention weight가 학습될 수 있습니다.

---

## 관련 노트

- [[Attention Mechanism]]
- [[Query-Key-Value]]
- [[Multi-Head Attention]]
- [[Transformer Model]]

---

## 내가 기억해야 할 핵심

- Self-Attention는 Self-Attention은 같은 sequence 안의 token들이 서로를 참고하도록 만드는 attention 연산이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
