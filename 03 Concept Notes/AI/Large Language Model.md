---
type: learning-note
topic: large-language-model
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Large Language Model

## 한 줄 요약

[[Large Language Model]]은 대규모 텍스트 데이터와 큰 Transformer 구조로 학습된 언어 모델이다.

---

## 핵심 개념

- 대부분 현대 LLM은 decoder-only [[Transformer Model]] 기반이다.
- 언어 생성, 요약, 코딩, 추론, 도구 사용 등 다양한 작업을 수행한다.
- 지식은 파라미터와 context 안에 나뉘어 존재하며, 최신/외부 지식에는 [[Retrieval-Augmented Generation (RAG)]]가 자주 사용된다.

---

## 수학적 형태

autoregressive LLM은 다음 token 분포를 반복적으로 예측한다.

$$
P(x_1,\cdots,x_T)=\prod_{t=1}^T P(x_t|x_{<t})
$$

---

## 컴퓨팅 관점의 입력과 출력

입력은 token ID sequence이고 출력은 다음 token 확률분포입니다. API 관점에서는 문자열을 넣고 문자열을 받지만 내부에서는 `[batch, seq_len] → [batch, seq_len, vocab_size]` 계산입니다.

---

## 간단한 toy 예시

프롬프트 `FNO를 설명해줘`가 token IDs로 바뀌고, 모델은 다음 token을 반복 생성해 답변 문자열을 만듭니다.

---

## 관련 노트

- [[Transformer Model]]
- [[GPT]]
- [[Retrieval-Augmented Generation (RAG)]]
- [[Prompt Engineering]]

---

## 내가 기억해야 할 핵심

- Large Language Model는 Large Language Model은 대규모 텍스트 데이터와 큰 Transformer 구조로 학습된 언어 모델이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
