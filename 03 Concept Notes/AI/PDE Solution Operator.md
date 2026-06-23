---
type: learning-note
topic: pde-solution-operator
tags:
  - pde
  - numerical-method
  - scientific-machine-learning
created: 2026-06-23
status: draft
---

# PDE Solution Operator

## 한 줄 요약

[[PDE Solution Operator]]는 PDE의 계수, 초기조건, 경계조건, source term 같은 입력 함수를 받아 solution field를 출력하는 mapping이다.

---

## 기본 예시

PDE를 추상적으로 다음처럼 쓰자.

$$
\mathcal{L}_a u = f
$$

여기서 $a$가 PDE 계수 또는 조건이고, $u$가 우리가 구하고 싶은 solution이다. 그러면 solver가 하는 일은 다음 operator로 볼 수 있다.

$$
\mathcal{G}^\dagger: a \mapsto u
$$

---

## 왜 operator인가?

$a$와 $u$는 보통 숫자 하나가 아니라 공간/시간에 정의된 함수이다.

예:

- $a(x)$: 공간적으로 변하는 material property
- $u(x)$: pressure 또는 temperature field
- $u(x,t)$: time-dependent solution field

따라서 PDE solver는 finite-dimensional vector function이라기보다 function-to-function mapping으로 해석할 수 있다.

---

## FNO와의 연결

[[Fourier Neural Operator (FNO)]]는 데이터셋

$$
\{(a_i,u_i)\}_{i=1}^n
$$

으로부터 다음 근사를 학습한다.

$$
\mathcal{G}_\theta(a) \approx \mathcal{G}^\dagger(a)
$$

즉, PDE를 매번 반복적으로 풀지 않고 빠르게 solution field를 예측하려는 surrogate이다.

---

## 컴퓨팅 관점의 입력과 출력

PDE solution operator는 수학적으로는 함수에서 함수로 가는 mapping이지만, 계산에서는 discretized array로 표현됩니다.

```text
input:  coefficients/IC/BC/source arrays
output: solution arrays
```

예를 들어 2D steady heat conduction이면:

```text
input.shape  = [Nx, Ny, 2]  # thermal conductivity field, heat source field
output.shape = [Nx, Ny, 1]  # temperature field
```

batch 학습에서는 앞에 batch dimension이 붙습니다.

---

## 간단한 toy 예시

`32×32` 격자에서 열전도 문제를 생각해 보겠습니다.

입력 channel 2개:

```text
k[i,j] = thermal conductivity
q[i,j] = volumetric heat source
```

출력 channel 1개:

```text
T[i,j] = steady-state temperature
```

PDE solver는 이 mapping을 수치적으로 풀고, FNO는 이 mapping을 데이터로 학습합니다.

---

## 관련 노트

- [[Neural Operator]]
- [[Operator Learning]]
- [[Fourier Neural Operator (FNO)]]
- [[Surrogate Model]]
- [[Scientific Machine Learning]]
