---
type: learning-note
topic: rnn
aliases:
  - Recurrent Neural Network
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# RNN

## 한 줄 요약

[[RNN]]은 sequence를 시간 순서대로 처리하면서 hidden state를 업데이트하는 recurrent neural network이다.

---

## 핵심 개념

- 이전 시점의 hidden state가 다음 시점 계산에 들어간다.
- 순차 처리 구조라 긴 sequence 병렬화가 어렵다.
- 장기 의존성 문제를 완화하기 위해 [[LSTM]] 같은 변형이 제안되었다.

---

## 수학적 형태

기본 RNN은 다음처럼 쓴다.

$$
h_t=\sigma(W_xx_t+W_hh_{t-1}+b)
$$

$$
y_t=W_yh_t
$$

---

## 컴퓨팅 관점의 입력과 출력

입력은 sequence tensor `[batch, seq_len, input_dim]`이고 hidden state는 `[batch, hidden_dim]`입니다. 출력은 마지막 hidden state 또는 모든 time step의 hidden states입니다.

---

## 간단한 toy 예시

10초 길이 센서 데이터에서 매초 3개 변수만 있으면 입력 shape은 `[batch,10,3]`이고 RNN은 시간 순서대로 하나씩 처리합니다.

---

## 관련 노트

- [[Recurrent Neural Network]]
- [[LSTM]]
- [[Transformer Model]]

---

## 내가 기억해야 할 핵심

- RNN는 RNN은 sequence를 시간 순서대로 처리하면서 hidden state를 업데이트하는 recurrent neural network이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
