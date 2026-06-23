---
type: learning-note
topic: vision-transformer
tags:
  - machine-learning
  - concept-note
created: 2026-06-23
status: draft
---
# Vision Transformer

## 한 줄 요약

[[Vision Transformer]]는 이미지를 patch sequence로 나누어 Transformer에 입력하는 computer vision 모델이다.

---

## 핵심 개념

- 이미지를 작은 patch로 자르고 각 patch를 token처럼 취급한다.
- CNN의 local inductive bias 대신 self-attention으로 patch 간 관계를 학습한다.
- 대규모 데이터와 사전학습에서 강력한 성능을 보인다.

---

## 수학적 형태

이미지 $H\times W$를 $P\times P$ patch로 나누면 token 수는 다음과 같다.

$$
N=\frac{HW}{P^2}
$$

각 patch를 펼친 뒤 linear projection하여 embedding으로 만든다.

---

## 컴퓨팅 관점의 입력과 출력

입력 이미지는 `[batch, height, width, channels]` 또는 `[batch, channels, height, width]`이고, patch로 나눈 뒤 `[batch, num_patches, d_model]` token sequence가 됩니다.

---

## 간단한 toy 예시

32×32 이미지를 8×8 patch로 자르면 16개 patch token이 생깁니다. ViT는 이 16개 token 사이 self-attention을 계산합니다.

---

## 관련 노트

- [[Transformer Model]]
- [[Self-Attention]]
- [[Token Embedding]]
- [[Convolutional Neural Network]]

---

## 내가 기억해야 할 핵심

- Vision Transformer는 Vision Transformer는 이미지를 patch sequence로 나누어 Transformer에 입력하는 computer vision 모델이다.
- 수학식만 외우기보다 어떤 정보를 어떤 표현 공간에서 처리하는지 이해하는 것이 중요하다.
