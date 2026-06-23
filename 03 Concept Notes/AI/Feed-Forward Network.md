---
type: learning-note
topic: feed-forward-network
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Feed-Forward Network

## 한 줄 요약

[[Feed-Forward Network]]는 Transformer block 안에서 각 token 위치에 독립적으로 적용되는 작은 MLP이다.

---

## 핵심 개념

- attention이 token 사이 정보를 섞는다면 FFN은 각 token의 feature를 비선형적으로 변환한다.
- 보통 hidden dimension을 크게 확장했다가 다시 줄인다.
- LLM 파라미터의 큰 부분이 FFN에 들어간다.

---

## 수학적 형태

Transformer FFN의 기본 형태는 다음과 같다.

$$
FFN(x)=W_2\sigma(W_1x+b_1)+b_2
$$

여기서 $\sigma$는 ReLU, GELU, SwiGLU 같은 activation이다.

---

## 컴퓨팅 관점의 입력과 출력

입력과 출력은 `[batch, seq_len, d_model]`입니다. 내부에서는 각 token마다 `d_model → d_ff → d_model` MLP를 독립적으로 적용합니다.

---

## 간단한 toy 예시

d_model=512, d_ff=2048이면 각 token vector가 512차원에서 2048차원으로 확장된 뒤 다시 512차원으로 줄어듭니다.

---

## 관련 노트

- [[Transformer Block]]
- [[Activation Function]]
- [[Mixture of Experts]]
- [[Transformer Model]]

---

## 내가 기억해야 할 핵심

- Feed-Forward Network는 Feed-Forward Network는 Transformer block 안에서 각 token 위치에 독립적으로 적용되는 작은 MLP이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
