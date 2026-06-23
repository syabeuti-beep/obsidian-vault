---
type: learning-note
topic: black-scholes-model
tags:
  - finance
  - option-pricing
  - stochastic-calculus
created: 2026-06-23
status: draft
---
# Black-Scholes Model

## 한 줄 요약

[[Black-Scholes Model]]은 주가가 geometric Brownian motion을 따른다고 가정하고 옵션 가격을 PDE로 유도하는 대표적인 연속시간 금융모델입니다.

---

## 기본 가정

주가 $S_t$가 다음 SDE를 따른다고 가정합니다.

$$
dS_t=\mu S_tdt+\sigma S_tdW_t
$$

또한 이상화된 시장을 가정합니다.

- 거래 비용과 세금 없음
- 연속적인 거래 가능
- 공매도와 차입 가능
- 무위험 이자율 $r$ 일정
- volatility $\sigma$ 일정
- arbitrage opportunity 없음

현실과 다르지만, SDE와 옵션 가격결정의 구조를 배우는 데 매우 좋은 출발점입니다.

---

## 핵심 아이디어

옵션 가격을 $V(S,t)$라고 두고 [[Ito Lemma]]를 적용합니다. 그 다음 주식과 옵션을 조합해 random term $dW_t$를 제거한 hedged portfolio를 만듭니다. 무작위성이 제거된 포트폴리오는 무위험 자산처럼 $r$만큼 수익을 내야 하므로 PDE가 나옵니다.

---

## Black-Scholes PDE

$$
\frac{\partial V}{\partial t}+\frac{1}{2}\sigma^2S^2\frac{\partial^2V}{\partial S^2}+rS\frac{\partial V}{\partial S}-rV=0
$$

European call option의 terminal condition은

$$
V(S,T)=\max(S-K,0)
$$

입니다.

---

## 닫힌형 해

European call price는 다음과 같습니다.

$$
C=S_0N(d_1)-Ke^{-rT}N(d_2)
$$

$$
d_1=\frac{\ln(S_0/K)+(r+\sigma^2/2)T}{\sigma\sqrt{T}},\quad d_2=d_1-\sigma\sqrt{T}
$$

여기서 $N(\cdot)$는 표준정규분포 CDF입니다.

---

## 공학수학 배경에서의 연결

Black-Scholes PDE는 확산항을 가진 parabolic PDE입니다. 열방정식과 구조적으로 닮아 있으며, 변수변환을 통해 heat equation 형태로 연결됩니다. 따라서 공학수학에서 배운 PDE 직관이 옵션 가격결정에도 이어집니다.

---

## 컴퓨팅 관점의 입력과 출력

```text
입력: S0, K, T, r, sigma, option_type
출력: option_price, Greeks(delta, gamma, vega 등)
```

Monte Carlo로도 계산할 수 있습니다.

```text
S_paths[n_paths, n_steps]
-> payoff = max(S_T - K, 0)
-> price = exp(-rT) * mean(payoff)
```

PDE solver를 쓰면 grid `[time, stock_price]` 위에서 $V(t,S)$를 계산합니다.

---

## 간단한 toy 예시

현재 주가 $S_0=100$, 행사가 $K=100$, 만기 $T=1$, 무위험이자율 $r=5\%$, 변동성 $\sigma=20\%$인 European call을 생각합니다.

Black-Scholes 공식에 넣으면 하나의 이론 가격이 나옵니다. 이 가격은 “주가가 오를 것 같다”는 주관적 예측보다, no-arbitrage와 동적 헤지 논리에서 나오는 가격입니다.

---

## 한계

- volatility가 실제로는 일정하지 않습니다.
- fat tail과 jump risk를 잘 설명하지 못합니다.
- 연속 거래와 완전한 hedge는 현실에서 불가능합니다.
- bid-ask spread, liquidity, transaction cost가 없습니다.

그래서 실제 퀀트에서는 local volatility, stochastic volatility, jump diffusion, rough volatility 같은 확장 모델을 사용합니다.

---

## 퀀트 트레이딩에서의 역할

- 옵션 이론 가격 산출
- implied volatility 계산
- Greeks 기반 risk management
- volatility trading과 market making의 기초

---

## 관련 노트

- [[Quant Trading]]
- [[Stochastic Differential Equation]]
- [[Ito Lemma]]
- [[Brownian Motion]]
- [[Backtesting]]

---

## 내가 기억해야 할 핵심

- Black-Scholes는 주가 SDE, Ito lemma, no-arbitrage hedge를 결합해 옵션 가격 PDE를 만듭니다.
- 수학적으로는 확률과정과 parabolic PDE가 만나는 대표 사례입니다.
- 현실 모델이라기보다 더 복잡한 금융모델을 이해하기 위한 기준점입니다.
