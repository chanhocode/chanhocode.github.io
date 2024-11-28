---
title: "[RAG] Embedding model 사용해보기 with GeminiAPI & BGE-M3"
writer: chanho
date: 2024-11-28 21:28:00 +0900
categories: [AI, LargeLanguageModel]
tags: [ai, llm, rag]
---

# Embedding Model

💡 `임베딩 이란?`

> 임베딩은 텍스트의 시맨틱 의미와 맥락을 포착하여 유사한 의미를 가진 텍스트의 임베딩이 벡터 공간에서 가깝게 위치하도록 한다. 예를 들어 '강아지를 동물 변원에 데려갔습니다"와 "고양이를 동물병원에 데려갔습니다" 라는 문장은 유사한 맥락을 가지므로, 이들의 임베딩은 벡터 공간에서 서로 가깝게 위치합니다.

## 활용 Documents

> 해당 임베딩 모델 테스트를 진행하기 위한 간단한 데이터를 준비했다.

```python
documents = [
  {
    "title": "컨테이너 선 _ 파나막스 (Container Ship _ Panamax)",
    "content": "**파나막스(Panamax)**와 **뉴 파나막스(New Panamax)**는 파나마 운하를 통과할 수 있는 선박의 크기 제한을 나타내는 용어로, 운하의 구조적 한계와 규격에 따라 설계되었습니다. 이 용어는 기존 운하와 확장된 운하를 기준으로 구분됩니다.\n\n**주요 특징:**\n- 파나막스: 기존 운하 설계에 맞춘 선박으로, 최대 흘수 12.04m, 적재 용량 약 5,000 TEU.\n- 뉴 파나막스: 2016년 확장 후 도입된 규격으로, 더 큰 선박과 최대 14,000 TEU까지 적재 가능.\n\n**역사적 의의:**\n- 1914년 파나마 운하 개통으로 글로벌 물류 혁신.\n- 크기 제한은 효율적 물류와 설계 표준화를 이끌어냄.\n\n**운항 특징:**\n- 수문의 폭, 수로 깊이, 다리 높이가 주요 제한 요소.\n- 조종사 탑승 및 환경 보호 규정을 준수해야 함.\n\n**미래:** 파나마 운하 확장과 대형 선박 증가로 물류 효율이 향상되고 해운 업계의 변화가 지속될 전망."
  },
  {
    "title": "컨테이너 선 _ 수에즈맥스 (Container Ship _ Suezmax)",
    "content": "**수에즈맥스(Suezmax)**는 수에즈 운하를 통과할 수 있는 최대 크기의 선박을 정의하며, 운하 구조적 제한(깊이, 폭, 길이)을 기반으로 설계되었습니다.\n\n**주요 특징:**\n- 최대 흘수 20.1m, 최대 적재 용량 약 10,000~15,000 TEU.\n- 확장 공사 후 더 큰 선박(New Suezmax)의 통과 가능성.\n\n**운항 특징:**\n- 수심 제한: 선박 흘수는 운하 수로의 깊이에 따라 제한.\n- 효율적 설계: 대규모 화물 운송으로 물류비 절감.\n\n**역사적 의의:**\n- 지중해와 홍해를 연결, 글로벌 물류의 12%를 담당.\n- 2015년 확장 후 용량 증가와 운송 시간 단축 실현.\n\n**미래:** 운하의 지속적 확장과 대형 선박 수요 증가로 글로벌 해운 업계의 중요한 요소로 유지될 전망."
  },
  {
    "title": "컨테이너 선 _ 아라막스 (Container Ship _ Aramax)",
    "content": "**아라막스(Aramax)**는 아라비아만 해역을 운항할 수 있는 최대 크기 선박을 정의하며, 수심이 얕은 해역과 좁은 항만 조건에 최적화되어 있습니다.\n\n**주요 특징:**\n- 최대 흘수 15m, 적재 용량 약 6,000~8,000 TEU.\n- 호르무즈 해협을 기준으로 설계.\n\n**운항 특징:**\n- 최적화된 설계로 중동 지역의 에너지 수송 지원.\n- 좁은 해역에서 높은 기동성과 안전성 제공.\n\n**역사적 의의:**\n- 원유 및 LNG 수출을 효율적으로 지원.\n- 지역 항만과 해협 제한에 맞춘 설계.\n\n**미래:** 중동 에너지 물류의 중요성 증가와 환경 규제 강화에 따라 지속적인 발전과 현대화가 기대됨."
  },
  {
    "title": "벌크선 _ 파나막스 (Bulk Carrier _ Panamax)",
    "content": "**파나막스(Panamax)** 벌크선은 파나마 운하를 통과할 수 있는 크기로 설계된 벌크 화물 운반선입니다.\n\n**주요 특징:**\n- 최대 흘수 12.04m, 적재 용량 약 65,000~75,000 톤.\n- 석탄, 곡물, 철광석 등 다양한 화물 운송 가능.\n\n**운항 특징:**\n- 표준화된 설계로 전 세계 항만 접근성 보유.\n- 물류 효율성과 경제성을 극대화.\n\n**역사적 의의:**\n- 파나마 운하를 중심으로 한 글로벌 무역 네트워크의 중심.\n- 파나마 운하 제한 조건에 맞춘 설계로 물류비 절감.\n\n**미래:** 지속적인 벌크 화물 수요 증가와 환경 규제 강화로 효율적이고 친환경적인 설계가 도입될 전망."
  },
  {
    "title": "벌크선 _ 아프라막스 (Bulk Carrier _ Aframax)",
    "content": "**아프라막스(Aframax)** 벌크선은 중대형 벌크 화물 운송에 사용되며, 효율성과 경제성을 제공하는 선박입니다.\n\n**주요 특징:**\n- 적재 용량 약 70,000~100,000 톤.\n- 석탄, 곡물, 철광석 등 다양한 화물 운송 가능.\n\n**운항 특징:**\n- 얕은 해역과 중형 항만에서의 운항 적합.\n- 경제적 운송으로 화물당 탄소 배출량 감소.\n\n**역사적 의의:**\n- 중형 항만과 해역의 물류 최적화.\n- 석유 및 벌크 화물 운송의 주요 선박으로 활용.\n\n**미래:** 지속 가능성과 친환경 설계로 기술 혁신과 새로운 요구를 충족할 가능성이 큼."
  },
  {
    "title": "벌크선 _ 수에즈맥스 (Bulk Carrier _ Suezmax)",
    "content": "**수에즈맥스(Suezmax)** 벌크선은 수에즈 운하를 기준으로 설계된 대형 벌크선입니다.\n\n**주요 특징:**\n- 최대 흘수 20.1m, 적재 용량 약 150,000~160,000 톤.\n- 대규모 화물 운반으로 경제성 극대화.\n\n**운항 특징:**\n- 수로 깊이와 폭 제한 준수로 설계.\n- 석탄, 철광석, 곡물 등 대량 화물 운송.\n\n**역사적 의의:**\n- 수에즈 운하를 통한 글로벌 물류 흐름의 중심.\n- 경제적 이점과 환경적 이점을 동시에 제공.\n\n**미래:** 대형 벌크 화물 수요와 환경 규제 강화에 따른 설계 및 기술 혁신이 기대됨."
  }
]

df = pd.DataFrame(documents)
df.columns = ['title', 'content']
```

