---
type: learning-note
topic: brownian-motion
tags:
  - probability
  - stochastic-process
  - finance
created: 2026-06-23
status: draft
---
# Brownian Motion

## 한 줄 요약

[[Brownian Motion]]은 연속시간에서 누적되는 무작위 충격을 나타내는 기본 확률과정이며, 금융 SDE에서 noise 항 $dW_t$의 원천입니다.

---

## 핵심 직관

Brownian motion $W_t$는 아주 작은 random step들이 매우 많이 누적된 한계로 볼 수 있습니다. 물리적으로는 입자의 불규칙 운동, 수학적으로는 random walk의 연속시간 한계입니다.

---

## 정의의 핵심 성질

표준 Brownian motion $W_t$는 다음 성질을 갖습니다.

1. $W_0=0$
2. 독립 증가량: 겹치지 않는 시간 구간의 변화량은 독립입니다.
3. 정규 증가량:
   $$
   W_t-W_s\sim \mathcal{N}(0,t-s),\quad 0\le s<t
   $$
4. sample path는 연속이지만 거의 어디서나 미분 불가능합니다.

마지막 성질 때문에 일반적인 미분방정식 도구를 그대로 쓰기 어렵고, [[Ito Lemma]] 같은 새로운 계산 규칙이 필요합니다.

---

## 공학수학 배경에서의 해석

공학수학에서 noise가 있는 시스템을 배웠다면 white noise $\xi(t)$를 떠올릴 수 있습니다. 형식적으로는 다음처럼 생각할 수 있습니다.

$$
dW_t \approx \xi(t)dt
$$

하지만 $W_t$는 일반 함수처럼 미분할 수 없기 때문에 엄밀하게는 Ito calculus가 필요합니다. 처음 배울 때는 “시간 $dt$ 동안 평균 0, 분산 $dt$인 충격이 들어온다”고 이해하면 됩니다.

---

## 수학적 스케일링

작은 시간 간격 $dt$에서 Brownian increment는 다음처럼 시뮬레이션합니다.

$$
\Delta W \sim \mathcal{N}(0,dt)=\sqrt{dt}Z,\quad Z\sim\mathcal{N}(0,1)
$$

중요한 점은 random fluctuation의 크기가 $dt$가 아니라 $\sqrt{dt}$ scale이라는 것입니다. 이 때문에 $(dW_t)^2$가 $dt$ scale로 남고, Ito 보정항이 생깁니다.

---

## 컴퓨팅 관점의 입력과 출력

```text
입력: n_paths, n_steps, dt, random_seed
난수: Z[n_paths, n_steps] ~ Normal(0, 1)
증분: dW = sqrt(dt) * Z
출력: W = cumulative_sum(dW, axis=time)
```

출력 배열은 `[n_paths, n_steps + 1]`입니다. 각 row는 하나의 Brownian path입니다.

---

## 간단한 toy 예시

1년을 252 거래일로 나누면 $dt=1/252$입니다. 하루 Brownian increment는 대략

$$
\Delta W\sim\mathcal{N}(0,1/252)
$$

따라서 표준편차는 $1/\sqrt{252}$입니다. 주가 SDE에서 volatility가 $\sigma=20\%$라면 하루 random return scale은 대략 $0.2/\sqrt{252}\approx 1.26\%$ 수준입니다.

---

## 퀀트 트레이딩에서의 역할

- [[Stochastic Differential Equation]]의 noise source
- geometric Brownian motion 기반 주가 모델
- [[Black-Scholes Model]]의 수학적 기반
- Monte Carlo pricing과 risk simulation

---

## 관련 노트

- [[Stochastic Process]]
- [[Stochastic Differential Equation]]
- [[Ito Lemma]]
- [[Black-Scholes Model]]

---

## 내가 기억해야 할 핵심

- Brownian motion은 random walk의 연속시간 극한입니다.
- $dW_t$는 평균 0, 분산 $dt$인 작은 무작위 충격으로 생각할 수 있습니다.
- 경로는 연속이지만 미분 불가능하므로 일반 chain rule이 아니라 Ito lemma가 필요합니다.
