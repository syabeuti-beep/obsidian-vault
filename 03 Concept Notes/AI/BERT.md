---
type: learning-note
topic: bert
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# BERT

## 한 줄 요약

[[BERT]]는 Transformer encoder만 사용해 양방향 문맥 표현을 학습하는 language model 계열이다.

---

## 핵심 개념

- 대표 학습 방식은 Masked Language Modeling이다.
- 문장 분류, 질의응답, 검색 embedding 등 이해 중심 태스크에 강하다.
- GPT와 달리 기본 BERT는 autoregressive generation보다 representation learning에 초점이 있다.

---

## 수학적 형태

Masked Language Modeling은 일부 token을 [MASK]로 가리고 원래 token을 예측한다.

$$
P(x_{masked}|x_{visible})
$$

---

## 컴퓨팅 관점의 입력과 출력

입력은 token IDs `[batch, seq_len]`, 출력은 token별 contextual embedding `[batch, seq_len, hidden]` 또는 classification vector `[batch, hidden]`입니다.

---

## 간단한 toy 예시

문장 분류에서는 `[CLS]` token의 출력 vector 하나를 classifier에 넣어 긍정/부정을 예측합니다.

---

## 관련 노트

- [[Transformer Encoder]]
- [[Transformer Model]]
- [[Large Language Model]]
- [[Token Embedding]]

---

## 내가 기억해야 할 핵심

- BERT는 BERT는 Transformer encoder만 사용해 양방향 문맥 표현을 학습하는 language model 계열이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