## Gemini API Embedding

> RAG의 테스트를 진행하기 위해 간단하게 사용할 수 있는 Google의 Gemini API Embedding에 대해 알아보고 사용해보고자 한다.

`Google`의 Gemini API는 텍스트 임베딩을 생성하는 모델을 제공중이다. 해당 임베딩은 텍스트의 의미와 맥락을 수치 벡터로 표현하여 시멘틱 검색, 텍스트 분류, 클러스터링 등 다양한 AI 작업에 응용할 수 있다.

### Gemni Embdeeing Model

> Gemini에서 API로 제공되는 임베딩 모델은 아래 코드를 통해 확인이 가능하다.

```python
for model in genai.list_models():
    if "embedContent" in model.supported_generation_methods:
        print(model.name)
```

: 해당 코드의 결과로 현재 두개의 임베딩 모델을 확인할 수 있다.

- models/embedding-001
- models/text-embedding-004

### Document Embedding

```python
def embed_fn(title, text):
    return genai.embed_content(
        model="models/text-embedding-004", content=text, task_type="retrieval_document", title=title
    )

df["Embeddings"] = df.apply(lambda row: embed_fn(row["title"], row["content"]), axis=1)
```

: 해당 코드를 통해 Document를 'text-embedding-004'를 통해 임베딩 하였다.

