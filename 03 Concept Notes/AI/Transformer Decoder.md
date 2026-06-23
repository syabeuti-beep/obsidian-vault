---
type: learning-note
topic: transformer-decoder
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Transformer Decoder

## 한 줄 요약

[[Transformer Decoder]]는 이전 token들만 보고 다음 token을 생성하도록 설계된 Transformer 구성 요소이다.

---

## 핵심 개념

- 미래 token을 보지 못하게 [[Causal Mask]]를 사용한다.
- GPT 계열 모델은 decoder-only Transformer이다.
- 원래 encoder-decoder Transformer에서는 decoder가 encoder output에도 cross-attention한다.

---

## 수학적 형태

decoder-only language modeling은 다음 확률을 학습한다.

$$
P(x_t|x_{<t})
$$

causal mask는 attention score에서 미래 위치를 $-\infty$로 만들어 softmax 후 weight가 0이 되게 한다.

---

## 컴퓨팅 관점의 입력과 출력

입력은 지금까지 생성된 token IDs `[batch, seq_len]`이고 출력은 다음 token 예측 logits `[batch, seq_len, vocab_size]`입니다.

---

## 간단한 toy 예시

prefix `The pump`가 들어오면 decoder는 다음 token으로 `failed`, `is`, `was` 등에 대한 확률을 출력합니다.

---

## 관련 노트

- [[Transformer Model]]
- [[Causal Mask]]
- [[GPT]]
- [[Large Language Model]]

---

## 내가 기억해야 할 핵심

- Transformer Decoder는 Transformer Decoder는 이전 token들만 보고 다음 token을 생성하도록 설계된 Transformer 구성 요소이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
