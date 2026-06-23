---
type: learning-note
topic: transformer-model
tags:
  - machine-learning
  - deep-learning
  - transformer
  - attention
created: 2026-06-23
status: draft
---

# Transformer Model

## 한 줄 요약

[[Transformer Model]]은 [[Recurrent Neural Network]]처럼 순차적으로 토큰을 처리하지 않고, [[Self-Attention]]을 이용해 입력 시퀀스 안의 모든 토큰 관계를 병렬적으로 계산하는 딥러닝 모델 구조이다.

대표 논문은 Vaswani et al.의 "Attention Is All You Need"이며, 현대 [[Large Language Model]], [[BERT]], [[GPT]], [[Vision Transformer]]의 기반 구조가 되었다. 또한 [[Retrieval-Augmented Generation (RAG)]]에서는 Transformer 기반 LLM이 검색된 외부 지식을 읽고 답변을 생성하는 엔진 역할을 한다.

---

## 왜 Transformer가 중요한가?

기존의 [[RNN]]이나 [[LSTM]]은 문장을 앞에서 뒤로 순차적으로 처리한다. 이 방식은 자연스러운 순서 정보를 잘 다룰 수 있지만, 긴 문맥을 기억하기 어렵고 병렬화가 제한된다.

Transformer는 다음 아이디어로 이 문제를 해결한다.

1. 모든 토큰을 동시에 입력한다.
2. 각 토큰이 다른 모든 토큰을 얼마나 참고해야 하는지 [[Attention Mechanism]]으로 계산한다.
3. 순서 정보는 별도의 [[Positional Encoding]]으로 넣는다.
4. 행렬 연산 중심 구조라 GPU/TPU에서 병렬 계산이 효율적이다.

즉, Transformer의 핵심은 "순차 처리 대신 관계 계산"이다.

---

## 전체 구조

원래 Transformer는 [[Encoder-Decoder Architecture]]로 제안되었다.

- [[Transformer Encoder]]: 입력 문장의 표현을 만든다.
- [[Transformer Decoder]]: 이전 출력과 encoder 정보를 참고해 다음 토큰을 생성한다.

하지만 실제 모델 계열은 목적에 따라 구조가 나뉜다.

| 계열 | 구조 | 예시 | 주 용도 |
|---|---|---|---|
| Encoder-only | Encoder만 사용 | [[BERT]] | 분류, 검색, 임베딩 |
| Decoder-only | Decoder만 사용 | [[GPT]] | 텍스트 생성, LLM |
| Encoder-Decoder | 둘 다 사용 | [[T5]], original Transformer | 번역, 요약 |

---

## 핵심 구성 요소

### 1. Token Embedding

입력 문장은 먼저 토큰으로 나뉜다. 각 토큰은 정수 ID로 변환되고, 다시 고차원 벡터인 [[Token Embedding]]으로 변환된다.

예:

```text
"The reactor is stable"
→ ["The", "reactor", "is", "stable"]
→ [101, 2387, 2003, 6540]
→ embedding vectors
```

LLM에서 단어가 아니라 subword token을 쓰는 이유는 희귀 단어와 새로운 조합을 더 잘 처리하기 위해서이다.

---

### 2. Positional Encoding

Transformer는 입력을 한 번에 보기 때문에 토큰의 순서를 자체적으로 알 수 없다. 따라서 각 토큰 위치에 대한 정보를 추가해야 한다.

이 역할을 하는 것이 [[Positional Encoding]]이다.

입력 표현은 대략 다음과 같다.

```text
input representation = token embedding + positional encoding
```

초기 Transformer 논문에서는 sin/cos 기반 위치 인코딩을 사용했다. 이후 모델들은 학습 가능한 위치 임베딩, [[Rotary Positional Embedding]], ALiBi 등 다양한 방식을 사용한다.

---

### 3. Self-Attention

[[Self-Attention]]은 Transformer의 가장 중요한 부분이다. 한 문장 안에서 각 토큰이 다른 토큰을 얼마나 참고해야 하는지 계산한다.

