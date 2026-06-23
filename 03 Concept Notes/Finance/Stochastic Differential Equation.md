---
type: learning-note
topic: stochastic-differential-equation
tags:
  - probability
  - stochastic-calculus
  - finance
created: 2026-06-23
status: draft
---
# Stochastic Differential Equation

## 한 줄 요약

[[Stochastic Differential Equation]]은 deterministic 변화율과 random shock을 함께 포함하는 미분방정식이며, 금융에서는 주가·금리·변동성을 연속시간에서 모델링하는 핵심 도구입니다.

---

## ODE와 SDE의 차이

ODE는 보통 다음처럼 씁니다.

$$
\frac{dx}{dt}=f(x,t)
$$

SDE는 다음처럼 씁니다.

$$
dX_t=a(X_t,t)dt+b(X_t,t)dW_t
$$

- $a(X_t,t)dt$: drift, 평균적인 방향성
- $b(X_t,t)dW_t$: diffusion 또는 volatility, 무작위 충격
- $W_t$: [[Brownian Motion]]

같은 초기조건에서 ODE는 하나의 경로를 주지만, SDE는 매번 다른 sample path를 줍니다.

---

## 대표 예시: geometric Brownian motion

주가 $S_t$를 모델링할 때 자주 쓰는 형태입니다.

$$
dS_t=\mu S_tdt+\sigma S_tdW_t
$$

더 정확히 쓰면

$$
dS_t=\mu S_t\,dt+\sigma S_t\,dW_t
$$

여기서 $\mu$는 기대수익률, $\sigma$는 변동성입니다. 가격이 음수가 되지 않는다는 장점이 있어 [[Black-Scholes Model]]의 출발점이 됩니다.

---

## 해석 방법

SDE는 일반적인 미분계수 $dX_t/dt$로 해석하기보다 적분형으로 이해하는 것이 안전합니다.

$$
X_t=X_0+\int_0^t a(X_s,s)ds+\int_0^t b(X_s,s)dW_s
$$

두 번째 적분은 Ito integral이며 일반 Riemann integral과 다릅니다.

---

## Euler-Maruyama 방법

컴퓨터에서는 시간을 잘라 다음처럼 근사합니다.

$$
X_{n+1}=X_n+a(X_n,t_n)\Delta t+b(X_n,t_n)\Delta W_n
$$

$$
\Delta W_n=\sqrt{\Delta t}Z_n,\quad Z_n\sim\mathcal{N}(0,1)
$$

ODE의 Euler method에 random increment가 추가된 형태로 보면 됩니다.

---

## 컴퓨팅 관점의 입력과 출력

```text
입력: x0, drift function a(x,t), diffusion function b(x,t), dt, n_steps, n_paths
난수: Z[n_paths, n_steps]
출력: X[n_paths, n_steps + 1]
```

금융에서는 각 path가 하나의 가능한 미래 시장 시나리오입니다. derivative pricing에서는 각 path의 payoff를 계산한 뒤 평균을 할인합니다.

---

## 간단한 toy 예시

주가가 현재 100, 연 기대수익률 5%, 연 변동성 20%라고 하겠습니다.

$$
S_{n+1}=S_n+0.05S_n\Delta t+0.20S_n\sqrt{\Delta t}Z_n
$$

$\Delta t=1/252$로 두면 매 거래일마다 deterministic drift와 random shock을 더해 다음 가격을 만듭니다.

---

## 퀀트 트레이딩에서의 역할

- 옵션 가격결정
- 금리 모델: Vasicek, CIR, Hull-White
- stochastic volatility model: Heston model
- Monte Carlo risk simulation
- optimal control, portfolio optimization의 연속시간 formulation

---

## 관련 노트

- [[Quant Trading]]
- [[Stochastic Process]]
- [[Brownian Motion]]
- [[Ito Lemma]]
- [[Black-Scholes Model]]

---

## 내가 기억해야 할 핵심

- SDE는 ODE에 random shock term을 붙인 것이지만, 계산 규칙은 일반 미분과 다릅니다.
- Brownian increment의 크기가 $\sqrt{dt}$ scale이므로 Ito calculus가 필요합니다.
- 처음에는 Euler-Maruyama simulation으로 path를 직접 만들어보는 것이 이해에 좋습니다.
