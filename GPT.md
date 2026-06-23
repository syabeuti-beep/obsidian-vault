---
type: learning-note
topic: gpt
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# GPT

## 한 줄 요약

[[GPT]]는 decoder-only Transformer로 이전 token들을 보고 다음 token을 예측하도록 학습된 generative language model 계열이다.

---

## 핵심 개념

- autoregressive language modeling을 사용한다.
- [[Causal Mask]] 덕분에 미래 token을 보지 않는다.
- 대규모로 확장하면 범용 [[Large Language Model]]의 기반이 된다.

---

## 수학적 형태

GPT의 기본 학습 목표는 다음 token likelihood를 최대화하는 것이다.

$$
\max_\theta \sum_t \log P_\theta(x_t|x_{<t})
$$

---

## 컴퓨팅 관점의 입력과 출력

입력은 prefix token IDs `[batch, seq_len]`, 출력은 각 위치의 다음 token logits `[batch, seq_len, vocab_size]`입니다. 생성 시에는 마지막 위치 logits에서 다음 token을 샘플링합니다.

---

## 간단한 toy 예시

입력 `The reactor is`가 들어오면 모델이 `stable`의 확률을 높게 줄 수 있고, 이 token을 붙여 다음 step을 반복합니다.

---

## 관련 노트

- [[Transformer Decoder]]
- [[Causal Mask]]
- [[Large Language Model]]
- [[Transformer Model]]

---

## 내가 기억해야 할 핵심

- GPT는 GPT는 decoder-only Transformer로 이전 token들을 보고 다음 token을 예측하도록 학습된 generative language model 계열이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
