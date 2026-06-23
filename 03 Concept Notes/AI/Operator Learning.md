---
type: learning-note
topic: operator-learning
tags:
  - machine-learning
  - scientific-machine-learning
  - pde
created: 2026-06-23
status: draft
---

# Operator Learning

## 한 줄 요약

[[Operator Learning]]은 하나의 함수값 예측이 아니라, 입력 함수와 출력 함수 사이의 mapping 자체를 데이터로부터 학습하는 문제 설정이다.

---

## 일반 supervised learning과 비교

일반 supervised learning:

$$
\mathbf{x}_i \mapsto \mathbf{y}_i
$$

Operator learning:

$$
a_i(x) \mapsto u_i(x)
$$

여기서 $a_i(x)$와 $u_i(x)$는 scalar가 아니라 공간 또는 시간에 정의된 함수/field이다.

---

## PDE에서의 의미

PDE solver는 입력 조건을 받아 solution field를 만든다.

$$
\text{coefficient, IC, BC, source} \mapsto \text{solution field}
$$

이 mapping을 [[PDE Solution Operator]]라고 볼 수 있다. [[Fourier Neural Operator (FNO)]]는 이 operator를 학습하는 대표적인 neural operator 모델이다.

---

## 컴퓨팅 관점의 입력과 출력

Operator learning dataset은 보통 `(입력 field tensor, 출력 field tensor)` 쌍으로 저장됩니다.

```text
train_x.shape = [num_cases, Nx, Ny, input_channels]
train_y.shape = [num_cases, Nx, Ny, output_channels]
```

예를 들어 1000개의 CFD case가 있고 각 case가 `128×128` grid의 4개 입력 channel과 3개 출력 channel을 가진다면:

```text
train_x.shape = [1000, 128, 128, 4]
train_y.shape = [1000, 128, 128, 3]
```

---

## 간단한 toy 예시

입력 field가 `inlet velocity profile + wall heat flux distribution`이고 출력 field가 `temperature field`라고 해보겠습니다. 각 simulation case 하나가 하나의 training sample입니다. Operator learning은 이 sample들을 보고 새로운 boundary condition field가 들어왔을 때 전체 temperature field를 예측하도록 학습합니다.

---

## 관련 노트

- [[Neural Operator]]
- [[PDE Solution Operator]]
- [[Fourier Neural Operator (FNO)]]
- [[Surrogate Model]]
- [[Scientific Machine Learning]]
