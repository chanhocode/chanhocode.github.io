---
title: "[RAG] RAG는 무엇이고 활용 방법은 무엇인가 with langchain"
writer: chanho
date: 2024-11-27 16:50:00 +0900
categories: [AI, LargeLanguageModel]
tags: [ai, llm, rag]
---

# RAG (Retrieval-Augmented Generation)

> Large Language Model을 확장하여, 주어진 컨텍스트나 질문에 대해 더욱 정확하고 풍부한 정보를 제공하는 방법으로, 질문에 대한 답변을 생성하기 위해 검색(Retrieval)과 생성(Generation)을 결합한 자연어 처리 기술이다.

`Retrieval Phase`: 외부 지식 소스(ex. DB, 문서 저장소, 검색 엔진)에서 관련 정보를 검색한다.<br>
`Generation Phase`: 검색된 정보를 기반으로 질문에 적절한 답변을 생성한다.

## 장점

1. `정보의 최신성 유지`

   > RAG는 외부 데이터베이스나 검색 엔진에서 정보를 실시간으로 가져오기 때문에, 사전 학습 모델이 학습한 시점 이후에 발생한 최신 정보 까지 활용 가능

2. `LLM모델의 추론 능력 강화`

   > 언어 모델의 "생성 능력"에 검색된 외부 데이터를 추가로 제공하므로, 도메인 지식이 부족한 질문에도 적절한 답변을 생성할 수 있다. 또한 이를 바탕으로 'Hallucination'방지에도 좋은 성능을 보인다.

3. 기존 데이터 활용 가능

   > 이미 존재하는 문서, 데이터베이스, 논문 등의 방대한 정보를 활용할 수 있어 새로운 데이터를 별도로 학습시키지 않아도 강력한 성능을 발휘할 수 있다.

4. 소규모 데이터로도 강력한 성능
   > 완벽하게 레이블링된 대규모 데이터 없이도, 검색된 정보와 사전 학습된 언어 모델을 결합해 좋은 결과를 생성할 수 있다.

## RAG Pipeline

📒 `Load Data - Text Split - Indexing - Retrieval - Generation`

### 1. Load Data

> 검색에 사용할 외부 데이터를 준비하는 과정이다. 데이터 소스(문서, PDF, DB, 웹 크롤링 결과)를 가져온다.

#### 웹 문서 Load: WebBaseLoader

> Lanchain에서 웹 문서를 로드하기 위해서 WebBaseLoader를 사용해 내용을 로드하고 파싱할 수 있다.

- web_path: 로드할 웹 페이지의 URL이다. 단일 문자열 또는 여러 개의 URL을 시퀀스 배열로 지정할 수 있다.
- bs_kwargs: 웹 크롤링 할때 사용하는 BeautifulSoup를 사용해서 HTML을 파싱할 때 인자들을 딕셔너리 형태로 제공하도록 한다. 또한 bs4.SoupStrainer을 사용해서 특정 클래스 이름을 가진 HTML 요소만 파싱할 수 있다.

```python
import bs4
from langchain_community.document_loaders import WebBaseLoader
...
data_loader = WebBaseLoader(
  web_paths="경로 또는 (p1,p2)를 통한 복수 경로",
  bs_kwargs=dict(
    parse_only=bs4.SoupStrainer(
      class_=("html 요소")
    )
  )
)
...
```

#### 텍스트 문서 Load: TextLoader

> Langchain의 기능중 'TextLoader'을 이용해서 텍스트 파일을 불러올 수 있다. 그리고 텍스트 파일의 내용을 랭체인의 Document 객체로 변환하고 이를 리스트 형태로 반환한다.

```python
from langchain_community.document_loaders import TextLoader

loader = TextLoader('text.txt')
data = loader.load() # type: <class 'list'>
```

#### 디렉토리 폴더 Load: DirectoryLoader

