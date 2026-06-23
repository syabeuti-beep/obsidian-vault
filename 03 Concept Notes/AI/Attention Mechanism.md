---
type: learning-note
topic: attention-mechanism
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Attention Mechanism

## 한 줄 요약

[[Attention Mechanism]]은 입력 요소들 중 현재 계산에 중요한 요소에 더 큰 가중치를 주어 정보를 모으는 방법이다.

---

## 핵심 개념

- 문장, 시계열, 공간 field처럼 여러 위치의 정보가 있을 때 모든 위치가 같은 중요도를 갖지는 않는다.
- Attention은 query와 key의 유사도를 계산해 weight를 만들고, 그 weight로 value를 가중합한다.
- [[Transformer Model]]에서는 attention이 token 사이의 관계를 직접 계산하는 핵심 연산이다.

---

## 수학적 형태

가장 기본적인 scaled dot-product attention은 다음과 같다.

$$
Attention(Q,K,V)=softmax\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

여기서 $QK^T$는 query와 key의 유사도, softmax는 가중치 정규화, $V$는 실제로 합쳐지는 정보이다.

---

## 컴퓨팅 관점의 입력과 출력

입력은 query/key/value tensor이고 출력은 value들을 가중합한 tensor입니다. 예를 들어 `[batch, seq_len, d_model]` hidden state에서 Q,K,V를 만들면 attention 출력도 보통 `[batch, seq_len, d_model]`입니다.

---

## 간단한 toy 예시

seq_len=3인 문장 `[pump, failed, overheated]`에서 `overheated` token의 query가 `pump` key와 높은 점수를 가지면 pump의 value가 출력에 크게 반영됩니다.

---

## 관련 노트

- [[Self-Attention]]
- [[Query-Key-Value]]
- [[Transformer Model]]
- [[Multi-Head Attention]]

---

## 내가 기억해야 할 핵심

- Attention Mechanism는 Attention Mechanism은 입력 요소들 중 현재 계산에 중요한 요소에 더 큰 가중치를 주어 정보를 모으는 방법이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
