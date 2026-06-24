---
type: concept
topic: process-rnd-metrics-doe
tags: [semiconductor-process-rnd, doe, process-metrics]
created: 2026-06-24
status: evergreen
---

# 공정 R&D 엔지니어가 보는 지표와 실험 설계

## 공정 R&D의 핵심 질문

공정 R&D는 단순히 “잘 깎인다”가 아니라 다음 질문에 답하는 일입니다.

> 주어진 장비와 재료에서, 제품 요구 조건을 만족하도록 공정 recipe를 어떤 방향으로 조정해야 하는가?

## 대표 성능 지표

| 지표 | 의미 | 측정/관찰 예시 |
|---|---|---|
| Etch rate | 단위 시간당 제거 두께 | nm/min |
| Selectivity | 목표막 대비 다른 막 식각률 비 | oxide:Si = 50:1 |
| CD bias | 식각 전후 선폭 변화 | nm |
| Profile angle | 옆벽 각도 | SEM cross-section |
| Uniformity | 웨이퍼 내 편차 | center-edge map |
| Roughness | 표면/옆벽 거칠기 | SEM, AFM |
| Defect | particle, residue, bridge | inspection tool |
| Electrical impact | 누설, 저항, threshold 변화 | device test |

## Recipe 변수와 응답 변수

```text
입력 변수 X
= [source power, bias power, pressure, gas ratio, flow, chuck temperature, time]

응답 변수 Y
= [etch rate, selectivity, CD bias, profile angle, defect, damage]
```

공정 R&D는 이 함수 `Y = f(X, chamber state, wafer stack)`를 실험적으로 학습하고 물리적으로 해석하는 일입니다.

## DOE의 직관

DOE는 Design of Experiments, 즉 실험계획법입니다. 모든 조합을 무작정 실험하지 않고, 영향이 큰 변수를 효율적으로 찾습니다.

예시:

| 실험 | Pressure | Bias power | O2 ratio | 관찰 |
|---|---:|---:|---:|---|
| A | 낮음 | 낮음 | 낮음 | 식각률 낮음, profile 양호 |
| B | 낮음 | 높음 | 낮음 | profile 좋지만 damage 증가 |
| C | 높음 | 낮음 | 높음 | polymer 감소, undercut 가능 |
| D | 높음 | 높음 | 높음 | 빠르지만 선택비 악화 가능 |

중요한 것은 “한 변수의 주효과”뿐 아니라 **상호작용**입니다. 예를 들어 O2 ratio 효과는 fluorocarbon gas와 bias 조건에 따라 달라질 수 있습니다.

## 공정 최적화는 trade-off 문제

| 원하는 것 | 동시에 생기는 위험 |
|---|---|
| 식각률 증가 | 선택비 저하, damage 증가 |
| 수직 profile | mask erosion, charging issue |
| 높은 선택비 | residue, 낮은 식각률 |
| 강한 polymer 보호 | bottom residue, etch stop |
| 낮은 damage | 식각 불완전, 낮은 생산성 |

따라서 면접에서는 “A를 올리면 B가 좋아진다”보다 “A를 올리면 B는 좋아지지만 C가 나빠질 수 있어 최적점을 찾아야 한다”는 식으로 말하는 것이 좋습니다.

## 불량 원인 분석 흐름

1. 계측 결과로 문제를 구체화한다.
   - CD가 커졌는가, 작아졌는가?
   - profile이 taper인가 bowing인가?
   - defect인지 electrical fail인지?
2. recipe 변화와 chamber 상태를 확인한다.
3. plasma chemistry, ion energy, temperature, wafer loading 관점에서 가설을 세운다.
4. DOE 또는 split lot으로 검증한다.
5. 개선 조건의 재현성과 공정 window를 확인한다.

## 공정 window

공정 window는 조건이 조금 흔들려도 제품 요구를 만족하는 안정 범위입니다. R&D에서는 단일 최적점뿐 아니라, 양산 가능성을 고려해 넓고 안정적인 window를 찾는 것이 중요합니다.

## 이 묶음 안의 관련 노트

- [[웨이퍼에서 소자까지 반도체 공정 큰 그림]]
- [[플라즈마 식각 장비와 조절 변수]]
- [[SK하이닉스 공정 R&D 지원자 관점 정리]]
