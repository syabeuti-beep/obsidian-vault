---
type: learning-note
topic: high-performance-computing
tags:
  - hpc
  - scientific-computing
created: 2026-06-23
status: draft
---
# High Performance Computing

## 한 줄 요약

[[High Performance Computing]]은 대규모 계산을 위해 많은 CPU/GPU, 고속 네트워크, 병렬 파일시스템을 활용하는 계산 기술이다.

---

## 핵심 개념

- CFD, 열수력, multiphysics simulation은 HPC 자원을 많이 사용한다.
- 대규모 neural network 학습과 추론도 GPU 기반 HPC와 연결된다.
- 성능은 연산량뿐 아니라 메모리 대역폭, 통신 비용, I/O 병목에 좌우된다.

---

## 수학적 형태

병렬 효율은 자주 다음처럼 본다.

$$
E_p=\frac{T_1}{pT_p}
$$

여기서 $T_1$은 단일 프로세서 시간, $T_p$는 $p$개 프로세서 시간이다.

---

## 열수력/HPC 연구와의 연결

FNO 같은 surrogate는 고비용 HPC solver의 반복 호출을 줄이는 데 쓸 수 있고, 반대로 FNO 학습에는 HPC/GPU 자원이 필요할 수 있다.

---

## 컴퓨팅 관점의 입력과 출력

입력은 계산 job script, mesh/data, solver executable이고 출력은 simulation result, log, checkpoint file입니다. 병렬 프로그램에서는 데이터를 여러 rank/GPU에 나눠 저장합니다.

---

## 간단한 toy 예시

1024 core로 CFD를 돌려 pressure field 파일과 residual log를 생성하고, 그 결과를 FNO 학습 데이터로 사용할 수 있습니다.

---

## 관련 노트

- [[Scientific Machine Learning]]
- [[Fourier Neural Operator (FNO)]]
- [[Surrogate Model]]
- [[Quant Trading]]
- [[Backtesting]]

---

## 내가 기억해야 할 핵심

- High Performance Computing는 High Performance Computing은 대규모 계산을 위해 많은 CPU/GPU, 고속 네트워크, 병렬 파일시스템을 활용하는 계산 기술이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
