---
type: learning-note
topic: convolutional-neural-network
tags:
  - machine-learning
  - neural-network
  - spatial-data
created: 2026-06-23
status: draft
---

# Convolutional Neural Network

## 한 줄 요약

[[Convolutional Neural Network]]는 grid 형태 데이터에서 local pattern을 추출하기 위해 작은 convolution kernel을 여러 위치에 공유해서 적용하는 neural network 구조이다.

---

## 핵심 아이디어

이미지나 CFD field처럼 grid 위의 데이터는 주변 점들과 강한 상관관계를 가진다. CNN은 모든 grid point를 fully connected로 연결하지 않고, 작은 주변 영역만 보는 kernel을 사용한다.

```text
local patch → convolution kernel → feature
```

---

## FCNN과의 차이

[[Fully Connected Neural Network]]는 입력 벡터의 모든 성분을 dense matrix로 연결한다.

$$
\mathbf{y}=W\mathbf{x}
$$

CNN은 같은 kernel을 여러 위치에 반복 적용한다. 그래서 spatial locality와 translation equivariance를 이용할 수 있고, 파라미터 수가 줄어든다.

---

## FNO와의 차이

CNN은 보통 local convolution을 사용한다.

```text
주변 grid cell 중심
```

[[Fourier Neural Operator (FNO)]]는 [[Fourier Transform]]을 통해 domain 전체의 frequency component를 섞는다.

```text
global spectral interaction 중심
```

따라서 FNO는 PDE solution operator처럼 long-range interaction이 중요한 field-to-field mapping에서 유리할 수 있다.

---

## 컴퓨팅 관점의 입력과 출력

CNN의 입력은 보통 image 또는 field tensor입니다. PyTorch에서는 자주 `[batch, channels, height, width]` 형식을 쓰고, TensorFlow/일부 FNO 코드에서는 `[batch, height, width, channels]`를 쓰기도 합니다.

예시:

```text
X.shape = [16, 3, 64, 64]  # batch 16, channel 3, 64x64 grid
Y.shape = [16, 8, 64, 64]  # convolution 후 feature channel 8
```

convolution layer는 spatial size를 유지하거나 padding/stride에 따라 바꿀 수 있습니다.

---

## 간단한 toy 예시

`64×64` 온도장 하나를 입력받아 hot spot mask를 예측한다고 해보겠습니다.

```text
input.shape  = [batch, 1, 64, 64]
output.shape = [batch, 1, 64, 64]
```

CNN kernel은 각 grid cell 주변의 작은 patch, 예를 들어 `3×3` 영역을 보고 local pattern을 추출합니다.

---

## 관련 노트

- [[Fully Connected Neural Network]]
- [[Spectral Convolution]]
- [[Fourier Neural Operator (FNO)]]
- [[Neural Operator]]
