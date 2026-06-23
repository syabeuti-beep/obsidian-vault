---
type: learning-note
topic: stochastic-process
tags:
  - probability
  - stochastic-process
  - finance
created: 2026-06-23
status: draft
---
# Stochastic Process

## 한 줄 요약

[[Stochastic Process]]는 시간 또는 공간에 따라 변하는 random variable의 모음이며, 금융에서는 주가, 금리, 변동성처럼 불확실하게 움직이는 양을 모델링하는 기본 언어입니다.

---

## 핵심 직관

일반 함수 $x(t)$는 시간 $t$가 주어지면 값이 하나로 정해집니다. 반면 확률과정 $X_t$는 시간마다 값이 확률적으로 정해집니다.

$$
\{X_t: t\ge 0\}
$$

여기서 $t$를 고정하면 $X_t$는 random variable이고, 하나의 실현 경로를 보면 시간에 따른 곡선처럼 보입니다.

---

## 공학수학과의 연결

ODE에서는 다음 식이 deterministic trajectory를 줍니다.

$$
\frac{dx}{dt}=f(x,t)
$$

확률과정에서는 같은 초기값에서 시작해도 sample path가 여러 개 생깁니다. 열수력 시뮬레이션에서 boundary condition이 불확실하면 출력 온도장이 분포를 갖는 것처럼, 금융에서는 가격 경로 자체가 확률적입니다.

---

## 주요 개념

- State: 현재 값 $X_t$
- Sample path: 한 번의 무작위 실험에서 나온 시간 경로
- Distribution: 특정 시간 $t$에서 가능한 값들의 확률분포
- Stationarity: 시간 이동 후에도 통계적 성질이 같은가
- Markov property: 미래가 과거 전체가 아니라 현재 상태에만 의존하는가
- Autocorrelation: 현재 값과 과거 값 사이의 상관성

---

## 수학적 형태

가장 단순한 이산시간 random walk는 다음과 같습니다.

$$
X_{n+1}=X_n+\epsilon_{n+1},\quad \epsilon_n\sim \text{i.i.d.}
$$

금융에서 로그가격이 random walk에 가깝다는 가정은 efficient market intuition과 연결됩니다. 연속시간 한계로 가면 [[Brownian Motion]]이 됩니다.

---

## 컴퓨팅 관점의 입력과 출력

확률과정을 시뮬레이션할 때는 연속시간을 격자로 자릅니다.

```text
입력: n_paths, n_steps, dt, initial_value, random_seed, model_parameters
출력: paths[n_paths, n_steps + 1]
```

예를 들어 Monte Carlo로 주가 경로 10,000개를 만들면 배열은 `[10000, 252]`처럼 됩니다. 각 row는 하나의 가능한 시장 시나리오입니다.

---

## 간단한 toy 예시

동전 던지기 random walk:

$$
X_0=0,\quad X_{n+1}=X_n+\begin{cases}1 & \text{heads}\\ -1 & \text{tails}\end{cases}
$$

100번 던지면 한 경로가 생기고, 10,000번 반복하면 최종 위치 $X_{100}$의 분포를 볼 수 있습니다. 이것이 Brownian motion과 SDE 직관의 출발점입니다.

---

## 퀀트 트레이딩에서의 역할

- 가격 경로 모델링
- volatility와 correlation 추정
- derivative pricing을 위한 Monte Carlo simulation
- risk metric의 분포 추정
- regime switching model, hidden Markov model 등 시장 상태 모델링

---

## 관련 노트

- [[Quant Trading]]
- [[Brownian Motion]]
- [[Stochastic Differential Equation]]

---

## 내가 기억해야 할 핵심

- 확률과정은 “시간에 따라 움직이는 확률변수”입니다.
- 공학의 deterministic trajectory가 하나의 해를 준다면, 확률과정은 가능한 경로들의 ensemble을 줍니다.
