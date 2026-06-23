---
type: learning-note
topic: transformer-encoder
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Transformer Encoder

## 한 줄 요약

[[Transformer Encoder]]는 입력 sequence 전체를 양방향으로 보고 각 token의 contextual representation을 만드는 Transformer 구성 요소이다.

---

## 핵심 개념

- encoder에서는 각 token이 앞뒤 모든 token을 볼 수 있다.
- BERT는 encoder-only Transformer의 대표 예시이다.
- 분류, 검색, embedding 생성, 문서 이해 등에 많이 사용된다.

---

## 수학적 형태

encoder block은 보통 다음을 반복한다.

$$
x'=x+SelfAttention(LayerNorm(x))
$$

$$
y=x'+FFN(LayerNorm(x'))
$$

---

## 컴퓨팅 관점의 입력과 출력

입력은 token IDs `[batch, seq_len]` 또는 embedding `[batch, seq_len, d_model]`이고 출력은 contextual embedding `[batch, seq_len, d_model]`입니다.

---

## 간단한 toy 예시

BERT encoder에 `reactor is stable`을 넣으면 각 token embedding이 문맥을 반영한 vector로 바뀝니다.

---

## 관련 노트

- [[Transformer Model]]
- [[BERT]]
- [[Self-Attention]]
- [[Encoder-Decoder Architecture]]

---

## 내가 기억해야 할 핵심

- Transformer Encoder는 Transformer Encoder는 입력 sequence 전체를 양방향으로 보고 각 token의 contextual representation을 만드는 Transformer 구성 요소이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
