---
type: learning-note
topic: layer-normalization
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Layer Normalization

## 한 줄 요약

[[Layer Normalization]]은 한 sample의 feature 차원에 대해 평균과 분산을 정규화해 학습을 안정화하는 방법이다.

---

## 핵심 개념

- batch 크기에 크게 의존하지 않아 sequence model에서 많이 쓰인다.
- Transformer에서는 attention과 FFN 주변에 LayerNorm을 배치한다.
- 최근 LLM은 보통 Pre-LN 구조를 사용한다.

---

## 수학적 형태

feature vector $x$에 대해

$$
\mu=\frac{1}{d}\sum_{i=1}^d x_i,\quad \sigma^2=\frac{1}{d}\sum_{i=1}^d (x_i-\mu)^2
$$

$$
LayerNorm(x)=\gamma\frac{x-\mu}{\sqrt{\sigma^2+\epsilon}}+\beta
$$

---

## 컴퓨팅 관점의 입력과 출력

입력 tensor shape은 유지됩니다. 예를 들어 `[batch, seq_len, d_model]`에서 마지막 dimension `d_model`에 대해 평균/분산을 계산합니다.

---

## 간단한 toy 예시

한 token vector `[2,4,6]`이 있으면 평균 4를 빼고 표준편차로 나누어 scale을 안정화한 뒤 learnable scale/bias를 적용합니다.

---

## 관련 노트

- [[Transformer Block]]
- [[Residual Connection]]
- [[Transformer Model]]

---

## 내가 기억해야 할 핵심

- Layer Normalization는 Layer Normalization은 한 sample의 feature 차원에 대해 평균과 분산을 정규화해 학습을 안정화하는 방법이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
