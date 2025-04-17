---
title: "[RAG] Modular RAG & Self-Routing: LLM 검색 효율을 위한 모듈화 전략"
writer: chanho
date: 2025-04-17 13:00:00 +0900
categories: [AI, LargeLanguageModel]
tags: [ai, rag]
---

> Retrieval-Augmented Generation(RAG)은 LargeLanguageModel에 발생하는 환각(hallucination) 문제를 해결하고 정확성을 높이기 위해 외부 지식 기반에서 정보를 검색해 답변에 활용하는 방식이다. 시간이 지남에 따라 RAG는 크게 세 가지 형태로 발전했다.

| 패러다임 | 핵심 개념 | 장점 | 단점 |
| --- | --- | --- | --- |
| **Naive RAG** | Index → Retrieve → Generate | 단순, 빠른 구현 | 정보 중복, 비효율적 검색 |
| **Advanced RAG** | Pre-/Post-Retrieval 최적화 | 정확한 검색, 성능 향상 | 복잡성 증가, 비용 상승 |
| **Modular RAG** | 개별 모듈 교체 및 조합 가능 | 뛰어난 확장성 및 유연성 | 높은 설계 난이도 |

RAG의 각 단계는 점진적으로 고도화되며, 기능 향상과 모듈화 가능성을 갖고 있다.

`Indexing 단계`에서는, 초기에는 단순히 텍스트를 일정 크기로 나누고 임베딩하는 방식이 사용된다. 이후 메타데이터를 활용하거나 그래프 구조를 결합하고, 문서 특성에 따라 최적의 청크 크기를 적용하는 등의 향상된 기법(Advanced)이 도입될 수 있으며, 더 나아가 FAISS, Qdrant 등 다양한 인덱스 라이브러리를 선택적으로 교체할 수 있는 Index 모듈 기반(Modular) 구조로 확장할 수 있다.

`Retrieval 단계`에서는, 기본적으로 k-NN 기반의 유사도 검색을 활용하지만, 이를 넘어서 하이브리드 검색, 쿼리 리라이팅, 결과 리랭킹 등의 전략으로 검색 성능을 높일 수 있다(advanced). 이러한 전략들은 Search 모듈로 분리되어 다중 검색 전략이나 코드 검색 등 특정 목적에 따라 유연하게 적용할 수 있다(modular).

`Generation 단계`에서는, 단순히 검색된 컨텍스트를 LLM에 입력해 응답을 생성하는 방식에서 시작한다. 이후에는 중복 제거, 정보 압축, 프롬프트 최적화, 그리고 LLM 자체의 파인튜닝 등을 통해 보다 정제된 응답을 생성할 수 있다(advanced). 마지막으로 Fusion 또는 Routing 모듈을 통해 다양한 답변 후보를 융합하거나, 상황에 따라 적절한 생성 전략을 선택함으로써 답변 품질을 관리하는 구조로 확장된다(modular).

### 🔧 Advanced RAG 예시 코드

```python
# Chroma 기반 벡터 DB 생성
vs = Chroma.from_documents(
    docs,
    embedding=OpenAIEmbeddings(),
    collection_metadata={"source": "kb-v1", "chunk_tokens": 256}
)

# 쿼리 확장 및 리랭킹 적용
retriever = MultiQueryRetriever(
    retriever=vs.as_retriever(), llm=ChatOpenAI("gpt-4o")
)

rerank = CohereReranker(top_n=4)
adv_retriever = ContextualCompressionRetriever(
    base_compressor=rerank, base_retriever=retriever
)

# 최종 답변 생성
llm_chain = RetrievalQA.from_chain_type(
    llm=ChatOpenAI("gpt-4o"),
    retriever=adv_retriever,
    chain_type="stuff"
)

```

### 🔧 Modular RAG 구현 예시 (LangGraph 사용)

```python
graph = lg.Graph()

graph.add_module("index",  IndexModule(vdb="qdrant"))
graph.add_module("search", SearchModule(hybrid=True, rerank="colbert"))
graph.add_module("fusion", FusionModule(strategy="query_fusion"))

graph.connect("search", "fusion")
graph.connect("index",  "search")

rag_pipeline = graph.compile(start="index", end="fusion")
answer = rag_pipeline.run(user_question)

```

### 종합 비교 (성능 vs 효율 관점)

| 지표 | Naive | Advanced | Modular |
| --- | --- | --- | --- |
| **정답률(Exact Match)** | ★★☆☆☆ | ★★★★☆ (+10‑25 pp)* | ★★★★★ (케이스별 최대화) |
| **응답속도** | ★★★★☆ (단순) | ★★★☆☆ (리랭크로 지연) | 설계에 따라 가변 |
| **GPU 비용** | 낮음 | 중간 (리랭크 모델) | 가변 (동적 라우팅) |
| **시스템 복잡도** | 낮음 | 중간 | 높음 |
| **유연성/모듈성** | 낮음 | 중간 | 최고 |

### 적용 및 상황별 추천

