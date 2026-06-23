---
type: learning-note
topic: value
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Value

## 한 줄 요약

[[Value]]는 attention에서 실제로 전달되고 가중합되는 정보 벡터이다.

---

## 핵심 개념

- Query-Key 유사도는 어떤 위치를 얼마나 볼지 결정하고, Value는 실제로 가져오는 내용이다.
- 각 token hidden vector에 $W_V$를 곱해 만든다.
- attention weight와 Value의 weighted sum이 출력이 된다.

---

## 수학적 형태

$$
V=XW_V
$$

$$
Output=Attention\ weight \cdot V
$$

---

## 컴퓨팅 관점의 입력과 출력

Value tensor shape은 `[batch, heads, seq_len, d_head]`입니다. attention weight가 정해진 뒤 실제로 weighted sum되는 정보입니다.

---

## 간단한 toy 예시

query-key 점수로 `pump`를 0.8만큼 참고하기로 했다면, pump token의 value vector가 0.8 비중으로 출력에 섞입니다.

---

## 관련 노트

- [[Query]]
- [[Key]]
- [[Query-Key-Value]]
- [[Attention Mechanism]]

---

## 내가 기억해야 할 핵심

- Value는 Value는 attention에서 실제로 전달되고 가중합되는 정보 벡터이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
