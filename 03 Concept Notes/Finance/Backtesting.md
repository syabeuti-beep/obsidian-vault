---
type: learning-note
topic: backtesting
tags:
  - finance
  - quant-trading
  - evaluation
created: 2026-06-23
status: draft
---
# Backtesting

## 한 줄 요약

[[Backtesting]]은 과거 데이터 위에서 trading strategy를 시뮬레이션하여 수익률, 리스크, 거래 비용, 안정성을 평가하는 과정입니다.

---

## 왜 필요한가?

퀀트 전략은 아이디어만으로는 충분하지 않습니다. 과거 데이터에서 다음을 확인해야 합니다.

- 신호가 미래 수익률과 관련이 있었는가?
- 거래 비용을 빼도 수익이 남는가?
- 특정 기간이나 종목군에만 우연히 맞은 것은 아닌가?
- drawdown을 감당할 수 있는가?
- 전략이 너무 자주 거래하지는 않는가?

---

## 기본 절차

1. 데이터 준비
   - 가격, 거래량, corporate action, survivorship-bias-free universe
2. Feature/Signal 계산
   - momentum, mean reversion, valuation factor 등
3. Portfolio rule
   - long/short, weight normalization, leverage constraint
4. Execution assumption
   - 다음날 open 체결, VWAP 체결, slippage model 등
5. Performance evaluation
   - CAGR, volatility, Sharpe ratio, max drawdown, turnover
6. Robustness check
   - 기간 분할, parameter sensitivity, out-of-sample test

---

## 핵심 지표

일별 수익률 $r_t$에 대해 Sharpe ratio는 보통 다음처럼 봅니다.

$$
\text{Sharpe}=\frac{\mathbb{E}[r_t-r_f]}{\sigma(r_t-r_f)}\sqrt{252}
$$

최대낙폭 maximum drawdown은 누적자산곡선이 이전 고점 대비 얼마나 하락했는지의 최댓값입니다.

---

## 컴퓨팅 관점의 입력과 출력

```text
입력:
  prices[time, asset]
  signals[time, asset]
  costs[asset] 또는 cost_model
  constraints
출력:
  weights[time, asset]
  trades[time, asset]
  pnl[time]
  metrics dict
```

핵심은 시점 정렬입니다. 오늘 종가로 계산한 signal이 오늘 종가 체결에 사용되면 look-ahead bias가 생길 수 있습니다. 보통 `signal[t] -> position[t+1] -> return[t+1]` 구조를 명확히 해야 합니다.

---

## 간단한 toy 예시

5일 mean reversion 전략:

$$
s_t=-\frac{P_t/P_{t-5}-1}{\sigma_{20}}
$$

최근 5일 많이 오른 종목은 short, 많이 내린 종목은 long한다고 가정합니다.

```text
prices[1000 days, 200 stocks]
-> returns_5d[1000, 200]
-> zscore signal[1000, 200]
-> weights[1000, 200]
-> daily_pnl[1000]
```

이후 Sharpe, drawdown, turnover, 거래비용 후 수익을 확인합니다.

---

## 흔한 함정

- Look-ahead bias: 미래 정보를 현재 의사결정에 사용
- Survivorship bias: 상장폐지 종목을 제외
- Data snooping: 많은 전략을 시도한 뒤 잘 된 것만 선택
- Overfitting: parameter를 과거 데이터에 과도하게 맞춤
- Ignoring costs: 수수료, spread, market impact 무시
- Capacity 문제: 전략 규모가 커지면 alpha가 사라짐

---

## 관련 노트

- [[Quant Trading]]

---

## 내가 기억해야 할 핵심

- Backtest는 전략의 “과거 시뮬레이션”이지 미래 수익 보장이 아닙니다.
- 시점 정렬, 거래 비용, universe 구성, out-of-sample 검증이 핵심입니다.
- 좋은 backtest는 수익률 그래프보다 실패 가능성을 드러내는 데 더 가치가 있습니다.
