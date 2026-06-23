---
type: learning-note
topic: transformer-block
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Transformer Block

## 한 줄 요약

[[Transformer Block]]은 self-attention, feed-forward network, residual connection, layer normalization으로 구성된 반복 단위이다.

---

## 핵심 개념

- 여러 Transformer block을 깊게 쌓으면 BERT, GPT 같은 모델이 된다.
- attention은 token 간 관계를 계산하고, FFN은 각 token feature를 변환한다.
- Residual connection과 LayerNorm은 학습 안정성을 높인다.

---

## 수학적 형태

Pre-LN 형태의 간단한 block은 다음과 같다.

$$
x'=x+MHA(LN(x))
$$

$$
y=x'+FFN(LN(x'))
$$

---

## 컴퓨팅 관점의 입력과 출력

입력은 `[batch, seq_len, d_model]`, 출력도 `[batch, seq_len, d_model]`입니다. 여러 block을 쌓아도 sequence length와 hidden dimension은 보통 유지됩니다.

---

## 간단한 toy 예시

문장 길이 5, d_model 16이면 block 하나는 `[1,5,16]` tensor를 받아 attention과 FFN을 거쳐 다시 `[1,5,16]`을 출력합니다.

---

## 관련 노트

- [[Transformer Model]]
- [[Self-Attention]]
- [[Feed-Forward Network]]
- [[Residual Connection]]
- [[Layer Normalization]]

---

## 내가 기억해야 할 핵심

- Transformer Block는 Transformer Block은 self-attention, feed-forward network, residual connection, layer normalization으로 구성된 반복 단위이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
