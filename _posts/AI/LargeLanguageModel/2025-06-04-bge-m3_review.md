---
title: "[Paper] 다국어 임베딩 모델 BGE-M3 리뷰 및 분석"
writer: chanho
date: 2025-06-04 13:30:00 +0900
categories: [AI, LargeLanguageModel]
tags: [ai, embed, paper]
---

[BGE-M3_arxiv](https://arxiv.org/pdf/2402.03216)

- **모델명**
    - M3-Embedding (Multi-Linguality, Multi-Functionality, Multi-Granularity Embedding)
    - BGE-m3 (BAAI General Embedding-m3)
- **주요 특징 및 목적**
    - **다국어 지원 (Multi-Linguality):** 100개 이상의 언어에 대한 통합적인 의미 검색 지원.
    - **다기능성 (Multi-Functionality):** Dense retrieval, sparse retrieval, multi-vector retrieval 세 가지 검색 기능을 동시에 지원.
    - **다중 세분성 (Multi-Granularity):** 짧은 문장부터 긴 문서까지 다양한 길이의 입력 처리 가능.
- **최대 입력 토큰 수 (Max Input Tokens)**
    - **8192 토큰**
- **모델 크기 (Model Size) / 백본 아키텍처 (Backbone Architecture)**
    - 560M Params
    - **XLM-RoBERTa** 모델을 백본으로 사용합니다.
    - 이 XLM-RoBERTa는 **RetroMAE 방식**으로 사전 학습된 버전입니다.
- **Tokenizer**
    - XLM-RoBERTa tokenizer 사용.

---

### 핵심 개념 정의

- **nDCG@10 (Normalized Discounted Cumulative Gain at 10):** 정보 검색 시스템의 성능을 평가하는 주요 지표 중 하나입니다. 검색 결과 상위 10개의 순서와 문서의 관련성을 모두 고려하여 평가하며, 값이 높을수록 검색 품질이 우수함을 의미합니다.
- **BM25:** 키워드 빈도 및 문서 빈도에 기반한 전통적인 검색 알고리즘입니다.
- **Dense Retrieval:** 쿼리와 문서를 밀집 벡터(dense vector)로 표현하고, 이 벡터 간의 유사도(예: 내적)를 기반으로 검색하는 방식. 일반적으로 [CLS] 토큰의 임베딩을 사용합니다.
- **Sparse Retrieval (Lexical Retrieval):** 각 단어(토큰)의 중요도를 임베딩을 통해 추정하고, 이를 기반으로 검색하는 방식. 전통적인 키워드 기반 검색과 유사한 특성을 가집니다. M3-Embedding에서는 각 토큰별 가중치(예: ReLU(W_lex·H))를 계산하여 동일 토큰 가중치 곱의 합으로 점수를 매깁니다.
- **Multi-vector Retrieval:** 쿼리와 문서 간의 세밀한 상호작용을 여러 임베딩 벡터를 통해 계산하는 방식. M3-Embedding에서는 ColBERT 스타일의 후기 상호작용(late interaction)을 사용하며, W_mul로 투영 후 풀링(pooling)합니다.
- **Self-Knowledge Distillation:** 모델 자체가 생성한 다양한 검색 기능의 관련성 점수를 통합하여 교사 신호(teacher signal)로 활용, 학습 품질을 향상시키는 기법.
- **InfoNCE Loss:** 대조 학습(contrastive learning)에 주로 사용되는 손실 함수로, 유사한 샘플은 가깝게, 다른 샘플은 멀게 임베딩하도록 학습합니다.
- **RetroMAE:** Masked Autoencoder 방식에 기반한 사전 학습 방법론으로, 원본 문장을 복원하는 과정에서 효과적인 표현 학습을 유도합니다.
- **ANCE (Approximate Nearest Neighbor Negative Contrastive Estimation):** 하드 네거티브 샘플(정답은 아니지만 모델이 정답과 유사하다고 판단하는 샘플)을 효과적으로 발굴하여 학습에 활용하는 기법.
- **mDPR, mContriever, mE5 large, E5mistral-7b, OpenAI-3:** 딥러닝 기반의 텍스트 임베딩 모델들입니다.

### 핵심 문제

기존의 다국어 및 긴 문서 검색 모델들이 특정 언어나 긴 컨텍스트 처리에서 가질 수 있는 성능적 한계를 극복하고, 보다 광범위하고 정확한 정보 검색을 가능하게 하기 위해 M3-Embedding이 제안된 것으로 유추할 수 있습니다.

### 연구 목적

1. M3-Embedding이라는 새로운 다국어 임베딩 방법론을 제안합니다.
2. 다양한 언어(MIRACL 데이터셋 기준) 및 긴 문서(MLDR 테스트셋 기준) 검색 벤치마크에서 M3-Embedding의 성능을 기존 최신 모델들과 비교하여 검증합니다.
3. 이를 통해 M3-Embedding의 효과와 실용성을 논의합니다.

## 논문 주요 내용 상세 요약

### **데이터 (Data)**

- **사전학습 (Pre-training) 데이터**
    - **규모:** 총 12억(1.2B) 개의 텍스트 쌍 (text pairs).
    - **언어:** 194개 언어.
    - **소스:** 위키피디아 (Wikipedia), mC4, CC-News, S2ORC, xP3, NLLB, CCMatrix 등 매우 다양한 출처의 데이터를 활용합니다.
    - **특징:** 강제 번역된 병렬 데이터(translation data), 레이블된 데이터(labeled data), 그리고 특히 장문(long document) 생성을 위한 합성 데이터(synthetic data)를 포함합니다.
- **파인튜닝 (Fine-tuning) 데이터:**
    - **구성:** 영어(8종), 중국어(7종)의 고품질 데이터셋 및 Mr.Tydi, MIRACL과 같은 다국어 데이터셋을 활용합니다.
    - **합성 데이터:** Wikipedia, Wudao, mC4의 장문 데이터 일부를 추출하고, GPT-3.5와 같은 대형 언어 모델을 사용하여 생성된 질문-본문 쌍 형태의 합성 데이터를 포함합니다.

### **모델 및 입력 (Model & Input)**

- **백본 (Backbone):** XLM-RoBERTa 모델을 사용하며, RetroMAE 방식으로 사전 학습된 버전을 활용합니다.
- **입력 최대 토큰:** 8192 토큰으로, 긴 문서 처리에 용이합니다.
- **Tokenizer:** XLM-RoBERTa tokenizer를 사용합니다.

### **주요 매개변수 (Hyperparameters)**

- **Optimizer:** AdamW
- **Learning Rate (학습률):** 통상적으로 1e-5 ~ 1e-4 범위 내에서 사용되며, 실제 학습 환경 및 스케줄에 따라 조정됩니다.
- **Scheduler (학습률 스케줄러):** WarmupDecayLR (초기 warmup 기간 이후 학습률을 점진적으로 감소시키는 방식)을 사용합니다.
- **Batch size (배치 크기):** 매우 크게 설정합니다 (예: 8,000 ~ 32,000). 이는 하드웨어 사양에 따라 조정됩니다. 대규모 배치는 인배치 네거티브(in-batch negatives) 샘플을 늘려 임베딩의 판별력을 높이는 데 기여합니다.
- **Gradient Accumulation & Clipping:** 학습 안정성 및 메모리 효율성을 위해 환경에 맞춰 조정하여 사용합니다. 특히 긴 시퀀스 처리 시 gradient checkpointing 기법을 활용합니다.

### **Retrieval/Score 방법 (학습 시 모두 사용)**

M3-Embedding은 학습 과정에서 다음 세 가지 검색/점수 계산 방식을 모두 활용하여 다기능성을 확보합니다:

- **Dense Retrieval:** [CLS] 토큰의 임베딩 간 내적(inner product)으로 유사도 점수를 계산합니다.
- **Sparse Retrieval:** 각 토큰의 임베딩 H에 가중치 행렬 W_lex를 곱하고 ReLU 활성화 함수를 적용하여 토큰별 가중치(weight = ReLU(W_lex · H))를 얻습니다. 이후 쿼리와 문서 간 동일 토큰의 가중치 곱을 합산하여 점수를 계산합니다.
- **Multi-vector Retrieval:** 각 토큰 임베딩을 별도의 가중치 행렬 W_mul로 투영(projection)하고, 풀링(pooling)을 거친 후 ColBERT 스타일의 후기 상호작용(late interaction)을 통해 점수를 계산합니다.

이 세 가지 점수는 최종적으로 가중합되어 하이브리드 검색 점수 s_rank (또는 논문에서는 s_inter)로 사용될 수 있습니다:

### **손실 함수 (Loss Function - 학습 목적)**

M3-Embedding의 학습은 여러 손실 함수의 조합으로 이루어집니다.

- **개별 검색 기능 손실 (Native Loss)**

: Dense retrieval (L_dense), Sparse retrieval (L_lex), Multi-vector retrieval (L_mul) 각각에 대해 InfoNCE (contrastive loss)를 계산합니다. 이 손실 함수는 긍정 쌍(positive pair)의 유사도는 높이고 부정 쌍(negative pair)의 유사도는 낮추도록 학습합니다.

- **Self-Knowledge Distillation Loss (자기 지식 증류 손실)**

: 세 가지 검색 방식의 예측 점수(s_dense, s_lex, s_mul)를 가중합하여 통합 점수 s_inter (교사 신호)를 생성합니다: s_inter = w1·s_dense + w2·s_lex + w3·s_mul. 이 s_inter를 소프트 레이블(soft label)로 사용하여, 각 개별 검색 방식의 예측(s_dense, s_lex, s_mul 각각)이 이 통합 점수를 모방하도록 학습합니다. 이는 일반적으로 softmax 함수와 cross-entropy 손실을 통해 계산됩니다 (L'_* = -p(s_inter) * log p(s_*)). 증류 손실은 L' = (λ1·L'_dense + λ2·L'_lex + λ3·L'_mul) / 3 와 같이 결합됩니다.

- **최종 학습 손실 (Final Loss)**

: 네이티브 손실들의 가중합 (L = (λ1·L_dense + λ2·L_lex + λ3·L_mul + L_inter) / 4, 여기서 L_inter는 s_inter를 직접 최적화하는 손실)과 자기 지식 증류 손실 L'을 결합하여 최종 손실 함수를 구성합니다.
L_final = (L + L') / 2 , 논문에서는 w1=1, w2=0.3, w3=1, λ1=1, λ2=0.1, λ3=1과 같은 가중치를 사용하여 학습 초기 s_lex의 낮은 정확도와 높은 L_lex의 영향을 줄였습니다.

- **활성화 함수 (Activation Functions)**
    - Sparse retrieval의 토큰 가중치 계산 시: ReLU
    - Self-knowledge distillation loss 계산 시: Softmax

### **Special Training Tricks**

- **Hard Negative Mining (하드 네거티브 샘플 발굴)**

**:** ANCE (Approximate Nearest Neighbor Negative Contrastive Estimation) 기법을 기반으로 합니다. 교사 모델(teacher model)을 사용하여 쿼리에 대해 높은 유사도 점수를 가지지만 실제로는 정답이 아닌 어려운 부정 샘플을 발굴하여 학습에 활용함으로써 모델의 판별력을 향상시킵니다.

- **Efficient Batching (효율적인 배치 구성)**
    - **시퀀스 길이별 그룹핑:** 학습 데이터를 입력 시퀀스 길이에 따라 그룹화하고, 동일 그룹 내에서 미니배치를 샘플링합니다. 이를 통해 패딩(padding)을 최소화하고 GPU 활용률을 높입니다.
    - **Gradient Checkpointing:** 긴 시퀀스 학습 시 메모리 사용량을 줄이기 위해 사용됩니다.
    - **GPU 간 임베딩 브로드캐스팅 (Cross-GPU Broadcasting):** 분산 학습 환경에서 각 GPU가 생성한 임베딩을 다른 GPU로 전송하여 매우 큰 규모의 인배치 네거티브(in-batch negative) 샘플을 효과적으로 활용할 수 있도록 합니다. 이를 통해 특히 8192 길이의 텍스트 처리 시 배치 크기를 20배 이상 늘릴 수 있었다고 합니다.

### **주요 단계별 학습 플로우 (Overall Training Flow)**

1. **사전학습 (Pre-training)**

: XLM-RoBERTa 백본 모델을 RetroMAE 방식으로 대규모 다국어 언어쌍 데이터(unsupervised)를 사용하여 사전 학습합니다. 이 단계에서는 주로 Dense retrieval 기능에 대해 기본적인 대조 학습(InfoNCE)을 수행합니다.

1. **파인튜닝 (Fine-tuning)**

: 레이블된 데이터 및 (장문 중심의) 합성 데이터를 포함한 다양한 데이터셋을 사용하여 모델을 파인튜닝합니다. 이 단계에서 Dense, Sparse, Multi-vector 세 가지 검색 기능을 모두 학습시키며, Self-Knowledge Distillation 기법을 적용하여 학습 품질을 극대화합니다. Hard negative mining 및 Efficient batching 기법을 적극적으로 활용합니다.

### 효과 검증 및 결과 (MLDR & MIRACL 데이터셋 기반)

제공된 표는 M3-Embedding 모델이 다양한 언어와 조건에서 기존 모델들과 비교하여 어떤 성능을 보이는지 nDCG@10 점수를 통해 보여줍니다.

- **MIRACL 데이터셋 (다국어 검색 성능, nDCG@10) [1]**
    - M3-Embedding은 "Dense", "Sparse", "Multi-vec", "Dense+Sparse", "All" 등 다양한 구성으로 평가되었습니다.
    - 대부분의 평가 언어에서 M3-Embedding, 특히 "All" 또는 "Dense+Sparse" 구성이 BM25, mDPR, mContriever, mE5 large, E5mistral-7b, OpenAI-3 등 기존 모델들보다 우수한 성능을 기록했습니다.
    - **성능 비교 예시 (M3-Embedding "All" 기준):**
        - 아랍어 (ar): **71.5** (OpenAI-3: 65.6, mE5 large: 68.7)
        - 독일어 (de): **76.3** (OpenAI-3: 73.6, mE5 large: 76.9 - mE5 large와 유사하거나 약간 낮음)
        - 한국어 (ko): **71.8** (OpenAI-3: 63.9, mE5 large: 68.1)
        - 일본어 (ja): **75.2** (OpenAI-3: 71.9, mE5 large: 71.5)
        - 크메르어 (km)에서는 M3-Embedding "All"이 **69.2**로, 다른 모델들(예: OpenAI-3 33.9, mE5 large 28.1)에 비해 특히 큰 폭의 성능 향상을 보였습니다.
- **MLDR 테스트셋 (다국어 긴 문서 검색, nDCG@10, Table 3)**
    - 이 평가는 최대 길이 8192 토큰의 긴 문서를 대상으로 합니다.
    - M3-Embedding ("Dense+Sparse", "All")은 여러 언어의 긴 문서 검색에서도 강력한 성능을 나타냈습니다.
    - **성능 비교 예시 (M3-Embedding "All" 기준):**
        - 영어 (en): **86.8** (E5mistral-7b: 70.2, text-embedding-ada-002: 59.8)
        - 한국어 (ko): **55.7** (E5mistral-7b: 32.7, mE5 large: 27.1)
        - 아랍어 (ar): **64.7** (E5mistral-7b: 29.6, text-embedding-ada-002: 16.3)
        - M3-Embedding의 "Sparse" 구성 요소 단독으로도 BM25와 같은 전통적인 희소 벡터 방식보다 많은 언어에서 높은 성능을 보였습니다.

---

**한계점 및 향후 연구 방향**

- 다양한 데이터셋 및 실제 시나리오에서의 일반화 가능성에 대한 추가 검증이 필요합니다.
- 매우 긴 문서(8192 토큰 초과) 처리 시 계산 자원 및 효율성 문제가 발생할 수 있으며, 이에 대한 추가 연구가 필요합니다.
- 100개 이상의 언어를 지원하지만, 언어별 성능 편차에 대한 심층 분석과 모든 언어에 대한 견고성 확보 노력이 필요합니다.
- 훈련 데이터의 언어별 불균형으로 인해 모델 성능이 언어에 따라 다를 수 있으며, 이는 잠재적으로 차별적이거나 불공정하게 보일 수 있는 윤리적 고려사항이 있습니다.
