---
title: "[Paper] Attention is All You Need를 읽어보자(4)_Attention 메커니즘이란?"
writer: chanho
date: 2024-11-14 21:45:00 +0900
categories: [AI, LargeLanguageModel]
tags: [ai, llm, paper]
---

> 오늘은 마지막 사전지식으로 `Attention Mechanism`에 대해 공유하고자 한다. Transformer는 Attention mechanism에 크게 의존한다. 이로 인해 기존의 RNN 기반 모델보다 훨씬 효율적이고 강력한 성능을 발휘할 수 있게 되었다. 특히, Self-Attention은 Transformer의 핵심 요소로, 모델이 시퀀스 내에서 중요한 부분을 선택적으로 강조할 수 있도록 해준다.

# Attention Mechanism

> Attention 메커니즘이란 시퀀스 데이터에서 중요한 부분에 `집중`할 수 있도록 하는 방법이다. 특히 자연어처리(NLP)와 컴퓨터 비전에서 매우 효과적으로 사용된다. 이 메커니즘은 입력 시퀀스의 각 요소에 가중치를 부여하여, 중요한 요소는 더 크게, 덜 중요한 요소는 더 작게 반영함으로써 문맥을 이해하는 데 도움을 준다.

## Attention 메커니즘의 기본 원리

> Attention 메커니즘은 입력 시퀀스의 각 요소가 다른 요소와 얼마나 관련이 있는지를 계산하여 가중치를 할당한다. 이렇게 할당된 가중치(weight)는 모델이 중요하게 여겨야 할 부분을 강조하는 데 사용된다.

<img width="709" alt="image" src="https://github.com/user-attachments/assets/2f3b8876-b6ec-4b58-8771-92804d8b6a53">
   _Image Source: [wekipedia_Attention](https://en.wikipedia.org/wiki/Attention_(machine_learning))_

1. `Query, Key, Value` 벡터 생성: 입력 시퀀스의 각 요소에 대해 Query(Q), Key(K), Value(V) 벡터를 생성합니다. Query는 찾고자 하는 정보의 기준을 나타내는 벡터, Key는 특정 정보(입력 시퀀스의 각 요소가 가지고 있는 고유한 정보)를 나타내는 역할을 하고, Value는 최종적으로 계산된 정보로 가중합을 만들 때 사용되는 벡터입니다. Query, Key, Value 벡터는 입력 임베딩에서 각각 다른 가중치 행렬로 변환하여 생성됩니다. 이는 학습 가능한 가중치로, 모델이 학습 과정에서 적절한 Query, Key, Value 벡터를 조정해 갑니다.

$$
\text{Attention(Q, K, V)} = \text{softmax} \left( \frac{QK^T}{\sqrt{d_k}} \right) V
$$

2. `유사도(가중치) 계산`: 각 Query와 Key 간의 유사도를 계산하여 중요도를 평가합니다. 일반적으로 내적(dot product) 연산을 통해 유사도를 계산합니다. $${QK^T}$$ 는 각 Query와 Key간의 유사도 점수를 계산한 결과로, 이 값이 클수록 Query와 Key가 높은 유사도를 가진다는 의미입니다.

3. `스케일링`: $$\frac{QK^T}{\sqrt{d_k}}$$: Key 벡터의 차원인 \[{d_k}\]로 나누어 스케일링 합니다. 이는 내적 결과값이 너무 커지지 않도록 안정화하여 학습 효율을 높이기 위한 과정입니다. 특히 차원이 커질수록 내적 결과가 커지므로 \[\sqrt{d_k}\]로 나누어줍니다.

- 스케일링 점수값의 크기를 조정하여, 큰 값으로 인해 소프트맥스 함수에서 특정 값들이 과도하게 치우지지 않도록 합니다.

4. `소프트맥스(Softmax) 적용`: $$\text{softmax} ( \frac{QK^T}{\sqrt{d_k}})$$ 스케일링 유사도 값에 소프트맥스 함수를 적용하여, 가중치 값들이 0과 1 사이로 정규화되고 합이 1이 되도록 만듭니다. 이 과정으로 각 입력 요소의 중요도를 표현하는 가중치 벡터가 완성됩니다.

- 소프트맥스: 시퀀스 내에서 어느 요소가 더 중요한지를 나타내며, 가중치를 0과 1사이로 정규화하여 확률처럼 표현합니다.

5. `가중치 계산`: $$\text{softmax} \left( \frac{QK^T}{\sqrt{d_k}} \right) V$$ 소프트맥스를 통해 계산된 가중치를 Value 벡터에 곱하고, 그 결과를 모두 최종 Attention 출력을 만듭니다. 이 가중합은 중요도가 높은 Value 정보는 더 크게 반영하고, 덜 중요한 Value는 작게 반영되므로, 입력 시퀀스에서 중요한 정보가 강조된 출력이 생성됩니다.

## Self-Attention

> Self-Attention은 Attention 메커니즘이 자기 자신에게 적용되는 형태입니다. 시퀀스 내의 모든 단어가 서로의 관계를 학습하는 과정입니다. Self-Attention의 목표는 특정 단어가 문맥에서 다른 언어와 얼마나 관련이 있는지 평가하여, 해당 단어가 문장에서 얼마나 중요한지를 가중치로 반영하는 것입니다.

- `병렬 처리 가능`: 모든 단어가 서로의 관계를 동시에 계산하므로, RNN과 달리 병렬 처리가 가능하여 학습 속도가 매우 빠릅니다.

  > 병렬 처리가 가능하지만 그 만큼 자원 소모가 큰 연산입니다. 시퀀스의 길이가 길어질수록 연산량과 메모리 사용량이 급격히 증가하고. 구체적으로 시간복잡도와 메모리 복잡도가 시퀀스 킬이에 대해 n^2의 비율로 증가합니다. 이를 해소하기 위해 Seare Attention, Local Attention, Efficient Transformer 과 같은 접근법이 나오고 있습니다.

- `장거리 의존성 처리`: Self-Attention은 시퀀스 내의 모든 단어 간의 관계를 한번에 평가하므로, 긴 시퀀스에서도 장거리 읜존성을 효과적으로 학습할 수 있습니다.

## Multi-Head Attention

> Multi-Head Attention은 Self-Attention의 확장 개념으로, 여러 개의 Attention Head를 사용하여 시퀀스 데이터를 다양한 관점에서 바라보게 합니다. 즉, 동일한 입력에 대해 여러 Attention 연산을 병렬로 수행하는 방식입니다. Transformer에서는 Multi-Head Attention을 사용하여 여러 개의 Attention을 병렬로 수행합니다.

`Head란 여러 개의 Self-Attention 연산을 병렬로 수행하는 개별 단위를 의미합니다.`

- `다양한 관점 학습`: 여러 헤드를 통해 다양한 문맥 관계를 학습할 수 있어, 모델이 시퀀스를 보다 정교하게 이해할 수 있습니다.

- `복잡한 관계 포착`: 여러 헤드가 병렬로 작동하며, 각 헤드가 서로 다른 정보를 포착함으로써 복잡한 언어적, 문맥적 관계를 동시에 학습합니다.

## Attention 메커니즘의 장점

1. 문맥적 중요도 반영: 문장에서 중요한 단어에 높은 가중치를 부여하고 문맥을 보다 잘 이해할 수 있게 합니다.
2. 병렬 처리 가능: 모든 입력을 동시에 처리할 수 있어, RNN 기반 모델보다 훨씬 효율적 입니다.
3. 긴 시퀀스 처리: 긴 문장이나 문맥이 긴 데이터에서도 필요한 정보를 학습하는 데 용이합니다.
