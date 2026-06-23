---
type: learning-note
topic: embedding-model
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Embedding Model

## 한 줄 요약

[[Embedding Model]]은 텍스트, 이미지, 코드 같은 입력을 의미를 담은 벡터로 변환하는 모델이다.

---

## 핵심 개념

- 비슷한 의미의 입력은 embedding space에서 가까운 벡터가 되도록 학습된다.
- RAG에서는 문서 chunk와 질문을 같은 벡터 공간에 매핑한다.
- 검색, clustering, 추천, 중복 제거에 사용된다.

---

## 수학적 형태

문서 $d$와 질문 $q$를 embedding 함수 $E$로 변환한다.

$$
z_d=E(d),\quad z_q=E(q)
$$

유사도는 cosine similarity로 자주 계산한다.

$$
\cos(z_q,z_d)=\frac{z_q\cdot z_d}{\|z_q\|\|z_d\|}
$$

---

## 컴퓨팅 관점의 입력과 출력

입력은 문자열 list이고 출력은 dense vector matrix입니다. 예: 문서 1000개, embedding 차원 768이면 `embeddings.shape=[1000,768]`입니다.

---

## 간단한 toy 예시

`pump failure`와 `pump malfunction`은 단어가 달라도 embedding vector가 가깝게 나와 semantic search에서 함께 검색될 수 있습니다.

---

## 관련 노트

- [[Retrieval-Augmented Generation (RAG)]]
- [[Vector Database]]
- [[Semantic Search]]
- [[Token Embedding]]

---

## 내가 기억해야 할 핵심

- Embedding Model는 Embedding Model은 텍스트, 이미지, 코드 같은 입력을 의미를 담은 벡터로 변환하는 모델이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
