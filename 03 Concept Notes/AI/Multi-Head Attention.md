---
type: learning-note
topic: multi-head-attention
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Multi-Head Attention

## 한 줄 요약

[[Multi-Head Attention]]은 attention을 여러 head로 나누어 서로 다른 관계를 병렬로 학습하는 방법이다.

---

## 핵심 개념

- 하나의 attention head만 쓰면 한 종류의 관계에 치우칠 수 있다.
- 여러 head는 문법 관계, 의미 관계, 위치 관계, 장거리 의존성을 서로 다르게 볼 수 있다.
- 각 head의 출력을 concat한 뒤 다시 선형 변환한다.

---

## 수학적 형태

각 head는 독립적으로 attention을 계산한다.

$$
head_i=Attention(QW_i^Q,KW_i^K,VW_i^V)
$$

전체 출력은 다음과 같다.

$$
MultiHead(Q,K,V)=Concat(head_1,\cdots,head_h)W^O
$$

---

## 컴퓨팅 관점의 입력과 출력

입력은 `[batch, seq_len, d_model]`이고 head 수가 8이면 내부적으로 head당 차원 `d_head=d_model/8`로 나뉩니다. 출력은 다시 concat되어 `[batch, seq_len, d_model]`이 됩니다.

---

## 간단한 toy 예시

head 1은 주어-동사 관계를 보고, head 2는 대명사 참조를 보고, head 3은 가까운 token을 보는 식으로 서로 다른 관계를 나눠 볼 수 있습니다.

---

## 관련 노트

- [[Attention Mechanism]]
- [[Self-Attention]]
- [[Query-Key-Value]]
- [[Transformer Model]]

---

## 내가 기억해야 할 핵심

- Multi-Head Attention는 Multi-Head Attention은 attention을 여러 head로 나누어 서로 다른 관계를 병렬로 학습하는 방법이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
