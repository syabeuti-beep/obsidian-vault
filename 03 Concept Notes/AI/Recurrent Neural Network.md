---
type: learning-note
topic: recurrent-neural-network
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Recurrent Neural Network

## 한 줄 요약

[[Recurrent Neural Network]]는 sequence 데이터를 시간 순서대로 처리하며 hidden state를 재사용하는 neural network이다.

---

## 핵심 개념

- [[RNN]]과 같은 개념을 풀어 쓴 이름이다.
- 문장, 음성, 시계열처럼 순서가 중요한 데이터에 사용되었다.
- 현대 NLP에서는 병렬화와 긴 문맥 처리 장점 때문에 [[Transformer Model]]이 많이 대체했다.

---

## 수학적 형태

$$
h_t=f_\theta(x_t,h_{t-1})
$$

현재 상태는 현재 입력과 이전 상태의 함수이다.

---

## 컴퓨팅 관점의 입력과 출력

코드에서는 time dimension을 가진 tensor를 넣습니다. 예: `[batch, time_steps, features] → [batch, hidden_dim]` 또는 `[batch, time_steps, hidden_dim]`입니다.

---

## 간단한 toy 예시

온도 센서 5개의 60초 기록을 넣어 다음 1초 뒤 온도를 예측하는 모델을 만들 수 있습니다.

---

## 관련 노트

- [[RNN]]
- [[LSTM]]
- [[Transformer Model]]

---

## 내가 기억해야 할 핵심

- Recurrent Neural Network는 Recurrent Neural Network는 sequence 데이터를 시간 순서대로 처리하며 hidden state를 재사용하는 neural network이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