| 시나리오 | 추천 접근 방법 |
| --- | --- |
| 빠른 PoC, MVP | Naive RAG |
| 전문 분야 (법률, 의료 등) | Advanced RAG |
| 다국어, 멀티모달 지원 및 확장성 중시 | Modular RAG |
| 코드, DB 등 복합 작업 수행 | Modular RAG 기반 LLM Agent 구성 |

# Modular RAG 및 Self-Routing RAG 이해

> RAG는 외부 지식을 활용해 LLM의 정확성을 높이는 좋은 방법이지만, 모든 질문에 항상 RAG를 수행하는 것이 비효율적이라는 의문이 들었다. 특히 단순한 대화나 질문 길이가 짧은 경우 RAG가 오히려 불필요한 리소스를 소모하거나 과도한 정보를 주입해 LLM의 답변 정확도를 떨어뜨릴 수도 있다고 생각했다.

## Modular RAG란?

Modular RAG는 RAG의 다양한 구성 요소(Index, Search, Fusion, Routing 등)를 독립된 모듈로 구성하고, 필요에 따라 각 모듈을 쉽게 교체하거나 재구성할 수 있는 유연한 구조를 의미한다.

### Modular RAG의 핵심

- **Routing Module**: 질문이 들어왔을 때 검색이 필요한지를 판단하고, 필요하면 외부 지식 검색을 수행하고 아니면 바로 LLM에 질문을 전달
- **Search Module**: 외부 정보를 실제로 검색하는 모듈
- **Fusion Module**: 검색된 정보를 LLM이 효과적으로 사용할 수 있도록 결합

## Self-Routing RAG

> 최근 “Self-Routing RAG: Binding Selective Retrieval with Knowledge Verbalization” 라는 논문에서 말하는 Self-Routing RAG는 RAG의 검색 여부 자체를 LLM이 직접 판단하도록 한 방식이다. 질문이 들어왔을 때, LLM이 질문의 성격을 파악하여 외부 지식 검색이 필요한지 여부를 결정하는 메커니즘으로 나의 고민과 내가 내리고 있던 결론과 유사했다.

### Self-Routing RAG의 주요 프로세스

1. **질문 입력**
2. **LLM이 검색 필요성 결정**
    - 검색이 필요하면 → 벡터 DB에서 관련 정보를 검색
    - 검색이 불필요하면 → 바로 LLM이 답변 생성
3. **답변 후 Reflection (자기 평가 및 수정)**

### Self-Routing 예시 코드

```python
from langchain_core.prompts import ChatPromptTemplate
from langgraph.graph import Graph

gate_prompt = ChatPromptTemplate.from_messages([
    ("system", "Output `RETRIEVE` if the question needs external knowledge, otherwise `NO_RETRIEVE`."),
    ("human", "{question}")
])

gate_llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

g = Graph()
g.add_node("router", gate_llm | gate_prompt)
g.add_node("retrieval_chain", retrieval_qa_chain)
g.add_node("direct_llm", direct_chat_chain)

g.add_edge("router", "retrieval_chain", condition=lambda x: x=="RETRIEVE")
g.add_edge("router", "direct_llm", condition=lambda x: x=="NO_RETRIEVE")

pipeline = g.compile(start="router")

```

## Modular RAG vs Self-Routing RAG 비교

| 구분 | Modular RAG | Self-Routing RAG |
| --- | --- | --- |
| 검색 결정 메커니즘 | Routing Module (별도 모듈) | LLM 자체 판단 |
| 구성의 유연성 | 매우 높음 (모듈 교체 및 조합 가능) | 높음 (단일 LLM으로 간단히 통합 가능) |
| 구현 복잡성 | 중~고 (모듈별 인터페이스 정의 필요) | 중 (LLM 파인튜닝 필요) |
| 리소스 및 비용 효율성 | 선택적 사용으로 효율적 | 선택적 사용으로 매우 효율적 |

---

## LLM Agent와 Modular RAG, Self-Routing RAG의 차이점

> 마지막으로 글을 맺기 전에, 내가 또 다른 고민으로 가졌던 부분은 "이러한 방식들이 LLM Agent와 다른 것이 무엇인가?" 였지만, 조금 더 고민해보니 명확한 차이가 있다고 판단해 기록을 남겨 놓고자 한다.

| 구분 | Self-Routing RAG / Modular RAG | LLM Agent |
| --- | --- | --- |
| 목적 | 외부 검색 정보 필요성 판단 및 정확성 개선 | 복잡한 목표 달성을 위한 다중 툴 사용 |
| 핵심 루프 | 검색 여부 판단 및 답변 생성 | 계획(Planning) → 다수의 툴 사용 → 상태 관리 |
| 사용되는 툴 | 주로 검색(벡터 DB) | 검색 외 다양한 API, DB, 코드 등 사용 |
| 메모리 및 계획 | 간단한 Reflection | 복잡한 계획 수립 및 장기/단기 메모리 활용 |