```markdown
- 데이터프레임 차원: (6, 3)
- 데이터프레임 컬럼: Index(['title', 'content', 'Embeddings'], dtype='object')

# Embeddings의 첫 번째 데이터의 임베딩 결과의 일부

> {'embedding': [-0.01840346, 0.015210435, 0.019861942, 0.016452577, 0.045706112, -0.027615234, 0.053394984, -0.03504379, 0.049246717, 0.039555907, -0.0156078795, 0.03842172, 0.06263071, -0.0039767427, 0.033486232, -0.028591583, 0.01640767, -0.045544785, -0.07141772, -0.0105059175, -0.015567026, 0.022975547, 0.06594105, -0.041251298, 0.0041016345, -0.015563263, -0.020574883, 0.045865867, 0.009999366, -0.00858956, 0.063167036, 0.040786996, 0.017781593, -0.032296445, 0.03935606, -0.003304071, -0.00806855, 0.016891267, 0.053529307, -0.08014855, -0...]}
```

## Question Embedding 및 유사도 높은 문서 출력

> 위 에서 'text-embedding-004'로 documents를 임베딩 했으므로 질문을 작성하고 동일한 모델로 임베딩을 해서 점곱 계산을 통해 질문과 문서의 유사도를 구할 수 있다. 점곱은 두 벡터 간의 방향성과 크기를 동시에 고려하여 유사도 점수를 제공하며 크기가 동일하고 방향이 유사할수록 점곲 값이 커지며, 이는 두 벡터가 의미적으로 가까움을 나타낸다.

- 질문: `컨테이너 선 중에서 최대 흘수 12.04m인 선박은 어떤 종류 인가?`
- 모델: `models/text-embedding-004`

```python
def find_best_passage(query, dataframe):
    """
    쿼리와 데이터프레임 내의 각 문서 사이의 거리를 점곱을 이용하여 계산합니다.
    """

    query_embedding = genai.embed_content(model="models/text-embedding-004", content=query, task_type="retrieval_query")
    query_vec = query_embedding.get("embedding")

    if query_vec is None:
        raise ValueError("Query embedding is missing or malformed")
    print("Query embedding generated:", query_vec[:5])

    embeddings = np.stack(dataframe["Embeddings"].apply(lambda x: x["embedding"]))
    print("Embeddings shape:", embeddings.shape)
    print("dataframe:", len(embeddings))

    if embeddings.shape[1] != len(query_vec):
        raise ValueError("Embedding dimensions do not match")

    # 점곱 계산
    dot_products = np.dot(embeddings, query_vec)
    print("Dot products:", dot_products)

    # 가장 높은 점수를 가진 텍스트 반환
    idx = np.argmax(dot_products)
    return dataframe.iloc[idx]

query = "컨테이너 선 중에서 최대 흘수 12.04m인 선박은 어떤 종류 인가?"
model = "models/text-embedding-004"

passage = find_best_passage(query, df)
```

위 코드를 수행하여 질문과 문서간 유사도를 측정하고 가장 높은 유사도를 가진 문서를 가져올 수 있다.

```
Query: 컨테이너 선 중에서 최대 흘수 12.04m인 선박은 어떤 종류 인가?
Query embedding generated: [-0.001457668, 0.011586195, -0.058092363, 0.049184415, 0.062245063]
Embeddings shape: (6, 768)
dataframe: 6
Dot products: [0.5815024  0.54061835 0.49384623 0.6115153  0.49807219 0.55653029]

title                       벌크선 _ 파나막스 (Bulk Carrier _ Panamax)
content       **파나막스(Panamax)** 벌크선은 파나마 운하를 통과할 수 있는 크기로 설계...
Embeddings    {'embedding': [0.018026227, -0.0034669237, 0.0...
Name: 3, dtype: object
```