> Langchain의 기능중 'DirectroyLoader'을 이용하면 특정 디렉토리 내의 모든 문서를 로드할 수 있다. 문서를 읽고 처리하기 위해서 'UnstructutedLoader'가 내부적으로 사용되어 텍스트 문서를 전처리하고 구조화된 형식으로 변환한다. 이를 위해서는 'unstructured' 라이브러리가 필요하다.

```python
from langchain_community.document_loaders import DirectoryLoader

loader = DirectoryLoader(path='./', glob='*.txt', loader_cls=TextLoader)
data = loader.load()
```

#### CSV File Load: CSVLoader

> Langchain의 'document_loaders'의 'CSVLoader' 클래스를 사용하여 CSV 파일을 로드할 수 있다. CSV 파일의 각 행을 추출하고 서로 다른 Document 객체로 변환하여 문서 객체로 이루어진 리스트 형태로 반환된다.

```python
from langchain_community.document_loaders.csv_loader import CSVLoader

loader = CSVLoader(file_path='text.csv', encoding='cp949')
data = loader.load()
```

#### Json File Load: JSONLoader

> Langchain의 'JsonLoader'을 활용해서 Json 데이터를 로드하고 이를 구조화된 형태로 변환 할 수 있다. 이때 JSONLoader을 사용하기 위해서는 'jq'라이브러리가 필요하다.

```python
# JSON 파일에서 특정 필드(.message[].content)만 추출하는 방법
from langchain_community.document_loaders import JSONLoader

loader = JSONLoader(
    file_path="./example_data/facebook_chat.json",
    jq_schema=".messages[].content",
    text_content=False,
)

docs = loader.load()

# JSON Lines(JSONL) 파일 로드
loader = JSONLoader(
    file_path="./example_data/facebook_chat_messages.jsonl",
    jq_schema=".content",
    text_content=False,
    json_lines=True,
)

docs = loader.load()

# 특정 필드에서 데이터 추출
loader = JSONLoader(
    file_path="./example_data/facebook_chat_messages.jsonl",
    jq_schema=".",
    content_key="sender_name",
    json_lines=True,
)

docs = loader.load()

# 메타데이터 포함
def metadata_func(record: dict, metadata: dict) -> dict:
    metadata["sender_name"] = record.get("sender_name")
    metadata["timestamp_ms"] = record.get("timestamp_ms")
    return metadata

loader = JSONLoader(
    file_path="./example_data/facebook_chat.json",
    jq_schema=".messages[]",
    content_key="content",
    metadata_func=metadata_func,
)

docs = loader.load()
print(docs[0].metadata)  # 메타데이터 출력

# 복잡한 데이터 추출, 중첩 구조에서 데이터를 추출하는 경우 jq_schema, metadata_func를 조합하여 사용
loader = JSONLoader(
    file_path="./example_data/nested_data.json",
    jq_schema=".nested_field[].sub_field",
    content_key="content",
    metadata_func=lambda record, metadata: {"extra_info": record.get("info")},
)

docs = loader.load()
```

#### PDF 문서 Load: PyPDFLoader

> Lanchain의 기능중 PyPDFLoader를 사용해서 PDF 파일에서 텍스트를 추출할 수 있다. 이를 위해서는 'pypdf'라이브러리가 필요하다.

```python
from langchain_community.document_loaders import PyPDFLoader

pdf_filepath = 'text.pdf'
loader = PyPDFLoader(pdf_filepath)
```

- 형식이 없는 PDF 문서를 로드하기 위해서는 'UnstruredPDFLoader'을 이용한다. 해당 클래스는 PDF 파일에서 텍스트를 추출할 때 내부적으로 unstructured 라이브러리를 활용한다. 이를 위해 'unstructured', 'unstructured-inference' 라이브러리가 필요하다.

```python
from langchain_community.document_loaders import UnstructuredPDFLoader

pdf_filepath = 'text.pdf'

loader = UnstructuredPDFLoader(pdf_filepath)
```

