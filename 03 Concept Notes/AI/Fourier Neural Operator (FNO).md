---
type: learning-note
topic: fourier-neural-operator
aliases:
  - FNO
  - Fourier Neural Operator
tags:
  - machine-learning
  - scientific-machine-learning
  - neural-operator
  - fourier-transform
  - surrogate-model
  - pde
created: 2026-06-23
status: draft
---

# Fourier Neural Operator (FNO)

## 한 줄 요약

[[Fourier Neural Operator (FNO)]]는 PDE solver처럼 "입력 함수 또는 field를 받아 출력 함수 또는 field를 만드는 mapping"을 학습하기 위한 [[Neural Operator]]이다. 핵심은 field를 [[Fourier Transform]]으로 frequency space에 보낸 뒤, 낮은 Fourier mode에 대해 학습 가능한 weight를 곱하고 다시 physical space로 돌아오는 [[Spectral Convolution]]이다.

---

## 이 노트의 목표

이 노트는 [[Fully Connected Neural Network]]의 기본 수학, 즉

$$
\mathbf{x}_{\ell+1}=\sigma(W_\ell\mathbf{x}_\ell+\mathbf{b}_\ell)
$$

정도는 이해하는 일반 공학 전공자가 FNO의 목적, 개념, 수학적 구조를 이해하도록 설명한다.

핵심 질문은 다음이다.

1. 왜 일반 neural network가 아니라 neural operator가 필요한가?
2. Fourier transform이 FNO에서 어떤 역할을 하는가?
3. FNO layer는 행렬 계산 관점에서 어떻게 이해할 수 있는가?
4. PDE와 열수력/HPC 문제에서 FNO는 어떤 의미를 가지는가?

---

## 1. FNO의 목적

FNO의 목적은 고비용 수치해석 solver를 완전히 대체한다기보다, 특정 문제 class에서 PDE solution operator를 빠르게 근사하는 [[Surrogate Model]]을 만드는 것이다.

예를 들어 다음과 같은 mapping을 생각하자.

$$
a(x) \mapsto u(x)
$$

여기서:

- $a(x)$: 공간적으로 변하는 입력 field, 예를 들어 permeability, source term, initial condition, boundary condition, geometry-related field
- $u(x)$: PDE를 풀어서 얻는 solution field, 예를 들어 pressure, velocity, temperature

PDE solver는 $a(x)$가 주어졌을 때 $u(x)$를 계산한다. FNO는 많은 $(a,u)$ 쌍을 학습해서 이 mapping 자체를 근사한다.

$$
\mathcal{G}_\theta(a) \approx \mathcal{G}^\dagger(a)=u
$$

여기서 $\mathcal{G}^\dagger$는 실제 PDE solution operator이고, $\mathcal{G}_\theta$는 FNO가 학습한 operator이다.

---

## 2. FCNN에서 FNO로 넘어가는 직관

### 2.1 FCNN은 벡터에서 벡터로 가는 함수

[[Fully Connected Neural Network]]는 보통 다음 함수를 학습한다.

$$
f_\theta: \mathbb{R}^d \rightarrow \mathbb{R}^m
$$

입력과 출력의 차원이 고정되어 있다.

예를 들어 10개의 입력 feature로 1개의 scalar를 예측하면:

$$
\mathbb{R}^{10} \rightarrow \mathbb{R}
$$

### 2.2 PDE field는 단순 벡터보다 함수에 가깝다

온도장 $T(x,y)$, 압력장 $p(x,y,z)$, 속도장 $\mathbf{u}(x,y,z,t)$는 본질적으로 연속 공간/시간 위의 함수이다.

컴퓨터에서는 grid 위에 저장하므로 벡터나 tensor처럼 보이지만, 실제 의미는 함수이다.

```text
continuous field: T(x,y)
discrete data: T[i,j]
```

resolution을 바꾸면 벡터 길이가 바뀐다. FCNN은 입력 차원이 고정되어 있으므로 resolution 변화에 약하다.

### 2.3 FNO는 함수에서 함수로 가는 operator를 학습한다

FNO의 목표는 다음과 같다.

$$
\mathcal{G}_\theta: \mathcal{A} \rightarrow \mathcal{U}
$$

즉,

$$
\text{input field} \mapsto \text{output field}
$$

를 학습한다.

---

