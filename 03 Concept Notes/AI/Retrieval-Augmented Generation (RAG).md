---
type: learning-note
topic: retrieval-augmented-generation
tags:
  - machine-learning
  - large-language-model
  - rag
  - information-retrieval
  - knowledge-base
created: 2026-06-23
status: draft
---

# Retrieval-Augmented Generation (RAG)

## 한 줄 요약

[[Retrieval-Augmented Generation (RAG)]]는 [[Large Language Model]]이 답변을 생성하기 전에 외부 지식 저장소에서 관련 문서를 검색하고, 그 검색 결과를 context로 넣어 답변하게 만드는 방법이다.

즉, RAG의 핵심은 "모델이 모든 지식을 파라미터 안에 외우게 하지 않고, 필요한 순간에 [[Information Retrieval]]로 가져와서 [[Transformer Model]] 기반 생성 모델에 주입하는 것"이다.

---

## 왜 RAG가 필요한가?

[[Large Language Model]]은 학습 데이터에서 배운 패턴을 바탕으로 답변하지만, 다음 한계가 있다.

1. 최신 정보나 사내/개인 문서처럼 학습 데이터에 없는 지식은 모를 수 있다.
2. 모델 파라미터 안의 지식은 출처 추적이 어렵다.
3. 잘 모르는 내용도 그럴듯하게 생성하는 [[Hallucination]] 문제가 있다.
4. 도메인 문서를 새로 반영하려고 매번 fine-tuning하기에는 비용이 크다.

RAG는 이 문제를 다음 방식으로 줄인다.

```text
질문 → 관련 문서 검색 → 검색 문서 + 질문을 LLM에 입력 → 근거 기반 답변 생성
```

---

## 기본 파이프라인

RAG 시스템은 보통 다음 단계로 구성된다.

### 1. 문서 수집

논문, 매뉴얼, 실험 기록, 코드 문서, 보고서, 위키, Notion/Obsidian 노트 등을 지식 소스로 모은다.

예시:

- 열수력 논문 PDF
- CFD 시뮬레이션 결과 설명 문서
- 코드 repository의 README와 API 문서
- 실험 조건과 benchmark 기록
- 개인 연구 노트와 [[Obsidian]] vault

### 2. Chunking

긴 문서를 LLM context에 그대로 넣기 어렵기 때문에 작은 단위로 나눈다. 이 과정을 [[Chunking]]이라고 한다.

좋은 chunk는 다음 조건을 만족해야 한다.

- 하나의 개념 또는 논리 단위를 유지한다.
- 너무 짧아서 맥락이 사라지지 않는다.
- 너무 길어서 검색 precision이 떨어지지 않는다.
- 표, 수식, 그림 캡션처럼 의미가 분산된 정보를 잘 보존한다.

### 3. Embedding 생성

각 chunk를 [[Embedding Model]]로 벡터로 변환한다. 의미가 비슷한 텍스트는 벡터 공간에서 가까워진다.

```text
문서 chunk → embedding vector
질문 → query embedding vector
```

### 4. Vector Database 저장

문서 embedding을 [[Vector Database]]에 저장한다. 대표적으로 FAISS, Chroma, Qdrant, Weaviate, Milvus 같은 도구를 사용할 수 있다.

저장할 때는 벡터만 넣는 것이 아니라 metadata도 함께 저장해야 한다.

- 문서 제목
- 원본 경로 또는 URL
- 페이지 번호
- 작성 날짜
- 실험 조건
- 논문 DOI
- 코드 버전

### 5. Retrieval

사용자 질문이 들어오면 질문도 embedding으로 바꾸고, vector database에서 가장 가까운 chunk를 찾는다.

기본 방식은 [[Semantic Search]]이다.

```text
query embedding과 가까운 top-k chunks 검색
```

필요하면 keyword search와 semantic search를 결합한 [[Hybrid Search]]를 사용한다.

### 6. Re-ranking

처음 검색된 chunk들이 항상 최적인 것은 아니다. [[Re-ranking]]은 검색 후보를 더 정밀한 모델로 다시 정렬하는 단계이다.

예를 들어 top-50을 빠르게 검색한 뒤, cross-encoder나 LLM으로 top-5를 다시 고를 수 있다.

