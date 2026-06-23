---
type: learning-note
topic: prompt-engineering
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Prompt Engineering

## 한 줄 요약

[[Prompt Engineering]]은 LLM이 원하는 방식으로 답하도록 지시문, context, 예시, 출력 형식을 설계하는 작업이다.

---

## 핵심 개념

- RAG에서는 검색된 근거 문서를 prompt에 어떻게 넣을지가 중요하다.
- 제약 조건, citation 요구, 모르면 모른다고 답하기 같은 지시가 hallucination을 줄인다.
- prompt는 모델의 능력을 바꾸기보다 주어진 context를 사용하는 방식을 조절한다.

---

## 수학적 형태

RAG prompt의 기본 구조는 다음과 같다.

```text
System instruction
User question
Retrieved context
Output format constraints
```

---

## 컴퓨팅 관점의 입력과 출력

입력은 instruction, user question, context, examples이고 출력은 LLM에 넣는 최종 prompt string입니다.

---

## 간단한 toy 예시

RAG에서 `근거에 있는 내용만 답하고, 없으면 모른다고 말하라`는 지시를 넣어 hallucination을 줄일 수 있습니다.

---

## 관련 노트

- [[Large Language Model]]
- [[Retrieval-Augmented Generation (RAG)]]
- [[Hallucination]]

---

## 내가 기억해야 할 핵심

- Prompt Engineering는 Prompt Engineering은 LLM이 원하는 방식으로 답하도록 지시문, context, 예시, 출력 형식을 설계하는 작업이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
