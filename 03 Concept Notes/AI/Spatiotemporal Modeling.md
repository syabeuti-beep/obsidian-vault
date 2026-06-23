---
type: learning-note
topic: spatiotemporal-modeling
tags:
  - spatiotemporal-modeling
  - scientific-machine-learning
created: 2026-06-23
status: draft
---
# Spatiotemporal Modeling

## 한 줄 요약

[[Spatiotemporal Modeling]]은 공간과 시간에 따라 변하는 데이터를 함께 모델링하는 방법이다.

---

## 핵심 개념

- 온도장, 속도장, 압력장, void fraction은 모두 시간과 공간에 따라 변할 수 있다.
- 모델은 spatial correlation과 temporal dynamics를 동시에 다뤄야 한다.
- Transformer, CNN, FNO, recurrent model 등이 spatiotemporal data에 적용될 수 있다.

---

## 수학적 형태

시간 의존 field는 다음 함수로 쓸 수 있다.

$$
u= u(x,t) \quad \text{or} \quad u(x,y,z,t)
$$

모델링 목표는 과거 field와 조건으로 미래 field를 예측하는 것이다.

$$
u_{t+1}=\mathcal{G}(u_t, a_t)
$$

---

## 열수력/HPC 연구와의 연결

열수력 transient 해석에서는 power 변화, flow instability, boiling dynamics처럼 시간-공간 상호작용이 중요하다.

---

## 컴퓨팅 관점의 입력과 출력

입력은 `[batch, time, spatial_dims..., channels]` 형태의 tensor이고 출력은 미래 time step 또는 전체 trajectory입니다.

---

## 간단한 toy 예시

과거 10 step의 온도장 `[batch,10,64,64,1]`을 넣어 다음 5 step `[batch,5,64,64,1]`을 예측합니다.

---

## 관련 노트

- [[Transformer Model]]
- [[Fourier Neural Operator (FNO)]]
- [[Scientific Machine Learning]]

---

## 내가 기억해야 할 핵심

- Spatiotemporal Modeling는 Spatiotemporal Modeling은 공간과 시간에 따라 변하는 데이터를 함께 모델링하는 방법이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