## 3. PDE solution operator 예시

Poisson equation을 예로 들자.

$$
-\nabla \cdot (a(x)\nabla u(x)) = f(x), \quad x\in D
$$

경계조건이 주어졌다고 하자. 여기서 $a(x)$가 공간적으로 변하는 계수라면, $a(x)$가 바뀔 때마다 solution $u(x)$도 달라진다.

이 문제는 다음 mapping으로 볼 수 있다.

$$
a(x) \mapsto u(x)
$$

즉 PDE를 푸는 과정 전체가 operator이다.

열수력 문제에서도 비슷하게 생각할 수 있다.

- 입력: 초기 온도장, 유속장, 경계조건, power distribution
- 출력: 일정 시간 후 온도장, 압력장, void fraction field

$$
\text{initial/boundary field} \mapsto \text{solution field}
$$

---

## 4. Neural operator의 일반 layer

많은 [[Neural Operator]]는 hidden field $v_t(x)$를 반복적으로 업데이트한다.

$$
v_{t+1}(x)=\sigma\left(Wv_t(x)+\int_D \kappa_\theta(x,y)v_t(y)dy\right)
$$

이 식의 의미를 FCNN 관점으로 풀면 다음과 같다.

### 4.1 Pointwise linear term

$$
Wv_t(x)
$$

위치 $x$마다 같은 행렬 $W$를 channel 방향에 적용한다. CNN의 1x1 convolution 또는 각 grid point에서 적용하는 작은 FC layer처럼 볼 수 있다.

### 4.2 Integral kernel term

$$
\int_D \kappa_\theta(x,y)v_t(y)dy
$$

위치 $x$의 새 값을 만들기 위해 domain 전체의 위치 $y$에서 오는 정보를 모은다. 즉 nonlocal interaction이다.

PDE solution은 한 지점의 값이 주변뿐 아니라 경계조건과 domain 전체 구조의 영향을 받을 수 있으므로 이 항이 중요하다.

### 4.3 Activation

$$
\sigma(\cdot)
$$

[[Activation Function]]은 nonlinear operator를 표현하기 위해 필요하다.

---

## 5. FNO의 핵심 아이디어

FNO는 위의 integral kernel을 직접 parameterize하지 않고 Fourier space에서 처리한다.

핵심 연산은 다음이다.

$$
K_\theta v = \mathcal{F}^{-1}\left(R_\theta \cdot \mathcal{F}(v)\right)
$$

여기서:

- $\mathcal{F}$: Fourier transform
- $\mathcal{F}^{-1}$: inverse Fourier transform
- $v$: hidden field
- $R_\theta$: Fourier mode별 학습 가능한 complex weight
- $K_\theta$: FNO가 학습하는 integral operator에 해당하는 부분

FNO layer는 보통 다음 형태이다.

$$
v_{t+1}(x)=\sigma\left(Wv_t(x)+\mathcal{F}^{-1}\left(R_\theta\cdot \mathcal{F}(v_t)\right)(x)\right)
$$

이 식 하나가 FNO의 핵심이다.

---

## 6. 행렬 계산 관점에서 FNO layer 이해하기

[[Discrete Fourier Transform]]을 행렬 $F$라고 하자. 그러면 FFT는 개념적으로 다음 행렬 곱이다.

$$
\hat{\mathbf{v}} = F\mathbf{v}
$$

Fourier coefficient에 weight $R$을 곱한다.

$$
\hat{\mathbf{y}} = R\hat{\mathbf{v}}
$$

다시 inverse Fourier transform을 한다.

$$
\mathbf{y} = F^{-1}\hat{\mathbf{y}}
$$

전체를 합치면:

$$
\mathbf{y} = F^{-1}RF\mathbf{v}
$$

FCNN의 linear layer가

$$
\mathbf{y}=W\mathbf{v}
$$

라면, FNO의 spectral convolution은

$$
\mathbf{y}=F^{-1}RF\mathbf{v}
$$

이다.

즉, FNO는 그냥 임의의 큰 행렬 $W$를 직접 학습하는 대신:

1. Fourier basis로 좌표계를 바꾸고,
2. frequency별 weight $R$을 적용하고,
3. 원래 physical space로 돌아온다.

이렇게 구조화된 선형 연산을 사용한다.

---

## 7. 왜 Fourier space인가?