예를 들어:

```text
"The pump failed because it overheated."
```

여기서 `it`은 `pump`를 가리킨다. Self-attention은 `it` 토큰이 `pump` 토큰에 높은 attention score를 주도록 학습할 수 있다.

---

## Query, Key, Value

Self-attention은 각 토큰 벡터를 세 가지 벡터로 변환한다.

- [[Query]]: 내가 찾고 싶은 정보
- [[Key]]: 내가 어떤 정보를 가지고 있는지 나타내는 색인
- [[Value]]: 실제로 전달할 정보

Attention score는 Query와 Key의 유사도로 계산한다.

수식으로는 다음과 같다.

$$
Attention(Q, K, V) = softmax\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

의미:

1. $QK^T$: 각 토큰이 다른 토큰과 얼마나 관련 있는지 계산
2. $\sqrt{d_k}$: 차원이 커질 때 dot product가 너무 커지는 것을 방지
3. softmax: 관련도를 확률처럼 정규화
4. $V$: 관련도에 따라 실제 정보를 가중합

---

## Multi-Head Attention

[[Multi-Head Attention]]은 attention을 여러 개의 head로 나누어 병렬로 수행하는 방식이다.

한 head는 문법적 관계를 볼 수 있고, 다른 head는 의미적 관계를 볼 수 있으며, 또 다른 head는 장거리 의존성을 볼 수 있다.

즉, 여러 관점에서 토큰 간 관계를 동시에 학습한다.

---

## Feed-Forward Network

Attention layer 뒤에는 각 토큰 위치마다 독립적으로 적용되는 [[Feed-Forward Network]]가 있다.

일반적으로 다음 구조를 가진다.

```text
Linear → Activation → Linear
```

LLM에서는 이 부분이 모델 파라미터의 큰 비중을 차지한다. 최근 모델에서는 [[Mixture of Experts]] 구조를 FFN에 적용하기도 한다.

---

## Residual Connection과 Layer Normalization

Transformer block은 안정적인 학습을 위해 다음을 사용한다.

- [[Residual Connection]]: 입력을 출력에 더해 gradient 흐름을 좋게 만든다.
- [[Layer Normalization]]: activation scale을 안정화한다.

간단히 표현하면:

```text
x = x + Attention(LayerNorm(x))
x = x + FFN(LayerNorm(x))
```

최근 LLM은 보통 Pre-LN 구조를 많이 사용한다.

---

## Transformer Block

하나의 [[Transformer Block]]은 보통 다음으로 구성된다.

1. LayerNorm
2. Multi-Head Self-Attention
3. Residual Connection
4. LayerNorm
5. Feed-Forward Network
6. Residual Connection

이 block을 여러 층 쌓으면 깊은 Transformer 모델이 된다.

---

## 학습 방식

Transformer의 학습 방식은 모델 목적에 따라 다르다.

### GPT 계열: Causal Language Modeling

[[GPT]] 같은 decoder-only 모델은 이전 토큰들만 보고 다음 토큰을 예측한다.

```text
P(next token | previous tokens)
```

이를 위해 미래 토큰을 보지 못하게 하는 [[Causal Mask]]를 사용한다.

### BERT 계열: Masked Language Modeling

[[BERT]]는 문장 중 일부 토큰을 가리고, 가려진 토큰을 맞히도록 학습한다.

```text
The reactor [MASK] stable.
```

이 방식은 양방향 문맥 이해에 유리하다.

---

## 장점

- 긴 문맥에서 토큰 간 관계를 직접 계산할 수 있다.
- RNN보다 병렬화가 쉽다.
- scaling law에 따라 모델 크기와 데이터가 커질수록 성능이 좋아지는 경향이 있다.
- 언어뿐 아니라 이미지, 음성, 단백질, 시계열 데이터에도 적용 가능하다.

---

## 한계

