---
title: "[Paper] 다국어 임베딩 모델 snowflake-arctic-embed-l-v2.0 리뷰 및 분석"
writer: chanho
date: 2025-06-09 10:10:00 +0900
categories: [AI, LargeLanguageModel]
tags: [ai, embedding, paper]
---

- **두 가지 주요 모델:** Arctic-Embed 2.0은 두 가지 크기의 오픈소스 텍스트 임베딩 모델로 제공:
    - **중간 크기 (Medium):** `snowflake-arctic-embed-m-v2.0`
    - **큰 크기 (Large):** `snowflake-arctic-embed-l-v2.0`
- 모델 사이즈: 586M params
- **기반 인코더 모델**
    - 중간 크기 모델은 `gte-multilingual-mlm-base` (Zhang et al., 2024)를 기반으로 합니다.
    - 큰 크기 모델은 `bge-m3-retromae` (Chen et al., 2024)를 기반으로 합니다.

### 2. 임베딩 차원 (Embedding Dimensions) 및 토큰 처리

- **Matryoshka Representation Learning (MRL) 지원**
    - Arctic-Embed 2.0 모델들은 MRL을 지원하여 임베딩 벡터의 크기를 효율적으로 관리하고 압축할 수 있습니다.
    - MRL은 사전훈련 및 미세조정 단계 모두에 적용되며, 단일 절단 차원(truncated dimensionality)으로 **256**을 사용합니다.
- **최대 토큰 수 (Max Sequence Length)**
    - 8192 Tokens

> 이 논문은 새로운 다국어 임베딩 모델인 **Arctic-Embed 2.0**을 소개합니다. Arctic-Embed 2.0은 영어 및 다국어 벤치마크에서 경쟁력 있는 검색 품질을 달성하며, MRL(Matryoshka Representation Learning)을 통해 크기 효율적인 임베딩을 지원하는 것을 목표로 합니다. 주요 성과로는 기존 다국어 모델에서 관찰되던 영어 검색 품질 저하 문제를 고품질 데이터에 집중함으로써 해결하고, 미세조정(fine-tuning)의 중요성을 강조한 점입니다. 논문은 Arctic-Embed 2.0 모델 소개 외에도, 이전 다국어 모델에서 영어 검색 품질이 저하되는 잠재적 원인을 조사하고, 사전 훈련된 체크포인트 평가가 미세조정된 성능을 예측하지 못하는 경우가 많다는 점을 밝히며, 미세조정이 교차 언어 전이(cross-lingual transfer)를 향상시키는 방식을 탐구합니다.
> 

### 핵심 개념 정의

- **교차 언어 전이 (Cross-Lingual Transfer, CLT):** 자원이 풍부한 소스 언어에서 학습된 지식이 자원이 제한적이거나 없는 대상 언어로 전이되는 현상입니다.
- **MRL (Matryoshka Representation Learning):** 임베딩의 차원을 유동적으로 줄이면서도 성능 저하를 최소화하여, 다양한 환경에서 효율적으로 임베딩을 사용할 수 있게 하는 학습 방법입니다.
- **nDCG@10 (Normalized Discounted Cumulative Gain at 10):** 검색 결과 상위 10개의 순위 품질을 평가하는 지표로, 검색 시스템의 정확도를 측정하는 데 사용됩니다.

### 핵심 문제 (기존 방식의 문제점)

- **영어 검색 성능 저하:** 다국어 지원을 위해 여러 언어 데이터를 학습시키면, 기준이 되는 영어에서의 검색 성능이 오히려 떨어지는 현상이 관찰되었습니다. 이는 낮은 품질의 다국어 학습 데이터 때문이거나, 모델이 동시에 처리할 수 있는 언어의 총량을 초과하여 발생하는 문제일 수 있다는 가설이 있습니다.
- **교차 언어 전이의 불확실성:** 다국어 검색 모델에서 성공적인 교차 언어 전이를 위한 명확한 공식이 부재합니다. 데이터 품질 정량화의 어려움, 다양한 언어에서의 도메인 외부 검색 평가 벤치마크 부족, 모델 용량 초과 방지 필요성 등이 제약 요인으로 작용합니다.
- **사전훈련 평가의 한계:** 사전훈련된 체크포인트의 평가 결과가 최종 미세조정 후의 성능을 제대로 예측하지 못하는 경우가 많아, 사전훈련 평가 방법 개선의 필요성이 대두되었습니다.

