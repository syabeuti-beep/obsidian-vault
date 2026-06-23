---
type: learning-note
topic: quant-trading
tags:
  - finance
  - quant-trading
  - stochastic-process
  - algorithmic-trading
created: 2026-06-23
status: draft
---
# Quant Trading

## 한 줄 요약

[[Quant Trading]]은 시장 데이터를 수학, 통계, 컴퓨터 알고리즘으로 해석하여 매매 의사결정을 자동화하거나 체계화하는 방식입니다.

---

## 왜 “퀀트”인가?

전통적인 discretionary trading은 사람이 뉴스, 기업 분석, 차트 등을 종합해 판단합니다. 반면 퀀트 트레이딩은 다음 질문을 수치화합니다.

- 어떤 데이터에서 반복 가능한 신호가 있는가?
- 그 신호가 우연이 아니라 통계적으로 의미 있는가?
- 거래 비용, 슬리피지, 리스크를 고려해도 수익성이 남는가?
- 전략이 시장 환경 변화에도 견딜 만큼 robust한가?

즉 퀀트 트레이딩의 핵심은 “좋은 예측 모델” 하나가 아니라, 데이터 → 가설 → 백테스트 → 리스크 관리 → 실행까지 이어지는 전체 시스템입니다.

---

## 핵심 구성 요소

1. 데이터
   - 가격: OHLCV, order book, tick data
   - 기초/매크로: 재무제표, 금리, 환율, 원자재, 뉴스
   - 대체 데이터: 위성, 웹 트래픽, 텍스트, 소셜 데이터

2. Alpha signal
   - 미래 수익률과 관련이 있을 것으로 기대되는 정량적 신호입니다.
   - 예: momentum, mean reversion, value, carry, volatility risk premium

3. Portfolio construction
   - 여러 신호와 자산을 조합해 실제 포지션 크기를 결정합니다.
   - 기대수익률만이 아니라 변동성, 상관관계, drawdown, 레버리지 제약을 고려합니다.

4. Risk management
   - 손실 가능성을 제한하고 전략이 망가졌는지 감시합니다.
   - VaR, expected shortfall, volatility targeting, stop rule 등이 쓰입니다.

5. Execution
   - 이론상 포지션을 실제 시장에서 체결하는 과정입니다.
   - 거래 비용, 시장 충격, bid-ask spread, latency가 중요합니다.

---

## 수학적 배경 지도

공학수학과 미분방정식 배경이 있다면, 퀀트 수학은 다음 순서로 접근하면 좋습니다.

1. 확률과 통계
   - random variable, expectation, variance, covariance
   - hypothesis test, confidence interval, regression
   - time series와 autocorrelation

2. 확률과정
   - 시간이 변하면서 무작위로 움직이는 변수 $X_t$를 다룹니다.
   - 주가, 금리, 변동성은 보통 deterministic ODE보다 [[Stochastic Process]]로 모델링합니다.

3. [[Brownian Motion]]
   - 연속시간 금융모델의 가장 기본적인 random noise입니다.
   - 물리의 diffusion/random walk와 직관적으로 연결됩니다.

4. [[Stochastic Differential Equation]]
   - 일반 미분방정식의 deterministic derivative 대신 무작위 충격 $dW_t$가 들어갑니다.
   - 예: $dS_t=\mu S_t dt+\sigma S_t dW_t$

5. [[Ito Lemma]]
   - SDE에서 변수변환을 할 때 쓰는 chain rule입니다.
   - 일반 미분의 chain rule과 달리 $(dW_t)^2=dt$에 해당하는 보정항이 생깁니다.

6. [[Black-Scholes Model]]
   - SDE와 Ito calculus가 실제 금융상품 가격결정에 어떻게 연결되는지 보여주는 대표 모델입니다.

---

## 공학수학 배경에서의 직관

공학수학에서 배운 ODE/PDE는 보통 다음 형태입니다.

$$
\frac{dx}{dt}=f(x,t)
$$