### 7.1 PDE와 주파수 구조

많은 PDE solution은 공간적으로 smooth한 큰 scale 구조와 작은 scale 구조가 섞여 있다. Fourier transform은 이 구조를 frequency별로 분리한다.

- 낮은 frequency: 큰 scale, smooth trend
- 높은 frequency: 작은 scale, sharp variation, noise

FNO는 보통 낮은 Fourier mode 일부만 학습한다.

### 7.2 convolution을 효율적으로 계산

Convolution theorem에 의해 physical space의 convolution은 Fourier space에서 곱셈이다.

$$
\mathcal{F}(u*g)=\mathcal{F}(u)\mathcal{F}(g)
$$

따라서 nonlocal integral kernel을 Fourier space에서 효율적으로 표현할 수 있다.

### 7.3 Resolution 변화에 대한 일반화

FNO가 학습하는 것은 특정 grid point 사이의 dense matrix가 아니라 Fourier mode에 대한 weight이다. 그래서 같은 operator를 다른 resolution grid에 적용하기가 FCNN보다 자연스럽다.

단, 이것이 모든 경우에 완벽한 resolution invariance를 보장한다는 뜻은 아니다. 데이터 분포, boundary condition, aliasing, high-frequency feature에 따라 성능이 달라진다.

---

## 8. FNO architecture 전체 흐름

FNO는 보통 다음 구조를 가진다.

```text
input field a(x)
→ lifting layer P
→ 여러 개의 Fourier layer
→ projection layer Q
→ output field u(x)
```

### 8.1 Lifting

입력 field $a(x)$의 channel 수를 hidden dimension으로 올린다.

$$
v_0(x)=P(a(x),x)
$$

여기서 위치 좌표 $x$를 함께 넣는 경우가 많다. 좌표를 넣으면 모델이 domain 내 위치 정보를 직접 사용할 수 있다.

### 8.2 Fourier layers

여러 번 다음 연산을 반복한다.

$$
v_{t+1}(x)=\sigma\left(Wv_t(x)+\mathcal{F}^{-1}\left(R_t\cdot \mathcal{F}(v_t)\right)(x)\right)
$$

각 layer는 global spectral mixing과 pointwise channel mixing을 함께 수행한다.

### 8.3 Projection

마지막 hidden field를 원하는 출력 channel로 바꾼다.

$$
u(x)=Q(v_T(x))
$$

예를 들어 출력이 pressure field 하나라면 channel 1개, velocity 2D field라면 channel 2개, velocity+temperature라면 더 많은 channel을 가질 수 있다.

---

## 9. 1D 구현 수준의 수학적 흐름

1D grid에 hidden field $v \in \mathbb{R}^{N\times c}$가 있다고 하자.

- $N$: grid point 수
- $c$: channel 수

### Step 1. FFT

각 channel에 대해 FFT를 적용한다.

$$
\hat{v} = \mathcal{F}(v)
$$

결과는 complex Fourier coefficient이다.

$$
\hat{v} \in \mathbb{C}^{N\times c}
$$

### Step 2. low modes 선택

낮은 mode $k=0,1,\cdots,K-1$만 사용한다.

$$
\hat{v}_k, \quad k<K
$$

### Step 3. mode별 channel mixing

각 Fourier mode마다 channel을 섞는 complex matrix를 곱한다.

$$
\hat{y}_k = R_k \hat{v}_k
$$

여기서:

$$
R_k \in \mathbb{C}^{c_{out}\times c_{in}}
$$

즉 각 frequency마다 작은 fully connected layer가 있는 것처럼 볼 수 있다. 단, 이 FC layer는 physical grid point가 아니라 channel 방향에 적용된다.

### Step 4. inverse FFT

$$
y = \mathcal{F}^{-1}(\hat{y})
$$

### Step 5. pointwise term과 activation

$$
v_{next}(x)=\sigma(y(x)+Wv(x))
$$

---

## 10. 2D/3D field로 확장

2D field에서는 Fourier mode가 $(k_x,k_y)$가 된다.

$$
\hat{v}_{k_x,k_y} = \mathcal{F}_{2D}(v)
$$

FNO는 보통 낮은 mode 영역만 사용한다.

```text
|kx| <= Kx, |ky| <= Ky
```

3D에서는 $(k_x,k_y,k_z)$ mode를 사용한다.