### 연구 목적 (제안된 해결책을 통한 해결)

1. 영어 및 다국어 벤치마크에서 경쟁력 있는 검색 품질을 달성하고 MRL을 통해 크기 효율적인 임베딩을 지원하는 **Arctic-Embed 2.0 모델을 소개**합니다.
2. 이전 다국어 모델에서 **영어 검색 품질이 저하되는 잠재적 원인을 조사**하고, 영어와 거리가 먼 언어에 대한 사전훈련이 영어 성능에 해를 끼친다는 가설을 경험적으로 반박하며, 향후 조사를 위한 대안적 설명을 제안합니다.
3. 사전훈련된 체크포인트 평가가 종종 **미세조정된 성능을 예측하는 데 실패함**을 밝혀, 개선된 사전훈련 평가 방법의 필요성을 강조합니다.
4. 미세조정이 일반적으로 교차 언어 전이를 향상시키는 반면, 다국어 환경에서의 **대조적 사전훈련은 부정적인 교차 언어 전이를 유발할 수 있음**을 보입니다.

### 논문 주요 내용 상세 요약

### 작동 원리 및 특징

Arctic-Embed 2.0은 이전 연구들에서 영감을 받은 3단계 훈련 프레임워크를 따릅니다: 마스크 언어 모델링(masked language modeling), 대조적 사전훈련(contrastive pretraining), 그리고 대조적 미세조정(contrastive finetuning)입니다.

- **데이터 품질 우선:** 양보다는 질에 중점을 두어, 낮은 품질의 다국어 학습 데이터 소스를 의도적으로 배제하고, 검색 기반 일관성 필터링을 수행하며, 미세조정을 위해 가능한 최상의 부정적 예시를 신중하게 채굴합니다. 이러한 품질 중심 접근 방식이 영어 검색 성능 저하를 관찰하지 않은 이유로 추정됩니다.
- **기반 모델 및 토크나이저:** 중간 크기 모델에는 `gte-multilingual-mlm-base`를, 큰 크기 모델에는 `bge-m3-retromae`를 사용하며, 두 모델 모두 XLM-R 토크나이저를 사용합니다.
- **훈련 데이터 구성:** 영어 미세조정 데이터는 Merrick et al. (2024)의 데이터 혼합을 따르고, 고품질 다국어 학습 샘플을 위해 MIRACL 훈련 세트를 추가합니다. Mr. Tydi는 MIRACL과의 중복으로 인해 제외하지만, MIRACL의 모든 언어를 사용합니다.
- **데이터 필터링:** 영어 전용 사전훈련 데이터에는 휴리스틱 및 일관성 품질 필터를 적용하고, 다국어 사전훈련 데이터에는 소규모 다국어 E5 모델을 사용하여 검색 기반 일관성 필터링을 채택합니다. 각 데이터셋은 약 3백만 개의 쿼리-문서 쌍으로 분할되며, 쌍의 문서가 해당 샤드의 모든 문서 내에서 벡터 유사성 기준 순위 20위 미만이면 필터링됩니다.
- **효율적인 부정적 예시 채굴 (Hard Negative Mining):** 가장 효과적인 부정적 예시를 선택하기 위해 NV Retriever의 전략을 채택합니다. 즉, 교사 임베딩 모델에 의해 가장 관련성이 높은 것으로 평가된 문서를 부정적 예시로 사용하되, 알려진 긍정적 예시 점수의 특정 비율을 초과하는 관련성 점수를 가진 부정적 예시는 잠재적 거짓 부정으로 간주하여 폐기합니다 [1]. 더 강력한 교사 모델이 더 높은 품질의 미세조정 데이터셋을 생성한다는 것을 확인했으며, 이전 연구에서 제안된 95%의 거짓-긍정 필터링 임계값 대신 다양한 임계값을 실험하여 더 높은 임계값에서 개선을 관찰했습니다.
- **MRL 지원:** Matryoshka Representation Learning을 지원하여, 임베딩 차원을 줄여도 검색 품질 저하를 최소화하면서 효율적인 임베딩 사용을 가능하게 합니다.