초기조건과 모델이 정해지면 해가 하나로 결정됩니다. 반면 금융시장에서 가격은 정보, 유동성, 참여자 행동, 뉴스 충격 때문에 매번 다른 경로를 탑니다. 그래서 다음처럼 deterministic drift와 random shock을 함께 씁니다.

$$
dX_t=a(X_t,t)dt+b(X_t,t)dW_t
$$

여기서 $a$는 평균적인 방향성, $b$는 불확실성의 크기, $W_t$는 Brownian motion입니다. 열전달에서 diffusion이 공간적으로 퍼지는 현상을 설명한다면, 금융의 확률모형에서는 불확실성이 시간에 따라 확산되는 구조를 설명한다고 볼 수 있습니다.

---

## 컴퓨팅 관점의 입력과 출력

퀀트 트레이딩 시스템을 코드로 보면 다음과 같습니다.

```text
가격/체결 데이터 [time, asset, features]
-> feature engineering [time, asset, factors]
-> alpha model [time, asset] expected return score
-> portfolio optimizer [time, asset] target weights
-> execution engine order list
-> realized PnL/risk report
```

예를 들어 일별 주식 500개, feature 20개, 5년치 데이터라면 입력 텐서는 대략 `[T=1250, N=500, F=20]` 형태가 됩니다. 모델 출력은 각 날짜와 종목에 대한 expected return score `[T, N]` 또는 목표 비중 `[T, N]`일 수 있습니다.

---

## 간단한 toy 예시

20일 momentum 전략을 생각해보겠습니다.

$$
r_{t}^{20}=\frac{P_t}{P_{t-20}}-1
$$

- 입력: 종가 배열 `prices[ticker, day]`
- feature: 최근 20일 수익률
- 규칙: $r_t^{20}$가 큰 상위 10% 종목 long, 하위 10% 종목 short
- 출력: 날짜별 포지션 weight matrix `[day, ticker]`
- 평가: 누적수익률, Sharpe ratio, maximum drawdown, turnover

이 전략은 단순하지만 퀀트의 전체 구조를 보여줍니다. 신호를 만들고, 포지션으로 바꾸고, 거래 비용을 반영하고, out-of-sample 성능을 검증해야 합니다.

---

## 주의할 함정

- Backtest overfitting: 과거 데이터에만 맞춘 전략은 미래에 무너질 수 있습니다.
- Look-ahead bias: 당시에는 알 수 없었던 정보를 사용하면 성능이 부풀려집니다.
- Survivorship bias: 살아남은 종목만 보면 과거 성과가 과대평가됩니다.
- Transaction cost: 작은 alpha는 수수료와 슬리피지로 사라질 수 있습니다.
- Regime change: 시장 구조가 바뀌면 과거 패턴이 더 이상 작동하지 않을 수 있습니다.

---

## 학습 순서

1. 확률/통계 기초 재정리
2. 수익률, 로그수익률, 변동성, 상관관계
3. [[Stochastic Process]]와 [[Brownian Motion]]
4. [[Stochastic Differential Equation]]와 [[Ito Lemma]]
5. [[Black-Scholes Model]]로 SDE 응용 보기
6. 백테스트와 리스크 관리
7. 실제 데이터로 small strategy 구현

---

## 관련 노트

- [[Stochastic Process]]
- [[Brownian Motion]]
- [[Stochastic Differential Equation]]
- [[Ito Lemma]]
- [[Black-Scholes Model]]
- [[Backtesting]]

---

## 내가 기억해야 할 핵심

- 퀀트 트레이딩은 수학 모델만이 아니라 데이터, 통계 검증, 포트폴리오, 리스크, 실행을 포함하는 시스템입니다.
- 공학수학의 ODE/PDE 직관은 SDE와 옵션 가격결정으로 확장될 수 있습니다.
- 확률미분방정식은 처음부터 엄밀한 측도론으로 접근하기보다, random walk → Brownian motion → SDE → Ito lemma 순서로 직관을 쌓는 것이 좋습니다.