열수력 CFD field에서는 2D/3D 공간과 시간까지 고려해야 하므로 다음 선택지가 있다.

1. 시간 marching: 현재 field를 넣어 다음 time step field를 예측
2. space-time FNO: 시간 차원까지 포함한 field를 한 번에 예측
3. parameter-to-solution: boundary/initial condition과 parameter field를 넣어 전체 solution field를 예측

---

## 11. 학습 데이터와 loss

학습 데이터는 보통 PDE solver나 실험/고충실도 시뮬레이션에서 얻은 pair이다.

$$
\{(a_i,u_i)\}_{i=1}^n
$$

FNO는 다음 loss를 최소화한다.

$$
\min_\theta \frac{1}{n}\sum_{i=1}^n \|\mathcal{G}_\theta(a_i)-u_i\|^2
$$

field 전체의 상대 오차를 쓰기도 한다.

$$
\frac{\|\mathcal{G}_\theta(a_i)-u_i\|_2}{\|u_i\|_2}
$$

물리 문제에서는 단순 MSE뿐 아니라 다음을 같이 고려할 수 있다.

- boundary condition violation
- conservation law residual
- PDE residual
- important scalar quantity error, 예: pressure drop, peak temperature, mass flow rate

---

## 12. FNO와 CNN의 차이

[[Convolutional Neural Network]]는 보통 local convolution kernel을 사용한다.

```text
주변 몇 개 grid cell만 보고 새 feature 계산
```

FNO의 spectral convolution은 Fourier transform을 통해 global information을 섞는다.

```text
domain 전체의 frequency component를 보고 새 feature 계산
```

따라서 FNO는 long-range dependency와 global operator를 표현하는 데 유리할 수 있다.

---

## 13. FNO의 장점

- PDE solution operator를 빠르게 근사할 수 있다.
- 학습 후 inference가 수치 solver보다 매우 빠를 수 있다.
- grid resolution 변화에 대해 일반 FCNN보다 자연스럽게 적용 가능하다.
- nonlocal interaction을 Fourier space에서 효율적으로 표현한다.
- [[Scientific Machine Learning]]과 [[Surrogate Model]] 연구에 잘 맞는다.

---

## 14. FNO의 한계와 주의점

- 학습 데이터 분포 밖 extrapolation은 여전히 어렵다.
- discontinuity, shock, sharp interface처럼 high-frequency 성분이 중요한 문제에서는 low-mode truncation이 손실을 만들 수 있다.
- 복잡한 geometry와 irregular mesh에는 기본 FNO가 바로 맞지 않을 수 있다.
- boundary condition 처리가 문제에 따라 까다롭다.
- FFT는 regular grid에 자연스럽기 때문에 unstructured mesh 문제에서는 다른 neural operator 변형이 필요할 수 있다.
- 물리 보존 법칙을 자동으로 만족하지는 않는다.

---

## 15. 열수력/HPC 연구와 연결해서 생각하기

FNO는 열수력과 HPC 연구에서 다음과 같은 방향으로 연결될 수 있다.

### 15.1 CFD surrogate model

고비용 CFD 시뮬레이션 대신, 특정 geometry/condition 범위에서 빠르게 field를 예측하는 surrogate로 사용할 수 있다.

예:

$$
\text{boundary condition, inlet profile, power distribution} \mapsto \text{temperature/velocity/pressure field}
$$

### 15.2 Parametric study acceleration

많은 parameter 조합을 sweep해야 할 때, 일부 고충실도 simulation으로 FNO를 학습하고 나머지 조건을 빠르게 탐색할 수 있다.

### 15.3 Uncertainty Quantification

입력 조건이 확률분포를 가질 때, Monte Carlo simulation을 고충실도 solver로만 돌리면 비용이 크다. FNO surrogate를 사용하면 [[Uncertainty Quantification]]의 반복 계산을 줄일 수 있다.

### 15.4 Multi-fidelity modeling

저충실도 solver와 고충실도 solver 데이터를 함께 사용해 FNO 기반 [[Multi-Fidelity Modeling]]을 구성할 수 있다.

---

## 16. FCNN과 FNO를 한 문장으로 비교

FCNN:

$$
\mathbf{y}=\sigma(W\mathbf{x}+\mathbf{b})
$$

고정 길이 벡터를 다른 고정 길이 벡터로 보낸다.

