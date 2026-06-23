---
type: learning-note
topic: activation-function
tags:
  - machine-learning
  - neural-network
  - math
created: 2026-06-23
status: draft
---

# Activation Function

## 한 줄 요약

[[Activation Function]]은 neural network의 각 층에서 선형 변환 결과에 적용되는 비선형 함수이며, 모델이 단순 선형 회귀를 넘어 복잡한 비선형 관계를 학습하게 만든다.

---

## 기본 형태

한 층의 계산은 보통 다음과 같다.

$$
\mathbf{x}_{\ell+1} = \sigma(W_\ell \mathbf{x}_\ell + \mathbf{b}_\ell)
$$

여기서 $\sigma$가 activation function이다. 벡터에 적용할 때는 각 성분마다 같은 함수를 적용한다.

$$
\sigma(\mathbf{z}) = [\sigma(z_1), \sigma(z_2), \cdots, \sigma(z_m)]^T
$$

---

## 대표 예시

### ReLU

$$
\mathrm{ReLU}(x)=\max(0,x)
$$

계산이 단순하고 deep network에서 gradient가 비교적 잘 흐른다.

### Tanh

$$
\tanh(x)=\frac{e^x-e^{-x}}{e^x+e^{-x}}
$$

출력이 $[-1,1]$ 범위에 있어 매끄러운 비선형 변환이 필요할 때 사용된다.

### GELU

[[Transformer Model]] 계열에서 자주 쓰이는 activation이다. ReLU처럼 음수 영역을 완전히 자르지 않고 부드럽게 조절한다.

---

## FNO에서의 역할

[[Fourier Neural Operator (FNO)]]의 한 layer는 Fourier space에서 선형 변환을 하고 다시 physical space로 돌아온 뒤 activation을 적용한다.

$$
v_{t+1}(x)=\sigma\left(Wv_t(x)+\mathcal{F}^{-1}(R\cdot \mathcal{F}(v_t))(x)\right)
$$

여기서 Fourier transform과 spectral weight $R$만 있으면 전체 layer가 거의 선형이므로, activation function이 nonlinear operator를 표현하는 데 중요하다.

---

## 컴퓨팅 관점의 입력과 출력

Activation function은 tensor의 shape을 보통 바꾸지 않습니다. 예를 들어 linear layer 결과가 `[32, 64]`이면 ReLU, tanh, GELU를 적용한 뒤에도 shape은 `[32, 64]`입니다.

```text
Z.shape = [batch, hidden_dim]
A = relu(Z)
A.shape = [batch, hidden_dim]
```

즉 activation은 각 숫자 성분에 독립적으로 적용되는 elementwise 연산입니다.

---

## 간단한 toy 예시

입력이 다음과 같다고 하겠습니다.

```text
z = [-2.0, -0.3, 0.0, 1.5]
```

ReLU를 적용하면:

```text
relu(z) = [0.0, 0.0, 0.0, 1.5]
```

선형 계산 결과 중 음수 신호를 잘라내면서 비선형성이 생깁니다.

---

## 관련 노트

- [[Fully Connected Neural Network]]
- [[Fourier Neural Operator (FNO)]]
- [[Spectral Convolution]]
