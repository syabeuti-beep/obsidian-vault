---
type: learning-note
topic: rotary-positional-embedding
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Rotary Positional Embedding

## 한 줄 요약

[[Rotary Positional Embedding]] 또는 RoPE는 query와 key 벡터를 위치에 따라 회전시켜 상대적 위치 정보를 attention에 넣는 방법이다.

---

## 핵심 개념

- 절대 위치 벡터를 더하는 대신, Q/K 공간에서 위치별 회전을 적용한다.
- 상대적 위치 차이가 attention score에 자연스럽게 반영된다.
- 많은 decoder-only LLM에서 긴 context 처리에 사용된다.

---

## 수학적 형태

개념적으로 위치 $m$의 벡터에 회전 행렬 $R_m$을 적용한다.

$$
q_m' = R_m q_m, \quad k_n'=R_n k_n
$$

그러면 내적 $q_m'^T k_n'$에 위치 차이 $(m-n)$ 정보가 들어간다.

---

## 컴퓨팅 관점의 입력과 출력

RoPE는 Q,K tensor의 마지막 feature dimension을 2차원 쌍으로 나누어 위치별 회전을 적용합니다. 입력/출력 shape은 그대로 `[batch, heads, seq_len, d_head]`입니다.

---

## 간단한 toy 예시

위치 10의 query와 위치 7의 key를 비교할 때, 두 벡터의 회전 차이에 상대 거리 3 정보가 들어갑니다.

---

## 관련 노트

- [[Positional Encoding]]
- [[Attention Mechanism]]
- [[Transformer Model]]
- [[Large Language Model]]

---

## 내가 기억해야 할 핵심

- Rotary Positional Embedding는 Rotary Positional Embedding 또는 RoPE는 query와 key 벡터를 위치에 따라 회전시켜 상대적 위치 정보를 attention에 넣는 방법이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