FNO Fourier layer:

$$
v_{t+1}(x)=\sigma\left(Wv_t(x)+\mathcal{F}^{-1}(R\cdot\mathcal{F}(v_t))(x)\right)
$$

함수/field를 Fourier space로 보내 global spectral interaction을 학습한 뒤 다시 field로 돌려보낸다.

---

## 내가 기억해야 할 핵심

- FNO는 [[Neural Operator]]의 한 종류이다.
- 목적은 PDE solution operator, 즉 field-to-field mapping을 학습하는 것이다.
- 일반 FCNN은 $\mathbb{R}^d \rightarrow \mathbb{R}^m$ mapping이고, FNO는 함수 공간 사이의 mapping을 목표로 한다.
- FNO의 핵심 layer는 $\mathcal{F}^{-1}(R\cdot\mathcal{F}(v))$이다.
- 행렬로 보면 spectral convolution은 $F^{-1}RF$이다.
- Fourier mode 중 낮은 mode만 학습해 큰 scale 구조를 효율적으로 포착한다.
- FNO는 빠른 PDE surrogate로 유용하지만, discontinuity, 복잡 geometry, out-of-distribution condition에서는 주의가 필요하다.

---

## 컴퓨팅 관점의 입력과 출력

FNO에서 “입력과 출력은 함수”라는 말은 코드에서는 grid tensor를 의미합니다. 예를 들어 2D 열수력 surrogate를 만든다면 입력은 다음처럼 구성할 수 있습니다.

```text
X.shape = [batch, Nx, Ny, input_channels]
```

입력 channel 예시:

```text
channel 0: inlet velocity field or mask
channel 1: wall heat flux field
channel 2: geometry/fluid mask
channel 3: x-coordinate
channel 4: y-coordinate
```

출력은 예측하려는 physical field입니다.

```text
Y_hat.shape = [batch, Nx, Ny, output_channels]
```

출력 channel 예시:

```text
channel 0: temperature
channel 1: pressure
channel 2: x-velocity
channel 3: y-velocity
```

따라서 실제 구현 관점에서는 `[batch, Nx, Ny, Cin] → [batch, Nx, Ny, Cout]` 매핑을 학습하는 모델입니다. 다만 이 tensor가 단순 feature vector가 아니라 공간 field의 discretization이라는 점이 FCNN과 다릅니다.

---

## 간단한 toy 예시

아주 작은 toy FNO 문제를 생각해 보겠습니다.

목표: 1D heat equation의 초기 온도 분포에서 일정 시간 후 온도 분포를 예측합니다.

```text
input  X.shape = [batch, 64, 2]
        channel 0: initial temperature T(x,0)
        channel 1: coordinate x

output Y.shape = [batch, 64, 1]
        channel 0: temperature T(x,t_final)
```

FCNN이라면 `64`개 값을 긴 벡터로 펼쳐 `[batch, 64] → [batch, 64]`로 처리할 수 있지만, FNO는 이 배열을 spatial field로 보고 FFT를 적용해 낮은 Fourier mode를 학습합니다. 그래서 같은 연산 구조를 `128`개 grid point에도 비교적 자연스럽게 적용할 수 있습니다.

---

## 관련 노트

- [[Fully Connected Neural Network]]
- [[Activation Function]]
- [[Neural Operator]]
- [[Operator Learning]]
- [[PDE Solution Operator]]
- [[Fourier Transform]]
- [[Discrete Fourier Transform]]
- [[Fast Fourier Transform]]
- [[Spectral Convolution]]
- [[Scientific Machine Learning]]
- [[Surrogate Model]]
- [[High Performance Computing]]
- [[Uncertainty Quantification]]
- [[Multi-Fidelity Modeling]]

---

## 다음에 확장할 질문

- FNO에서 Fourier mode 개수 $K$는 어떻게 정하는가?
- FNO가 discontinuity나 shock 문제에서 약한 이유는 무엇인가?
- FNO와 PINN은 목적과 학습 방식이 어떻게 다른가?
- irregular geometry나 unstructured mesh에서는 어떤 neural operator를 써야 하는가?
- 열수력 CFD 데이터에서 input/output channel을 어떻게 설계해야 하는가?
- FNO surrogate의 uncertainty를 어떻게 추정할 수 있는가?
