---
type: learning-note
topic: mixture-of-experts
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Mixture of Experts

## 한 줄 요약

[[Mixture of Experts]]는 여러 expert subnetwork 중 일부만 선택적으로 활성화해 계산량 대비 모델 용량을 키우는 구조이다.

---

## 핵심 개념

- router가 token마다 사용할 expert를 선택한다.
- 모든 파라미터를 매 token마다 쓰지 않으므로 sparse activation 구조이다.
- LLM에서는 주로 [[Feed-Forward Network]] 부분을 expert들로 나눈다.

---

## 수학적 형태

router가 expert 확률을 만든 뒤 top-k expert를 선택한다.

$$
g(x)=softmax(W_rx)
$$

$$
y=\sum_{i\in TopK(g)} g_i(x)E_i(x)
$$

---

## 컴퓨팅 관점의 입력과 출력

입력 token hidden tensor `[batch, seq_len, d_model]`에서 router가 expert index와 weight를 고릅니다. 선택된 expert FFN만 계산하고 출력 shape은 다시 `[batch, seq_len, d_model]`입니다.

---

## 간단한 toy 예시

token `temperature`는 과학 expert로, token `therefore`는 일반 언어 expert로 보내는 식의 routing이 일어날 수 있습니다.

---

## 관련 노트

- [[Feed-Forward Network]]
- [[Transformer Model]]
- [[Large Language Model]]

---

## 내가 기억해야 할 핵심

- Mixture of Experts는 Mixture of Experts는 여러 expert subnetwork 중 일부만 선택적으로 활성화해 계산량 대비 모델 용량을 키우는 구조이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