### 7. Generation

마지막으로 검색된 문서 조각을 prompt에 넣고 [[Transformer Model]] 기반 LLM이 답변을 생성한다.

```text
System instruction
+ 사용자 질문
+ 검색된 근거 문서
→ 답변
```

이때 좋은 RAG 답변은 단순히 문서를 붙여넣는 것이 아니라, 질문에 맞춰 근거를 종합하고, 모르는 내용은 모른다고 말해야 한다.

---

## Transformer와의 관계

RAG 자체는 [[Transformer Model]]의 내부 구조가 아니라, Transformer 기반 LLM을 외부 지식 시스템과 연결하는 architecture pattern이다.

- [[Transformer Model]]: 입력 context를 읽고 다음 토큰을 생성하는 핵심 모델 구조
- [[Embedding Model]]: 질문과 문서를 벡터 공간에 매핑하는 모델
- [[Vector Database]]: 관련 문서를 빠르게 찾기 위한 저장소
- [[Prompt Engineering]]: 검색 결과를 LLM이 잘 사용하도록 context를 구성하는 방법

즉, Transformer가 "읽고 생성하는 엔진"이라면, RAG는 "필요한 자료를 찾아 엔진에 넣어주는 검색-생성 workflow"이다.

---

## RAG와 Fine-tuning 비교

| 접근 | 목적 | 장점 | 한계 |
|---|---|---|---|
| RAG | 외부 지식을 실시간으로 참조 | 최신 문서 반영 쉬움, 출처 제공 가능 | 검색 품질에 크게 의존 |
| Fine-tuning | 모델의 행동 양식 또는 특정 패턴 학습 | 출력 스타일/형식/태스크 적응에 강함 | 지식 업데이트 비용 큼 |
| Prompting | context 안에서 지시 | 빠르고 단순 | context 길이와 일관성 한계 |

일반적으로 "새 지식을 넣고 싶다"면 먼저 RAG를 고려하고, "답변 방식이나 작업 수행 능력을 바꾸고 싶다"면 fine-tuning을 고려한다.

---

## RAG 품질을 좌우하는 요소

### Retrieval 품질

검색 단계에서 관련 문서를 못 찾으면 LLM이 아무리 좋아도 답변이 나빠진다. 이를 "garbage in, garbage out" 문제로 볼 수 있다.

중요한 질문:

- query가 적절히 확장되었는가?
- top-k가 너무 작거나 크지 않은가?
- chunk 크기가 적절한가?
- keyword 기반 검색과 semantic search를 결합해야 하는가?
- metadata filtering이 필요한가?

### Context 구성

검색된 chunk를 prompt에 넣을 때 순서와 형식이 중요하다.

- 가장 관련 높은 문서를 앞에 둘 것
- 출처를 명시할 것
- 서로 충돌하는 근거가 있으면 표시할 것
- 질문과 무관한 chunk는 제거할 것

### 답변 생성 제약

RAG prompt에는 보통 다음 제약을 넣는다.

```text
제공된 context에 근거해서만 답하라.
근거가 부족하면 모른다고 말하라.
중요한 주장에는 출처를 표시하라.
```

이는 [[Hallucination]]을 줄이는 데 도움이 되지만, 완전히 제거하지는 못한다.

---

## 평가 방법

RAG 시스템은 LLM 답변만 평가하면 안 되고, retrieval과 generation을 분리해서 봐야 한다.

### Retrieval 평가

- Recall@k: 정답 근거가 top-k 안에 들어왔는가?
- Precision@k: 검색된 chunk 중 실제 관련 있는 비율은 얼마인가?
- MRR: 정답 근거가 얼마나 높은 순위에 있는가?

### Generation 평가

- Faithfulness: 답변이 검색된 근거에 충실한가?
- Answer relevance: 질문에 직접 답하는가?
- Citation accuracy: 인용한 출처가 실제 주장과 맞는가?
- Completeness: 필요한 조건, 예외, 한계를 충분히 언급했는가?

---

## 열수력/HPC 연구와 연결해서 생각하기

RAG는 원자력 열수력 연구에서 "연구 지식 접근 시스템"으로 유용할 수 있다.

가능한 사용 사례:

