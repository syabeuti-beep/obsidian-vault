---
type: learning-note
topic: surrogate-model
tags:
  - scientific-machine-learning
  - numerical-simulation
  - reduced-cost-model
created: 2026-06-23
status: draft
---

# Surrogate Model

## 한 줄 요약

[[Surrogate Model]]은 계산 비용이 큰 고충실도 모델, 실험, 또는 수치해석 solver를 더 빠르게 근사하기 위해 만든 대체 모델이다.

---

## 왜 필요한가?

열수력 CFD, multiphysics simulation, uncertainty quantification, design optimization에서는 같은 종류의 계산을 수백~수만 번 반복해야 할 수 있다. 매번 고충실도 solver를 돌리면 비용이 너무 크다.

Surrogate model은 일부 고충실도 데이터로부터 다음 mapping을 학습한다.

$$
\text{input condition} \mapsto \text{quantity of interest or field}
$$

---

## 예시

- 입력: inlet velocity, pressure, heat flux
- 출력: pressure drop, peak temperature, CHF margin

또는 field-to-field 형태:

- 입력: boundary condition field, material property field
- 출력: temperature/velocity/pressure field

이 field-to-field surrogate에는 [[Fourier Neural Operator (FNO)]] 같은 [[Neural Operator]]가 잘 맞는다.

---

## 주의점

Surrogate model은 학습 데이터가 덮고 있는 조건 범위 안에서는 빠르고 유용하지만, 데이터 분포 밖 extrapolation에서는 크게 틀릴 수 있다. 따라서 validation, uncertainty estimate, physics consistency check가 중요하다.

---

## 컴퓨팅 관점의 입력과 출력

Surrogate model의 입력/출력은 문제에 따라 scalar vector일 수도 있고 field tensor일 수도 있습니다.

Scalar QoI surrogate:

```text
input.shape  = [batch, 10]  # operating conditions
output.shape = [batch, 2]   # pressure drop, peak temperature
```

Field surrogate:

```text
input.shape  = [batch, Nx, Ny, Cin]
output.shape = [batch, Nx, Ny, Cout]
```

즉 무엇을 대체하느냐에 따라 FCNN, CNN, FNO 같은 모델 선택이 달라집니다.

---

## 간단한 toy 예시

고충실도 CFD가 한 case당 10시간 걸린다고 해보겠습니다. 먼저 300개 case를 CFD로 계산해 dataset을 만듭니다. 그 다음 surrogate model을 학습해 새로운 조건에서 0.01초 안에 `pressure_drop`과 `peak_temperature`를 예측하게 만들 수 있습니다.

---

## 관련 노트

- [[Fourier Neural Operator (FNO)]]
- [[Neural Operator]]
- [[Scientific Machine Learning]]
- [[Uncertainty Quantification]]
- [[Reduced Order Model]]
