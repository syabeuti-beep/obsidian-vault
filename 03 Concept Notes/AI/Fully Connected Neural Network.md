---
type: learning-note
topic: fully-connected-neural-network
tags:
  - machine-learning
  - neural-network
  - math
created: 2026-06-23
status: draft
---

# Fully Connected Neural Network

## 한 줄 요약

[[Fully Connected Neural Network]]는 입력 벡터에 대해 반복적으로 "선형 변환 + bias + [[Activation Function]]"을 적용하는 함수 근사 모델이다.

공학적으로는 다음과 같은 계산 블록을 여러 층 쌓은 것이다.

$$
\mathbf{x}_{\ell+1} = \sigma(W_\ell \mathbf{x}_\ell + \mathbf{b}_\ell)
$$

여기서 $W_\ell$은 행렬, $\mathbf{b}_\ell$은 bias vector, $\sigma$는 비선형 활성화 함수이다.

---

## 행렬 계산으로 보는 neural network

입력 벡터가 $\mathbf{x} \in \mathbb{R}^d$라고 하자. 한 층은 다음처럼 계산된다.

$$
\mathbf{z} = W\mathbf{x} + \mathbf{b}
$$

$$
\mathbf{y} = \sigma(\mathbf{z})
$$

차원은 다음과 같다.

$$
W \in \mathbb{R}^{m \times d}, \quad \mathbf{x} \in \mathbb{R}^d, \quad \mathbf{b} \in \mathbb{R}^m, \quad \mathbf{y} \in \mathbb{R}^m
$$

즉, 한 층은 $d$차원 입력을 $m$차원 feature로 바꾸는 함수이다.

---

## 왜 activation function이 필요한가?

만약 모든 층이 선형이면,

$$
\mathbf{x}_2 = W_2(W_1\mathbf{x}_0 + \mathbf{b}_1) + \mathbf{b}_2
$$

이는 다시 하나의 선형 변환으로 합쳐진다.

$$
\mathbf{x}_2 = (W_2W_1)\mathbf{x}_0 + (W_2\mathbf{b}_1 + \mathbf{b}_2)
$$

따라서 아무리 깊게 쌓아도 결국 하나의 선형 모델과 같다. [[Activation Function]]은 이 선형성의 한계를 깨서 복잡한 비선형 함수를 근사하게 만든다.

---

## 함수 근사 관점

Fully connected network는 다음과 같은 형태의 함수 $f_\theta$를 학습한다.

$$
f_\theta: \mathbb{R}^d \rightarrow \mathbb{R}^m
$$

예를 들어 입력이 온도, 압력, 유속 같은 몇 개의 변수이고 출력이 CHF, 압력 강하, 열전달계수라면 FCNN은 "유한 차원 벡터에서 유한 차원 벡터로 가는 함수"를 학습한다.

이 관점은 [[Fourier Neural Operator (FNO)]]를 이해할 때 중요하다. FNO는 단순한 벡터 함수가 아니라, 함수 전체를 입력받아 함수 전체를 출력하는 [[Neural Operator]]를 학습한다.

---

## 컴퓨팅 관점의 입력과 출력

FCNN을 코드로 구현할 때 입력은 보통 `float32` tensor입니다. 예를 들어 batch size가 32이고 각 sample이 10개의 feature를 가진다면 입력 shape은 다음과 같습니다.

```text
X.shape = [32, 10]
```

출력이 2개의 값을 예측하는 문제라면 마지막 layer의 출력 shape은 다음과 같습니다.

```text
Y_hat.shape = [32, 2]
```

즉 FCNN은 컴퓨팅 관점에서 `[batch, input_dim]` tensor를 받아 `[batch, output_dim]` tensor로 바꾸는 함수입니다. 중간 hidden layer가 64차원이라면 계산 흐름은 `[32, 10] → [32, 64] → [32, 64] → [32, 2]`처럼 됩니다.

---

## 간단한 toy 예시

열수력 실험 correlation을 학습한다고 가정해 보겠습니다.

입력 10차원 벡터:

```text
[inlet_temperature, pressure, mass_flux, heat_flux, hydraulic_diameter,
 channel_length, inlet_quality, fluid_property_1, fluid_property_2, power]
```

출력 2차원 벡터:

```text
[pressure_drop, peak_wall_temperature]
```

이 경우 FCNN은 `R^10 → R^2` 매핑을 학습합니다.

---

## 관련 노트

- [[Activation Function]]
- [[Neural Operator]]
- [[Fourier Neural Operator (FNO)]]
- [[Surrogate Model]]
