---
type: learning-note
topic: positional-encoding
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Positional Encoding

## 한 줄 요약

[[Positional Encoding]]은 순서를 직접 알 수 없는 Transformer 입력에 token 위치 정보를 추가하는 방법이다.

---

## 핵심 개념

- [[Transformer Model]]은 token들을 병렬로 처리하기 때문에 순서 정보를 별도로 넣어야 한다.
- 초기 논문에서는 sin/cos 기반 고정 위치 인코딩을 사용했다.
- 현대 LLM에서는 학습형 position embedding, [[Rotary Positional Embedding]], ALiBi 등이 쓰인다.

---

## 수학적 형태

입력 표현은 보통 다음처럼 만든다.

$$
h_i = token\_embedding_i + positional\_encoding_i
$$

sin/cos 방식의 예:

$$
PE_{pos,2i}=\sin(pos/10000^{2i/d})
$$

$$
PE_{pos,2i+1}=\cos(pos/10000^{2i/d})
$$

---

## 컴퓨팅 관점의 입력과 출력

입력 token embedding shape이 `[batch, seq_len, d_model]`이면 positional encoding도 broadcast 가능한 `[seq_len, d_model]` 또는 `[batch, seq_len, d_model]`입니다. 더한 뒤 shape은 변하지 않습니다.

---

## 간단한 toy 예시

`reactor is stable`과 `stable is reactor`는 같은 token set을 가질 수 있지만 위치 encoding이 다르므로 Transformer가 순서를 구분할 수 있습니다.

---

## 관련 노트

- [[Transformer Model]]
- [[Rotary Positional Embedding]]
- [[Token Embedding]]

---

## 내가 기억해야 할 핵심

- Positional Encoding는 Positional Encoding은 순서를 직접 알 수 없는 Transformer 입력에 token 위치 정보를 추가하는 방법이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
