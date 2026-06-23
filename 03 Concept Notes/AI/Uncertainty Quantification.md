---
type: learning-note
topic: uncertainty-quantification
tags:
  - uncertainty
  - scientific-computing
created: 2026-06-23
status: draft
---
# Uncertainty Quantification

## 한 줄 요약

[[Uncertainty Quantification]]은 입력, 모델, 수치해석, 실험의 불확실성이 결과에 어떻게 전파되는지 정량화하는 분야이다.

---

## 핵심 개념

- 입력 조건이 확률분포를 가지면 출력도 분포를 가진다.
- Monte Carlo, polynomial chaos, Bayesian methods, surrogate model이 사용된다.
- 공학 안전 해석에서는 평균뿐 아니라 신뢰구간, worst-case, exceedance probability가 중요하다.

---

## 수학적 형태

입력 $X$가 random variable이고 모델이 $Y=f(X)$라면 UQ는 $Y$의 분포, 평균, 분산 등을 추정한다.

$$
\mathbb{E}[Y],\quad Var[Y],\quad P(Y>y_{crit})
$$

---

## 열수력/HPC 연구와의 연결

열수력 안전 해석에서는 power, boundary condition, closure model uncertainty가 peak temperature, pressure drop, CHF margin에 미치는 영향을 평가할 수 있다.

---

## 컴퓨팅 관점의 입력과 출력

입력은 uncertain parameter distribution과 model이고 출력은 quantity of interest의 분포, 평균, 분산, 신뢰구간입니다.

---

## 간단한 toy 예시

inlet temperature를 정규분포로 샘플링해 1000번 surrogate를 실행하고 peak temperature의 95% 구간을 계산합니다.

---

## 관련 노트

- [[Surrogate Model]]
- [[Fourier Neural Operator (FNO)]]
- [[Scientific Machine Learning]]
- [[High Performance Computing]]
- [[Quant Trading]]
- [[Backtesting]]

---

## 내가 기억해야 할 핵심

- Uncertainty Quantification는 Uncertainty Quantification은 입력, 모델, 수치해석, 실험의 불확실성이 결과에 어떻게 전파되는지 정량화하는 분야이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
