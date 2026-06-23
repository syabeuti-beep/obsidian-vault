---
type: learning-note
topic: discrete-fourier-transform
tags:
  - math
  - numerical-method
  - spectral-method
created: 2026-06-23
status: draft
---

# Discrete Fourier Transform

## 한 줄 요약

[[Discrete Fourier Transform]]은 grid 위에 샘플링된 유한 개의 값을 주파수 성분의 유한 개 계수로 바꾸는 변환이다.

[[Fourier Neural Operator (FNO)]]는 neural network layer 안에서 DFT/FFT를 사용해 field data를 Fourier space로 옮긴 뒤 spectral weight를 곱한다.

---

## 1D DFT

$N$개의 grid value가 있다고 하자.

$$
\mathbf{u}=[u_0,u_1,\cdots,u_{N-1}]^T
$$

DFT는 다음처럼 정의된다.

$$
\hat{u}_k = \sum_{j=0}^{N-1} u_j e^{-i2\pi jk/N}, \quad k=0,1,\cdots,N-1
$$

역 DFT는 다음과 같다.

$$
u_j = \frac{1}{N}\sum_{k=0}^{N-1} \hat{u}_k e^{i2\pi jk/N}
$$

---

## 행렬로 보는 DFT

DFT도 결국 행렬 곱이다.

$$
\hat{\mathbf{u}} = F\mathbf{u}
$$

여기서 $F$의 성분은 다음과 같다.

$$
F_{kj}=e^{-i2\pi jk/N}
$$

역변환은 다음처럼 쓸 수 있다.

$$
\mathbf{u} = F^{-1}\hat{\mathbf{u}}
$$

따라서 [[Fully Connected Neural Network]]를 행렬 계산으로 이해하고 있다면, DFT도 특수한 복소수 행렬을 곱하는 계산으로 볼 수 있다.

---

## 2D field에서의 DFT

CFD나 열수력 field처럼 2D grid가 있으면 $u(x,y)$를 주파수 index $(k_x,k_y)$의 계수로 바꾼다.

$$
\hat{u}_{k_x,k_y} = \sum_x \sum_y u(x,y)e^{-i2\pi(k_x x/N_x + k_y y/N_y)}
$$

이 계수들은 각 방향의 공간 주파수 성분을 나타낸다.

---

## FNO에서 중요한 점

FNO는 DFT로 얻은 모든 mode를 쓰지 않고, 보통 낮은 frequency mode만 남긴다.

```text
physical field → FFT → low Fourier modes 선택 → 학습 가능한 complex weight 곱 → inverse FFT
```

이는 계산량을 줄이고 큰 scale의 operator 구조를 학습하기 위한 선택이다.

---

## 컴퓨팅 관점의 입력과 출력

DFT의 입력은 길이 `N`의 실수 또는 복소수 배열이고 출력은 길이 `N`의 복소수 배열입니다.

```text
u.shape = [N]
uhat.shape = [N]
```

batch와 channel이 있으면 보통 grid dimension에 대해서만 DFT를 적용합니다. 예를 들어 FNO에서 1D hidden field가 `[batch, N, channels] = [16, 128, 32]`이면 FFT 후 shape은 여전히 `[16, 128, 32]`이지만 dtype은 complex가 됩니다.

---

## 간단한 toy 예시

길이 4짜리 signal을 생각해 보겠습니다.

```text
u = [1, 0, -1, 0]
```

DFT는 이 배열을 4개의 frequency coefficient로 바꿉니다. 이 예시는 하나의 cosine 파형에 가까우므로 특정 frequency coefficient가 크게 나오고 나머지는 작게 나옵니다. FNO는 이런 coefficient 중 낮은 mode 일부만 선택해 학습 가능한 weight를 곱합니다.

---

## 관련 노트

- [[Fourier Transform]]
- [[Fast Fourier Transform]]
- [[Spectral Convolution]]
- [[Fourier Neural Operator (FNO)]]