수행한 결과는 위에와 같다. 결과를 보면 최대 흘수 12.04m 인 사이즈는 파나막스를 잘 찾았지만 질문에 컨테이너 선 이라고 했지만, 벌크선 파나막스를 유사도가 가장 높게 나왔다. 이에 한글 말고 영어로 `What type of container ship has a maximum draft of 12.04m?` 이라고 질문 한 결과는 아래와 같다.

```
title                  컨테이너 선 _ 파나막스 (Container Ship _ Panamax)
content       **파나막스(Panamax)**와 **뉴 파나막스(New Panamax)**는 파나...
Embeddings    {'embedding': [-0.01840346, 0.015210435, 0.019...
Name: 0, dtype: object
```

질문에서 의도한 '컨테이너선 파나막스'에 대한 문서를 제대로 가져 왔다. 이에 Google의 'text-embedding-004'는 한국어를 지원한다고 공식 문서에 적혀있지만, 미세조정 없이 서비스에 이용하기에는 주의가 필요할 것으로 보인다. 이에 찾아본 바로는 Google에서 미세조정을 지원하는 모델은 `textembedding-gecko` 및 `textembedding-gecko-multilingual` 에서 지원 하는 것으로 보이므로 좀 더 알아보려면 [해당 링크](https://cloud.google.com/vertex-ai/generative-ai/docs/models/tune-embeddings?hl=ko)를 참조하면 도움이 될 것으로 보인다.

`이에 'Huggingface'에서 'Open weight'모델로 공개 되어 있으면서 한국어 임베딩에도 좋은 성능을 보인다는 BGE-m3와 같은 모델을 좀 더 알아보고자 한다.`

## dragonkue/BGE-m3-ko Local Model Embedding

> dragonkue에서 'BAAI/bge-m3'모델을 한글 데이터셋을 이용해 파인튜닝한 임베딩 모델로 bge-m3 모델에는 영어와 중국어 이외에 다른 언어에는 학습이 부족하기에 'Ai hub 기계독해' 데이터셋을 활용해 추가 학습한 모델이다. 현재 RAG를 활용할 데이터셋이 매우 적기 때문에 미리 한국어 파인튜닝된 모델을 활용하고 기회가 되면 해당 모델에 추가 학습을 진행하여 임시적으로 사용해보려고 계획중 이다.

### SentenceTransformer 활용 모델 사용

> Transformers에서 지원하는 SentenceTransformer를 활용하면 모델을 직접 다운하지 않고도 해당 모델을 로드해서 사용해 볼 수 있다. 사용하고자 하는 'BGE-m3-ko' 모델의 공식 문서의 예제 코드는 아래와 같다.

```python
from sentence_transformers import SentenceTransformer

# Download from the 🤗 Hub
model = SentenceTransformer("dragonkue/bge-m3-ko")
# Run inference
sentences = [
    '수급권자 중 근로 능력이 없는 임산부는 몇 종에 해당하니?',
    '내년부터 저소득층 1세 미만 아동의 \n의료비 부담이 더 낮아진다!\n의료급여제도 개요\n□ (목적) 생활유지 능력이 없거나 생활이 어려운 국민들에게 발생하는 질병, 부상, 출산 등에 대해 국가가 의료서비스 제공\n□ (지원대상) 국민기초생활보장 수급권자, 타 법에 의한 수급권자 등\n\n| 구분 | 국민기초생활보장법에 의한 수급권자 | 국민기초생활보장법 이외의 타 법에 의한 수급권자 |\n| --- | --- | --- |\n| 1종 | ○ 국민기초생활보장 수급권자 중 근로능력이 없는 자만으로 구성된 가구 - 18세 미만, 65세 이상 - 4급 이내 장애인 - 임산부, 병역의무이행자 등 | ○ 이재민(재해구호법) ○ 의상자 및 의사자의 유족○ 국내 입양된 18세 미만 아동○ 국가유공자 및 그 유족․가족○ 국가무형문화재 보유자 및 그 가족○ 새터민(북한이탈주민)과 그 가족○ 5․18 민주화운동 관련자 및 그 유가족○ 노숙인 ※ 행려환자 (의료급여법 시행령) |\n| 2종 | ○ 국민기초생활보장 수급권자 중 근로능력이 있는 가구 | - |\n',
    '이어 이날 오후 1시30분부터 열릴 예정이던 스노보드 여자 슬로프스타일 예선 경기는 연기를 거듭하다 취소됐다. 조직위는 예선 없이 다음 날 결선에서 참가자 27명이 한번에 경기해 순위를 가리기로 했다.',
]
embeddings = model.encode(sentences)
print(embeddings.shape)
# [3, 1024]

# Get the similarity scores for the embeddings
similarities = model.similarity(embeddings, embeddings)
print(similarities.shape)
# [3, 3]
```

> 해당 방법을 활용하면 모델을 로드하고 사용해 볼 수 있다.

### transformers의 AutoModel 활용 Local에서 Embedding 모델 사용

> 나는 모델을 로컬에 저장하고 사용을 하고 싶어 해당 모델을 로컬에 받아서 위 코드에 모델 경로를 매개변수로 주었더니 value error가 발생하였다. 또한 langchain의 HuggingFaceEmbeddings을 이용해도 동일한 문제가 발생하였다. 내가 정확한 방법을 찾지 못한 걸 수도 있지만, 다른 방법을 찾았다 이를 공유하고자 한다.

```python
def encode_sentences(sentences, model_path):
    """
    문장 리스트를 입력으로 받아 해당 문장의 임베딩 벡터를 반환하는 함수.

    Args:
    - sentences (list of str): 임베딩할 문장 리스트.
    - model_path (str): 로컬 또는 허깅페이스 모델 경로.

    Returns:
    - torch.Tensor: 문장 임베딩 텐서. 크기는 (num_sentences, embedding_dim).
    """
    tokenizer = AutoTokenizer.from_pretrained(model_path)
    model = AutoModel.from_pretrained(model_path)

    inputs = tokenizer(sentences, padding=True, truncation=True, return_tensors="pt")

    with torch.no_grad():  # 학습 중이 아니므로 gradient 계산 방지
        outputs = model(**inputs)

    # Pooling (mean pooling)
    embeddings = torch.mean(outputs.last_hidden_state, dim=1)
    return embeddings

query_embedding = encode_sentences([query], model_path)[0]
print("Query embedding shape:", query_embedding.shape)
```

: 해당 함수를 사용해 질문과 문서들을 임베딩 할 수 있다. 'text-embedding-004' 모델과 비교를 위해 동일한 문서와 질문을 이용해서 검색을 수행한 결과는 아래와 같다.

```
질문: "컨테이너 선 중에서 최대 흘수 12.04m인 선박은 어떤 종류 인가?"

Query embedding shape: torch.Size([1024])
Embeddings shape: torch.Size([6, 1024])
Best similarity score: 0.7852264046669006
Best Passage:
title               컨테이너 선 _ 파나막스 (Container Ship _ Panamax)
content    **파나막스(Panamax)**와 **뉴 파나막스(New Panamax)**는 파나...
Name: 0, dtype: object

---

질문2: "What type of container ship has a maximum draft of 12.04m?"

Query embedding shape: torch.Size([1024])
Embeddings shape: torch.Size([6, 1024])
Best similarity score: 0.7567333579063416
Best Passage:
title               컨테이너 선 _ 파나막스 (Container Ship _ Panamax)
content    **파나막스(Panamax)**와 **뉴 파나막스(New Panamax)**는 파나...
Name: 0, dtype: object
```

> 위 결과와 같이 한글과 영어 두 부분 모두 내가 원하는 결과를 주었다. 해당 모델을 이용해서 좀 더 큰 데이터를 다뤄보고 결과가 좋으면 좀더 파인튜닝을 통해 서비스에 적합한 모델로 활용해 볼 수 있을 것으로 보인다.

## 검토 예정 한국어 임베딩 모델 with Hugging Face

- [bespin-global/klue-sroberta-base-continue-learning-by-mnr](https://huggingface.co/bespin-global/klue-sroberta-base-continue-learning-by-mnr)
- [intfloat/multilingual-e5-large](https://huggingface.co/intfloat/multilingual-e5-large)