### 효과 검증 및 결과

- **벤치마크 성능:** MTEB-R (영어), CLEF (다국어), MIRACL (다국어) 벤치마크에서 경쟁력 있는 nDCG@10 점수를 달성했습니다. 예를 들어, Arctic-Embed 2.0-M 모델은 MTEB-R에서 0.554, CLEF에서 0.534, MIRACL에서 0.592의 점수를 기록했습니다. Arctic-Embed 2.0-L 모델은 MTEB-R에서 0.556, CLEF에서 0.541, MIRACL에서 0.649를 기록했습니다.
- **교차 언어 전이(CLT) 실험:**
    - **미세조정 전 평가는 오해의 소지가 있음:** 미세조정 없이 130K 단계 체크포인트는 MIRACL의 영어 하위 집합에서 8K 단계 체크포인트보다 18.3% 성능이 낮아 보이지만, 미세조정 후에는 3.1% 더 나은 성능을 보입니다.
    - **대조적 사전훈련에서의 제한적 CLT:** 사전훈련 데이터로 표현되지 않은 언어 계열 대부분에서 사전 및 사후 미세조정 평가 점수에서 부정적인 추세를 관찰했습니다. 미세조정을 통한 평가에서는 사전훈련의 이점이 처음 10K 단계 내에서 정점에 도달한 후, 사전훈련 데이터로 표현되지 않은 언어에 대해서는 성능이 저하되기 시작합니다.
    - **미세조정의 역할:** 장기간의 사전훈련이 다운스트림 검색 성능에 미치는 부정적인 영향을 미세조정 단계가 "되돌리는" 구체적인 증거를 발견했습니다. 이는 사전훈련이 유용한 "지식"을 모델에 전달할 수 있지만, 작업에 맞게 "재보정"하는 미세조정이 최종 성능에 결정적임을 시사합니다.
- **데이터 품질의 중요성:** 낮은 품질의 다국어 훈련 데이터가 이전 연구에서 관찰된 성능 격차의 주요 원인일 수 있으며, 데이터 품질이 양보다 더 중요하다는 가설을 뒷받침합니다.

## 학습 관련

모델 훈련에 사용된 데이터는 크게 사전훈련(Pretraining) 데이터와 미세조정(Finetuning) 데이터로 나뉩니다. Arctic-Embed 2.0은 특히 데이터의 '질'을 강조하며, 이는 기존 다국어 모델의 영어 성능 저하 문제를 해결하는 데 중요한 역할을 한 것으로 보입니다.

### 1.1. 기반 모델 (Base Models for Masked Language Modeling)

- **초기화 모델**
    - **중간 크기 (Medium) 모델:** `m-GTE` (MLM-pretrained)를 사용하여 초기화. 이는 논문에서 언급된 `gte-multilingual-mlm-base` (Zhang et al., 2024)와 동일한 것으로 보입니다.
    - **큰 크기 (Large) 모델:** `BGE-M3` (RetroMAE-pretrained)를 사용하여 초기화. 이는 논문 및 이전 추가 자료에서 언급된 `bge-m3-retromae` (Chen et al., 2024)와 동일한 기반 모델입니다.
- **토크나이저 (Tokenizer)**
    - 두 모델 모두 **XLM-R 토크나이저**를 사용합니다.
    - **Llama3 토크나이저 실험:** Llama3 토크나이저도 실험되었으나, 효율 저하 및 효과가 미미하여 최종적으로 채택되지 않았습니다.
        - Llama3 토크나이저 실험 시에는 임베딩 레이어를 포함한 전체 가중치(weight)를 랜덤으로 재초기화한 후, MLM 사전훈련 없이 직접 사전훈련(pretraining)을 진행했습니다.

### 1.2. 사전훈련 데이터 (Pretraining Data)

