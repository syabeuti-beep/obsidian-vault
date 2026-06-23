---
type: learning-note
topic: encoder-decoder-architecture
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Encoder-Decoder Architecture

## 한 줄 요약

[[Encoder-Decoder Architecture]]는 입력을 encoder가 표현으로 압축하고 decoder가 그 표현을 참고해 출력을 생성하는 구조이다.

---

## 핵심 개념

- 기계번역처럼 입력 sequence와 출력 sequence가 모두 있는 문제에서 자연스럽다.
- 원래 Transformer 논문은 encoder-decoder 구조였다.
- encoder는 입력 전체를 이해하고, decoder는 이전 출력과 encoder 정보를 보며 다음 token을 생성한다.

---

## 수학적 형태

sequence-to-sequence mapping은 다음처럼 볼 수 있다.

$$
h=Encoder(x_1,\cdots,x_n)
$$

$$
y_t=Decoder(y_{<t},h)
$$

---

## 컴퓨팅 관점의 입력과 출력

encoder 입력은 source token IDs `[batch, src_len]`, decoder 입력은 target prefix IDs `[batch, tgt_len]`입니다. 출력은 target 위치별 vocabulary logits `[batch, tgt_len, vocab_size]`입니다.

---

## 간단한 toy 예시

영어 `The pump failed`를 한국어로 번역할 때 encoder가 영어 문장을 hidden states로 만들고 decoder가 `펌프가 고장났다`를 한 token씩 생성합니다.

---

## 관련 노트

- [[Transformer Encoder]]
- [[Transformer Decoder]]
- [[Transformer Model]]
- [[T5]]

---

## 내가 기억해야 할 핵심

- Encoder-Decoder Architecture는 Encoder-Decoder Architecture는 입력을 encoder가 표현으로 압축하고 decoder가 그 표현을 참고해 출력을 생성하는 구조이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
