---
type: learning-note
topic: residual-connection
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Residual Connection

## 한 줄 요약

[[Residual Connection]]은 layer의 출력에 입력을 더해 깊은 network의 gradient 흐름을 돕는 구조이다.

---

## 핵심 개념

- 입력을 완전히 새로 변환하는 대신 변화량을 학습하게 만든다.
- deep network에서 vanishing gradient 문제를 완화한다.
- Transformer block에서는 attention과 FFN 각각에 residual connection이 붙는다.

---

## 수학적 형태

기본 형태는 다음과 같다.

$$
y=x+F(x)
$$

Pre-LN Transformer에서는 대략 다음처럼 쓴다.

$$
x=x+Attention(LayerNorm(x))
$$

$$
x=x+FFN(LayerNorm(x))
$$

---

## 컴퓨팅 관점의 입력과 출력

입력과 함수 출력의 shape이 같아야 더할 수 있습니다. 예: `x.shape=[batch, seq_len, d_model]`, `F(x).shape=[batch, seq_len, d_model]`이면 `y=x+F(x)`입니다.

---

## 간단한 toy 예시

layer가 당장 좋은 변환을 못 배워도 residual 덕분에 최소한 입력 x를 그대로 다음 layer로 전달할 수 있습니다.

---

## 관련 노트

- [[Transformer Block]]
- [[Layer Normalization]]
- [[Transformer Model]]

---

## 내가 기억해야 할 핵심

- Residual Connection는 Residual Connection은 layer의 출력에 입력을 더해 깊은 network의 gradient 흐름을 돕는 구조이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
