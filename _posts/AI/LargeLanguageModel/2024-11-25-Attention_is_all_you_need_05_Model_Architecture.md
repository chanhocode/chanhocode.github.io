---
title: "[LMM] Attention is All You Need를 읽어보자(5)_Model Architecture"
writer: chanho
date: 2024-11-25 21:58:00 +0900
categories: [AI, LargeLanguageModel]
tags: [ai, llm]
---

> 오늘은 일이 많아 작성하지 못하고 있던 'Attention is All You Need' 논문의 Model Architecture 부분을 리뷰 하고자 한다.

# Attention is All You Need - Transformer Model

> [Attention Is All You Need](https://arxiv.org/abs/1706.03762)
> Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N. Gomez, Lukasz Kaiser, Illia Polosukhin [Submitted on 12 Jun 2017 (v1), last revised 2 Aug 2023 (this version, v7)]

## Model Architecture

`Transformer`는 Encoder-Decoder의 전체적인 구조를 따르며, 인코더와 디코더 모두에 대해 스택 형태의 'Self-attention'과 'point-wise', 'fully connected layers'을 사용한다.

<img width="709" alt="image" src="https://github.com/user-attachments/assets/4ec68454-01d9-4377-a359-996f34aa71b9">
   _Image Source: [Attention is all you need](https://arxiv.org/pdf/1706.03762)_

- **인코더-디코더 구조**: 트랜스포머 모델의 핵심 아이디어는 입력 데이터를 먼저 압축 'Encoding' 한 뒤, 이를 기반으로 원하는 출력을 점진적으로 'Decoding' 하는 방식입니다. 예를 들어, 입력이 영어 문장이라면, 이를 벡터로 변환(z)한 뒤, 디코더가 이를 기반으로 대응하는 한국어 문장을 생성합니다.
- **auto-regressive**: 이전에 생성한 결과를 다음 결과를 만들 때 참조한다는 의미입니다. 즉, 디코더는 예측된 단어를 입력으로 다시 사용합니다.
- **self-attention**: 입력 데이터의 각 요소가 서로 어떤 연관성을 가지는지 계산하여 중요도를 부여합니다. 예를 들어, 문장에서 어떤 단어가 다른 단어와 긴밀한 관계인지 파악하는 메커니즘입니다.

### 3.1 Encoder and Decoder Stacks

`Encoder`

- 구조: 6개의 동일한 레이어를 쌓아올린 구조
- 레이어 구성: 각 레이어에는 두 개의 하위 레이어가 있다. 첫 번째는 'Multi-head self attention' 메커니즘이고, 두 번째는 위치별로 완전히 연결된 단순한 'Feed-Forward Network'이다. 두 개의 하위 레이어 각각에 잔여 연결을 사용한다. 두 개의 하위 레이어에 잔여 연결을 적용한 다음 레이어 정규화를 수행한다. 이러한 잔여 연결을 용이하게 하기 위해 모델의 모든 하위 레이어와 임베딩된 레이어뿐만 아니라 모델의 모든 하위 레이어는 고정된 차원(512)으로 변환 됩니다.
  - Multi-head Self-Attention: 입력 문장 내 각 단어가 다른 단어들과 어떻게 관련이 있는지 계산
  - Feed-Forward Network: 어텐션 결과를 더 복잡한 패턴으로 변환
  - 잔여 연결(Residual Connection): 입력에 결과를 더해 네트워크가 학습하기 더 쉽게 만든다.
  - 레이어 정규화(Layer Normalization): 계산값의 안정성을 보장한다.

`Decoder`

- 구조: 인코더와 동일하게 6개의 레이어로 구성
- 레이어 구성: 각 인코더 레이어에 있는 두 개의 하위 레이어 외에도 디코더는 세 번째 하위 레이어를 삽입하여 인코더 스택의 출력에 대해 멀티 헤드 인코더 스택의 출력에 대한 'Attention'을 한다. 인코더와 마찬가지로, 각 하위 레이어 주변에 잔여 연결을 사용한 다음 레이어 정규화를 수행한다. 또한 디코더 스택의 'self-attention' 레이어를 수정하여 위치가 후속 위치에 영향을 주지 않도록한다.
- Encoder-Decoder Attention: 디코더가 인코더 출력의 중요한 부분에 집중할 수 있도록 한다.
- Future Masking: 자기회기적 특성을 유지하기 위해, 현재 단어는 미래 단어를 참조하지 않도록 설정한다.

### 3.2 Attention

> 어텐션 함수는 Query와 Key-value 쌍을 출력에 매핑하는 함수로 설명할 수 있다. 이떄 쿼리, 키, 값, 출력은 모두 벡터이다. 출력은 값들의 가중 합(Weighted sum)으로 계산되며, 각 값에 할당된 가중치는 쿼리와 해당 키 간의 호환성 함수에 의해 계산된다.

<img width="709" alt="image" src="https://github.com/user-attachments/assets/84aa963b-9e84-4318-8b70-b78ba873379d">
   _Image Source: [Attention is all you need](https://arxiv.org/pdf/1706.03762)_

- **Attention**: 입력 데이터를 처리할 때 어느 부분에 더 집중할지 결정하는 메커니즘
- **Query**: 현재 내가 '집중하고 싶은 것'
- **Key**: 문장의 각 단어가 가진 '설명서' 같은 역할
- **Value**: 실제로 활용될 정보
- **Weighted Sum**: 쿼리와 키의 관련성이 클 수록, 해당 값에 더 높은 가중치 부여
- **호환성 함수**: 쿼리와 키의 관련성을 측정하는 방법으로 Transformer에서는 주로 'dot product' 방식으로 계산하며, 연산 결과가 클수록 두 벡터가 더 관련이 높다고 판단한다.

#### 3.2.1 Scaled Dot- Product Attention

> Query, Key, Value의 관계를 계산해 출력을 생성하는 메커니즘이다. 출력은 값의 가중합으로 계산되며 가중치는 쿼리와 키 간의 호환성 함수로 결정된다.

`작동 방식`

- 점곱 계산: 쿼리와 키를 점곱 하여 각 쿼리가 모든 키와 얼마나 관련 있는지 계산
- 스케일링: 점곱결과를 \(\sqrt{dk}\)로 나누어 값의 크기를 조정한다.
- 소프트맥스: 점곱 결과를 확률로 변환하여 각 키에 대한 가중치를 생성한다.
- 가중합: 가중치를 값에 곱한 후 합산해 최종 출력을 생성한다.

#### 3.2.2 Multi-Head Attention

> 입력 데이터의 서로 다른 부분을 다양한 관점에서 동시에 분석한다.

- 작동 방식
  - 입력 데이터를 여러 차원으로 나눈다.
  - 각 차원에서 독립적으로 Attention을 수행한다.
  - 결과를 Concat하고 다시 변환해 최종 출력을 생성한다.
    이를 통해 하나의 어텐션 헤드가 놓칠 수 있는 세부 정보를 다수의 어텐션 헤드가 포착 가능하다.

#### 3.2.3 Applications of Attention in our Model

> Transformer는 멀티 헤드 어텐션을 다음 세가지 방식으로 사용한다.

1. Encoder-Decoder Attention
   : 쿼리는 이전 디코더 레이어에서 가져오며, 키 와 값은 인코더의 출력에서 가져온다. 이를 통해 디코더의 각 위치가 입력 시퀀스의 모든 위치를 참조할 수 있다.
2. Self-Attention in the Encoder
   : 인코더의 Self-Attention 레이어 에서는 쿼리, 키, 값 모두 동일한 출저에서 가져 온다. 이를 통해 인코더의 각 위치가 이전 레이어의 모든 위치를 참조할 수 있습니다.
3. Self-Attention in the Decoder
   : 디코더의 Self-Attention 레이어 에서는 디코더의 각 위치가 현재 위치와 그 이전 위치만 참조할 수 있다. auto-regressive 특성을 유지하기 위해 디코더에서 왼쪽으로는 정보 흐름을 방지해아 한다.(스케일드 점곱 어텐션 내에서 특정 연경을 차단하여 구현)

### 3.3 Position-wise Feed-Forward Networks

> 두 개의 선형 변환(Linear Transformation)과 ReLU 활성화 함수 사용을 통한 비선형성 추가의 구성을 가진다. 각 위치에서 독립적으로 처리하며 파라미터의 크기는 512(모델의 기본차원)를 가지고 중간 레이어 크기는 2048을 가진다.

### 3.4 Embeddings and Sotfmax

1. `Embeddings`: 다른 시퀀스 변환 모델들과 마찬가지로, 트랜스포머는 입력 토큰과 출력 토큰을 벡터로 변환하기 위해 학습된 임베딩을 사용한다.
2. `Softmax`: 디코더 출력 벡터를 다음 토큰의 확률로 변환하기 위해 학습된 선형 변환과 소프트맥스 함수를 사용한다.(다음에 나올 단어를 소프트맥스 함수를 적용하여 확률 분포를 계산)
3. `가중치 공유`: 모델에서 두 임베딩 레이어와 소프트맥스 전에 사용되는 선형 변환의 가중치 행렬을 공유한다.(메모리 절약과 효율적인 학습에 기여)
4. `임베딩 레이어에서 가중치 조정`: 임베딩 레이어에서는 가중치 행렬을 **$d_{model}$**로 곱해 조정한다.(임베딩 벡터의 값 크기를 조정해 학습을 더 안정적으로 만든다.)

### 3.5 Positional Encoding

1. `순서 정보 추가의 필요성`: 트랜스포머 모델은 순환 구조(recurrence)나 합성곱(convolution)을 사용하지 않기 떄문에, 시퀀스의 순서 정보를 활용하려면 추가적인 위치 정보를 모델에 주입해야한다.
2. `Positional Encoding`: 입력 임베딩에 위치 인코딩을 더하여, 단어 간의 상대적 또는 절대적 위치 정보를 제공한다. 위치 인코딩은 입력 임베딩과 동일한 차원을 가지며, 두 데이터를 쉽게 더할 수 있다.
3. `sine & cosine 선택 이유`: 주어진 위치의 상대적 차이를 쉽게 학습하도록 돕기 위해 사인과 코사인 함수를 사용

> 위치 인코딩은 트랜스포머 모델에 시퀀스의 순서 정보를 제공합니다. 사인/코사인 방식을 통해 상대적 위치 정보를 효율적으로 학습하고, 긴 시퀀스에도 일반화 가능성을 보장할 수 있습니다.

### 4 Why Self-Attention

1. `Efficiency`: 셀프 어텐션은 병렬 처리가 가능해서 계산속도가 빠르다.
2. `Long-Range Dependency Learning`: 문장에서 멀리 떨어진 단어 간 관계를 학습하기 쉽도록, 입력과 출력 사이 경로 길이가 짧다. 순환 레이어나 합성곱 레이어는 경로 길이가 길어져 학습이 어려울 수 있다.
3. `Interpretability`: 셀프 어텐션은 각 단어가 다른 언어와 어떻게 연결되는지 시각적으로 확인이 가능하다.(문법적 구조나 문장의 의미적 관계를 명확히 드러낸다.)
4. `Simplicity`: 복잡한 순환구조(RNN)이나 다층 합성곱(CNN) 없이도 강력한 성능을 발휘한다.

### 5 Training

- 데이터: 대규모 데이터셋(영어-독일어\_37,000 토큰, 영어-프랑스어\_32,000 토큰)
- 학습환경: GPU 8개(Nvidia P100) (Base: 12시간, Big: 3.5일)
- 최적화 및 정규화: Adam, 학습률 스케줄링, 드롭아웃, 라벨 스무딩
- 성과: 낮은 학습 비용으로 BLEU 점수에서 최고의 성능 달성

### 6 Result

- 기계 번역: 트랜스포머는 기존 최고 성능 모델을 BLEU 점수화 학습 비용 측면에서 능가.
- 모델 변형 실험: 어텐션 헤드와 키/값 크기, 드롭아웃, 위치 인코딩 등 다양한 요소가 성능에 영향을 미친다.
- 구문 분석: 트랜스포머는 구조적 제약이 있는 과제에서도 강력한 일반화 성능을 보인다.