- **주요 타겟 언어:** 영어, 프랑스어, 독일어, 스페인어, 이탈리아어, 포르투갈어, 폴란드어 (주로 유럽 언어 중심)
- **데이터 출처 및 비율:**
    - mC4: 60.6%
    - CC News: 15.0%
    - mWiki: 2.0%
    - Arctic Embed (자체 데이터셋으로 추정): 22.4%
- **데이터 처리:**
    - **일관성 필터링 (Consistency Filtering):** 각 언어별, 데이터셋별로 `top-20-in-3M-shard retrieval consistency filtering`을 적용합니다. 이는 논문에서 언급된 검색 기반 일관성 필터링과 유사한 개념으로, 3백만 개의 샤드 내에서 상위 20위 안에 드는 검색 일관성을 기준으로 필터링하는 방식입니다.
    - **총 데이터 규모:** 전체 쿼리-문서 쌍(pair)은 약 **14억 1천만(1.41 billion) 쌍**에 달합니다.

### 1.3. 미세조정 데이터 (Finetuning Data)

- **특징:** 고품질의 선별된 데이터(high-quality, curated negatives 포함)를 사용한 소규모 정제셋(small, curated set)입니다.
- **영어 데이터:** Merrick et al. (2024)의 데이터 혼합 방식을 따릅니다.
- **다국어 데이터:** MIRACL (Zhang et al., 2023b) 훈련 세트를 추가합니다.

### 2. 훈련 방법 및 함수 (Training Methods and Functions)

Arctic-Embed 2.0은 **3단계 훈련 프레임워크**를 따릅니다.

1. 마스크 언어 모델링 (Masked Language Modeling) - 기반 모델 사용
2. 대조적 사전훈련 (Contrastive Pretraining)
3. 대조적 미세조정 (Contrastive Finetuning) [1]

### 2.1. 대조적 사전훈련 (Contrastive Pretraining)

- **손실 함수 (Loss Function):** **InfoNCE 대조적 손실 (InfoNCE contrastive loss)**
    - 온도 매개변수 (Temperature τ): **0.02**
- **네거티브 샘플링 (Negative Sampling):** **무작위 배치 내 네거티브 샘플 (Random in-batch negatives)**
- **멀티소스 배치 처리 (Multi-source Batching):** 한 번에 하나의 데이터 소스에서만 데이터를 추출하며, 언어별 하위 집합(subset)은 독립적인 소스로 취급합니다.
- **토큰 길이 (Token Length):**
    - 쿼리 (Query): **32 토큰.**
    - 문서 (Document): **256 토큰.**
- **Matryoshka Representation Learning (MRL):** 사전훈련 단계에 적용되며, 절단 차원은 256입니다.

### 2.2. 대조적 미세조정 (Contrastive Finetuning)

- **손실 함수 (Loss Function):** 사전훈련과 동일하게 **InfoNCE 대조적 손실 (InfoNCE contrastive loss)** 사용.
- **네거티브 샘플링 (Negative Sampling):** 무작위 방식이 아닌, "커스텀 선정된 어려운 네거티브 샘플 (Custom-selected hard negatives)"을 사용합니다.
    - 이는 논문에서 언급된 NV Retriever (Moreira et al., 2024) 전략을 따르며, 교사 임베딩 모델을 사용하여 관련성 높은 문서를 부정적 예시로 사용하되, 거짓 부정을 필터링하는 방식입니다.
- **토큰 길이 (Token Length):**
    - 쿼리 (Query) 및 문서 (Document) 모두 최대 **512 토큰.**
- **Matryoshka Representation Learning (MRL):** 미세조정 단계에도 적용되며, 절단 차원은 256입니다.

### 3. 하이퍼파라미터 (Hyperparameters)

**공통 하이퍼파라미터**

- **손실 함수 (Loss Function):**
    - 사전훈련: InfoNCE 대조적 손실 (InfoNCE contrastive loss) 사용, 온도 매개변수 (Temperature τ)는 0.02.
    - 미세조정: 사전훈련과 동일하게 InfoNCE 대조적 손실 사용.
- **MRL 절단 차원 (MRL Truncated Dimensionality):**
    - 사전훈련: 256
    - 미세조정: 256

**데이터 관련 하이퍼파라미터:**

