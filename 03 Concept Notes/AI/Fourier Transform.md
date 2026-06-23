---
type: learning-note
topic: fourier-transform
tags:
  - math
  - signal-processing
  - spectral-method
created: 2026-06-23
status: draft
---

# Fourier Transform

## 한 줄 요약

[[Fourier Transform]]은 공간 또는 시간에 있는 함수를 여러 주파수 성분의 합으로 표현하는 변환이다.

직관적으로는 "복잡한 파형을 sine/cosine 파동들의 조합으로 분해하는 방법"이다.

---

## 연속 Fourier transform

1차원 함수 $u(x)$에 대해 Fourier transform은 다음처럼 쓸 수 있다.

$$
\hat{u}(k)=\int u(x)e^{-i2\pi kx}dx
$$

역변환은 다음과 같다.

$$
u(x)=\int \hat{u}(k)e^{i2\pi kx}dk
$$

여기서:

- $x$: physical space의 위치
- $k$: frequency 또는 wavenumber
- $u(x)$: 원래 함수
- $\hat{u}(k)$: 각 주파수 성분의 계수

---

## Discrete Fourier Transform

컴퓨터에서는 연속 함수가 아니라 grid 위의 값들을 다룬다.

$$
u_j = u(x_j), \quad j=0,1,\cdots,N-1
$$

이때 [[Discrete Fourier Transform]]은 다음과 같다.

$$
\hat{u}_k = \sum_{j=0}^{N-1} u_j e^{-i2\pi jk/N}
$$

역변환은 다음과 같다.

$$
u_j = \frac{1}{N}\sum_{k=0}^{N-1} \hat{u}_k e^{i2\pi jk/N}
$$

실제 계산에서는 [[Fast Fourier Transform]] 알고리즘을 사용해 빠르게 계산한다.

---

## 저주파와 고주파

- 저주파: 전체적인 큰 구조, smooth trend
- 고주파: 급격한 변화, 작은 scale 구조, noise

[[Fourier Neural Operator (FNO)]]는 모든 Fourier mode를 다 쓰지 않고 보통 낮은 mode 일부만 학습한다. 이는 PDE solution operator에서 큰 scale 구조가 중요한 경우가 많기 때문이다.

---

## convolution과 Fourier transform

Fourier transform의 중요한 성질 중 하나는 convolution이 Fourier space에서는 곱셈이 된다는 점이다.

$$
\mathcal{F}(u * g)=\mathcal{F}(u)\mathcal{F}(g)
$$

이 성질 때문에 [[Spectral Convolution]]은 physical space에서 직접 convolution kernel을 적용하는 대신 Fourier space에서 coefficient를 곱하는 방식으로 계산할 수 있다.

---

## 컴퓨팅 관점의 입력과 출력

코드에서는 연속 함수 `u(x)`를 직접 넣지 않고 grid 위에서 샘플링한 배열을 넣습니다. 예를 들어 1D field를 128개 grid point에서 샘플링하면 입력은 `u.shape=[128]`이고 FFT 출력은 `uhat.shape=[128]`인 complex-valued 배열입니다. 2D field라면 `u.shape=[64,64] → uhat.shape=[64,64]`입니다. 출력은 같은 크기의 배열이지만 값의 의미가 physical value가 아니라 frequency coefficient입니다.

---

## 간단한 toy 예시

1D 온도 분포 `T=[1.0,1.7,2.0,1.7,1.0,0.3,0.0,0.3]`에 FFT를 적용하면 몇 개의 sine/cosine 성분으로 구성되어 있는지 나타내는 복소수 coefficient 배열이 나옵니다. 낮은 frequency coefficient가 크면 전체적으로 부드러운 파형이고, 높은 frequency coefficient가 크면 급격한 변화가 많다는 뜻입니다.

---

## 관련 노트

- [[Discrete Fourier Transform]]
- [[Fast Fourier Transform]]
- [[Spectral Convolution]]
- [[Fourier Neural Operator (FNO)]]
