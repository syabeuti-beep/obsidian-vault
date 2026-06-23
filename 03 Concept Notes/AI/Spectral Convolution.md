---
type: learning-note
topic: spectral-convolution
tags:
  - machine-learning
  - spectral-method
  - fourier-transform
created: 2026-06-23
status: draft
---

# Spectral Convolution

## 한 줄 요약

[[Spectral Convolution]]은 physical space에서 직접 convolution을 계산하는 대신, [[Fourier Transform]]으로 frequency space에 간 뒤 Fourier coefficient에 학습 가능한 weight를 곱하고 다시 physical space로 돌아오는 연산이다.

---

## convolution의 기본 아이디어

1D에서 convolution은 다음과 같다.

$$
(u*g)(x)=\int g(x-y)u(y)dy
$$

이는 위치 $x$의 값을 계산할 때 주변 또는 전체 위치 $y$의 값을 kernel $g$로 가중합하는 것이다.

---

## Fourier space에서 convolution은 곱셈

Fourier transform의 convolution theorem에 의해:

$$
\mathcal{F}(u*g)=\mathcal{F}(u)\mathcal{F}(g)
$$

즉, physical space에서 적분 convolution을 직접 계산하는 대신 frequency space에서 coefficient-wise multiplication을 할 수 있다.

---

## FNO의 spectral convolution

[[Fourier Neural Operator (FNO)]]의 핵심 연산은 다음과 같다.

$$
K_\theta v = \mathcal{F}^{-1}\left(R_\theta \cdot \mathcal{F}(v)\right)
$$

여기서:

- $v$: hidden field
- $\mathcal{F}(v)$: Fourier coefficient
- $R_\theta$: 학습 가능한 complex-valued weight
- $\cdot$: Fourier mode별 곱셈
- $\mathcal{F}^{-1}$: inverse Fourier transform

실제 구현에서는 낮은 Fourier mode 몇 개만 $R_\theta$로 학습하고, 나머지 mode는 버리거나 0으로 둔다.

---

## 행렬 계산으로 이해하기

[[Discrete Fourier Transform]]을 행렬 $F$라고 하면 spectral convolution은 다음처럼 쓸 수 있다.

$$
\mathbf{y} = F^{-1} R F \mathbf{x}
$$

여기서 $R$은 Fourier coefficient에 곱해지는 행렬이다. 가장 단순하게는 diagonal matrix처럼 각 frequency를 scaling한다고 볼 수 있고, FNO에서는 channel 사이 mixing까지 포함한다.

즉 FCNN의 한 층이

$$
\mathbf{y} = W\mathbf{x}
$$

라면 spectral convolution은

$$
\mathbf{y} = F^{-1} R F\mathbf{x}
$$

처럼 "Fourier basis로 좌표계를 바꾼 뒤, 그 좌표계에서 선형 변환하고, 다시 원래 좌표계로 돌아오는 연산"이다.

---

## 왜 낮은 mode만 쓰는가?

많은 PDE solution은 큰 scale 구조가 중요한 경우가 많다. 낮은 Fourier mode는 domain 전체의 smooth한 구조를 담고, 높은 mode는 작은 scale 변화나 noise를 담는다.

FNO는 낮은 mode를 중심으로 operator를 학습해 계산량을 줄이고 resolution 변화에 대해 비교적 잘 일반화하도록 만든다.

---

## 컴퓨팅 관점의 입력과 출력

Spectral convolution의 입력은 physical space field tensor이고 출력도 physical space field tensor입니다. 중간에서만 Fourier coefficient로 바뀝니다.

1D FNO 예시:

```text
v.shape      = [batch, N, in_channels]
v_hat.shape  = [batch, N_freq, in_channels]  # complex
y_hat.shape  = [batch, N_freq, out_channels] # selected modes only modified
y.shape      = [batch, N, out_channels]
```

즉 사용자 입장에서는 field를 넣고 field를 받지만, layer 내부에서는 FFT → complex weight 곱 → IFFT가 일어납니다.

---

## 간단한 toy 예시

길이 64의 1D 압력 분포 hidden field가 있고 channel이 16개라고 해보겠습니다.

```text
v.shape = [1, 64, 16]
```

FFT 후 낮은 mode 12개만 사용합니다. 각 mode마다 `16×16` complex matrix를 곱해 channel을 섞고 inverse FFT를 하면 다시 `[1, 64, 16]` field가 됩니다.

---

## 관련 노트

- [[Fourier Transform]]
- [[Discrete Fourier Transform]]
- [[Fast Fourier Transform]]
- [[Neural Operator]]
- [[Fourier Neural Operator (FNO)]]
