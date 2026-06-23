---
type: learning-note
topic: chunking
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Chunking

## 한 줄 요약

[[Chunking]]은 긴 문서를 검색과 LLM context 입력에 적합한 작은 조각으로 나누는 과정이다.

---

## 핵심 개념

- chunk가 너무 작으면 문맥이 끊기고, 너무 크면 검색 정확도가 떨어진다.
- 문단, heading, 수식, 표, figure caption 같은 논리 단위를 보존하는 것이 중요하다.
- RAG 품질은 chunk 설계에 크게 좌우된다.

---

## 수학적 형태

문서 $D$를 여러 chunk로 나눈다.

$$
D \rightarrow \{c_1,c_2,\cdots,c_n\}
$$

각 chunk에 대해 embedding을 만든다.

$$
z_i=E(c_i)
$$

---

## 컴퓨팅 관점의 입력과 출력

입력은 긴 문서 string이고 출력은 chunk string list와 metadata list입니다. 예: 20페이지 PDF → 150개 chunk.

---

## 간단한 toy 예시

Transformer 문서 한 개를 heading 기준으로 나누면 `Self-Attention`, `QKV`, `FFN` 같은 chunk들이 만들어져 RAG 검색 단위가 됩니다.

---

## 관련 노트

- [[Retrieval-Augmented Generation (RAG)]]
- [[Embedding Model]]
- [[Semantic Search]]

---

## 내가 기억해야 할 핵심

- Chunking는 Chunking은 긴 문서를 검색과 LLM context 입력에 적합한 작은 조각으로 나누는 과정이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
