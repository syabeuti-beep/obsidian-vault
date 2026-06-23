---
type: learning-note
topic: token-embedding
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Token Embedding

## 한 줄 요약

[[Token Embedding]]은 discrete token ID를 neural network가 계산할 수 있는 연속 벡터로 바꾸는 lookup table이다.

---

## 핵심 개념

- 단어, subword, code token 등은 먼저 정수 ID가 된다.
- embedding matrix에서 해당 ID의 행을 꺼내 dense vector로 사용한다.
- 의미적으로 비슷한 token들은 학습 후 embedding space에서 가까운 방향을 가질 수 있다.

---

## 수학적 형태

vocabulary 크기가 $V$, embedding 차원이 $d$이면 embedding matrix는 다음과 같다.

$$
E\in\mathbb{R}^{V\times d}
$$

ID가 $i$인 token의 embedding은 $E_i$이다.

---

## 컴퓨팅 관점의 입력과 출력

입력은 integer token ID tensor `[batch, seq_len]`이고 출력은 float embedding tensor `[batch, seq_len, d_model]`입니다.

---

## 간단한 toy 예시

`pump`의 token ID가 4821이고 embedding 차원이 6이면 embedding table의 4821번째 row, 예를 들어 `[0.1,-0.3,...]`를 가져옵니다.

---

## 관련 노트

- [[Transformer Model]]
- [[Large Language Model]]
- [[Positional Encoding]]

---

## 내가 기억해야 할 핵심

- Token Embedding는 Token Embedding은 discrete token ID를 neural network가 계산할 수 있는 연속 벡터로 바꾸는 lookup table이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
