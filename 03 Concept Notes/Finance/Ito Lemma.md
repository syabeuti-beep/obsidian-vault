---
type: learning-note
topic: ito-lemma
tags:
  - probability
  - stochastic-calculus
  - finance
created: 2026-06-23
status: draft
---
# Ito Lemma

## 한 줄 요약

[[Ito Lemma]]는 [[Stochastic Differential Equation]]에서 함수 $f(X_t,t)$의 변화량을 계산하는 stochastic chain rule입니다.

---

## 왜 일반 chain rule이 안 되는가?

일반 미적분에서는 $x(t)$가 미분 가능하면

$$
df=f_xdx+f_tdt
$$

처럼 계산합니다. 하지만 SDE에서는

$$
dX_t=a(X_t,t)dt+b(X_t,t)dW_t
$$

이고, [[Brownian Motion]]의 변화량은 $dW_t\sim \sqrt{dt}$ scale입니다. 따라서 Taylor expansion에서 $(dW_t)^2$ 항이 $dt$ scale로 살아남습니다.

---

## 공식

$X_t$가

$$
dX_t=a(X_t,t)dt+b(X_t,t)dW_t
$$

를 따르고 $Y_t=f(X_t,t)$라면,

$$
dY_t=\left(f_t+a f_x+\frac{1}{2}b^2 f_{xx}\right)dt+b f_x dW_t
$$

여기서 핵심은 $\frac{1}{2}b^2f_{xx}$ 보정항입니다.

---

## 공학수학 배경에서의 직관

일반 chain rule은 일차항만 남기고 고차항을 버립니다. 그러나 Brownian increment에서는

$$
(dW_t)^2\sim dt
$$

처럼 행동하므로 이차항 일부가 사라지지 않습니다. 확산 방정식에서 second derivative가 퍼짐을 결정하듯이, Ito lemma의 second derivative 항도 불확실성의 curvature 효과를 반영합니다.

---

## 예시: 로그 주가

주가가 geometric Brownian motion을 따른다고 합시다.

$$
dS_t=\mu S_tdt+\sigma S_tdW_t
$$

로그 $f(S)=\ln S$에 Ito lemma를 적용하면

$$
d\ln S_t=\left(\mu-\frac{1}{2}\sigma^2\right)dt+\sigma dW_t
$$

단순히 $dS/S$라고 생각하면 $\mu dt+\sigma dW_t$가 될 것 같지만, 실제로는 $-\frac{1}{2}\sigma^2$ 보정이 생깁니다.

---

## 컴퓨팅 관점의 입력과 출력

Ito lemma 자체는 수식 변환 도구입니다.

```text
입력: SDE dX = a dt + b dW, 변환 함수 f(x,t)
출력: transformed SDE df = drift_new dt + diffusion_new dW
```

symbolic algebra로 미분항을 계산할 수 있고, simulation에서는 변환된 process의 drift/diffusion을 검증할 수 있습니다.

---

## 간단한 toy 예시

$X_t=W_t$이고 $f(X)=X^2$라면 $a=0$, $b=1$, $f_x=2X$, $f_{xx}=2$입니다.

$$
d(W_t^2)=dt+2W_tdW_t
$$

일반 chain rule이라면 $2W_tdW_t$만 나올 것 같지만, Ito lemma에서는 $dt$ 항이 추가됩니다.

---

## 퀀트 트레이딩에서의 역할

- [[Black-Scholes Model]] 유도
- option price $V(S,t)$의 SDE 계산
- log return dynamics 유도
- numeraire change, risk-neutral pricing의 기초

---

## 관련 노트

- [[Stochastic Differential Equation]]
- [[Brownian Motion]]
- [[Black-Scholes Model]]
- [[Quant Trading]]

---

## 내가 기억해야 할 핵심

- Ito lemma는 SDE의 chain rule입니다.
- $dW_t$의 제곱이 $dt$ scale로 남기 때문에 second derivative 보정항이 생깁니다.
- Black-Scholes 유도에서 옵션 가격 함수 $V(S,t)$의 변화량을 계산하는 데 필수입니다.
