---
type: learning-note
topic: reduced-order-model
tags:
  - reduced-order-model
  - scientific-computing
created: 2026-06-23
status: draft
---
# Reduced Order Model

## 한 줄 요약

[[Reduced Order Model]]은 고차원 물리 시스템을 적은 수의 중요한 mode나 latent variable로 축약해 빠르게 계산하는 모델이다.

---

## 핵심 개념

- 대표적으로 POD, Galerkin projection, autoencoder 기반 ROM이 있다.
- 고충실도 solver보다 빠른 예측을 목표로 한다.
- FNO 같은 neural operator와 마찬가지로 surrogate modeling의 한 축이다.

---

## 수학적 형태

field $u(x,t)$를 basis mode의 합으로 근사할 수 있다.

$$
u(x,t)\approx \sum_{i=1}^r a_i(t)\phi_i(x)
$$

여기서 $r$은 원래 자유도보다 훨씬 작다.

---

## 열수력/HPC 연구와의 연결

CFD field를 적은 POD mode로 축약하면 parametric study나 control에서 계산 비용을 줄일 수 있다.

---

## 컴퓨팅 관점의 입력과 출력

입력은 parameter 또는 시간이고 출력은 적은 수의 modal coefficient입니다. coefficient를 basis와 곱해 full field를 복원합니다.

---

## 간단한 toy 예시

온도장 snapshot 1000개에서 POD mode 10개를 뽑고, 새 조건에서는 10개 coefficient만 예측해 전체 온도장을 재구성합니다.

---

## 관련 노트

- [[Surrogate Model]]
- [[Scientific Machine Learning]]
- [[Fourier Neural Operator (FNO)]]

---

## 내가 기억해야 할 핵심

- Reduced Order Model는 Reduced Order Model은 고차원 물리 시스템을 적은 수의 중요한 mode나 latent variable로 축약해 빠르게 계산하는 모델이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