### 2. Text Split

> 데이터를 적절하게 분할하여 검색 효율과 정확성을 높인다. 데이터는 일반적으로 문서 단위로 저장되는데, 이 문서들을 일정한 크기로 나누는 'Chunkings (문서를 일정 크기의 텍스트 조각으로 나눔)' 과정을 진행한다. 이때 적절한 'Oberlap'을 주어 연속적인 조각 사이에 중복을 허용해 문맥이 잘려 나가는 것을 방지한다. 이러한 과정을 활용하기 위해 LangChain에서는 'TextSplitter'을 활용할 수 있다.

#### CharacterTextSplitter

> 'ChatacterTextSplitter'클래스는 텍스트를 문자 단위로 분할하는데 사용된다.

- `separator`: 분할된 청크의 구분하기 위한 기준이 되는 문자열
- `chunk_size`: 청크의 최대 길이
- `chunk_overlap`: 청크 사이에 중복으로 포함할 문자의 수
- `length_function`: 청크의 길이를 계산하는 함수

#### RecursiveCharacterTextSplitter

> 'RecursiveCharacterTextSplitter' 클래스는 텍스트를 재귀적으로 분할하여 의미적으로 관련 있는 텍스트들이 같이 있도록 하는 목적으로 설계되어 있다. 이 과정에서 문자 리스트 (['\n\n', '\n', ' ', ''])의 문자를 순서대로 사용하여 텍스트를 분할하며, 분할된 청크들이 설정된 chunk_size보다 작아질 때까지 이 과정을 반복한다.

#### Tokenizer

> LLM을 사용할때 모델이 처리할 수 있는 토큰에는 한계가 있다. 이에 입력 데이터가 모델의 정해진 토큰을 초과하지 않도록 해당 모델에 적용되는 토크나이저를 기준으로 텍스트를 분리하고, 이 토큰들의 수를 기준으로 텍스트를 청크로 나누면 모델의 토큰 수를 조절 할 수 있다.

### 3. Indexing

> 데이터를 빠르고 효과적으로 검색하기 위해 구조화된 형태로 저장하는 것이다. 이를 위해서 인덱싱 방법으로는 Vector Indexing, Embeddings를 활용할 수 있으며 벡터 저장소로는 Faiss, Weaviate, Pinecone, Milvus, Chroma 등을 활용할 수 있다.

#### Embedding

> 텍스트 데이터를 숫자로 이루어진 벡터로 변환하는 과정이다. 임베딩 과정은 텍스트의 의미적인 정보를 보존하도록 설계되어 있어, 벡터 공간에서 가까이 위치한 텍스트 조각들은 의미적으로 유사한 것으로 간주한다.

👉 `Embedding Method`

- `embed_documents`: 여러 문서(텍스트 데이터 집합)를 벡터 공간에 임베딩한다. 주로 대량의 텍스트 데이터를 처리할 때 유용하다. 예시로 문서 검색 시스템에서 데이터베이스에 저장된 모든 문서를 임베딩하여 검색 준비를 한다.

```python
# -- openAI Example
from langchain.embeddings import OpenAIEmbeddings

embedding = OpenAIEmbeddings() # 임베딩 객체
documents = ["This is a sample document.", "Another example text."] # 문서 데이터
document_vectors = embedding.embed_documents(documents) # 문서 임베딩
print(document_vectors)
```

- `embed_query`: 단일 텍스트 쿼리를 벡터 공간에 임베딩 한다. 사용자가 입력한 쿼리를 문서와 비교해 유사도를 계산하는 데 사용된다. 예시로 사용자의 질문을 벡터화 하여 가장 관련성 높은 문서를 검색하는 것을 들 수 있다.

