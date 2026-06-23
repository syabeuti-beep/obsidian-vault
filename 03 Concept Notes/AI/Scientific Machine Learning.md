---
type: learning-note
topic: scientific-machine-learning
tags:
  - machine-learning
  - numerical-method
  - pde
  - hpc
created: 2026-06-23
status: draft
---

# Scientific Machine Learning

## 한 줄 요약

[[Scientific Machine Learning]]은 물리 법칙, 수치해석, 시뮬레이션 데이터, 머신러닝을 결합해 과학/공학 문제를 푸는 연구 분야이다.

---

## 일반 machine learning과 다른 점

일반 ML은 데이터 분포에서 예측 성능을 높이는 데 초점을 두는 경우가 많다. SciML은 여기에 다음 요구가 추가된다.

- PDE, conservation law, boundary condition 같은 물리 제약
- numerical solver와의 연결
- extrapolation과 stability 문제
- uncertainty quantification
- HPC simulation data 활용
- 해석 가능성과 검증 가능성

---

## 대표 방향

- [[Physics-Informed Neural Network]]: PDE residual을 loss에 넣어 학습
- [[Neural Operator]]: PDE solution operator를 데이터로부터 학습
- [[Fourier Neural Operator (FNO)]]: Fourier space에서 operator를 학습
- [[Surrogate Model]]: 고비용 solver의 빠른 대체 모델
- [[Reduced Order Model]]: 고차원 시스템을 저차원 표현으로 축약

---

## 열수력/HPC와의 연결

열수력 연구에서는 CFD, system code, 실험 데이터, uncertainty quantification이 모두 중요하다. SciML은 이들을 연결해 빠른 예측, inverse problem, parameter study, design optimization을 가능하게 하는 도구가 될 수 있다.

---

## 컴퓨팅 관점의 입력과 출력

SciML 코드는 보통 물리 변수와 ML tensor를 함께 다룹니다. 입력은 실험/시뮬레이션 데이터, PDE residual 계산에 필요한 coordinate, boundary condition 등이 될 수 있습니다.

예시 tensor:

```text
coords.shape = [num_points, 2]       # x, y
fields.shape = [batch, Nx, Ny, C]    # T, p, u, v 등
params.shape = [batch, num_params]   # Reynolds number, heat flux 등
```

출력은 scalar QoI, field, residual, uncertainty 등 문제 설정에 따라 달라집니다.

---

## 간단한 toy 예시

2D 열전도 문제에서 SciML 모델을 만든다고 해보겠습니다. 입력은 좌표 `(x,y)`와 열원 `q(x,y)`, 출력은 온도 `T(x,y)`입니다. 데이터 loss는 solver 결과와의 차이를 줄이고, physics loss는 `-k∇²T=q` residual을 줄이도록 만들 수 있습니다.

---

## 관련 노트

- [[Fourier Neural Operator (FNO)]]
- [[Surrogate Model]]
- [[Neural Operator]]
- [[High Performance Computing]]
- [[Uncertainty Quantification]]
