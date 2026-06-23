---
type: learning-note
topic: physics-informed-neural-network
tags:
  - scientific-machine-learning
  - pde
created: 2026-06-23
status: draft
---
# Physics-Informed Neural Network

## 한 줄 요약

[[Physics-Informed Neural Network]]는 데이터 loss와 함께 PDE residual, 초기조건, 경계조건 residual을 loss에 넣어 학습하는 neural network 방법이다.

---

## 핵심 개념

- PINN은 solution function 자체를 neural network로 근사하는 경우가 많다.
- 데이터가 적어도 물리 방정식을 soft constraint로 활용할 수 있다.
- stiff PDE, 난류, 복잡 geometry, multi-scale 문제에서는 학습이 어려울 수 있다.

---

## 수학적 형태

PDE가 $\mathcal{N}[u](x)=0$이면 PINN loss는 다음처럼 구성된다.

$$
\mathcal{L}=\mathcal{L}_{data}+\lambda_r\|\mathcal{N}[u_\theta]\|^2+\lambda_b\mathcal{L}_{BC}+\lambda_i\mathcal{L}_{IC}
$$

---

## 열수력/HPC 연구와의 연결

열수력 문제에서는 conservation law residual이나 boundary condition을 loss에 넣을 수 있지만, 강한 비선형성과 다상유동 closure 때문에 안정적 학습이 쉽지 않을 수 있다.

---

## 컴퓨팅 관점의 입력과 출력

입력은 좌표 tensor `[num_points, dim]`와 조건이고 출력은 해당 좌표에서의 solution value입니다. loss 계산 때 자동미분으로 PDE residual을 구합니다.

---

## 간단한 toy 예시

1D heat equation에서 좌표 `(x,t)`를 넣으면 `T(x,t)`를 출력하고, autograd로 `T_t - alpha T_xx`를 계산해 0에 가깝게 만듭니다.

---

## 관련 노트

- [[Scientific Machine Learning]]
- [[Fourier Neural Operator (FNO)]]
- [[Surrogate Model]]

---

## 내가 기억해야 할 핵심

- Physics-Informed Neural Network는 Physics-Informed Neural Network는 데이터 loss와 함께 PDE residual, 초기조건, 경계조건 residual을 loss에 넣어 학습하는 neural network 방법이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
