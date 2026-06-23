---
type: learning-note
topic: neural-operator
tags:
  - machine-learning
  - scientific-machine-learning
  - operator-learning
created: 2026-06-23
status: draft
---

# Neural Operator

## 한 줄 요약

[[Neural Operator]]는 숫자 벡터 하나를 다른 벡터로 보내는 함수가 아니라, 함수 전체를 입력받아 함수 전체를 출력하는 operator를 neural network로 근사하는 방법이다.

---

## 함수와 operator의 차이

일반적인 [[Fully Connected Neural Network]]는 다음 함수를 학습한다.

$$
f_\theta: \mathbb{R}^d \rightarrow \mathbb{R}^m
$$

반면 operator는 함수 공간 사이의 mapping이다.

$$
\mathcal{G}: \mathcal{A} \rightarrow \mathcal{U}
$$

예를 들어 PDE에서는 다음 mapping을 생각할 수 있다.

$$
a(x) \mapsto u(x)
$$

여기서:

- $a(x)$: 공간에 따라 변하는 계수, 초기조건, 경계조건, source term
- $u(x)$: PDE solution field

즉, 입력도 함수이고 출력도 함수이다.

---

## PDE solution operator

PDE 문제를 추상적으로 쓰면 다음과 같다.

$$
\mathcal{L}_a u = f
$$

여기서 $a$는 PDE 계수 또는 조건이다. PDE를 푸는 것은 $a$가 주어졌을 때 $u$를 찾는 것이다.

따라서 PDE solver는 다음 operator처럼 볼 수 있다.

$$
\mathcal{G}^\dagger(a)=u
$$

[[Neural Operator]]는 이 실제 solution operator $\mathcal{G}^\dagger$를 데이터로부터 학습한다.

---

## 왜 일반 neural network와 다른가?

Grid가 고정되어 있으면 field를 긴 벡터로 펼쳐서 FCNN에 넣을 수도 있다.

```text
u(x_1), u(x_2), ..., u(x_N) → giant vector
```

하지만 이 방식은 다음 문제가 있다.

1. grid resolution이 바뀌면 입력 차원이 바뀐다.
2. local/global spatial structure를 잘 이용하지 못한다.
3. 고해상도 field에서는 파라미터 수가 너무 커진다.
4. PDE solution의 연속적인 함수 구조를 직접 반영하지 못한다.

Neural operator는 grid point의 개별 값보다 함수 사이의 mapping을 배우는 것을 목표로 한다.

---

## 일반적 layer 형태

많은 neural operator는 다음 형태의 반복 layer를 가진다.

$$
v_{t+1}(x)=\sigma\left(Wv_t(x)+\int_D \kappa_\theta(x,y)v_t(y)dy\right)
$$

여기서:

- $v_t(x)$: layer $t$에서의 hidden field
- $Wv_t(x)$: pointwise linear transform
- $\kappa_\theta(x,y)$: 위치 $y$의 정보가 위치 $x$에 영향을 주는 kernel
- 적분항: domain 전체 정보를 모으는 nonlocal interaction
- $\sigma$: [[Activation Function]]

[[Fourier Neural Operator (FNO)]]는 이 적분 kernel을 Fourier space에서 효율적으로 표현한다.

---

## 컴퓨팅 관점의 입력과 출력

Neural operator도 코드에서는 결국 tensor를 입력받습니다. 차이는 이 tensor가 단순 feature vector가 아니라 grid 위의 field라는 점입니다.

예를 들어 2D coefficient field `a(x,y)`를 입력받아 solution field `u(x,y)`를 출력한다면:

```text
input_a.shape  = [batch, Nx, Ny, input_channels]
output_u.shape = [batch, Nx, Ny, output_channels]
```

예: `[8, 64, 64, 3] → [8, 64, 64, 1]`. 여기서 3개 입력 channel은 permeability, source term, boundary mask일 수 있고, 출력 1개 channel은 pressure field일 수 있습니다.

---

## 간단한 toy 예시

Poisson equation surrogate를 생각해 보겠습니다.

입력:

```text
a[i,j] = grid cell (i,j)의 material coefficient
shape = [64, 64, 1]
```

출력:

```text
u[i,j] = 같은 grid cell에서의 solution value
shape = [64, 64, 1]
```

Neural operator는 단일 점의 값을 예측하는 것이 아니라 전체 field 배열을 한 번에 예측합니다.

---

## 관련 노트

- [[Fourier Neural Operator (FNO)]]
- [[Operator Learning]]
- [[PDE Solution Operator]]
- [[Scientific Machine Learning]]
- [[Surrogate Model]]