```python
# -- openAI Example
from langchain.embeddings import OpenAIEmbeddings

embedding = OpenAIEmbeddings() # 임베딩 객체 생성
query = "What is semantic search?" # 검색 쿼리
query_vector = embedding.embed_query(query) # 쿼리 임베딩
print(query_vector)  # 벡터 출력
```

| 메소드          | 입력 데이터             | 출력 데이터         | 주요 사용 사례                |
| --------------- | ----------------------- | ------------------- | ----------------------------- |
| embed_documents | 다수의 문서 텍스트 집합 | 각 문서의 벡터 표현 | 문서 검색, 문서 분류          |
| embed_query     | 단일 쿼리 텍스트        | 쿼리의 벡터 표현    | 의미 검색, 텍스트 유사도 계산 |

👉 `Google GenAI 활용`

> `pip install langchain_google_genai`를 통해 라이브러리 설치

```python
from langchain_google_genai import GoogleGenerativeAIEmbeddings

embeddings_model = GoogleGenerativeAIEmbeddings(model='models/embedding-001')

embeddings = embeddings_model.embed_documents(
    [
        '안녕하세요!',
        '어! 오랜만이에요',
        '이름이 어떻게 되세요?',
        '날씨가 추워요',
        'Hello LLM!'
    ]
)
len(embeddings), len(embeddings[0])

embedded_query = embeddings_model.embed_query('첫인사를 하고 이름을 물어봤나요?')

for embedding in embeddings:
    print(cos_sim(embedding, embedded_query))
```

#### Vector Store

> 벡터 저장소는 고차원 벡터 데이터를 효율적으로 저장하고 검색할 수 있는 데이터베이스 또는 시스템을 의미한다. 주로 텍스트, 이미지, 소리 등 다양한 데이터를 벡터 공간에 매핑한 결과(임베딩 벡터)를 다루며, 대규모 벡터 데이터셋에서 유사성을 빠르게 검색할 수 있도록 설계되었다.

👉 `주요 기능`

- 벡터 저장: 텍스트, 이미지, 소리 등 데이터를 임베딩 모델을 통해 벡터로 변환한 후 저장한다. 벡터는 데이터의 의미적, 시각적, 또는 오디오적 특성을 수치화한 고차원 벡터이다.
- 벡터 검색: 저장된 벡터들 중에서 사용자가 제공한 쿼리 벡터와 가장 유사한 항목을 찾는 과정이다. 예를 들어, 검색 엔진에서 사용자가 질문하면, 관련성 높은 문서나 텍스트를 반환한다.
- 결과 반환: 검색된 벡터와의 유사도 점수를 기반으로 가장 관련성 높은 결과를 반환한다.

👉 `Chroma`

> LangChain과 통합 가능한 경량 벡터 저장소 이다. NLP 기반 벡터 검색 애플리케이션에 적합하다.

### 4. Retrieval

> 사용자의 질문(Query)에 맞는 가장 관련성 높은 데이터를 검색하는 과정이다. 사용자가 입력한 질문을 텍스트 임베딩으로 변환하고, 벡터 인덱스와 비교하여 관련 텍스트 조각을 반환한다. 검색 방식에는 'k-NN', 'BM25'등이 있다.

- k-NN(k-Nearest Neighbors): 임베딩 간 유사도를 계산해 가장 유사한 데이터 k개를 반환한다.
- BM25: 텍스트 기반 전통적 알고리즘.

### 5. Genetation

> 검색된 텍스트를 바탕으로 사용자 질문에 대한 최종 답변을 생성한다.

이와 같이 RAG에 대해 간략하게 정리해봤다. 이후 게시글에서는 해당 내용들의 좀더 세부적인 내용또는 새로운 요소들에 대해 작성하고자 한다. 해당 게시글을 작성하기 위해 위키독스의 '[랭체인(LangChain) 입문부터 응용까지
](https://wikidocs.net/book/14473)'을 참조했다. 좀더 자세한 내용이 필요하다면 해당 위키독스를 참조하면 좋을 것으로 보인다.
