---
type: learning-note
topic: causal-mask
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Causal Mask

## 한 줄 요약

[[Causal Mask]]는 autoregressive generation에서 현재 token이 미래 token을 보지 못하게 막는 attention mask이다.

---

## 핵심 개념

- GPT 같은 decoder-only 모델은 다음 token 예측을 학습한다.
- 학습 중 정답 sequence 전체가 입력되어도 미래 정보를 보지 못해야 한다.
- causal mask는 attention matrix의 upper triangular 부분을 차단한다.

---

## 수학적 형태

sequence 길이가 $n$일 때 위치 $i$는 $j\le i$인 위치만 볼 수 있다.

$$
M_{ij}=\begin{cases}0, & j\le i\\ -\infty, & j>i\end{cases}
$$

attention score에 $M$을 더한 뒤 softmax를 적용한다.

---

## 컴퓨팅 관점의 입력과 출력

입력은 attention score matrix `[batch, heads, seq_len, seq_len]`에 더해지는 mask입니다. 출력 tensor shape은 바꾸지 않고 미래 위치의 attention weight만 0이 되게 만듭니다.

---

## 간단한 toy 예시

길이 4 sequence에서 2번째 token은 3번째, 4번째 token을 볼 수 없습니다. 그래서 답을 미리 훔쳐보지 못합니다.

---

## 관련 노트

- [[Transformer Decoder]]
- [[GPT]]
- [[Large Language Model]]
- [[Attention Mechanism]]

---

## 내가 기억해야 할 핵심

- Causal Mask는 Causal Mask는 autoregressive generation에서 현재 token이 미래 token을 보지 못하게 막는 attention mask이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