- **네거티브 샘플링 (Negative Sampling):**
    - 사전훈련: 무작위 배치 내 네거티브 샘플 (Random in-batch negatives) 사용.
    - 미세조정: "커스텀 선정된 어려운 네거티브 샘플 (Custom-selected hard negatives)" 사용.
- **쿼리 토큰 길이 (Query Token Length):**
    - 사전훈련: 32 토큰.
    - 미세조정: 최대 512 토큰.
- **문서 토큰 길이 (Document Token Length):**
    - 사전훈련: 256 토큰.
    - 미세조정: 최대 512 토큰.

**학습 관련 하이퍼파라미터:**

- **배치 크기 (Batch Size):**
    - 사전훈련: 32,768
    - 미세조정: "256 세트"로 구성. 각 세트는 1개의 쿼리, 1개의 긍정적 문서(positive document), 10개의 부정적 문서(negative documents)로 이루어짐.
- **학습률 (Learning Rate) - Large 모델:**
    - 사전훈련: 3e-5
    - 미세조정: 5e-6
- **학습률 (Learning Rate) - Medium 모델:**
    - 사전훈련: 1e-4
    - 미세조정: 1e-5
- **학습률 스케줄 (Learning Rate Schedule):**
    - 사전훈련: 선형 워밍업-안정-감쇠 (Linear Warmup-Stable-Decay, WSD) 스케줄을 3 에폭(epoch) 동안 적용
    - 미세조정: 워밍업(warmup) 없음. 총 9,342 스텝(step) 중 초기 6,000 스텝 동안 선형 감쇠(linear decay) 적용
- **에폭 (Epochs):**
    - 사전훈련: 3 에폭
    - 미세조정: 스텝 기반으로 진행되어 특정 에폭 수가 명시되지 않음 (총 9,342 스텝)

**인프라 및 최적화 (주로 사전훈련 시 정보):**

- **GPU:**
    - 사전훈련: 32대의 H100 GPU를 사용한 분산 학습 환경
    - 미세조정: 특정 GPU 정보는 제공되지 않음.
- **최적화 (Optimization):**
    - 사전훈련: 활성화 체크포인팅(Activation Checkpointing) 사용
    - 미세조정: 특정 최적화 기법 정보는 제공되지 않음

### 평가 데이터 및 설정 (Evaluation Data and Settings)

- **CLEF 벤치마크:** 5개 유럽 언어에 대해 평가. 쿼리 수, 문서 수, 정답 개수, 질문당 평균 정답 등이 표기됨.
- **토큰 길이 허용 (벤치마크 비교 시):**
    - E5 모델: 512 토큰
    - 그 외 모델 (Arctic-Embed 포함 추정): 8192 토큰 [추가 자료]. 이는 Arctic-Embed-L 모델의 긴 컨텍스트 지원 능력과 일치합니다.

### 5. 소거 실험 (Ablation Studies)

- 사전훈련 스텝 수를 단축한 상황(예: 13,000 스텝)에서 다양한 언어, 모델, 데이터 조합의 영향을 비교하는 실험을 수행했습니다. 이는 훈련 과정의 효율성과 각 요소의 기여도를 파악하는 데 중요한 역할을 합니다.

## 주요 이점 및 시사점

- **영어 성능 저하 없는 다국어 지원:** 고품질 데이터에 집중함으로써, 다국어 지원을 확장하면서도 영어 검색 성능이 저하되지 않음을 보여주었습니다.
- **미세조정의 결정적 역할:** 모델에 "지식"을 주입하는 대규모 사전훈련과 함께, 가장 정제된 긍정적 및 어려운 부정적 예시로 모델을 신중하게 "재보정"하는 미세조정의 중요성을 강조합니다.
- **크기 효율적 임베딩:** MRL 지원을 통해 다양한 환경에서 모델을 효율적으로 배포하고 사용할 수 있는 가능성을 열었습니다.
- **기존 통념에 대한 도전:** 장기간의 대조적 사전훈련이 항상 교차 언어 전이를 향상시키지 않으며, 사전훈련 단계의 평가가 최종 성능을 담보하지 않는다는 점을 실험적으로 보여주었습니다.
