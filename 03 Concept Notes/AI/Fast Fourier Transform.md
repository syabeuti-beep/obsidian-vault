---
type: learning-note
topic: fast-fourier-transform
tags:
  - math
  - numerical-method
  - spectral-method
created: 2026-06-23
status: draft
---

# Fast Fourier Transform

## 한 줄 요약

[[Fast Fourier Transform]]은 [[Discrete Fourier Transform]]을 빠르게 계산하는 알고리즘 계열이다.

DFT를 정의 그대로 계산하면 $O(N^2)$ 연산이 필요하지만, FFT는 보통 $O(N\log N)$에 계산한다.

---

## 왜 중요한가?

[[Fourier Neural Operator (FNO)]]는 layer마다 Fourier transform과 inverse Fourier transform을 수행한다.

```text
field → FFT → spectral weight multiplication → inverse FFT
```

따라서 FFT가 빠르기 때문에 FNO의 spectral convolution도 실용적으로 계산할 수 있다.

---

## 행렬 관점

DFT는 특수한 행렬 $F$를 곱하는 연산이다.

$$
\hat{\mathbf{u}}=F\mathbf{u}
$$

직접 행렬 곱으로 계산하면 $N^2$개의 곱셈이 필요하다. FFT는 Fourier matrix의 반복 구조를 이용해서 같은 결과를 더 적은 연산으로 계산한다.

---

## FNO와의 연결

FNO의 spectral convolution은 개념적으로 다음과 같다.

$$
\mathbf{y}=F^{-1}RF\mathbf{x}
$$

실제 구현에서는 $F$와 $F^{-1}$를 큰 dense matrix로 만들지 않고 FFT/IFFT 함수로 계산한다.

---

## 컴퓨팅 관점의 입력과 출력

FFT는 DFT와 입력/출력 shape은 같지만 계산 알고리즘이 빠릅니다.

```text
input:  u.shape = [N]
output: uhat.shape = [N]
```

직접 DFT 행렬을 만들면 `[N, N] @ [N]` 계산이라 `O(N^2)`이지만, FFT 함수는 같은 결과를 `O(N log N)`에 계산합니다. FNO 구현에서는 `torch.fft.rfft`, `torch.fft.fft2` 같은 함수를 사용합니다.

---

## 간단한 toy 예시

`N=1024`인 1D field에 대해 DFT를 직접 계산하면 대략 백만 번 수준의 곱셈이 필요합니다. FFT를 쓰면 훨씬 적은 연산으로 같은 Fourier coefficient를 얻습니다. 그래서 FNO layer 안에서 매번 Fourier transform을 해도 실용적인 계산 시간이 나옵니다.

---

## 관련 노트

- [[Discrete Fourier Transform]]
- [[Fourier Transform]]
- [[Spectral Convolution]]
- [[Fourier Neural Operator (FNO)]]
