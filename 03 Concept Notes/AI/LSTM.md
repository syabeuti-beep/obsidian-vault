---
type: learning-note
topic: lstm
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# LSTM

## 한 줄 요약

[[LSTM]]은 RNN의 장기 의존성 학습 문제를 완화하기 위해 cell state와 gate를 도입한 recurrent network이다.

---

## 핵심 개념

- forget/input/output gate로 정보를 얼마나 유지하고 버릴지 조절한다.
- 일반 RNN보다 긴 sequence 정보를 더 잘 보존한다.
- 하지만 순차 처리 구조라 Transformer보다 병렬화가 어렵다.

---

## 수학적 형태

핵심은 cell state $c_t$를 gate로 조절하는 것이다.

$$
f_t=\sigma(W_f[h_{t-1},x_t]+b_f)
$$

$$
c_t=f_t\odot c_{t-1}+i_t\odot \tilde{c}_t
$$

---

## 컴퓨팅 관점의 입력과 출력

입력은 `[batch, seq_len, input_dim]`이고 내부 state는 hidden state `h_t`와 cell state `c_t` 두 개입니다. 출력은 sequence 전체 또는 마지막 hidden state입니다.

---

## 간단한 toy 예시

유량 시계열 100 step을 입력해 다음 step의 압력 변동을 예측할 때 LSTM은 오래된 정보를 cell state에 일부 보존합니다.

---

## 관련 노트

- [[RNN]]
- [[Recurrent Neural Network]]
- [[Transformer Model]]

---

## 내가 기억해야 할 핵심

- LSTM는 LSTM은 RNN의 장기 의존성 학습 문제를 완화하기 위해 cell state와 gate를 도입한 recurrent network이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