- 열수력 논문 corpus에서 특정 closure model, turbulence model, benchmark case 검색
- CFD 코드 문서와 input deck 설명을 기반으로 질의응답
- 실험 데이터 보고서에서 boundary condition과 uncertainty 정보 검색
- [[High Performance Computing]] job log, scaling report, profiling 결과를 모아 troubleshooting assistant 구축
- [[Scientific Machine Learning]] 논문과 코드 구현을 연결하는 연구 노트 검색

예를 들어 사용자가 "subcooled boiling CHF 예측에서 사용된 ML feature는 무엇인가?"라고 질문하면, RAG 시스템은 관련 논문 chunk를 검색하고, 모델이 그 근거를 요약해 답변할 수 있다.

---

## 흔한 실패 모드

- 검색된 문서가 질문과 미묘하게 다르다.
- chunk가 너무 작아 수식이나 조건의 맥락이 끊긴다.
- chunk가 너무 커서 검색 점수가 흐려진다.
- OCR/PDF parsing 오류로 중요한 표나 단위가 깨진다.
- LLM이 검색 근거에 없는 내용을 추가한다.
- 출처는 맞지만 주장의 위치가 틀리다.
- 오래된 문서와 최신 문서가 충돌한다.

---

## 내가 기억해야 할 핵심

- RAG는 LLM에 외부 지식을 검색해서 넣는 방법이다.
- 핵심 구성 요소는 chunking, embedding, vector database, retrieval, re-ranking, generation이다.
- RAG는 [[Transformer Model]]을 대체하는 모델 구조가 아니라, Transformer 기반 LLM을 지식 저장소와 연결하는 시스템 구조이다.
- 지식 업데이트에는 fine-tuning보다 RAG가 보통 더 유연하다.
- RAG 성능은 검색 품질, chunk 설계, context 구성, 답변 제약에 크게 좌우된다.
- 연구용 RAG에서는 출처, metadata, citation accuracy가 특히 중요하다.

---

## 컴퓨팅 관점의 입력과 출력

RAG 시스템의 입력은 사용자 질문 string이고, 내부적으로는 여러 자료구조로 변환됩니다.

```text
question: str
query_embedding.shape = [embedding_dim]
retrieved_chunks: List[str]
prompt: str
LLM output: str
```

문서 index는 미리 다음처럼 만들어 둡니다.

```text
chunk_texts: List[str] length = num_chunks
chunk_embeddings.shape = [num_chunks, embedding_dim]
metadata: List[dict]
```

즉 RAG는 `질문 문자열 → 검색된 context 목록 → LLM 답변 문자열`로 이어지는 pipeline입니다.

---

## 간단한 toy 예시

질문이 `FNO에서 Fourier mode를 왜 자르나요?`라고 해보겠습니다.

1. 질문을 embedding vector로 변환합니다.
2. Obsidian/FNO 문서 chunk 중 유사도가 높은 5개를 찾습니다.
3. 그 5개 chunk를 prompt에 넣습니다.
4. LLM이 근거 chunk를 바탕으로 답변합니다.

이때 검색이 잘못되면 생성 답변도 틀릴 가능성이 커집니다.

---

## 관련 노트

- [[Transformer Model]]
- [[Large Language Model]]
- [[Embedding Model]]
- [[Vector Database]]
- [[Semantic Search]]
- [[Hybrid Search]]
- [[Information Retrieval]]
- [[Prompt Engineering]]
- [[Hallucination]]
- [[Scientific Machine Learning]]
- [[High Performance Computing]]
- [[Surrogate Model]]

---

## 다음에 확장할 질문

- RAG에서 chunk size와 overlap은 어떻게 정해야 하는가?
- 논문 PDF의 수식, 표, 그림 캡션을 RAG에 넣을 때 어떤 preprocessing이 필요한가?
- Hybrid search는 pure vector search보다 언제 유리한가?
- 연구 노트 기반 RAG에서 Obsidian wikilink와 metadata를 어떻게 활용할 수 있는가?
- RAG evaluation dataset은 어떻게 만들어야 하는가?
- 열수력 논문 corpus에서 benchmark case와 physical condition을 metadata로 어떻게 설계할 수 있는가?