- Attention 계산량이 시퀀스 길이 $n$에 대해 $O(n^2)$이다.
- 긴 문맥을 처리할 때 메모리 사용량이 크다.
- 대규모 학습에는 막대한 데이터와 연산 자원이 필요하다.
- attention weight가 항상 인간이 이해하는 설명이라고 보기는 어렵다.

---

## 열수력/HPC 연구와 연결해서 생각하기

Transformer 자체는 원자력 열수력 모델은 아니지만, 연구적으로 연결할 수 있는 지점이 있다.

- [[Surrogate Model]]: 고비용 CFD 계산을 대체하는 빠른 근사 모델
- [[Fourier Neural Operator (FNO)]]: PDE solution operator를 학습하는 field-to-field surrogate model
- [[Reduced Order Model]]: 복잡한 물리장을 저차원 표현으로 축약
- [[Scientific Machine Learning]]: 물리 법칙과 신경망을 결합
- [[Spatiotemporal Modeling]]: 시간에 따라 변하는 유동장 예측
- [[Uncertainty Quantification]]: 예측 불확실성 추정
- [[High Performance Computing]]: 대규모 모델 학습과 대규모 시뮬레이션의 공통 기반

예를 들어, 열수력 시뮬레이션의 압력장, 속도장, 온도장 데이터를 token처럼 표현하고, Transformer로 시공간 상관관계를 학습하는 연구 방향을 생각할 수 있다.

---

## 내가 기억해야 할 핵심

- Transformer는 RNN을 대체한 sequence modeling 구조이다.
- 핵심은 [[Self-Attention]]이다.
- Query, Key, Value는 attention 계산의 기본 단위이다.
- 위치 정보는 [[Positional Encoding]]으로 넣는다.
- GPT는 decoder-only, BERT는 encoder-only이다.
- 계산량은 강력하지만 $O(n^2)$ attention 비용이 한계이다.
- HPC 관점에서는 행렬 곱 중심 구조라 병렬화와 GPU 활용에 잘 맞는다.

---

## 컴퓨팅 관점의 입력과 출력

Transformer의 입력은 token ID 배열입니다. 예를 들어 batch size 4, sequence length 128이면:

```text
input_ids.shape = [4, 128]
```

embedding layer를 거치면:

```text
hidden.shape = [4, 128, d_model]
```

언어 모델의 출력은 보통 각 위치에서 vocabulary 전체에 대한 logit입니다.

```text
logits.shape = [4, 128, vocab_size]
```

즉 Transformer는 token sequence를 hidden vector sequence로 바꾸고, 목적에 따라 다음 token, class label, embedding vector 등을 출력합니다.

---

## 간단한 toy 예시

문장 `The pump failed`가 4개 token으로 나뉘고 embedding 차원이 8이라고 해보겠습니다.

```text
input_ids.shape = [1, 4]
hidden.shape    = [1, 4, 8]
```

self-attention은 각 token이 나머지 3개 token을 얼마나 참고할지 `4×4` attention matrix로 계산합니다.

---

## 관련 노트

- [[Attention Mechanism]]
- [[Self-Attention]]
- [[Multi-Head Attention]]
- [[Positional Encoding]]
- [[Query-Key-Value]]
- [[Transformer Encoder]]
- [[Transformer Decoder]]
- [[GPT]]
- [[BERT]]
- [[Large Language Model]]
- [[Retrieval-Augmented Generation (RAG)]]
- [[Scientific Machine Learning]]
- [[Surrogate Model]]
- [[Fourier Neural Operator (FNO)]]
- [[High Performance Computing]]

---

## 다음에 확장할 질문

- 왜 attention score를 $\sqrt{d_k}$로 나누는가?
- RoPE는 기존 positional encoding과 어떻게 다른가?
- Transformer를 CFD field prediction에 적용할 때 tokenization은 어떻게 해야 하는가?
- 긴 시계열/공간 격자 데이터에서 attention의 $O(n^2)$ 비용을 어떻게 줄일 수 있는가?
- Physics-informed Transformer는 기존 PINN과 무엇이 다른가?
