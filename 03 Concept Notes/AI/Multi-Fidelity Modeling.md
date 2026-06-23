---
type: learning-note
topic: multi-fidelity-modeling
tags:
  - scientific-machine-learning
  - surrogate-model
created: 2026-06-23
status: draft
---
# Multi-Fidelity Modeling

## 한 줄 요약

[[Multi-Fidelity Modeling]]은 저충실도 모델의 많은 데이터와 고충실도 모델의 적은 데이터를 함께 사용해 정확도와 비용의 균형을 맞추는 방법이다.

---

## 핵심 개념

- 저충실도 모델은 싸지만 bias가 있고, 고충실도 모델은 비싸지만 정확하다.
- 두 정보를 결합하면 적은 비용으로 더 좋은 surrogate를 만들 수 있다.
- FNO도 multi-fidelity 데이터와 결합해 field surrogate로 확장할 수 있다.

---

## 수학적 형태

간단한 additive correction 형태는 다음과 같다.

$$
y_H(x) \approx \rho y_L(x)+\delta_\theta(x)
$$

여기서 $y_L$은 low-fidelity, $y_H$는 high-fidelity 결과이다.

---

## 열수력/HPC 연구와의 연결

예를 들어 system code나 coarse CFD 결과를 low-fidelity로, fine-grid LES/DNS 또는 실험 데이터를 high-fidelity로 사용할 수 있다.

---

## 컴퓨팅 관점의 입력과 출력

입력은 low-fidelity dataset과 high-fidelity dataset입니다. 출력은 high-fidelity 결과를 더 싸게 예측하는 corrected model입니다.

---

## 간단한 toy 예시

coarse CFD 10000개와 fine CFD 100개를 함께 써서 fine CFD와 가까운 pressure drop surrogate를 학습합니다.

---

## 관련 노트

- [[Surrogate Model]]
- [[Fourier Neural Operator (FNO)]]
- [[Scientific Machine Learning]]
- [[High Performance Computing]]

---

## 내가 기억해야 할 핵심

- Multi-Fidelity Modeling는 Multi-Fidelity Modeling은 저충실도 모델의 많은 데이터와 고충실도 모델의 적은 데이터를 함께 사용해 정확도와 비용의 균형을 맞추는 방법이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
