---
type: learning-note
topic: hallucination
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Hallucination

## 한 줄 요약

[[Hallucination]]은 LLM이 실제 근거가 없거나 틀린 내용을 그럴듯하게 생성하는 현상이다.

---

## 핵심 개념

- 모델은 확률적으로 그럴듯한 다음 token을 생성하므로 사실성 보장이 자동으로 되지 않는다.
- RAG, citation, tool use, verification이 hallucination을 줄이는 데 도움이 된다.
- 연구/공학 문서에서는 단위, 조건, citation 오류가 특히 위험하다.

---

## 컴퓨팅 관점의 입력과 출력

입력은 LLM prompt이고 출력은 생성 문자열입니다. hallucination은 출력 문자열 안에 근거 없는 claim이 포함되는 실패 모드입니다.

---

## 간단한 toy 예시

문서에 없는 CHF correlation 이름을 모델이 그럴듯하게 만들어 말하면 hallucination입니다.

---

## 관련 노트

- [[Large Language Model]]
- [[Retrieval-Augmented Generation (RAG)]]
- [[Prompt Engineering]]

---

## 내가 기억해야 할 핵심

- Hallucination는 Hallucination은 LLM이 실제 근거가 없거나 틀린 내용을 그럴듯하게 생성하는 현상이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
